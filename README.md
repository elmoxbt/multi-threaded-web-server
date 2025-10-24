# Multi-Threaded Web Server

A simple HTTP web server written in Rust that demonstrates concurrent request handling using a custom thread pool implementation.

## Features

- **Custom Thread Pool**: Implements a thread pool from scratch using Rust's concurrency primitives
- **Graceful Shutdown**: Properly terminates worker threads when the server stops
- **Connection Limiting**: Accepts a limited number of connections (configurable)
- **Static File Serving**: Serves HTML files for root path and 404 pages
- **Zero Dependencies**: Built using only Rust's standard library

## How It Works

The server uses a thread pool with 4 worker threads to handle incoming HTTP requests concurrently. Each worker thread waits for jobs from a shared channel and processes them independently. This design prevents spawning a new thread for every connection, which improves performance and resource management.

### Architecture

- **ThreadPool**: Manages a fixed number of worker threads
- **Worker**: Represents individual threads that execute jobs
- **Message Enum**: Coordinates communication between the pool and workers
- **Channel**: Distributes incoming connections across workers

## Project Structure

```
multi-threaded-web-server/
├── Cargo.toml
├── src/
│   ├── bin/
│   │   └── main.rs      # Server setup and request handling
│   └── lib.rs           # ThreadPool implementation
├── index.html           # Homepage
├── 404.html             # Not found page
└── README.md
```

## Installation

```bash
cd multi-threaded-web-server
cargo build --release
```

## Usage

Start the server:

```bash
cargo run --release
```

The server will bind to `127.0.0.1:7878` and accept 2 connections before gracefully shutting down.

### Testing

Open your browser and navigate to:
- `http://127.0.0.1:7878/` → Serves `index.html`
- `http://127.0.0.1:7878/anything-else` → Serves `404.html`

Or use `curl`:

```bash
curl http://127.0.0.1:7878/
```

## Configuration

To modify the server behavior, edit `src/bin/main.rs`:

- **Port**: Change `"127.0.0.1:7878"` to your desired address
- **Thread Pool Size**: Change `ThreadPool::new(4)` to use more/fewer threads
- **Connection Limit**: Change `.take(2)` to accept more connections (or remove to run indefinitely)

## Implementation Details

### Thread Pool

The `ThreadPool` creates a fixed number of worker threads at startup. Jobs are sent through an MPSC channel wrapped in `Arc<Mutex<>>` to allow multiple workers to safely receive tasks.

### Graceful Shutdown

When the `ThreadPool` is dropped, it:
1. Sends `Terminate` messages to all workers
2. Waits for each worker thread to finish its current job
3. Joins all threads before exiting

### Request Handling

The server performs basic HTTP parsing by checking if the request starts with `GET / HTTP/1.1`. Responses include appropriate status lines and HTML content.

## Limitations

- Only handles GET requests
- No proper HTTP header parsing
- Serves only two static files
- No MIME type handling
- No request routing beyond root path

## Future Improvements

- Add proper HTTP request/response parsing
- Implement routing system
- Support multiple HTTP methods
- Add middleware support
- Enable dynamic content generation
- Implement logging

## License

MIT License – free to use and modify.
