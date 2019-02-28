---
layout: post
title: Rewriting software and how to avoid it
---

Rewrites. Ugh what a gross word in software development. Believe me, I know all about them. I've been writing software for over
nine years, and I've spent most of it rewriting things because I'm a perfectionist. This post is about how you can avoid the
problems I've had, and when a rewrite might actually be the right thing to do.

EDIT (2019-02-28 11:22 UTC): I've received some confusion about what constitutes a rewrite, so for the purposes of this article
I'm defining rewrite as "any operation that halts bug fixes and enhancements for an excessively long duration."

# What causes rewrites?

I've identified a few main causes of software rewrites.

- Framework doesn't scale and suddenly we have to deal with much more traffic than anticipated.
- Tools we're using are slowing down development a lot and we can't keep up with demand because of it.
- Framework got deprecated.
- Framework has adverse effect on hardware for some reason or another.
- No way to fulfill a software requirement with chosen framework.
- Team found a new toy and is excited about it.
- Code Quality was so low the project was deemed not worth keeping around.

You may notice a lot of these are due to a framework, which leads to my first piece of advice:

## Only use a framework for long term projects if the framework has seen heavy production use

For this post, framework is any software dependency with strong opinions on how your code is executed, or how it interfaces
with end-user hardware. As a general rule of thumb, if most of your world revolves around a dependency, that dependency is
a framework.

Back to the advice, please keep in mind when choosing a framework that this decision will follow you for the lifetime of the
software. Know the weaknesses of your framework, and make sure you're comfortable with them. One of those weaknesses may be
"developers aren't committed to long term support". Finally, keep in mind that "no framework" might actually be your best choice.
No framework usually forces you to write your abstractions into your user code, giving you complete control over them, reducing
the likelihood you'll need to perform a full rewrite. This approach also helps cut down on unnecessary resource bloat.

## Take some developer time periodically to stop building features and fixing bugs

Wow that's a weird one, but I've seen many companies brought to their knees by refusing to take time to address technical debt.
Make sure some portion of regular developer time is devoted to not working on tickets. This allows the developers time to
put things in order and build useful high level abstractions which they can use to resolve tickets faster. Lumberjacks need to
sharpen their tools sometimes, developers have similar needs even if they're more abstract and harder to describe. Question your
devs periodically on if they feel like the time allotment given for this is appropriate. They might need more, they might need
less.

## Do not allow fewer than 3 developers to work long term on any project with monetary value

Devs can and often do outsmart themselves. They create abstractions that introduce more cognitive load than is necessary, or
that aren't necessary because the standard library or a framework already in use has a similar abstraction. Keeping them social
prevents too much of this insanity from occurring. Developer isolation is ok for short durations, but try and keep this from
being a permanent arrangement.

## Require code to be reviewed before it's checked in

Most code should get at least one peer approval before it's sent in to the project. In my experience more than 3 reviewers
can result in too many pedantic comments which can be demoralizing, but every project should strive to have changes reviewed by
at least one other programmer.

## Hold architecture meetings and write down your thoughts

These should be as lightweight as possible as they can rapidly spiral into huge resource sucks, but before taking on a large
project the development team should have a very rough idea of what the high level code will look like and how the separate
pieces will interface with one another. I also find journalling and/or blogging to be a really useful way to collect and compare
large amounts of thoughts. It's made me much more confident in my decisions.

## Rewrites are a big deal. Do not undertake them lightly.

Most senior devs don't need to be told this twice, but please keep in mind that rewrites are the single largest work item
you can make for yourself. Make sure that the rewrite you're planning on undertaking will have real measurable benefits for
the bottom line of the business. Try and take your passion out of the equation here. Get multiple opinions. Is there any
other way to get these same benefits that doesn't involve a complete rewrite?

# So when should I rewrite?

An ounce of prevention is worth a pound of cure, but sometimes you just got to bite the bullet and do it. If operations
simply cannot continue unless a rewrite is performed, or you anticipate this may happen in the future, it might be a good time
to rewrite and change your technique. Make sure before you've done this that you've done a comprehensive retrospective about
how you got to this point, and how you're going to avoid arriving at it again.

I hope this post was informative, if you liked this post or feel I got something wrong feel free to email me at
[kieseljake@gmail.com](mailto:kieseljake@gmail.com)

