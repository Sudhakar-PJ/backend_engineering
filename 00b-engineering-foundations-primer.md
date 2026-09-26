# Engineering Foundations Primer

> **Read this once, at the very start of the curriculum, before T1.**
> Not a tier. Not tracked. Just prerequisite mental models so that later topics — event loop, memory leaks, file descriptors, signals, syscalls — have a foundation.
> **Estimated time**: 2–3 hours. Read it straight through. No exercises. Confirm you understand, move on.

---

## Why This Exists

Every tier in this curriculum assumes you have an intuitive model of what a computer actually does when it runs your code. Without that model:

- "Non-blocking I/O" is a slogan, not a mechanism.
- "Memory leak" sounds like a bug in the language instead of retained references.
- "`EMFILE: too many open files`" is a mystery instead of a file descriptor limit.
- "SIGTERM handler" is copy-pasted boilerplate instead of a specific OS signal.

None of these require deep systems knowledge. But they do require **a small, correct mental model** that most tutorials skip.

This primer gives you that model. It is deliberately short, concrete, and Node-focused.

---

## 1. Process, Program, Thread

**Program**: a file on disk. Nothing is running. It's a binary, a script, an executable. `node` is a program. Your `app.js` is a program.

**Process**: a _running instance_ of a program. The OS has allocated it memory, a process ID (PID), a slice of CPU time, file descriptors, and a security context. If you run `node app.js` twice, you have two processes.

**Thread**: a unit of execution _inside_ a process. A process has at least one thread (the "main thread"). Threads within a process share memory and file descriptors. A process can have many threads, each running code in parallel on a different CPU core.

**Why this matters for Node**:

- Node runs your JavaScript on **one main thread** — the event loop thread.
- It has a small **libuv thread pool** (default 4 threads) for things like `fs`, `crypto`, `dns`, and `zlib` that the OS doesn't natively do asynchronously.
- Everything else — your callbacks, promise resolutions, HTTP request handling — runs on the main thread.

**Consequences**:

- Blocking the main thread blocks _everything_ — every other request, every timer, every callback.
- CPU-heavy work needs `worker_threads` (separate threads) or `cluster` (separate processes).
- Node's concurrency is about **overlapping I/O**, not **parallel CPU**.

---

## 2. Memory Model — Stack vs Heap

When a program runs, the OS gives it memory in two main regions:

**Stack**:

- Small (typically 1–8 MB per thread).
- Fast (LIFO allocation — just move a pointer).
- Stores local variables, function arguments, return addresses.
- **Automatically reclaimed** when a function returns.

**Heap**:

- Large (gigabytes).
- Slower (general-purpose allocator).
- Stores objects, arrays, closures, anything whose lifetime is not "until this function returns."
- **Reclaimed by a garbage collector** when no references point to it.

**Why this matters for Node**:

- V8 heap is where your JS objects live. Its size is controlled by `--max-old-space-size` (default ~2GB on 64-bit systems, sometimes 1.5GB).
- A memory leak in Node means: an object on the heap is still reachable from some root reference (a global, a closure, an event listener, a timer), so V8 can't collect it, so the heap grows until `OOMKilled`.
- Stack overflow (`RangeError: Maximum call stack size exceeded`) is a _different_ problem than heap exhaustion. Deep recursion blows the stack; retained objects blow the heap.

**Consequences**:

- `Buffer` and `ArrayBuffer` allocate memory **outside** the V8 heap (in C++ land). `process.memoryUsage().heapUsed` won't show them, but `rss` will.
- Long-lived closures retain their captured variables — if the closure never dies, neither do the captures.
- Unbounded caches (a `Map` that only ever grows) are a heap leak, not a "memory management issue."

---

## 3. CPU, Cores, and Syscalls

**CPU core**: a physical or logical unit that executes instructions. Modern servers have 4–128 cores. A single-threaded process can use **one** core at a time. Multiple processes/threads can use multiple cores.

**Cache hierarchy** (L1/L2/L3): small, very fast memory between the CPU and RAM. When your code reuses the same data (hot loop, hot key), it's fast. When it jumps around (random memory access), it's slow. This is why "data locality" matters in high-performance systems — but for a beginner, you mostly don't need to think about it. **Awareness only.**

**Syscall**: a request from your program to the kernel to do something. `read()`, `write()`, `open()`, `close()`, `fork()`, `socket()`, `epoll_wait()` — these are all syscalls.

- Every I/O operation is (at some layer) a syscall.
- Syscalls have overhead (~100ns–1µs) because they cross the user/kernel boundary.
- **Minimizing syscalls** is a real optimization at scale (batching, buffering, `sendfile` for zero-copy).

**Why this matters for Node**:

- Node's event loop uses `epoll` (Linux), `kqueue` (macOS), or `IOCP` (Windows) to wait on many sockets/file descriptors efficiently. This is what "non-blocking I/O" actually means — the kernel tells you when a socket is ready.
- When you call `fs.readFile`, Node hands the syscall off to the libuv thread pool so the main thread doesn't block.
- `process.hrtime.bigint()` uses `clock_gettime` under the hood — a syscall.

**Consequences**:

- Reading 1M small files is slow because of syscall overhead, not Node.
- High-throughput services batch their I/O to reduce syscalls.
- `strace` (a debugging tool) shows you exactly which syscalls your program makes.

---

## 4. Signals and Exit Codes

**Signal**: an asynchronous notification from the OS to a process. Used for control (stop, terminate, reload, interrupt).

**Common signals**:

- `SIGTERM` (15) — polite termination request. `kill <pid>` sends this by default.
- `SIGINT` (2) — keyboard interrupt. `Ctrl+C` sends this.
- `SIGKILL` (9) — forced, unblockable termination. Cannot be caught.
- `SIGHUP` (1) — historically "hang up," now often "reload config."
- `SIGUSR1`, `SIGUSR2` — user-defined, often used for "dump something."

**Exit code**: a 0–255 integer returned when a process ends. Convention:

- `0` = success
- non-zero = failure (`1` = generic, `2` = usage error, `126`/`127` = exec problems, etc.)

**Why this matters for Node**:

- Kubernetes/Docker sends `SIGTERM` before `SIGKILL` (with a grace period, default 30s). Your service must catch `SIGTERM`, drain in-flight requests, close DB pools, then exit with code 0.
- `Ctrl+C` sends `SIGINT`. Your CLI tools should catch it and clean up.
- `process.exit(0)` returns exit code 0. `process.exit(1)` returns 1.
- Unhandled exceptions in Node exit with code 1 by default.

**Consequences**:

- **Never call `process.exit()` before flushing stdout** — you can truncate logs.
- A graceful shutdown handler is not decoration — it's the difference between a clean deploy and dropping requests.
- `SIGKILL` cannot be caught — this is why K8s gives you a grace period.

---

## 5. stdio and File Descriptors

**File descriptor (FD)**: an integer handle the kernel uses to refer to an open file, socket, pipe, or device.

- `0` = stdin
- `1` = stdout
- `2` = stderr
- Everything else is allocated dynamically (`3`, `4`, `5`, ...).

**Limit**: each process has a max number of open FDs, controlled by `ulimit -n`. Default on many systems is 1024 or 4096. In containers, often 1024.

**Why this matters for Node**:

- Every open socket, file, HTTP connection, DB connection uses an FD.
- When you hit the limit, you get `EMFILE: too many open files`.
- **This is the most common production outage cause at scale.** A leak of unclosed connections or file handles kills the process once it hits `ulimit`.
- The fix: set `ulimit -n` higher (e.g., 65535), and _also_ fix leaks (unclosed sockets, unclosed `fs.createReadStream`).

**Consequences**:

- `lsof -p <pid>` shows you every FD a process has open.
- A simple `setInterval` that opens a socket and never closes it will exhaust FDs within hours at high frequency.
- Node's default HTTP `Agent` reuses sockets (keep-alive) — a crucial detail for FD consumption.

---

## 6. Environment and Configuration

**Environment variable**: a key-value pair the OS passes to a process at startup. Set at shell level (`export FOO=bar`), by a process manager (Docker, systemd, PM2), or explicitly (`FOO=bar node app.js`).

**PATH**: a special env var — a list of directories the shell searches to find executables by name. `node` works because it's in your PATH.

**Why this matters for Node**:

- `process.env.FOO` reads environment variables. `process.env` is populated at process start — you can mutate it at runtime, but child processes inherit the _current_ values.
- Secrets should be passed via env vars (or secret managers) — never hardcoded, never committed.
- `.env` files are a _dev convenience_, not a production mechanism. In production, env vars come from the orchestrator.

**Consequences**:

- Fail-fast: validate required env vars at startup with Zod. A missing `DATABASE_URL` should crash the service immediately, not on the first DB query.
- `.env.example` documents required vars without committing secrets.
- Twelve-Factor App: config lives in the environment, not in code.

---

## 7. Binary and Executable (Awareness Only)

**Binary**: a file containing CPU instructions (machine code). ELF on Linux, Mach-O on macOS, PE on Windows.

**Dynamic linking**: most binaries don't contain every library they need. They _link_ to shared libraries (`.so` on Linux, `.dylib` on macOS) at runtime. `ldd node` shows what Node links against (libc, libpthread, libdl, etc.).

**Shebang**: `#!/usr/bin/env node` at the top of a script tells the OS to run it with `node`.

**Why this matters for Node (barely)**:

- When you `npm install` a native module (like `bcrypt` or `sharp`), it compiles or downloads a `.node` binary matching your OS/arch. This is why `node_modules` isn't portable across platforms.
- When you see `Error: Cannot find module '...node'`, it's usually a native module missing or built for the wrong platform.
- Docker images for Node must match the target platform (Alpine uses musl, not glibc — some native modules break).

**You don't need to know more than this.** Compilers, linkers, and ABI details are out of scope for backend engineering.

---

## What This Primer Enables

With this mental model, later topics stop being mysterious:

| Later topic            | Now makes sense because...                                     |
| ---------------------- | -------------------------------------------------------------- |
| T1.2 Event loop        | You know Node runs on one thread with libuv doing I/O          |
| T1.4 Memory leaks      | You know heap vs stack, and reachability-based GC              |
| T1.7 Streams           | You know syscalls have overhead, so buffering/batching matters |
| T1.7 File descriptors  | You know `EMFILE` is a limit, not a bug                        |
| T2.8 Graceful shutdown | You know `SIGTERM` is a signal, and exit codes matter          |
| T5.9 SIGTERM handling  | You know the K8s grace period before `SIGKILL`                 |
| T6.7 Debugging         | You know `strace`, `lsof`, `perf` inspect syscalls/FDs/CPU     |
| All tiers              | You know what "blocking" and "non-blocking" actually mean      |

---

## Confirm and Move On

You've finished the primer when you can answer these in one sentence each (no need to write them down — just confirm to yourself):

1. What is the difference between a process and a thread?
2. What is the difference between stack memory and heap memory?
3. What is a syscall, and why does it matter for Node?
4. What is `SIGTERM`, and why does your service need to handle it?
5. What is a file descriptor, and what happens when you run out?
6. Why should secrets be passed as environment variables, not hardcoded?

If any answer is fuzzy, re-read that section. Otherwise:

> **Move to `01-tier-1-language-runtime.md` and begin T1.**
