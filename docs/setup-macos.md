# Running RuView from source on macOS (no hardware)

A short path from a fresh clone to the full dashboard on a Mac, using the
sensing server's simulated data source. Tested on an Apple Silicon Mac
(arm64) with Xcode command line tools installed. Everything shown in this mode
is `SYNTHETIC`: the server generates the data and no WiFi signal is read.

## 1. Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --profile minimal
source "$HOME/.cargo/env"
```

`v2/rust-toolchain.toml` pins the toolchain, so rustup downloads the right
version the first time you run `cargo` inside `v2/`.

## 2. Fetch the submodules

The Rust workspace depends on crates vendored as git submodules. A plain
`git clone` leaves them empty and the build fails with
`failed to read .../vendor/rufield/.../Cargo.toml`.

```bash
git submodule update --init --depth 1
```

(Or clone with `git clone --recurse-submodules` in the first place.)

## 3. Build the sensing server

```bash
cd v2
cargo build --release -p wifi-densepose-sensing-server --bin sensing-server
```

The first build takes a few minutes.

## 4. Run it in simulated mode

From `v2/`:

```bash
./target/release/sensing-server --source simulated \
  --http-port 3000 --ws-port 3001 --data-dir "$HOME/.ruview-data"
```

The server binds to `127.0.0.1` only. `--data-dir` keeps runtime state out of
the checkout (the default `data/` would land inside `v2/`). Without
`RUVIEW_API_TOKEN` set, `/api/v1/*` is unauthenticated, which is fine for a
loopback-only demo.

## 5. Open the dashboard

- Dashboard: <http://localhost:3000/ui/index.html>
- 3D observatory: <http://localhost:3000/ui/observatory.html>
- Health check: `curl http://localhost:3000/health` should report
  `"source":"simulated","status":"ok"`.

The dashboard shows a "SYNTHETIC DATA" banner while in this mode. For real
sensing, flash an ESP32-S3 node (see `firmware/esp32-csi-node/README.md`) and
run with `--source esp32`.
