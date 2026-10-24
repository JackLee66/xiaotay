# 小tay陪学 · 考研督学（v2）

一个纯网页的考研打卡 App：一个角色陪你学习，你完成任务它夸你，日历记录你学过多少天。

## 手机上怎么打开

1. 电脑上保持服务在跑（见下）。
2. 手机连同一个 Wi-Fi，浏览器地址栏**手动输入**：
   `http://192.168.1.8:8080`
   必须带 `http://`。有些浏览器会自动把地址升级成 https，那样会打不开（服务器只讲 http）。
3. 想当 App 用：Chrome 菜单 →「添加到主屏幕」；iPhone 用 Safari 分享 →「添加到主屏幕」。

电脑上启动服务（关掉窗口就停了）：

```
python -m http.server 8080 --bind 0.0.0.0 --directory "C:\Users\admin\Documents\deepseek-harness\default-workspace\xiaoyou"
```

## 现在有什么

**首页**
- 顶部：考研倒计时 D-xxx、连续天数
- 角色 + 台词气泡
- 今日进度环（完成 x / 4）
- 今日任务：点一下打卡，变粉、角色换成开心表情并说一句夸你的话；长按可以取消
- 全部完成 → 庆祝弹窗

**日历页**
- 最上面：距离 2028 考研初试还有多少天（考试日 2027-12-18）
- 三个数：累计学习天数 / 连续签到 / 当月打卡
- 月历：完成的当天自动标色（浅粉=做了一部分，深粉=全部完成）；只签到没做任务=淡粉
- 「今日签到」按钮：只是想打个卡也能记一天
- 左右箭头翻月份，点某一天能看到当天完成了多少

**设置**（右上角齿轮）：改称呼、改考试日期、增删任务、清空数据。

## 数据

全部存在浏览器 localStorage 里（key `xiaotay.v2`），不上传、不联网、换设备不同步。
清浏览器数据 / 换手机 = 记录没了。

## 文件

| 文件 | 作用 |
| --- | --- |
| `index.html` | 整个 App（HTML + CSS + JS + 角色 SVG 都在这一个文件里） |
| `manifest.json` | 加到桌面的图标/名称 |
| `sw.js` | 离线缓存（只在 https 或 localhost 生效） |
| `icon-*.png` | 图标 |
| `_dev/` | 开发用的截图脚本、测试页，可以忽略 |

## 想改的地方在哪

- 考试日期：`index.html` 里 `const EXAM = '2027-12-18';`
- 默认任务：`const DEFAULT_TASKS = [...]`（也可以在设置里改）
- 台词库：`const LINES = {...}`（greet / praise / allDone / idle / comeback …）
- 角色立绘：`const MAID_SVG = \`...\``，换成图片就是把这段 SVG 替换成 `<img>`
- 主题色：CSS 顶部的 `:root` 变量

## 下一步（还没做）

- 换成你自己的表情包立绘（需要 8 张：平淡 / 笑 / 兴奋 / 委屈 / 困 / 专注 / 问好 / 久别）
- 数据云同步（要后端）
- 提醒推送（网页版在 iOS 上不可靠）

## 表情包组（stickers/）

`
stickers/
  normal/   1.png  2.png  3.png  ...
  happy/    1.png  2.png  ...
  excited/  1.png  ...
  sleepy/   sad/   shy/   focus/   hello/
`

- 每种情绪一个文件夹，图片按 **1.png、2.png、3.png…** 数字命名（每组最多 16 张）。
- 每次切到某个情绪，从该组随机挑一张，**连续两次不重复**。
- 同一情绪 **2.5 秒内不重新抽**，避免连点时闪图。
- 某组空 → 用 normal 组；normal 也空 → 显示内置手绘女仆。

情绪触发（当前）：

- **normal** —— 打开 App 的默认状态
- **sleepy** —— 晚上 22:00 到早上 6:00
- **happy** —— 这次打开之后完成了任意一件，会话内一直保持
- **excited** —— 这次打开之后全部完成
- **sad / shy / focus / hello** —— 目录预留，还没接

表情**不写进存储**：关掉重开一定回到 normal（或当前时段对应的 sleepy）。
同一组里有几张就随机几张，连续两次不会重复。

## 台词库（LINES）

全部台词写在 `index.html` 的 `const LINES = { ... }` 里，现在约 130 条，按情绪分组：

| 组 | 什么时候说 |
| --- | --- |
| `greet.morning / noon / evening / night` | 每天第一次打开，按时间段选一组 |
| `praise.t_en / t_math / t_pol / t_major / generic` | 完成对应任务时（自定义任务走 generic） |
| `allDone` | 四件全做完 |
| `halfway` | 正好做完一半 |
| `lateNight` | 23 点后完成一件，紧跟一句 |
| `undo` | 长按撤销打卡 |
| `signIn` | 签到成功 |
| `streak` | 连续签到 7 / 14 / 30 / 60 / 100 / 200 天，`{n}` 是占位符 |
| `examSprint` | 倒计时 ≤ 7 天，打开时替代问候 |
| `examDay` | 考试当天 |
| `comeback` | 隔 3 天以上没来 |
| `lonely` | 久别回来时同时切到的表情 |
| `idle` | 发呆时每隔 45-90 秒冒一句 |

改台词直接改这些数组，加几条删几条都不影响逻辑，`pick()` 会自动随机。

## 可替换的图片位一览

| # | 位置 | 路径 | 建议尺寸 | 空着时显示 |
| --- | --- | --- | --- | --- |
| 1 | 角色表情包 | stickers/<情绪>/1.png…16.png | 长边 1000-1200 | 内置手绘女仆 |
| 2 | 任务卡图标 | art/task-t_en.png 等 | 256x256 | 任务的 emoji |
| 3 | 任务卡通用图标 | art/task-generic.png | 256x256 | 上面的 emoji |
| 4 | 底栏图标 | art/tab-home.png、art/tab-cal.png | 128x128 | emoji 🏠 📅 |
| 5 | 顶部连续天数小火苗 | art/flame.png | 64x64 | emoji 🔥 |
| 6 | 全部完成弹窗顶图 | art/party.png | 256x256 | emoji 🎉 |
| 7 | 手机桌面图标 | icon-192.png、icon-512.png、icon-180.png | 192/512/180 | 生成好的 tay 图标 |
| 8 | 日历页签到按钮里的头像 | art/sign.png | 160x160 圆形 | 按钮上没有头像 |

任务 id：t_en 英语单词 / t_math 数学 / t_pol 政治 / t_major 专业课。
自定义任务没有专属 id，统一用 art/task-generic.png。

所有图片位都缺图自动回落，不会出现破图。透明度 PNG 最佳。

> 台词和表情都只影响显示，不参与任何计算。改坏了刷新一下就回来了。
