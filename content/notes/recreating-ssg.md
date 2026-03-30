---
{
    "title": "Recreating the SSG from Scratch",
    "date": "2026-03-28",
    "description": "Why I built my own Static Site Generator instead of using modern frameworks.",
    "keywords": "Recreational Programming, Rust, custom SSG, Raditya Mahatma Ghosi, SiputBiru",
    "draft": true
}
---

## Motivation

First things first, we need to talk about why I am even doing this.

Before we jump into *how* I built [Cangkang](https://github.com/SiputBiru/cangkang)[^1], I originally planned to create this blog using something standard like Astro. But recently, I was watching a video by Gingerbill[^2], the creator of the Odin Programming Language[^3], where he created an SSG completely by himself. It made me stop and wonder: *"Why don't I just do it myself?"*

Usually, when developers want to spin up a blog, we reach for the usual suspects—React, Astro, or Vite SSG. But honestly, I think that is just too boring. I wanted to build this from the ground up.

## How

With the motivation out of the way, let's jump right into how I actually built this thing.

### Project Plan

Everything starts with a plan. I wanted to build this SSG engine entirely from scratch with zero external dependencies, written purely in Rust. Why Rust? I had heard that Rust has some really powerful string manipulation capabilities, so I just wanted to put it to the test.

### Lexer

### Parser

### FileSystem

###

[^1]: *Cangkang* is the Indonesian word for "shell". Since I go by SiputBiru (Blue Snail), it felt fitting that the engine hosting my notes would be my shell

[^2]: [Gingerbill "I Built My Own Static Site Generator With Odin in an Afternoon!"](https://www.youtube.com/watch?v=YvnTsiIFXeI)

[^3]: [Odin Programming Language](https://odin-lang.org/)
