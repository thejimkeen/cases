# RPG 职业教育系统 - 技术实现方案

## 文档概述

**目标：** 将《场景剧本01-货车选择的第一天》等教学场景，实现为可在 Web 端和移动端运行的交互式学习系统。

**核心特点：**
- 剧情驱动的决策教学
- 真实的工作场景模拟（填单、计算、查询）
- 学生数据追踪与分析
- 教师端管理与监控

**预期用户：**
- 学生端：手机/电脑访问，完成场景任务
- 教师端：电脑访问，管理案例、查看数据

---

## 一、系统架构设计

### 1.1 整体技术架构

```
┌─────────────────────────────────────────────────────┐
│                   客户端层 (Client)                   │
├─────────────────────────────────────────────────────┤
│  学生端 (Mobile/Web)      │    教师端 (Web)          │
│  - React / Vue.js         │    - React + Ant Design  │
│  - 响应式设计             │    - 数据可视化 (ECharts)│
│  - PWA 支持               │    - 案例编辑器          │
└─────────────┬───────────────────────┬───────────────┘
              │                       │
              │     HTTPS / WSS       │
              │                       │
┌─────────────▼───────────────────────▼───────────────┐
│                 API 网关层 (Gateway)                  │
│  - Nginx / Traefik                                   │
│  - 认证中间件 (JWT)                                  │
│  - 限流、日志                                         │
└─────────────┬───────────────────────────────────────┘
              │
┌─────────────▼───────────────────────────────────────┐
│                 应用层 (Application)                  │
├─────────────────────────────────────────────────────┤
│  业务服务 (Node.js / Python / Go)                    │
│  ├─ 场景引擎服务 (Scene Engine)                      │
│  ├─ 用户服务 (User Service)                          │
│  ├─ 评分服务 (Scoring Service)                       │
│  ├─ 数据分析服务 (Analytics Service)                 │
│  └─ 通知服务 (Notification Service)                  │
└─────────────┬───────────────────────────────────────┘
              │
┌─────────────▼───────────────────────────────────────┐
│                  数据层 (Data)                        │
├─────────────────────────────────────────────────────┤
│  主数据库 (PostgreSQL / MySQL)                       │
│  - 用户、课程、进度、成绩                             │
│                                                      │
│  文档数据库 (MongoDB)                                │
│  - 场景定义、决策树、剧本                            │
│                                                      │
│  缓存 (Redis)                                        │
│  - 会话、实时状态、排行榜                            │
│                                                      │
│  对象存储 (OSS / S3)                                 │
│  - 图片、音频、视频资源                              │
└─────────────────────────────────────────────────────┘
```

### 1.2 技术栈选择

#### 前端技术栈

**方案 A：Vue.js 全家桶（推荐初创团队）**

```javascript
// 核心框架
Vue 3 + TypeScript
Vite (构建工具)
Pinia (状态管理)
Vue Router (路由)

// UI 组件库
学生端: Vant (移动端)
教师端: Element Plus (桌面端)

// 可视化
ECharts (数据图表)

// 其他工具
Axios (HTTP 请求)
Day.js (时间处理)
```

**优势：**
- 学习曲线平缓，国内文档丰富
- 生态完整，组件库成熟
- 适合中小团队快速开发

**方案 B：React 生态（推荐有经验团队）**

```javascript
// 核心框架
React 18 + TypeScript
Next.js (SSR 框架)
Zustand / Redux Toolkit (状态管理)

// UI 组件库
学生端: Ant Design Mobile
教师端: Ant Design

// 其他同上
```

**优势：**
- 社区更大，第三方库更多
- 适合构建大型应用
- 招聘更容易

#### 后端技术栈

**方案 A：Node.js（推荐，与前端统一语言）**

```javascript
// 核心框架
Node.js 18+ 
Express.js / Koa.js / NestJS

// ORM
Prisma / TypeORM

// 认证
JWT + Passport.js

// 实时通信
Socket.io (可选)

// 任务队列
Bull (基于 Redis)
```

**方案 B：Python（推荐如果需要 AI 能力）**

```python
# 核心框架
Python 3.10+
FastAPI (高性能异步框架)

# ORM
SQLAlchemy

# 任务队列
Celery

# 数据分析
Pandas, NumPy (用于教师端数据分析)
```

**方案 C：Go（推荐追求极致性能）**

```go
// 核心框架
Go 1.20+
Gin / Echo

// ORM
GORM

// 优势：性能好，部署简单（单二进制文件）
```

#### 数据库选择

```
关系型数据库：PostgreSQL（推荐）
- 用户、课程、进度、成绩等结构化数据
- 支持 JSON 字段，灵活性好
- 开源、性能优秀

文档数据库：MongoDB
- 场景定义（JSON 格式的决策树）
- 灵活的 Schema，适合快速迭代

缓存：Redis
- 会话管理
- 实时排行榜
- 分布式锁

对象存储：阿里云 OSS / 腾讯云 COS / AWS S3
- 图片、音频、视频
```

### 1.3 系统模块划分

```
系统模块
├── 学生端 (Student App)
│   ├── 登录/注册模块
│   ├── 场景游戏模块
│   │   ├── 剧情展示
│   │   ├── 决策选择
│   │   ├── 工具使用（填单、计算）
│   │   └── 进度保存
│   ├── 个人中心
│   │   ├── 成绩记录
│   │   ├── 成就展示
│   │   └── 能力雷达图
│   └── 排行榜
│
├── 教师端 (Teacher Dashboard)
│   ├── 案例管理
│   │   ├── 场景编辑器
│   │   ├── 发布/下架
│   │   └── 版本管理
│   ├── 班级管理
│   │   ├── 学生列表
│   │   ├── 分组管理
│   │   └── 权限设置
│   ├── 数据分析
│   │   ├── 完成率统计
│   │   ├── 决策路径热力图
│   │   ├── 常见错误分析
│   │   └── 个体学习报告
│   └── 系统设置
│
└── 后端服务 (Backend Services)
    ├── 用户服务 (User Service)
    ├── 场景引擎 (Scene Engine)
    ├── 评分服务 (Scoring Service)
    ├── 数据统计服务 (Analytics Service)
    └── 通知服务 (Notification Service)
```

---

## 二、核心功能模块详解

### 2.1 场景引擎 (Scene Engine)

**职责：** 驱动剧情推进、管理游戏状态

#### 核心概念

```
场景 (Scene)：一个完整的教学案例（如"货车选择的第一天"）
节点 (Node)：场景中的一个决策点或剧情点
选项 (Option)：玩家可以做出的选择
状态 (State)：当前游戏的全局状态（时间、成本、信任度等）
```

#### 数据结构设计

**场景定义 (Scene Definition)**

```json
{
  "sceneId": "scene_001",
  "title": "货车选择的第一天",
  "description": "学习如何根据货物特性选择运输方式",
  "difficulty": 2,
  "estimatedTime": 20,
  "learningGoals": [
    "信息收集能力",
    "多因素决策",
    "风险评估"
  ],
  "startNodeId": "node_000",
  "nodes": { /* 节点定义，见下方 */ },
  "globalState": {
    "time": "08:45",
    "budget": 1500,
    "trustZhangLei": 50,
    "infoCompleteness": 0
  },
  "npcs": [
    {
      "id": "zhang_lei",
      "name": "张雷",
      "role": "调度主管",
      "avatar": "/assets/zhang_lei.png",
      "initialTrust": 50
    }
  ]
}
```

**节点定义 (Node Definition)**

```json
{
  "nodeId": "node_001",
  "type": "dialogue",
  "title": "接到任务",
  "background": "/assets/bg_office.jpg",
  "speaker": "zhang_lei",
  "dialogue": {
    "text": "林晓，仓库有一批货要发...",
    "audio": "/assets/audio/zhang_001.mp3"
  },
  "narration": "张雷把一张订单拍在你桌上...",
  "stateChanges": {
    "time": "+5min"
  },
  "options": [
    {
      "id": "A",
      "text": "直接在系统里安排一辆厢式货车",
      "hint": "厢式货车更安全，应该没问题",
      "timeRequired": 2,
      "nextNodeId": "node_002a",
      "conditions": null,
      "effects": {
        "experience.impulsive": -2
      }
    },
    {
      "id": "B",
      "text": "先去仓库了解货物的具体情况",
      "hint": "连货是什么都不知道，怎么安排车？",
      "timeRequired": 10,
      "nextNodeId": "node_003",
      "effects": {
        "experience.infoGathering": +5,
        "trustZhangLei": +5
      }
    }
  ],
  "tools": null
}
```

**交互节点（填单节点）**

```json
{
  "nodeId": "node_007",
  "type": "form",
  "title": "填写运输单",
  "instruction": "请根据收集的信息填写运输调度单",
  "form": {
    "formId": "transport_order_001",
    "fields": [
      {
        "id": "vehicle_type",
        "label": "车辆类型",
        "type": "select",
        "required": true,
        "options": [
          { "value": "box_truck", "label": "厢式货车" },
          { "value": "flatbed", "label": "平板车" }
        ],
        "validation": {
          "rule": "required"
        }
      },
      {
        "id": "vehicle_count",
        "label": "车辆数量",
        "type": "number",
        "required": true,
        "min": 1,
        "max": 5,
        "validation": {
          "rule": "custom",
          "function": "validateVehicleCount"
        }
      },
      {
        "id": "rain_protection",
        "label": "防雨措施",
        "type": "select",
        "required": false,
        "showIf": "vehicle_type === 'flatbed'",
        "options": [
          { "value": "none", "label": "不做防护" },
          { "value": "standard", "label": "标准篷布" },
          { "value": "heavy_duty", "label": "加厚防水篷布" },
          { "value": "plastic_wrap", "label": "塑料膜包装" }
        ]
      }
    ],
    "submitButton": "提交调度单",
    "onSubmit": {
      "validate": true,
      "scoreLogic": "calculateTransportScore",
      "nextNodeId": "node_008"
    }
  }
}
```

**计算工具节点**

```json
{
  "nodeId": "node_005",
  "type": "calculator",
  "title": "成本核算",
  "instruction": "请计算两种方案的总成本",
  "calculator": {
    "type": "cost_calculator",
    "template": "transport_cost",
    "inputs": [
      {
        "id": "vehicle_type",
        "label": "车辆类型",
        "type": "select",
        "options": [
          { "value": "box_truck", "label": "厢式货车", "unitPrice": 1200 },
          { "value": "flatbed", "label": "平板车", "unitPrice": 800 }
        ]
      },
      {
        "id": "vehicle_count",
        "label": "车辆数量",
        "type": "number"
      },
      {
        "id": "labor_hours",
        "label": "人工工时",
        "type": "number",
        "hint": "每人每小时50元"
      }
    ],
    "formula": {
      "totalCost": "vehicle_type.unitPrice * vehicle_count + labor_hours * 50"
    },
    "output": {
      "display": "总成本：¥{totalCost}",
      "saveToState": "calculatedCost"
    }
  },
  "nextNodeId": "node_006"
}
```

#### 引擎核心逻辑

**场景状态机 (Scene State Machine)**

```javascript
class SceneEngine {
  constructor(sceneDefinition, initialState) {
    this.scene = sceneDefinition;
    this.state = {
      currentNodeId: sceneDefinition.startNodeId,
      globalState: { ...sceneDefinition.globalState, ...initialState },
      history: [],
      decisions: []
    };
  }

  // 获取当前节点
  getCurrentNode() {
    return this.scene.nodes[this.state.currentNodeId];
  }

  // 处理玩家选择
  async makeChoice(optionId) {
    const currentNode = this.getCurrentNode();
    const option = currentNode.options.find(opt => opt.id === optionId);
    
    if (!option) {
      throw new Error('Invalid option');
    }

    // 检查条件
    if (option.conditions && !this.evaluateConditions(option.conditions)) {
      return { success: false, message: '不满足选择条件' };
    }

    // 记录决策
    this.state.decisions.push({
      nodeId: this.state.currentNodeId,
      optionId: optionId,
      timestamp: Date.now()
    });

    // 应用效果
    this.applyEffects(option.effects);

    // 跳转到下一个节点
    this.state.history.push(this.state.currentNodeId);
    this.state.currentNodeId = option.nextNodeId;

    // 保存进度
    await this.saveProgress();

    return {
      success: true,
      nextNode: this.getCurrentNode(),
      stateChanges: option.effects
    };
  }

  // 应用状态变化
  applyEffects(effects) {
    if (!effects) return;
    
    for (const [key, value] of Object.entries(effects)) {
      if (typeof value === 'string' && value.startsWith('+')) {
        // 增量变化
        const delta = parseInt(value);
        this.state.globalState[key] = (this.state.globalState[key] || 0) + delta;
      } else if (typeof value === 'string' && value.startsWith('-')) {
        // 减量变化
        const delta = parseInt(value.substring(1));
        this.state.globalState[key] = (this.state.globalState[key] || 0) - delta;
      } else {
        // 直接赋值
        this.state.globalState[key] = value;
      }
    }
  }

  // 评估条件
  evaluateConditions(conditions) {
    // 示例: conditions = "infoCompleteness >= 80"
    const context = this.state.globalState;
    return eval(conditions); // 生产环境应使用安全的表达式引擎
  }

  // 保存进度
  async saveProgress() {
    const progressData = {
      sceneId: this.scene.sceneId,
      currentNodeId: this.state.currentNodeId,
      globalState: this.state.globalState,
      history: this.state.history,
      decisions: this.state.decisions,
      timestamp: Date.now()
    };
    
    // 调用 API 保存
    await api.saveProgress(progressData);
  }

  // 恢复进度
  static async loadProgress(sceneId, userId) {
    const progressData = await api.loadProgress(sceneId, userId);
    const sceneDefinition = await api.getScene(sceneId);
    
    const engine = new SceneEngine(sceneDefinition, {});
    engine.state = progressData;
    
    return engine;
  }

  // 计算最终评分
  calculateScore() {
    const scorer = new SceneScorer(this.scene, this.state);
    return scorer.calculate();
  }
}
```

**使用示例**

```javascript
// 初始化场景
const engine = new SceneEngine(sceneDefinition, {
  studentId: '12345',
  className: 'logistics_2024_01'
});

// 显示当前节点
const currentNode = engine.getCurrentNode();
renderNode(currentNode);

// 玩家做出选择
const result = await engine.makeChoice('B'); // 选择"先去仓库"

if (result.success) {
  // 显示下一个节点
  renderNode(result.nextNode);
  
  // 显示状态变化提示
  showFeedback(result.stateChanges);
}

// 场景结束时计算评分
if (engine.isSceneComplete()) {
  const score = engine.calculateScore();
  showScoreScreen(score);
}
```

### 2.2 评分系统 (Scoring Service)

#### 评分维度

```javascript
const scoringDimensions = {
  // 信息收集能力
  infoGathering: {
    weight: 0.25,
    metrics: [
      'hasCheckedWeather',      // 是否查看天气
      'hasCheckedCustomer',     // 是否查看客户记录
      'hasAskedWarehouse',      // 是否询问仓库
      'infoCompletenessPct'     // 信息完整度百分比
    ]
  },
  
  // 决策质量
  decisionQuality: {
    weight: 0.30,
    metrics: [
      'decisionSpeed',          // 决策速度（不是越快越好）
      'optimalPathMatch',       // 是否走最优路径
      'riskAssessment',         // 风险评估是否到位
      'costEfficiency'          // 成本效率
    ]
  },
  
  // 沟通协调
  communication: {
    weight: 0.20,
    metrics: [
      'proactiveCommunication', // 主动沟通次数
      'customerHandling',       // 客户应对质量
      'clarityOfReporting'      // 汇报清晰度
    ]
  },
  
  // 应急处理
  emergencyHandling: {
    weight: 0.15,
    metrics: [
      'responseTime',           // 响应速度
      'problemSolvingQuality',  // 问题解决质量
      'contingencyPlanning'     // 预案思维
    ]
  },
  
  // 结果质量
  resultQuality: {
    weight: 0.10,
    metrics: [
      'taskCompletion',         // 任务完成度
      'customerSatisfaction',   // 客户满意度
      'damageLoss'              // 货物损失
    ]
  }
};
```

#### 评分算法实现

```javascript
class SceneScorer {
  constructor(scene, gameState) {
    this.scene = scene;
    this.state = gameState;
  }

  calculate() {
    const scores = {};
    let totalScore = 0;

    // 计算每个维度的得分
    for (const [dimension, config] of Object.entries(scoringDimensions)) {
      const dimensionScore = this.calculateDimensionScore(dimension, config);
      scores[dimension] = dimensionScore;
      totalScore += dimensionScore * config.weight;
    }

    // 确定评级
    const grade = this.determineGrade(totalScore);

    // 生成反馈
    const feedback = this.generateFeedback(scores);

    return {
      totalScore,
      grade,
      dimensionScores: scores,
      feedback,
      achievements: this.checkAchievements(),
      npcReactions: this.getNPCReactions(grade)
    };
  }

  calculateDimensionScore(dimension, config) {
    switch (dimension) {
      case 'infoGathering':
        return this.scoreInfoGathering();
      case 'decisionQuality':
        return this.scoreDecisionQuality();
      case 'communication':
        return this.scoreCommunication();
      case 'emergencyHandling':
        return this.scoreEmergencyHandling();
      case 'resultQuality':
        return this.scoreResultQuality();
      default:
        return 0;
    }
  }

  scoreInfoGathering() {
    let score = 0;
    const decisions = this.state.decisions;

    // 是否查看天气（+20分）
    const hasCheckedWeather = decisions.some(d => 
      d.optionId === 'check_weather' || d.nodeId === 'node_weather'
    );
    if (hasCheckedWeather) score += 20;

    // 是否查看客户记录（+15分，额外加分项）
    const hasCheckedCustomer = decisions.some(d => 
      d.optionId === 'check_customer'
    );
    if (hasCheckedCustomer) score += 15;

    // 是否去仓库询问（+15分）
    const hasAskedWarehouse = decisions.some(d => 
      d.nodeId === 'node_003'
    );
    if (hasAskedWarehouse) score += 15;

    // 信息完整度（0-50分）
    const infoCompleteness = this.state.globalState.infoCompleteness || 0;
    score += infoCompleteness * 0.5;

    return Math.min(score, 100);
  }

  scoreDecisionQuality() {
    let score = 60; // 基准分

    // 检查是否选择了最优方案（平板车+加厚篷布）
    const finalDecision = this.getFinalDecision();
    if (finalDecision.vehicleType === 'flatbed' && 
        finalDecision.rainProtection === 'heavy_duty') {
      score += 30; // 最优方案
    } else if (finalDecision.vehicleType === 'box_truck') {
      score += 20; // 次优方案（成本高但最安全）
    } else if (finalDecision.vehicleType === 'flatbed' && 
               finalDecision.rainProtection === 'standard') {
      score -= 20; // 风险方案（会导致货物受损）
    }

    // 成本效率
    const actualCost = this.state.globalState.actualCost || 0;
    const budgetCost = this.state.globalState.budget || 1500;
    if (actualCost <= budgetCost * 0.7) {
      score += 10; // 成本节约明显
    }

    return Math.max(0, Math.min(score, 100));
  }

  scoreCommunication() {
    let score = 50; // 基准分

    // 是否主动通知客户延误（+40分）
    const notifiedCustomer = this.state.decisions.some(d => 
      d.nodeId === 'node_emergency' && d.optionId === 'notify_customer'
    );
    if (notifiedCustomer) score += 40;

    // 是否向主管汇报清晰（+10分）
    const reportedToManager = this.state.decisions.some(d =>
      d.nodeId === 'node_report'
    );
    if (reportedToManager) score += 10;

    return Math.min(score, 100);
  }

  determineGrade(totalScore) {
    if (totalScore >= 90) return 'S';
    if (totalScore >= 80) return 'A';
    if (totalScore >= 70) return 'B';
    if (totalScore >= 60) return 'C';
    return 'D';
  }

  generateFeedback(scores) {
    const feedback = [];

    // 针对每个维度生成反馈
    for (const [dimension, score] of Object.entries(scores)) {
      if (score >= 80) {
        feedback.push({
          dimension,
          type: 'positive',
          message: this.getPositiveFeedback(dimension)
        });
      } else if (score < 60) {
        feedback.push({
          dimension,
          type: 'improvement',
          message: this.getImprovementFeedback(dimension)
        });
      }
    }

    return feedback;
  }

  getPositiveFeedback(dimension) {
    const messages = {
      infoGathering: '✅ 信息收集很到位，决策前收集了所有关键信息',
      decisionQuality: '✅ 决策质量优秀，在成本、风险、时效之间找到了平衡',
      communication: '✅ 沟通协调能力强，主动与客户和上级沟通',
      emergencyHandling: '✅ 应急处理得当，遇到突发情况能冷静应对',
      resultQuality: '✅ 结果优秀，货物安全送达，客户满意'
    };
    return messages[dimension] || '';
  }

  getImprovementFeedback(dimension) {
    const messages = {
      infoGathering: '⚠️ 信息收集不足，决策前应该了解更多背景信息',
      decisionQuality: '⚠️ 决策质量有待提高，需要更全面地权衡各个因素',
      communication: '⚠️ 沟通协调需要加强，遇到问题应主动沟通',
      emergencyHandling: '⚠️ 应急处理能力需要提升，突发情况下反应不够快',
      resultQuality: '⚠️ 结果不理想，需要反思决策过程中的问题'
    };
    return messages[dimension] || '';
  }

  checkAchievements() {
    const achievements = [];
    const decisions = this.state.decisions;

    // 未雨绸缪：决策前查看天气
    if (decisions.some(d => d.nodeId === 'node_weather')) {
      achievements.push({
        id: 'weather_check',
        name: '未雨绸缪',
        description: '决策前主动查看天气预报',
        icon: '🌦️'
      });
    }

    // 信息猎手：收集了95%以上的信息
    if (this.state.globalState.infoCompleteness >= 95) {
      achievements.push({
        id: 'info_hunter',
        name: '信息猎手',
        description: '收集了95%以上的关键信息',
        icon: '🔍'
      });
    }

    // 完美首秀：第一次任务就拿S级
    if (this.determineGrade(this.calculate().totalScore) === 'S') {
      achievements.push({
        id: 'perfect_debut',
        name: '完美首秀',
        description: '第一个任务获得S级评价',
        icon: '🏆'
      });
    }

    return achievements;
  }

  getNPCReactions(grade) {
    const reactions = {
      zhang_lei: {
        S: '干得漂亮，林晓。你今天让我看到了一个物流人应该有的样子。',
        A: '还行，下次记得提前看天气预报。',
        B: '勉强及格，但还有很多需要改进的地方。',
        C: '这次做得不够好，好好反思一下。',
        D: '你这样是通过不了试用期的。'
      }
    };

    return {
      zhang_lei: reactions.zhang_lei[grade] || reactions.zhang_lei.C
    };
  }

  getFinalDecision() {
    // 从决策历史中提取最终的运输方案
    const transportDecision = this.state.decisions.find(d => 
      d.nodeId === 'node_final_decision'
    );
    return transportDecision?.data || {};
  }
}
```

### 2.3 交互工具实现

#### 填单系统

**运输调度单组件 (TransportForm.vue)**

```vue
<template>
  <div class="transport-form">
    <div class="form-header">
      <h3>运输调度单</h3>
      <span class="form-number">No. {{ formNumber }}</span>
    </div>

    <div class="form-body">
      <div class="form-section">
        <h4>基本信息</h4>
        <div class="form-row">
          <label>客户名称</label>
          <input type="text" v-model="form.customerName" readonly />
        </div>
        <div class="form-row">
          <label>目的地</label>
          <input type="text" v-model="form.destination" readonly />
        </div>
        <div class="form-row">
          <label>截止时间</label>
          <input type="text" v-model="form.deadline" readonly />
        </div>
      </div>

      <div class="form-section">
        <h4>货物信息</h4>
        <div class="info-card" v-if="gameState.cargoInfo">
          <div class="info-item">
            <span class="label">品类：</span>
            <span class="value">{{ gameState.cargoInfo.type }}</span>
          </div>
          <div class="info-item">
            <span class="label">重量：</span>
            <span class="value">{{ gameState.cargoInfo.weight }} kg</span>
          </div>
          <div class="info-item">
            <span class="label">体积：</span>
            <span class="value">{{ gameState.cargoInfo.volume }} m³</span>
          </div>
          <div class="info-item warning" v-if="gameState.cargoInfo.warnings">
            <span class="label">⚠️ 注意：</span>
            <span class="value">{{ gameState.cargoInfo.warnings.join('、') }}</span>
          </div>
        </div>
        <div class="hint" v-else>
          💡 提示：你还没有收集货物信息，建议先去仓库查看
        </div>
      </div>

      <div class="form-section">
        <h4>车辆安排</h4>
        <div class="form-row">
          <label class="required">车辆类型</label>
          <select v-model="form.vehicleType" @change="onVehicleTypeChange">
            <option value="">请选择</option>
            <option value="box_truck">厢式货车</option>
            <option value="flatbed">平板车</option>
          </select>
        </div>

        <!-- 车辆信息卡片 -->
        <div class="vehicle-info-card" v-if="form.vehicleType">
          <div class="vehicle-spec">
            <div class="spec-item">
              <span class="label">载重：</span>
              <span class="value">{{ vehicleSpecs[form.vehicleType].capacity }}</span>
            </div>
            <div class="spec-item">
              <span class="label">容积：</span>
              <span class="value">{{ vehicleSpecs[form.vehicleType].volume }}</span>
            </div>
            <div class="spec-item">
              <span class="label">单价：</span>
              <span class="value">¥{{ vehicleSpecs[form.vehicleType].price }}</span>
            </div>
          </div>
          
          <!-- 容积检查提示 -->
          <div class="validation-hint" v-if="showVolumeWarning">
            ⚠️ 货物体积 {{ gameState.cargoInfo.volume }} m³，
            单车容积 {{ vehicleSpecs[form.vehicleType].volume }}，
            建议使用 {{ suggestedVehicleCount }} 辆
          </div>
        </div>

        <div class="form-row">
          <label class="required">车辆数量</label>
          <input 
            type="number" 
            v-model.number="form.vehicleCount" 
            min="1" 
            max="5"
            @input="calculateCost"
          />
        </div>

        <!-- 平板车特有：防雨措施 -->
        <div class="form-row" v-if="form.vehicleType === 'flatbed'">
          <label class="required">防雨措施</label>
          <select v-model="form.rainProtection" @change="calculateCost">
            <option value="">请选择</option>
            <option value="none">不做防护（不推荐）</option>
            <option value="standard">标准篷布（免费）</option>
            <option value="heavy_duty">加厚防水篷布（+150元/车）</option>
            <option value="plastic_wrap">塑料膜包装（+400元）</option>
          </select>
          
          <!-- 天气预警 -->
          <div class="weather-warning" v-if="gameState.hasCheckedWeather && gameState.weather.rain">
            🌧️ 天气预报显示下午有{{ gameState.weather.rainLevel }}，建议选择加厚防水篷布
          </div>
        </div>

        <div class="form-row">
          <label>出发时间</label>
          <input type="time" v-model="form.departureTime" />
        </div>
      </div>

      <div class="form-section">
        <h4>成本核算</h4>
        <div class="cost-breakdown">
          <div class="cost-item">
            <span class="label">车辆费用：</span>
            <span class="value">¥{{ costs.vehicleCost }}</span>
          </div>
          <div class="cost-item" v-if="costs.rainProtectionCost > 0">
            <span class="label">防雨措施：</span>
            <span class="value">¥{{ costs.rainProtectionCost }}</span>
          </div>
          <div class="cost-item" v-if="costs.laborCost > 0">
            <span class="label">人工费用：</span>
            <span class="value">¥{{ costs.laborCost }}</span>
          </div>
          <div class="cost-item total">
            <span class="label">总成本：</span>
            <span class="value">¥{{ costs.totalCost }}</span>
          </div>
          <div class="cost-item budget">
            <span class="label">预算：</span>
            <span class="value">¥{{ gameState.budget }}</span>
          </div>
          <div class="cost-item" :class="{ 'over-budget': isOverBudget }">
            <span class="label">{{ isOverBudget ? '超出预算：' : '节省：' }}</span>
            <span class="value">¥{{ Math.abs(gameState.budget - costs.totalCost) }}</span>
          </div>
        </div>
      </div>

      <div class="form-section">
        <h4>备注</h4>
        <textarea 
          v-model="form.notes" 
          placeholder="记录你的决策理由，这将影响你的评分..."
          rows="4"
        ></textarea>
      </div>
    </div>

    <div class="form-footer">
      <button class="btn-secondary" @click="saveAsDraft">保存草稿</button>
      <button 
        class="btn-primary" 
        @click="submitForm"
        :disabled="!isFormValid"
      >
        提交调度单
      </button>
    </div>

    <!-- 验证错误提示 -->
    <div class="validation-errors" v-if="validationErrors.length > 0">
      <div class="error-item" v-for="error in validationErrors" :key="error">
        ❌ {{ error }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TransportForm',
  props: {
    gameState: {
      type: Object,
      required: true
    },
    onSubmit: {
      type: Function,
      required: true
    }
  },
  data() {
    return {
      form: {
        customerName: '星辉服饰',
        destination: '城东物流园',
        deadline: '15:00',
        vehicleType: '',
        vehicleCount: 1,
        rainProtection: '',
        departureTime: '10:00',
        notes: ''
      },
      vehicleSpecs: {
        box_truck: {
          capacity: '5吨',
          volume: '40 m³',
          price: 1200
        },
        flatbed: {
          capacity: '10吨',
          volume: '不限',
          price: 800
        }
      },
      costs: {
        vehicleCost: 0,
        rainProtectionCost: 0,
        laborCost: 0,
        totalCost: 0
      },
      validationErrors: []
    };
  },
  computed: {
    formNumber() {
      return `TD${Date.now().toString().slice(-6)}`;
    },
    showVolumeWarning() {
      if (!this.form.vehicleType || !this.gameState.cargoInfo) return false;
      if (this.form.vehicleType === 'flatbed') return false; // 平板车无容积限制
      
      const cargoVolume = this.gameState.cargoInfo.volume;
      const vehicleVolume = 40; // 厢式货车容积
      return cargoVolume > vehicleVolume;
    },
    suggestedVehicleCount() {
      if (!this.gameState.cargoInfo) return 1;
      const cargoVolume = this.gameState.cargoInfo.volume;
      return Math.ceil(cargoVolume / 40);
    },
    isOverBudget() {
      return this.costs.totalCost > this.gameState.budget;
    },
    isFormValid() {
      return (
        this.form.vehicleType &&
        this.form.vehicleCount > 0 &&
        (this.form.vehicleType !== 'flatbed' || this.form.rainProtection) &&
        this.validationErrors.length === 0
      );
    }
  },
  methods: {
    onVehicleTypeChange() {
      // 重置防雨措施选项
      if (this.form.vehicleType !== 'flatbed') {
        this.form.rainProtection = '';
      }
      this.calculateCost();
      this.validate();
    },
    
    calculateCost() {
      // 车辆费用
      const vehiclePrice = this.vehicleSpecs[this.form.vehicleType]?.price || 0;
      this.costs.vehicleCost = vehiclePrice * this.form.vehicleCount;

      // 防雨措施费用
      const rainProtectionPrices = {
        none: 0,
        standard: 0,
        heavy_duty: 150 * this.form.vehicleCount,
        plastic_wrap: 400
      };
      this.costs.rainProtectionCost = rainProtectionPrices[this.form.rainProtection] || 0;

      // 人工费用（厢式货车需要人工装卸）
      if (this.form.vehicleType === 'box_truck') {
        const boxesPerWorker = 20; // 每人每小时装20箱
        const totalBoxes = this.gameState.cargoInfo?.boxes || 80;
        const hours = Math.ceil(totalBoxes / boxesPerWorker / 4); // 4个人
        this.costs.laborCost = hours * 4 * 50; // 每人每小时50元
      } else {
        this.costs.laborCost = 0;
      }

      // 总成本
      this.costs.totalCost = 
        this.costs.vehicleCost + 
        this.costs.rainProtectionCost + 
        this.costs.laborCost;
    },

    validate() {
      this.validationErrors = [];

      // 车辆类型必填
      if (!this.form.vehicleType) {
        this.validationErrors.push('请选择车辆类型');
      }

      // 厢式货车容积检查
      if (this.form.vehicleType === 'box_truck' && this.gameState.cargoInfo) {
        const requiredCount = this.suggestedVehicleCount;
        if (this.form.vehicleCount < requiredCount) {
          this.validationErrors.push(
            `货物体积需要至少${requiredCount}辆厢式货车`
          );
        }
      }

      // 平板车必须选择防雨措施
      if (this.form.vehicleType === 'flatbed' && !this.form.rainProtection) {
        this.validationErrors.push('平板车必须选择防雨措施');
      }

      // 天气检查：如果有雨且选择了不做防护
      if (this.gameState.hasCheckedWeather && 
          this.gameState.weather.rain && 
          this.form.rainProtection === 'none') {
        this.validationErrors.push(
          '⚠️ 天气预报有雨，不做防护会导致货物受损'
        );
      }

      // 成本检查
      if (this.isOverBudget) {
        this.validationErrors.push(
          `成本超出预算¥${this.costs.totalCost - this.gameState.budget}`
        );
      }

      return this.validationErrors.length === 0;
    },

    saveAsDraft() {
      // 保存草稿到本地存储
      localStorage.setItem('transport_form_draft', JSON.stringify(this.form));
      this.$message.success('草稿已保存');
    },

    async submitForm() {
      if (!this.validate()) {
        this.$message.error('请检查表单填写');
        return;
      }

      // 计算决策质量分数
      const decisionScore = this.calculateDecisionScore();

      // 提交数据
      const submissionData = {
        form: this.form,
        costs: this.costs,
        decisionScore,
        timestamp: Date.now()
      };

      // 调用父组件传入的回调
      await this.onSubmit(submissionData);
    },

    calculateDecisionScore() {
      let score = 0;

      // 最优方案：平板车 + 加厚篷布
      if (this.form.vehicleType === 'flatbed' && 
          this.form.rainProtection === 'heavy_duty') {
        score = 95;
      }
      // 次优方案：厢式货车
      else if (this.form.vehicleType === 'box_truck') {
        score = 85;
      }
      // 风险方案
      else if (this.form.vehicleType === 'flatbed' && 
               this.form.rainProtection === 'standard' &&
               this.gameState.weather.rain) {
        score = 50; // 会导致货物受损
      }

      // 成本效率加分
      if (!this.isOverBudget && this.costs.totalCost < this.gameState.budget * 0.7) {
        score += 5;
      }

      // 备注完整性
      if (this.form.notes.length > 20) {
        score += 5;
      }

      return Math.min(score, 100);
    }
  },
  mounted() {
    // 尝试恢复草稿
    const draft = localStorage.getItem('transport_form_draft');
    if (draft) {
      this.form = { ...this.form, ...JSON.parse(draft) };
      this.calculateCost();
    }
  }
};
</script>

<style scoped>
.transport-form {
  max-width: 800px;
  margin: 20px auto;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
  padding: 24px;
}

.form-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid #409eff;
  padding-bottom: 12px;
  margin-bottom: 24px;
}

.form-section {
  margin-bottom: 24px;
}

.form-section h4 {
  font-size: 16px;
  color: #303133;
  margin-bottom: 12px;
  padding-left: 8px;
  border-left: 3px solid #409eff;
}

.form-row {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
}

.form-row label {
  width: 120px;
  font-weight: 500;
  color: #606266;
}

.form-row label.required::after {
  content: ' *';
  color: #f56c6c;
}

.form-row input,
.form-row select {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  font-size: 14px;
}

.form-row input:focus,
.form-row select:focus {
  outline: none;
  border-color: #409eff;
}

.info-card {
  background: #f4f4f5;
  padding: 16px;
  border-radius: 4px;
  border-left: 3px solid #409eff;
}

.info-item {
  display: flex;
  padding: 4px 0;
}

.info-item .label {
  font-weight: 500;
  color: #606266;
  margin-right: 8px;
}

.info-item.warning {
  color: #e6a23c;
}

.vehicle-info-card {
  background: #ecf5ff;
  padding: 12px;
  border-radius: 4px;
  margin-top: 8px;
}

.vehicle-spec {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.spec-item {
  display: flex;
  flex-direction: column;
}

.validation-hint {
  margin-top: 12px;
  padding: 8px;
  background: #fff3cd;
  border-left: 3px solid #ffc107;
  font-size: 13px;
}

.weather-warning {
  margin-top: 8px;
  padding: 8px;
  background: #fff3cd;
  border-left: 3px solid #ffc107;
  font-size: 13px;
}

.cost-breakdown {
  background: #f9f9f9;
  padding: 16px;
  border-radius: 4px;
}

.cost-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #e4e7ed;
}

.cost-item.total {
  font-weight: bold;
  font-size: 16px;
  color: #303133;
  border-top: 2px solid #dcdfe6;
  padding-top: 12px;
  margin-top: 4px;
}

.cost-item.over-budget {
  color: #f56c6c;
}

textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  font-size: 14px;
  font-family: inherit;
  resize: vertical;
}

.form-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid #e4e7ed;
}

.btn-primary,
.btn-secondary {
  padding: 10px 24px;
  border-radius: 4px;
  font-size: 14px;
  border: none;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background: #409eff;
  color: #fff;
}

.btn-primary:hover {
  background: #66b1ff;
}

.btn-primary:disabled {
  background: #c0c4cc;
  cursor: not-allowed;
}

.btn-secondary {
  background: #fff;
  color: #606266;
  border: 1px solid #dcdfe6;
}

.btn-secondary:hover {
  border-color: #409eff;
  color: #409eff;
}

.validation-errors {
  margin-top: 16px;
  padding: 12px;
  background: #fef0f0;
  border-left: 3px solid #f56c6c;
  border-radius: 4px;
}

.error-item {
  color: #f56c6c;
  font-size: 13px;
  padding: 4px 0;
}

.hint {
  padding: 12px;
  background: #e6f7ff;
  border-left: 3px solid #1890ff;
  border-radius: 4px;
  font-size: 13px;
  color: #0050b3;
}
</style>
```

这个填单组件的设计特点：

1. **真实工作场景**：模拟真实的运输调度单
2. **智能提示**：根据已收集的信息给出提示
3. **实时验证**：填写过程中实时检查合理性
4. **成本计算**：自动计算总成本并对比预算
5. **决策记录**：要求学生记录决策理由
6. **条件显示**：根据选择动态显示相关字段

---

**文档太长，我将分成多个文件继续...**
