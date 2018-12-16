---
title: 在团队环境的 EF Core中迁移
author: bricelam
ms.author: bricelam
ms.date: 10/30/2017
ms.openlocfilehash: e8ff7f468d5ab6dbd6285f1abf9199e413288d10
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997690"
---
<a name="migrations-in-team-environments"></a>在团队环境中的迁移
===============================
在团队环境中使用迁移时，需额外注意模型快照文件。 该文件会告诉你团队成员的迁移是否能够与你的迁移顺利合并，以及你是否需要通过重新创建迁移来解决代码冲突，然后再进行共享。

<a name="merging"></a>合并
-------
合并团队成员的迁移时，可能会在模型快照文件中出现冲突。 如果两边的变更不相关联且合并规模较小，则两个迁移可以共存。 例如，可能会在客户实体类型配置中出现如下所示的合并冲突：

    <<<<<<< Mine
    b.Property<bool>("Deactivated");
    =======
    b.Property<int>("LoyaltyPoints");
    >>>>>>> Theirs

由于这两个属性需要在最终模型中共存，因此请同时添加这两个属性来完成合并。 在许多情况下，版本控制系统可能会自动为你合并此类更改。

``` csharp
b.Property<bool>("Deactivated");
b.Property<int>("LoyaltyPoints");
```

在此类情况下，你的迁移和团队成员的迁移是相互独立的。 由于可以首先应用任一迁移，因此在与团队共享你的迁移之前，无需对其进行更多的更改。

<a name="resolving-conflicts"></a>解决冲突
-------------------
有时会在合并模型快照文件的过程中遇到真正的冲突。 例如，你和团队成员可能重命名了同一属性。

    <<<<<<< Mine
    b.Property<string>("Username");
    =======
    b.Property<string>("Alias");
    >>>>>>> Theirs

如果遇到这种冲突，请通过重新创建你的迁移来解决。 请执行以下步骤：

1. 中止合并，回退到合并前的工作目录
2. 删除迁移 （但保留模型更改）
3. 将团队成员的更改合并到你的工作目录
4. 重新添加迁移

完成后，可以按正确的顺序应用两个迁移。 先应用团队成员的迁移，将该列重命名为 *Alias*，然后你的迁移将其重命名为 *Username*。

你的迁移可以安全地与团队的其他人共享。
