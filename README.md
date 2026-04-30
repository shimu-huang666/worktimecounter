# 兼职工时与收入面板

> 一款面向兼职工作者的工时追踪与收入计算工具，支持账号登录、云端同步、多设备数据共享。

## 功能概览

| 模块 | 说明 |
|------|------|
| **实时计时** | 秒级刷新当前工作时长、今日收入、今日进度 |
| **周期统计** | 按自定义结算周期统计总工时、总收入、预计收入、日均收入等 |
| **上班日选择** | 日历视图按月分页，点击日期标记/取消上班 |
| **单日时间设置** | 对指定日期单独设置上下班时间（覆盖默认值） |
| **午休扣除** | 支持设置午休时间段，自动从工时中扣除 |
| **默认休息日** | 按星期勾选默认休息日（如周六日） |
| **收入柱状图** | 当前周期内每日收入的可视化柱状图 |
| **账号系统** | 邮箱注册/登录，基于 Supabase Auth |
| **云端同步** | 登录后自动将设置同步至云端，支持多设备访问 |
| **本地缓存** | 未登录时数据保存在 localStorage，离线可用 |

## 技术栈

- **前端**：纯 HTML + CSS + JavaScript，零构建依赖，单文件部署
- **后端/数据库**：[Supabase](https://supabase.com)（PostgreSQL + Auth + REST API）
- **Supabase SDK**：`@supabase/supabase-js@2`（CDN 引入）

## 快速开始

### 1. 前提条件

- 一个 [Supabase](https://supabase.com) 账号（免费套餐即可）
- 现代浏览器（Chrome / Firefox / Edge / Safari）

### 2. 创建 Supabase 项目

1. 登录 [Supabase Dashboard](https://app.supabase.com)
2. 点击 **New Project**，填写项目名称和数据库密码
3. 选择离你最近的区域，点击 **Create new project**
4. 等待项目初始化完成

### 3. 创建数据库表

进入项目的 **SQL Editor**，执行以下 SQL：

```sql
CREATE TABLE work_profiles (
  id        BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id   UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  settings_json JSONB NOT NULL DEFAULT '{}',
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT work_profiles_user_id_key UNIQUE (user_id)
);

-- 启用 RLS（行级安全）
ALTER TABLE work_profiles ENABLE ROW LEVEL SECURITY;

-- 用户只能读写自己的数据
CREATE POLICY "Users can read own profile"
  ON work_profiles FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own profile"
  ON work_profiles FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own profile"
  ON work_profiles FOR UPDATE
  USING (auth.uid() = user_id);
```

### 4. 启用邮箱认证

1. 进入 **Authentication** → **Providers**
2. 确保 **Email** 已启用
3. （可选）关闭 **Confirm email** 以跳过邮箱验证步骤，方便测试

### 5. 获取 API 密钥

进入 **Settings** → **API**，复制以下两个值：

- **Project URL**：格式为 `https://xxxxx.supabase.co`
- **anon public key**：以 `eyJ` 开头的 JWT Token

### 6. 配置应用

打开 `index.html`，找到第 995-996 行，替换为你的实际值：

```javascript
const SUPABASE_URL = 'https://你的项目ID.supabase.co';
const SUPABASE_ANON_KEY = '你的anon_key';
```

### 7. 部署 / 运行

**方式一：直接打开（不推荐）**

直接双击 `index.html` 用浏览器打开。此方式使用 `file://` 协议，部分浏览器可能阻止跨域请求导致登录失败。

**方式二：本地 HTTP 服务（推荐）**

```bash
# 使用 Python
python -m http.server 8080

# 或使用 Node.js
npx serve .

# 或使用 PHP
php -S localhost:8080
```

然后访问 `http://localhost:8080`。

**方式三：部署到静态托管**

将 `index.html` 上传至任意静态托管服务即可：

- GitHub Pages
- Vercel
- Netlify
- Cloudflare Pages
- Supabase Storage（开启 Public 访问）

## 使用说明

### 基础参数

| 参数 | 说明 |
|------|------|
| 时薪 | 每小时工资（元） |
| 默认上班时间 | 全局默认的上班打卡时间 |
| 默认下班时间 | 全局默认的下班打卡时间 |
| 午休开始/结束 | 午休时间段，自动从工时中扣除 |
| 结算周期 | 收入统计的起止日期范围 |

### 上班日选择

- 点击日历中的日期 → **选中该日期并标记为上班日**
- 再次点击已选中的日期 → **切换上班/休息状态**
- 按月分页浏览，点击「上个月」「下个月」切换

### 单日时间设置

- 在日历中点击某个日期后，下方会出现「单日上下班设置」面板
- 可以为特定日期设置独立的上下班时间（例如某天加班）
- 点击「恢复该日默认」可清除单独设置，回到全局默认时间
- 设置了单独时间的日期在日历中左上角会显示「自」标记

### 账号与同步

| 操作 | 说明 |
|------|------|
| 注册 | 输入邮箱和密码（至少 6 位），点击注册 |
| 登录 | 输入邮箱和密码，点击登录 |
| 自动登录 | 下次打开页面时自动恢复登录状态 |
| 同步到云端 | 手动将当前设置上传到云端 |
| 自动同步 | 每次修改设置后自动延迟 800ms 同步 |
| 退出登录 | 清除当前会话，回到本地缓存模式 |

### 统计指标说明

| 指标 | 含义 |
|------|------|
| 今日工时 | 当前时间距上班时间，扣除午休后的实际工作时长，秒级更新 |
| 今日收入 | 今日工时 × 时薪 |
| 今日进度 | 今日工时 / 今日标准工时 × 100% |
| 当前周期收入 | 周期内已过去且已勾选的上班日收入总和 |
| 该周期预计收入 | 包含未来已勾选上班日的预计收入总和 |
| 当前周期总工时 | 已过去上班日的工时总和 |
| 已选上班天数 | 已过去且已勾选的上班日数量 |
| 上班日均收入 | 周期收入 / 已工作天数 |
| 当前周期日均工时 | 周期总工时 / 已工作天数 |

## 数据结构

应用设置以 JSON 格式存储在 localStorage 和 Supabase `work_profiles.settings_json` 中：

```json
{
  "rate": 15,
  "workStart": "09:00",
  "workEnd": "18:30",
  "breakStart": "12:00",
  "breakEnd": "13:30",
  "periodStart": "2026-04-16",
  "periodEnd": "2026-05-16",
  "weekends": [0, 6],
  "activeDates": ["2026-04-16", "2026-04-17", "2026-04-21"],
  "dailyOverrides": {
    "2026-04-21": {
      "workStart": "08:00",
      "workEnd": "20:00"
    }
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `rate` | number | 时薪（元/小时） |
| `workStart` | string | 默认上班时间（HH:mm） |
| `workEnd` | string | 默认下班时间（HH:mm） |
| `breakStart` | string | 午休开始时间 |
| `breakEnd` | string | 午休结束时间 |
| `periodStart` | string | 结算周期开始日期（YYYY-MM-DD） |
| `periodEnd` | string | 结算周期结束日期 |
| `weekends` | number[] | 默认休息日（0=周日, 1=周一, ..., 6=周六） |
| `activeDates` | string[] | 已勾选的上班日期列表 |
| `dailyOverrides` | object | 单日时间覆盖，key 为日期字符串 |

## 项目结构

```
worktimecounter/
├── index.html    # 完整应用（HTML + CSS + JS 单文件）
└── README.md     # 本文档
```

## 常见问题

### 登录时提示 "Failed to fetch"

**原因**：Supabase 服务不可达，常见于以下情况：

1. **项目已暂停** — Supabase 免费套餐超过 7 天无活动会自动暂停项目。登录 Dashboard 点击 Restore 恢复。
2. **项目已删除** — 需要重新创建项目并更新配置。
3. **使用 file:// 协议打开** — 浏览器安全策略阻止跨域请求。改用 HTTP 服务（见部署章节）。
4. **网络问题** — 检查是否能访问 supabase.co。

### 数据丢失

- 未登录用户的数据保存在浏览器 localStorage，清除浏览器数据会导致丢失。
- 登录后数据会自动同步到云端，建议注册账号使用。

### 页面显示异常

- 确保使用现代浏览器（Chrome 90+、Firefox 90+、Edge 90+、Safari 15+）。
- 如果图表不显示，检查浏览器是否禁用了 JavaScript。

## 版本

- **v1.0.4** — 当前版本（Shimu）
