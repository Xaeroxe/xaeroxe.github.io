---
layout: post
title: Init Struct Pattern
excerpt_separator: <!--end_excerpt-->
---

# Introduction

Let's talk initialization of complex structures in Rust. There's a few popular ways to go about this, some of which include
the `pub fn new()` convention and the builder pattern. In this blog post I'm going to compare these, and also introduce a new
pattern which I'm going to call the Init Struct Pattern.

<!--end_excerpt-->

# New

The first and most common way to initialize structures is by declaring a function in the struct with the following signature.

```rust
pub fn new() -> Self {
    Self {
        // init me
    }
}
```

This is pretty straightforward and works well for simple structs. However it starts to have problems as the complexity
of the struct grows. For example,

```rust
impl YakShaver {
    pub fn new(clipper_size: u32, gas_powered_clippers: bool, solar_powered_clippers: bool, color_to_dye_yak: &str) -> Self {
        // Body is irrelevant
    }
}

// In some other file, or maybe even another crate we now have to construct this type.
// Unless you've looked at the definition for `fn new` recently, you might not remember that the second argument
// creates some CO2 emissions if flipped.

let yak_shaver = YakShaver::new(3, false, true, "magenta");
```

There's another problem with this pattern, it makes things breaking changes when they don't have to be. For example, I know
most of my users are going to want their yak clippers to be black, why would you want anything else? Bob wants something else.
Bob is our downstream user who has opinions on clipper colors. He wants them red. Okay, let's add a parameter for clipper colors.


```rust
impl YakShaver {
    pub fn new(...same as above..., clipper_color: &str) -> Self {
        // Body is irrelevant
    }
}

let yak_shaver = YakShaver::new(3, false, true, "magenta", "red");
```

Except now we have a problem. Everyone needs to specify their clipper color, even though 99+% of users want them black. This seems silly and verbose. We also can't
release Bob's new feature until we do a major version release. Otherwise we'd break everyone else's code. Bob's not super happy about having to wait, and
we aren't thrilled about requiring all of our users to specify something that, to them, seems obvious.

Builder pattern to the rescue! We can avoid this situation in the future with some careful planning.

# Builder Pattern

Builders are neat because they don't require us to specify *everything* to build our struct, which means we can add Bob's red clippers in a minor release without breaking
anything. They also prefix every field with its name, which makes the code more readable. For example

```rust
pub struct YakShaverBuilder {
    clipper_size: u32,
    gas_powered_clippers: bool,
    solar_powered_clippers: bool,
    color_to_dye_yak: String,
    clipper_color: String,
}

impl YakShaverBuilder {
    pub fn new() -> Self {
        Self {
            clipper_size: 3,
            gas_powered_clippers: false,
            solar_powered_clippers: true,
            color_to_dye_yak: String::from("brown"),
            clipper_color: String::from("black"),
        }
    }

    pub fn clipper_size(mut self, v: u32) -> Self {
        self.clipper_size = v;
        self
    }

    pub fn gas_powered_clippers(mut self, v: bool) -> Self {
        self.gas_powered_clippers = v;
        self
    }

    pub fn solar_powered_clippers(mut self, v: bool) -> Self {
        self.solar_powered_clippers = v;
        self
    }

    pub fn color_to_dye_yak(mut self, v: String) -> Self {
        self.color_to_dye_yak = v;
        self
    }

    pub fn clipper_color(mut self, v: String) -> Self {
        self.clipper_color = v;
        self
    }

    pub fn build(self) -> YakShaver {
        YakShaver {
            clipper_size: self.clipper_size,
            gas_powered_clippers: self.gas_powered_clippers,
            solar_powered_clippers: self.solar_powered_clippers,
            color_to_dye_yak: self.color_to_dye_yak,
            clipper_color: self.clipper_color,
        }
    }
}

let yak_shaver = YakShaverBuilder::new()
    .clipper_size(4)
    .color_to_dye_yak(String::from("hot pink"))
    .clipper_color(String::from("red"))
    .build();
```

Phew! Still with me? Hopefully this revealed the big downside of the builder pattern. It's very verbose. Somewhere between two and three times as many lines as
`pub fn new() -> Self` depending on how you're counting. So it seems like builder pattern might be overkill for tiny structs, but it comes with too many benefits to be
ignored for large structs. What if we could get the best of both worlds? I hope to achieve that with my next proposal.

# Init Struct Pattern

We can combine a few features of the Rust language to get the same benefits of the builder pattern with much less verbosity. I'll start off with an example.

```rust
pub struct YakShaverInit {
    pub clipper_size: u32,
    pub gas_powered_clippers: bool,
    pub solar_powered_clippers: bool,
    pub color_to_dye_yak: String,
    pub clipper_color: String,
    #[doc(hidden)]
    pub __non_exhaustive: () // This is a hack, we might be able to stop using it in the future.
}

impl Default for YakShaverInit {
    fn default() -> Self {
        Self {
            clipper_size: 3,
            gas_powered_clippers: false,
            solar_powered_clippers: true,
            color_to_dye_yak: String::from("brown"),
            clipper_color: String::from("black"),
        }
    }
}

impl YakShaverInit {
    pub fn init(self) -> YakShaver {
        YakShaver {
            clipper_size: self.clipper_size,
            gas_powered_clippers: self.gas_powered_clippers,
            solar_powered_clippers: self.solar_powered_clippers,
            color_to_dye_yak: self.color_to_dye_yak,
            clipper_color: self.clipper_color,
        }
    }
}

let yak_shaver = YakShaverInit {
    clipper_size: 4,
    color_to_dye_yak: String::from("hot pink"),
    clipper_color: String::from("red"),
    ..Default::default(),
}.init();
```

Looks pretty similar to the builder pattern! Indeed it has a lot of the same benefits. Though we don't need function definitions
for every field, and it doesn't involve returning `Self` several times. If our init needs to do any complex work with the input given to it,
it can do so in `fn init()`. So, we can add new fields to our structure without doing a major release, we're not requiring everyone to specify every field, and we've
significantly reduced the verbosity of our definition in comparison to the builder pattern. I'd call this a win!

I used a couple features here not everyone may be familiar with. What's that `..Default::default()`? That is called struct update syntax, it tells the compiler
to copy all of the remaining fields from the output of `Default::default()` which is defined in `impl Default for YakShaverInit`. This also uses `#[doc(hidden)]`
on a pub field! That hides the field from the docs which should discourage people from adding it to their struct initialization for `YakShaverInit`. If they did,
then they might be able to finish the construction without specifying `..Default::default()` at the end, and that means that their code would break if we added new fields
to the `YakShaverInit` struct. We can't prevent this right now, only discourage it. If Rust added more ways for us to use actual non exhaustive structs, defined with
`#[non_exhaustive]` then we might be able to prevent this in the future making the pattern foolproof. If people like this idea I might write the relevant rust-lang 
RFCs to make this possible.

Adding additional private fields to the init structure is also a breaking change! All fields of it must be public. This seems silly, because the following code is legal.

```rust
pub struct HalfPublic {
    pub a: i32,
    b: u32,
}

impl Default for HalfPublic {
    fn default() -> Self {
        Self {
            a: 0,
            b: 0,
        }
    }
}

let mut half_public = HalfPublic::default();
half_public.a = 10;
```

So I think I'm going to write an RFC to rust-lang suggesting we make the following syntax legal

```rust
let half_public = HalfPublic {
    a: 10,
    ..Default::default(),
}
```

Look out for the RFCs I've mentioned! Anyway I hope you enjoyed this blog post, let me know if you have any comments at kieseljake+blog@gmail.com

