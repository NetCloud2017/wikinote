### YAML 核心规则： 
1. 主要以 Key: + 空格 + Value的格式书写；
2. 必须使用 *空格键*打出的空格；不能用Tab键打出的空格； 否则语法错误。
3. 缩进要保持一致
4. 分号 *:* 和 *-* 后都要有一个空格；*:* 后填值， *-* 写列表的行值；
### YAML 数据类型： 
1. 纯量： 布尔值， 数字， 字符串， 空值（null或~表示）
2. 列表 List
3. 映射 Map
纯量
```
# 字符串
string_value: Hello YAML
quoted_string: "This is a string with a \n newline."

# 数字
integer_value: 1024
float_value: 3.14159

# 布尔值
is_active: true
is_enabled: false

# 空值
empty_value: null
another_empty: ~
```
> 字符串使用注意： 默认无需引号：my_string: Hello YAML； 单引号 (’)：不会转义特殊字符，所有内容视为字面量； 双引号 (”)：会转义特殊字符，例如 \n 会被解析为换行符。

序列
```
# 一个简单的水果列表
fruits:
  - Apple
  - Orange
  - Strawberry

# 也可以使用行内（inline）格式
fruits_inline: [Apple, Orange, Strawberry]

```

映射
```
# 描述一个用户的基本信息
user:
  name: John Doe
  age: 30
  email: john.doe@example.com

# 也可以使用行内（inline）格式
user_inline: {name: Jane Doe, age: 28}
```
#### 多行文本
**字面量块 `|`**：保留每一个换行。适合要“按原样呈现”的文本。

```
about: |
  第一行
  第二行
  第三行
```

解析后：`"第一行\n第二行\n第三行\n"`（注意末尾有 1 个换行，因为默认是clip-保留一个结尾换行（\n），但丢掉多余的尾随空行）。

**折叠块** `>`：把换行折叠为空格（空行保留为换行）

```
about: >
  这一段
  被写成了多行
  但会折叠成一行显示

  空行用来分段
  分段之间会出现真正的换行
```

解析后：`"这一段 被写成了多行 但会折叠成一行显示\n\n空行用来分段 分段之间会出现真正的换行\n"` （你看到：单个换行被折成空格；空行保留为 `\n`；末尾仍有一个换行，因为clip-保留一个结尾换行（\n），但丢掉多余的尾随空行）
#### 注释
使用井号 (#) 进行单行注释。从 # 开始到该行末尾的所有内容都会被忽略。

```
# 这是一个注释，它将被解析器忽略。
key: value # 这也是一个行内注释。
```



