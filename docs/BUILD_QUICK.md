# 快速构建参考 (Quick Build Reference)

## 🚀 一分钟快速开始

### 首次构建
```bash
# 1. 安装依赖
npm install

# 2. 开发运行
npm run tauri dev

# 3. 生产打包
npm run tauri build
```

## 📝 常用命令

### 开发
```bash
# 标准开发模式
npm run tauri dev

# 带调试日志
npm run tauri:debug

# 仅前端开发
npm run dev
```

### 测试
```bash
# 运行测试
cd src-tauri && cargo test
```

### 构建
```bash
# 当前平台
npm run tauri build

# macOS Apple Silicon
npm run tauri build -- --target aarch64-apple-darwin

# macOS Intel
npm run tauri build -- --target x86_64-apple-darwin

# macOS Universal
npm run tauri build -- --target universal-apple-darwin

# Linux AppImage
npm run tauri build -- --bundles appimage

# Linux .deb
npm run tauri build -- --bundles deb

# Windows MSI
npm run tauri build -- --bundles msi
```

## 🔧 前置要求检查

```bash
# 检查 Node.js
node --version  # 需要 v20+

# 检查 Rust
rustc --version  # 需要稳定版

# 检查 npm
npm --version
```

## 📦 构建产物位置

```
src-tauri/target/release/bundle/
├── macos/          # .app 和 .dmg
├── deb/            # .deb 包
├── appimage/       # AppImage
├── msi/            # .msi 安装包
└── nsis/           # NSIS 安装包
```

## ❓ 遇到问题？

查看完整文档：
- **[中文详细指南](./BUILD_ZH.md)**
- **[English Guide](./BUILD_EN.md)**

或前往 [GitHub Issues](https://github.com/lbjlaq/Antigravity-Manager/issues)

---

**提示**: 首次编译 Rust 后端可能需要 5-10 分钟，后续会快很多。
