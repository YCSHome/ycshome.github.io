## Neovim的缓冲区调教

缓冲区是 vim 中一个十分重要的存在。我们对一切文件的操作（不管是不是文件吧）都需要先创建一个缓冲区，然后对缓冲区进行操作。这是对缓冲区了解的第一步。

### 相关命令

  命令 `buffer` 可以将当前窗口更改至相应的缓冲区，可以通过句柄（数字）来得到或者用 `tab` 自动补全出来相应的文件名。
  
  命令 `buffers` `ls` 可以列出所有满足 `buflisted=true` 的缓冲区。不过可以通过 `buffers!` `ls!` 强制列出来

### 相关API

#### `nvim_create_buf`
```
nvim_create_buf({listed}, {scratch})                       *nvim_create_buf()*
```
创建一个全新、空白未命名的缓冲区。这是我们搞事情的第一步。

参数一共两个。
+ `listed`
  一个 `bool`，用来决定缓冲区是否可以被 `buffers` 或者 `ls` 列出来。

+ `scratch`
  一个 `bool`，当值为 `true` 时创造出来的缓冲区 `modeline` 不打开，否则为打开。对 [modeline的解释](https://zhuanlan.zhihu.com/p/151289861)

返回值就是创建的缓冲区的句柄。
