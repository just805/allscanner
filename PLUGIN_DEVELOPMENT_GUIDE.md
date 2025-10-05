# Hidden Shield 插件开发指南

> 版本：1.0.0  
> 更新时间：2025-10-06  
> 适用于：Hidden Shield v0.0.1+

---

## 📚 目录

1. [插件系统概述](#1-插件系统概述)
2. [开发环境准备](#2-开发环境准备)
3. [快速开始](#3-快速开始)
4. [插件项目结构](#4-插件项目结构)
5. [配置文件详解](#5-配置文件详解)
6. [插件生命周期](#6-插件生命周期)
7. [数据输出规范](#7-数据输出规范)
8. [调试与测试](#8-调试与测试)
9. [打包与发布](#9-打包与发布)
10. [最佳实践](#10-最佳实践)
11. [常见问题](#11-常见问题)

---

## 1. 插件系统概述

### 1.1 什么是 Hidden Shield 插件？

Hidden Shield 插件是独立的安全工具模块，可以扩展系统的扫描和检测能力。插件具有以下特点：

- ✅ **独立运行**：每个插件是独立的可执行程序
- ✅ **标准化接口**：通过标准输入/输出与系统交互
- ✅ **多语言支持**：支持 Go、Python、Node.js 等运行时
- ✅ **沙箱隔离**：插件在隔离环境中运行，保证系统安全
- ✅ **热插拔**：无需重启系统即可安装/卸载插件

### 1.2 插件分类

| 类别 | 说明 | 示例 |
|------|------|------|
| **scan** | 主动扫描 | 端口扫描、漏洞扫描 |
| **fingerprint** | 指纹识别 | Web框架识别、设备指纹 |
| **asset** | 资产收集 | 子域名收集、IP探测 |
| **exploit** | 漏洞利用 | SQL注入、XSS测试 |
| **report** | 报告生成 | PDF报告、Excel导出 |

### 1.3 支持的运行时

- **Go** (推荐)：高性能、编译型，适合扫描类插件
- **Python**：丰富的库，适合数据处理和AI分析
- **Node.js**：适合Web接口和异步任务
- **Shell**：适合系统集成和简单脚本

---

## 2. 开发环境准备

### 2.1 必需工具

```bash
# Go 开发环境 (推荐 1.19+)
go version

# 或 Python 环境 (推荐 3.8+)
python --version

# 或 Node.js 环境 (推荐 16+)
node --version
```

### 2.2 克隆开发模板

```bash
# 进入插件开发目录
cd plugins/

# 方式1: 复制现有插件作为模板
cp -r asset_collector my_plugin

# 方式2: 从零创建
mkdir my_plugin && cd my_plugin
```

### 2.3 安装开发工具（可选）

```bash
# 插件验证工具
go install github.com/hidden-shield/plugin-validator@latest

# 插件打包工具
go install github.com/hidden-shield/plugin-packager@latest
```

---

## 3. 快速开始

### 3.1 创建第一个插件（Go）

**步骤1：创建项目结构**

```bash
mkdir my_scanner && cd my_scanner
go mod init my_scanner
```

**步骤2：编写 main.go**

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Result struct {
	Status  string                 `json:"status"`
	Message string                 `json:"message"`
	Data    map[string]interface{} `json:"data"`
}

func main() {
	// 1. 获取目标参数
	if len(os.Args) < 2 {
		outputError("缺少目标参数")
		os.Exit(1)
	}
	target := os.Args[1]

	// 2. 执行扫描逻辑
	fmt.Fprintf(os.Stderr, "INFO: 开始扫描目标 %s\n", target)
	
	// TODO: 实现你的扫描逻辑
	data := map[string]interface{}{
		"target":    target,
		"ports":     []int{80, 443, 8080},
		"timestamp": "2025-10-06T00:00:00Z",
	}

	// 3. 输出结果
	outputSuccess("扫描完成", data)
}

func outputSuccess(message string, data map[string]interface{}) {
	result := Result{
		Status:  "success",
		Message: message,
		Data:    data,
	}
	json.NewEncoder(os.Stdout).Encode(result)
}

func outputError(message string) {
	result := Result{
		Status:  "error",
		Message: message,
		Data:    nil,
	}
	json.NewEncoder(os.Stdout).Encode(result)
}
```

**步骤3：创建 metadata.json**

```json
{
  "name": "我的扫描器",
  "key": "my_scanner",
  "category": "scan",
  "version": "1.0.0",
  "author": "Your Name",
  "description": "这是一个示例扫描插件",
  "tags": ["扫描", "示例"],
  "runtime": "go",
  "entry": "main.go",
  "permissions": {
    "network": ["*"],
    "filesystem": [],
    "system": ["tcp_connect"]
  },
  "config": {
    "timeout": 60,
    "maxWorkers": 10
  },
  "inputFormat": "IP地址或域名",
  "outputFormat": "json"
}
```

**步骤4：测试运行**

```bash
go run main.go example.com
```

**预期输出：**
```json
{
  "status": "success",
  "message": "扫描完成",
  "data": {
    "target": "example.com",
    "ports": [80, 443, 8080],
    "timestamp": "2025-10-06T00:00:00Z"
  }
}
```

---

## 4. 插件项目结构

### 4.1 标准结构（Go插件）

```
my_plugin/
├── main.go                 # 主程序入口 (必需)
├── go.mod                  # Go模块依赖 (必需)
├── go.sum                  # 依赖校验文件
├── metadata.json           # 插件元数据 (必需)
├── config.json             # 默认配置文件 (可选)
├── README.md               # 插件说明文档 (推荐)
├── CHANGELOG.md            # 版本更新日志 (推荐)
├── config/                 # 配置模块
│   ├── config.go
│   └── config.json
├── modules/                # 子模块目录
│   ├── scanner/
│   │   └── scanner.go
│   └── parser/
│       └── parser.go
├── types/                  # 数据类型定义
│   └── types.go
├── utils/                  # 工具函数
│   └── utils.go
├── wordlists/              # 字典文件
│   └── common.txt
├── payloads/               # 测试载荷
│   └── xss.txt
└── tests/                  # 单元测试
    └── scanner_test.go
```

### 4.2 Python插件结构

```
my_plugin/
├── main.py                 # 主程序入口 (必需)
├── requirements.txt        # Python依赖 (必需)
├── metadata.json           # 插件元数据 (必需)
├── config.json             # 默认配置
├── README.md
├── modules/
│   ├── __init__.py
│   └── scanner.py
└── tests/
    └── test_scanner.py
```

### 4.3 Node.js插件结构

```
my_plugin/
├── index.js                # 主程序入口 (必需)
├── package.json            # npm配置 (必需)
├── metadata.json           # 插件元数据 (必需)
├── config.json
├── README.md
├── src/
│   └── scanner.js
└── tests/
    └── scanner.test.js
```

---

## 5. 配置文件详解

### 5.1 metadata.json（插件元数据）

**完整示例：**

```json
{
  "name": "端口扫描器",
  "key": "port_scanner",
  "category": "scan",
  "version": "1.2.0",
  "author": "Security Team <security@example.com>",
  "description": "高性能端口扫描工具，支持TCP/UDP扫描和服务识别",
  "tags": ["端口扫描", "TCP", "UDP", "服务识别"],
  "runtime": "go",
  "entry": "main.go",
  "icon": "data:image/svg+xml;base64,...",
  "homepage": "https://github.com/your-org/port-scanner",
  "license": "MIT",
  
  "permissions": {
    "network": ["*"],
    "filesystem": ["/tmp/scanner/*", "output/*"],
    "system": ["tcp_connect", "udp_send", "dns_query"],
    "data": ["read", "write"]
  },
  
  "dependencies": [
    "github.com/projectdiscovery/naabu@v2.0.0",
    "golang.org/x/net"
  ],
  
  "config": {
    "timeout": 300,
    "maxWorkers": 50,
    "retryTimes": 3,
    "ports": "1-65535",
    "scanType": "syn",
    "serviceDetect": true
  },
  
  "inputFormat": "IP地址、CIDR、域名",
  "outputFormat": "json",
  "estimatedTime": "1-5分钟（视目标规模）",
  
  "features": [
    "支持TCP和UDP扫描",
    "服务版本检测",
    "自动排除CDN",
    "支持自定义端口范围"
  ],
  
  "changelog": [
    {
      "version": "1.2.0",
      "date": "2025-10-01",
      "changes": [
        "新增服务指纹识别",
        "优化扫描速度（提升30%）",
        "修复UDP扫描bug"
      ]
    }
  ]
}
```

**字段说明：**

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 插件显示名称 |
| `key` | string | ✅ | 插件唯一标识（小写字母+下划线） |
| `category` | string | ✅ | 插件分类（scan/fingerprint/asset/exploit/report） |
| `version` | string | ✅ | 版本号（遵循语义化版本） |
| `author` | string | ✅ | 作者信息 |
| `description` | string | ✅ | 插件描述（200字以内） |
| `tags` | array | ✅ | 标签（用于搜索） |
| `runtime` | string | ✅ | 运行时（go/python/nodejs/shell） |
| `entry` | string | ✅ | 入口文件 |
| `permissions` | object | ❌ | 权限声明 |
| `dependencies` | array | ❌ | 依赖列表 |
| `config` | object | ❌ | 默认配置 |
| `inputFormat` | string | ❌ | 输入格式说明 |
| `outputFormat` | string | ❌ | 输出格式（json/text/xml） |
| `features` | array | ❌ | 功能特性列表 |
| `changelog` | array | ❌ | 版本更新日志 |

### 5.2 config.json（运行时配置）

```json
{
  "timeout": 300,
  "maxWorkers": 10,
  "retryTimes": 3,
  "debug": false,
  
  "scanner": {
    "ports": "80,443,8080-8090",
    "threads": 50,
    "scanType": "syn"
  },
  
  "output": {
    "format": "json",
    "path": "./output",
    "verbose": true
  },
  
  "integrations": {
    "fofa": {
      "enabled": true,
      "email": "${FOFA_EMAIL}",
      "key": "${FOFA_KEY}"
    }
  }
}
```

**支持环境变量**：使用 `${VAR_NAME}` 语法引用环境变量。

---

## 6. 插件生命周期

### 6.1 执行流程

```
┌─────────────────┐
│  系统调度器      │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 1. 参数验证     │  检查目标、配置、权限
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 2. 环境准备     │  加载配置、初始化资源
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 3. 插件执行     │  main.go / main.py 入口
│                 │  - 接收参数（命令行参数）
│                 │  - 执行业务逻辑
│                 │  - 输出进度（stderr）
│                 │  - 输出结果（stdout）
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 4. 结果处理     │  解析JSON、保存数据库
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ 5. 清理资源     │  关闭连接、删除临时文件
└─────────────────┘
```

### 6.2 输入参数

**命令行参数：**

```bash
# 基本形式
./my_plugin <target> [options]

# 示例
./port_scanner 192.168.1.1
./subdomain_finder example.com
./asset_collector "京东集团"
```

**环境变量：**

系统会自动注入以下环境变量：

```bash
HS_SCAN_ID=154d04f43a31056db554919e749a4517     # 扫描任务ID
HS_PROJECT_ID=a88976852b32c0efcefe51e4b5feff1c  # 项目ID
HS_GATEWAY_URL=http://127.0.0.1:8080           # 网关地址
HS_CONFIG_PATH=/path/to/config.json            # 配置文件路径
```

### 6.3 输出规范

**标准输出（stdout）**：JSON格式的结果数据

```json
{
  "status": "success",
  "message": "扫描完成",
  "data": {
    "target": "example.com",
    "results": []
  }
}
```

**标准错误（stderr）**：日志和进度信息

```
INFO: 开始扫描 example.com
INFO: 发现 3 个开放端口
WARN: 端口 22 响应超时
ERROR: 连接被拒绝
```

---

## 7. 数据输出规范

### 7.1 通用输出格式

```json
{
  "status": "success|error|warning",
  "message": "操作描述",
  "data": {
    // 业务数据
  },
  "metadata": {
    "version": "1.0.0",
    "timestamp": "2025-10-06T00:00:00Z",
    "duration": 120.5,
    "target": "example.com"
  }
}
```

### 7.2 扫描类插件输出

```json
{
  "status": "success",
  "message": "端口扫描完成",
  "data": {
    "target": "192.168.1.1",
    "open_ports": [
      {
        "port": 80,
        "protocol": "tcp",
        "service": "http",
        "version": "nginx/1.18.0",
        "banner": "HTTP/1.1 200 OK"
      },
      {
        "port": 443,
        "protocol": "tcp",
        "service": "https",
        "version": "nginx/1.18.0",
        "ssl": {
          "issuer": "Let's Encrypt",
          "expires": "2026-01-01"
        }
      }
    ],
    "closed_ports": 65533,
    "filtered_ports": 0
  },
  "metadata": {
    "scan_type": "syn",
    "duration": 45.2,
    "timestamp": "2025-10-06T12:00:00Z"
  }
}
```

### 7.3 资产收集类插件输出

参考 `asset_collector` 的输出格式：

```json
{
  "status": "success",
  "message": "资产收集完成",
  "data": {
    "enterprise": {
      "name": "京东集团",
      "unified_code": "91110000802940206A",
      "legal_person": "刘强东",
      "establish_date": "2007-04-20",
      // ... 更多企业信息
    },
    "subdomains": [
      {
        "domain": "www.jd.com",
        "ips": ["103.235.46.39"],
        "http_code": 200,
        "title": "京东(JD.COM)-正品低价...",
        "cdn": false
      }
    ],
    "ips": [
      {
        "ip": "103.235.46.39",
        "cdn": false,
        "open_ports": [80, 443],
        "services": ["http", "https"],
        "location": "北京市"
      }
    ],
    "urls": [],
    "crawlers": []
  },
  "metadata": {
    "modules": ["enterprise", "subdomain", "ip", "url", "crawler"],
    "total_assets": 5,
    "processed_assets": 5,
    "duration": 308.5
  }
}
```

### 7.4 错误输出

```json
{
  "status": "error",
  "message": "扫描失败：连接超时",
  "error": {
    "code": "TIMEOUT",
    "details": "连接目标 192.168.1.1:80 超时（30秒）",
    "suggestion": "请检查网络连接或增加超时时间"
  },
  "metadata": {
    "timestamp": "2025-10-06T12:00:00Z"
  }
}
```

---

## 8. 调试与测试

### 8.1 本地调试

**方式1：直接运行**

```bash
cd plugins/my_plugin
go run main.go example.com
```

**方式2：编译后运行**

```bash
go build -o my_plugin main.go
./my_plugin example.com
```

**方式3：使用调试器**

```bash
# VS Code launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Plugin",
      "type": "go",
      "request": "launch",
      "mode": "debug",
      "program": "${workspaceFolder}/main.go",
      "args": ["example.com"]
    }
  ]
}
```

### 8.2 集成测试

**模拟Worker调用：**

```bash
# 设置环境变量
export HS_SCAN_ID=test-scan-001
export HS_GATEWAY_URL=http://localhost:8080

# 运行插件
cd plugins/my_plugin
go run main.go example.com 2>debug.log
```

**查看日志：**

```bash
# 查看进度日志（stderr）
cat debug.log

# 查看结果（stdout）
go run main.go example.com | jq .
```

### 8.3 单元测试

```go
// tests/scanner_test.go
package tests

import (
	"testing"
)

func TestScanner(t *testing.T) {
	target := "example.com"
	result, err := scanner.Scan(target)
	
	if err != nil {
		t.Fatalf("扫描失败: %v", err)
	}
	
	if result.Status != "success" {
		t.Errorf("期望 success，实际 %s", result.Status)
	}
}
```

运行测试：

```bash
go test ./tests/...
```

### 8.4 性能测试

```bash
# 基准测试
go test -bench=. -benchmem

# 压力测试
for i in {1..100}; do
  ./my_plugin example.com &
done
wait
```

---

## 9. 打包与发布

### 9.1 打包前检查

**检查清单：**

- [ ] 所有文件编码为 UTF-8
- [ ] metadata.json 格式正确
- [ ] README.md 完整
- [ ] 删除调试代码和日志
- [ ] 版本号已更新
- [ ] 测试通过
- [ ] 无敏感信息（密码、密钥）

### 9.2 创建 ZIP 包

**自动打包（推荐）：**

```bash
# 在插件目录执行
cd plugins/my_plugin
zip -r my_plugin_v1.0.0.zip . \
  -x "*.git*" \
  -x "output/*" \
  -x "*.log" \
  -x "*.tmp"
```

**包含的文件：**

```
my_plugin_v1.0.0.zip
├── main.go
├── go.mod
├── go.sum
├── metadata.json
├── config.json
├── README.md
├── CHANGELOG.md
├── config/
├── modules/
├── types/
└── utils/
```

**排除的文件：**
- `.git/`
- `output/`
- `*.log`
- `*.tmp`
- 测试数据

### 9.3 上传插件

**方式1：通过Web界面上传**

1. 登录 Hidden Shield
2. 导航到 **配置管理 → 插件管理**
3. 点击 **上传插件**
4. 选择 ZIP 文件
5. 等待上传和验证

**方式2：通过API上传**

```bash
curl -X POST http://localhost:8080/api/plugins/upload \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@my_plugin_v1.0.0.zip"
```

**方式3：手动复制（开发环境）**

```bash
# 复制到系统插件目录
cp -r plugins/my_plugin data/plugins/

# 重启Worker
cd backend && go run ./cmd/worker/
```

### 9.4 版本管理

**语义化版本（SemVer）：**

```
主版本号.次版本号.修订号

1.0.0 → 1.0.1  修复bug
1.0.1 → 1.1.0  新增功能
1.1.0 → 2.0.0  破坏性变更
```

**CHANGELOG.md 示例：**

```markdown
# 更新日志

## [1.2.0] - 2025-10-06

### 新增
- 支持UDP扫描
- 新增服务指纹识别

### 优化
- 扫描速度提升30%
- 减少内存占用

### 修复
- 修复并发竞争条件
- 修复超时不生效的bug

## [1.1.0] - 2025-09-01
...
```

---

## 10. 最佳实践

### 10.1 代码规范

**✅ 推荐：**

```go
// 使用有意义的变量名
scanResults := performScan(target)

// 添加详细注释
// ScanPort 扫描指定端口并返回服务信息
func ScanPort(ip string, port int) (*PortInfo, error) {
	// ...
}

// 错误处理
if err != nil {
	log.Printf("ERROR: 扫描失败 - %v", err)
	return nil, fmt.Errorf("扫描端口 %d 失败: %w", port, err)
}

// 使用常量
const (
	DefaultTimeout = 30 * time.Second
	MaxRetries     = 3
)
```

**❌ 避免：**

```go
// 不要使用单字母变量（除了循环）
r := scan(t)

// 不要忽略错误
scan(target)  // 没有检查错误

// 不要硬编码
time.Sleep(30 * time.Second)  // 应该使用配置
```

### 10.2 性能优化

**1. 并发控制**

```go
// 使用Worker池
const MaxWorkers = 50
sem := make(chan struct{}, MaxWorkers)

for _, target := range targets {
	sem <- struct{}{}
	go func(t string) {
		defer func() { <-sem }()
		scan(t)
	}(target)
}
```

**2. 超时控制**

```go
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()

select {
case result := <-resultChan:
	return result, nil
case <-ctx.Done():
	return nil, fmt.Errorf("操作超时")
}
```

**3. 资源清理**

```go
// 使用defer确保资源释放
conn, err := net.Dial("tcp", addr)
if err != nil {
	return err
}
defer conn.Close()
```

### 10.3 安全最佳实践

**1. 输入验证**

```go
// 验证IP地址
ip := net.ParseIP(input)
if ip == nil {
	return fmt.Errorf("无效的IP地址: %s", input)
}

// 验证域名
if !isValidDomain(domain) {
	return fmt.Errorf("无效的域名: %s", domain)
}
```

**2. 避免路径遍历**

```go
// 错误示例
filepath.Join(baseDir, userInput)  // 用户可以输入 "../../../etc/passwd"

// 正确做法
cleanPath := filepath.Clean(userInput)
if strings.Contains(cleanPath, "..") {
	return fmt.Errorf("非法路径")
}
```

**3. 限制资源使用**

```go
// 限制读取大小
reader := io.LimitReader(resp.Body, 10*1024*1024)  // 最多10MB

// 限制并发数
const MaxConcurrent = 100

// 限制执行时间
ctx, cancel := context.WithTimeout(ctx, 5*time.Minute)
defer cancel()
```

### 10.4 日志规范

**日志级别：**

```go
// INFO: 一般信息
fmt.Fprintf(os.Stderr, "INFO: 开始扫描 %s\n", target)

// WARN: 警告（不影响继续执行）
fmt.Fprintf(os.Stderr, "WARN: 端口 %d 响应慢\n", port)

// ERROR: 错误（影响结果）
fmt.Fprintf(os.Stderr, "ERROR: 连接失败 - %v\n", err)

// DEBUG: 调试信息（仅开发环境）
if debugMode {
	fmt.Fprintf(os.Stderr, "DEBUG: 发送数据包 %v\n", packet)
}
```

**日志格式：**

```
[时间戳] [级别] [模块] 消息

2025-10-06 12:00:00 INFO [scanner] 开始扫描 example.com
2025-10-06 12:00:05 WARN [scanner] 端口 22 超时
2025-10-06 12:00:10 ERROR [scanner] 连接失败
```

### 10.5 配置管理

**使用配置文件而非硬编码：**

```go
type Config struct {
	Timeout    int      `json:"timeout"`
	MaxWorkers int      `json:"maxWorkers"`
	Ports      []int    `json:"ports"`
	Debug      bool     `json:"debug"`
}

func LoadConfig(path string) (*Config, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, err
	}
	
	var cfg Config
	if err := json.Unmarshal(data, &cfg); err != nil {
		return nil, err
	}
	
	return &cfg, nil
}
```

**支持环境变量覆盖：**

```go
timeout := cfg.Timeout
if envTimeout := os.Getenv("SCANNER_TIMEOUT"); envTimeout != "" {
	if t, err := strconv.Atoi(envTimeout); err == nil {
		timeout = t
	}
}
```

---

## 11. 常见问题

### Q1: 插件上传后无法执行？

**可能原因：**
1. `metadata.json` 格式错误
2. `runtime` 或 `entry` 字段配置错误
3. 缺少必要的依赖文件
4. 权限配置不足

**解决方法：**
```bash
# 验证JSON格式
cat metadata.json | jq .

# 检查入口文件
ls -la main.go

# 测试本地运行
go run main.go test-target
```

### Q2: 如何调试插件执行失败？

**查看Worker日志：**

```bash
# 开发环境
cd backend
go run ./cmd/worker/ 2>&1 | tee worker.log

# 生产环境
docker logs -f hidden-shield-worker
```

**查看数据库扫描记录：**

```sql
SELECT * FROM scans WHERE id = 'your-scan-id';
SELECT * FROM plugin_security_scans WHERE origin_scan_id = 'your-scan-id';
```

### Q3: 如何处理大数据量输出？

**分批输出：**

```go
// 不要一次性加载所有数据到内存
// 使用流式处理
encoder := json.NewEncoder(os.Stdout)
for _, result := range scanResults {
	encoder.Encode(result)
}
```

**输出摘要 + 详细文件：**

```go
// stdout 输出摘要
summary := Summary{
	Total: len(results),
	FilePath: "/tmp/detailed_results.json",
}
json.NewEncoder(os.Stdout).Encode(summary)

// 详细结果写入文件
saveDetailedResults("/tmp/detailed_results.json", results)
```

### Q4: 如何实现进度更新？

**通过stderr输出进度：**

```go
total := len(targets)
for i, target := range targets {
	fmt.Fprintf(os.Stderr, "INFO: 进度 %d/%d (%.1f%%)\n", 
		i+1, total, float64(i+1)/float64(total)*100)
	scan(target)
}
```

**通过API更新数据库：**

```go
func updateProgress(scanID string, current, total int) error {
	url := fmt.Sprintf("%s/api/scans/%s/progress", gatewayURL, scanID)
	data := map[string]int{
		"processed_assets": current,
		"total_assets": total,
	}
	// 发送PATCH请求
	return sendPatch(url, data)
}
```

### Q5: Python/Node.js 插件如何打包依赖？

**Python：**

```bash
# 安装依赖到本地目录
pip install -r requirements.txt -t ./vendor

# 在代码中添加路径
import sys
sys.path.insert(0, './vendor')
```

**Node.js：**

```bash
# 打包时包含 node_modules
zip -r plugin.zip . -x "*.git*"
```

### Q6: 如何处理API限流？

**实现指数退避：**

```go
func callAPIWithRetry(url string, maxRetries int) (*Response, error) {
	for i := 0; i < maxRetries; i++ {
		resp, err := http.Get(url)
		if err != nil {
			return nil, err
		}
		
		if resp.StatusCode == 429 {  // Too Many Requests
			waitTime := time.Duration(math.Pow(2, float64(i))) * time.Second
			fmt.Fprintf(os.Stderr, "WARN: API限流，等待 %v 后重试\n", waitTime)
			time.Sleep(waitTime)
			continue
		}
		
		return resp, nil
	}
	return nil, fmt.Errorf("达到最大重试次数")
}
```

### Q7: 如何与其他插件共享数据？

**方式1：通过数据库**

```go
// 读取其他插件的结果
var subdomains []string
db.Query("SELECT domain FROM asset_subdomains WHERE enterprise_id = ?", enterpriseID)
```

**方式2：通过文件系统**

```go
// 写入共享数据
os.WriteFile("/tmp/shared_data.json", data, 0644)

// 其他插件读取
data, _ := os.ReadFile("/tmp/shared_data.json")
```

**方式3：通过Gateway API**

```go
// 调用Gateway API获取数据
resp, _ := http.Get(gatewayURL + "/api/asset-collection/enterprises/" + enterpriseID)
```

---

## 📖 参考资源

### 官方文档
- [Hidden Shield 官方文档](https://docs.hidden-shield.com)
- [插件API参考](https://docs.hidden-shield.com/api)
- [示例插件仓库](https://github.com/hidden-shield/plugins)

### 开发工具
- [插件验证工具](https://github.com/hidden-shield/plugin-validator)
- [插件模板生成器](https://github.com/hidden-shield/plugin-generator)
- [插件市场](https://market.hidden-shield.com)

### 社区
- [开发者论坛](https://forum.hidden-shield.com)
- [GitHub Discussions](https://github.com/hidden-shield/hidden-shield/discussions)
- [技术支持](mailto:support@hidden-shield.com)

---

## 📝 附录

### A. 完整的Go插件模板

参见：[templates/go-plugin-template.zip](../templates/go-plugin-template.zip)

### B. 完整的Python插件模板

参见：[templates/python-plugin-template.zip](../templates/python-plugin-template.zip)

### C. metadata.json JSON Schema

参见：[schemas/metadata.schema.json](../schemas/metadata.schema.json)

---

**文档版本**：v1.0.0  
**最后更新**：2025-10-06  
**贡献者**：Hidden Shield Team

如有问题或建议，请通过 [GitHub Issues](https://github.com/hidden-shield/hidden-shield/issues) 反馈。

