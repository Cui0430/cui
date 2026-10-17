# 大学生个人主页运行说明

## 当前功能

- 使用学号和姓名注册、登录，登录成功进入个人主页。
- 基本资料：大学、专业、年级、个人简介。
- 任职经历与获奖证书：添加、编辑、删除。
- 项目经历：添加、编辑、删除。
- 个人数据存入 Oracle；个人资料接口由登录令牌保护。

学号和姓名仅用于本地学习演示，不能验证真实学生身份，不要把它当作正式认证方式。

## 启动

确认 Oracle 服务运行，`backend/.env` 中的 Oracle 用户、密码和服务名正确。

在项目根目录开一个终端启动后端：

```powershell
cd D:\Users\ASUS\vscodepro\vue-test
.\backend\.venv\Scripts\python.exe .\portfolio_api.py
```

第一次启动会创建 `PROFILES`、`EXPERIENCES` 表，并在已有 `USERS` 表缺字段时补充学号/姓名字段。

再开一个终端启动前端：

```powershell
cd D:\Users\ASUS\vscodepro\vue-test
npm run dev
```

打开 Vite 在终端显示的地址，通常是 `http://localhost:5173/`。两个终端都要保持运行。

## 接口流程

```text
注册 / 登录页面
  → Axios POST /api/register 或 /api/login
  → FastAPI 查询或写入 Oracle USERS
  → 返回 JWT，前端保存
  → 进入个人主页
  → 读取或修改 PROFILES、EXPERIENCES
```

`portfolio_api.py` 是当前个人主页后端入口。旧的 `backend/app/main.py` 属于原登录/活动项目，不要用 `python -m app.main` 启动本版本。
