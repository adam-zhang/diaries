
# 如何用 jq 解析 JSON

首先，你需要安装 jq，它已被大多数 GNU/Linux 发行版选中，并使用各自的软件包安装程序命令进行安装。在 Arch Linux 上：
```
$ sudo pacman -S jq
```
在 Debian、Ubuntu、Linux Mint 上：
```
$ sudo apt-get install jq
```
在 Fedora 上：
```
$ sudo dnf install jq
```
在 openSUSE 上：
```
$ sudo zypper install jq
```
对于其它操作系统或平台参见官方的安装指导。jq 的基本过滤和标识符功能jq 可以从 STDIN 或文件中读取 JSON 数据。你可以根据情况使用。单个符号 "." 是最基本的过滤器。这些过滤器也称为对象标识符-索引。jq 使用单个 "." 过滤器基本上相当将输入的 JSON 文件格式化输出。单引号：不必始终使用单引号。但是如果你在一行中组合几个过滤器，那么你必须使用它们。双引号：你必须用两个双引号括起任何特殊字符，如 @、＃、$，例如 jq .foo.”@bar”。原始数据打印：不管出于任何原因，如果你只需要最终解析的数据（不包含在双引号内），请使用带有 -r 标志的 jq 命令，如下所示：jq -r .foo.bar。解析特定数据要过滤出 JSON 的特定部分，你需要了解格式化输出的 JSON 文件的数据层次结构。来自维基百科的 JSON 数据示例：
```
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25,
  "address": {
    "streetAddress": "21 2nd Street",
    "city": "New York",
    "state": "NY",
    "postalCode": "10021"
},
  "phoneNumber": [
{
  "type": "home",
  "number": "212 555-1234"
},
{
  "type": "fax",
  "number": "646 555-4567"
}
],
  "gender": {
  "type": "male"
  }
}
```

我将在本教程中将此 JSON 数据用作示例，将其保存为 sample.json。假设我想从 sample.json 文件中过滤出地址。所以命令应该是这样的：$ jq .address sample.json示例输出：
```
{
  "streetAddress": "21 2nd Street",
  "city": "New York",
  "state": "NY",
  "postalCode": "10021"
}
```
再次，我想要邮政编码，然后我要添加另一个对象标识符-索引，即另一个过滤器。
```
$ cat sample.json | jq .address.postalCode
```
另请注意，过滤器区分大小写，并且你必须使用完全相同的字符串来获取有意义的输出，否则就是 null。
从 JSON 数组中解析元素JSON 数组的元素包含在方括号内，这无疑是非常通用的。要解析数组中的元素，你必须使用 "[]" 标识符以及其他对象标识符索引。在此示例 JSON 数据中，电话号码存储在数组中，要从此数组中获取所有内容，你只需使用括号，像这个示例：
```
$ jq .phoneNumber[] sample.json
```
假设你只想要数组的第一个元素，然后使用从 0 开始的数组对象编号，对于第一个项目，使用 [0]，对于下一个项目，它应该每步增加 1。
```
$ jq .phoneNumber[0] sample.json
```
脚本编程示例假设我只想要家庭电话，而不是整个 JSON 数组数据。这就是用 jq 命令脚本编写的方便之处。
```
$ cat sample.json | jq -r '.phoneNumber[] | select(.type == "home") | .number'
```
首先，我将一个过滤器的结果传递给另一个，然后使用 select 属性选择特定类型的数据，再次将结果传递给另一个过滤器。解释每种类型的 jq 过滤器和脚本编程超出了本教程的范围和目的。强烈建议你阅读 jq 手册，以便更好地理解下面的内容。资源：
https://stedolan.github.io/jq/manual/
http://www.compciv.org/recipes/cli/jq-for-parsing-json/
https://lzone.de/cheat-sheet/jqvia: 
https://www.ostechnix.com/how-to-parse-and-pretty-print-json-with-linux-commandline-tools/

