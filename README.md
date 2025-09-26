# tokio-pager: An asynchronous, Tokio-friendly pager

Unfortunately, Rust's [pager] crate does not play nicely with Tokio.
It leaves threads open after the Tokio runtime exits, resulting in a
nasty I/O error after a CLI program using both pager and a Tokio runtime
exits. This may be due to the fact that the pager crate actually runs the
pager in the _parent_ process, meaning that the Tokio runtime, in the
child process, exits before the pager, leaving dangling file descriptors
and the aforementioned I/O error from Tokio.

Enter **tokio-pager**. This crate exports a `Pager` struct that allows the
use of a pager subprocess in a way that plays nicely with Tokio.

`Pager` pipes its output to program specified in the `$PAGER`environment
variable, except under two conditions:

1. If the value of `$PAGER` is `cat`, `/usr/bin/cat`, or anything that 
   ends in `/cat` (`/bin/cat`, etc.), then the output is not paged at all
   (`cat` is not launched).
2. If `stdout` is not a TTY, such as when output is being redirected to a
   file, the output is not paged.

`Pager` respects the value of the `$LESS` environment variable (with some
caveats—see `PagerEnv::pager_env()` for details).

## Pager support

Right now, **tokio-pager** only works with `less`, but support for other
pagers is planned in the future.

[pager]: https://crates.io/crates/pager
