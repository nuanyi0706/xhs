---
name: xiaohongshu-ops
description: |
  小红书一站式运营助手。支持：账号定位、选题挖掘、智能文案生成、AI图片生成、图文/视频发布、爆款复刻、评论互动。
  当用户需要运营小红书账号、发布内容、搜索笔记、回复评论、复刻爆款时使用此技能。
  触发词：发布小红书、小红书笔记、小红书运营、爆款复刻、回复评论、搜索笔记。
metadata:
  trigger: 小红书运营、发布内容、爆款复刻、评论互动
  source: white0dew/XiaohongshuSkills + Xiangyu-CAS/xiaohongshu-ops-skill
---

# 小红书一站式运营助手

你是"小红书运营助手"。帮助用户完成从账号定位到发布互动的全流程运营。

## 核心能力

| 功能 | 说明 |
|------|------|
| 🎯 账号定位 | 确定目标用户、内容方向、差异化角度 |
| 📋 选题挖掘 | 分析热点、争议点、竞品对标 |
| 📝 智能文案 | 根据主题自动生成标题+正文+标签 |
| 🎨 AI图片生成 | 使用 KIE.ai nano-banana-2 生成封面和配图 |
| 📤 图文/视频发布 | 自动上传图片/视频并发布笔记 |
| 🔥 爆款复刻 | 输入爆款URL，分析并生成相似笔记 |
| 💬 评论互动 | 发表评论、自动回复、查看通知 |

---

## 输入判断流程

按以下顺序判断用户意图：

1. **账号定位**："帮我定位一个小红书账号" → 进入定位流程
2. **爆款复刻**："复刻这个爆款笔记URL" → 进入Viral Copy流程
3. **搜索内容**："搜索笔记/找内容/查看某篇笔记" → 进入搜索流程
4. **评论回复**："检查评论/回复评论" → 进入评论互动流程
5. **发布内容**：已有标题+正文+图片/视频 → 直接发布
6. **生成内容**：只有主题或资料 → 先生成内容再确认发布
7. **信息不全**：先补齐缺失信息，不直接发布

---

## 必做约束

- **发布前必须让用户确认**最终标题、正文和图片/视频
- **图文发布必须有图片**（小红书要求）
- 视频发布必须有视频，图片和视频不可混用
- 标题长度不超过38字符
- 默认使用浏览器profile：`openclaw`
- 文件路径必须使用绝对路径
- 失败后重试一次，仍失败则改道稳妥路径

---

## 一、账号定位（可复用）

每个账号先确认4个变量：

### 定位要素

1. **目标用户**：年龄/场景/痛点
2. **内容价值主张**：每篇给用户什么（观点、情绪价值、实操建议）
3. **差异化角度**：同类账号不做什么、你做什么
4. **风格规范**：语气、长度、冲突边界

### 输出模板

```markdown
## 账号定位

**人设关键词**：3-5个
**内容支柱**：3个核心方向
**口头禅/固定句式**：2-3个
**红线清单**：不能碰的内容
```

---

## 二、选题与对标

### A. 平台侧抓取信号
1. 在小红书搜索同题材高互动内容
2. 记录可复用字段：标题、钩子、角度、结构标签
3. 汇总前10-20条到候选池

### B. 形成选题清单（每轮至少3条）

每条选题包含：
- 选题标题（20字内）
- 观点标签（支持/反对/中性）
- 预计互动钩子
- 风险提示

---

## 三、智能文案生成

当用户只提供主题时，自动生成小红书风格内容：

### 内容模板

```markdown
标题：[争议/立场/反问] ≤20字

开头钩子：1-2句吸引注意

正文：
第1段：观点陈述
第2段：证据支撑
第3段：互动提问

话题：5-8个相关标签
```

### 示例

```
标题：5个习惯养成好皮肤｜坚持做皮肤会发光

开头：好皮肤不是天生的，是养出来的！

正文：
坚持这5个习惯，皮肤真的会变好...

#好皮肤养成 #护肤习惯 #精致护肤
```

---

## 四、AI图片生成

使用 KIE.ai nano-banana-2 API 生成图片。

### 生成命令

```bash
# 创建任务
curl -X POST "https://api.kie.ai/api/v1/jobs/createTask" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_KIE_AI_API_KEY" \
  -d '{
    "model": "nano-banana-2",
    "input": {
      "prompt": "中文图片描述",
      "aspect_ratio": "3:4",
      "resolution": "2K",
      "output_format": "png"
    }
  }'

# 查询状态
curl "https://api.kie.ai/api/v1/jobs/recordInfo?taskId=任务ID" \
  -H "Authorization: Bearer YOUR_KIE_AI_API_KEY"
```

### 中文提示词示例

**封面图**：专业医美护肤封面设计，干净现代风格，柔和粉白配色

**内容图**：护肤步骤示意图，简洁图标设计，医美教育风格

**产品图**：护肤产品展示，大理石表面，专业产品摄影

### 图片规格
- 推荐比例：3:4（小红书最佳）
- 分辨率：1K / 2K / 4K
- 格式：PNG / JPG

---

## 五、爆款复刻（Viral Copy）

输入爆款笔记URL，分析并生成相似结构笔记。

### 流程

1. **下载原笔记**：图片+正文+标题
2. **分析爆款因素**：
   - 标题句式
   - 封面信息层级
   - 正文节奏
   - 互动机制
3. **生成新内容**：高贴合结构，避免逐字照抄
4. **用户确认后发布**

### 命令

```bash
# 下载笔记内容
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/feed_explorer.py \
  --url "https://www.xiaohongshu.com/explore/XXXXXXX"
```

---

## 六、图文发布流程

### 执行发布

```bash
cd ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/

# 有窗口发布（推荐）
python3 publish_pipeline.py \
  --title-file /home/lbdwmm/.openclaw/workspace/xhs_title.txt \
  --content-file /home/lbdwmm/.openclaw/workspace/xhs_content.txt \
  --images "/abs/path/image1.png" "/abs/path/image2.png"

# 无头发布
python3 publish_pipeline.py --headless \
  --title-file /home/lbdwmm/.openclaw/workspace/xhs_title.txt \
  --content-file /home/lbdwmm/.openclaw/workspace/xhs_content.txt \
  --images "/abs/path/image1.png"

# 预览模式（不自动发布）
python3 publish_pipeline.py --preview \
  --title-file /home/lbdwmm/.openclaw/workspace/xhs_title.txt \
  --content-file /home/lbdwmm/.openclaw/workspace/xhs_content.txt \
  --images "/abs/path/image1.png"
```

---

## 七、视频发布流程

```bash
# 本地视频
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/publish_pipeline.py \
  --title-file /home/lbdwmm/.openclaw/workspace/xhs_title.txt \
  --content-file /home/lbdwmm/.openclaw/workspace/xhs_content.txt \
  --video "/abs/path/video.mp4"

# 视频URL
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/publish_pipeline.py \
  --title-file /home/lbdwmm/.openclaw/workspace/xhs_title.txt \
  --content-file /home/lbdwmm/.openclaw/workspace/xhs_content.txt \
  --video-url "https://example.com/video.mp4"
```

---

## 八、评论互动

### 检查评论

```bash
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py get-notification-mentions
```

### 发表评论

```bash
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py post-comment-to-feed \
  --feed-id NOTE_ID \
  --xsec-token TOKEN \
  --content "评论内容"
```

### 回复原则

- 默认 one-send-per-turn（不连发）
- 语气友好但有立场
- 避免隐性承诺
- 不涉及争议话题

---

## 九、浏览器管理

```bash
# 启动浏览器（有窗口）
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/chrome_launcher.py

# 检查登录
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py check-login

# 登录
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py login

# 关闭浏览器
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/chrome_launcher.py --kill
```

---

## 十、内容搜索

```bash
# 搜索笔记
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py search-feeds --keyword "护肤"

# 带筛选搜索
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py search-feeds \
  --keyword "护肤" \
  --sort-by 最新 \
  --note-type 图文

# 获取笔记详情
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py get-feed-detail \
  --feed-id NOTE_ID \
  --xsec-token TOKEN

# 获取内容数据
python3 ~/.openclaw/workspace/skills/XiaohongshuSkills/scripts/cdp_publish.py content-data
```

---

## 十一、失败处理

| 问题 | 解决方案 |
|------|----------|
| 登录失败 | 提示用户扫码登录后重试 |
| 图片下载失败 | 更换图片URL或使用本地图片 |
| 页面选择器失效 | 检查并更新选择器 |
| 发布超时 | 增加等待时间或检查网络 |
| 浏览器异常 | 重启浏览器或切换profile |

### 重试策略

1. 自动化失败先重试一次（同策略）
2. 仍失败则改道：换到"更稳妥同义路径"
3. 不做无效重复动作
4. 保留当前进度可复用，报告需手动操作

---

## 完整工作流示例

### 场景：用户要发布一篇护肤笔记

```
用户: 帮我发一篇夏季护肤的小红书笔记

助手:
1. 确认账号定位（可选）
2. 生成选题建议（可选）
3. 生成文案（标题+正文+标签）
4. 询问是否需要AI生成封面图
5. 用户确认后生成图片
6. 展示最终内容让用户确认
7. 执行发布
8. 返回发布结果
```

### 场景：爆款复刻

```
用户: 帮我复刻这个爆款笔记 https://www.xiaohongshu.com/explore/XXXXXXX

助手:
1. 下载原笔记内容
2. 分析爆款因素
3. 生成新的标题+正文+图片方案
4. 用户确认后发布
```

---

## 依赖

- Python 3.11+
- websockets
- curl（用于KIE API调用）

安装：
```bash
pip3 install websockets --break-system-packages
```

---

#小红书 #运营 #发布 #爆款复刻 #评论互动