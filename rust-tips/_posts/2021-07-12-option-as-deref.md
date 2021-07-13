---
layout: post
title:  "Rust Tip: Option::as_deref and friends"
date:   2021-07-12
---

In Rust, it is not uncommon to have to convert something like `&Option<String>`
into `Option<&str>`. For example, imagine I'm storing an optional text
description in a type, like this:

```rust
pub struct Foo {
    description: Option<String>,
    /* ... */
}
```

If I wanted someone with a reference to a `Foo` to be able to access the
value of the description, I could create a getter, which usually looks like
this:

```rust
impl Foo {
    pub fn description(&self) -> Option<&str> {
        self.description.as_ref().map(|string| string.as_str())

        // Or:
        // self.description.as_ref().map(String::as_str)

        // Or even (because String implements Deref<Target = str>):
        // self.description.as_ref().map(|string| &*string)
    }
}
```

However, you can write this even more concisely, using [`Option::as_deref`]:

```rust
impl Foo {
    pub fn description(&self) -> Option<&str> {
        self.description.as_deref()
    }
}
```

So what's happening here? You can see in the source code (go to the [method
documentation][`Option::as_deref`] and click \[src\]), it looks very similar to
our own long-hand implementation, but more generic. This is what it looks like,
at the time of writing:

```rust 
impl<T: Deref> Option<T> {
    pub fn as_deref(&self) -> Option<&T::Target> {
        self.as_ref().map(|t| t.deref())
    }
}
```

When I call `some_option.as_deref()`, it takes a reference to the inner value, and
dereferences it using the type's `Deref` implementation. In our specific
case, `String` implements `Deref<Target = str>`, so it goes from
an `&Option<String>`, to an `Option<&String>`, to an `Option<&str>`. Just like
our original code!

This works for any type that implements `Deref`. In this case, `String`
implements `Deref<Target = str>`, which allows a `&String` to be converted into
`&str`. In human terms, we would call `String` (a string buffer) the "owned"
string type, and `str` (a string slice) as the "borrowed" string type. [^1]
This implementation is also provided for other "owned" and "borrowed" type
pairs, like `Vec<T>` and `[T]`, `CString` and `CStr`, and even `Box<T>` and
`T`. [^2]

Another example where this may be useful is in implementations of the [`Error`
trait][`Error`]. Suppose I have a error type that looks like this:

```rust
pub struct MyError {
    source: Option<Box<dyn Error + 'static>>,
    /* ... */
}
```

The `Error` trait has a `source()` method that allows you to specify a wrapped
"source" error. In my case, if my error value has a source, I'm storing it as a
trait object in a `Box`, but that has to be adapted to match the type required
by the `source()` method. In the past, I'd write the method like this:

```rust
impl Error for MyError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        self.source.as_ref().map(Box::as_ref)
    }
}
```

This can also be made simpler using `Option::as_deref()`, since `Box<T>`
implements `Deref<Target = T>`:

```rust
impl Error for MyError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        self.source.as_deref()
    }
}
```

# Friends of `Option::as_deref`

In this post I used `Option::as_deref` because it is the simplest (and probably
most useful) variant, but there are several related methods that perform
similar conversions:

- `Option` has another method, [`as_deref_mut`][`Option::as_deref_mut`], which
  does the same thing but for mutable references. Instead of `Deref`, it
requires `T: DerefMut`, and converts a `&mut Option<T>` into `Option<&mut <T as
Deref>::Target>`.

- `Result` has its own [`as_deref`][`Result::as_deref`] and
  [`as_deref_mut`][`Result::as_deref_mut`], which convert `&[mut] Result<T, E>`
into `Result<&[mut] <T as Deref>::Target, &[mut] E>`.

# Conclusion

`Option::as_deref` caught my eye because it was an interesting shorthand for a
method chain that I was used to writing by hand. Frankly, I wasn't sure if I
was going to use it over the alternatives. I generally prefer being explicit
over being concise when writing code; I don't mind having to write a few extra
characters that will take at most a few seconds of my time, and `as_deref` just
seemed to be a bit too complex and obscure for my liking.

However, as I've thought more about it, `as_deref` is quite powerful,
especially because of how versatile, portable, and adaptive it is. As I
mentioned in one of the footnotes ([^2]), in some cases, you could change the
storage type of a value, like `Option<String>` to `Option<Box<str>>`, and
`as_deref` would continue to work just fine with the new type. It would even
produce the exact same output type!

Because of this, I think it has the potential to _reduce_ mental effort and
technical debt. And the potential for confusion caused by its relative
obscurity can also be reduced if it is used more often and becomes more
familiar. So I think I am going to give it a try, and I encourage you to try it
too, if you aren't using it already!

---

# Footnotes

[^1]:
    Technically these semantics are provided by the `Borrow`/`BorrowMut` and
`ToOwned` traits, but in most cases, any type that implements
`Borrow`/`BorrowMut` probably also implements `Deref`/`DerefMut` in the same
way.

[^2]:
    The impl of `Deref<Target = T>` for `Box<T>` is _incredibly useful_ in this
case. If we wanted to change the storage of the string, for example so that it
is a [`Box<str>` instead of a `String`][boxed-str], `Option::as_deref()` would
still work and would still provide us with an `Option<&str>` because of this
impl. And it would also work for replacing `Vec<T>` with a `Box<[T]>`.

[`Option::as_deref`]: https://doc.rust-lang.org/stable/std/option/enum.Option.html#method.as_deref
[`Option::as_deref_mut`]: https://doc.rust-lang.org/stable/std/option/enum.Option.html#method.as_deref_mut
[`Result::as_deref`]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.as_deref
[`Result::as_deref_mut`]: https://doc.rust-lang.org/stable/std/result/enum.Result.html#method.as_deref_mut
[`Error`]: https://doc.rust-lang.org/stable/std/error/trait.Error.html
[boxed-str]: https://users.rust-lang.org/t/use-case-for-box-str-and-string/8295/4
