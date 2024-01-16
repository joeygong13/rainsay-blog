---
title: "Vim"
date: 2023-08-09T17:27:59+08:00
draft: false 
categories: ['奇妙的世界']
tags: ['vim', 'tool']
---

## 1. 查找与替换

> s 意为替换模式

命令模式为 `:{范围标记}s{分隔符}{需要查找的内容}{分割符}{替换内容}{分隔符}{替换标记}` ，

- 范围标记，可选，默认为当前行， `%` 表示全文
- 分隔符，一般为非字母数字的字符
- 替换标记， `g, golbal` 在查询范围内全部替换

几个例子：

| 普通替换, 替换当前行的第一个 foo 为 bar | `:s/foo/bar`  |
| --- | --- |
| 替换当前行的所有 foo 为 bar | `:s/foo/bar/g`  |
| 替换全文的 foo 为 bar | `:%s/foo/bar/g` |
| 替换多行的 foo 为 bar，n1或n2留空代表 首行或尾行 | `:n1,n2s/foo/bar/g`  |

### 1.1 使用正则

在正规表达式中使用 **\(** 和 **\)** 符号括起正规表达式，即可在后面使用**\1**、**\2** 等变量来访问 **\(** 和 **\)** 中的内容。

| Vim语法 | Perl语法 | 含义 |
| --- | --- | --- |
| \\+ | + | 1-任意个 |
| \\? | ? | 0-1个 |
| \\{n,m} | {n,m} | n-m个 |
| \\(和\\) | (和) | 分组 |


## 2. 执行 Shell

在 ：模式下，使用 ！开启一个shell 命令，如 `: !pwd`
可以读取 shell 命令产生的数据，或直接从 vim 缓冲区生成命令：

- 读取 shell 命令产生的数据，使用 read 指令，如：`:read !pwd`
- 从缓冲区输入 shell 命令，使用 write 指令，如 `:write !sh`，注意是执行缓冲区的每一行的内容

当给定一个范围给 shell 命令时，意思为：有 [range] 所指定的行会传给 cmd 作为标准输入，然后用 cmd 的标准输出替换 [range] 原本的行，下面是个例子，使用排序后的内容替换原来的内容 `:2,$!sort -t',' -k2`（使用每行的第二个关键字进行排序，排序后的内容替换原来的内容）

## 3. 多个文件

- `ls` 列出打开的文件列表，% 代表当前窗口的文件
- `bnext` 和 `bprev` 当前窗口展示下/上一个文件的缓冲区
- `buffer N`，使用 ls 输出的序号，直接跳转到某个编号的缓冲区
- `edit`放弃更改，直接读取文件覆盖当前缓冲区

### 切分窗口

- `vsplit`垂直切分，新窗口默认使用原来的缓冲区，加文件名，可以在新窗口打开文件
- `split`水平切分，效果同上
- `C-w`+`hjkl`，切换光标所在的窗口

## 4. 寄存器
使用双引号 " 选择一个寄存器，如 "a, "b, "c，寄存器名字需为为一个字符，有几个特殊寄存器：

- +，为系统剪切板
- 0，为复制专用寄存器

## 5. 对象

`aw` 选中当前单词

在 `visual` 模式下, 快速选中一个被引号或 <> 标签包含的词组，可用 `i"`(i, inner 内包含) 可选中 `"hello"` 的 `hello`，`a"`（all 全包含） 则可选中整个 `"hello"`


## 6. Jetbrain 中的 Vim 快捷键

```vim
"" Source your .vimrc
"source ~/.vimrc
let mapleader=" "

"" -- Suggested options --
" Show a few lines of context around the cursor. Note that this makes the
" text scroll if you mouse-click near the start or end of the window.
set scrolloff=5

"" Ide
set ideajoin
set ideaput

" Do incremental searching.
set incsearch

" Don't use Ex mode, use Q for formatting.
map Q gq

"" view
nnoremap zm :action ToggleZenMode <CR>

"" formatting
nnoremap == :action ReformatCode <CR>
nnoremap -- :action OptimizeImports <CR>
vnoremap cc :action CommentByLineComment <CR>

"" searching
nnoremap ss :action FindInPath <CR>
nnoremap <leader>x :action CloseContent <CR>
nnoremap <leader>f :action SelectInProjectView <CR>

"" server
map <leader>sd <Action>(ChooseDebugConfiguration)
map <leader>sx <Action>(Stop)
map <leader>sc <Action>(RunClass)

"" git
map <leader>gh :action Vcs.ShowTabbedFileHistory <CR>
map <leader>gs :action Vcs.UpdateProject <CR>
map <leader>ga :action CheckinProject <CR>
map <leader>gu :action Vcs.Push <CR>

"" code operator
map <leader>r <Action>(RenameElement)
map <leader>b <Action>(ToggleLineBreakpoint)
nmap <leader>l <Action>(GotoDeclaration)
nmap <leader>m <Action>(GotoDeclarationOnly)
nmap <leader>i <Action>(GotoImplementation)
nmap <leader>u <Action>(GotoSuperMethod)
nmap <leader>h <Action>(Back)
nmap <leader>p <Action>(PasteMultiple)


"" code generate
nnoremap <leader>mg :action GenerateGetterAndSetter <CR>

"" tab select
nmap <C-l> <Action>(NextTab)
nmap <C-h> <Action>(PreviousTab)
nnoremap <leader>1 1gt
nnoremap <leader>2 2gt
nnoremap <leader>3 3gt
nnoremap <leader>4 4gt
nnoremap <leader>5 5gt
nnoremap <leader>6 6gt
nnoremap <leader>7 7gt
nnoremap <leader>8 8gt
nnoremap <leader>9 9gt

" Find more examples here: https://jb.gg/share-ideavimrc
sethandler <C-i> i:ide

```

## 7. VS Code 中的 Vim

```json
{
    "vim.leader": "<space>",
    "vim.normalModeKeyBindings": [
        { "before": ["<S-l>"], "after": ["$"] },
        { "before": ["<S-h>"], "after": ["^"] },
        { "before": ["<leader>", "<leader>"], "commands": ["workbench.action.showCommands"] },
        { "before": ["<leader>", "x"], "commands": ["workbench.action.closeActiveEditor"] },
        { "before": ["<C-l>"], "commands": ["workbench.action.nextEditor"] },
        { "before": ["<C-h>"], "commands": ["workbench.action.previousEditor"] },
        { "before": ["leader", "f"], "commands": ["workbench.files.action.showActiveFileInExplorer"] },
        { "before": ["s", "s"], "commands": ["workbench.action.findInFiles"] },
        { "before": ["<leader>", "s"], "commands": ["workbench.view.search"] },
    ],
}
```