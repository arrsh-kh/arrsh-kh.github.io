---
layout: post
author: Arrsh Khusaria
tags: nadia spacial-audio progress-report
---

---

## Introduction

The very first thing and the most important thing in a spatial audio system is the speaker position, so I thought that's a good start.

When I started this project, my goal was simple: make every seat in the venue the best seat. No more dead zones, no more spots where the audio sounds like garbage. Just consistent, high-quality sound coverage everywhere.

---

The very first speaker pos looked like this.

## Setting Up the Foundation

### Defining the Environment

The very first speaker pos looked like this.

First I defined the Environment struct:

```c
typedef struct {
	float width;
	float length;
	float deptht;
} environment;
```

`width` is the width of the venue, `length` is the height, `depth` is how deep is the venue.

I considered the stage as a 2d plain, thats why height is length. Don't ask me about the naming conventions.

### Speaker Types and Specifications

Then the speaker-type enum:

```c
typedef enum {
    SPEAKER_TOP,
    SPEAKER_SUB,
    SPEAKER_MONITOR,
    SPEAKER_FILL,
    SPEAKER_ARRAY
} SpeakerType;
```

And then the speaker info that actually matters:

```c
typedef struct {
    char model[32];
    float yaw_deg;
    float spl_peak_dB;
    float bandwidth_low_Hz;
    float bandwidth_high_Hz;
    float horiz_dispersion_deg;
    float vert_dispersion_deg;
    float max_throw_m;
    float weight_kg;
    float box_height;
    int num_lf;
    int num_mf;
    int num_hf;
} Speaker;
```

This struct contains all the physical and acoustic properties I need to calculate optimal positioning.

---

## The Ellipse Coverage Model

The next thing I thought was about the speaker coverage. To make every seat the best seat I thought why not make each speaker cover a patch on the ground so every speaker covers an individual patch on the ground. And the first thing that came when wondering how to do it was of course line arrays. Each row of a line array covers an entire line of space.

Since the area on the ground is an ellipse I needed the major and minor axis of the ellipse to calculate the spread.

Here's the math that calculates the ellipse dimensions:

```c
void get_ellipse_axes(
    float height,
    float tilt_deg,
    float v_disp_deg,
    float h_disp_deg,
    float *major_axis,
    float *minor_axis
) {
    float alpha = tilt_deg * M_PI / 180.0f;
    float theta = v_disp_deg * M_PI / 180.0f;
    float phi   = h_disp_deg * M_PI / 180.0f;

    if (major_axis != NULL) {
        *major_axis = height * (tanf(alpha + theta/2) - tanf(alpha - theta/2));
    }
    if (minor_axis != NULL) {
        *minor_axis = 2.0f * height * tanf(phi / 2.0f);
    }
}
```

I didn't use the area of the ellipse to calculate it because you will always have gaps between the ellipses and only ground speakers can cover that area.

The beauty of this approach is that it gives us precise control over coverage patterns. Each speaker's elliptical footprint can be calculated exactly, allowing us to eliminate gaps and minimize overlap.

---

## Calculating Optimal Tilt Angles

And to calculate how much tilt is needed for the top speaker to cover the end of the venue I did more math!

```c
float compute_tilt_for_end_distance(float height, float vert_disp_deg, float target_distance) {
    float theta = vert_disp_deg * M_PI / 180.0f;
    float alpha = atanf(target_distance / height) - theta / 2.0f;
    return alpha * 180.0f / M_PI;
}
```

I loved doing the math here and it was also funny because just before this I was watching a 3blue1brown video where he proved that "A 2D plain passing through a cone makes an ellipse on the plain" which is what I am doing here.

The timing couldn't have been more perfect!

---

## The Array Generation Algorithm

And then I make a for loop that calculates how many speakers and what tilt are needed to cover one line spread. I store the major axis of the previous speaker's ellipse and so the next speakers ellipse starts directly below. I also included how speakers position shift backwards the lower they get. And also added a condition which prevents the end facing backwards. I divided the width of the venue by the minor axis of the topmost speaker so I can get the number of line arrays needed to cover the whole width.

The algorithm handles several critical details:
- **Speaker spacing**: Each speaker is positioned 0.5m below the previous one
- **Backshift calculation**: When speakers tilt, they physically move backwards due to their box height
- **Coverage continuity**: Each speaker's coverage ellipse starts exactly where the previous one ends
- **Boundary conditions**: Prevents speakers from tilting so far they face backwards
- **Array distribution**: Calculates optimal spacing between multiple arrays to cover venue width

```c
for (int a = 0; a < num_arrays; a++) {
    float array_x = start_x + a * step;
    float target_end = hall.depth;

    printf("Array %d @ x = %.2f m\n", a + 1, array_x);

    for (int i = 0; ; i++) {
        float current_height = base_height - i * speaker_spacing;
        if (current_height <= target_floor) break;

        float pitch = compute_tilt_for_end_distance(current_height, mk2.vert_dispersion_deg, target_end);
        float pitch_rad = pitch * M_PI / 180.0f;
        float vertical_drop = mk2.box_height * sinf(pitch_rad);
        float backshift     = mk2.box_height * (1.0f - cosf(pitch_rad));
        float mouth_height  = current_height - vertical_drop;

        float major = 0.0f, minor = 0.0f;
        get_ellipse_axes(mouth_height, pitch, mk2.vert_dispersion_deg, mk2.horiz_dispersion_deg, &major, &minor);
        float target_start = target_end - major;
        if (target_start <= 0.0f) break;

        printf("  Speaker %d - Height: %.2f m, Pitch: %.2f°, Covers: %.2f → %.2f m, Width: %.2f m\n",
               i + 1, current_height, pitch, target_start, target_end, minor);

        target_end = target_start;
    }
    printf("\n");
}
```

This algorithm works backwards from the target distance, calculating each speaker's position and tilt to create seamless coverage. The key insights are:

- **Vertical positioning**: Each speaker drops by a fixed spacing amount
- **Backshift compensation**: Tilted speakers shift backwards, affecting their effective position  
- **Seamless coverage**: Each speaker's coverage starts exactly where the previous one ends
- **Array spacing**: Multiple arrays across the venue width ensure full horizontal coverage

---

## Results: The Moment of Truth

This prints out the placement height, tilt, and the distance covered (major axis) of each speaker and the spacing between them:

```
== Venue Setup ==
Venue Width: 30.00 m, Depth: 50.00 m

== Main Arrays (14.00 m spacing) ==

     |     |     |
     |     |     |
     |     |     |
     |     |     |
     |     |     |
     |     |     |
     |     |     |
     |     |     |
     |     |     |
     |     |     |
[Array 1][Array 2][Array 3]

Array 1 @ x = 1.00 m
  Speaker 1 - Height: 14.00 m, Pitch: 69.36°, Covers: 29.86 → 50.00 m, Width: 27.06 m
  Speaker 2 - Height: 13.50 m, Pitch: 60.67°, Covers: 20.10 → 29.86 m, Width: 26.13 m
  Speaker 3 - Height: 13.00 m, Pitch: 52.10°, Covers: 14.18 → 20.10 m, Width: 25.21 m
  Speaker 4 - Height: 12.50 m, Pitch: 43.59°, Covers: 10.09 → 14.18 m, Width: 24.31 m
  Speaker 5 - Height: 12.00 m, Pitch: 35.06°, Covers: 7.02 → 10.09 m, Width: 23.43 m
  Speaker 6 - Height: 11.50 m, Pitch: 26.41°, Covers: 4.56 → 7.02 m, Width: 22.56 m
  Speaker 7 - Height: 11.00 m, Pitch: 17.50°, Covers: 2.47 → 4.56 m, Width: 21.70 m
  Speaker 8 - Height: 10.50 m, Pitch: 8.23°, Covers: 0.61 → 2.47 m, Width: 20.86 m

Array 2 @ x = 15.00 m
  Speaker 1 - Height: 14.00 m, Pitch: 69.36°, Covers: 29.86 → 50.00 m, Width: 27.06 m
  Speaker 2 - Height: 13.50 m, Pitch: 60.67°, Covers: 20.10 → 29.86 m, Width: 26.13 m
  Speaker 3 - Height: 13.00 m, Pitch: 52.10°, Covers: 14.18 → 20.10 m, Width: 25.21 m
  Speaker 4 - Height: 12.50 m, Pitch: 43.59°, Covers: 10.09 → 14.18 m, Width: 24.31 m
  Speaker 5 - Height: 12.00 m, Pitch: 35.06°, Covers: 7.02 → 10.09 m, Width: 23.43 m
  Speaker 6 - Height: 11.50 m, Pitch: 26.41°, Covers: 4.56 → 7.02 m, Width: 22.56 m
  Speaker 7 - Height: 11.00 m, Pitch: 17.50°, Covers: 2.47 → 4.56 m, Width: 21.70 m
  Speaker 8 - Height: 10.50 m, Pitch: 8.23°, Covers: 0.61 → 2.47 m, Width: 20.86 m

Array 3 @ x = 29.00 m
  Speaker 1 - Height: 14.00 m, Pitch: 69.36°, Covers: 29.86 → 50.00 m, Width: 27.06 m
  Speaker 2 - Height: 13.50 m, Pitch: 60.67°, Covers: 20.10 → 29.86 m, Width: 26.13 m
  Speaker 3 - Height: 13.00 m, Pitch: 52.10°, Covers: 14.18 → 20.10 m, Width: 25.21 m
  Speaker 4 - Height: 12.50 m, Pitch: 43.59°, Covers: 10.09 → 14.18 m, Width: 24.31 m
  Speaker 5 - Height: 12.00 m, Pitch: 35.06°, Covers: 7.02 → 10.09 m, Width: 23.43 m
  Speaker 6 - Height: 11.50 m, Pitch: 26.41°, Covers: 4.56 → 7.02 m, Width: 22.56 m
  Speaker 7 - Height: 11.00 m, Pitch: 17.50°, Covers: 2.47 → 4.56 m, Width: 21.70 m
  Speaker 8 - Height: 10.50 m, Pitch: 8.23°, Covers: 0.61 → 2.47 m, Width: 20.86 m
```

Perfect! Three arrays of 8 speakers each provide complete coverage of the venue, with each speaker precisely positioned and angled to eliminate gaps while minimizing overlap.

---

So this was the first version of the speaker_pos.c

See you in Progress report #2

---
