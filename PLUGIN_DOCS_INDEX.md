# Hidden Shield 插件开发文档索引

> 📚 完整的插件开发资源库

---

## 🎯 新手入门

### 1. [快速入门指南](./PLUGIN_QUICK_START.md) ⚡
**⏱️ 10分钟快速上手**

从零开始，10分钟创建你的第一个插件。包含：
- 模板复制
- 代码编写
- 测试运行
- 打包上传

**适合人群**：初次接触插件开发

---

### 2. [完整开发指南](./PLUGIN_DEVELOPMENT_GUIDE.md) 📖
**📚 全面系统的开发文档**

包含11个主要章节：
1. 插件系统概述
2. 开发环境准备
3. 快速开始
4. 插件项目结构
5. 配置文件详解
6. 插件生命周期
7. 数据输出规范
8. 调试与测试
9. 打包与发布
10. 最佳实践
11. 常见问题

**适合人群**：需要深入了解插件系统的开发者

---

## 🛠️ 开发资源

### 模板文件

#### Go 插件模板
**路径**: `docs/templates/go-plugin-template/`

包含：
- ✅ `main.go` - 完整的入口程序模板
- ✅ `metadata.json` - 元数据配置
- ✅ `config.json` - 运行时配置
- ✅ `go.mod` - Go模块定义
- ✅ `README.md` - 说明文档

**使用方法**：
```bash
cp -r docs/templates/go-plugin-template plugins/my_plugin
cd plugins/my_plugin
go mod tidy
```

---

#### Python 插件模板
**路径**: `docs/templates/python-plugin-template/`

包含：
- ✅ `main.py` - 完整的入口程序模板
- ✅ `metadata.json` - 元数据配置
- ✅ `config.json` - 运行时配置
- ✅ `requirements.txt` - Python依赖
- ✅ `README.md` - 说明文档

**使用方法**：
```bash
cp -r docs/templates/python-plugin-template plugins/my_plugin
cd plugins/my_plugin
pip install -r requirements.txt
```

---

### 配置规范

#### metadata.json JSON Schema
**路径**: `docs/schemas/metadata.schema.json`

用于验证 `metadata.json` 配置文件的格式正确性。

**在线验证**：
- [JSONSchema Validator](https://www.jsonschemavalidator.net/)
- [JSON Schema Lint](https://jsonschemalint.com/)

**VS Code 集成**：
```json
// .vscode/settings.json
{
  "json.schemas": [
    {
      "fileMatch": ["**/metadata.json"],
      "url": "./docs/schemas/metadata.schema.json"
    }
  ]
}
```

---

## 📚 参考示例

### 官方示例插件

#### 1. 资产收集器（asset_collector）
**路径**: `plugins/asset_collector/`

**功能**：企业资产全流程收集工具

**特点**：
- ✅ 主+子模块架构
- ✅ 5个功能模块联动
- ✅ 完整的配置管理
- ✅ 详细的进度报告
- ✅ 结构化数据输出

**学习重点**：
- 模块化设计
- 并发处理
- API集成
- 数据提取与清洗

**文档**：
- [README.md](../plugins/asset_collector/README.md)
- [WORKFLOW.md](../plugins/asset_collector/WORKFLOW.md)

---

#### 2. 端口扫描器（port_scanner）
**路径**: `data/plugins/port_scanner/`

**功能**：高性能TCP/UDP端口扫描

**特点**：
- ✅ SYN/Connect扫描
- ✅ 服务识别
- ✅ Banner抓取
- ✅ 并发优化

**学习重点**：
- 网络编程
- 性能优化
- 超时控制
- 错误处理

---

## 🔧 开发工具

### 推荐IDE配置

#### VS Code
推荐扩展：
- Go (Go Team at Google)
- Python (Microsoft)
- JSON (ZainChen.json)
- Error Lens (Alexander)

#### GoLand / PyCharm
JetBrains全家桶，专业IDE。

---

### 调试技巧

#### 本地调试
```bash
# Go插件
cd plugins/my_plugin
go run main.go example.com 2>debug.log

# Python插件
cd plugins/my_plugin
python main.py example.com 2>debug.log
```

#### 查看日志
```bash
# 查看标准错误（进度日志）
cat debug.log

# 查看标准输出（结果数据）
go run main.go example.com | jq .
```

#### 集成测试
```bash
# 模拟Worker环境
export HS_SCAN_ID=test-001
export HS_GATEWAY_URL=http://localhost:8080
cd backend && go run ./cmd/worker/
```

---

## 📦 打包与发布

### 打包命令

**Go插件**：
```bash
cd plugins/my_plugin
zip -r my_plugin_v1.0.0.zip . \
  -x "*.git*" \
  -x "output/*" \
  -x "*.log"
```

**Python插件**：
```bash
cd plugins/my_plugin
zip -r my_plugin_v1.0.0.zip . \
  -x "*.git*" \
  -x "__pycache__/*" \
  -x "*.pyc" \
  -x "output/*"
```

### 上传方式

1. **Web界面**：配置管理 → 插件管理 → 上传插件
2. **API接口**：`POST /api/plugins/upload`
3. **手动复制**：`cp -r plugins/my_plugin data/plugins/`

---

## 🌐 在线资源

### 官方网站
- [主页](https://hidden-shield.com)
- [文档中心](https://docs.hidden-shield.com)
- [API文档](https://docs.hidden-shield.com/api)

### 社区
- [GitHub仓库](https://github.com/hidden-shield/hidden-shield)
- [开发者论坛](https://forum.hidden-shield.com)
- [技术博客](https://blog.hidden-shield.com)

### 插件市场
- [官方插件市场](https://market.hidden-shield.com)
- [第三方插件列表](https://github.com/hidden-shield/awesome-plugins)

---

## ❓ 获取帮助

### 常见问题
参见 [完整开发指南 - 第11章](./PLUGIN_DEVELOPMENT_GUIDE.md#11-常见问题)

### 提交问题
- [GitHub Issues](https://github.com/hidden-shield/hidden-shield/issues)
- [社区论坛](https://forum.hidden-shield.com)

### 技术支持
- 📧 Email: support@hidden-shield.com
- 💬 WeChat: hidden-shield-support
- 📞 Phone: 400-XXX-XXXX (工作日 9:00-18:00)

---

## 🤝 贡献指南

欢迎贡献你的插件和文档！

### 贡献插件
1. Fork 仓库
2. 创建分支 `feature/my-plugin`
3. 开发并测试你的插件
4. 提交 Pull Request

### 贡献文档
1. 发现文档错误或改进点
2. 创建分支 `docs/improvement`
3. 修改文档
4. 提交 Pull Request

### 行为准则
- ✅ 遵守开源协议
- ✅ 尊重社区成员
- ✅ 提供高质量的代码和文档
- ✅ 积极参与讨论

---

## 📄 许可证

Hidden Shield 采用 [MIT License](../LICENSE)

您可以自由地：
- ✅ 使用
- ✅ 复制
- ✅ 修改
- ✅ 分发

---

**祝你开发愉快！🚀**

*最后更新：2025-10-06*

