# 事件语义与验证模式

按目标涉及的部分读取；以下是决策清单，不是要求每次实现所有机制。

## 事实与成功条件

| 观测事实 | 合理语义 | 不能推导 |
|---|---|---|
| 按钮点击 | clicked / confirmed | 请求成功、功能完成 |
| 原网络请求发起 | submitted / requested | 服务端持久化成功 |
| 原请求成功回调 | finished(result=success)，依原接口真实保证 | 后续生成完成 |
| 原保存成功回调 | downloaded/saved，依契约 | 用户从相册再次打开 |
| 外部分享调用 | share_stage=handoff_request | 外部平台发布成功 |
| 原内存赋值 | updated + persistence_scope=memory | 后端保存成功 |
| 实际结果数据可用 | ready | 已选中并应用 |

稳定错误码用分类/业务 code，不发送 raw error、URL、聊天正文、姓名等无必要原文。不为本次需求扩大全局身份或用户数据采集。

## 未知、空态、画像

初始未加载是 unknown；有效读取后列表为空才是 empty。loaded_count 只数真实数据，不能把占位或尚未分页数据当全量。状态变化需要已知基线；初次读取不要伪造 populated → empty。

person 属性是用户当前已确认状态，event 属性是这次事实的快照。资料成功后可以通过现有 PostHog 同步入口刷新画像，失败不能把意图写成事实。收藏存在性 flag 需有效读取；账号切换后不能把旧响应写到新用户。缺少可靠 owner 信号时明确限制。

## 入口与上下文

不要从当前 Tab 反推详情入口，复用页面可能来自个人页、发现页、通知或返回栈。

优先复用现有 tracking context 和 analytics-only extra properties。区分页面 surface 与入口 entry_source。保留父内容 ID、商品 ID、位置和会话关联；不可因补新入口来源覆盖既有 source/surface/version。

核对属性合并顺序：权威 surface、位置、版本不能被泛化 extra 属性覆盖。optional context 为空时保留旧行为，不能为了新埋点改变 UI 或业务回调目标。索引 0 基/1 基依既有契约分别处理，不能全局“统一”。

## 页面、面板与可见性

- 禁止在页面模型或控制器构造时发送 page_viewed；它可能被 eager 构造或在覆盖层后初始化。
- 保活 Tab 可能只有首次挂载通知，切回须利用既有进入信号；避免首次双报。
- 检查 引导、登录、弹层、全屏等覆盖层与应用活跃状态；不要只确认 view 已 mount。
- 面板有效内容曝光与“展开按钮被点击”分开；空态、加载、错误不记成内容 viewed。
- 退出原因只能表示真实已知原因，未知 dismiss 不假定 close_button。

## 商品曝光、点击与会话

先写清当前需求的可见比例、连续时长、去重 scope、session 重置条件；比例/时长不是通用常量。其他功能的阈值不能直接套用。

曝光依据真实 viewport 与裁剪交集。预加载、图片下载完成、懒加载容器挂载、屏外 layout 都不能单独当曝光。后台、遮挡、切 Tab、路由失活会中断连续可见计时；是否重返可重新累计依契约。用 analytics observer，不改滚动、加载或导航行为。

去重 key 和重访语义按具体 surface 定义，不能只靠 product_id 跨页面全局压掉。网格可能按 session + surface + item occurrence 去重，Gallery 可能每次稳定选中都生成新 impression。分别用往返序列验证，不能用一个全局 Set 套所有 surface。点击关联“这次展示”的上下文。点击本身不必补造曝光：若未达曝光门槛，`has_impression=false`、可选的曝光关联 ID 省略；具体字段沿当前契约，不硬编码。

收藏成功在原权威成功分支采集；失败回滚不能报成功，也不能为消除 race 修改原收藏状态机。analytics 的 operation ID 应匹配当次回调，不能在 await 后读取可能已被覆盖的全局当前 context。

## 异步关联

根据实际需要选 operation_id 或 session_id，不把三者套在每个事件。记录发起账号；完成时核对 owner/已有会话代次，丢弃旧账号统计，不阻断业务处理。

任务 ready 要依原流程确认的真实新结果，不能只依据传输状态、旧结果 URL 或进度 100%。applied 依原成功保存和当次结果匹配。保留原业务 session/schema，用 analytics 独立关联。

自动循环播放可以按每会话首轮完成记一次，同时让原轮播继续运行；不能为埋点停止播放或改变时长。

## 最小回归矩阵

按事件选择适用项：正常/失败/取消/无变化；首次/重复/返回；unknown/empty/populated；旧/新 context；不同入口与同商品多 surface；异步晚到/账号切换；隐藏/遮挡/后台/重返；成功回调重复；已排除模块。

测试断言事件次数、完整属性值及类型、缺省属性确实不存在、关联 ID 正确、去重和业务副作用保持原样。用 Fake clock 验证时长，不用长 sleep。文本扫描只能辅助发现 magic string/残留引用，不能代替语义测试。
