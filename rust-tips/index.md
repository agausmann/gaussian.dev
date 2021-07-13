---
title: Rust Tips & Tricks
layout: home
list_category: rust-tips
---

This is a place for me to make mini-posts about the cool things I discover in
the Rust ecosystem. At the time of writing, I've been using Rust for almost 5
years (since late 2016), and it has grown so much since then. While I've been
eager to adopt the cool new features, like the try operator (`?`) and
async/await, there are still many habits of mine that have not evolved as
quickly.

There are tons of new functions added to the standard library with every
release. While they're all in the changelogs, most of them don't get nearly as
much press as the more "major" features, and I personally overlook many of
them. However, there are some really interesting gems in there!

This whole series started when I stumbled across `Option::as_deref`, a neat
shorthand for a family of `Option` adapter chains that I've had to write many
times. It's a relatively recent addition, as it was stabilized in Rust 1.40
(December 2019). But that was still a year and a half ago, and I never really
acknowledged its existence until today.

Discoveries like that are awesome, and I want to share those experiences with
you all! I hope that you can also learn something from them.
