# Build & Package Guide

This document provides detailed instructions on how to build and package Antigravity Tools from source.

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Development Setup](#development-setup)
- [Local Development](#local-development)
- [Production Build](#production-build)
- [Platform-Specific Notes](#platform-specific-notes)
- [Troubleshooting](#troubleshooting)

## 🔧 Prerequisites

### Required Software

1. **Node.js** (v20 or higher recommended)
   - Download: https://nodejs.org/
   - Verify: `node --version`

2. **Rust** (stable)
   - Install:
     ```bash
     curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
     ```
   - Verify: `rustc --version`

3. **npm** or **pnpm** (package manager)
   - npm comes with Node.js
   - Or install pnpm: `npm install -g pnpm`

### Platform-Specific Dependencies

#### macOS
```bash
# Xcode Command Line Tools
xcode-select --install
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install -y \
  libwebkit2gtk-4.1-dev \
  build-essential \
  curl \
  wget \
  file \
  libssl-dev \
  libgtk-3-dev \
  libayatana-appindicator3-dev \
  librsvg2-dev \
  patchelf \
  pkg-config \
  libsoup-3.0-dev \
  javascriptcoregtk-4.1 \
  libjavascriptcoregtk-4.1-dev \
  libnm-dev \
  xdg-utils
```

#### Linux (Arch)
```bash
sudo pacman -S webkit2gtk base-devel curl wget file openssl gtk3 libayatana-appindicator librsvg
```

#### Windows
- Install [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
- Install [WebView2](https://developer.microsoft.com/en-us/microsoft-edge/webview2/)

## 🚀 Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/lbjlaq/Antigravity-Manager.git
cd Antigravity-Manager

# 2. Install dependencies
npm install

# 3. Run in development mode
npm run tauri dev

# 4. Build for production
npm run tauri build
```

## 🛠️ Development Setup

### 1. Clone the Project

```bash
git clone https://github.com/lbjlaq/Antigravity-Manager.git
cd Antigravity-Manager
```

### 2. Install Frontend Dependencies

```bash
# Using npm
npm install

# Or using pnpm
pnpm install
```

### 3. Verify Rust Environment

```bash
cd src-tauri
cargo check
cd ..
```

## 💻 Local Development

### Start Development Server

```bash
# Standard development mode
npm run tauri dev

# Development mode with debug logging
npm run tauri:debug
```

This will:
1. Start Vite dev server (port 1420)
2. Compile Rust backend
3. Launch Tauri app window
4. Enable hot-reload (frontend changes auto-refresh)

### Frontend-Only Development

If you only need to develop the UI:

```bash
npm run dev
```

Then visit `http://localhost:1420` in your browser.

### Run Tests

```bash
# Rust backend tests
cd src-tauri
cargo test --all --all-features

# Return to project root
cd ..
```

## 📦 Production Build

### Build for All Platforms

```bash
npm run tauri build
```

### Build for Specific Platform

#### macOS

```bash
# Apple Silicon (M1/M2/M3)
npm run tauri build -- --target aarch64-apple-darwin

# Intel Mac
npm run tauri build -- --target x86_64-apple-darwin

# Universal Binary (supports both Intel and Apple Silicon)
npm run tauri build -- --target universal-apple-darwin
```

#### Linux

```bash
# Default build (generates .deb and AppImage)
npm run tauri build

# Build AppImage only
npm run tauri build -- --bundles appimage

# Build .deb only
npm run tauri build -- --bundles deb
```

#### Windows

```bash
# Default build (generates .msi and portable)
npm run tauri build

# Build MSI installer only
npm run tauri build -- --bundles msi

# Build NSIS installer only
npm run tauri build -- --bundles nsis
```

### Build Artifacts Location

After building, artifacts are located at:

```
src-tauri/target/release/bundle/
├── macos/          # macOS .app and .dmg
├── deb/            # Linux .deb packages
├── appimage/       # Linux AppImage
├── msi/            # Windows .msi installer
└── nsis/           # Windows NSIS installer
```

## 🔍 Platform-Specific Notes

### macOS

#### Code Signing

For distribution to other users, code signing is recommended:

1. Prepare Apple Developer certificate
2. Set environment variables:
   ```bash
   export APPLE_CERTIFICATE="Certificate Name"
   export APPLE_CERTIFICATE_PASSWORD="Certificate Password"
   export APPLE_ID="Your Apple ID"
   export APPLE_PASSWORD="App-Specific Password"
   ```
3. Build:
   ```bash
   npm run tauri build
   ```

#### Create DMG

```bash
# Use provided script
./scripts/package_dmg.sh
```

#### Fix "App is Damaged" Issue

Locally built apps may be blocked by macOS Gatekeeper. Fix with:

```bash
sudo xattr -rd com.apple.quarantine "/Applications/Antigravity Tools.app"
```

### Linux

#### AppImage Permissions

Built AppImage needs executable permissions:

```bash
chmod +x src-tauri/target/release/bundle/appimage/*.AppImage
```

#### System Tray Support

Ensure system tray libraries are installed:

```bash
# Ubuntu/Debian
sudo apt-get install libayatana-appindicator3-dev

# Arch
sudo pacman -S libayatana-appindicator
```

### Windows

#### Administrator Privileges

Some features may require admin privileges. Configure in `src-tauri/tauri.conf.json`:

```json
{
  "bundle": {
    "windows": {
      "wix": {
        "requireAdmin": false
      }
    }
  }
}
```

## 🐳 Docker Build

### Build Docker Image

```bash
cd docker
docker build -t antigravity-tools:local .
```

### Run Container

```bash
docker run -d \
  --name antigravity \
  -p 8045:8045 \
  -e API_KEY=sk-your-api-key \
  -v ~/.antigravity_tools:/root/.antigravity_tools \
  antigravity-tools:local
```

## ❓ Troubleshooting

### Q: Build fails with "webkit2gtk not found"

**A:** Install WebKit2GTK development package:

```bash
# Ubuntu/Debian
sudo apt-get install libwebkit2gtk-4.1-dev

# Arch
sudo pacman -S webkit2gtk
```

### Q: macOS shows "xcrun: error: invalid active developer path"

**A:** Install Xcode Command Line Tools:

```bash
xcode-select --install
```

### Q: Windows build fails with MSVC errors

**A:** Install Microsoft C++ Build Tools with the following components:
- MSVC v142 or later
- Windows 10 SDK

### Q: Rust compilation is slow or stuck

**A:** Try these solutions:

1. Clean build cache:
   ```bash
   cd src-tauri
   cargo clean
   cd ..
   ```

2. Use mirror registry (if in China):
   Create `~/.cargo/config.toml`:
   ```toml
   [source.crates-io]
   replace-with = 'ustc'
   
   [source.ustc]
   registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
   ```

3. Increase parallel build jobs:
   ```bash
   export CARGO_BUILD_JOBS=4
   ```

### Q: Frontend build fails with TypeScript errors

**A:** Ensure correct TypeScript version:

```bash
npm install typescript@~5.8.3 --save-dev
```

### Q: How to reduce build artifact size?

**A:** Enable more optimizations in `src-tauri/Cargo.toml`:

```toml
[profile.release]
opt-level = 'z'     # Optimize for size
lto = true          # Link-time optimization
codegen-units = 1   # Better optimization
strip = true        # Remove symbols
```

### Q: App starts slowly in development mode

**A:** This is normal due to Rust compilation time. Subsequent starts will be faster. Try:

1. Use incremental compilation (enabled by default)
2. Use faster linker:
   ```bash
   # macOS/Linux
   cargo install mold
   
   # Configure in Cargo.toml
   [target.x86_64-unknown-linux-gnu]
   linker = "clang"
   rustflags = ["-C", "link-arg=-fuse-ld=mold"]
   ```

### Q: How to see detailed build logs?

**A:** Enable verbose logging:

```bash
# Tauri build logs
npm run tauri build -- --verbose

# Rust compilation logs
RUST_LOG=debug npm run tauri dev
```

## 📚 Additional Resources

- [Tauri Official Documentation](https://tauri.app/v2/guides/)
- [Rust Official Tutorial](https://www.rust-lang.org/learn)
- [Project Issue Tracker](https://github.com/lbjlaq/Antigravity-Manager/issues)
- [Contributing Guide](../CONTRIBUTING.md) (if available)

## 🤝 Getting Help

If you encounter issues:

1. Check the Troubleshooting section above
2. Search [GitHub Issues](https://github.com/lbjlaq/Antigravity-Manager/issues)
3. Submit a new Issue with detailed information:
   - Operating system and version
   - Node.js and Rust versions
   - Complete error logs
   - Steps to reproduce

---

**Happy Building!** 🚀
