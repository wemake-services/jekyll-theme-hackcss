---
layout: post
title: Programming is (NP?) Hard 
date: 2017-05-09 16:48:00
categories: programming
short_description: It's really easy to make "silly" mistakes
image_preview: https://pbs.twimg.com/profile_images/3074406340/16ba89878ed12c976424200c68233693_400x400.png 
---

tl;dr;

Programming is hard. I made some really silly mistakes. Avoid globals, embrace code review and tests.

# Motivation

When I was speaking with my grad school advisor a few weeks ago, we had the idea do some basic analysis on `unsafe` usage in Rust. It's a fairly simple concept, so I quickly wrote up something to read a file line-by-line and count lines in `unsafe` functions or blocks.

Fun story time: Making sure that the script doesn't think the unsafe block/function ends too early is a simplification of one of my favorite "easy" programming questions -- verify that a statement with various delimiters is valid (e.g. that the open and close delimiters match). You can probably see that in my code since I love pushing delimiters to stacks (even when I don't need to). Enough about my weird hobbies.

I was then distracted by a conference for a week, and when I returned to work yesterday, I finished my script off by writing something to interact with Github. The idea is that you could run this over all Rust projects in Github (or some subset--I just wrote it yesterday people!) and see how people are using `unsafe.`

Instead of discussing what we might be able to learn from the script's correct results, this post is about the ways I messed it up and then posted the mistakes to twitter.

# Mistake: abuse of global variables
Yup. I stored all of my counts in globals. Yes, I heard my Intro to Programming professor's voice in my head saying "DON'T use globals unless you have to!!!"

I forgot to clear the counts between repositories.

I noticed this right as I published a tweet about my work. "Why are these numbers monotonically increasing," I thought. Then I screeched and immediately deleted it.

![Globals abuse]({{ site.url }}/assets/mistake0.png)

# Mistake: abuse of endless if/if/elif/else/more ifs

Ok, so that was a pretty gigantic mistake, right? Every real developer knows to avoid globals and if you don't, make sure you reset them. Oops. But at least I fixed it before I told The Whole World (who reads Rust-related tweets) about this. That's great--so I [tweeted](https://twitter.com/avadacatavra/status/861663413570764800). I even [mentioned](https://twitter.com/avadacatavra/status/861663592424378369) my near miss!

As it turns out, I had another glaring mistake. This morning, I started adding some stats on how many `panic!`s there are. As I read over my code, I thought, "something is wrong here..." 

```
 for line in open(filename).readlines():
        # skip comment lines
        if re.match(regex, line):
            comment += 1
        # skip blank lines
        if len(line.strip()) == 0:
            blank += 1

        code += 1
        #...
```

In case you missed it (like I did), I say I'm skipping comments/blank lines **and then I don't**! It's an easy fix, but why did I decide to make such an awful cascade of `if`s? No clue. Seemed like a good idea at the time.

![Endless if]({{ site.url }}/assets/mistake1.png)

# Mistake ?

So this is where I'm currently at. It doesn't 100% match up with the comparable `cloc` stats, but it's a *lot* closer.

![More mistakes?]({{ site.url }}/assets/mistakex.png)

# Lessons

Don't be like [me](https://twitter.com/avadacatavra/status/861909509400395776).

## Listen to your professors

Avoid globals. Please don't take my degree away.

## Make your code easier to read

If it's easier to read, it's easier to catch silly mistakes (this is why I avoid Perl).

## Code reviews are the best

Someone else definitely would have caught this in a review (I hope)

## Testing is also the best

If I'd initially compared my results to `cloc` output, I probably would have caught these mistakes earlier and not told all of Twitter.

# Conclusions

## Security related

The errors I detail here are logic errors--I, the programmer, did something horribly wrong. This is a *huge* problem in security implementations. I work on practical security and implementations...so...*buries face in formal verification papers*

## Rust related

Rust wouldn't have saved me from these particular mistakes, but it has saved me from tons of other mistakes, like typos that would result in similar logic errors.

## Life(?) related

I've been programming for 8 years. I've had one job or another (that pays me to program!) for 5 of those years. In college, I helped teach intro to programming (do as I say, not as I do, please). I still make really silly mistakes.

Some developers will tell you that real devs don't make mistakes like this. If that's the case, then I guess I'm not a real dev (and I'm fine with that). I think it's probably more likely that they're lying or they don't do much work.



