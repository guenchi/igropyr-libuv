# igropyr-libuv

`(igropyr libuv)` — a minimal [libuv][libuv] FFI layer for [Chez
Scheme][chez], the I/O foundation of [Igropyr][igropyr]. It talks to
libuv directly through Chez's FFI, with **no C shim**: TCP listen /
connect / read / write, async DNS, async and streaming file reads, and a
millisecond clock, all driven by one `uv-poll!` pump.

The library knows nothing about green processes; it delivers events to
the upper layer through a single hook installed with `uv-set-deliver!`.
That keeps the scheduler and the I/O layer independent — Igropyr's actor
runtime is one consumer, but the layer stands on its own.

## The callback invariant

Code running inside a libuv callback (anything reached from `uv-poll!`)
must **never yield, never block in `receive`, and never raise**.
Callbacks only copy data, mutate registries, and deliver messages.
Yielding would unwind a continuation through a C stack frame and corrupt
the process. Honor this and the rest is ordinary Scheme.

## What it exports

```
uv-init!  uv-poll!  now-ms  uv-set-deliver!
tcp-listen!  tcp-stop-listen!  tcp-connect!  dns-resolve!
tcp-read-start!  tcp-write!  tcp-writev!  tcp-write-foreign!  tcp-close!
file-read-async!
file-stream-open!  file-stream-read!  file-stream-close!
file-stream-own!   file-stream-raw!   file-stream-chunk-ptr
conn?  conn-handle  conn-owner  conn-set-owner!  conn-state  conn-count
uv-strerror
```

## Dependencies

This is the I/O module extracted from Igropyr; it is **not standalone**.
It imports two small sibling libraries from Igropyr:

- `(igropyr platform)` — supported-host detection and shared-library
  loading (the `libuv` / `libc` candidate lists per OS, struct-offset
  constants).
- `(igropyr util)` — a handful of dependency-free string helpers.

Both live in the [Igropyr][igropyr] source tree. To use `(igropyr
libuv)`, put `libuv.sc` alongside `platform.sc` and `util.sc` in an
`igropyr/` directory on your library path, and have **libuv** installed
on the host (Homebrew `libuv` on macOS, the `libuv` package on
Linux/FreeBSD).

```sh
# with igropyr/{libuv,platform,util}.sc reachable from the library path
CHEZSCHEMELIBDIRS=. CHEZSCHEMELIBEXTS=.sc scheme --script your-program.ss
```

## License

MIT. See [LICENSE](LICENSE).

[chez]: https://www.scheme.com
[libuv]: https://libuv.org
[igropyr]: https://github.com/guenchi/Igropyr
