# 插件开发快速入门

⏱️ **10分钟快速上手**

---

## 步骤1：复制模板（1分钟）

```bash
# 进入插件开发目录
cd plugins/

# 复制Go插件模板
cp -r ../docs/templates/go-plugin-template my_scanner

# 进入项目
cd my_scanner
```

---

## 步骤2：修改元数据（2分钟）

编辑 `metadata.json`：

```json
{
  "name": "我的扫描器",
  "key": "my_scanner",
  "category": "scan",
  "version": "1.0.0",
  "author": "你的名字 <your@email.com>",
  "description": "一个简单的端口扫描器",
  "tags": ["扫描", "端口"],
  "runtime": "go",
  "entry": "main.go"
}
```

---

## 步骤3：实现业务逻辑（5分钟）

编辑 `main.go`，在 `performScan` 函数中添加你的代码：

```go
func performScan(target string, config *Config) (map[string]interface{}, error) {
	// 1. 解析目标
	logInfo(fmt.Sprintf("解析目标: %s", target))
	
	// 2. 执行扫描
	logInfo("开始扫描...")
	openPorts := []int{}
	
	// 示例：扫描常见端口
	commonPorts := []int{80, 443, 22, 21, 3306, 6379, 27017}
	for _, port := range commonPorts {
		if isPortOpen(target, port) {
			openPorts = append(openPorts, port)
			logInfo(fmt.Sprintf("发现开放端口: %d", port))
		}
	}
	
	// 3. 返回结果
	results := map[string]interface{}{
		"target":     target,
		"open_ports": openPorts,
		"total":      len(openPorts),
	}
	
	return results, nil
}

// 简单的端口检测函数
func isPortOpen(host string, port int) bool {
	addr := fmt.Sprintf("%s:%d", host, port)
	conn, err := net.DialTimeout("tcp", addr, 2*time.Second)
	if err != nil {
		return false
	}
	conn.Close()
	return true
}
```

记得添加 `import "net"`！

---

## 步骤4：测试运行（1分钟）

```bash
# 初始化依赖
go mod tidy

# 运行测试
go run main.go example.com

# 查看输出
go run main.go example.com | jq .
```

**预期输出：**

```json
{
  "status": "success",
  "message": "扫描成功",
  "data": {
    "target": "example.com",
    "open_ports": [80, 443],
    "total": 2
  },
  "metadata": {
    "version": "1.0.0",
    "timestamp": "2025-10-06T12:00:00Z",
    "duration": 5.2,
    "target": "example.com"
  }
}
```

---

## 步骤5：打包上传（1分钟）

```bash
# 打包
zip -r my_scanner_v1.0.0.zip . -x "*.git*" -x "output/*"

# 通过Web界面上传
# 1. 登录 Hidden Shield
# 2. 导航到"配置管理 → 插件管理"
# 3. 点击"上传插件"
# 4. 选择 my_scanner_v1.0.0.zip
```

---

## ✅ 完成！

现在你可以在系统中创建扫描任务，选择你的插件来执行扫描了！

---

## 🚀 下一步

### 进阶功能

1. **添加配置支持**
   - 编辑 `config.json`
   - 在代码中读取配置参数

2. **并发扫描**
   - 使用 goroutine 提高性能
   - 添加 Worker 池控制并发数

3. **进度报告**
   - 通过 stderr 输出进度信息
   - 更新数据库进度字段

4. **错误处理**
   - 添加重试机制
   - 输出详细的错误信息

### 学习资源

- 📖 [完整开发指南](./PLUGIN_DEVELOPMENT_GUIDE.md)
- 🔧 [插件API文档](https://docs.hidden-shield.com/api)
- 💡 [示例插件库](https://github.com/hidden-shield/plugins)
- 🤝 [开发者社区](https://forum.hidden-shield.com)

---

**遇到问题？**

- 查看 [常见问题](./PLUGIN_DEVELOPMENT_GUIDE.md#11-常见问题)
- 提交 [GitHub Issue](https://github.com/hidden-shield/hidden-shield/issues)
- 联系技术支持 support@hidden-shield.com

