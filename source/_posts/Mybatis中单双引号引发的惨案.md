---
title: Mybatis中单双引号引发的惨案
copyright: true
permalink: 1
top: 0
date: 2019-09-03 23:04:24
tags:
	- Mybatis
	- Sql
categories:
	- 技术
password:
---



出现奇怪的bug，那就从头来过，不要漏掉任何地方！



<!-- more -->





### `#{}`与`${}`的区别

> `#{}`是预编译处理，`${}`是字符串替换Mybatis在处理#{}时，会将sql中的#{}替换为?号， 调用PreparedStatement的set方法来赋值；
> Mybatis在处理${}时，就是把${}替换成变量的值。
> 使用#{}可以有效的防止SQL注入，提高系统安全性。

再通俗的说，使用`${}`mybatis会把参数加上双引号，而`${}` 你给啥，sql语句中就是啥，如下示例:

```sql
select * from table where name = #{name}  name->小明 
## 结果：select * from table where name = "小明"
select * from table where name = ${name}  name->小明 
## 结果：select * from table where name = 小明

```

### 问题

最近有个功能需要从sqlserver中去数据，有个脚本很简单如下:

```sql
select * from table where id in(...) 
```

id已经创建索引了，考虑到数据传输，我每次设置的集合大小为100个，因为这是再简单不过的语句了，直接上线给别人使用，但是别人的反馈是，使用50个id需要40多秒！！！ 这就有点吓人了，幸好此场景只是在半夜定时的去使用，慢一点不会对第二天有影响，但是白天想要测试的时候就懵了。当然了40多s就别提是否影响别人使用了，基本上就已经崩溃了好不好！！！

这就有点吓人了，幸好此场景只是在半夜定时的去使用，慢一点不会对第二天有影响，但是白天想要测试的时候就懵了。当然了40多s就别提是否影响别人使用了，基本上就已经崩溃了好不好！！！

下面简化了一下，对应的xml代码如下:

```xml
<select id="selectTbdIdByLbdIdList" resultType="xxx.xxx.xxMapper">
    SELECT id ,tid FROM table where id IN
    <foreach collection="list" item="item" open="(" close=")" separator=",">
        #{item}
    </foreach>
</select>
```

debug 模式下的输出如下:

```
| ==>  Preparing: SELECT id ,tid FROM table where id IN ( ?,?,?,?,?,?...) 
| ==> Parameters: 123(String),234(String),345(String),456(String),
| <==      Total: ....
```

我把sql整理出来放在sqlserver客户端去执行

```sql
SELECT id ,tid FROM table where id IN ( "123","234","345"...);
```

刚开始执行报错了，后面把双引号改成单引号就行了，即

```sql
SELECT id ,tid FROM table where id IN ( '123','234','345'...);
耗时: 0.092s
```

**记住这里的单双引号的问题** 

??? 很快啊，这是什么情况，第一次遇到这种情况，直接运行sql很快，但是通过mybatis就很慢。

所以我首先怀疑是`ORM`框架的问题，接着我用`JDBC`快速写了个demo，来验证，代码如下：

```java
String connectionUrl = "jdbc:sqlserver://xxx:8838;DatabaseName=xxx;user=xxx;password=xx";
Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");
Connection con = DriverManager.getConnection(connectionUrl);
Statement stmt = con.createStatement();
String SQL = "SELECT id ,tid FROM table where id IN ( '123','234','345'...)";
long s = System.nanoTime();
ResultSet rs = stmt.executeQuery(SQL);
System.out.println((System.nanoTime() - s) / 1_000_000);

// Iterate through the data in the result set and display it.
while (rs.next()) {
    System.out.println(rs.getString("id") + " ---> " + rs.getString("tid"));
}

// 耗时0.109ms
```

这里也是很快，没什么问题，忽略`ORM`的问题。

因为我这里用的是`Mybatis-Plus`，所以我又怀疑是mp的问题，于是debug代码，最后卡在这个地方:

```java
//PreparedStatementHandler.class
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();// 卡在这一行
    return resultSetHandler.handleResultSets(ps);
}
```

但这是`Mybatis`的代码，再者说mp只是简化了代码生成这一块，对`Mybatis`本身的执行没有影响，所以mp也被排除！

这个时候已经过去很长时间了，整个人很懵，怎么会这样？？？这么简单的sql还会出这么大的问题！我重新理了下思绪，此处的sql是在sqlserver上执行的，那会不会是sqlserver上的问题呢？

我突然灵光一闪，刚刚debug出来的脚本直接放在sqlserver的客户端上执行的时候是有问题的，我后面是把双引号改成单引号才成功的，我赶紧调整了xml中的脚本，如下：

```sql
<select id="selectTbdIdByLbdIdList" resultType="xxx.xxx.xxMapper">
    SELECT id ,tid FROM table where id IN
    <foreach collection="list" item="item" open="(" close=")" separator=",">
        '${item}'
    </foreach>
</select>

```

然后再执行，debug出来的脚本如下:

```
| ==>  Preparing: SELECT id ,tid FROM table where id IN ( '123','234','345','456'...) 
| ==> Parameters: 
| <==      Total: ....

```

耗时: 0.100s!!!!

如释重负，原来是双引号惹的祸！

SqlServer是不支持双引号的，但是mybatis最后生成的sql使用的双引号，当然这对mysql是没问题的，当然也有例外

> 如果SQL服务器模式启用了NSI_QUOTES，可以只用单引号引用字符串。用双引号引用的字符串被解释为一个识别符。

所以我遇到的情况是就是生成带双引号的脚本丢给sqlserver执行的时候，被sql服务器误认为是一个识别符，类似java中类型的强转，此时索引是不生效的，也就是说一开的in查询时没有使用到索引的！！！话说那个表中有700w条记录，怪不得每次查询50条的时候，耗时很均匀，都在40多秒。。。。。



回到开头，这种情况就是借助`${}`来解决，当然是用它是有隐患的，因为它并不能防止sql注入，但是对于我这边的场景不会出现这种情况，所以我赶紧的把其他地方也都改了过来！！！





### 最后

解决问题还是要**大胆假设，小心求证** 事实的真相只有一个！！！

另外在debug的时候，顺便看到了`#{}`和`${}`的拼接代码，放在了下面

```java
// ForEachSqlNode
public void appendSql(String sql) {
    GenericTokenParser parser = new GenericTokenParser("#{", "}", content -> {
        String newContent = content.replaceFirst("^\\s*" + item + "(?![^.,:\\s])", itemizeItem(item, index));
        if (itemIndex != null && newContent.equals(content)) {
            newContent = content.replaceFirst("^\\s*" + itemIndex + "(?![^.,:\\s])", itemizeItem(itemIndex, index));
        }
        return "#{" + newContent + "}";
    });

    delegate.appendSql(parser.parse(sql));
}

```



```java
// TextSqlNode  
private GenericTokenParser createParser(TokenHandler handler) {
  return new GenericTokenParser("${", "}", handler);
}

```



```java
// GenericTokenParser
public String parse(String text) {
    if (text == null || text.isEmpty()) {
        return "";
    }
    // search open token
    int start = text.indexOf(openToken);
    if (start == -1) {
        return text;
    }
    char[] src = text.toCharArray();
    int offset = 0;
    final StringBuilder builder = new StringBuilder();
    StringBuilder expression = null;
    while (start > -1) {
        if (start > 0 && src[start - 1] == '\\') {
            // this open token is escaped. remove the backslash and continue.
            builder.append(src, offset, start - offset - 1).append(openToken);
            offset = start + openToken.length();
        } else {
            // found open token. let's search close token.
            if (expression == null) {
                expression = new StringBuilder();
            } else {
                expression.setLength(0);
            }
            builder.append(src, offset, start - offset);
            offset = start + openToken.length();
            int end = text.indexOf(closeToken, offset);
            while (end > -1) {
                if (end > offset && src[end - 1] == '\\') {
                    // this close token is escaped. remove the backslash and continue.
                    expression.append(src, offset, end - offset - 1).append(closeToken);
                    offset = end + closeToken.length();
                    end = text.indexOf(closeToken, offset);
                } else {
                    expression.append(src, offset, end - offset);
                    offset = end + closeToken.length();
                    break;
                }
            }
            if (end == -1) {
                // close token was not found.
                builder.append(src, start, src.length - start);
                offset = src.length;
            } else {
                builder.append(handler.handleToken(expression.toString()));
                offset = end + closeToken.length();
            }
        }
        start = text.indexOf(openToken, offset);
    }
    if (offset < src.length) {
        builder.append(src, offset, src.length - offset);
    }
    return builder.toString();
}

```

