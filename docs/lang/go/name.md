# **Go 常见命名单词与区别详解**

## 1. 创建相关
| 单词       | 含义           | 特点/区别                              | 场景                     | 示例                         |
| ---------- | -------------- | -------------------------------------- | ------------------------ | ---------------------------- |
| **New**    | 返回新实例     | 常用于构造函数模式，不一定涉及外部资源 | 返回对象/指针            | `NewUser(name string) *User` |
| **Create** | 创建并持久化   | 伴随副作用（写数据库、发请求等）       | 资源落地、业务创建       | `CreateUser(ctx, user)`      |
| **Build**  | 按步骤组装     | 过程型创建，可有中间状态               | 复杂对象生成、构造器模式 | `BuildRequest(cfg)`          |
| **Init**   | 初始化已有对象 | 不新建，修改现有实例初始状态           | 模块启动、复用对象       | `u.Init(name)`               |
| **Setup**  | 配置并准备可用 | 环境准备，可包含 Init 和配置           | 服务/测试准备            | `SetupServer(s)`             |
| **Make**   | Go 关键字      | 创建 slice/map/channel                 | 集合初始化               | `make(map[string]string)`    |

---

## 2. 获取相关
| 单词       | 含义              | 特点/区别                    | 场景                   | 示例                   |
| ---------- | ----------------- | ---------------------------- | ---------------------- | ---------------------- |
| **Get**    | 获取已有数据      | 主动取回，不关心数据来源     | Getter 方法、缓存      | `GetUser(id)`          |
| **Fetch**  | 获取并可能更新    | 常涉及外部请求/耗时操作      | HTTP、API 拉取         | `FetchPostsFromAPI()`  |
| **Find**   | 搜索并返回匹配项  | 查找一个或多个符合条件的对象 | 列表/集合搜索          | `FindUserByName(name)` |
| **Load**   | 从存储加载        | 来源固定（磁盘、DB、缓存）   | 文件、配置、持久化数据 | `LoadConfig(path)`     |
| **Read**   | 顺序读取          | 流/文件/网络读取，强调过程   | I/O 操作               | `ReadFile(file)`       |
| **Query**  | 数据库/结构化查询 | 可能带条件，返回集合         | SQL/搜索引擎           | `QueryUsers(filter)`   |
| **Lookup** | 键值查找          | 快速定位值，不遍历           | Map、索引查找          | `Lookup(key)`          |

---

## 3. 设置与更新
| 单词        | 含义          | 特点/区别                       | 场景                 | 示例                         |
| ----------- | ------------- | ------------------------------- | -------------------- | ---------------------------- |
| **Set**     | 直接赋值      | 一次性更新某个字段/配置         | Setter 方法          | `SetTimeout(d)`              |
| **Assign**  | 指派          | 语义比 Set 更正式，可批量       | 权限分配、变量赋值   | `AssignRole(user, role)`     |
| **Apply**   | 应用配置/操作 | 批量生效，可能有校验            | 样式、策略应用       | `ApplyConfig(cfg)`           |
| **Update**  | 局部更新      | 改动已有对象的一部分            | 数据库更新、状态更新 | `UpdateUserEmail(id, email)` |
| **Modify**  | 修改          | 同 Update，但多用于内部状态变更 | 对象变更             | `ModifySettings(s)`          |
| **Refresh** | 刷新数据      | 从源重新加载最新状态            | 缓存、列表刷新       | `RefreshCache()`             |
| **Replace** | 替换          | 完全替换原值                    | 文件替换、全量更新   | `ReplaceConfig(newCfg)`      |
| **Patch**   | 补丁式更新    | 局部改动，不全量替换            | JSON Patch、增量更新 | `PatchUserData(id, patch)`   |

---

## 4. 删除与清理
| 单词       | 含义     | 特点/区别            | 场景           | 示例                     |
| ---------- | -------- | -------------------- | -------------- | ------------------------ |
| **Delete** | 删除对象 | 逻辑/物理删除均可    | 删除用户、记录 | `DeleteUser(id)`         |
| **Remove** | 移除     | 更偏从集合中移出     | 列表移除元素   | `RemoveItem(list, item)` |
| **Drop**   | 丢弃     | 不关心状态，直接放弃 | 数据库表删除   | `DropTable(name)`        |
| **Clear**  | 清空     | 全部移除             | 清空缓存       | `ClearCache()`           |
| **Purge**  | 彻底清除 | 不可恢复             | 历史记录清理   | `PurgeLogs()`            |

---

## 5. 检查与验证
| 单词         | 含义         | 特点/区别        | 场景                   | 示例                    |
| ------------ | ------------ | ---------------- | ---------------------- | ----------------------- |
| **Check**    | 检查条件     | 简单判断         | 健康检查               | `CheckHealth()`         |
| **Verify**   | 校验真实性   | 结果需要被信任   | 签名验证               | `VerifyToken(token)`    |
| **Validate** | 校验合法性   | 多用于输入/配置  | 表单验证               | `ValidateInput(data)`   |
| **Ensure**   | 确保条件成立 | 可能包含修复步骤 | 创建目录（不存在则建） | `EnsureDirExists(path)` |
| **Confirm**  | 确认         | 用户交互式确认   | 弹窗确认               | `ConfirmAction()`       |
| **Test**     | 测试         | 单元/集成测试    | 断言、用例             | `TestConnection()`      |

---

## 6. 转换与处理
| 单词            | 含义             | 特点/区别         | 场景                       | 示例                   |
| --------------- | ---------------- | ----------------- | -------------------------- | ---------------------- |
| **Convert**     | 类型转换         | 保留等价数据      | `int` 转 `string`          | `ConvertToString(val)` |
| **Transform**   | 结构转换         | 结构或形态改变    | JSON → Struct              | `TransformUserDTO(u)`  |
| **Cast**        | 强制类型转换     | 低层级语义，少用  | 指针类型转换               | `CastToType(x)`        |
| **Map**         | 映射             | 集合元素映射转换  | Map 函数                   | `MapStrings(list, fn)` |
| **Marshal**     | 序列化           | Go → JSON/XML/... | `json.Marshal(v)`          |                        |
| **Unmarshal**   | 反序列化         | JSON/XML/... → Go | `json.Unmarshal(data, &v)` |                        |
| **Serialize**   | 序列化（通用）   | 数据结构 → 字节流 | `Serialize(obj)`           |                        |
| **Deserialize** | 反序列化（通用） | 字节流 → 数据结构 | `Deserialize(data)`        |                        |

---

## 7. 控制与执行
| 单词        | 含义       | 特点/区别           | 场景                  | 示例 |
| ----------- | ---------- | ------------------- | --------------------- | ---- |
| **Run**     | 执行主任务 | 主流程执行          | `RunServer()`         |      |
| **Do**      | 执行某操作 | 一般性执行          | `DoRequest()`         |      |
| **Exec**    | 执行命令   | 系统/SQL/外部程序   | `ExecCommand(cmd)`    |      |
| **Execute** | 执行       | 同 Exec，语义更正式 | `ExecutePlan()`       |      |
| **Process** | 处理       | 包含多个步骤        | `ProcessMessage(msg)` |      |
| **Operate** | 操作       | 偏业务操作          | `OperateMachine()`    |      |

---

## 8. 常见布尔前缀
| 前缀               | 含义      | 场景                | 示例              |
| ------------------ | --------- | ------------------- | ----------------- |
| **Is**             | 是否      | 状态判断            | `IsActive()`      |
| **Has**            | 拥有/存在 | 对象内部状态        | `HasPermission()` |
| **Can**            | 能够/可行 | 对象 能力/条件      | `CanRetry()`      |
| **Should**         | 应该/推荐 | 操作 规则/策略/推荐 | `ShouldUpdate()`  |
| **Allow**          | 允许/可以 | 检查外部规则或权限  | `AllowGuest()`    |
| **Enable/Disable** | 开关      | 模块启停            | `EnableCache()`   |

---







## 9. 常用结构体后缀（概念区分）

| 后缀         | 语义     | 区别              | 示例            |
| ------------ | -------- | ----------------- | --------------- |
| **Info**     | 信息快照 | 概览/元数据       | `FileInfo`      |
| **Data**     | 核心数据 | 原始/传输数据     | `SensorData`    |
| **Detail**   | 详细数据 | 完整业务数据      | `OrderDetail`   |
| **Stats**    | 统计数据 | 聚合/分析         | `UserStats`     |
| **Summary**  | 摘要     | 简要报告          | `OrderSummary`  |
| **Record**   | 记录     | 历史条目          | `LogRecord`     |
| **Entry**    | 集合项   | 列表元素          | `QueueEntry`    |
| **Config**   | 配置     | 参数集合          | `DBConfig`      |
| **Param**    | 参数     | 输入参数          | `LoginParam`    |
| **Request**  | 请求     | API 输入          | `LoginRequest`  |
| **Response** | 响应     | API 输出          | `LoginResponse` |
| **Result**   | 结果     | 操作输出          | `QueryResult`   |
| **State**    | 状态     | 当前状态          | `TaskState`     |
| **Status**   | 状态码   | 枚举化状态        | `JobStatus`     |
| **Model**    | 数据模型 | ORM/持久化        | `UserModel`     |
| **Entity**   | 业务实体 | DDD 核心对象      | `UserEntity`    |
| **Profile**  | 档案     | 配置概要/用户资料 | `UserProfile`   |
| **Meta**     | 元数据   | 数据的描述信息    | `ImageMeta`     |

---







1. 

	- **item / elem**：更通用的集合元素，没有强调记录或条目性质。
	- **node**：强调数据结构中的节点（树、链表、图）。
	- **record**：更偏数据库或日志上下文，比 entry 更正式。
	- **entry**：介于 item 和 record 之间，既可以是集合元素，也可以是日志/配置条目。

	

info 内容

meta 描述数据的属性、上下文





| **词汇**                                                     | **含义**                 | **使用场景**                                 | **示例命名**                                                 |
| ------------------------------------------------------------ | ------------------------ | -------------------------------------------- | ------------------------------------------------------------ |
| **data**                                                     | 原始数据，可处理/计算    | 内存存储、数据库记录、文件内容、网络 payload | userData, configData, rawData                                |
| **info**                                                     | 信息、描述性内容、元数据 | 日志信息、状态信息、事件信息、展示           | userInfo, serverInfo, logInfo                                |
| **val / value**                                              | 值，单个变量或临时数据   | 函数内部、循环、临时存储                     | val := 42, maxValue                                          |
| **item / elem / entry**                                      | 集合中的元素             | 列表、切片、map 遍历                         | for _, item := range list, entry := record                   |
| **node**                                                     | 节点                     | 树结构、图结构、链表                         | treeNode, node.next                                          |
| **buf / buffer**                                             | 缓冲区                   | I/O、网络、临时存储                          | buf := make([]byte, 1024)                                    |
| **tmp / temp**                                               | 临时变量                 | 临时计算、临时存储                           | tmp := processData()                                         |
| **id / uid / gid / sid**                                     | 唯一标识符               | 用户ID、组ID、会话ID                         | userID, sessionID                                            |
| **ts / timestamp**                                           | 时间戳                   | 记录事件时间                                 | createdAt, updatedAt, expireAt                               |
| **list / slice / arr**                                       | 有序集合                 | 切片、数组                                   | userList, itemsSlice                                         |
| **map / dict**                                               | 键值对集合               | 内存映射、缓存                               | userMap, configMap                                           |
| **set**                                                      | 唯一集合                 | 唯一值存储                                   | idSet, uniqueKeys                                            |
| **queue / stack / chan**                                     | 队列/栈/通道             | 并发任务、消息处理                           | jobQueue, eventChan                                          |
| **ctx**                                                      | 上下文                   | 超时、取消、Trace ID                         | ctx := context.Background()                                  |
| **cfg / config / opts / options**                            | 配置或选项               | 系统配置、函数参数                           | dbConfig, clientOpts                                         |
| **msg / message**                                            | 消息                     | 网络通信、事件、日志                         | msg := getMessage(), eventMessage                            |
| **payload / body**                                           | 数据载荷                 | HTTP请求、RPC消息                            | payload := req.Body                                          |
| **event**                                                    | 事件                     | 系统事件、消息总线                           | userEvent, jobEvent                                          |
| **record / entry**                                           | 数据记录                 | 数据库行、日志记录                           | logEntry, dbRecord                                           |
| **status / state**                                           | 状态                     | 对象状态、任务状态                           | taskStatus, serverState                                      |
| **flag**                                                     | 标志                     | 条件控制                                     | isEnabled, hasError                                          |
| **ready / active / enabled / disabled**                      | 布尔状态                 | 启用、活动标志                               | isReady, featureEnabled                                      |
| **ok / success / fail / valid**                              | 检查结果                 | 函数返回值、校验                             | ok, err := check()                                           |
| **Run / Execute / Perform / Do**                             | 执行动作                 | 任务、命令、流程                             | RunServer(), ExecuteJob(), PerformMigration(), doRequest()   |
| **Start / Stop / Shutdown / Close**                          | 生命周期                 | 启动/停止服务或资源                          | StartWorker(), StopServer(), Shutdown(), CloseFile()         |
| **Get / Fetch / Find / Lookup / Search / List / Scan**       | 读取/查询                | 内存、缓存、数据库、远程                     | GetUser(), FetchProfile(), FindItem(), LookupByKey(), SearchUsers(), ListOrders(), ScanRows() |
| **Set / Update / Add / Remove / Delete / Drop / Append / Push / Pop** | 修改/增删                | 内存、数据库、集合                           | SetConfig(), UpdateUser(), AddItem(), RemoveFromCache(), DeleteRecord(), DropTable(), AppendSlice(), PushJob(), PopItem() |
| **Parse / Decode / Unmarshal / Encode / Marshal**            | 解析/序列化              | 文本、JSON、Protobuf                         | ParseURL(), DecodeJSON(), UnmarshalProto(), EncodeBase64(), MarshalJSON() |
| **Convert / Transform / Format / Normalize / Sanitize**      | 转换/格式化              | 数据转换、格式化、清理                       | ConvertMillis(), TransformData(), FormatTime(), NormalizeEmail(), SanitizeHTML() |
| **Check / Validate / Verify / Ensure / Assert / Confirm / Guard** | 校验/检查                | 输入验证、业务规则、断言                     | ValidateInput(), VerifySignature(), EnsureDirExists(), AssertEqual(), ConfirmAction(), GuardPermission() |
| **Send / Post / Publish / Emit / Push / Receive / Read / Pull / Subscribe / Consume** | 消息/通信                | 网络请求、事件、消息队列                     | SendRequest(), PostMessage(), PublishEvent(), EmitSignal(), PushJob(), ReceiveMessage(), ReadFrame(), PullUpdates(), SubscribeChannel(), ConsumeQueue() |
| **Handle / Process / Serve / Dispatch**                      | 处理                     | 请求/任务/事件                               | HandleRequest(), ProcessBatch(), ServeHTTP(), DispatchJob()  |
| **Schedule / Enqueue / Work / Await / Wait**                 | 并发/调度                | 任务调度、goroutine管理                      | ScheduleJob(), EnqueueTask(), Worker.Do(), AwaitCompletion(), WaitAll() |