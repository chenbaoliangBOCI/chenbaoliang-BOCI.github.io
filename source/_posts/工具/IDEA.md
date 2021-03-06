---
title: IDEA
date: 2017-01-01 09:00:00
tags: [工具]
---

# 常用快捷键


编辑代码

- `shift+F6`：重命名
- `ctrl+alt+l`：格式化代码
- `ctrl+alt+o`：优化导入类和包
- `ctrl+d`：复制行
- `ctrl+y`：删除行
- `ctrl+F4`：关闭当前编辑面板
- `ctrl+shift+Up/Down`：代码向上/下移动
- `ctrl+/`/`ctrl+shift+/`：行注释或块注释

导航

- `ctrl+e`/`ctrl+shift+e`：最近打开/编辑的文件
- `alt+home`：导航栏
- `F11`/`shift+F11`：设置/显示书签
- `ctrl+shift+a`：全局搜索操作和设置
- `alt+7`：显示类结构
- `ctrl+w`：选中代码
- `ctrl+shift+加号或减号`：展开或折叠代码

# 关键设置

自动补全快捷键：keymap-Main menu-Code-Completion-Basic

自动补全取消大小写敏感：Editor-General-Code Completion-Case sensitive completion

编辑区字体：Editor-Font

换行符：Editor-Code Style-Line seperator

Tab：Editor-Code Style-Java

编码：Editor-File Encodings

拼写检查：Editor-inspections-Spelling-typo



# 切换 editor 主题时保留 ide 整体主题

1. 编辑文件 `$User/.IdeaIC201x.x/config/options/options.xml`
2. `<property name="change.laf.on.editor.theme.change" value="false" />`

---
# 选中变量时，在滚动条上显示所有出现的位置

Editor - Colors Scheme - General - code - Identifier under caret - 勾选 Error stripe mark

---
# Live Templates

**iterations**

- `fori` Create iteration loop
  `for (int $INDEX$ = 0; $INDEX$ < $LIMIT$; $INDEX$++)`
- `itar` Iterate elements of array
  `for (int $INDEX$ = 0; $INDEX < $ARRAY$.length; $INDEX$++)`
- `itco` Iterate elements of Collection
  `for ($ITER_TYPE$ $ITER$ = $COLLECTION$.iterator(); $ITER$.hasNext(); )`
- `iten` Iterate Enumeration
  `while ($ENUM$.hasMoreElements())`
- `iter` Iterate Iterable`
  foreach
- `itit` Iterate Iterator
  `while ($ITER.hasNext())`
- `itli` Iterate elements of List
  `for (int $INDEX$ = 0; $INDEX$ < $LIST$.size(); $INDEX$++)`
- `ittok` Iterate tokens from String
  `for (java.util.StringTokenizer $TOKENIZER$ = new java.util.StringTokenizer(String); $TOKENIZER$.hasMoreTokens(); )`
- `itve` Iterate elements of Vector
  `for (int $INDEX$ = 0; $INDEX$ < $VECTOR$.size(); $INDEX$++)`
- `ritar` Iterate elements of array in reverse order
  `for (int $INDEX$ = $ARRAY$.length - 1; $INDEX$ >= 0; $INDEX$--)`

**other**

- `ifn` Inserts "if null" statement
- `inn` Inserts "if not null" statement
- `psvm` main() method declaration

**output**

- `sout` System.out.println()

**plain**

- `psfs` public static final String

**surround**

- `I` foreach

# 内存优化

```
-Xms1024m
-Xmx2048m
-XX:ReservedCodeCacheSize=1024m
-XX:+UseCompressedOops
```

