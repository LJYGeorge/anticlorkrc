# 网站资源爬虫工具部署指南

## 目录
- [系统要求](#系统要求)
- [安装步骤](#安装步骤)
- [配置说明](#配置说明)
- [运行说明](#运行说明)
- [常见问题](#常见问题)
- [故障排除](#故障排除)
- [安全建议](#安全建议)

## 系统要求

### 基础环境
- Node.js 18.0.0 或更高版本
- npm 9.0.0 或更高版本
- 现代浏览器（Chrome、Firefox、Edge 等）
- 2GB 以上可用内存
- 10GB 以上可用磁盘空间

### 操作系统支持
- Windows 10/11
- macOS 10.15+
- Ubuntu 20.04+ / CentOS 8+

### 网络要求
- 稳定的互联网连接
- 支持 HTTP/HTTPS 协议
- 可选：代理服务器支持

## 安装步骤

### 1. 环境准备

```bash
# 检查 Node.js 版本
node --version

# 检查 npm 版本
npm --version

# 安装/更新 Node.js（如需要）
# Windows: 从 https://nodejs.org 下载安装包
# macOS: 
brew install node

# Linux:
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 2. 项目安装

```bash
# 克隆项目
git clone <项目地址>
cd website-crawler

# 安装依赖
npm install

# 创建配置文件
cp .env.example .env
```

### 3. 配置环境变量

编辑 `.env` 文件：

```env
# 服务器配置
PORT=3000
NODE_ENV=production

# 存储配置
SAVE_PATH=/path/to/downloads
MAX_CONCURRENT=5

# 安全配置
RATE_LIMIT=100
TIMEOUT=30000

# 代理配置（可选）
PROXY_ENABLED=false
PROXY_HOST=
PROXY_PORT=
PROXY_USERNAME=
PROXY_PASSWORD=
```

### 4. 设置存储权限

```bash
# Linux/macOS
sudo mkdir -p /path/to/downloads
sudo chown -R $USER:$USER /path/to/downloads
sudo chmod 755 /path/to/downloads

# Windows (管理员 PowerShell)
New-Item -Path "C:\path\to\downloads" -ItemType Directory
icacls "C:\path\to\downloads" /grant Users:(OI)(CI)F
```

## 配置说明

### 1. 基础配置

`config.js` 主要配置项：

```javascript
{
  // 爬虫配置
  crawler: {
    maxDepth: 2,           // 最大爬取深度
    maxConcurrent: 5,      // 最大并发请求数
    timeout: 30000,        // 请求超时时间（毫秒）
    retries: 3             // 失败重试次数
  },

  // 资源配置
  resources: {
    images: true,          // 是否下载图片
    styles: true,          // 是否下载样式表
    scripts: true,         // 是否下载脚本
    documents: false       // 是否下载文档
  },

  // 反检测配置
  antiDetection: {
    enabled: true,         // 启用反检测
    rotateUserAgent: true, // 随机切换 User-Agent
    useProxy: false,       // 使用代理
    delay: {
      min: 1000,          // 最小延迟（毫秒）
      max: 3000           // 最大延迟（毫秒）
    }
  }
}
```

### 2. 安全配置

```javascript
{
  security: {
    rateLimit: {
      windowMs: 15 * 60 * 1000,  // 15 分钟
      max: 100                    // 最大请求次数
    },
    cors: {
      origin: '*',               // 允许的来源
      methods: ['GET', 'POST']   // 允许的方法
    },
    helmet: {
      enabled: true,            // 启用安全头
      hidePoweredBy: true      // 隐藏服务器信息
    }
  }
}
```

## 运行说明

### 开发环境

```bash
# 启动开发服务器
npm run dev

# 运行测试
npm test

# 代码检查
npm run lint
```

### 生产环境

```bash
# 构建项目
npm run build

# 启动服务
npm start

# 使用 PM2 运行（推荐）
pm2 start npm --name "crawler" -- start
```

## 常见问题

### 1. 安装问题

Q: 安装依赖时报错 `node-gyp rebuild` 失败
A: 检查系统是否安装了必要的编译工具
```bash
# Windows
npm install --global --production windows-build-tools

# Linux
sudo apt-get install build-essential python3

# macOS
xcode-select --install
```

Q: 提示 `EACCES: permission denied`
A: 修复 npm 权限问题
```bash
sudo chown -R $USER:$USER ~/.npm
sudo chown -R $USER:$USER ~/.config
```

### 2. 运行问题

Q: 服务启动失败，提示端口被占用
A: 更改配置文件中的端口号，或关闭占用端口的程序
```bash
# 查找占用端口的进程
lsof -i :3000
# 或
netstat -ano | findstr :3000
```

Q: 内存使用过高
A: 调整配置中的并发数和缓存设置
```javascript
{
  maxConcurrent: 3,        // 降低并发数
  maxMemoryCache: 100,     // 限制内存缓存（MB）
  garbageCollection: true  // 启用主动垃圾回收
}
```

### 3. 爬取问题

Q: 某些网站无法爬取
A: 检查以下几点：
- 确认网站是否允许爬虫（robots.txt）
- 尝试启用代理
- 调整请求延迟
- 检查是否需要特定的请求头

Q: 下载的资源不完整
A: 可能的解决方案：
- 增加请求超时时间
- 启用失败重试
- 检查磁盘空间
- 验证文件权限

### 4. 性能问题

Q: CPU 使用率过高
A: 优化配置：
```javascript
{
  optimization: {
    useWorkerThreads: true,    // 启用工作线程
    maxWorkers: 4,             // 限制工作线程数
    batchSize: 10              // 调整批处理大小
  }
}
```

Q: 磁盘 I/O 过高
A: 调整存储策略：
```javascript
{
  storage: {
    useCompression: true,      // 启用压缩
    compressionLevel: 6,       // 压缩级别（1-9）
    bufferSize: 64 * 1024     // 缓冲区大小（字节）
  }
}
```

## 故障排除

### 1. 日志检查

```bash
# 查看应用日志
tail -f logs/app.log

# 查看错误日志
tail -f logs/error.log

# 查看系统日志
sudo journalctl -u crawler
```

### 2. 性能监控

```bash
# 监控 CPU 和内存
top -p $(pgrep -f crawler)

# 监控磁盘使用
df -h
du -sh /path/to/downloads/*
```

### 3. 网络诊断

```bash
# 检查网络连接
curl -v https://target-site.com

# 测试代理连接
curl -v -x proxy.example.com:8080 https://target-site.com
```

## 安全建议

### 1. 系统安全

- 定期更新系统和依赖包
- 使用非 root 用户运行应用
- 配置防火墙规则
- 启用 HTTPS

### 2. 数据安全

- 定期备份下载的资源
- 加密敏感配置信息
- 实施访问控制
- 监控异常活动

### 3. 网络安全

- 使用 VPN 或代理
- 实施请求限速
- 避免恶意网站
- 保护 API 密钥

## 维护建议

### 1. 定期维护

```bash
# 更新依赖
npm update

# 清理缓存
npm cache clean --force

# 磁盘清理
find /path/to/downloads -type f -atime +30 -delete
```

### 2. 监控告警

建议配置以下监控：
- CPU 使用率 > 80%
- 内存使用率 > 90%
- 磁盘使用率 > 85%
- 请求失败率 > 10%
- 响应时间 > 5s

### 3. 备份策略

```bash
# 自动备份脚本示例
#!/bin/bash
backup_dir="/path/to/backups"
date_str=$(date +%Y%m%d)
tar -czf "$backup_dir/crawler_$date_str.tar.gz" /path/to/downloads
find "$backup_dir" -type f -mtime +7 -delete
```

## 技术支持

如遇问题，请：
1. 查看详细日志
2. 检查配置文件
3. 参考故障排除指南
4. 提交 Issue 获取帮助

## 许可证

MIT 许可证 - 详见 LICENSE 文件