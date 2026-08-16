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
