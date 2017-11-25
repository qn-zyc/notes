<!-- TOC -->

- [context 源码阅读](#context-源码阅读)
    - [Context 规则](#context-规则)
    - [emptyCtx](#emptyctx)
    - [WithCancel](#withcancel)
    - [WithDeadline](#withdeadline)
    - [WithValue](#withvalue)

<!-- /TOC -->

# context 源码阅读

## Context 规则

不要在结构体中存储 Context, 而应该将 Context 当做参数传递给需要它的函数, 而且 Context 应该是第一个参数, 建议命名为 ctx:

```go
func DoSomething(ctx Context, arg Arg) error {
    // ... use ctx ...
}
```

不要传递一个 nil 值的 Context, 即使那个函数允许这样, 当不确定需要使用哪个 Context 时, 使用 `Context.TODO`.

Context 的 Values 方法只用于在程序和接口间传递请求相关的数据, 不要传递一些可选参数???

同一 Context 可用在不同的 goroutines 中, 它是线程安全的.


## emptyCtx

不会被取消(永远返回 nil 的 channel), 没有 values, 没有 deadline, 它是 int 的别名.


## WithCancel

WithCancel 返回的是 `*cancelCtx`, 它有一个 done 字段, 类型是 `chan struct{}`, 返回的 cancelFunc 主要是将这个 done close 掉, 这样监听的其他 goroutine 就能得知这个 context 是被 cancel 了.

另外, 为了能监听到 parent context 的 cancel 消息, 它还在 parent context 的 children 字段中加入自己, 这个字段是个 `map[canceler]struct{}`, 其实是当做 set 来用的, key 就是 `cancelCtx`(注意是值类型而不是指针). 

在初始化时如果它的 parent 的 done 不为 nil(为 nil 就什么也不做), 它会递归地向上查找 parent, 直到找到一个 `*cancelCtx` 类型的 parent, 然后向 parent 的 children 字段加入自己. 如果找不到一个 `*cancelCtx` 类型的 parent, 就单独启动一个 goroutine, 监听 parent.done 和自己的 done, 监听 parent.done 是为了调用自己的 cancel, 也就是说 parent 不是 `*cancelCtx` 类型的 context 时 parent 调用不了自己的 cancel 方法, 所以需要通过 select parent.done 来得知 parent 已经结束了. 监听自己的 done 是为了及时退出 goroutine.

当调用 cancel 方法时, 会 close 自己的 done channel, 并且记录 err. 遍历自己的 children, 调用 children 的 cancel 方法, 之后将 children 置为 nil.

另外当 cancel 指令是发生在自己这一层时(通过返回的 cancelFunc 触发的), 还会从 parent.children 中将自己删除, 这样 parent 在 cancel 就不会再一次调用自己的 cancel 方法了(其实多次调用并没有关系, 这里可能是考虑性能).

也就是说 cancel 会向下传递不会向上传递.


## WithDeadline

先判断一下自己的 deadline 是不是在 parent.deadline 之后, 如果是, 就直接返回 `WithCancel(parent)`.

给 parent 添加上 cancel 的逻辑, 初始化 `*timerCtx`.

如果自己的 deadline 已经过期了, 就调用自己的 cancel, 但是依然会返回自己和 cancelFunc.

设置一个定时器, 在 deadline 时调用 cancel 方法.

timerCtx 的 cancel 方法就是调用了 cancelCtx.cancel(), 只是多加了 stop timer 的代码.


## WithValue

保存了一对 key-value, 调用 Value 方法时先检查是不是这个 key, 不是的话再调用 `parent.Value()`.

key 最好不要是 string 或者其他内建类型, 可以使用包级别的别名, 这样其他包就获取不到这个值了, 只能通过方法来获取.

每个 context 中只保留一对 key-value, 查找时一级一级查找, 并没有 map 这样的结构.