---
title: Anko SQLite
date: 2017-11-03 11:05:26
tags:
- Android
- Anko
- Kotlin
categories:
- Anko

---

![anko](https://github.com/Kotlin/anko/blob/master/doc/logo.png?raw=true)

您是否厌倦了使用Android Cursor解析SQLite查询结果?
您必须编写大量的样板代码来解析查询结果行，并将其封装在无数的try..finally块中，以便正确地关闭所有打开的资源。

Anko提供了许多扩展功能，以简化与SQLite数据库的工作。

## 目录 ##

* [在项目中使用Anko SQLite](#using_anko_sqlite)
* [访问数据库](#access_database)
* [创建和删除表](#creat_drop_tables)
* [插入数据](#insert_data)
* [查询数据](#query_data)
* [解析查询结果](#parsing_query)
* [自定义行解析器](#custom_row_parsers)
* [Cursor流](#cursor_streams)
* [更新值](#updat_values)
* [事务](#transactions)

<!-- more -->

## <a name="using_anko_sqlite"></a>在项目中使用Anko SQLite

将`anko-sqlite`的依赖添加到您的`build.gradle`:

```gradle
dependencies {
    compile "org.jetbrains.anko:anko-sqlite:$anko_version"
}
```

## <a name="access_database"></a>访问数据库

如果使用`SQLiteOpenHelper`，通常会调用`getReadableDatabase()`或`getWritableDatabase()`(在生产代码中结果是相同的)，但是必须确保在接收到的`SQLiteDatabase`上调用`close()`方法。
另外，您必须在某个地方缓存helper类，如果您在几个线程中使用它，那么您必须知道并发访问。
这一切都很艰难。
这就是为什么Android开发人员并不热衷于使用默认的SQLite API，而是更喜欢使用诸如ORMs之类的相当昂贵的包装器。

Anko提供了一个特殊的类ManagedSQLiteOpenHelper，它可以无缝地替换缺省值。以下是你如何使用它:

```
class MyDatabaseOpenHelper(ctx: Context) : ManagedSQLiteOpenHelper(ctx, "MyDatabase", null, 1) {
    companion object {
        private var instance: MyDatabaseOpenHelper? = null

        @Synchronized
        fun getInstance(ctx: Context): MyDatabaseOpenHelper {
            if (instance == null) {
                instance = MyDatabaseOpenHelper(ctx.getApplicationContext())
            }
            return instance!!
        }
    }

    override fun onCreate(db: SQLiteDatabase) {
        // 这里创建表
        db.createTable("Customer", ifNotExists = true, 
                    "id" to INTEGER + PRIMARY_KEY + UNIQUE,
                    "name" to TEXT,
                    "photo" to BLOB)
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        // 在这里，您可以像往常一样删除表
        db.dropTable("User", true)
    }
}

// 访问属性的Context
val Context.database: MyDatabaseOpenHelper
    get() = MyDatabaseOpenHelper.getInstance(getApplicationContext())
```

什么感觉?不要将你的代码封装到try块中，现在你可以这样写:

```
database.use {
    // `this` is a SQLiteDatabase instance
}
```
在`{}`里执行了所有的代码之后，数据库将会被关闭。

异步调用的例子:

```
class SomeActivity : Activity() {
    private fun loadAsync() {
        async(UI) {
            val result = bg { 
                database.use { ... }
            }
            loadComplete(result)
        }
    }
}
```

> 🐧 下面提到的这些方法和所有方法可能会抛出SQLiteException。
> 你必须自己处理，因为Anko假装错误不会发生，这是不合理的。

## <a name="creat_drop_tables"></a>创建和删除表

对于Anko，你可以很容易地创建新的表，并删除现有的表。语法很简单。

```
database.use {
    createTable("Customer", true, 
        "id" to INTEGER + PRIMARY_KEY + UNIQUE,
        "name" to TEXT,
        "photo" to BLOB)
}
```

在SQLite中，有5种主要类型:`NULL`、`INTEGER`、`REAL`、`TEXT`和`BLOB`。
但是每一列可能都有一些修饰符，比如主键(`PRIMARY KEY`)或惟一(`UNIQUE`)的。
您可以将这些修饰符附加到主类型名中。

要删除一个表，使用dropTable函数:

```
dropTable("User", true)
```

## <a name="insert_data"></a>插入数据

通常，您需要一个ContentValues实例来将行插入到表中。
这是一个例子:

```
val values = ContentValues()
values.put("id", 5)
values.put("name", "John Smith")
values.put("email", "user@domain.org")
db.insert("User", null, values)
```

Anko允许消除这些仪式,通过直接把值作为insert()函数的参数:

```
// Where db is an SQLiteDatabase
// eg: val db = database.writeableDatabase
db.insert("User", 
    "id" to 42,
    "name" to "John",
    "email" to "user@domain.org"
)
```

或者使用database.use:

```
database.use {
    insert("User", 
        "id" to 42,
        "name" to "John",
        "email" to "user@domain.org"
}
```

请注意，在上面的示例中，`database`是一个数据库助手实例，而`db`是一个`SQLiteDatabase`对象

函数`insertOrThrow()`, `replace()`, `replaceOrThrow()`也存在并具有类似的语义。

## <a name="query_data"></a>查询数据

Anko提供了一个方便的查询构建器。
它可能是用db.select(tableName, vararg columns)，db是SQLiteDatabase的一个实例。

方法 | 描述
-----|-----
`column(String)`	| 添加一个用于选择查询的列
`distinct(Boolean)`	| 	不同的查询
`whereArgs(String)`	| 	指定原始字符串`where`查询
`whereArgs(String, args)`	✨| 指定带有参数的`where`查询
`whereSimple(String, args)`	| 	指定带有`?`标志参数的`where`查询
`orderBy(String, [ASC/DESC])`	| 	按列排序
`groupBy(String)`	| 	按列分组
`limit(count: Int)`	| 	限制查询结果行数
`limit(offset: Int, count: Int)`	| 	用偏移量限制查询结果行数
`having(String)`	| 	指定原始`having`表达式
`having(String, args)` ✨	| 	指定带有参数的`having`表达式


函数标记✨以一种特殊的方式解析它的参数。
它们允许您以任何顺序提供值，并支持无缝地转义。

```
db.select("User", "name")
    .whereArgs("(_id > {userId}) and (name = {userName})",
        "userName" to "John",
        "userId" to 42)
```

在这里，`{userId}`部分将被`42`替换，`{userName}`替换为`'John'`。
如果它的类型不是数值(`Int`、`Float`等)或`Boolean`，则该值将被转义。
对于其他类型，将被使用`toString()`表示。

`whereSimple`函数接受String类型的参数。
它与SQLiteDatabase中的query()相同(问号`?`将会被实值的实值所取代)

我们如何执行查询?
使用`exec()`函数。
它接受一个扩展函数，它的类型是`Cursor.() -> T`。
它只是启动接收的扩展函数，然后关闭Cursor，这样您就不需要自己动手了:

```
db.select("User", "email").exec {
	// 用电子邮件做一些事情
}
```

## <a name="parsing_query"></a>解析查询结果

因此，我们有一些Cursor，我们如何将它解析为普通类呢?
Anko提供了一些功能parseSingle, parseOpt和parseList，这样做更容易。

方法 | 描述
-----|-----
`parseSingle(rowParser): T` |	解析1行
`parseOpt(rowParser): T?` |	解析0或1行
`parseList(rowParser): List<T>` |	 解析0或多行

注意，`parseSingle()`和`parseOpt()`将抛出一个异常，如果接收到的Cursor包含超过一行。

现在的问题是:什么是`rowParser`?每个函数都支持两种不同类型的解析器: `RowParser`和`MapRowParser`:

```
interface RowParser<T> {
    fun parseRow(columns: Array<Any>): T
}

interface MapRowParser<T> {
    fun parseRow(columns: Map<String, Any>): T
}
```

如果您想以一种非常有效的方式编写查询，请使用RowParser(但是您必须知道每个列的索引)。
`parseRow`接受了一个`Any`列表(`Any`类型实际上都可以是Long、Double、String或ByteArray)。
另一方面，`MapRowParser`允许使用列名来获取行值。

Anko已经有了简单的单列行的解析器:

* `ShortParser`
* `IntParser`
* `LongParser`
* `FloatParser`
* `DoubleParser`
* `StringParser`
* `BlobParser`

同样，您可以从类构造函数创建row解析器。假设你有一个class:

```
class Person(val firstName: String, val lastName: String, val age: Int)
```

解析器将非常简单:

```
val rowParser = classParser<Person>()
```

现在，如果主构造函数有可选的参数，那么Anko`不支持`创建这样的解析器。
另外，请注意，构造函数将使用Java反射来调用，因此编写一个自定义的`RowParser`对于大型数据集来说是更合理的。

如果使用Anko db.select()构建器，可以直接调用`parseSingle`、`parseOpt`或`parseList`，并传递一个适当的解析器。

## <a name="custom_row_parsers"></a>自定义行解析器

例如，让我们为columns (Int, String, String)创建一个新的解析器。最幼稚的做法是:

```
class MyRowParser : RowParser<Triple<Int, String, String>> {
    override fun parseRow(columns: Array<Any>): Triple<Int, String, String> {
        return Triple(columns[0] as Int, columns[1] as String, columns[2] as String)
    }
}
```

现在，我们的代码中有三个显式的类型转换。让我们通过使用rowParser函数来摆脱它们:

```
val parser = rowParser { id: Int, name: String, email: String ->
    Triple(id, name, email)
}
```

是它!`rowParser`将所有的类型转换为底层，您可以根据您的需要来命名lambda参数。

## <a name="cursor_streams"></a> Cursor流

Anko提供了一种以功能方式访问SQLite Cursor的方法。
只需调用cursor.assequence()或cursormap.asmapsequence()扩展函数，就可以得到一个行序列。
不要忘记关闭Cursor :)

## <a name="updat_values"></a>更新值

让我们给我们的一个用户一个新名字:

```
update("User", "name" to "Alice")
    .where("_id = {userId}", "userId" to 42)
    .exec()
```

更新还有一个whereSimple()方法，以防您想要以传统方式提供查询:

```
update("User", "name" to "Alice")
    .whereSimple("_id = ?", 42)
    .exec()
```

## <a name="transactions"></a>事务

有一个名为`transaction()`的特殊函数，它允许您将多个数据库操作封装在一个SQLite事务中。

```
transaction {
    // 你的事务代码
}
```

如果在`{}`块中没有抛出异常，那么该事务将被标记为成功。

> 🐧 如果你想中止事务出于某种原因,抛出TransactionAbortException。
> 在这种情况下，您不需要自己处理这个异常。