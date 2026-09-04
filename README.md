# igropyr-libuv

`(igropyr libuv)` — a minimal [libuv][libuv] FFI layer for [Chez
Scheme][chez], the I/O foundation of [Igropyr][igropyr]. It talks to
libuv directly through Chez's FFI, with **no C shim**: the event loop,
the process-wide I/O buffers, a millisecond clock, and the raw bindings
for TCP, timers, DNS and the filesystem — all driven by one `uv-poll!`
pump.

## What changed in 1.5.2

This library used to carry the connection layer as well — `tcp-listen!`,
`tcp-write!`, `dns-resolve!`, `file-read-async!`, the `conn` record and
their kin. **Those moved to `(igropyr tcp)` in the [Igropyr][igropyr]
tree**, which imports this one. What remains here is the binding layer:
the loop, the buffers, and the FFI.

If you were using this library for TCP or asynchronous file I/O, you now
need `tcp.sc` from the main tree as well; this repository alone no
longer provides those entry points.

## The callback invariant

Code running inside a libuv callback (anything reached from `uv-poll!`)
must **never yield, never block in `receive`, and never raise**.
Callbacks only copy data, mutate registries, and deliver messages.
Yielding would unwind a continuation through a C stack frame and corrupt
the process. Honor this and the rest is ordinary Scheme.

## What it exports

98 names, in four groups:

```
the loop      uv-init!  uv-poll!  uv-wakeup!  uv-in-callback?
              uv-loop-handle  uv-live-handle-count
clocks        now-ms  now-ns
buffers       uv-sockaddr-lease  uv-scratch-lease  uv-peername-lease
              uv-read-buf-base  uv-read-buf-size  uv-write-scratch-size
raw bindings  uv-tcp-init  uv-tcp-bind  uv-listen  uv-accept
              uv-read-start  uv-write  uv-try-write  uv-close
              uv-timer-init  uv-timer-start  uv-timer-stop
              uv-getaddrinfo  uv-fs-open  uv-fs-read  uv-fs-stat  ...
              plus the size constants, the O_* and S_IF* flags,
              memcpy-*, and a few libc entry points
```

The three `*-lease` procedures hand out the process-wide buffers with
interrupts disabled for the duration of a thunk; that is how a caller
uses them safely rather than by holding the address.

## Dependencies

This is the I/O module extracted from Igropyr; it is **not standalone**.
The chain is `libuv → platform → util`:

- [`(igropyr platform)`][platform] — supported-host detection and
  shared-library loading (the `libuv` / `libc` candidate lists per OS,
  struct-offset constants).
- [`(igropyr util)`][util] — a handful of dependency-free string
  helpers (a dependency of `platform`).

Both also live in the [Igropyr][igropyr] source tree. To use `(igropyr
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
[platform]: https://github.com/guenchi/igropyr-platform
[util]: https://github.com/guenchi/igropyr-util
