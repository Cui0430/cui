# 校园活动报名系统：前后端复盘与注册实操

> 本文是本项目的操作手册。第一次搭建环境时从第 4 节开始；以后每天开发时重点看第 5 节和第 10 节。

## 0. 命令什么时候执行

这些命令不是每次都全部执行。

### 第一次搭建后端环境

先进入后端目录：

```powershell
cd D:\Users\ASUS\vscodepro\vue-test\backend
```

创建虚拟环境，只执行一次：

```powershell
python -m venv .venv
```

激活虚拟环境。每打开一个新的终端执行一次：

```powershell
.\.venv\Scripts\Activate.ps1
```

安装依赖。第一次搭建、重新创建 `.venv` 或新增依赖时执行：

```powershell
python -m pip install -r requirements.txt
```

创建 Oracle 配置文件，只执行一次：

```powershell
Copy-Item .env.example .env
notepad .env
```

### 以后每次启动后端

如果当前终端前面没有 `(.venv)`，先执行：

```powershell
cd D:\Users\ASUS\vscodepro\vue-test\backend
.\.venv\Scripts\Activate.ps1
```

然后启动后端：

```powershell
python -m app.main
```

看到下面信息后，不要关闭这个终端：

```text
Uvicorn running on http://127.0.0.1:8000
```

如果当前已经显示 `(.venv)`，可以直接执行：

```powershell
python -m app.main
```

### 修改后端代码以后

当前启动命令不会自动重启。修改 `main.py` 后按 `Ctrl + C` 停止，再重新执行：

```powershell
python -m app.main
```

也可以使用自动重载模式：

```powershell
python -m uvicorn app.main:app --reload --port 8000
```

### 这些命令分别做什么

| 命令 | 作用 | 执行时机 |
| --- | --- | --- |
| `python -m venv .venv` | 创建项目专用 Python 环境 | 第一次或环境损坏后 |
| `.\.venv\Scripts\Activate.ps1` | 激活虚拟环境 | 每个新终端一次 |
| `python -m pip install -r requirements.txt` | 安装依赖 | 第一次或依赖变化后 |
| `Copy-Item .env.example .env` | 创建数据库配置文件 | 第一次一次 |
| `notepad .env` | 编辑数据库用户名和密码 | 第一次或配置变化后 |
| `python -m app.main` | 启动 FastAPI 后端 | 每次运行后端 |
| `npm run dev` | 启动 Vue 前端 | 每次运行前端 |

## 目录和关键文件

```text
vue-test/
├─ backend/
│  ├─ app/main.py              后端接口、Oracle 操作
│  ├─ requirements.txt         Python 依赖清单
│  ├─ .env.example             数据库配置模板
│  └─ .env                     本机真实配置，不提交到 Git
├─ src/
│  ├─ views/login.vue          登录页面
│  ├─ views/register.vue       注册页面
│  ├─ api/index.js             登录、注册请求
│  ├─ utils/index.js           Axios 地址、Token 拦截器
│  ├─ router/index.js          页面路由
│  └─ stores/counter.js        保存用户和 Token
├─ .env.development            Vue 的后端地址
└─ PROJECT_GUIDE.md            本复盘文档
```

## 1. 当前项目完成到哪里

当前已经完成：

- Vue 登录页面
- Axios 请求封装
- FastAPI 后端
- Oracle XE 数据库连接
- `USERS` 用户表自动创建
- 学生和管理员测试账号
- 密码哈希保存
- `POST /api/login` 登录接口
- JWT 登录令牌
- 前端 Pinia 保存用户和 Token
- `GET /health` 健康检查
- `POST /api/register` 注册接口
- Vue 注册页面

当前还没有完成：

- 活动表和活动接口
- 报名表和报名接口
- 管理员发布、修改、删除活动
- 完整的 Token 鉴权依赖

## 2. 软件之间的关系

```text
Oracle XE
  ↑ 由 python-oracledb 连接
FastAPI 后端（127.0.0.1:8000）
  ↑ 由 Axios/Apipost 发送 HTTP 请求
Vue 前端（127.0.0.1:5173）
```

每个软件的职责：

| 软件 | 做什么 |
| --- | --- |
| Oracle XE | 真正保存用户和业务数据 |
| PL/SQL Developer | 查看 Oracle 表、执行 SQL |
| PyCharm | 编写和运行 Python 后端 |
| FastAPI | 定义接口、处理业务 |
| Apipost | 不经过 Vue，直接测试 HTTP 接口 |
| Vue/Vite | 显示页面、收集输入、调用接口 |
| Axios | 在 Vue 中发送 HTTP 请求 |
| Pinia | 在前端保存登录用户和 Token |

PL/SQL Developer 和 PyCharm 不是直接连接：

```text
PyCharm 中的 Python 代码 ─┐
                          ├─ Oracle XE
PL/SQL Developer ─────────┘
```

Python 和 PL/SQL Developer 使用同一个数据库连接：

```text
用户：campus_app
地址：127.0.0.1:1521/XEPDB1
```

## 3. 一次登录请求经过哪些文件

```text
src/views/login.vue
  → src/api/index.js
  → src/utils/index.js
  → POST http://127.0.0.1:8000/api/login
  → backend/app/main.py 的 login()
  → Oracle USERS 表
  → 返回 JSON
  → src/stores/counter.js 保存 Token
```

前端调用：

```javascript
await login({
  username: form.username,
  password: form.password
})
```

请求函数：

```javascript
export function login(payload) {
  return request({
    url: '/api/login',
    method: 'post',
    data: payload,
  })
}
```

后端接口：

```python
@app.post("/api/login")
def login(payload: LoginRequest):
    ...
```

## 4. 数据库表是怎么创建的

先用 PL/SQL Developer 以 `SYSTEM` 登录 `XEPDB1`，创建项目用户：

```sql
CREATE USER campus_app IDENTIFIED BY "CampusApp123";

GRANT CREATE SESSION, CREATE TABLE, CREATE SEQUENCE,
      CREATE TRIGGER, CREATE VIEW TO campus_app;

ALTER USER campus_app QUOTA UNLIMITED ON USERS;
```

之后后端启动时，`init_database()` 执行：

```text
python -m app.main
  → on_startup()
  → init_database()
  → CREATE TABLE users
```

当前表结构：

| 字段 | 含义 |
| --- | --- |
| `id` | 用户编号 |
| `username` | 用户名，唯一 |
| `password_hash` | 加密后的密码 |
| `user_role` | `student` 或 `admin` |
| `name` | 姓名 |
| `student_id` | 学号 |
| `phone` | 手机号 |
| `created_at` | 创建时间 |

在 PL/SQL Developer 中查看：

```text
campus_app → Tables → USERS
```

查看数据：

```sql
SELECT * FROM users;
```

## 5. 启动顺序

### 5.1 Oracle

确保 Oracle 服务和 Listener 正在运行。检查端口：

```powershell
Test-NetConnection 127.0.0.1 -Port 1521
```

看到 `TcpTestSucceeded : True` 即可。

### 5.2 后端

```powershell
cd D:\Users\ASUS\vscodepro\vue-test\backend
.\.venv\Scripts\Activate.ps1
python -m app.main
```

浏览器打开：

```text
http://127.0.0.1:8000/health
```

### 5.3 前端

另开一个终端：

```powershell
cd D:\Users\ASUS\vscodepro\vue-test
npm run dev
```

浏览器打开：

```text
http://127.0.0.1:5173/
```

当前 Oracle 方案不需要执行 `docker compose up -d --build`。

## 6. 注册接口设计

### 6.1 接口信息

```text
POST http://127.0.0.1:8000/api/register
Content-Type: application/json
```

### 6.2 请求体

```json
{
  "username": "student002",
  "password": "student456",
  "name": "张三",
  "student_id": "20260002",
  "phone": "13800000002"
}
```

### 6.3 后端处理过程

```text
接收 JSON
  → 校验用户名和密码长度
  → 查询 username 是否已存在
  → 密码加盐哈希
  → INSERT INTO users
  → 提交事务 connection.commit()
  → 返回注册成功
```

注册接口的核心代码：

```python
@app.post("/api/register")
def register(payload: RegisterRequest):
    with get_connection() as connection:
        with connection.cursor() as cursor:
            if user_exists(cursor, payload.username):
                return {"code": 1, "msg": "用户名已存在", "data": None}

            cursor.execute(
                """
                INSERT INTO users
                    (username, password_hash, user_role, name, student_id, phone)
                VALUES
                    (:username, :password_hash, 'student',
                     :name, :student_id, :phone)
                """,
                username=payload.username,
                password_hash=hash_password(payload.password),
                name=payload.name,
                student_id=payload.student_id,
                phone=payload.phone,
            )
            connection.commit()

    return {"code": 0, "msg": "注册成功", "data": {...}}
```

注册用户固定是 `student`，管理员不能通过公开注册接口创建。

## 7. Apipost 实操注册接口

1. 启动后端。
2. 打开 Apipost，新建 HTTP 接口。
3. 方法选择 `POST`。
4. 地址填写：

```text
http://127.0.0.1:8000/api/register
```

5. 进入 `Body`，选择 `raw`，格式选择 `JSON`。
6. 粘贴：

```json
{
  "username": "student002",
  "password": "student456",
  "name": "张三",
  "student_id": "20260002",
  "phone": "13800000002"
}
```

7. 点击“发送”。

成功响应：

```json
{
  "code": 0,
  "msg": "注册成功",
  "data": {
    "username": "student002",
    "role": "student",
    "name": "张三"
  }
}
```

然后在 PL/SQL Developer 中刷新 `Tables → USERS`，执行：

```sql
SELECT id, username, user_role, name, student_id, phone
FROM users
WHERE username = 'student002';
```

你应该能看到刚注册的数据。再次使用同一个用户名注册，应该返回：

```json
{
  "code": 1,
  "msg": "用户名已存在",
  "data": null
}
```

## 8. Vue 注册页面实操

注册页面文件：

```text
src/views/register.vue
```

登录页已有：

```html
<router-link to="/register">立即注册</router-link>
```

路由在 `src/router/index.js` 中：

```javascript
{
  path: '/register',
  name: 'register',
  component: () => import('../views/register.vue'),
}
```

前端请求函数在 `src/api/index.js`：

```javascript
export function register(payload) {
  return request({
    url: '/api/register',
    method: 'post',
    data: payload,
  })
}
```

页面表单点击注册后，会发送：

```javascript
await register({
  username: form.username,
  password: form.password,
  name: form.name,
  student_id: form.student_id || null,
  phone: form.phone || null
})
```

验证步骤：

1. 后端窗口保持运行。
2. 前端执行 `npm run dev`。
3. 打开 `http://127.0.0.1:5173/`。
4. 点击“立即注册”。
5. 填写用户名、密码、确认密码、姓名。
6. 点击“注册”。
7. 注册成功后回到登录页。
8. 使用新账号登录。

## 9. 注册失败时按这个顺序排查

### 后端没有启动

检查后端窗口是否有：

```text
Uvicorn running on http://127.0.0.1:8000
```

### 连接 Oracle 失败

访问：

```text
http://127.0.0.1:8000/health
```

如果失败，检查 `backend/.env` 和 Oracle 服务。

### 用户名已存在

换一个用户名，或者在 PL/SQL Developer 查询：

```sql
SELECT username FROM users;
```

### 前端请求失败

确认 `.env.development`：

```ini
VITE_API_BASE_URL=http://127.0.0.1:8000
```

修改后要重启 Vite。

## 10. 下一步完整项目路线

```text
登录
  → 注册
  → 活动表
  → GET /api/activities
  → 活动详情
  → 报名表
  → POST /api/activities/{id}/register
  → 取消报名
  → JWT 鉴权
  → 管理员权限
```
