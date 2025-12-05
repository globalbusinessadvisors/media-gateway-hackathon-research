# CLI Build and Deployment Guide
## TV Discovery System - Rust CLI

**Version:** 1.0
**Date:** 2025-12-05

---

## Table of Contents

1. [Build Configuration](#build-configuration)
2. [Development Setup](#development-setup)
3. [Build Scripts](#build-scripts)
4. [CI/CD Pipeline](#cicd-pipeline)
5. [Release Process](#release-process)
6. [Distribution](#distribution)
7. [Installation Methods](#installation-methods)

---

## Build Configuration

### build.rs - Protocol Buffer Compilation

```rust
// build.rs

use std::env;
use std::path::PathBuf;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let out_dir = PathBuf::from(env::var("OUT_DIR")?);

    // Compile protocol buffers
    tonic_build::configure()
        .build_server(false)  // CLI only needs client
        .build_client(true)
        .out_dir(&out_dir)
        .compile(
            &[
                "proto/discovery.proto",
                "proto/auth.proto",
                "proto/device.proto",
                "proto/account.proto",
                "proto/preferences.proto",
            ],
            &["proto"],
        )?;

    // Recompile if proto files change
    println!("cargo:rerun-if-changed=proto/");

    Ok(())
}
```

### Cargo.toml - Complete Configuration

```toml
[package]
name = "tv-discover-cli"
version = "0.1.0"
edition = "2021"
authors = ["TV Discover Team <team@tv-discover.com>"]
description = "Unified CLI for global TV content discovery across streaming platforms"
readme = "README.md"
homepage = "https://tv-discover.com"
repository = "https://github.com/tv-discover/cli-core"
license = "MIT OR Apache-2.0"
keywords = ["cli", "tv", "streaming", "discovery", "entertainment"]
categories = ["command-line-utilities", "multimedia"]

[dependencies]
# CLI Framework
clap = { version = "4.4", features = ["derive", "env", "wrap_help", "cargo"] }
clap_complete = "4.4"

# TUI Framework
ratatui = "0.25"
crossterm = { version = "0.27", features = ["event-stream", "serde"] }

# gRPC and Networking
tonic = { version = "0.10", features = ["tls", "tls-roots", "tls-webpki-roots"] }
prost = "0.12"
prost-types = "0.12"
tower = { version = "0.4", features = ["retry", "balance", "timeout", "limit"] }
hyper = { version = "0.14", features = ["client", "http2"] }
hyper-tls = "0.5"

# Async Runtime
tokio = { version = "1.35", features = ["full"] }
tokio-stream = { version = "0.1", features = ["sync"] }
futures = "0.3"
futures-util = "0.3"
async-stream = "0.3"

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
toml = "0.8"
serde_yaml = "0.9"

# Error Handling
thiserror = "1.0"
anyhow = "1.0"
eyre = "0.6"

# Authentication and Security
oauth2 = "4.4"
keyring = { version = "2.2", features = ["apple-native", "windows-native", "linux-native"] }
ring = "0.17"
base64 = "0.21"

# Configuration
dirs = "5.0"
config = { version = "0.13", features = ["toml", "yaml"] }
shellexpand = "3.1"

# Logging and Tracing
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json", "ansi"] }
tracing-appender = "0.2"

# Terminal and UI Utilities
console = "0.15"
indicatif = { version = "0.17", features = ["tokio"] }
colored = "2.1"
textwrap = "0.16"

# QR Code
qrcode = { version = "0.13", features = ["svg"] }

# HTTP Client (for OAuth)
reqwest = { version = "0.11", features = ["json", "rustls-tls"], default-features = false }

# Utilities
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1.6", features = ["v4", "serde"] }
url = { version = "2.5", features = ["serde"] }
lazy_static = "1.4"
once_cell = "1.19"

# Platform-specific
[target.'cfg(windows)'.dependencies]
winapi = { version = "0.3", features = ["wincon", "winuser"] }

[target.'cfg(unix)'.dependencies]
libc = "0.2"

[build-dependencies]
tonic-build = { version = "0.10", features = ["prost"] }
prost-build = "0.12"

[dev-dependencies]
# Testing
mockall = "0.12"
wiremock = "0.5"
tempfile = "3.8"
assert_cmd = "2.0"
predicates = "3.0"
proptest = "1.4"
criterion = { version = "0.5", features = ["html_reports"] }

# Test utilities
tokio-test = "0.4"
test-case = "3.3"

[profile.dev]
opt-level = 0
debug = true

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true
panic = "abort"

[profile.bench]
opt-level = 3
debug = false
lto = true

# Smaller binary size profile
[profile.release-small]
inherits = "release"
opt-level = "z"
lto = true
codegen-units = 1
strip = true

[[bin]]
name = "tvd"
path = "src/main.rs"

[[bench]]
name = "search_benchmark"
harness = false

[features]
default = ["native-tls"]
native-tls = ["tonic/tls-roots"]
rustls = ["tonic/tls-webpki-roots"]
```

### .cargo/config.toml - Platform-Specific Build Configuration

```toml
[build]
rustflags = ["-C", "target-cpu=native"]

[target.x86_64-unknown-linux-gnu]
linker = "clang"
rustflags = [
    "-C", "link-arg=-fuse-ld=lld",
    "-C", "link-arg=-Wl,--compress-debug-sections=zlib",
]

[target.x86_64-unknown-linux-musl]
linker = "rust-lld"
rustflags = ["-C", "target-feature=+crt-static"]

[target.x86_64-apple-darwin]
rustflags = [
    "-C", "link-arg=-mmacosx-version-min=10.15",
    "-C", "link-arg=-Wl,-dead_strip",
]

[target.aarch64-apple-darwin]
rustflags = [
    "-C", "link-arg=-mmacosx-version-min=11.0",
    "-C", "link-arg=-Wl,-dead_strip",
]

[target.x86_64-pc-windows-msvc]
rustflags = [
    "-C", "link-arg=/SUBSYSTEM:CONSOLE",
    "-C", "link-arg=/OPT:REF",
    "-C", "link-arg=/OPT:ICF",
]

[alias]
# Convenience aliases
xtask = "run --package xtask --"
build-release = "build --release --locked"
install-local = "install --path . --locked"
```

---

## Development Setup

### Prerequisites Installation Script

```bash
#!/bin/bash
# scripts/setup-dev.sh

set -e

echo "Setting up TV Discover CLI development environment..."

# Check Rust installation
if ! command -v rustc &> /dev/null; then
    echo "Installing Rust..."
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
    source $HOME/.cargo/env
fi

# Install required Rust components
echo "Installing Rust components..."
rustup component add rustfmt clippy
rustup target add x86_64-unknown-linux-gnu
rustup target add x86_64-apple-darwin
rustup target add x86_64-pc-windows-msvc

# Install additional tools
echo "Installing cargo tools..."
cargo install cargo-watch
cargo install cargo-expand
cargo install cargo-audit
cargo install cargo-outdated
cargo install cargo-llvm-cov

# Install protobuf compiler
if [[ "$OSTYPE" == "darwin"* ]]; then
    brew install protobuf
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
    sudo apt-get update
    sudo apt-get install -y protobuf-compiler
fi

# Install cross-compilation tools (optional)
cargo install cross

echo "Development environment setup complete!"
echo "Run 'cargo build' to build the project."
```

### Justfile - Task Runner

```justfile
# justfile - Task automation for TV Discover CLI

# Default recipe
default:
    @just --list

# Build the project
build:
    cargo build

# Build release version
build-release:
    cargo build --release --locked

# Run the CLI
run *ARGS:
    cargo run -- {{ARGS}}

# Run tests
test:
    cargo test --all-features

# Run tests with coverage
test-coverage:
    cargo llvm-cov --html --open

# Run benchmarks
bench:
    cargo bench

# Format code
fmt:
    cargo fmt --all

# Check formatting
fmt-check:
    cargo fmt --all -- --check

# Run clippy
clippy:
    cargo clippy --all-targets --all-features -- -D warnings

# Security audit
audit:
    cargo audit

# Check for outdated dependencies
outdated:
    cargo outdated

# Generate documentation
doc:
    cargo doc --no-deps --open

# Clean build artifacts
clean:
    cargo clean

# Watch for changes and rebuild
watch:
    cargo watch -x build

# Watch for changes and run tests
watch-test:
    cargo watch -x test

# Install locally
install:
    cargo install --path . --locked

# Cross-compile for all platforms
cross-compile: cross-linux cross-macos cross-windows

# Cross-compile for Linux
cross-linux:
    cross build --release --target x86_64-unknown-linux-gnu
    cross build --release --target x86_64-unknown-linux-musl

# Cross-compile for macOS
cross-macos:
    cross build --release --target x86_64-apple-darwin
    cross build --release --target aarch64-apple-darwin

# Cross-compile for Windows
cross-windows:
    cross build --release --target x86_64-pc-windows-msvc

# Generate shell completions
completions:
    mkdir -p completions
    cargo run -- completions bash > completions/tvd.bash
    cargo run -- completions zsh > completions/_tvd
    cargo run -- completions fish > completions/tvd.fish

# Run pre-commit checks
pre-commit: fmt-check clippy test

# Release preparation
prepare-release VERSION:
    @echo "Preparing release {{VERSION}}"
    @sed -i 's/^version = .*/version = "{{VERSION}}"/' Cargo.toml
    @cargo build --release
    @git add Cargo.toml Cargo.lock
    @git commit -m "Bump version to {{VERSION}}"
    @git tag -a "v{{VERSION}}" -m "Release v{{VERSION}}"
    @echo "Release v{{VERSION}} prepared. Push with: git push && git push --tags"
```

### Makefile - Alternative Build System

```makefile
# Makefile for TV Discover CLI

.PHONY: all build test clean install help

# Variables
CARGO := cargo
BINARY_NAME := tvd
TARGET_DIR := target
RELEASE_DIR := $(TARGET_DIR)/release
VERSION := $(shell grep '^version' Cargo.toml | head -1 | cut -d'"' -f2)

# Colors for output
GREEN := \033[0;32m
YELLOW := \033[0;33m
RED := \033[0;31m
NC := \033[0m # No Color

all: build ## Build the project

help: ## Show this help message
	@echo 'Usage: make [target]'
	@echo ''
	@echo 'Available targets:'
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  $(GREEN)%-15s$(NC) %s\n", $$1, $$2}'

build: ## Build the project
	@echo "$(GREEN)Building $(BINARY_NAME)...$(NC)"
	$(CARGO) build

release: ## Build release version
	@echo "$(GREEN)Building release version...$(NC)"
	$(CARGO) build --release --locked

test: ## Run tests
	@echo "$(GREEN)Running tests...$(NC)"
	$(CARGO) test --all-features

test-verbose: ## Run tests with verbose output
	$(CARGO) test --all-features -- --nocapture

bench: ## Run benchmarks
	@echo "$(GREEN)Running benchmarks...$(NC)"
	$(CARGO) bench

fmt: ## Format code
	@echo "$(GREEN)Formatting code...$(NC)"
	$(CARGO) fmt --all

clippy: ## Run clippy
	@echo "$(GREEN)Running clippy...$(NC)"
	$(CARGO) clippy --all-targets --all-features -- -D warnings

check: fmt clippy test ## Run all checks

clean: ## Clean build artifacts
	@echo "$(YELLOW)Cleaning build artifacts...$(NC)"
	$(CARGO) clean

install: release ## Install the binary
	@echo "$(GREEN)Installing $(BINARY_NAME)...$(NC)"
	$(CARGO) install --path . --locked

dist: release ## Create distribution packages
	@echo "$(GREEN)Creating distribution packages...$(NC)"
	@mkdir -p dist
	@tar -czf dist/$(BINARY_NAME)-$(VERSION)-x86_64-linux.tar.gz -C $(RELEASE_DIR) $(BINARY_NAME)
	@echo "$(GREEN)Distribution packages created in dist/$(NC)"

.PHONY: docker
docker: ## Build Docker image
	docker build -t tv-discover-cli:$(VERSION) .
	docker tag tv-discover-cli:$(VERSION) tv-discover-cli:latest
```

---

## Build Scripts

### Complete Build Script

```bash
#!/bin/bash
# scripts/build.sh - Complete build script for all platforms

set -e

VERSION=${VERSION:-$(grep '^version' Cargo.toml | head -1 | cut -d'"' -f2)}
BUILD_DIR="build"
DIST_DIR="dist"

echo "Building TV Discover CLI v${VERSION}"

# Clean previous builds
rm -rf ${BUILD_DIR} ${DIST_DIR}
mkdir -p ${BUILD_DIR} ${DIST_DIR}

# Build for Linux (GNU)
echo "Building for Linux (x86_64-unknown-linux-gnu)..."
cross build --release --target x86_64-unknown-linux-gnu
cp target/x86_64-unknown-linux-gnu/release/tvd ${BUILD_DIR}/tvd-linux-amd64
strip ${BUILD_DIR}/tvd-linux-amd64

# Build for Linux (MUSL - static binary)
echo "Building for Linux MUSL (x86_64-unknown-linux-musl)..."
cross build --release --target x86_64-unknown-linux-musl
cp target/x86_64-unknown-linux-musl/release/tvd ${BUILD_DIR}/tvd-linux-amd64-static
strip ${BUILD_DIR}/tvd-linux-amd64-static

# Build for macOS Intel
echo "Building for macOS Intel (x86_64-apple-darwin)..."
cross build --release --target x86_64-apple-darwin
cp target/x86_64-apple-darwin/release/tvd ${BUILD_DIR}/tvd-darwin-amd64
strip ${BUILD_DIR}/tvd-darwin-amd64

# Build for macOS Apple Silicon
echo "Building for macOS Apple Silicon (aarch64-apple-darwin)..."
cross build --release --target aarch64-apple-darwin
cp target/aarch64-apple-darwin/release/tvd ${BUILD_DIR}/tvd-darwin-arm64
strip ${BUILD_DIR}/tvd-darwin-arm64

# Build for Windows
echo "Building for Windows (x86_64-pc-windows-msvc)..."
cross build --release --target x86_64-pc-windows-msvc
cp target/x86_64-pc-windows-msvc/release/tvd.exe ${BUILD_DIR}/tvd-windows-amd64.exe

# Create distribution archives
echo "Creating distribution archives..."

# Linux GNU
tar -czf ${DIST_DIR}/tvd-${VERSION}-linux-amd64.tar.gz -C ${BUILD_DIR} tvd-linux-amd64
echo "$(sha256sum ${DIST_DIR}/tvd-${VERSION}-linux-amd64.tar.gz | cut -d' ' -f1)" > ${DIST_DIR}/tvd-${VERSION}-linux-amd64.tar.gz.sha256

# Linux MUSL
tar -czf ${DIST_DIR}/tvd-${VERSION}-linux-amd64-static.tar.gz -C ${BUILD_DIR} tvd-linux-amd64-static
echo "$(sha256sum ${DIST_DIR}/tvd-${VERSION}-linux-amd64-static.tar.gz | cut -d' ' -f1)" > ${DIST_DIR}/tvd-${VERSION}-linux-amd64-static.tar.gz.sha256

# macOS Intel
tar -czf ${DIST_DIR}/tvd-${VERSION}-darwin-amd64.tar.gz -C ${BUILD_DIR} tvd-darwin-amd64
echo "$(sha256sum ${DIST_DIR}/tvd-${VERSION}-darwin-amd64.tar.gz | cut -d' ' -f1)" > ${DIST_DIR}/tvd-${VERSION}-darwin-amd64.tar.gz.sha256

# macOS ARM
tar -czf ${DIST_DIR}/tvd-${VERSION}-darwin-arm64.tar.gz -C ${BUILD_DIR} tvd-darwin-arm64
echo "$(sha256sum ${DIST_DIR}/tvd-${VERSION}-darwin-arm64.tar.gz | cut -d' ' -f1)" > ${DIST_DIR}/tvd-${VERSION}-darwin-arm64.tar.gz.sha256

# Windows
zip ${DIST_DIR}/tvd-${VERSION}-windows-amd64.zip ${BUILD_DIR}/tvd-windows-amd64.exe
echo "$(sha256sum ${DIST_DIR}/tvd-${VERSION}-windows-amd64.zip | cut -d' ' -f1)" > ${DIST_DIR}/tvd-${VERSION}-windows-amd64.zip.sha256

echo "Build complete! Artifacts in ${DIST_DIR}/"
ls -lh ${DIST_DIR}/
```

---

## CI/CD Pipeline

### GitHub Actions - Complete CI/CD

```yaml
# .github/workflows/ci.yml

name: CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]
  release:
    types: [ created ]

env:
  CARGO_TERM_COLOR: always
  RUST_BACKTRACE: 1

jobs:
  test:
    name: Test Suite
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        rust: [stable, beta]
    steps:
      - uses: actions/checkout@v4

      - name: Install Rust
        uses: dtolnay/rust-toolchain@master
        with:
          toolchain: ${{ matrix.rust }}

      - name: Install Protobuf Compiler
        run: |
          if [ "$RUNNER_OS" == "Linux" ]; then
            sudo apt-get update
            sudo apt-get install -y protobuf-compiler
          elif [ "$RUNNER_OS" == "macOS" ]; then
            brew install protobuf
          elif [ "$RUNNER_OS" == "Windows" ]; then
            choco install protoc
          fi
        shell: bash

      - name: Cache cargo registry
        uses: actions/cache@v3
        with:
          path: ~/.cargo/registry
          key: ${{ runner.os }}-cargo-registry-${{ hashFiles('**/Cargo.lock') }}

      - name: Cache cargo index
        uses: actions/cache@v3
        with:
          path: ~/.cargo/git
          key: ${{ runner.os }}-cargo-index-${{ hashFiles('**/Cargo.lock') }}

      - name: Cache cargo build
        uses: actions/cache@v3
        with:
          path: target
          key: ${{ runner.os }}-cargo-build-target-${{ hashFiles('**/Cargo.lock') }}

      - name: Run tests
        run: cargo test --all-features --verbose

  lint:
    name: Linting
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable
        with:
          components: rustfmt, clippy

      - name: Install Protobuf Compiler
        run: sudo apt-get update && sudo apt-get install -y protobuf-compiler

      - name: Check formatting
        run: cargo fmt --all -- --check

      - name: Run clippy
        run: cargo clippy --all-targets --all-features -- -D warnings

  security:
    name: Security Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run cargo audit
        uses: actions-rs/audit-check@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

  coverage:
    name: Code Coverage
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable

      - name: Install Protobuf Compiler
        run: sudo apt-get update && sudo apt-get install -y protobuf-compiler

      - name: Install cargo-llvm-cov
        run: cargo install cargo-llvm-cov

      - name: Generate coverage
        run: cargo llvm-cov --all-features --workspace --lcov --output-path lcov.info

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: lcov.info
          fail_ci_if_error: true

  build:
    name: Build Release
    needs: [test, lint]
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            target: x86_64-unknown-linux-gnu
            artifact_name: tvd
            asset_name: tvd-linux-amd64
          - os: ubuntu-latest
            target: x86_64-unknown-linux-musl
            artifact_name: tvd
            asset_name: tvd-linux-amd64-static
          - os: macos-latest
            target: x86_64-apple-darwin
            artifact_name: tvd
            asset_name: tvd-darwin-amd64
          - os: macos-latest
            target: aarch64-apple-darwin
            artifact_name: tvd
            asset_name: tvd-darwin-arm64
          - os: windows-latest
            target: x86_64-pc-windows-msvc
            artifact_name: tvd.exe
            asset_name: tvd-windows-amd64.exe
    steps:
      - uses: actions/checkout@v4

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.target }}

      - name: Install Protobuf Compiler
        run: |
          if [ "$RUNNER_OS" == "Linux" ]; then
            sudo apt-get update
            sudo apt-get install -y protobuf-compiler
          elif [ "$RUNNER_OS" == "macOS" ]; then
            brew install protobuf
          elif [ "$RUNNER_OS" == "Windows" ]; then
            choco install protoc
          fi
        shell: bash

      - name: Build release
        run: cargo build --release --locked --target ${{ matrix.target }}

      - name: Strip binary (Unix)
        if: runner.os != 'Windows'
        run: strip target/${{ matrix.target }}/release/${{ matrix.artifact_name }}

      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: ${{ matrix.asset_name }}
          path: target/${{ matrix.target }}/release/${{ matrix.artifact_name }}

  release:
    name: Create Release
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name == 'release'
    steps:
      - uses: actions/checkout@v4

      - name: Download all artifacts
        uses: actions/download-artifact@v3
        with:
          path: artifacts

      - name: Create archives
        run: |
          cd artifacts
          for dir in */; do
            dirname=$(basename "$dir")
            cd "$dir"
            if [[ $dirname == *"windows"* ]]; then
              zip "../${dirname}.zip" *
            else
              tar -czf "../${dirname}.tar.gz" *
            fi
            cd ..
          done

      - name: Upload release assets
        uses: softprops/action-gh-release@v1
        with:
          files: artifacts/*.*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml

stages:
  - test
  - build
  - release

variables:
  CARGO_HOME: $CI_PROJECT_DIR/.cargo
  RUST_BACKTRACE: "1"

cache:
  paths:
    - .cargo/
    - target/

test:
  stage: test
  image: rust:latest
  before_script:
    - apt-get update && apt-get install -y protobuf-compiler
    - rustup component add clippy rustfmt
  script:
    - cargo fmt -- --check
    - cargo clippy -- -D warnings
    - cargo test --all-features
  only:
    - merge_requests
    - main
    - develop

security-audit:
  stage: test
  image: rust:latest
  script:
    - cargo install cargo-audit
    - cargo audit
  allow_failure: true

build:linux:
  stage: build
  image: rust:latest
  before_script:
    - apt-get update && apt-get install -y protobuf-compiler
  script:
    - cargo build --release --locked
    - strip target/release/tvd
  artifacts:
    paths:
      - target/release/tvd
    expire_in: 1 week

build:docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
```

---

## Release Process

### Version Bumping Script

```bash
#!/bin/bash
# scripts/bump-version.sh

set -e

CURRENT_VERSION=$(grep '^version' Cargo.toml | head -1 | cut -d'"' -f2)
echo "Current version: $CURRENT_VERSION"

if [ -z "$1" ]; then
    echo "Usage: $0 <major|minor|patch|VERSION>"
    exit 1
fi

case $1 in
    major|minor|patch)
        NEW_VERSION=$(cargo set-version --bump $1 --dry-run | grep 'version' | cut -d'"' -f2)
        ;;
    *)
        NEW_VERSION=$1
        ;;
esac

echo "Bumping version to: $NEW_VERSION"

# Update Cargo.toml
sed -i.bak "s/^version = .*/version = \"$NEW_VERSION\"/" Cargo.toml
rm Cargo.toml.bak

# Update Cargo.lock
cargo check

# Commit changes
git add Cargo.toml Cargo.lock
git commit -m "chore: bump version to $NEW_VERSION"

# Create tag
git tag -a "v$NEW_VERSION" -m "Release v$NEW_VERSION"

echo "Version bumped to $NEW_VERSION"
echo "Push with: git push && git push --tags"
```

### Release Checklist

```markdown
# Release Checklist

## Pre-release
- [ ] All tests passing
- [ ] Security audit clean
- [ ] Documentation updated
- [ ] CHANGELOG.md updated
- [ ] Version bumped
- [ ] Built for all platforms
- [ ] Smoke tests completed

## Release
- [ ] Tag created
- [ ] GitHub release created
- [ ] Binaries uploaded
- [ ] Checksums generated
- [ ] Release notes published

## Post-release
- [ ] Homebrew formula updated
- [ ] Chocolatey package updated
- [ ] Cargo crate published
- [ ] Docker image pushed
- [ ] Documentation site updated
- [ ] Announcement posted
```

---

## Distribution

### Homebrew Formula

```ruby
# Formula/tvd.rb

class Tvd < Formula
  desc "TV Discovery CLI - Search and discover content across streaming platforms"
  homepage "https://tv-discover.com"
  url "https://github.com/tv-discover/cli-core/archive/v0.1.0.tar.gz"
  sha256 "..."
  license "MIT"

  depends_on "rust" => :build
  depends_on "protobuf" => :build

  def install
    system "cargo", "install", "--locked", "--root", prefix, "--path", "."

    # Install shell completions
    generate_completions_from_executable(bin/"tvd", "completions")
  end

  test do
    assert_match "tvd #{version}", shell_output("#{bin}/tvd --version")
  end
end
```

### Chocolatey Package

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- tvd.nuspec -->
<package xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">
  <metadata>
    <id>tvd</id>
    <version>0.1.0</version>
    <title>TV Discover CLI</title>
    <authors>TV Discover Team</authors>
    <owners>TV Discover Team</owners>
    <projectUrl>https://tv-discover.com</projectUrl>
    <licenseUrl>https://github.com/tv-discover/cli-core/blob/main/LICENSE</licenseUrl>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>
      Unified CLI for discovering TV content across streaming platforms.
      Search, browse, and manage your watchlist from the command line.
    </description>
    <summary>TV Discovery CLI</summary>
    <tags>cli tv streaming discovery entertainment</tags>
  </metadata>
  <files>
    <file src="tools\**" target="tools" />
  </files>
</package>
```

### Docker Image

```dockerfile
# Dockerfile

# Build stage
FROM rust:1.75 as builder

WORKDIR /usr/src/tvd

# Install protobuf compiler
RUN apt-get update && apt-get install -y protobuf-compiler && rm -rf /var/lib/apt/lists/*

# Copy manifests
COPY Cargo.toml Cargo.lock ./
COPY proto proto/

# Copy source
COPY src src/
COPY build.rs .

# Build release
RUN cargo build --release --locked

# Runtime stage
FROM debian:bookworm-slim

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    ca-certificates \
    libssl3 \
    && rm -rf /var/lib/apt/lists/*

# Copy binary from builder
COPY --from=builder /usr/src/tvd/target/release/tvd /usr/local/bin/tvd

# Create non-root user
RUN useradd -m -u 1000 tvduser
USER tvduser

# Set entrypoint
ENTRYPOINT ["/usr/local/bin/tvd"]
CMD ["--help"]
```

---

## Installation Methods

### Installation Script (Unix)

```bash
#!/bin/sh
# install.sh - Universal installation script

set -e

# Detect OS and architecture
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)

case "$ARCH" in
    x86_64) ARCH="amd64" ;;
    aarch64) ARCH="arm64" ;;
    *) echo "Unsupported architecture: $ARCH"; exit 1 ;;
esac

# Determine download URL
VERSION="${VERSION:-latest}"
if [ "$VERSION" = "latest" ]; then
    VERSION=$(curl -s https://api.github.com/repos/tv-discover/cli-core/releases/latest | grep '"tag_name"' | sed -E 's/.*"v([^"]+)".*/\1/')
fi

DOWNLOAD_URL="https://github.com/tv-discover/cli-core/releases/download/v${VERSION}/tvd-${VERSION}-${OS}-${ARCH}.tar.gz"

# Download and install
TMP_DIR=$(mktemp -d)
cd "$TMP_DIR"

echo "Downloading tvd v${VERSION}..."
curl -L "$DOWNLOAD_URL" -o tvd.tar.gz

echo "Installing..."
tar -xzf tvd.tar.gz
sudo mv tvd-${OS}-${ARCH} /usr/local/bin/tvd
sudo chmod +x /usr/local/bin/tvd

# Cleanup
cd -
rm -rf "$TMP_DIR"

echo "tvd v${VERSION} installed successfully!"
echo "Run 'tvd --help' to get started."
```

### Installation via Cargo

```bash
# Install from crates.io
cargo install tv-discover-cli

# Or from source
cargo install --git https://github.com/tv-discover/cli-core

# Or from local directory
git clone https://github.com/tv-discover/cli-core
cd cli-core
cargo install --path .
```

### Package Manager Commands

```bash
# Homebrew (macOS/Linux)
brew tap tv-discover/tap
brew install tvd

# Chocolatey (Windows)
choco install tvd

# Scoop (Windows)
scoop bucket add tv-discover https://github.com/tv-discover/scoop-bucket
scoop install tvd

# APT (Debian/Ubuntu)
curl -fsSL https://tv-discover.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/tvd.gpg
echo "deb [signed-by=/usr/share/keyrings/tvd.gpg] https://apt.tv-discover.com stable main" | sudo tee /etc/apt/sources.list.d/tvd.list
sudo apt update
sudo apt install tvd

# YUM/DNF (RHEL/Fedora)
sudo dnf config-manager --add-repo https://yum.tv-discover.com/tvd.repo
sudo dnf install tvd
```

---

## Conclusion

This build and deployment guide provides complete automation for:

1. **Multi-platform compilation** with optimized binaries
2. **Comprehensive CI/CD** with testing, linting, and security checks
3. **Automated releases** with GitHub Actions and GitLab CI
4. **Multiple distribution channels** including package managers
5. **Container support** with Docker
6. **Easy installation** for end users

The build system ensures consistent, reproducible builds across all supported platforms while maintaining high code quality standards.
