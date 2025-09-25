# Go 命名规范速查

> 本文是一个可直接放入团队仓库的命名规范速查表。内容覆盖：**通用词汇、分层、HTTP/DB、并发、I/O、观测、鉴权、测试/CLI、动词清单、布尔前缀、Initialism 规则、常用缩写、变量/接收者约定、常见易混词区别**。  
> 目标：**简洁统一、见名知意、贴合 Go 习惯**。

# 使用方式
- **决策顺序**：先判定“层”（handler/service/repo/dao）→ 再选“语义词”（data/info/detail/meta 等）→ 再套“动词或后缀”（Get/With/ID 等）。
- **命名三问**：这个名字是否最短？是否表达最清楚？是否与项目其它部分一致？

# 总体原则
- 包名小写单数，无下划线：`handler`, `service`, `repo`, `cache`, `cctx`
- 结构体/接口用名词（接口优先 *er/able*）：`UserService`, `Reader`, `Cacheable`
- 函数名用动词或动宾：`NewClient`, `WithTimeout`, `GetUser`
- 变量名短而清：`cfg`, `ctx`, `req`, `resp`, `db`, `rdb`, `mu`
- 集合用复数：`users`, `items`, `records`
- Initialism 全大写：`userID`, `parseURL`, `toJSON`, `HTTPClient`

# 场景词汇表（精炼版）

## 信息语义
| 词     | 含义            | 示例                         | 区别               |
| ------ | --------------- | ---------------------------- | ------------------ |
| data   | 原始/通用数据   | `rawData`, `userData`        | 最泛化，不承诺结构 |
| info   | 概要信息        | `UserInfo`, `BuildInfo`      | 字段少、展示友好   |
| detail | 详细信息        | `OrderDetail`, `ErrorDetail` | 相比 info 更完整   |
| meta   | 元数据/附加属性 | `RequestMeta`, `MetaMap`     |                    |

**Info = “给人看的业务概要”，Meta = “给系统看的附加上下文”**。

## 配置与参数

| 词         | 含义     | 示例                      | 区别            |
| ---------- | -------- | ------------------------- | --------------- |
| config/cfg | 静态配置 | `AppConfig`, `dbCfg`      | 启动期/系统级   |
| option/opt | 可选项   | `WithTimeout`, `opts`     | 调用时覆盖/链式 |
| param      | 单个参数 | `urlParam`, `pathParam`   | 入参的一个键值  |
| args       | 参数集合 | `cmdArgs`, `reqArgs`      | 多参数/可变参   |
| flag       | 标志位   | `debugFlag`, `dryRunFlag` | 典型布尔/CLI    |

## 时间与上下文
| 词                | 含义            | 示例                            |
| ----------------- | --------------- | ------------------------------- |
| ctx               | 上下文          | `ctx := context.Background()`   |
| deadline/timeout  | 截止/超时       | `ctxDeadline`, `httpTimeout`    |
| ts/timestamp      | 时间点          | `createdTs`, `updatedAt`        |
| start/end         | 起止时间        | `startTime`, `endTime`          |
| ttl/expire        | 生命周期/过期点 | `cacheTTL`, `expireAt`          |
| duration/interval | 时长/间隔       | `retryDuration`, `tickInterval` |

## 网络 / HTTP / RPC
| 词                   | 含义           | 示例                                   |
| -------------------- | -------------- | -------------------------------------- |
| url/uri              | 地址           | `apiURL`, `baseURI`                    |
| host/addr/port       | 主机/地址/端口 | `dbHost`, `serverAddr`, `httpPort`     |
| path/route/method    | 路径/路由/方法 | `urlPath`, `apiRoute`, `httpMethod`    |
| header/body/query    | 头/体/查询     | `reqHeader`, `respBody`, `queryParams` |
| client/server(srv)   | 客户端/服务端  | `httpClient`, `rpcSrv`                 |
| handler/middleware   | 处理器/中间件  | `userHandler`, `authMiddleware`        |
| mux/router           | 复用器/路由器  | `httpMux`, `router`                    |
| session/cookie/token | 会话/令牌      | `sessionID`, `csrfToken`               |

## 分层/架构
| 词                   | 角色           | 示例                                     |
| -------------------- | -------------- | ---------------------------------------- |
| handler              | HTTP 入口      | `UserHandler`                            |
| service/svc          | 业务逻辑       | `UserService`                            |
| repo                 | 仓储抽象       | `UserRepo`                               |
| dao                  | 直连表/ORM     | `userDAO`                                |
| store                | 泛存储         | `sessionStore`                           |
| manager/mgr          | 管理器         | `connManager`                            |
| client               | 外部依赖客户端 | `DBClient`, `RedisClient`                |
| factory/builder      | 构造/链式      | `clientFactory`, `queryBuilder`          |
| adapter/proxy/plugin | 适配/代理/插件 | `httpAdapter`, `authProxy`, `otelPlugin` |

## 数据库 / 缓存
| 词                  | 含义       | 示例                               |
| ------------------- | ---------- | ---------------------------------- |
| db/sql              | 数据库/SQL | `db *sql.DB`, `sqlStmt`            |
| tx                  | 事务       | `tx`, `beginTx`                    |
| row/rows            | 行/多行    | `row`, `rows`                      |
| table/column/schema | 表/列/模式 | `userTable`, `colName`, `dbSchema` |
| index/idx           | 索引       | `userIdx`                          |
| cache               | 缓存       | `userCache`, `cacheEntry`          |
| query/exec          | 查/执      | `queryUser`, `execStmt`            |
| migrate/seed        | 迁移/灌数  | `migrate`, `seedData`              |

## 并发 / 同步
| 词             | 含义           | 示例                     |
| -------------- | -------------- | ------------------------ |
| go/goroutine   | 协程           | `go fn()`, `goroutineID` |
| chan/ch        | 通道           | `msgCh`, `doneCh`        |
| queue/job/task | 队列/作业/任务 | `jobQueue`, `taskID`     |
| worker/pool    | 工作者/池      | `workerPool`, `poolSize` |
| sync/mutex/mu  | 同步/锁        | `mu sync.Mutex`          |
| rwmu/rwlock    | 读写锁         | `var rwmu sync.RWMutex`  |
| once           | 单次           | `sync.Once`              |
| atomic         | 原子           | `atomic.AddInt64`        |
| wg             | 等待组         | `var wg sync.WaitGroup`  |

## 文件 / I-O
| 词            | 含义           | 示例                            |
| ------------- | -------------- | ------------------------------- |
| file/dir/path | 文件/目录/路径 | `logFile`, `workDir`, `absPath` |
| reader/writer | 读/写器        | `bufReader`, `fileWriter`       |
| stream        | 流             | `inputStream`                   |
| buffer/buf    | 缓冲           | `readBuf`, `bytesBuf`           |
| loader/saver  | 加载/保存      | `configLoader`, `stateSaver`    |

## 观测 / 可观测性
| 词                      | 含义             | 示例                            |
| ----------------------- | ---------------- | ------------------------------- |
| log/logger              | 日志/记录器      | `logger.Info`, `appLogger`      |
| trace/tracer/span       | 追踪/片段        | `traceID`, `tracer`, `rootSpan` |
| metric/metrics          | 指标             | `httpMetrics`, `dbMetrics`      |
| stats                   | 统计             | `callStats`                     |
| counter/gauge/histogram | 计数/仪表/直方图 | Prom 指标名                     |
| timer/profiler          | 计时/剖析        | `execTimer`, `pprof`            |

## 安全 / 认证
| 词                     | 含义           | 示例                             |
| ---------------------- | -------------- | -------------------------------- |
| auth/user/account      | 认证/用户/账户 | `authUser`, `accountID`          |
| role/permission/policy | 角色/权限/策略 | `userRole`, `hasPermission`      |
| secret/cred            | 密钥/凭证      | `apiSecret`, `dbCred`            |
| token/jwt/cert         | 令牌/JWT/证书  | `jwtToken`, `tlsCert`            |
| signature/hash/salt    | 签名/哈希/盐   | `requestSignature`, `sha256Hash` |

## 测试 / CLI
| 词                    | 含义                | 示例                                          |
| --------------------- | ------------------- | --------------------------------------------- |
| test/case/suite       | 测试/用例/套件      | `TestXxx`, `testCase`, `testSuite`            |
| mock/fake/stub        | 模拟/伪造/存根      | `mockRepo`, `fakeClient`                      |
| assert/expect         | 断言/期望           | `assert.Equal`, `expectErr`                   |
| cmd/cli/exec/run/exit | 命令/执行/运行/退出 | `rootCmd`, `execCmd`, `runServer`, `exitCode` |
| arg/flag/env          | 参数/标志/环境      | `cmdArgs`, `helpFlag`, `envVar`               |

# 常用动词清单（函数名）
- **构造/初始化**：`New`, `Init`, `Open`, `Load`
- **链式配置**：`WithXxx`
- **CRUD**：`Get`, `Create`, `Update`, `Delete`, `List`
- **判断**：`Is`, `Has`, `Can`, `Should`
- **执行**：`Do`, `Run`, `Execute`, `Call`
- **生命周期**：`Start`, `Stop`, `Serve`, `Close`, `Shutdown`
- **转换/序列化**：`Parse`, `Format`, `Encode`, `Decode`, `Marshal`, `Unmarshal`
- **资源**：`Acquire`, `Release`
- **其他**：`Clone`, `Merge`, `Split`, `Validate`

# 布尔前缀规范（强烈建议）
- `isActive`, `isValid`
- `hasError`, `hasPermission`
- `canRetry`, `canAccess`
- `shouldLog`, `shouldRetry`
- `featureEnabled`, `loggingEnabled`

# Initialism（缩写）大小写规则
> 在驼峰命名中保持缩写**全大写**：  
> ✅ `userID`, `parseURL`, `toJSON`, `HTTPClient`  
> ❌ `userId`, `parseUrl`, `HttpClient`

**常见缩写**：`API, ASCII, CPU, CSS, DNS, EOF, GUID, HTML, HTTP, HTTPS, ID, IP, JSON, OAuth, QPS, RAM, RPC, SLA, SQL, SSH, TCP, TLS, UDP, UI, UID, URI, URL, UTF8, UUID, XML, CSRF, XSRF`

# 变量与接收者约定（约定俗成）
- `ctx`：`context.Context`（总是第一个参数）
- `r`, `w`：`*http.Request` / `http.ResponseWriter`
- `db`, `rdb`：数据库 / Redis 客户端
- `mu`, `rwmu`：互斥锁 / 读写锁
- `wg`：`sync.WaitGroup`
- `tx`：事务
- `i`, `j`, `k`, `idx`：循环索引
- `buf`, `b`, `tmp`：缓冲/临时

# 易混词速辨
- **Data / Info / Detail / Meta**：原始 / 概要 / 细节 / 附加属性  
- **Config / Option**：静态初始化 / 调用期可选项（`WithXxx`）  
- **Repo / DAO / Store**：仓储抽象 / 表级访问 / 泛存储  
- **URL / Host / Addr / Path / Route**：完整地址 / 主机 / 地址(host+port) / 路径 / 路由规则  
- **Timeout / Deadline**：相对时长 / 绝对时间点

# 示例片段（可复制）
```go
// 包与分层
package handler // service, repo, dao, cache, cctx

// 结构体与接口
type UserService interface {
    GetUser(ctx context.Context, userID int64) (*UserInfo, error)
}

type userService struct {
    repo UserRepo
}

func NewUserService(repo UserRepo) UserService {
    return &userService{repo: repo}
}

type UserRepo interface {
    GetByID(ctx context.Context, userID int64) (*UserRecord, error)
}
```









| **层**         | **入参结构体**      | **出参结构体**     | **说明**                        |
| -------------- | ------------------- | ------------------ | ------------------------------- |
| Handler (HTTP) | XxxRequest          | XxxResponse        | 只在传输层使用 Request/Response |
| Service        | XxxInput            | XxxOutput          | 业务语义、与 HTTP 解耦          |
| Repo/DAO       | XxxArgs / XxxFilter | XxxRow / XxxRecord | 数据访问口吻，贴近 DB           |

- 入参统一 Args；查询条件放 Filter，排序 Sort，分页 Page。
- 出参用 Row/Record；**不要**在 DAO 层返回领域模型。
- Patch*Args 用指针表示“是否变更该列”。
- 列举枚举：type UserSortBy string（或 iota 枚举）。