# **Go 命名参考手册（完整版）**







## **1. 基本规则**





- **小驼峰**：内部使用（未导出）变量、函数。
- **大驼峰**：导出（可被外部包使用）变量、函数、结构体、接口。
- **全大写 + 下划线**：少用，一般用于常量或枚举。
- **包名**：短小、小写、无下划线，避免与标准库冲突。
- **避免冗余**：不重复类型信息，例如 userStruct → user。





------





## **2. 变量命名**





**特点**：短小清晰，表达语义。





### **2.1 通用变量**





- **data**：通用数据。
- **val / value**：值。
- **item / elem / entry**：集合项。
- **node**：节点。
- **buf / buffer**：缓冲区。
- **tmp / temp**：临时变量。







### **2.2 状态/标志**





- **ok**：是否成功。
- **ready**：准备好。
- **enabled / disabled**：启用/禁用。
- **valid / invalid**：有效/无效。
- **dirty**：数据已修改但未同步。







### **2.3 计数/索引**





- **i, j, k**：循环变量。
- **count / num / total**：计数。
- **idx / index**：索引。







### **2.4 时间/标识**





- **ts**：时间戳。
- **createdAt / updatedAt / expireAt**：时间字段。
- **id / uid / gid / sid**：各种标识符。







### **2.5 集合**





- **list / slice**：切片。
- **map**：映射。
- **set**：集合。
- **queue / stack / chan**：队列、栈、通道。







### **2.6 上下文/配置**





- **ctx**：上下文。
- **cfg / conf**：配置。
- **opts / options**：选项。





------





## **3. 常量命名**





**特点**：表达固定值、枚举、默认参数。



- **枚举/状态**：

```
const (
    StatusOK    = 200
    StatusError = 500
)
```

**环境/配置**：

```
const (
    EnvProduction = "production"
    EnvDev        = "dev"
)
```

**默认值**：

```
const DefaultTimeout = 5 * time.Second
```

**说明**：导出常量一般用驼峰（如 DefaultTimeout），内部可全大写。

## **4. 函数/方法命名**





**特点**：动词开头，清楚表达行为。





### **4.1 创建/销毁**





- **New**：构造函数，如 NewClient。
- **Create**：创建业务对象或资源。
- **Init**：初始化。
- **Destroy / Dispose / Close**：销毁或关闭资源。







### **4.2 启动/停止/运行**





- **Start**：启动服务或任务。
- **Stop / Shutdown**：停止。
- **Run**：运行长期任务或主循环。
- **Execute / Exec**：执行一次性命令或任务。
- **Perform**：执行过程或计算，常用于有步骤的执行。
- **Do**：灵活内部执行，一般不导出。





**区别**：



- **Run**：持续运行，例如服务或 goroutine。
- **Execute**：执行一次任务。
- **Perform**：完成业务逻辑或流程。
- **Do**：内部实现，可用于测试或小功能。







### **4.3 读取/查询**





- **Get**：获取值。
- **Fetch**：拉取，常表示远程或缓存。
- **Find / Lookup / Search**：查找。
- **List**：列出集合。







### **4.4 修改/更新**





- **Set**：设置值。
- **Update**：更新数据。
- **Add / Append / Push**：增加。
- **Remove / Delete / Drop / Pop**：删除。







### **4.5 转换/处理**





- **Parse**：解析字符串或格式。
- **Convert / Transform**：转换类型或数据。
- **Serialize / Marshal / Unmarshal**：序列化/反序列化。







### **4.6 校验/检查**





- **Check**：检查状态。
- **Validate**：验证正确性。
- **Verify**：验证真实性。
- **Ensure / Must**：确保条件成立。







### **4.7 状态查询**





- **IsXxx / HasXxx / CanXxx / ShouldXxx**：布尔返回值。







### **4.8 工具方法**





- **Compute / Calculate**：计算。
- **Build / Compose**：构建。
- **Generate**：生成内容。
- **Handle / Process**：处理。





------





## **5. 结构体命名**





**特点**：名词或名词短语，表达实体、概念、容器。



- **实体模型**：User、Order、Product。
- **容器/管理器**：Cache、Pool、Manager。
- **请求/响应**：Request、Response、Payload。
- **配置/选项**：Config、Options、Settings。
- **服务/客户端**：Service、Client、Server、Handler。





**示例**：

```
type User struct {
    ID    int
    Name  string
    Email string
}
```



## **6. 接口命名**





**特点**：表示一种能力或行为，常用 **-er** 结尾。



- **Reader / Writer / Closer / Seeker**：标准库风格。
- **Fetcher / Processor / Notifier / Logger**：行为。
- **Service 接口**：UserService、PaymentProcessor。





**示例**：

```
type Reader interface {
    Read(p []byte) (n int, err error)
}

type UserService interface {
    CreateUser(ctx context.Context, u *User) error
}
```





## **7. 包名**





- **规则**：小写、单数、简短。
- **示例**：cache、config、client、server、http、json。
- **避免**：下划线、复数、与标准库重复。





------





## **8. 常见命名建议与反例**





- **避免重复类型**：userStruct → user。
- **避免含糊**：data1, data2 → 改为 rawData, parsedData。
- **错误命名**：HandleAllThing() → HandleEvents()。
- **接口命名**：避免 IUserService 前缀。



