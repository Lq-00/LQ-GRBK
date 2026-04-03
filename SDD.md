# 3D 个人网站 - 系统设计文档（SDD）

## 1. 技术限制与非功能需求

### 1.1 硬件环境约束

| 硬件 | 规格 | 影响 |
|------|------|------|
| **用户电脑配置** | AMD Ryzen 7 7735H（8 核）, 16GB DDR5, RTX 4060 Laptop（8GB）, 512GB NVMe SSD | 开发机器性能充足，可流畅运行 Three.js 3D 渲染 |
| **访客电脑配置** | 配置差异大 | 需考虑低配置设备的降级方案 |
| **网络环境** | Wi-Fi/有线网络 | 3D 模型和纹理需压缩优化 |

### 1.2 性能需求

| 指标 | 目标值 | 说明 |
|------|--------|------|
| **首屏加载时间** | ≤3 秒 | 从访问页面到首屏内容可见 |
| **3D 场景 FPS** | ≥30FPS | 保证动画流畅 |
| **路由切换延迟** | ≤300ms | 页面切换响应时间 |
| **滚动动画延迟** | ≤16ms (60FPS) | 滚动触发动画流畅度 |

### 1.3 可用性需求

- **离线访问**：支持 PWA 离线缓存（可选）
- **错误处理**：3D 加载失败时显示降级内容（静态图）
- **SEO 友好**：支持 SSR 或预渲染（可选）

### 1.4 可扩展性需求

- **模块化设计**：各功能模块可独立替换
- **内容可扩展**：新增文章/作品只需添加文件
- **3D 场景可扩展**：支持导入新的 3D 模型和动画

---

## 2. 系统整体架构

### 2.1 架构概述

系统采用**静态单页应用 (SPA)** 架构：

```
┌─────────────────────────────────────────────────────────────┐
│                      用户浏览器                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    CDN / 静态托管                            │
│                  (Vercel / Netlify)                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Vue 3 单页应用                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ Vue Router  │  │   Pinia     │  │   Three.js 3D       │  │
│  │   路由管理   │  │  状态管理   │  │    渲染引擎         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ Tailwind CSS│  │  GSAP       │  │  Markdown Parser    │  │
│  │   样式      │  │  动画       │  │   文章解析          │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    本地文件系统                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  /content/  │  │  /public/   │  │   vite.config.js    │  │
│  │  文章&配置   │  │  静态资源   │  │   构建配置          │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 进程/部署架构

| 组件 | 职责 | 部署方式 |
|------|------|----------|
| **Vue 前端应用** | 负责渲染和用户交互 | Vercel/Netlify 自动部署 |
| **CDN** | 静态资源分发 | Vercel/Netlify 内置 |

---

## 3. 数据流图

### 3.1 页面渲染流程

```
用户访问页面
     │
     ▼
Vue Router 匹配路由
     │
     ▼
加载对应 View 组件
     │
     ├──────────────────┐
     ▼                  ▼
获取页面数据        初始化 3D 场景
     │                  │
     ▼                  ▼
读取 JSON/MD 文件    Three.js 渲染循环
     │                  │
     ▼                  ▼
组件渲染            响应用户交互
     │
     ▼
页面展示
```

### 3.2 文章读取流程

```
用户访问 /articles
     │
     ▼
ArticlesView 组件挂载
     │
     ▼
读取 /content/articles/ 目录
     │
     ▼
解析所有 MD 文件的 Frontmatter
     │
     ▼
按日期排序
     │
     ▼
渲染文章列表
```

### 3.3 3D 游戏交互流程

```
用户进入 /game 页面
     │
     ▼
初始化 Three.js 场景
     │
     ├─────────────┬─────────────┐
     ▼             ▼             ▼
创建地面       创建小车       创建障碍物
     │             │             │
     └─────────────┴─────────────┘
                    │
                    ▼
           初始化物理引擎 (Cannon.js)
                    │
                    ▼
           监听键盘事件
                    │
                    ▼
           更新小车位置/速度
                    │
                    ▼
           碰撞检测与响应
                    │
                    ▼
           渲染循环 (60 FPS)
```

### 3.4 滚动触发 3D 动画流程

```
用户滚动页面
     │
     ▼
监听 scroll 事件
     │
     ▼
计算滚动进度 (scrollY / documentHeight)
     │
     ▼
根据进度更新 3D 元素状态
     │
     ├─────────────┬─────────────┐
     ▼             ▼             ▼
   旋转          移动          缩放
     │             │             │
     └─────────────┴─────────────┘
                    │
                    ▼
           Three.js 重新渲染
```

---

## 4. 模块设计详解

### 4.1 前端模块

#### 4.1.1 3D 渲染模块

| 组件 | 职责 | 技术实现 |
|------|------|----------|
| **ThreeScene** | 3D 场景基类 | Three.js Scene |
| **CameraController** | 相机控制 | PerspectiveCamera + OrbitControls |
| **Lighting** | 场景光照 | AmbientLight + DirectionalLight |
| **ModelLoader** | 3D 模型加载 | GLTFLoader |
| **AnimationMixer** | 动画混合 | Three.js AnimationMixer |

#### 4.1.2 路由模块

```javascript
// src/router/index.js
const routes = [
  {
    path: '/',
    name: 'home',
    component: () => import('@/views/HomeView.vue')
  },
  {
    path: '/about',
    name: 'about',
    component: () => import('@/views/AboutView.vue')
  },
  {
    path: '/projects',
    name: 'projects',
    component: () => import('@/views/ProjectsView.vue')
  },
  {
    path: '/articles',
    name: 'articles',
    component: () => import('@/views/ArticlesView.vue')
  },
  {
    path: '/articles/:slug',
    name: 'article-detail',
    component: () => import('@/views/ArticleDetail.vue')
  },
  {
    path: '/game',
    name: 'game',
    component: () => import('@/views/GameView.vue')
  },
  {
    path: '/contact',
    name: 'contact',
    component: () => import('@/views/ContactView.vue')
  }
]
```

#### 4.1.3 状态管理模块

```javascript
// src/stores/app.js
import { defineStore } from 'pinia'

export const useAppStore = defineStore('app', {
  state: () => ({
    isLoading: false,
    currentTheme: 'light',
    threeScene: null
  }),
  actions: {
    setLoading(value) {
      this.isLoading = value
    },
    setScene(scene) {
      this.threeScene = scene
    }
  }
})
```

### 4.2 3D 游戏模块

```
GameEngine
│
├── Car
│   ├── position: Vector3
│   ├── velocity: Vector3
│   ├── speed: number
│   ├── steering: number
│   ├── move(direction: string)
│   ├── stop()
│   └── update(deltaTime: number)
│
├── PhysicsWorld
│   ├── world: CANNON.World
│   ├── bodies: Map<string, CANNON.Body>
│   ├── addBody()
│   ├── removeBody()
│   └── step(deltaTime: number)
│
├── CollisionHandler
│   ├── onCollide(a: Body, b: Body)
│   ├── checkCollision()
│   └── applyImpulse()
│
└── GameLoop
    ├── start()
    ├── stop()
    └── update()
```

---

## 5. 数据存储设计

### 5.1 数据存储方案

由于采用轻量级方案，**不使用传统数据库**，数据存储在文件系统中：

#### 5.1.1 文章数据
- **存储位置**：`src/content/articles/*.md`
- **文件格式**：Markdown + Frontmatter

```markdown
---
title: 文章标题
date: 2026-04-03
tags: [Vue, Three.js]
summary: 文章摘要
---

文章正文内容...
```

#### 5.1.2 作品数据
- **存储位置**：`src/content/data/projects.json`
- **格式**：
```json
[
  {
    "id": 1,
    "title": "项目名称",
    "description": "项目描述",
    "image": "/images/project1.png",
    "link": "https://github.com/...",
    "techStack": ["Vue", "Three.js"],
    "date": "2026-04"
  }
]
```

#### 5.1.3 个人信息数据
- **存储位置**：`src/content/data/about.json`
- **格式**：
```json
{
  "name": "乱气",
  "bio": "个人简介...",
  "skills": [
    { "name": "Vue", "level": 80 },
    { "name": "Three.js", "level": 60 }
  ],
  "experience": [
    {
      "company": "公司名称",
      "role": "职位",
      "start": "2024-01",
      "end": "至今",
      "description": "工作描述"
    }
  ],
  "education": [
    {
      "school": "学校名称",
      "degree": "学历",
      "year": "2020-2024"
    }
  ],
  "contact": {
    "email": "your@email.com",
    "github": "https://github.com/...",
    "wechat": "微信号"
  }
}
```

---

## 6. 接口设计

*本项目为静态网站，无后端 API 接口*

### 6.1 内部数据加载接口

| 接口 | 方法 | 用途 |
|------|------|------|
| `/content/articles/*.md` | GET | 获取文章内容 |
| `/content/data/projects.json` | GET | 获取作品列表 |
| `/content/data/about.json` | GET | 获取个人信息 |

---

## 7. 目录结构

```
project-root/
├── public/
│   ├── favicon.ico
│   └── models/                 # 3D 模型文件
│       └── car.glb
├── src/
│   ├── assets/
│   │   ├── main.css
│   │   └── styles/
│   ├── components/
│   │   ├── Navbar.vue
│   │   ├── ThreeScene.vue
│   │   ├── ProjectCard.vue
│   │   └── ...
│   ├── views/
│   │   ├── HomeView.vue
│   │   ├── AboutView.vue
│   │   ├── ProjectsView.vue
│   │   ├── ArticlesView.vue
│   │   ├── ArticleDetail.vue
│   │   ├── GameView.vue
│   │   └── ContactView.vue
│   ├── router/
│   │   └── index.js
│   ├── stores/
│   │   ├── app.js
│   │   └── game.js
│   ├── utils/
│   │   ├── three.js
│   │   └── markdown.js
│   ├── game/
│   │   ├── Car.js
│   │   ├── Physics.js
│   │   └── GameLoop.js
│   ├── content/
│   │   ├── articles/           # 文章 MD 文件
│   │   └── data/               # JSON 数据
│   │       ├── projects.json
│   │       └── about.json
│   ├── App.vue
│   └── main.js
├── index.html
├── package.json
├── vite.config.js
├── tailwind.config.js
└── postcss.config.js
```

---

## 8. 技术栈选型

### 8.1 核心技术栈

| 技术 | 版本 | 用途 | 选择理由 |
|------|------|------|----------|
| Vue | 3.x | 前端框架 | 轻量、易上手、生态完善 |
| Vite | 5.x | 构建工具 | 极速热更新、开发体验好 |
| Three.js | r160+ | 3D 渲染 | 最成熟的 WebGL 库 |
| Vue Router | 4.x | 路由管理 | Vue 官方路由 |
| Pinia | 2.x | 状态管理 | Vue 3 推荐，比 Vuex 更简洁 |
| Tailwind CSS | 3.x | 样式框架 | 原子化 CSS，开发效率高 |

### 8.2 辅助库

| 技术 | 用途 |
|------|------|
| GSAP | 滚动触发动画、时间轴动画 |
| markdown-it | Markdown 解析 |
| @vueuse/core | Vue 组合式 API 工具集 |
| cannon-es | 3D 物理引擎（用于小车游戏） |

---

## 9. 性能优化策略

### 9.1 代码层面
- **代码分割**：路由懒加载
- **3D 模型优化**：使用 GLTF 压缩格式 (Draco)
- **纹理压缩**：使用 KTX2/Basis 格式
- **LOD (Level of Detail)**：远处物体使用低模

### 9.2 渲染优化
- **限制渲染范围**：不可见时暂停 Three.js 渲染
- **对象池**：复用游戏对象，减少 GC
- **批量渲染**：合并静态几何体

### 9.3 加载优化
- **骨架屏**：页面加载时显示占位
- **渐进式加载**：先加载低清资源，再替换高清
- **预加载**：关键资源预加载

---

## 10. 部署方案

### 10.1 推荐部署平台

#### 方案 A: Vercel (推荐)
- **优点**：
  - 免费个人版
  - 自动 HTTPS
  - 全球 CDN
  - 自动部署（连接 GitHub）
  - 自定义域名
- **部署步骤**：
  1. 推送代码到 GitHub
  2. 在 Vercel 导入项目
  3. 自动构建部署

#### 方案 B: Netlify
- **优点**：与 Vercel 类似，免费额度充足
- **部署方式**：同上

#### 方案 C: 自有服务器
- **优点**：完全控制
- **缺点**：需要维护服务器、配置 HTTPS
- **部署方式**：`npm run build` 后上传到 Nginx/Apache

---

## 11. 技术风险与应对

| 风险 | 影响 | 应对方案 |
|------|------|----------|
| Three.js 学习曲线 | 开发进度延迟 | 参考官方示例和文档 |
| 物理引擎性能 | 游戏卡顿 | 简化碰撞体，使用简单的盒型碰撞 |
| Markdown 文件过多 | 加载慢 | 分页加载、虚拟滚动 |
| 3D 效果兼容性 | 部分浏览器不支持 | 降级方案（静态图） |
| 移动端不支持 | 手机用户无法访问 | 后续可扩展响应式布局 |

---

## 修订记录

| 版本 | 日期 | 修改内容 | 作者 |
|------|------|----------|------|
| v1.0 | 2026-04-03 | 初始版本 | 乱气 |
