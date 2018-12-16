---
title: 级联删除 - EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: ee8e14ec-2158-4c9c-96b5-118715e2ed9e
uid: core/saving/cascade-delete
ms.openlocfilehash: 15b7e69676ef9aeb70121fcec404c34a17e5e2bb
ms.sourcegitcommit: 8d04a2ad98036f32ca70c77ce3040c5edb1cdf82
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/11/2018
ms.locfileid: "44384834"
---
# <a name="cascade-delete"></a>级联删除

级联删除通常在数据库术语中用来描述一种允许在删除某行时自动触发删除相关行的特性。 EF Core 删除行为还介绍了一个密切相关的概念，即子实体与父实体的关系已断开时自动删除该子实体，这通常称为“删除孤立项”。

EF Core 实现多种不同的删除行为，并允许配置各个关系的删除行为。 EF Core 还实现基于[关系的需求](../modeling/relationships.md#required-and-optional-relationships)为每个关系自动配置有用的默认删除行为的约定。

## <a name="delete-behaviors"></a>删除行为
删除行为在 *DeleteBehavior* 枚举器类型中定义，并且可以传递到 *OnDelete* Fluent API 来控制是主体/父实体的删除还是依赖实体/子实体关系的断开会对依赖实体/子实体产生副作用。

删除主体/父实体或断开与子实体的关系时，EF 可以采取三个操作：
* 可以删除子项/依赖项
* 子项的外键值可以设置为 null
* 子项保持不变

> [!NOTE]  
> 仅当使用 EF Core 删除主体且将依赖实体加载到内存中（即对于跟踪的依赖项）时才应用 EF Core 模型中配置的删除行为。 需要在数据库中设置相应的级联行为以确保未由上下文跟踪的数据已应用必要的操作。 如果使用 EF Core 创建数据库，将为你设置此级联行为。

对于上述第二个操作，如果外键不可以为 null，则将外键值设置为 null 是无效的。 （不可为 null 的外键相当于必需关系。）在这些情况下，EF Core 会跟踪外键属性是否已被标记为 null，直到调用 SaveChanges，此时会引发异常，因为无法将更改永久保存到数据库中。 这类似于从数据库中获取约束冲突。

有四个删除行为，如下表中列出。

### <a name="optional-relationships"></a>可选关系
对于可选关系（可以为 null 的外键），可以保存 null 外键值，从而产生以下影响：

| 行为名称               | 对内存中的依赖项/子项的影响    | 对数据库中的依赖项/子项的影响  |
|:----------------------------|:---------------------------------------|:---------------------------------------|
| **Cascade**                 | 删除实体                   | 删除实体                   |
| **ClientSetNull**（默认） | 外键属性设置为 null | 无                                   |
| **SetNull**                 | 外键属性设置为 null | 外键属性设置为 null |
| **Restrict**                | 无                                   | 无                                   |

### <a name="required-relationships"></a>必选关系
对于必选关系（不可为 null 的外键），_不可以_保存 null 外键值，从而产生以下影响：

| 行为名称         | 对内存中的依赖项/子项的影响 | 对数据库中的依赖项/子项的影响 |
|:----------------------|:------------------------------------|:--------------------------------------|
| **Cascade**（默认） | 删除实体                | 删除实体                  |
| **ClientSetNull**     | SaveChanges 引发异常                  | 无                                  |
| **SetNull**           | SaveChanges 引发异常                  | SaveChanges 引发异常                    |
| **Restrict**          | 无                                | 无                                  |

在上表中，“无”可能会造成约束冲突。 例如，如果已删除主体/子实体，但不执行任何操作来更改依赖项/子项的外键，则由于发生外键约束冲突，数据库将可能在 SaveChanges 时引发异常。

高级别：
* 如果实体在没有父项时不能存在，且希望 EF 负责自动删除子项，则使用“Cascade”。
  * 在没有父项时不能存在的实体通常使用必选关系，其中“Cascade”是默认值。
* 如果实体可能有或可能没有父项，且希望 EF 负责为你将外键变为 null，则使用“ClientSetNull”
  * 在没有父项时可以存在的实体通常使用可选关系，其中“ClientSetNull”是默认值。
  * 如果希望数据库即使在未加载子实体时也尝试将 null 值传播到子外键，则使用“SetNull”。 但是，请注意，数据库必须支持此操作，并且如此配置数据库可能会导致其他限制，实际上这通常会使此选项不适用。 这就是*SetNull*不是默认值的原因。
* 如果不希望 EF Core 始终自动删除实体或自动将外键变为 null，则使用“Restrict”。 请注意，这要求使用代码手动同步子实体及其外键值，否则将引发约束异常。

> [!NOTE]
> 在 EF Core（与 EF6 不同）中，不会立即产生级联影响，而是仅在调用 SaveChanges 时产生。

> [!NOTE]  
> **EF Core 2.0 中的更改：** 在以前版本中，“Restrict”将导致跟踪的依赖实体中的可选外键属性设置为 null，并且是可选关系的默认删除行为。 在 EF Core 2.0 中，引入了“ClientSetNull”以表示该行为，并且它会成为可选关系的默认值。 “Restrict”的行为已调整为永远不会对依赖实体产生副作用。

## <a name="entity-deletion-examples"></a>实体删除示例

以下代码是可下载并运行的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/CascadeDelete/)的一部分。 此示例显示了，当删除父实体时，可选关系和必选关系的每个删除行为会发生的情况。

[!code-csharp[Main](../../../samples/core/Saving/Saving/CascadeDelete/Sample.cs#DeleteBehaviorVariations)]

我们来看一看每个变化以了解所发生的情况。

### <a name="deletebehaviorcascade-with-required-or-optional-relationship"></a>具有必选或可选关系的 DeleteBehavior.Cascade

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After deleting blog '1':
    Blog '1' is in state Deleted with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  Saving changes:
    DELETE FROM [Posts] WHERE [PostId] = 1
    DELETE FROM [Posts] WHERE [PostId] = 2
    DELETE FROM [Blogs] WHERE [BlogId] = 1

  After SaveChanges:
    Blog '1' is in state Detached with 2 posts referenced.
      Post '1' is in state Detached with FK '1' and no reference to a blog.
      Post '2' is in state Detached with FK '1' and no reference to a blog.
```

* 博客标记为已删除
* 文章最初保持不变，因为在调用 SaveChanges 之前不会发生级联
* SaveChanges 发送对依赖项/子项（文章）和主体/父项（博客）的删除
* 保存后，所有实体都会分离，因为它们现在已从数据库中删除

### <a name="deletebehaviorclientsetnull-or-deletebehaviorsetnull-with-required-relationship"></a>具有必选关系的 DeleteBehavior.ClientSetNull 或 DeleteBehavior.SetNull

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After deleting blog '1':
    Blog '1' is in state Deleted with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  Saving changes:
    UPDATE [Posts] SET [BlogId] = NULL WHERE [PostId] = 1

  SaveChanges threw DbUpdateException: Cannot insert the value NULL into column 'BlogId', table 'EFSaving.CascadeDelete.dbo.Posts'; column does not allow nulls. UPDATE fails. The statement has been terminated.
```

* 博客标记为已删除
* 文章最初保持不变，因为在调用 SaveChanges 之前不会发生级联
* SaveChanges 尝试将文章外键设置为 null，但会失败，因为外键不可以为 null

### <a name="deletebehaviorclientsetnull-or-deletebehaviorsetnull-with-optional-relationship"></a>具有可选关系的 DeleteBehavior.ClientSetNull 或 DeleteBehavior.SetNull

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After deleting blog '1':
    Blog '1' is in state Deleted with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  Saving changes:
    UPDATE [Posts] SET [BlogId] = NULL WHERE [PostId] = 1
    UPDATE [Posts] SET [BlogId] = NULL WHERE [PostId] = 2
    DELETE FROM [Blogs] WHERE [BlogId] = 1

  After SaveChanges:
    Blog '1' is in state Detached with 2 posts referenced.
      Post '1' is in state Unchanged with FK 'null' and no reference to a blog.
      Post '2' is in state Unchanged with FK 'null' and no reference to a blog.
```

* 博客标记为已删除
* 文章最初保持不变，因为在调用 SaveChanges 之前不会发生级联
* SaveChanges 尝试在删除主体/父项（博客）之前将依赖项/子项（文章）的外键设置为 null
* 保存后，将删除主体/父项（博客），但仍会跟踪依赖项/子项（文章）
* 跟踪的依赖项/子项（文章）现在具有 null 外键值，并且对删除的主体/父项（博客）的引用已删除

### <a name="deletebehaviorrestrict-with-required-or-optional-relationship"></a>具有必选或可选关系的 DeleteBehavior.Restrict

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After deleting blog '1':
    Blog '1' is in state Deleted with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  Saving changes:
  SaveChanges threw InvalidOperationException: The association between entity types 'Blog' and 'Post' has been severed but the foreign key for this relationship cannot be set to null. If the dependent entity should be deleted, then setup the relationship to use cascade deletes.
```

* 博客标记为已删除
* 文章最初保持不变，因为在调用 SaveChanges 之前不会发生级联
* 由于*Restrict*指告知 EF 不要自动将外键设置为 null，因此它保持为非 null，而 SaveChanges 将引发异常但不进行保存

## <a name="delete-orphans-examples"></a>删除孤立项示例

以下代码是可下载并运行的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/CascadeDelete/)的一部分。 此示例显示了，当断开父项/主体及其子项/依赖项之间的关系时，可选关系和必选关系的每个删除行为会发生的情况。 在此示例中，通过从主体/父项（博客）上的集合导航属性中删除依赖项/子项（文章）来断开关系。 但是，如果将从依赖项/子项到主体/父项的引用变为 null，则行为相同。

[!code-csharp[Main](../../../samples/core/Saving/Saving/CascadeDelete/Sample.cs#DeleteOrphansVariations)]

我们来看一看每个变化以了解所发生的情况。

### <a name="deletebehaviorcascade-with-required-or-optional-relationship"></a>具有必选或可选关系的 DeleteBehavior.Cascade

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After making posts orphans:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Modified with FK '1' and no reference to a blog.
      Post '2' is in state Modified with FK '1' and no reference to a blog.

  Saving changes:
    DELETE FROM [Posts] WHERE [PostId] = 1
    DELETE FROM [Posts] WHERE [PostId] = 2

  After SaveChanges:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Detached with FK '1' and no reference to a blog.
      Post '2' is in state Detached with FK '1' and no reference to a blog.
```

* 文章标记为已修改，因为断开关系导致外键标记为 null
  * 如果外键不可以为 null，则即使实际值标记为 null 也不会更改
* SaveChanges 发送对依赖项/子项（文章）的删除
* 保存后，依赖项/子项（文章）会分离，因为它们现在已从数据库中删除

### <a name="deletebehaviorclientsetnull-or-deletebehaviorsetnull-with-required-relationship"></a>具有必选关系的 DeleteBehavior.ClientSetNull 或 DeleteBehavior.SetNull

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After making posts orphans:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Modified with FK 'null' and no reference to a blog.
      Post '2' is in state Modified with FK 'null' and no reference to a blog.

  Saving changes:
    UPDATE [Posts] SET [BlogId] = NULL WHERE [PostId] = 1

  SaveChanges threw DbUpdateException: Cannot insert the value NULL into column 'BlogId', table 'EFSaving.CascadeDelete.dbo.Posts'; column does not allow nulls. UPDATE fails. The statement has been terminated.
```

* 文章标记为已修改，因为断开关系导致外键标记为 null
  * 如果外键不可以为 null，则即使实际值标记为 null 也不会更改
* SaveChanges 尝试将文章外键设置为 null，但会失败，因为外键不可以为 null

### <a name="deletebehaviorclientsetnull-or-deletebehaviorsetnull-with-optional-relationship"></a>具有可选关系的 DeleteBehavior.ClientSetNull 或 DeleteBehavior.SetNull

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After making posts orphans:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Modified with FK 'null' and no reference to a blog.
      Post '2' is in state Modified with FK 'null' and no reference to a blog.

  Saving changes:
    UPDATE [Posts] SET [BlogId] = NULL WHERE [PostId] = 1
    UPDATE [Posts] SET [BlogId] = NULL WHERE [PostId] = 2

  After SaveChanges:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK 'null' and no reference to a blog.
      Post '2' is in state Unchanged with FK 'null' and no reference to a blog.
```

* 文章标记为已修改，因为断开关系导致外键标记为 null
  * 如果外键不可以为 null，则即使实际值标记为 null 也不会更改
* SaveChanges 将依赖项/子项（文章）的外键设置为 null
* 保存后，依赖项/子项（文章）现在具有 null 外键值，并且对删除的主体/父项（博客）的引用已删除

### <a name="deletebehaviorrestrict-with-required-or-optional-relationship"></a>具有必选或可选关系的 DeleteBehavior.Restrict

```
  After loading entities:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Unchanged with FK '1' and reference to blog '1'.
      Post '2' is in state Unchanged with FK '1' and reference to blog '1'.

  After making posts orphans:
    Blog '1' is in state Unchanged with 2 posts referenced.
      Post '1' is in state Modified with FK '1' and no reference to a blog.
      Post '2' is in state Modified with FK '1' and no reference to a blog.

  Saving changes:
  SaveChanges threw InvalidOperationException: The association between entity types 'Blog' and 'Post' has been severed but the foreign key for this relationship cannot be set to null. If the dependent entity should be deleted, then setup the relationship to use cascade deletes.
```

* 文章标记为已修改，因为断开关系导致外键标记为 null
  * 如果外键不可以为 null，则即使实际值标记为 null 也不会更改
* 由于*Restrict*指告知 EF 不要自动将外键设置为 null，因此它保持为非 null，而 SaveChanges 将引发异常但不进行保存

## <a name="cascading-to-untracked-entities"></a>级联到未跟踪的实体

调用“SaveChanges”时，级联删除规则将应用于由上下文跟踪的所有实体。 上述所有示例都符合该情况，即生成用于删除主体/父项（博客）和所有依赖项/子项（文章）的 SQL 的原因：

```sql
    DELETE FROM [Posts] WHERE [PostId] = 1
    DELETE FROM [Posts] WHERE [PostId] = 2
    DELETE FROM [Blogs] WHERE [BlogId] = 1
```

如果仅加载主体（例如，当为不含 `Include(b => b.Posts)` 的博客创建查询以包含文章时），则 SaveChanges 只会生成用于删除主体/父项的 SQL：

```sql
    DELETE FROM [Blogs] WHERE [BlogId] = 1
```

只有在数据库已配置相应的级联行为后，才会删除依赖项/子项（文章）。 如果使用 EF 创建数据库，则会为你设置此级联行为。
