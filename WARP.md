# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

# Project: Sculptor

Sculptor is an unofficial backend for the Minecraft mod [Figura](https://github.com/FiguraMC/Figura), written in Rust. It serves as a replacement for the official backend, handling user authentication, avatar hosting, and synchronization.

## Development

### Prerequisites
- Rust (2024 edition)
- `cargo`

### Common Commands

- **Run in development:**
  ```bash
  # Ensure Config.toml exists (copy from Config.example.toml)
  cargo run
  ```

- **Build for release:**
  ```bash
  cargo build --release
  ```

- **Run in release mode:**
  ```bash
  cargo run --release
  ```

- **Run tests:**
  ```bash
  cargo test
  ```

- **Docker Build:**
  The project uses a multi-stage Dockerfile with `cargo-chef` and `cargo-zigbuild`.
  ```bash
  docker build -t sculptor .
  ```

### Configuration
The application requires a `Config.toml` file in the working directory.
- Copy `Config.example.toml` to `Config.toml` to start.
- `Config.toml` controls listening address, metrics, auth providers, limits, and "advanced users" (admins/special users).

## Architecture

### Core Components

- **Entry Point:** `src/main.rs` initializes the environment, logging, configuration, and starts the Axum server.
- **State Management:** `src/state/state.rs` defines `AppState`, which holds the global state:
    - `UManager`: User manager (sessions, registration).
    - `session` / `subscribes`: `DashMap`s for WebSocket connections and message broadcasting.
    - `config`: `RwLock`-wrapped application configuration.
- **Authentication:** `src/auth/` handles user verification via external providers (Mojang, Ely.By).
    - `UManager` (`src/auth/auth.rs`) stores user sessions in memory (`DashMap`). **Note:** Standard user sessions are *not* persisted to disk and will be lost on restart.
- **API:** `src/api/` contains the HTTP and WebSocket route handlers.
    - `src/api/figura`: Implements the Figura mod protocol (WebSocket, avatar endpoints).
    - `src/api/sculptor`: Sculptor-specific endpoints.

### Data Persistence

- **User Sessions:** In-memory (transient).
- **Avatars:** Stored on the filesystem in `data/avatars` (configurable via `AVATARS` env or default).
- **Assets:** Stored in `data/assets`.
- **Configuration & Bans:**
    - `Config.toml`: Main config and "advanced users".
    - `banned-players.json`: Minecraft ban list (optional integration).
    - `src/utils/auxiliary.rs` contains logic to watch these files and hot-reload changes.

### Key Directories
- `src/api`: Web server endpoints.
- `src/auth`: Authentication logic and user management.
- `src/state`: Global application state structs.
- `src/utils`: Helper functions, including file watchers and update checkers.
