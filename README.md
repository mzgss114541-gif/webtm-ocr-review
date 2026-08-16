# WebTiebaManager OCR Review 插件 (ocr_review)

**本插件是 [WebTiebaManager](https://github.com/TiebaMeow/WebTiebaManager)（贴吧吧务管理工具）的插件**，放置于其 `plugins/` 目录。

[WebTiebaManager](https://github.com/TiebaMeow/WebTiebaManager) (WTM) 的插件：对贴吧帖子中的**图片自动进行 OCR 识别**，当图片文字命中关键词时触发规则处理（删除/确认/忽略）。

- 🖼️ 帖子带图 → 自动下载图片 → OCR 识别文字 → 匹配关键词
- 🔑 在 WTM 规则编辑器中注册自定义条件 `ImageOCRHit`
- 💾 结果按 PID 缓存，避免重复 OCR
- 🛠 管理页 `/ocr-review`：查看识别结果、手动扫描、配置
- 🔒 安全：仅下载白名单域名图片、拦截内网地址、敏感信息脱敏
- 🧠 本地 OCR 引擎（RapidOCR），免费离线，图片不泄露给第三方

## 功能

- 自动处理 WTM 爬到的带图帖子，OCR 图片文字
- 规则条件 `ImageOCRHit`：图片文字命中关键词 → 执行规则操作
- 管理页查看每个帖子的 OCR 结果（含图片预览、识别文本）
- 手动扫描调试（按 PID）
- 自动缓存 + 自动清理（上限 500 条）

## 安装

1. 安装 OCR 依赖：

```bash
pip install rapidocr_onnxruntime
```

> **⚠️ 无头服务器（无桌面环境/图形库）注意**：
>
> RapidOCR 依赖 `opencv-python`，而默认的 `opencv-python` 需要系统图形库（`libGL.so.1`）。在无头服务器（SSH 服务器、Docker 容器、云主机）上安装或运行时，会报错：
>
> ```
> ImportError: libGL.so.1: cannot open shared object file: No such file or directory
> ```
>
> **解决方法**：用无头版本代替默认版本：
>
> ```bash
> pip uninstall opencv-python -y 2>/dev/null || true
> pip install opencv-python-headless
> ```
>
> `opencv-python-headless` 不依赖图形库，功能对 OCR 完全够用。
>
> 如果你的服务器有桌面环境（本地装了显卡/图形驱动），默认 `opencv-python` 即可，无需替换。

2. 将 `ocr_review.py` 放入 WTM 的 `plugins/` 目录：

```bash
cp ocr_review.py /path/to/WebTiebaManager/plugins/
chown <wtm-user>:<wtm-user> /path/to/WebTiebaManager/plugins/ocr_review.py
```

3. 重启 WTM：

```bash
systemctl restart webtieba   # 或你的 WTM 启动方式
```

4. 浏览器打开管理页：

```
http://<wtm-host>:36799/ocr-review
```

## 使用

1. 在 WTM 的规则编辑器中，新建规则
2. 添加条件，类型选择 `ImageOCRHit`（图片OCR命中）
3. 在条件文本里输入要匹配的关键词
4. 配置规则操作（删除/确认/忽略，与普通规则一致）

帖子带图片且 OCR 文字命中关键词时，规则自动触发。

## 工作原理

- 插件在 WTM 规则系统中注册自定义条件类型 `ImageOCRHit`
- 处理帖子时自动下载图片（仅限允许的图床域名，拦截内网/SSRF）
- 用 RapidOCR（本地 ONNX 推理）识别图片文字
- 结果与关键词匹配，返回命中/未命中，交由 WTM 规则引擎处理

## 安全性

- **API 需登录认证**：所有管理 API（查看结果、手动扫描、配置、清缓存）都依赖 WTM 的登录认证（`current_user_depends`），未登录返回 `401`
- **SSRF 防护**：仅下载白名单图床域名（百度系），拦截内网/保留地址（127.0.0.0/8、10.0.0.0/8、172.16.0.0/12 等），防 SSRF
- **大小限制**：单图 ≤10MB，防止恶意大图消耗资源
- **日志脱敏**：bduss、stoken、Cookie 等敏感 token 在日志中打码
- **本地 OCR**：图片仅在本地处理，不上传第三方

## 许可证

[MIT](LICENSE)

## 关于本项目

本项目由 [mzgss114541-gif] 主导设计与开发，部分编码环节使用 **DeepSeek Harness**（DSH，AI agent 框架）作为辅助。

代码经过实际生产环境验证，但建议使用前 review 代码、按需调整。

**Contributors**

- [mzgss114541-gif](https://github.com/mzgss114541-gif) — 设计、开发、测试、部署
- [DeepSeek Harness](https://github.com/deepseek-ai) — AI agent（辅助实现）
