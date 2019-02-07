---
layout: post
title: ECS ramblings
---

# Introduction

This post is mainly written for people familiar with [Amethyst](https://github.com/amethyst/amethyst) and I won't be spending a lot of time
going over the history of the project and how we got to where we are today.

A small summary and some reminders is in order though.

For those who aren't familiar
[I have opinions on how an ECS should be implemented for Amethyst.](https://community.amethyst-engine.org/t/igneous-a-path-to-a-usable-engine/422/).

We have three names flying around for our ECS project so I'll provide a quick summary of them here.

- `specs` The OG amethyst ECS, this has been kicking around for a long time. It's got a few problems
that have manifested for Amethyst in unfortunate ways which inspired two spin off/reboot projects.
- `nitric` [@torkleyy]'s spin on an ECS, developed [here](https://gitlab.com/nitric/nitric) on gitlab.
- `igneous` My proposal for a fork of `specs` that can be used to solve problems with it. Doesn't actually exist beyond a few drafts and proposals.

[@torkleyy]: https://github.com/torkleyy

# Unifying new development work

When I first wrote the igneous proposal I hadn't spoken with torkleyy as much as I should have about my ideas. I'm hoping that with the
creation of this blog I can get better at sharing my ideas. So I'll start with my perspective of what's happened since the igneous post.

Torkleyy reached out to me to express interest in unifying our efforts. Nitric has some interesting ideas and enhancements that igneous
didn't, but some of those ideas and enhancements have made integrating some igneous features more difficult. So we're working towards 
solutions that express the best of both worlds as best we can.

There's a lot of nuance and smaller ideas that I'm not quite as interested in covering here especially because I consider them unproblematic.
So I'll cover the most important ideas.

## Automatic component registration

At one point I wanted to automatically register component types to lift a cognitive load from the user. This has proven difficult, mainly
in that components with generics are very important, but also incomplete types. The type becomes "complete" as soon as the
generics are filled, but I haven't yet figured out how to submit the type to the inventory only when the component type is complete.
I'll keep pondering this, and if any readers have ideas about how this can be done and can make a proof of concept please let me know.

## Macros to generate `System`s using simpler syntax

This is at the top of my priority list, but has been difficult to implement due to...

## Support for registering multiple components and resources of the same type with alternate "keys"

Making this work with macros to generate systems has been difficult, mainly because my existing proposals have been highly
focused around only needing a type.

Here's some alternatives I'm considering

### proc_macro separated keys

```rust
struct System {
    caching_data: i32,
    other: u32,
    stuff: u64,
}

#[system_run("foo", "bar", "baz")]
fn run(system: &mut System, foo_resource: &Foo, bar_resource: &mut Bar, baz_resource: &Baz) {
    // System body here
}
```

This works pretty well and solves most of our problems, my only concern is the fact that keys are separate from the types syntactically.
That might not be such a big deal though.


### decl_macro keys inline

```rust
struct System {
    caching_data: i32,
    other: u32,
    stuff: u64,
}

system_run! {
    fn run(system: &mut System, foo_resource: &Foo;"foo", bar_resource: &mut Bar;"bar", baz_resource: &Baz;"baz") {
        // System body here
    }
}
```

Oh hey! The keys are now adjacent to the types. That's neat. This approach has problems of its own though.

- There's more magic happening behind the scenes here which can make the error messages of this setup confusing
- I used a semicolon to separate the type and key here, but that has a few known collisions with existing Rust syntax. May or may not be a problem if the decl macro can identify a "complete type" vs an incomplete one.
- Even if you use a different delimiter that doesn't collide you can't guarantee Rust won't use that delimiter in the future.

### Just cut the keys

While this was the first thing that came to mind I kind of hate it. There's utility in being able to register multiple components of the
same type, and if we're designing from the ground up I'd like to cut this limitation of specs.

## Resource and data dependencies aka Resource/System initialization

There's a lot of similarities between this problem and the system macros, to the point where I'll declare this problem automatically
solved as soon as we pick a solution for that one.

## Misc .join() improvement mentioned in igneous proposal

Still on track! No reason we can't do this.

## Dynamic system graph support and cutting the gaps in the shred dispatcher

I think both torkleyy and I lack concrete plans for this as of right now, but we're very eager to make this happen.

## What's the name?

Good question, I like `igneous` personally but for now we're still using `nitric`. I tried to raise the topic with @torkleyy but
I didn't get a response on that statement.

