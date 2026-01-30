# 构建打包指南 (Build Guide)

本文档详细说明如何从源代码构建和打包 Antigravity Tools。

## 📋 目录

- [前置要求](#前置要求)
- [快速开始](#快速开始)
- [开发环境设置](#开发环境设置)
- [本地开发](#本地开发)
- [生产打包](#生产打包)
- [平台特定说明](#平台特定说明)
- [常见问题](#常见问题)

## 🔧 前置要求

### 必需软件

1. **Node.js** (推荐 v20 或更高版本)
   - 下载地址: https://nodejs.org/
   - 验证安装: `node --version`

2. **Rust** (稳定版)
   - 安装命令:
     ```bash
     curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
     ```
   - 验证安装: `rustc --version`

3. **npm** 或 **pnpm** (包管理器)
   - npm 随 Node.js 自动安装
   - 或安装 pnpm: `npm install -g pnpm`

### 平台特定依赖

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
- 安装 [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
- 安装 [WebView2](https://developer.microsoft.com/en-us/microsoft-edge/webview2/)

## 🚀 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/lbjlaq/Antigravity-Manager.git
cd Antigravity-Manager

# 2. 安装依赖
npm install

# 3. 开发模式运行
npm run tauri dev

# 4. 生产打包
npm run tauri build
```

## 🛠️ 开发环境设置

### 1. 克隆项目

```bash
git clone https://github.com/lbjlaq/Antigravity-Manager.git
cd Antigravity-Manager
```

### 2. 安装前端依赖

```bash
# 使用 npm
npm install

# 或使用 pnpm
pnpm install
```

### 3. 验证 Rust 环境

```bash
cd src-tauri
cargo check
cd ..
```

## 💻 本地开发

### 启动开发服务器

```bash
# 标准开发模式
npm run tauri dev

# 带调试日志的开发模式
npm run tauri:debug
```

这将会：
1. 启动 Vite 开发服务器 (端口 1420)
2. 编译 Rust 后端
3. 启动 Tauri 应用窗口
4. 启用热重载 (前端代码修改会自动刷新)

### 仅前端开发

如果只需要开发前端界面：

```bash
npm run dev
```

然后在浏览器访问 `http://localhost:1420`

### 运行测试

```bash
# Rust 后端测试
cd src-tauri
cargo test --all --all-features

# 返回项目根目录
cd ..
```

## 📦 生产打包

### 构建所有平台

```bash
npm run tauri build
```

### 构建特定平台

#### macOS

```bash
# Apple Silicon (M1/M2/M3)
npm run tauri build -- --target aarch64-apple-darwin

# Intel Mac
npm run tauri build -- --target x86_64-apple-darwin

# Universal Binary (同时支持 Intel 和 Apple Silicon)
npm run tauri build -- --target universal-apple-darwin
```

#### Linux

```bash
# 默认构建 (生成 .deb 和 AppImage)
npm run tauri build

# 仅构建 AppImage
npm run tauri build -- --bundles appimage

# 仅构建 .deb
npm run tauri build -- --bundles deb
```

#### Windows

```bash
# 默认构建 (生成 .msi 和便携版)
npm run tauri build

# 仅构建 MSI 安装包
npm run tauri build -- --bundles msi

# 仅构建 NSIS 安装包
npm run tauri build -- --bundles nsis
```

### 构建产物位置

构建完成后，产物位于：

```
src-tauri/target/release/bundle/
├── macos/          # macOS .app 和 .dmg
├── deb/            # Linux .deb 包
├── appimage/       # Linux AppImage
├── msi/            # Windows .msi 安装包
└── nsis/           # Windows NSIS 安装包
```

## 🔍 平台特定说明

### macOS

#### 代码签名

如果需要分发给其他用户，建议进行代码签名：

1. 准备 Apple 开发者证书
2. 设置环境变量：
   ```bash
   export APPLE_CERTIFICATE="证书名称"
   export APPLE_CERTIFICATE_PASSWORD="证书密码"
   export APPLE_ID="你的Apple ID"
   export APPLE_PASSWORD="应用专用密码"
   ```
3. 构建：
   ```bash
   npm run tauri build
   ```

#### 创建 DMG

```bash
# 使用提供的脚本
./scripts/package_dmg.sh
```

#### 修复 "应用已损坏" 问题

本地构建的应用可能被 macOS Gatekeeper 阻止，使用以下命令修复：

```bash
sudo xattr -rd com.apple.quarantine "/Applications/Antigravity Tools.app"
```

### Linux

#### AppImage 权限

构建的 AppImage 需要可执行权限：

```bash
chmod +x src-tauri/target/release/bundle/appimage/*.AppImage
```

#### 系统托盘支持

确保已安装系统托盘库：

```bash
# Ubuntu/Debian
sudo apt-get install libayatana-appindicator3-dev

# Arch
sudo pacman -S libayatana-appindicator
```

### Windows

#### 管理员权限

某些功能可能需要管理员权限，在 `src-tauri/tauri.conf.json` 中配置：

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

## 🐳 Docker 构建

### 构建 Docker 镜像

```bash
cd docker
docker build -t antigravity-tools:local .
```

### 运行容器

```bash
docker run -d \
  --name antigravity \
  -p 8045:8045 \
  -e API_KEY=sk-your-api-key \
  -v ~/.antigravity_tools:/root/.antigravity_tools \
  antigravity-tools:local
```

## ❓ 常见问题

### Q: 构建失败，提示找不到 webkit2gtk

**A:** 安装 WebKit2GTK 开发包：

```bash
# Ubuntu/Debian
sudo apt-get install libwebkit2gtk-4.1-dev

# Arch
sudo pacman -S webkit2gtk
```

### Q: macOS 上提示 "xcrun: error: invalid active developer path"

**A:** 安装 Xcode Command Line Tools：

```bash
xcode-select --install
```

### Q: Windows 上构建失败，提示 MSVC 相关错误

**A:** 安装 Microsoft C++ Build Tools，确保包含以下组件：
- MSVC v142 或更高版本
- Windows 10 SDK

### Q: Rust 编译很慢或卡住

**A:** 尝试以下方法：

1. 清理构建缓存：
   ```bash
   cd src-tauri
   cargo clean
   cd ..
   ```

2. 使用国内镜像源（如果在中国）：
   创建 `~/.cargo/config.toml`：
   ```toml
   [source.crates-io]
   replace-with = 'ustc'
   
   [source.ustc]
   registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
   ```

3. 增加并行编译任务：
   ```bash
   export CARGO_BUILD_JOBS=4
   ```

### Q: 前端构建失败，提示 TypeScript 错误

**A:** 确保 TypeScript 版本正确：

```bash
npm install typescript@~5.8.3 --save-dev
```

### Q: 如何减小构建产物大小？

**A:** 在 `src-tauri/Cargo.toml` 中启用更多优化：

```toml
[profile.release]
opt-level = 'z'     # 优化大小
lto = true          # 链接时优化
codegen-units = 1   # 更好的优化
strip = true        # 移除符号
```

### Q: 开发模式下应用启动很慢

**A:** 这是正常现象，因为 Rust 编译需要时间。后续启动会快很多。可以尝试：

1. 使用增量编译（默认启用）
2. 使用更快的链接器：
   ```bash
   # macOS/Linux
   cargo install mold
   
   # 在 Cargo.toml 中配置
   [target.x86_64-unknown-linux-gnu]
   linker = "clang"
   rustflags = ["-C", "link-arg=-fuse-ld=mold"]
   ```

### Q: 如何查看详细的构建日志？

**A:** 启用详细日志：

```bash
# Tauri 构建日志
npm run tauri build -- --verbose

# Rust 编译日志
RUST_LOG=debug npm run tauri dev
```

## 📚 更多资源

- [Tauri 官方文档](https://tauri.app/v2/guides/)
- [Rust 官方教程](https://www.rust-lang.org/learn)
- [项目 Issue 跟踪](https://github.com/lbjlaq/Antigravity-Manager/issues)
- [贡献指南](../CONTRIBUTING.md) (如果有)

## 🤝 获取帮助

如果遇到问题：

1. 查看本文档的常见问题部分
2. 搜索 [GitHub Issues](https://github.com/lbjlaq/Antigravity-Manager/issues)
3. 提交新的 Issue 并提供详细信息：
   - 操作系统和版本
   - Node.js 和 Rust 版本
   - 完整的错误日志
   - 复现步骤

---

**祝构建顺利！** 🚀
