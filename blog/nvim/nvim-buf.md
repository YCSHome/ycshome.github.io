## Neovim的缓冲区

缓冲区是 vim 中一个十分重要的存在。我们对一切文件的操作（不管是不是文件吧）都需要先创建一个缓冲区，然后对缓冲区进行操作。这是对缓冲区了解的第一步。

所有 API 我都是对着官方文档机翻过来、配合上自己实践后完全理解之后进行转述的。同时会加入大量自己的理解和自己写的示例。

几乎可以说国内找不到第二篇如此详细的文章了。

### 相关命令

命令 `buffer` 可以将当前窗口更改至相应的缓冲区，可以通过句柄（数字）来得到或者用 `tab` 自动补全出来相应的文件名。

命令 `buffers` `ls` 可以列出所有满足 `buflisted=true` 的缓冲区。不过可以通过 `buffers!` `ls!` 强制列出来

### 相关API

--------------------------------------------------------------------------------

``` lua
nvim_create_buf({listed}, {scratch})
```

创建一个全新、空白未命名的缓冲区。这是我们搞事情的第一步。

+ `listed`
  一个 `bool`，用来决定缓冲区是否可以被 `buffers` 或者 `ls` 列出来。

+ `scratch`
  一个 `bool`，当值为 `true` 时创造出来的缓冲区 `modeline` 不打开，否则为打开。对 [modeline的解释](https://zhuanlan.zhihu.com/p/151289861)

返回值就是创建的缓冲区的句柄。

一般来说我们只需要 `local buf = vim.api.nvim_create_buf(false, true)` 就可以了。

--------------------------------------------------------------------------------

``` lua
nvim_get_current_buf()
```

获取当前光标所在位置的缓冲区的句柄。挺常用的吧。

--------------------------------------------------------------------------------

``` lua
nvim_set_current_buf({buffer})
```

用来设置当前窗口的缓冲区。

不过当 `textlock` 激活的时候不允许使用（不太明白这是什么？）

+ `buffer` 一个数字，代表缓冲区的句柄。

--------------------------------------------------------------------------------

``` lua
nvim_buf_get_option({buffer}, {name})
```

挺重要的一个API，可以返回当前缓冲区的相关配置选项的值。

+ `buffer`  
  缓冲区句柄，若为 `0` 则意味着当前缓冲区

+ `name`  
  相关 `option` 的名字

比如我想知道当前缓冲区的 `number` 选项信息如何:

``` lua
:lua print(vim.api.nvim_buf_get_option(0, "number"))
```

即可。

--------------------------------------------------------------------------------

``` lua
nvim_buf_set_option({buffer}, {name}, {value})
```

设置指定缓冲区的指定配置的值

+ `buffer`  
  缓冲区句柄，若为 `0` 则为当前缓冲区

+ `name`  
  指定的 `option`

+ `value`  
  指定的值

例子:
``` lua
vim.api.nvim_buf_set_option(0, "number", true)
```

--------------------------------------------------------------------------------

``` lua
nvim_buf_attach({buffer}, {send_buffer}, {opts})
```

~~版面差点撑爆Deepl的翻译上限~~

用来激活一个通道上的缓冲区更新事件，或者用来在 lua 回调。 请配合后文中的 API Buffer Updates 理解一些概念，但是没有多大帮助

官方示例:
> 在全局变量的 `events` 中获取缓冲区的更新。
> 可以使用 `print(vim.inspect(events))` 来查看其内容
>
> ``` lua
> events = {}
> vim.api.nvim_buf_attach(0, false, {
>   on_lines = function(...)
>     table.insert(events, {...})
>   end
> })
> ```

+ `buffer`  
  缓冲区句柄，若为 `0` 则为当前缓冲区

+ `send_buffer`
  如果初始通知应该包含整个缓冲区，那么第一个通知将会是 `nvim_buf_lines_event`，否则将会是 `nvim_buf_changedtick_event`。这不能在 lua 回调中使用。

+ `opts`
  这是一个可选参数（但是你必须提供一个列表，即使是空列表）
  + `on_lines`  
    更改缓冲区内容时的 lua 回调。返回 `true` 以分离。参数：
    + 一个字符串: "lines"（真的是这玩意）
    + 缓冲区句柄
    + `b:changedtick`
    + 更改的第一行（以0开始）
    + 更改的最后一行
    + 先前内容的字节数量。
    + deleted_codepoints（如果 `utf_sizes` 为真）
    + deleted_codeunits（如果 `utf_sizes` 为真）（不太明白这两个参数有啥用）

  + `on_bytes`  
    相对于 `on_lines`，它将会提供更加详细的信息。返回 `true` 时分离。参数：
    + 一个字符串: "bytes"（真的是这玩意）
    + 缓冲区句柄
    + `b:changedtick`
    + 更改后的开始行（以0开始）
    + 更改后的结束行
    + 更改后的字节偏移量
    + 变更后的文本的旧尾行
    + 变更后的文本的旧尾列
    + 变更后的文本的旧结束字节长度
    + 变更后的文本的新尾行
    + 变更后的文本的新尾列
    + 变更后的文本的新结束字节长度
  
  + `on_changedtick`  
    在没有文字变化的情况下，在 `changedtick` 的增量上调用的 lua 回调。参数：
    + 一个字符串 `changedtick`
    + 缓冲区句柄
    + `b:changedtick`
  
  + `on_detach`
    分离时调用的 lua 回调，参数：
    + 一个字符串 `detach`
    + 缓冲区句柄

  + `on_reload`  
    重新加载时调用的回调。此时整个缓冲区内容应该被认为是被改变了的。参数：
    + 一个字符串 `reload`
    + 缓冲区句柄

  + `utf_sizes`  
    包括被替换区域的 `utf-32` 和 `utf-16` 的大小，其实是作为 `on_lines` 的参数 qwq

  + `preview`
    同时也将它添加到命令预览中，也就是 `inccommand` 事件。

返回值：如果链接失败那么就返回 false，否则就是 true

--------------------------------------------------------------------------------

### API Buffer Updates

Neovim 中的 API 客户端允许你通过一种类似 `附加` 的方式来实时更新缓冲区。这和一个与 autocmd有关的`TextChanged` 非常像，但是功能更加强大并且更加精细。也就是说可以用来监控当前缓冲区。

一般来说，我们可以调用 `nvim_buf_attach()` 获取以下通知。

``` lua
nvim_buf_lines_event[{buf}, {changedtick}, {firstline}, {lastline}, {linedata}, {more}]
```

当在 `fristline` 与 `lastline`（不包括）之间的文本被改变为 `linedata` 列表中的新文本时。精细程度是一行，也就是说如果改变了一个字符，整行都会被传回。

当然，如果 `changedtick` 为 `v:null`（`nil`） 的时候，这意味着屏幕上的行即显示出来的文本被改变了，而不是缓冲区的内容被改变了。`linedata` 当然包括被改变的屏幕行。这会发生在用 `inccommand` 显示缓冲区预览的时候。

+ `buf`  
  API 对应的句柄

+ `changedtick`  
  缓冲的`b:changedtick` 的值。你可以通过 `help b:changedtick` 了解这是什么东西。你完全可以发送一个用来检查 `b:changedtick` 的 API 命令作为请求的一部分，来确保没有其它的变化。

+ `firstline`  
  被替换的行号。以 0 开始。即如果是第一行被替换了，那么 `fristline` 其实是 0 而不是 1

+ `lastline`  
  未被替换的第一行的行号。不过由于是以 0 代表第一行，如果第 2 行到第 5 行被替换了，那么 `lastline` 值为 5 而不是 6。如果这个事件是附加后的初始更新之一，那么 `lastline` 值为 -1。

+ `linedata`  
  包含新缓冲区内的字符串。换行符将会被省略、并且空行会被视为空字符串。

+ `more`
  一个 Boolean 值，用来表示当前变更是否被划分成了多个 `nvim_buf_lines_event` 通知。

```
nvim_buf_changedtick_event[{buf}, {changedtick}]
```

如果 `b:changedtick` 增加，那么这就可以用来撤回/重做。

+ `buf`  
  缓冲区句柄

+ `changedtick`  
  缓冲区的 `b:changedtick` 的新值

```
nvim_buf_detach_event[{buf}]
```

如果缓冲区被分离，也就是更新被禁止时，显示的调用方式是通过 `nvim_buf_detach()`。或者这在下述情况下被隐式触发：

- 缓冲区被丢弃并且没有被隐藏

- 缓冲区被重新加载，例如使用 `:edit` 或者是别的外部变化所触发的 `checktime` 或者 `autoread`

- 更常见的，就是缓冲区从内存中被卸载了

+ `buf`  
  缓冲区句柄

__接下来是样例__

不过这个样例只是辅助理解它的操作。你无法通过它来明白在 lua 中的细节操作。

当你调用了 `nvim_buf_attach()` 并且将 `send_buffer` 设置为 `true`，此时会先发送：
```
nvim_buf_lines_event[{buf}, {changedtick}, 0, -1, [""], v:false]
```

然后某个憨憨在缓冲区里加入了两行，发送
```
nvim_buf_lines_event[{buf}, {changedtick}, 0, 0, ["line1", "line2"], v:false]
```

然后这个憨憨在一个"Hello world"的行末尾加上了一个字符 "!"，发送
```
nvim_buf_lines_event[{buf}, {changedtick}, {linenr}, {linenr} + 1, ["Hello world!"], v:false]
```

这个憨憨在第三行输入"20dd"删掉了20行代码，发送
```
nvim_buf_lines_event[{buf}, {changedtick}, 2, 22, [], v:false]
```

这个憨憨用 `VISUAL LINE` 模式选择了 3-5 行并且在第六行输入 "p" ，发送
```
nvim_buf_lines_event[{buf}, {changedtick}, 2, 5,
  ['pasted line 1', 'pasted line 2', 'pasted line 3', 'pasted line 4',
   'pasted line 5', 'pasted line 6'],
  v:false
]
```

这个憨憨用 ":edit" 刷新了页面，发送
```
nvim_buf_detach_event[{buf}]
```

但是我目前还没有找到获取通知的API

### 后记

`nvim_buf_attach` 是我整得最难受的一个 API，国内压根没有教程资源，官方给的示例又不清不楚，让人难受到吐。

现在还没有更新完qwq
