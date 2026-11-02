# 仅 PostHog 改动边界

## 判断单位是 diff hunk，不是文件路径

账号管理模块 中可能有合法 capture 调整；`tracking/` 中也可能写出非法的业务干预。不能靠白名单目录证明范围正确。

| 变化 | 处理 |
|---|---|
| 添加 typed event/key、修改目标事件属性 | 允许；检查全局类型/名称兼容 |
| 在现有成功分支追加 capture，读取该分支已有值 | 允许；不移动原语句，不增加请求 |
| 用专用 helper 保存操作 ID、计时和去重状态 | 允许；不能被功能消费 |
| 为现有业务方法增加 optional callback/default 参数 | 禁止；即使旧调用仍可编译 |
| 为传 entry_source 修改路由或数据访问等业务接口 | 禁止；使用已有 analytics carrier，否则记录缺口 |
| 新增 tracking-only API 的属性参数 | 允许；先核实该 API 确实只服务 analytics，调用方仍不改变业务接口 |
| 为提高成功率新增 retry、改变任务调度、缓存或回滚 | 禁止 |
| 修改业务 session/model 序列化字段以存埋点 ID | 禁止；如确需持久化，使用独立 analytics store |
| 为观察任务结果给业务清理方法增加结果参数 | 禁止；在已有可见值处传给 tracking 对象 |
| 新增观察器读取现有结果 | 允许；只改 analytics 状态，不替换业务 delegate，不影响原通知传递 |
| 让业务 await analytics、用 analytics guard 跳过功能 | 禁止 |
| 页面加 tracking 专用生命周期观察 | 允许；验证 modifier 生命周期和无业务状态写入 |
| 修改原保存回调、提前/延后 toast/dismiss | 禁止 |
| 原成功回调内加 capture | 允许；回调原签名和业务语句保持原样 |
| 为无回调的保存 API 替换 selector/delegate | 禁止；若无既有完成信号，只能记录意图或缺口 |
| 因名称含 deleteAccount 就上报删除成功 | 禁止；查看实现是否真的调用删除接口 |

## logout 示例

原接口：

```typescript
function logout() {
    // 原有同步清理和身份处理
}
```

不能改为：

```typescript
function logout(beforeIdentityReset?: () => void) { /* ... */ }
```

可以在原调用点保留 `logout()` 调用并添加埋点。但只有确实为同步、无失败返回路径的清理，且账号归属已核实，才可在返回后报告约定的清理完成。若方法是异步发起器、有早退或已 reset PostHog 身份，直接在下一行报 completed 不成立。利用原有权威信号；没有就仅记录 confirm/requested。不得为捕获旧身份重排清理或修改 logout。

## 对“顺便修复”的处理

发现删除仅退出、集合名称仅写内存、toast 早于相册完成、状态枚举不会进入 complete、默认卡片是 placeholder：维持原功能行为，让事件契约如实反映它。不要补删除 API、持久化、toast 修复、状态机修复或真实卡片数据。

## 可执行的最终自审

记录开始时的 base/ref 和状态。对本次文件进行 diff；已有 PR 使用实际 base，不能默认 main。必要时逐方法对比原签名/原业务语句；对 logout 等用户点名的方法应保持实现逐字一致。

移除本次 analytics 专用新增语句的思维检查仅是辅助，不是语义证明。还要核对请求入参/次数、业务赋值、控制流、异步任务及取消、回调注册、导航副作用。业务行为断言与埋点断言分别检查。

发现越界，只撤回自己引入的功能改动，将其替换为符合边界的采集方案；找不到方案就保留缺口。不得还原用户或其他 session 的改动。
