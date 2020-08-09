---
layout: post
title: Personal notes about Rust
category: Rust
---

Closures are "characterized" via traits, the Rust book seems [saying](https://doc.rust-lang.org/book/ch19-05-advanced-functions-and-closures.html) closures are not types:

> Unlike closures, `fn` is a type rather than a trait, ... (!?)

Closures are not [`Sized`](https://github.com/pretzelhammer/rust-blog/blob/master/posts/sizedness-in-rust.md) then, they can only be passed and returned using trait objects. The lifetime constraint on closures is `'static` (this is natural, like functions) by default, but not always.

The following "type" represents a [Parser](https://fsharpforfunandprofit.com/posts/understanding-parser-combinators/) closure whose input is a `str` slice (with lifetime `'i`), the closure itself has lifetime `'p`

```Rust
type Parser<'i, 'p, T> = dyn Fn(&'i str) -> Option<(T, &'i str)> + 'p;
```

then the function `pchar` parses if the first character of the slice is some character `c`

```Rust
fn pchar<'i, 'p, T: From<char> + 'i>(c: char) -> Box<Parser<'i, 'p, T>>
{
    Box::new(move |s: &'i str| s.strip_prefix(c).and_then(|r| Some((c.into(), r))))
}
```

and a combinator

```Rust
fn combine<'i, 'a, 'ra, 'b, 'rb, 'c, A: From<char> + 'i, B: From<char> + 'i>(
    pa: &'ra Parser<'i, 'a, A>,
    pb: &'rb Parser<'i, 'b, B>,
) -> Box<Parser<'i, 'c, (A, B)>>
where
    // 'a: 'i,
    // 'b: 'i,
    // 'ra: 'c,
    // 'rb: 'c,
    // 'c: 'i,
{
    Box::new(move |s: &'i str| {
        pa(s).and_then(|(ra, sa)| pb(sa).and_then(|(rb, sb)| Some(((ra, rb), sb))))
    })
}
```

Some explicit lifetime contraints are needed, first the [implementation](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=689aa33bd8d7f08e6a29efde53e935ae) of `combine` will not compile without either `'ra: 'c` or `'rb: 'c`


Next you can update your site name, avatar and other options using the _config.yml file in the root of your repository (shown below).

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.