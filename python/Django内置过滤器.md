# Django内置过滤器

可以查询下表来总览Django的内置过滤器：

| 过滤器             | 说明                                                  |
| ------------------ | ----------------------------------------------------- |
| add                | 加法                                                  |
| addslashes         | 添加斜杠                                              |
| capfirst           | 首字母大写                                            |
| center             | 文本居中                                              |
| cut                | 切除字符                                              |
| date               | 日期格式化                                            |
| default            | 设置默认值                                            |
| default_if_none    | 为None设置默认值                                      |
| dictsort           | 字典排序                                              |
| dictsortreversed   | 字典反向排序                                          |
| divisibleby        | 整除判断                                              |
| escape             | 转义                                                  |
| escapejs           | 转义js代码                                            |
| filesizeformat     | 文件尺寸人性化显示                                    |
| first              | 第一个元素                                            |
| floatformat        | 浮点数格式化                                          |
| force_escape       | 强制立刻转义                                          |
| get_digit          | 获取数字                                              |
| iriencode          | 转换IRI                                               |
| join               | 字符列表链接                                          |
| last               | 最后一个                                              |
| length             | 长度                                                  |
| length_is          | 长度等于                                              |
| linebreaks         | 行转换                                                |
| linebreaksbr       | 行转换                                                |
| linenumbers        | 行号                                                  |
| ljust              | 左对齐                                                |
| lower              | 小写                                                  |
| make_list          | 分割成字符列表                                        |
| phone2numeric      | 电话号码                                              |
| pluralize          | 复数形式                                              |
| pprint             | 调试                                                  |
| random             | 随机获取                                              |
| rjust              | 右对齐                                                |
| safe               | 安全确认                                              |
| safeseq            | 列表安全确认                                          |
| slice              | 切片                                                  |
| slugify            | 转换成ASCII                                           |
| stringformat       | 字符串格式化                                          |
| striptags          | 去除HTML中的标签                                      |
| time               | 时间格式化                                            |
| timesince          | 从何时开始                                            |
| timeuntil          | 到何时多久                                            |
| title              | 所有单词首字母大写                                    |
| truncatechars      | 截断字符                                              |
| truncatechars_html | 截断字符                                              |
| truncatewords      | 截断单词                                              |
| truncatewords_html | 截断单词                                              |
| unordered_list     | 无序列表                                              |
| upper              | 大写                                                  |
| urlencode          | 转义url                                               |
| urlize             | url转成可点击的链接                                   |
| urlizetrunc        | urlize的截断方式                                      |
| wordcount          | 单词计数                                              |
| wordwrap           | 单词包裹                                              |
| yesno              | 将True，False和None，映射成字符串‘yes’，‘no’，‘maybe’ |

**为模版过滤器提供参数的方式是：过滤器后加个冒号，再紧跟参数，中间不能有空格！**

**目前只能为过滤器最多提供一个参数！**

### 1. add

把add后的参数加给value。像这样：

```
{{ value|add:"2" }}
```

如果value为 4,则会输出6.

过滤器首先会强制把两个值转换成Int类型。 如果强制转换失败, 它会试图使用各种方式把两个值相加。

对于下面的例子：

```
{{ first|add:second }}
```

如果first是[1, 2, 3]，second是[4, 5, 6], 将会输出[1, 2, 3, 4, 5, 6].

### 2. addslashes

在引号前面加上斜杆。常用于在CSV中转义字符串。像这样：

```
{{ value|addslashes }}
```

如果value是"I'm using Django", 输出将变成 "I\'m using Django".

### 3. capfirst

大写变量的第一个字母。 如果第一个字符不是字母，该过滤器将不会生效。

```
{{ value|capfirst }}
```

如果value是"django", 输出将变成"Django"。

### 4. center

在给定的宽度范围内居中.

```
"{{ value|center:"15" }}"
```

如果value是"Django"，输出将是`Django`。

### 5. cut

移除value中所有的与给定参数相同的字符串。

```
{{ value|cut:" " }}
```

如果value为“String with spaces”，输出将为"Stringwithspaces"。

### 6. date

根据给定格式对一个日期变量进行格式化。

可用的格式字符串：

| 格式化字符 | 描述                                                 | 示例输出                                            |
| ---------- | ---------------------------------------------------- | --------------------------------------------------- |
| a          | 'a.m.'或'p.m.'                                       | 'a.m.'                                              |
| A          | 'AM'或'PM'                                           | 'AM'                                                |
| b          | 月份，文字形式，3个字母，小写。                      | “jan”                                               |
| B          | 未实现。                                             |                                                     |
| c          | ISO 8601格式                                         | 2008-01-02T10:30:00.000123+02:00                    |
| d          | 月的日子，带前导零的2位数字。                        | '01'到'31'                                          |
| D          | 周几的文字表述形式，3个字母。                        | 'Fri'                                               |
| e          | 时区名称                                             | ''，'GMT'，'-500'，'US/Eastern'等                   |
| E          | 月份，分地区。                                       |                                                     |
| f          | 时间                                                 | '1'，'1:30'                                         |
| F          | 月，文字形式。                                       | 'January'                                           |
| g          | 12小时格式，无前导零。                               | '1'到'12'                                           |
| G          | 24小时格式，无前导零。                               | '0'到'23'                                           |
| h          | 12小时格式。                                         | '01'到'12'                                          |
| H          | 24小时格式。                                         | '00'到'23'                                          |
| i          | 分钟                                                 | '00'到'59'                                          |
| I          | 夏令时间，无论是否生效。                             | '1'或'0'                                            |
| j          | 没有前导零的月份的日子。                             | '1'到'31'                                           |
| l          | 星期几，完整英文名                                   | 'Friday'                                            |
| L          | 布尔值是否是一个闰年。                               | True或False                                         |
| m          | 月，2位数字带前导零。                                | '01'到'12'                                          |
| M          | 月，文字，3个字母。                                  | “Jan”                                               |
| n          | 月无前导零。                                         | '1'到'12'                                           |
| N          | 美联社风格的月份缩写。                               | 'Jan.'，'Feb.'，'March'，'May'                      |
| o          | ISO-8601周编号                                       | '1999'                                              |
| O          | 与格林威治时间的差，单位小时。                       | '+0200'                                             |
| P          | 时间为12小时                                         | '1 am'，'1:30 pm'，'midnight'，'noon'，'12：30 pm'> |
| r          | RFC 5322格式化日期。                                 | 'Thu， 21 Dec 2000 16:01:07 +0200'                  |
| s          | 秒，带前导零的2位数字。                              | '00'到'59'                                          |
| S          | 一个月的英文序数后缀，2个字符。                      | 'st'，'nd'，'rd'或'th'                              |
| t          | 给定月份的天数。                                     | 28 to 31                                            |
| T          | 本机的时区。                                         | 'EST'，'MDT'                                        |
| u          | 微秒。                                               | 000000 to 999999                                    |
| U          | 自Unix Epoch以来的秒数（1970年1月1日00:00:00 UTC）。 |                                                     |
| w          | 星期几，数字无前导零。                               | '0'（星期日）至'6'（星期六）                        |
| W          | ISO-8601周数，周数从星期一开始。                     | 1，53                                               |
| y          | 年份，2位数字。                                      | '99'                                                |
| Y          | 年，4位数。                                          | '1999'                                              |
| z          | 一年中的日子                                         | 0到365                                              |
| Z          | 时区偏移量，单位为秒。                               | -43200到43200                                       |

范例：

{{ value|date:"D d M Y" }}

如果value是一个datetime对象，比如datetime.datetime.now()，输出将是字符串'Wed 09 Jan 2008'。

可以将date与time过滤器结合使用，以呈现datetime值的完整表示形式。 例如。：

```
{{ value|date:"D d M Y" }} {{ value|time:"H:i" }}
```

### 7. default

为变量提供一个默认值。

```
{{ value|default:"nothing" }}
```

### 8. default_if_none

如果（且仅当）value为None，则使用给定的默认值。

```
{{ value|default_if_none:"nothing" }}
```

### 9. dictsort

接受一个包含字典元素的列表，并返回按参数中给出的键排序后的列表。

```
{{ value|dictsort:"name" }}
```

如果value为：

```
[
    {'name': 'zed', 'age': 19},
    {'name': 'amy', 'age': 22},
    {'name': 'joe', 'age': 31},
]
```

那么输出将是：

```
[
    {'name': 'amy', 'age': 22},
    {'name': 'joe', 'age': 31},
    {'name': 'zed', 'age': 19},
]
```

还也可以做更复杂的事情，如：

```
{% for book in books|dictsort:"author.age" %}
    * {{ book.title }} ({{ book.author.name }})
{% endfor %}
```

如果books是：

```
[
    {'title': '1984', 'author': {'name': 'George', 'age': 45}},
    {'title': 'Timequake', 'author': {'name': 'Kurt', 'age': 75}},
    {'title': 'Alice', 'author': {'name': 'Lewis', 'age': 33}},
]
```

那么输出将是：

```
* Alice (Lewis)
* 1984 (George)
* Timequake (Kurt)
```

dictsort也可以按指定索引对多维列表进行排序，像这样：

```
{{ value|dictsort:0 }}
```

如果value为：

```
[
    ('a', '42'),
    ('c', 'string'),
    ('b', 'foo'),
]
```

那么输出将是：

```
[
    ('a', '42'),
    ('b', 'foo'),
    ('c', 'string'),
]
```

必须提供整数索引，不能是字符串。 以下产生空输出：

```
{{ values|dictsort:"0" }}
```

### 10. dictsortreversed

前面过滤器的反序功能。

### 11. divisibleby

如果value可以被参数整除，则返回True。

```
{{ value|divisibleby:"3" }}
```

如果value是21，则输出True。

### 12. escape

转义字符串的HTML。

转义仅在字符串输出时应用，因此在链接的过滤器序列中escape的位置无关紧要，就像它是最后一个过滤器。 如果要立即应用转义，请使用`force_escape`过滤器。

### 13. escapejs

转义用于JavaScript字符串的字符。 确保在使用模板生成JavaScript / JSON时避免语法错误。

```
{{ value|escapejs }}
```

如果value为`testing\r\njavascript \'string" <b>escaping</b>`，输出将为`testing\\u000D\\u000Ajavascript \\u0027string\\u0022 \\u003Cb\\u003Eescaping\\u003C/b\\u003E`。

### 14. filesizeformat

格式化为直观的文件大小形式（即'13 KB', '4.1 MB', '102 bytes'等）。

```
{{ value|filesizeformat }}
```

如果value为123456789，输出将是117.7 MB。

### 15. first

返回列表中的第一项。

```
{{ value|first }}
```

如果value是列表['a'， 'b'， 'c'] ，输出将为'a'。

### 16. floatformat

当不使用参数时，将浮点数舍入到小数点后一位，但前提是要显示小数部分。 像这样：

```
value           模板语法                输出
34.23234    {{ value | floatformat }}   34.2
34.00000    {{ value | floatformat }}   34
34.26000    {{ value | floatformat }}   34.3
```

如果与数字整数参数一起使用，将数字四舍五入为小数位数。 像这样：

```
value           模板语法                输出
34.23234    {{ value | floatformat：3 }} 34.232
34.00000    {{ value | floatformat：3 }} 34.000
34.26000    {{ value | floatformat：3 }} 34.260
```

特别有用的是传递0（零）作为参数，它将使float浮动到最接近的整数。

```
value           模板语法                输出
34.23234    {{ value | floatformat：“0” }}   34
34.00000    {{ value | floatformat：“0” }}   34
39.56000    {{ value | floatformat：“0” }}   40
```

如果传递给floatformat的参数为负，则会将一个数字四舍五入到小数点后的位置，但前提是要显示一个小数部分。 像这样：

```
value           模板语法                输出
34.23234    {{ value | floatformat：“ - 3” }}    34.232
34.00000    {{ value | floatformat：“ - 3” }}    34
34.26000    {{ value | floatformat：“ - 3” }}    34.260
```

### 17. force_escape

立即转义HTML字符串。

### 18. get_digit

给定一个整数，返回所请求的数字，1表示最右边的数字，2表示第二个最右边的数字，以此类推。

```
{{ value|get_digit:"2" }}
```

如果value为123456789，则输出8。

### 19. iriencode

将IRI（国际化资源标识符）转换为适合包含在URL中的字符串。

```
{{ value|iriencode }}
```

如果value是`?test=1&me=2`，输出则是`?test=1&amp;me=2`。

### 20. join

使用字符串连接列表，类似Python的str.join(list)

```
{{ value|join:" // " }}
```

如果value是列表['a'， 'b'， 'c'] ，输出为`a // b // c`。

### 21. last

返回列表中的最后一个项目。类似first过滤器。

```
{{ value|last }}
```

### 22. length

返回对象的长度。 这适用于字符串和列表。

```
{{ value|length }}
```

如果value是['a'， 'b'， 'c'， 'd']或"abcd"，输出将为4。

对于未定义的变量，过滤器返回0。

### 23. length_is

如果对象的长度等于参数值，则返回True，否则返回False。

```
{{ value|length_is:"4" }}
```

如果value是['a'， 'b'， 'c'， 'd']或"abcd"，输出将为True。

### 24. linebreaks

替换纯文本中的换行符为`<p>`标签。

```
{{ value|linebreaks }}
```

如果value是`Joel\nis a slug`，输出将为`<p>Joel<br />is a slug</p>`。

### 25. linebreaksbr

替换纯文本中的换行符为`<br />`标签。

```
{{ value|linebreaksbr }}
```

如果value是`Joel\nis a slug`，输出将为`Joel<br />is a slug`。

### 26. linenumbers

显示带行号的文本。

{{ value|linenumbers }}

如果value为：

```
one
two
three
```

输出将是：

```
1. one
2. two
3. three
```

### 27. ljust

给定宽度下左对齐。

```
"{{ value|ljust:"10" }}"
```

如果value为Django，则输出为`Django`。

### 28. lower

将字符串转换为全部小写。

```
{{ value|lower }}
```

### 29. make_list

将对象转换为字符的列表。对于字符串，直接拆分为单个字符的列表。对于整数，在创建列表之前将参数强制转换为unicode字符串。

```
{{ value|make_list }}
```

如果value是字符串"Joel"，输出将是列表['J'， 'o' ， 'e'， 'l']。

如果value为123，输出为列表['1'， '2'， '3']。

### 30. phone2numeric

将电话号码（可能包含字母）转换为其等效数字。

```
{{ value|phone2numeric }}
```

如果value为`800-COLLECT`，输出将为800-2655328。

### 31. pluralize

如果值不是1,则返回一个复数形式，通常在后面添加's'表示。

例如：

```
You have {{ num_messages }} message{{ num_messages|pluralize }}.
```

如果`num_messages`是1，则输出为`You have 1 message`。 如果num_messages是2，输出为`You have 2 messages`。

另外如果需要的不是's'后缀的话, 可以提供一个备选的参数给过滤器：

```
You have {{ num_walruses }} walrus{{ num_walruses|pluralize:"es" }}.
```

对于非一般形式的复数,可以同时指定单复数形式，用逗号隔开。例如：

```
You have {{ num_cherries }} cherr{{ num_cherries|pluralize:"y,ies" }}.
```

### 32. pprint

用于调试的过滤器。

### 33. random

返回给定列表中的随机项。

```
{{ value|random }}
```

### 34. rjust

右对齐给定宽度字段中的值。

```
"{{ value|rjust:"10" }}"
```

### 35. safe

将字符串标记为安全，不需要转义。不再赘述。

### 36. safeseq

将safe过滤器应用于序列的每个元素。 与对序列进行其他过滤操作（例如join）一起使用时非常有用。

```
{{ some_list|safeseq|join:", " }}
```

在这种情况下，不能直接使用safe过滤器，因为它首先将变量转换为字符串，而不是使用序列的各个元素。

### 37. slice

返回列表的一部分。也就是切片，与Python的列表切片相同的语法。

```
{{ some_list|slice:":2" }}
```

如果some_list是['a'， 'b'， 'c'] ，输出将为['a'， 'b']。

### 38. slugify

转换为ASCII。空格转换为连字符。删除不是字母数字，下划线或连字符的字符。转换为小写。还会去除前导和尾随空格。

```
{{ value|slugify }}
```

如果value是`Joel is a slug`，输出为`joel-is-a-slug`。

### 39. stringformat

根据参数，格式化变量。

```
{{ value|stringformat:"E" }}
```

如果value为10，输出将为1.000000E+01。

### 40. striptags

尽可能的去除HTML中的标签。

```
{{ value|striptags }}
```

如果value是`<b>Joel</b> <button>is</button> a <span>slug</span>`，输出`Joel is a slug`。

### 41. time

根据给定的格式，格式化时间。给定格式可以是预定义的`TIME_FORMAT`，也可以是与date过滤器相同的自定义格式。

```
{{ value|time:"H:i" }}
```

如果value等于datetime.datetime.now()，则输出字符串`01:23`。

time过滤器只接受格式字符串中与时间相关的参数，而不是日期。如果需要格式化date值，请改用date过滤器。

### 42. timesince

将日期格式设为自该日期起的时间（例如，“4天，6小时”）。

采用一个可选参数，它是一个包含用作比较点的日期的变量。例如，如果`blog_date`是表示2006年6月1日午夜的日期实例，并且`comment_date`是2006年6月1日08:00，则以下将返回“8 hours”：

```
{{ blog_date|timesince:comment_date }}
```

### 43. timeuntil

类似于timesince，它测量从现在开始直到给定日期或日期时间的时间。例如，如果今天是2006年6月1日，而`conference_date`是2006年6月29日，则`{{ conference_date | timeuntil }}`将返回“4 weeks”。

可选参数是一个包含用作比较点的日期变量。如果`from_date`为2006年6月22日，则以下内容将返回“1 weeks”：

```
{{ conference_date|timeuntil:from_date }}
```

### 44. title

将所有单词的首字母大写，其它字母小写。

```
{{ value|title }}
```

如果value为“my FIRST post”，输出将为“My First Post”。

### 45. truncatechars

如果字符串包含的字符总个数多于指定的字符数量，那么会被截断掉后面的部分。截断的字符串将以“...”结尾。

```
{{ value|truncatechars:9 }}
```

如果value是`Joel is a slug`，输出为`Joel i...`。

### 46. truncatechars_html

类似于truncatechars，但是会保留HTML标记。

```
{{ value|truncatechars_html:9 }}
```

如果value是`<p>Joel is a slug</p>`，输出`<p>Joel i...</p>`。

### 47. truncatewords

在一定数量的字数后截断字符串。与truncatechars不同的是，这个以字的个数计数，而不是字符计数。

```
{{ value|truncatewords:2 }}
```

如果value 是`Joel is a slug`, 输出为`Joel is ...`。

字符串中的换行符将被删除。

### 48. truncatewords_html

类似于truncatewords，但是保留HTML标记。

```
{{ value|truncatewords_html:2 }}
```

HTML内容中的换行符将保留。

### 49. unordered_list

接收一个嵌套的列表，返回一个HTML的无序列表，但不包含开始和结束的`<ul>`标签。

例如，如果var 包含['States', ['Kansas', ['Lawrence', 'Topeka'], 'Illinois']]， 那么`{{ var|unordered_list }}`将返回：

```
<li>States
<ul>
        <li>Kansas
        <ul>
                <li>Lawrence</li>
                <li>Topeka</li>
        </ul>
        </li>
        <li>Illinois</li>
</ul>
</li>
```

### 50. upper

将字符串转换为全部大写的形式：

```
{{ value|upper }}
```

### 51. urlencode

转义要在URL中使用的值。

```
{{ value|urlencode }}
```

如果value为`https://www.example.org/foo?a=b&c=d`，输出`https%3A//www.example.org/foo%3Fa%3Db%26c%3Dd`。

### 52. urlize

将文字中的网址和电子邮件地址转换为可点击的链接。

该模板标签适用于前缀为`http://`，`https://`的链接，或者`www`。

由urlize生成的链接会向其中添加rel="nofollow"属性。

```
{{ value|urlize }}
```

如果value是`Check out www.djangoproject.com`，输出`Check out <a href="http://www.djangoproject.com" rel="nofollow">www.djangoproject.com</a>`。

除了超级链接之外，urlize也会将电子邮件地址转换为邮件地址链接。 如果value是`Send questions to foo@example.com`，输出将是`Send questions to <a href="mailto:foo@example.com">foo@example.com</a>`。

### 53. urlizetrunc

将网址和电子邮件地址转换为可点击的链接，就像urlize，但截断长度超过给定字符数限制的网址。

```
{{ value|urlizetrunc:15 }}
```

如果value是`Check out www.djangoproject.com`，将输出`Check out <a href="http://www.djangoproject.com" rel="nofollow">www.djangopr...</a>'`.

与urlize一样，此过滤器应仅应用于纯文本。

### 54. wordcount

返回单词的个数。

```
{{ value|wordcount }}
```

如果value是`Joel is a slug`，输出4。

### 55. wordwrap

以指定的行长度，换行单词。

```
{{ value|wordwrap:5 }}
```

如果value是`Joel is a slug`，输出为：

```
Joel
is a
slug
```

### 56. yesno

将True，False和None，映射成字符串‘yes’，‘no’，‘maybe’。

```
{{ value|yesno:"yeah,no,maybe" }}
```

## 国际化标签和过滤器

Django还提供了一些模板标签和过滤器，用以控制模板中国际化的每个方面。它们允许对翻译，格式化和时区转换进行粒度控制。

### 1. i18n

此标签允许在模板中指定可翻译文本。要启用它，请将`USE_I18N`设置为True，然后加载`{％ load i18n ％}`。

### 2. l10n

此标签提供对模板的本地化控制，只需要使用`{％ load l10n ％}`。通常将`USE_L10N`设置为True，以便本地化默认处于活动状态。

### 3. tz

此标签对模板中的时区进行控制。 像l10n，只需要使用`{％ load tz }`，但通常还会将`USE_TZ`设置为True，以便默认情况下转换为本地时间。

## 其他标签和过滤器库

Django附带了一些其他模板标签，必须在`INSTALLED_APPS`设置中显式启用，并在模板中启用`{% load %}`标记。

### 1. django.contrib.humanize

一组Django模板过滤器，用于向数据添加“人性化”，更加可读。

### 2. static

static标签用于链接保存在`STATIC_ROOT`中的静态文件。例如：

```
{% load static %}
<img src="{% static "images/hi.jpg" %}" alt="Hi!" />
```

还可以使用变量：

```
{% load static %}
<link rel="stylesheet" href="{% static user_stylesheet %}" type="text/css" media="screen" />
```

还可以像下面这么使用：

```
{% load static %}
{% static "images/hi.jpg" as myphoto %}
<img src="{{ myphoto }}"></img>
```

