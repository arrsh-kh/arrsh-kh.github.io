---
layout: post
author: Arrsh Khusaria
tags: NADIA spacial-audio
---

# From Instagram Reels to Open Source Audio: The Story Behind NADIA

What started as mindless scrolling through Instagram reels turned into a deep dive into professional audio technology and ultimately, the creation of an open-source alternative to proprietary spatial audio systems.

## The Instagram Rabbit Hole

A few months ago, while doom-scrolling through Instagram reels, I stumbled across something that stopped me mid-scroll: someone showcasing their spatial sound design of a scene from *Kimetsu no Yaiba* (Demon Slayer) inside a Dolby studio. The immersive audio experience was captivating, and I found myself wondering what software they used to create such precise spatial positioning.

After some research, I discovered it was Dolby Studios and it was expensive as hell. But the subject had already hooked me. I opened my laptop and started diving deep into speaker layouts in Dolby Atmos, determined to replicate something similar by taking stereo tracks and applying some FFmpeg magic.

## The FFmpeg Adventures (And Failures)

I tried everything I could think of:
- Converting audio to multiple channels—didn't work
- Splitting tracks by frequencies—didn't work  
- Using a CNN to separate instruments from tracks and assign them to different speaker channels—still didn't work

After countless attempts, I came to a frustrating conclusion: either FFmpeg couldn't handle the entire process, or I was missing something fundamental (though I'd like to think my FFmpeg skills aren't terrible). Eventually, I gave up.

## The Night Club Revelation

Two months later, I was watching a techno DJ set when I noticed an unusual set of speakers. The design was unlike anything I'd seen before. I wanted to know the manufacturer, so I checked the description and found the venue was Drumsheds, but couldn't find information about their specific speaker brand. Disappointed but curious, I had a new idea: what would it take to build my own nightclub like Drumsheds in India?

I asked ChatGPT (my "ex-girlfriend" as I like to call it), and it suggested VOID Acoustics speakers. I found those funky-looking speakers, the VOID Acoustics Air series. As someone who grew up with typical JBL speakers, I was intrigued by their unconventional design and wondered how they actually worked.

After more pondering with ChatGPT on the Night Club, it suggested L-Acoustics.

## Discovering the Gold Standard

I wondered why these generic-looking speakers were considered the gold standard in professional audio. My research revealed that L-Acoustics invented line arrays. But why do they get so much credit for this? I could understand the sound quality aspect, but what's so special about line arrays? Hadn't anyone thought of connecting multiple speakers and hanging them before?

Then I learned about the real magic: people said L-Acoustics speakers were so good that audiences couldn't tell where the speakers were located—the sound remained consistent throughout the entire venue. This led me to discover the incredible science behind what seems like such a simple concept, and eventually to NADIA and L-ISA (L-Acoustics Immersion Sound Architecture).

My mind was blown. The amount of science and engineering behind what appears to be just "hanging speakers together" is incredible! I can't explain most of it adequately, but you can watch this excellent explanation to understand the complexity involved. It can adjust specific audio ranges to specific speakers so every place is the best place in the house and their DJ support can do it in real time!

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden;">
  <iframe src="https://www.youtube.com/embed/fEEFB3shPbE" 
          style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;" 
          frameborder="0" allowfullscreen>
  </iframe>
</div>


This is the comparison with and without L-ISA active:

<p float="left">
  <img src="https://www.l-acoustics.com/wp-content/uploads/2025/01/REV-TEXT-With-Hyperreal.jpg" width="45%" />
  <img src="https://www.l-acoustics.com/wp-content/uploads/2025/01/REV-TEXT-Without-Hyperreal.jpg" width="45%" />
</p>

## The Closed Source Wall

I wanted to dig deeper into the technology, but then I hit the wall that changed everything: **closed source**. 

If I wanted to explore further, I would need to buy the L-ISA software AND a significant number of L-Acoustics speakers, since their system only works with their own hardware. This proprietary lock-in frustrated me to the point where I decided to take action.

I wanted to build something myself—and make it open source so others could improve it, use it for free, and benefit from community contributions.

## Building NADIA

Under the GPL 2.0 license, I created NADIA with the ambitious goal of making it comparable to both Dolby Atmos and L-ISA and making it real time, but with the power of open-source collaboration behind it.

The vision is simple: spatial audio technology shouldn't be locked behind expensive proprietary systems as it refuses constant development in exchange of money. With NADIA, I hope to democratize access to professional-grade spatial audio tools and let the community drive innovation forward.

## What's Next?

NADIA is still in its early stages, but the foundation is there. I believe that with community support and contributions, we can create something that rivals the proprietary solutions while remaining free and open for everyone to use and improve.

The journey from Instagram reels to building an open-source spatial audio system has been unexpected, but it perfectly illustrates how curiosity, frustration with closed systems, and the power of open-source development can lead to something meaningful.

---

*Interested in contributing to NADIA or learning more about the project? [github.com/arrsh-kh/nadia]*
