# elcton — 华为轻智能手表课程表（HarmonyOS Lite Wearable）

FIT3 / GT 系列轻智能手表上的课程表应用，附**取件码云同步服务端**与**一键 CI 出包流水线**——push 代码，GitHub Actions 直接产出可安装包。

技术栈：HML + CSS + JavaScript（JerryScript Lite ES6 子集），纯 FA 工程，零第三方依赖。

## 功能

- **今日课表**：按节次时间轴展示，当前课 / 已上 / 未上状态实时高亮
- **周视图**：整周网格 + 单 / 双周筛选（按开学日期自动计算当前周）
- **课程管理**：详情页增删改（名称 / 老师 / 教室 / 节次 / 起止时间 / 周次 / 单双周 / 颜色）
- **取件码云同步**：手表输入 6 位取件码拉取课表；同步页内置**分步连通自检**（逐跳探测手表 → 服务端链路），断网点「重跑自检」即恢复
- **本地持久化**：拉取即存，离线可用

## 设备与分支

| 分支 | 形态 | 屏幕 | 说明 |
| --- | --- | --- | --- |
| `master` | 方屏 | FIT3 412×484 | 主力版本 |
| `circle-screen` | 圆屏 | GT 466×466 / 390×390 | 头栏、按钮、列表按圆屏安全区几何重排 |

## 仓库结构

```
entry/                     # 主工程（5 页：index 今日 / week 周视图 / detail 课程编辑 / sync 同步 / code 取件码）
web/                       # 取件码服务端（PHP，无数据库依赖）
  ├─ api.php               #   多用户版：取件码只读拉取 / did 免密识别 / 编辑密码换设备解锁
  └─ save.php              #   单 token 简版 + 网页编辑器（index.html / kebiao-index.html）
scripts/
  └─ clean_config_in_app.py  # 构建后 config.json Lite schema 清洗
tools/
  ├─ lan-server.py         # 内网联调本地服务
  └─ sign/                 # 纯 JS HAP 签名器 + verify_sign.py 密码学验签闸门
.github/workflows/build.yml  # CI：CLT 6.1.1.280 构建 →（可选）签名 → 验签 → artifact
```

## 快速开始

1. fork 后 push 或手动触发 `workflow_dispatch`，CI 产物在 artifact `elcton-<run>`：
   - `elcton-default-unsigned.app` —— 未签名包（**零配置即可产出**）
2. 想要签名可直接装表的 `.hap`：
   - 在 [AGC](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html) 用你自己的 CSR 创建调试证书，下载 `.p7b` 放到 `certs/app.p7b`（已被 gitignore，勿提交）
   - 仓库 Secret `SIGN_P12_PEM` 存私钥 PEM（约定见 `tools/sign/sign-app-to-hap.js` 的 loadSignKey）
   - CI 自动签名，并用**密码学验签闸门**确认私钥与证书配对——不配对直接构建红、不出包，从根上杜绝"错误 30"装机失败
3. 通过华为穿戴 App 安装到手表。证书身份会写入包内（`FORCE_P7B_BUNDLE=1`）：**一个 p7b = 一个安装身份**，不同应用各备一张证书即可并存。

## 云同步部署

任意 PHP 空间部署 `web/` 即可，首次运行自动建 `data/` 目录。权限模型：

- **取件码**（6 位数字）：只读，手表用它拉课表
- **编辑密码**（6 位字母数字）：换设备时解锁，解锁后该设备自动登记为拥有者
- **did**（32 位设备标识）：本人回访免密直接进入

`$TOKEN` / 前端 `TOKEN` 默认 `CHANGE_ME` 占位，真实值走主机上的配置文件，**不入库**。

## 适配要点（实测结论，改布局前先读）

- **圆屏安全区**：行内容必须落在该行**行顶 y** 处的圆内安全宽内（`half = sqrt(R²−(R−y)²)`，行顶最窄）；头栏用居中收紧而非左右分列，卡片宽度收到 80%
- **背景 / 圆角 / 点击挂 div，text 只放字**——背景挂 text 会真机文字错位丢失
- **容器给死宽高**——自适应宽时父靠子量宽、子靠父量宽，引擎解成 0 宽导致空字
- css 编译内联进页面 js 的 `_styleSheet`，产物体检直接 grep js

## 相关项目

- ⭐ **[lite-watch-starter](https://github.com/lantianhcgp/lite-watch-starter)** —— Agent-ready HarmonyOS Lite Wearable 应用模板：一键 CI 构建 + 签名流水线，AI agent 读 README 即可接管全流程。想从零做自己的 Lite 手表应用，先看它。

## License

[MIT](LICENSE)。`tools/sign/` 签名器源自 [kqakqakqa/hap-sign-utils](https://github.com/kqakqakqa/hap-sign-utils)（MIT）。
