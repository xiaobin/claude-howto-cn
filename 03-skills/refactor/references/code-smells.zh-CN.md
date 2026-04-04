# 代码异味目录

基于 Martin Fowler 的《重构》（第二版）的全面代码异味参考。代码异味是更深层问题的症状——它们表明代码设计可能存在问题。

> "代码异味是通常对应于系统更深层问题的表面现象。" — Martin Fowler

---

## 臃肿者

代表某些已经增长到无法有效处理的代码异味。

### 长方法

**迹象：**
- 方法超过 30-50 行
- 需要滚动才能看到整个方法
- 多层嵌套
- 解释各部分的注释

**为什么不好：**
- 难以理解
- 难以独立测试
- 变更有意想不到的后果
- 内部隐藏重复逻辑

**重构方法：**
- 提取方法
- 用查询替代临时变量
- 引入参数对象
- 用方法对象替代方法
- 分解条件

**示例（之前）：**
```javascript
function processOrder(order) {
  // 验证订单（20 行）
  if (!order.items) throw new Error('No items');
  if (order.items.length === 0) throw new Error('Empty order');
  // ... 更多验证

  // 计算总计（30 行）
  let subtotal = 0;
  for (const item of order.items) {
    subtotal += item.price * item.quantity;
  }
  // ... 税、运费、折扣

  // 发送通知（20 行）
  // ... 邮件逻辑
}
```

**示例（之后）：**
```javascript
function processOrder(order) {
  validateOrder(order);
  const totals = calculateOrderTotals(order);
  sendOrderNotifications(order, totals);
  return { order, totals };
}
```

---

### 大类

**迹象：**
- 类有许多实例变量（>7-10 个）
- 类有许多方法（>15-20 个）
- 类名模糊（Manager、Handler、Processor）
- 方法未使用所有实例变量

**为什么不好：**
- 违反单一职责原则
- 难以测试
- 变更影响无关功能
- 难以复用部分

**重构方法：**
- 提取类
- 提取子类
- 提取接口

**检测：**
```
代码行数 > 300
方法数量 > 15
字段数量 > 10
```

---

### 原始类型痴迷

**迹象：**
- 使用原始类型表示领域概念（字符串表示 email，int 表示 money）
- 原始类型数组而非对象
- 类型代码的字符串常量
- 魔术数字/字符串

**为什么不好：**
- 缺少类型级别验证
- 逻辑分散在代码库中
- 容易传递错误值
- 缺少领域概念

**重构方法：**
- 用对象替代原始类型
- 用类替代类型代码
- 用子类替代类型代码
- 用状态/策略替代类型代码

**示例（之前）：**
```javascript
const user = {
  email: 'john@example.com',     // 只是字符串
  phone: '1234567890',           // 只是字符串
  status: 'active',              // 魔术字符串
  balance: 10050                 // 美分为整数
};
```

**示例（之后）：**
```javascript
const user = {
  email: new Email('john@example.com'),
  phone: new PhoneNumber('1234567890'),
  status: UserStatus.ACTIVE,
  balance: Money.cents(10050)
};
```

---

### 长参数列表

**迹象：**
- 方法有 4+ 参数
- 参数总是同时出现
- 布尔标志改变方法行为
- 经常传递 null/undefined

**为什么不好：**
- 难以正确调用
- 参数顺序混淆
- 表明方法做太多事
- 难以添加新参数

**重构方法：**
- 引入参数对象
- 保留整个对象
- 用方法调用替代参数
- 移除标志参数

**示例（之前）：**
```javascript
function createUser(firstName, lastName, email, phone,
                    street, city, state, zip,
                    isAdmin, isActive, createdBy) {
  // ...
}
```

**示例（之后）：**
```javascript
function createUser(personalInfo, address, options) {
  // personalInfo: { firstName, lastName, email, phone }
  // address: { street, city, state, zip }
  // options: { isAdmin, isActive, createdBy }
}
```

---

### 数据簇

**迹象：**
- 相同的 3+ 字段反复一起出现
- 参数总是同时传递
- 类的字段子集属于一起

**为什么不好：**
- 重复处理逻辑
- 缺少抽象
- 难以扩展
- 表明隐藏的类

**重构方法：**
- 提取类
- 引入参数对象
- 保留整个对象

**示例：**
```javascript
// 数据簇：(x, y, z) 坐标
function movePoint(x, y, z, dx, dy, dz) { }
function scalePoint(x, y, z, factor) { }
function distanceBetween(x1, y1, z1, x2, y2, z2) { }

// 提取 Point3D 类
class Point3D {
  constructor(x, y, z) { }
  move(delta) { }
  scale(factor) { }
  distanceTo(other) { }
}
```

---

## 面向对象滥用者

表明 OOP 原则使用不完整或错误的异味。

### Switch 语句

**迹象：**
- 长的 switch/case 或 if/else 链
- 多处相同的 switch
- 按类型代码 switch
- 添加新 case 需要各处变更

**为什么不好：**
- 违反开闭原则
- 变更影响到所有 switch 位置
- 难以扩展
- 通常表明缺少多态

**重构方法：**
- 用多态替代条件
- 用子类替代类型代码
- 用状态/策略替代类型代码

**示例（之前）：**
```javascript
function calculatePay(employee) {
  switch (employee.type) {
    case 'hourly':
      return employee.hours * employee.rate;
    case 'salaried':
      return employee.salary / 12;
    case 'commissioned':
      return employee.sales * employee.commission;
  }
}
```

**示例（之后）：**
```javascript
class HourlyEmployee {
  calculatePay() {
    return this.hours * this.rate;
  }
}

class SalariedEmployee {
  calculatePay() {
    return this.salary / 12;
  }
}
```

---

### 临时字段

**迹象：**
- 实例变量仅在某些方法中使用
- 条件设置的字段
- 某些 case 的复杂初始化

**为什么不好：**
- 令人困惑——字段存在但可能为 null
- 难以理解对象状态
- 表明隐藏的条件逻辑

**重构方法：**
- 提取类
- 引入 Null Object
- 用局部变量替代临时字段

---

### 拒绝遗产

**迹象：**
- 子类不使用继承的方法/数据
- 子类覆盖为空
- 继承用于代码复用而非 IS-A 关系

**为什么不好：**
- 错误的抽象
- 违反里氏替换原则
- 误导的层次结构

**重构方法：**
- 下推方法/字段
- 用委托替代子类
- 用委托替代继承

---

### 接口不同的替代类

**迹象：**
- 两个类做相似的事
- 相同概念的不同方法名
- 可以互换使用

**为什么不好：**
- 重复实现
- 无公共接口
- 难以切换

**重构方法：**
- 重命名方法
- 移动方法
- 提取超类
- 提取接口

---

## 变更阻碍者

使变更困难的异味——变更一件事需要变更其他许多事。

### 分散变化

**迹象：**
- 一个类因多种不同原因变更
- 不同区域的变更触发相同的类编辑
- 类是"上帝类"

**为什么不好：**
- 违反单一职责
- 高变更频率
- 合并冲突

**重构方法：**
- 提取类
- 提取超类
- 提取子类

**示例：**
`User` 类因以下原因变更：
- 身份验证变更
- 个人资料变更
- 计费变更
- 通知变更

→ 提取：`AuthService`、`ProfileService`、`BillingService`、`NotificationService`

---

### 霰弹手术

**迹象：**
- 一个变更需要在许多类中编辑
- 小功能需要触及 10+ 文件
- 变更分散，难以找到所有位置

**为什么不好：**
- 容易漏掉一处
- 高耦合
- 变更容易出错

**重构方法：**
- 移动方法
- 移动字段
- 内联类

**检测：**
查找：添加一个字段需要在 >5 个文件中变更。

---

### 并行继承层次结构

**迹象：**
- 在一个层次结构中创建子类需要在另一个中也创建
- 类前缀匹配（例如 `DatabaseOrder`、`DatabaseProduct`）

**为什么不好：**
- 双重维护
- 层次结构间耦合
- 容易忘记一侧

**重构方法：**
- 移动方法
- 移动字段
- 消除一个层次结构

---

## 多余者

不必要且应移除的东西。

### 过度注释

**迹象：**
- 解释代码做什么的注释
- 注释掉的代码
- 永远留着的 TODO/FIXME
- 注释中的道歉

**为什么不好：**
- 注释会撒谎（变得不同步）
- 代码应该自解释
- 死代码造成混淆

**重构方法：**
- 提取方法（名称解释内容）
- 重命名（清晰的注释替代）
- 移除注释代码
- 引入断言

**好注释 vs 坏注释：**
```javascript
// 坏：解释做什么
// 循环遍历用户并检查是否活跃
for (const user of users) {
  if (user.status === 'active') { }
}

// 好：解释为什么
// 仅活跃用户——非活跃的由清理任务处理
const activeUsers = users.filter(u => u.isActive);
```

---

### 重复代码

**迹象：**
- 多处相同的代码
- 略有变化的相似代码
- 复制粘贴模式

**为什么不好：**
- Bug 修复需要在多处进行
- 不一致风险
- 代码库膨胀

**重构方法：**
- 提取方法
- 提取类
- 提升方法（在层次结构中）
- 形成模板方法

**检测规则：**
任何重复 3+ 次的代码都应提取。

---

### 冗余类

**迹象：**
- 类存在理由不足
- 无附加值的包装器
- 过度工程的结果

**为什么不好：**
- 维护开销
- 不必要的间接层
- 复杂度无益处

**重构方法：**
- 内联类
- 折叠层次结构

---

### 死代码

**迹象：**
- 不可达的代码
- 未使用的变量/方法/类
- 注释掉的代码
- 不可能条件后的代码

**为什么不好：**
- 混淆
- 维护负担
- 减慢理解

**重构方法：**
- 移除死代码
- 安全删除

**检测：**
```bash
# 查找未使用的导出
# 查找未引用的函数
# IDE "unused" 警告
```

---

### 推测性通用性

**迹象：**
- 只有一个子类的抽象类
- "为了未来使用"的未使用参数
- 仅委托的方法
- 一个用例的"框架"

**为什么不好：**
- 复杂度无益处
- YAGNI（你不需要它）
- 难以理解

**重构方法：**
- 折叠层次结构
- 内联类
- 移除参数
- 重命名方法

---

## 耦合者

代表类之间过度耦合的异味。

### 特性依恋

**迹象：**
- 方法使用另一个类的数据比自己的多
- 大量获取另一个对象的 getter
- 数据和行为分离

**为什么不好：**
- 行为位置错误
- 封装性差
- 难以维护

**重构方法：**
- 移动方法
- 移动字段
- 提取方法（然后移动）

**示例（之前）：**
```javascript
class Order {
  getDiscountedPrice(customer) {
    // 大量使用 customer 数据
    if (customer.loyaltyYears > 5) {
      return this.price * customer.discountRate;
    }
    return this.price;
  }
}
```

**示例（之后）：**
```javascript
class Customer {
  getDiscountedPriceFor(price) {
    if (this.loyaltyYears > 5) {
      return price * this.discountRate;
    }
    return price;
  }
}
```

---

### 不当亲密

**迹象：**
- 类访问彼此的私有部分
- 双向引用
- 子类对父类了解太多

**为什么不好：**
- 高耦合
- 变更级联
- 难以修改一个而不改另一个

**重构方法：**
- 移动方法
- 移动字段
- 将双向引用改为单向
- 提取类
- 隐藏委托

---

### 消息链

**迹象：**
- 长的方法调用链：`a.getB().getC().getD().getValue()`
- 客户端依赖导航结构
- "火车残骸"代码

**为什么不好：**
- 脆弱——任何变更都破坏链
- 违反得墨忒耳定律
- 耦合到结构

**重构方法：**
- 隐藏委托
- 提取方法
- 移动方法

**示例：**
```javascript
// 坏：消息链
const managerName = employee.getDepartment().getManager().getName();

// 好：隐藏委托
const managerName = employee.getManagerName();
```

---

### 中间人

**迹象：**
- 仅委托给另一个的类
- 一半方法是委托
- 无附加值

**为什么不好：**
- 不必要的间接层
- 维护开销
- 架构混乱

**重构方法：**
- 移除中间人
- 内联方法

---

## 异味严重性指南

| 严重性 | 描述 | 行动 |
|----------|-------------|--------|
| **关键** | 阻塞开发，导致 bug | 立即修复 |
| **高** | 显著维护负担 | 在当前 sprint 修复 |
| **中** | 明显但可管理 | 计划在近期修复 |
| **低** | 小不便 | 随缘修复 |

---

## 快速检测检查清单

扫描代码时使用此检查清单：

- [ ] 任何方法超过 30 行？
- [ ] 任何类超过 300 行？
- [ ] 任何方法有超过 4 个参数？
- [ ] 任何重复的代码块？
- [ ] 任何按类型代码的 switch/case？
- [ ] 任何未使用的代码？
- [ ] 任何大量使用另一个类数据的方法？
- [ ] 任何长的方法调用链？
- [ ] 任何解释"做什么"而非"为什么"的注释？
- [ ] 任何应该是对象的原始类型？

---

## 进一步阅读

- Fowler, M. (2018). *重构：改善既有代码的设计*（第二版）
- Kerievsky, J. (2004). *重构到模式*
- Feathers, M. (2004). *有效处理遗留代码*
