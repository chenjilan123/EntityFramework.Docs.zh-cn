---
title: 空间数据-EF Core
author: bricelam
ms.author: bricelam
ms.date: 11/01/2018
ms.assetid: 2BDE29FC-4161-41A0-841E-69F51CCD9341
uid: core/modeling/spatial
ms.openlocfilehash: cf488c6b7d94ca19018efe1c23ff410fe7eb594b
ms.sourcegitcommit: 81c53ac43d8f15b900f117294ec71dc49fe028fa
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2018
ms.locfileid: "51817905"
---
# <a name="spatial-data"></a>空间数据

> [!NOTE]
> 此功能是 EF Core 2.2 中的新增功能。

空间数据表示的物理位置和形状的对象。 多个数据库对于此类型的数据提供支持，因此可编制索引和查询以及其他数据。 常见方案包括从一个位置，给定距离内的对象的查询或选择其边框包含给定的位置的对象。 EF Core 支持使用空间数据类型映射[NetTopologySuite](https://github.com/NetTopologySuite/NetTopologySuite)空间库。

## <a name="installing"></a>安装

若要使用 EF Core 使用空间数据，您需要安装相应的支持 NuGet 包。 需要安装哪些程序包依赖于正在使用的提供程序。

EF Core 提供程序                        | 空间 NuGet 包
--------------------------------------- | ---------------------
Microsoft.EntityFrameworkCore.SqlServer | [Microsoft.EntityFrameworkCore.SqlServer.NetTopologySuite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer.NetTopologySuite)
Microsoft.EntityFrameworkCore.Sqlite    | [Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite)
Microsoft.EntityFrameworkCore.InMemory  | [NetTopologySuite](https://www.nuget.org/packages/NetTopologySuite)
Npgsql.EntityFrameworkCore.PostgreSQL   | [Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite)

## <a name="reverse-engineering"></a>反向工程

空间 NuGet 包还启用[反向工程](../managing-schemas/scaffolding.md)模型空间属性，但你需要安装包***之前***运行`Scaffold-DbContext`或`dotnet ef dbcontext scaffold`。 如果不这样做，你将收到有关未找到类型映射的列的警告，并将跳过列。

## <a name="nettopologysuite-nts"></a>NetTopologySuite (NTS)

NetTopologySuite 是用于.NET 的空间库。 EF Core，映射到空间数据的数据库中的类型在模型中使用 NTS 类型。

若要启用通过 NTS 空间类型的映射，请对提供程序的 DbContext 选项生成器调用 UseNetTopologySuite 方法。 例如，与 SQL Server 您会这样调用它。

``` csharp
optionsBuilder.UseSqlServer(
    @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=WideWorldImporters",
    x => x.UseNetTopologySuite());
```

有几种空间数据类型。 使用哪种类型取决于你想要允许的形状的类型。 下面是可用于在模型中的属性的 NTS 类型的层次结构。 它们位于`NetTopologySuite.Geometries`命名空间。 GeoAPI 包中的相应接口 (`GeoAPI.Geometries`命名空间) 也可用。

* geometry
  * 点
  * LineString
  * 多边形
  * GeometryCollection
    * MultiPoint
    * MultiLineString
    * MultiPolygon

> [!WARNING]
> 不支持 NTS CircularString、 CompoundCurve、 和 CurePolygon。

使用基本的 Geometry 类型允许任何类型的形状中，以指定的属性。

可以使用以下实体类将映射到表中[Wide World Importers 示例数据库](http://go.microsoft.com/fwlink/?LinkID=800630)。

``` csharp
[Table("Cities", Schema = "Application"))]
class City
{
    public int CityID { get; set; }

    public string CityName { get; set; }

    public IPoint Location { get; set; }
}

[Table("Countries", Schema = "Application"))]
class Country
{
    public int CountryID { get; set; }

    public string CountryName { get; set; }

    // Database includes both Polygon and MultiPolygon values
    public IGeometry Border { get; set; }
}
```

### <a name="creating-values"></a>创建值

您可以使用构造函数创建 geometry 对象;但是，NTS 建议改为使用 geometry 工厂。 由此你可以指定默认 SRID （所使用的坐标空间引用系统），并为您提供了控制精度模型 （在计算过程中使用） 和坐标序列 （确定哪些坐标-维度等更高级的功能和度量值-是可用）。

``` csharp
var geometryFactory = NtsGeometryServices.Instance.CreateGeometryFactory(srid: 4326);
var currentLocation = geometryFactory.CreatePoint(-122.121512, 47.6739882);
```

> [!NOTE]
> 4326 指 WGS 84，GPS 和其他地理系统中使用的标准。

### <a name="longitude-and-latitude"></a>经度和纬度

在 NTS 坐标都的 X 和 Y 值。 若要表示经度和纬度，经度和 Y 的纬度使用 X。 请注意，这是**向后**从`latitude, longitude`通常看到这些值的格式。

### <a name="srid-ignored-during-client-operations"></a>在客户端操作期间忽略的 SRID

NTS 在操作期间忽略 SRID 值。 它假定平面坐标系统。 这意味着，如果指定经度和纬度，距离、 长度和区域将以度为单位计量不像某些客户端评估值方面的坐标。 对于更有意义的值，首先需要项目到另一个使用等库的坐标系统的坐标[ProjNet4GeoAPI](https://github.com/NetTopologySuite/ProjNet4GeoAPI)之前计算这些值。

如果某个操作是由 EF Core 通过 SQL server 评估，将由数据库决定结果的单位。

下面是使用 ProjNet4GeoAPI 计算两个城市之间的距离的示例。

``` csharp
static class GeometryExtensions
{
    static readonly IGeometryServices _geometryServices = NtsGeometryServices.Instance;
    static readonly ICoordinateSystemServices _coordinateSystemServices
        = new CoordinateSystemServices(
            new CoordinateSystemFactory(),
            new CoordinateTransformationFactory(),
            new Dictionary<int, string>
            {
                // Coordinate systems:

                // (3857 and 4326 included automatically)

                // This coordinate system covers the area of our data.
                // Different data requires a different coordinate system.
                [2855] =
                @"
                    PROJCS[""NAD83(HARN) / Washington North"",
                        GEOGCS[""NAD83(HARN)"",
                            DATUM[""NAD83_High_Accuracy_Regional_Network"",
                                SPHEROID[""GRS 1980"",6378137,298.257222101,
                                    AUTHORITY[""EPSG"",""7019""]],
                                AUTHORITY[""EPSG"",""6152""]],
                            PRIMEM[""Greenwich"",0,
                                AUTHORITY[""EPSG"",""8901""]],
                            UNIT[""degree"",0.01745329251994328,
                                AUTHORITY[""EPSG"",""9122""]],
                            AUTHORITY[""EPSG"",""4152""]],
                        PROJECTION[""Lambert_Conformal_Conic_2SP""],
                        PARAMETER[""standard_parallel_1"",48.73333333333333],
                        PARAMETER[""standard_parallel_2"",47.5],
                        PARAMETER[""latitude_of_origin"",47],
                        PARAMETER[""central_meridian"",-120.8333333333333],
                        PARAMETER[""false_easting"",500000],
                        PARAMETER[""false_northing"",0],
                        UNIT[""metre"",1,
                            AUTHORITY[""EPSG"",""9001""]],
                        AUTHORITY[""EPSG"",""2855""]]
                "
            });

    public static IGeometry ProjectTo(this IGeometry geometry, int srid)
    {
        var geometryFactory = _geometryServices.CreateGeometryFactory(srid);
        var transformation = _coordinateSystemServices.CreateTransformation(geometry.SRID, srid);

        return GeometryTransform.TransformGeometry(
            geometryFactory,
            geometry,
            transformation.MathTransform);
    }
}
```

``` csharp
var seattle = new Point(-122.333056, 47.609722) { SRID = 4326 };
var redmond = new Point(-122.123889, 47.669444) { SRID = 4326 };

var distance = seattle.ProjectTo(2855).Distance(redmond.ProjectTo(2855));
```

## <a name="querying-data"></a>查询数据

在 LINQ 中，NTS 方法和属性可用作数据库函数都将转换为 SQL。 例如，下列查询中转换的距离和 Contains 方法。 本文结束时表显示了所支持的各种 EF Core 提供程序的成员。

``` csharp
var nearestCity = db.Cities
    .OrderBy(c => c.Location.Distance(currentLocation))
    .FirstOrDefault();

var currentCountry = db.Countries
    .FirstOrDefault(c => c.Border.Contains(currentLocation));
```

## <a name="sql-server"></a>SQL Server

如果您使用 SQL Server，有一些其他操作，您应注意。

### <a name="geography-or-geometry"></a>Geography 或 geometry

默认情况下，空间属性映射到`geography`SQL Server 中的列。 若要使用`geometry`，[配置的列类型](xref:core/modeling/relational/data-types)在模型中。

### <a name="geography-polygon-rings"></a>地理多边形环

当使用`geography`列类型，SQL Server 将对施加其他要求的外部环 （或命令行程序） 和内部环 （或漏洞）。 外部环必须为方向逆时针和内环顺时针旋转。 NTS 将值发送到数据库之前会验证此。

### <a name="fullglobe"></a>FullGlobe

SQL Server 具有非标准的几何图形类型来表示整个地球时使用`geography`列类型。 它还具有一种方法来表示多边形根据 （而无需外部环） 完整的球形。 NTS 支持这两个命令。

> [!WARNING]
> 不支持 NTS FullGlobe 和基于它的多边形。

## <a name="sqlite"></a>SQLite

下面是一些其他信息，这对于使用 SQLite。

### <a name="installing-spatialite"></a>安装 SpatiaLite

在 Windows 中，本机 mod_spatialite 库作为 NuGet 包依赖项分发。 其他平台需要单独安装。 这通常是使用软件程序包管理器。 例如，您可以在 Ubuntu 和 Homebrew 在 MacOS 上使用 APT。

``` sh
# Ubuntu
apt-get install libsqlite3-mod-spatialite

# macOS
brew install libspatialite
```

### <a name="configuring-srid"></a>配置 SRID

在 SpatiaLite，需要指定每个列 SRID 列。 默认 SRID 为`0`。 指定不同的 SRID 使用 ForSqliteHasSrid 方法。

``` csharp
modelBuilder.Entity<City>().Property(c => c.Location)
    .ForSqliteHasSrid(4326);
```

### <a name="dimension"></a>维度

类似于 SRID，列的维度 （或坐标） 还指定为列的一部分。 默认坐标为 X 和 Y。 启用其他坐标 （Z 和 M） 使用 ForSqliteHasDimension 方法。

``` csharp
modelBuilder.Entity<City>().Property(c => c.Location)
    .ForSqliteHasDimension(Ordinates.XYZ);
```

## <a name="translated-operations"></a>翻译后的操作

此表显示了哪些 NTS 成员的每个 EF Core 提供程序转换为 SQL。

NetTopologySuite | SQL Server (geometry) | SQL Server (geography) | SQLite | Npgsql
--- |:---:|:---:|:---:|:---:
Geometry.Area | ✔ | ✔ | ✔ | ✔
Geometry.AsBinary() | ✔ | ✔ | ✔ | ✔
Geometry.AsText() | ✔ | ✔ | ✔ | ✔
Geometry.Boundary | ✔ | | ✔ | ✔
Geometry.Buffer(double) | ✔ | ✔ | ✔ | ✔
Geometry.Buffer (double，int) | | | ✔
Geometry.Centroid | ✔ | | ✔ | ✔
Geometry.Contains(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.ConvexHull() | ✔ | ✔ | ✔ | ✔
Geometry.CoveredBy(Geometry) | | | ✔ | ✔
Geometry.Covers(Geometry) | | | ✔ | ✔
Geometry.Crosses(Geometry) | ✔ | | ✔ | ✔
Geometry.Difference(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.Dimension | ✔ | ✔ | ✔ | ✔
Geometry.Disjoint(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.Distance(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.Envelope | ✔ | | ✔ | ✔
Geometry.EqualsExact(Geometry) | | | | ✔
Geometry.EqualsTopologically(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.GeometryType | ✔ | ✔ | ✔ | ✔
Geometry.GetGeometryN(int) | ✔ | | ✔ | ✔
Geometry.InteriorPoint | ✔ | | ✔
Geometry.Intersection(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.Intersects(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.IsEmpty | ✔ | ✔ | ✔ | ✔
Geometry.IsSimple | ✔ | | ✔ | ✔
Geometry.IsValid | ✔ | ✔ | ✔ | ✔
Geometry.IsWithinDistance (Geometry，double) | ✔ | | ✔
Geometry.Length | ✔ | ✔ | ✔ | ✔
Geometry.NumGeometries | ✔ | ✔ | ✔ | ✔
Geometry.NumPoints | ✔ | ✔ | ✔ | ✔
Geometry.OgcGeometryType | ✔ | ✔ | ✔
Geometry.Overlaps(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.PointOnSurface | ✔ | | ✔ | ✔
Geometry.Relate (Geometry，字符串) | ✔ | | ✔ | ✔
Geometry.Reverse() | | | ✔ | ✔
Geometry.SRID | ✔ | ✔ | ✔ | ✔
Geometry.SymmetricDifference(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.ToBinary() | ✔ | ✔ | ✔ | ✔
Geometry.ToText() | ✔ | ✔ | ✔ | ✔
Geometry.Touches(Geometry) | ✔ | | ✔ | ✔
Geometry.Union() | | | ✔
Geometry.Union(Geometry) | ✔ | ✔ | ✔ | ✔
Geometry.Within(Geometry) | ✔ | ✔ | ✔ | ✔
GeometryCollection.Count | ✔ | ✔ | ✔ | ✔
GeometryCollection [int] | ✔ | ✔ | ✔ | ✔
LineString.Count | ✔ | ✔ | ✔ | ✔
LineString.EndPoint | ✔ | ✔ | ✔ | ✔
LineString.GetPointN(int) | ✔ | ✔ | ✔ | ✔
LineString.IsClosed | ✔ | ✔ | ✔ | ✔
LineString.IsRing | ✔ | | ✔ | ✔
LineString.StartPoint | ✔ | ✔ | ✔ | ✔
MultiLineString.IsClosed | ✔ | ✔ | ✔ | ✔
Point.M | ✔ | ✔ | ✔ | ✔
Point.X | ✔ | ✔ | ✔ | ✔
Point.Y | ✔ | ✔ | ✔ | ✔
Point.Z | ✔ | ✔ | ✔ | ✔
Polygon.ExteriorRing | ✔ | ✔ | ✔ | ✔
Polygon.GetInteriorRingN(int) | ✔ | ✔ | ✔ | ✔
Polygon.NumInteriorRings | ✔ | ✔ | ✔ | ✔

## <a name="additional-resources"></a>其他资源

* [SQL Server 中的空间数据](https://docs.microsoft.com/sql/relational-databases/spatial/spatial-data-sql-server)
* [SpatiaLite 主页](https://www.gaia-gis.it/fossil/libspatialite)
* [Npgsql 空间文档](http://www.npgsql.org/efcore/mapping/nts.html)
* [PostGIS 文档](http://postgis.net/documentation/)
