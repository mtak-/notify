# Notify

[![» Crate](https://flat.badgen.net/crates/v/notify)][crate]
[![» Docs](https://flat.badgen.net/badge/api/docs.rs/df3600)][docs]
[![» CI](https://flat.badgen.net/travis/passcod/notify/main)][build]
[![» Downloads](https://flat.badgen.net/crates/d/notify)][crate]
[![» Conduct](https://flat.badgen.net/badge/contributor/covenant/5e0d73)][coc]
[![» Public Domain](https://flat.badgen.net/badge/license/CC0-1.0/purple)][cc0]

_Cross-platform filesystem notification library for Rust._

**Caution! This is unstable code!**

You likely want either [the latest 4.0 release] or [5.0.0-pre.1].

[the latest 4.0 release]: https://github.com/passcod/notify/tree/v4.0.12#notify
[5.0.0-pre.1]: https://github.com/passcod/notify/tree/v5.0.0-pre.1#notify

(Looking for desktop notifications instead? Have a look at [notify-rust] or
[alert-after]!)

- **incomplete [Guides and in-depth docs][wiki]**
- [API Documentation][docs]
- [Crate page][crate]
- [Changelog][changelog]
- Earliest supported Rust version: **1.32.0**

As used by: [alacritty], [cargo watch], [cobalt], [docket], [mdBook], [pax]
[rdiff], [rust-analyzer], [timetrack], [watchexec], [xi-editor], and others.

## Notify is abandoned

Sorry.

Notify has been years of my life and as much as it’s a tough decision, I’m also
greatly relieved. It’s been great, it’s been not so great; it’s now time. I got
some distance, took a hard look at it all, and realised I don’t want to do this
any more. For way longer than I should have let this go on for, Notify sparked
negative joy, and I’m Marie-Kondo-ing it out.

The logistics: several people have commit bit, and several people have publish
bit, and the project is also covered by the
[Rust Bus](https://users.rust-lang.org/t/bus-factor-1-for-crates/17046).

If you want to take over or get commit/publish bits and you’re a
trusted/respected community member, just ask. If you’re not a trusted/respected
community member, try forking first.

I will not merge PRs, I will not commit unless it’s an emergency, I will not
respond to issues or comments unless I really really feel like it, and the goal
is total disengagement.

So Long 🔭 And Thanks For All The Fish 🐬

## Installation

```toml
[dependencies]
crossbeam-channel = "0.3.8"
notify = "5.0.0-pre.2"
```

## Usage

The examples below are aspirational only, to preview what the final release may
have looked like. They may not work. Refer to [the API documentation][docs] instead.

```rust
use notify::{RecommendedWatcher, RecursiveMode, Result, watcher};
use std::time::Duration;

fn main() -> Result<()> {
    // Automatically select the best implementation for your platform.
    // You can also access each implementation directly e.g. INotifyWatcher.
    let mut watcher = watcher(Duration::from_secs(2))?;

    // Add a path to be watched. All files and directories at that path and
    // below will be monitored for changes.
    watcher.watch("/home/test/notify", RecursiveMode::Recursive)?;

    // This is a simple loop, but you may want to use more complex logic here,
    // for example to handle I/O.
    for event in &watcher {
        match event {
            Ok(event) => println!("changed: {:?}", event.path),
            Err(err) => println!("watch error: {:?}", err),
        };
    }

    Ok(())
}
```

### With a channel

To get a channel for advanced or flexible cases, use:

```rust
let rx = watcher.channel();

loop {
    match rx.recv() {
        // ...
    }
}
```

To pass in a channel manually:

```rust
let (tx, rx) = crossbeam_channel::unbounded();
let mut watcher: RecommendedWatcher = Watcher::with_channel(tx, Duration::from_secs(2))?;

for event in rx.iter() {
    // ...
}
```

### With precise events

By default, Notify issues generic events that carry little additional
information beyond what path was affected. On some platforms, more is
available; stay aware though that how exactly that manifests varies. To enable
precise events, use:

```rust
use notify::Config;
watcher.configure(Config::PreciseEvents(true));
```

### With notice events

Sometimes you want to respond to some events straight away, but not give up the
advantages of debouncing. Notice events appear once immediately when the occur
during a debouncing period, and then a second time as usual at the end of the
debouncing period:

```rust
use notify::Config;
watcher.configure(Config::NoticeEvents(true));
```

### With ongoing events

Sometimes frequent writes may be missed or not noticed often enough. Ongoing
write events can be enabled to emit more events even while debouncing:

```rust
use notify::Config;
watcher.configure(Config::OngoingEvents(Some(Duration::from_millis(500))));
```

### Without debouncing

To receive events as they are emitted, without debouncing at all:

```rust
let mut watcher = immediate_watcher()?;
```

With a channel:

```rust
let (tx, rx) = unbounded();
let mut watcher: RecommendedWatcher = Watcher::immediate_with_channel(tx)?;
```

### Serde

Events can be serialisable via [serde]. To enable the feature:

```toml
notify = { version = "5.0.0-pre.2", features = ["serde"] }
```

## Platforms

- Linux / Android: inotify
- macOS: FSEvents
- Windows: ReadDirectoryChangesW
- All platforms: polling

### FSEvents

Due to the inner security model of FSEvents (see [FileSystemEventSecurity]),
some event cannot be observed easily when trying to follow files that do not
belong to you. In this case, reverting to the pollwatcher can fix the issue,
with a slight performance cost.

## License

Notify was undergoing a transition to using the
[Artistic License 2.0][artistic] from [CC Zero 1.0][cc0]. A part of
the code is only under CC0, and another part, including _all new code_ since
commit [`3378ac5a`], is under _both_ CC0 and Artistic. When the project was to be
entirely free of CC0 code, the license would be formally changed (and that would
have incurred a major version bump). As part of this, contributions to Notify since
would agree to release under both.

[`3378ac5a`]: https://github.com/passcod/notify/commit/3378ac5ad5f174dfeacce6edadd7ded1a08d384e

## Origins

Inspired by Go's [fsnotify] and Node.js's [Chokidar], born out of need for
[cargo watch], and general frustration at the non-existence of C/Rust
cross-platform notify libraries.

Written by [Félix Saparelli] and awesome [contributors].

[Chokidar]: https://github.com/paulmillr/chokidar
[FileSystemEventSecurity]: https://developer.apple.com/library/mac/documentation/Darwin/Conceptual/FSEvents_ProgGuide/FileSystemEventSecurity/FileSystemEventSecurity.html
[Félix Saparelli]: https://passcod.name
[alacritty]: https://github.com/jwilm/alacritty
[alert-after]: https://github.com/frewsxcv/alert-after
[artistic]: ./LICENSE.ARTISTIC
[build]: https://travis-ci.org/passcod/notify
[cargo watch]: https://github.com/passcod/cargo-watch
[cc0]: ./LICENSE
[changelog]: ./CHANGELOG.md
[cobalt]: https://github.com/cobalt-org/cobalt.rs
[coc]: http://contributor-covenant.org/version/1/4/
[contributors]: https://github.com/passcod/notify/graphs/contributors
[crate]: https://crates.io/crates/notify
[docket]: https://iwillspeak.github.io/docket/
[docs]: https://docs.rs/notify/5.0.0-pre.1/notify/
[fsnotify]: https://github.com/go-fsnotify/fsnotify
[handlebars-iron]: https://github.com/sunng87/handlebars-iron
[hotwatch]: https://github.com/francesca64/hotwatch
[mdBook]: https://github.com/rust-lang-nursery/mdBook
[notify-rust]: https://github.com/hoodie/notify-rust
[pax]: https://pax.js.org/
[rdiff]: https://github.com/dyule/rdiff
[rust-analyzer]: https://github.com/rust-analyzer/rust-analyzer
[serde]: https://serde.rs/
[timetrack]: https://github.com/joshmcguigan/timetrack
[watchexec]: https://github.com/mattgreen/watchexec
[wiki]: https://github.com/passcod/notify/wiki
[xi-editor]: https://xi-editor.io/
