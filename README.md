# d3-geo

地图投影有时作为点的转换实现。比如球形墨卡托:

```js
function mercator(x, y) {
  return [x, Math.log(Math.tan(Math.PI / 4 + y / 2))];
}
```

如果你的几何包含连续无限的点集时，这是一个合理的 *数学* 方法。然而，计算机并没有无限的内存，所以我们必须使用离散几何，比如多边形和折线!

离散几何使得从球面投影到平面更加困难。球面多边形的边缘是[geodesics(测地线)](https://en.wikipedia.org/wiki/Geodesic)(大圆的部分)，而不是直线。测地线投射到平面上，除了 [gnomonic](#geoGnomonic) 之外所有的地图投影都是曲线，因此精确的投影需要沿每条弧插值。`D3` 使用 [adaptive sampling(自适应采样)](https://bl.ocks.org/mbostock/3795544)灵感来 [line simplification method(线简化方法)](https://bost.ocks.org/mike/simplify/)，以平衡精度和性能。

多边形和折线的投影还必须处理球面与平面的拓扑差异。一些投影需要切割 [对向子午线](https://bl.ocks.org/mbostock/3788999) 的几何图形，而另一些投影则需要将几何图形 [裁剪为一个大圆](https://bl.ocks.org/mbostock/3021474)。

球面多边形还需要一个 [winding order convention(缠绕顺序约定)](https://bl.ocks.org/mbostock/a7bdfeb041e850799a8d3dce4d8c50c8) 来确定多边形的哪边是内边: 小于半球的多边形的外环必须是顺时针方向, 而 [larger than a hemisphere(大于半球的多边形的外环)](https://bl.ocks.org/mbostock/6713736) 必须是逆时针的. 代表孔的内圈必须使用与其外圈的相反的环绕顺序。这种环绕规则也被 [TopoJSON](https://github.com/topojson) 和 [ESRI shapefiles](https://github.com/mbostock/shapefile) 采用。但是，它与 `GeoJSON` 的 [RFC 7946]((https://tools.ietf.org/html/rfc7946#section-3.1.6)) **相反**。(另请注意，标准 `GeoJSON WGS84` 使用平面的等距坐标，而不是球面坐标，因此可能需要 [stitching](https://github.com/d3/d3-geo-projection/blob/master/README.md#geostitch) 以去除对对向子午线的切割)。

D3的方法提供了很好的表现力: 你可以为你的数据选择正确的投影和样貌。D3 支持许多常见的投影以及 [特殊的地理投影](https://github.com/d3/d3-geo-projection). 更多信息可以参考 [工具制作指南](https://vimeo.com/106198518#t=20m0s).

D3 使用 [GeoJSON](http://geojson.org/geojson-spec.html) 来表现地理特征. (也可以参考[TopoJSON](https://github.com/mbostock/topojson), 一种更加紧密的对 GeoJSON 的编码). 可以使用 [shp2geo](https://github.com/mbostock/shapefile/blob/master/README.md#shp2geo) 将 shape 文件转为 GeoJSON, 这是 [shapefile package](https://github.com/mbostock/shapefile) 的一部分. 有关 d3-geo 和相关工具的介绍，请参阅[ Command-Line Cartography](https://medium.com/@mbostock/command-line-cartography-part-1-897aa8f8ca2c).

## Installing

使用 `NPM`: `npm install d3-geo`. 此外还可以下载 [latest release](https://github.com/d3/d3-geo/releases/latest). 可以直接从 [d3js.org](https://d3js.org) 以 [standalone library](https://d3js.org/d3-geo.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分直接引入. 支持 `AMD`, `CommonJS` 和基础的标签引入形式. 如果使用标签引入会暴露 `d3` 全局变量:

```html
<script src="https://d3js.org/d3-array.v1.min.js"></script>
<script src="https://d3js.org/d3-geo.v1.min.js"></script>
<script>

var projection = d3.geoNaturalEarth1(),
    path = d3.geoPath(projection);

</script>
```

[在浏览器中测试 d3-geo.](https://tonicdev.com/npm/d3-geo)

## API Reference

* [Paths](#paths)
* [Projections](#projections) ([Azimuthal](#azimuthal-projections), [Composite](#composite-projections), [Conic](#conic-projections), [Cylindrical](#cylindrical-projections))
* [Raw Projections](#raw-projections)
* [Spherical Math](#spherical-math)
* [Spherical Shapes](#spherical-shapes)
* [Streams](#streams)
* [Transforms](#transforms)
* [Clipping](#clipping)

### Paths

地理路径生成器, [d3.geoPath](#geoPath) 与 [d3-shape](https://github.com/d3/d3-shape) 很像: 给定一个 GeoJSON 或者特征对象，会生成一个 `SVG` 路径字符串，也可以将其 [渲染到 Canvas](https://bl.ocks.org/mbostock/3783604) 上. 建议使用 Canvas 进行动态或者交互式投影以提高性能. 路径生成器可以与 [projections](#projections) 或 [transforms](#transforms) 一起使用，或者可以直接将平面几何呈现到 Canvas 或 SVG 中.

<a href="#geoPath" name="geoPath">#</a> d3.<b>geoPath</b>([<i>projection</i>[, <i>context</i>]]) [<>](https://github.com/d3/d3-geo/blob/master/src/path/index.js "Source")

使用默认的设置创建一个新的地理路径生成器. 如果指定了 *projection*, 则设置 [当前投影](#path_projection). 如果指定了 *context* 则设置当前 [当前上下文](#path_context).

<a href="#_path" name="_path">#</a> <i>path</i>(<i>object</i>[, <i>arguments…</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/path/index.js "Source")

渲染指定的 *object*, 可以是一个 GeoJSON 特征或者几何对象:

* Point - 单个点.
* MultiPoint - 一组点集.
* LineString - 一组点集表示的连续的线.
* MultiLineString - 组点集数组表示的多条线.
* Polygon - 点集数组表示的多边形(可能有镂空).
* MultiPolygon - 表示多个多边形的点集.
* GeometryCollection - 一组几何对象.
* Feature - 包含上述几何对象之一的特征.
* FeatureCollection - 一组集合特征.

也支持类型为 *Sphere*, 在绘制地球轮廓时很有用; 球体没有坐标系. 任何额外的参数都会传递给 [pointRadius](#path_pointRadius) 访问器.

显示多个特征时, 可以将其与特征集合结合:

```js
svg.append("path")
    .datum({type: "FeatureCollection", features: features})
    .attr("d", d3.geoPath());
```

或者使用多个 `path` 元素:

```js
svg.selectAll("path")
  .data(features)
  .enter().append("path")
    .attr("d", d3.geoPath());
```

多个 path 元素通常比单个 path 元素慢. 但是多个 path 在单独设置样式以及交互时很有用(比如点击和鼠标移入). Canvas 的渲染(可以参考[*path*.context](#path_context)) 通常比 SVG 更快, 但是需要更多的精力去设置样式和交互.

<a href="#path_area" name="path_area">#</a> <i>path</i>.<b>area</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/path/area.js "Source")

返回指定的 GeoJSON 对象投影后的平面区域的面积(通常是平方像素). `Point`, `MultiPoint`, `LineString` 和 `MultiLineString` 面积为零. 对于 `Polygon` 和 `MultiPolygon`, 这个方法会首先计算外环的面积，然后减去镂空的面积. 这个方法遵循任何 [projection](#path_projection) 的裁剪; 参考 [*projection*.clipAngle](#projection_clipAngle) 和 [*projection*.clipExtent](#projection_clipExtent). 这与 [d3.geoBounds](#geoBounds) 的平面形式等价.

<a href="#path_bounds" name="path_bounds">#</a> <i>path</i>.<b>bounds</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/path/bounds.js "Source")

返回指定的 GeoJSON 对象投影后的平面包裹框(通常是像素). 包裹框被表示为一个二维数组: \[\[*x₀*, *y₀*\], \[*x₁*, *y₁*\]\], 其中 *x₀* 是最小 *x*-坐标, *y₀* 是最小 *y*-坐标, *x₁* 是最大 *x*-坐标, and *y₁* 是最大 *y*-坐标. 这对于缩放到特定的特性非常方便. (注意, 在投影平面坐标中, 最小纬度通常是最大 y 值, 而最大纬度通常是最小 y 值). 这个方法遵循任何 [projection](#path_projection) 的裁剪; 参考 [*projection*.clipAngle](#projection_clipAngle) 和 [*projection*.clipExtent](#projection_clipExtent). 这与 [d3.geoArea](#geoArea) 的平面形式等价.

<a href="#path_centroid" name="path_centroid">#</a> <i>path</i>.<b>centroid</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/path/centroid.js "Source")

返回指定的 GeoJSON 对象投影后的平面质心(通常是像素). 这对于标记州或县的边界或显示符号非常方便. 例如 [非邻接统计地图](https://bl.ocks.org/mbostock/4055908) 可以以质心为中心进行缩放. 这个方法遵循任何 [projection](#path_projection) 的裁剪; 参考 [*projection*.clipAngle](#projection_clipAngle) 和 [*projection*.clipExtent](#projection_clipExtent). 这与 [d3.geoCentroid](#geoCentroid) 的平面形式等价.

<a href="#path_measure" name="path_measure">#</a> <i>path</i>.<b>measure</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/path/measure.js "Source")

返回指定的 GeoJSON 对象投影后的平面周长(通常是像素). `Point`, `MultiPoint`, `LineString` 和 `MultiLineString` 周长为零. 对于 `Polygon` 和 `MultiPolygon`, 这个方法会计算所有环的总长度. 这个方法遵循任何 [projection](#path_projection) 的裁剪; 参考 [*projection*.clipAngle](#projection_clipAngle) 和 [*projection*.clipExtent](#projection_clipExtent). 这与 [d3.geoLength](#geoLength) 的平面形式等价.

<a href="#path_projection" name="path_projection">#</a> <i>path</i>.<b>projection</b>([<i>projection</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/path/index.js "Source")

如果指定了 *projection* 则将当前投影设置为指定的投影. 如果没有指定 *projection* 则返回当前的投影, 默认为 `null`. `null` 投影表示一个恒等转换: 输入的集合不被计算, 而是直接在原始坐标中呈现. 这对于 [预投影](https://bl.ocks.org/mbostock/5557726) 来说省略了坐标计算因此速度更快, 或用于等矩形投影的快速绘制.

给定的 *projection* 通常是 `D3` 的内置 [地理投影](#projections); 但是任何暴露 [*projection*.stream](#projection_stream) 函数的对象都可以被使用. 参考 `D3` 的 [transforms](#transforms) 获取有关任意几何变换的更多例子.

<a href="#path_context" name="path_context">#</a> <i>path</i>.<b>context</b>([<i>context</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/path/index.js "Source")

如果指定了 *context* 则将当前渲染上下设置为指定的 *context* 并返回当前路径生成器. 如果 *context* 为 `null` 则[path generator](#_path) 将返回 SVG 路径字符串; 如果 *context* 非 `null` 则路径生成器将调用指定上下文上的方法来呈现几何图形. 上下文必须实现 [CanvasRenderingContext2D API](https://www.w3.org/TR/2dcontext/#canvasrenderingcontext2d) 的以下子集:

* *context*.beginPath()
* *context*.moveTo(*x*, *y*)
* *context*.lineTo(*x*, *y*)
* *context*.arc(*x*, *y*, *radius*, *startAngle*, *endAngle*)
* *context*.closePath()

如果没有指定 *context* 则返回当前的渲染上下文, 默认为 `null`.

<a href="#path_pointRadius" name="path_pointRadius">#</a> <i>path</i>.<b>pointRadius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/path/index.js "Source")

如果指定了 *radius* 则将当前显示的 `Point` 和 `MultiPoint` 的半径设置为指定的数值. 如果没有指定 *radius* 则返回当前半径访问器, 默认为 `4.5`. 半径通常被指定为一个常数, 也可以被指定为一个返回数值的函数, 函数形式可以为每个特征单独计算半径值, 会传递 [path generator](#_path) 的参数. 例如, 如果你的 GeoJSON 数据有额外的数属性, 你可以使用访问器函数的形式去设置不同的半径; 也可以使用 [d3.symbol](https://github.com/d3/d3-shape#symbols) 和 [projection](#geoProjection) 来获取更大的灵活性.

### Projections

投影可以将球面多边形几何映射为平面多边形几何. `D3` 提供集中标准投影的实现:

* [Azimuthal(方位投影)](#azimuthal-projections)
* [Composite(合成投影)](#composite-projections)
* [Conic(圆锥投影)](#conic-projections)
* [Cylindrical(圆柱投影)](#cylindrical-projections)

更多投影参考 [d3-geo-projection](https://github.com/d3/d3-geo-projection). 你也可以使用 [d3.geoProjection](#geoProjection) 或者 [d3.geoProjectionMutator](#geoProjectionMutator) 实现 [自定义投影](#raw-projections)

<a href="#_projection" name="_projection">#</a> <i>projection</i>(<i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

返回一个新的数组 \[*x*, *y*\](通常是像素) 来表示给定的 *point* 经过投影后的坐标. 给定的点必须是以度为单位的 \[*longitude*, *latitude*\] 形式. 如果给定的 *point* 没有定义投影位置则可能返回 `null`, 比如当点在投影的裁剪边界之外时.

<a href="#projection_invert" name="projection_invert">#</a> <i>projection</i>.<b>invert</b>(<i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

根据指定的点坐标 \[*x*, *y*\](通常是像素) 计算出该点对应的经过投影前的以度为单位的坐标 \[*longitude*, *latitude*\](逆投影). 如果指定的点没有定义投影位置时可能返回 `null`, 例如当点在投影的裁剪边界之外时.

这种方法只定义在可逆投影上.

<a href="#projection_stream" name="projection_stream">#</a> <i>projection</i>.<b>stream</b>(<i>stream</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

为指定的输出 *stream(流)* 返回 [projection stream(投影流)](#streams). 任何输入的几何图形输出之前都会被投影. 一个典型的投影包括几个几何变换: 首先将输入几何图形转换为弧度, 三轴旋转, 裁剪或沿着对向子午线剪切, 最后通过自适应重采样, 缩放和平移投影到平面上.

<a href="#projection_preclip" name="projection_preclip">#</a> <i>projection</i>.<b>preclip</b>([<i>preclip</i>])

如果指定了 *preclip* 则将投影的球形裁剪设置为指定的裁剪函数并返回投影. 如果没有指定 *preclip* 则返回当前的预裁剪函数(参考 [preclip](#preclip)).

<a href="#projection_postclip" name="projection_postclip">#</a> <i>projection</i>.<b>postclip</b>([<i>postclip</i>])

如果指定了 *postclip*, 则将投影的笛卡尔裁剪设置为指定的函数并返回投影. 如果没有指定 *postclip* 则返回当前的笛卡尔坐标裁剪(参考 [postclip](#postclip)).

<a href="#projection_clipAngle" name="projection_clipAngle">#</a> <i>projection</i>.<b>clipAngle</b>([<i>angle</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *angle* 则以度为单位设置投影的裁剪角度并返回投影. 如果 *angle* 为 `null` 则切换为 [对向子午线裁剪](https://bl.ocks.org/mbostock/3788999) 而不是 `small-circle` 裁剪. 如果没有指定 *angle* 则返回当前的裁剪角度, 默认为 `null`. `small-circle` 独立于通过 [*projection*.clipExtent](#projection_clipExtent) 定义的视窗裁剪.

可以参考 [*projection*.preclip](#projection_preclip), [d3.geoClipAntimeridian](#geoClipAntimeridian), [d3.geoClipCircle](#geoClipCircle).

<a href="#projection_clipExtent" name="projection_clipExtent">#</a> <i>projection</i>.<b>clipExtent</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *extent* 则将投影的视窗裁剪范围设置为指定的像素包裹框并返回投影. *extent* 以数组 \[\[<i>x₀</i>, <i>y₀</i>\], \[<i>x₁</i>, <i>y₁</i>\]\] 的形式指定, 其中 <i>x₀</i> 为视窗的左侧坐标, <i>y₀</i> 为顶部, <i>x₁</i> 为右侧, <i>y₁</i> 为底部坐标. 如果 *extent* 为 `null` 则不会执行视窗裁剪. 如果 *extent* 没有指定则返回当前的视窗裁剪范围, 默认为 `null`. 视窗裁剪独立于通过 [*projection*.clipAngle](#projection_clipAngle) 设置的 `small-circle` 裁剪.

参考 [*projection*.postclip](#projection_postclip), [d3.geoClipRectangle](#geoClipRectangle).

<a href="#projection_scale" name="projection_scale">#</a> <i>projection</i>.<b>scale</b>([<i>scale</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *scale* 则将投影的缩放因子设置为指定的值并返回投影. 如果没有指定 *scale* 则返回当前的缩放因子, 默认的缩放因子由具体的投影方式决定. 缩放因子与投影点之间的距离成线性关系; 但是在不同的投影中不同.

<a href="#projection_translate" name="projection_translate">#</a> <i>projection</i>.<b>translate</b>([<i>translate</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *translate* 则将投影的平移偏移量设置为指定的二元数组 [<i>t<sub>x</sub></i>, <i>t<sub>y</sub></i>] 并返回投影. 如果没有指定 *translate* 则返回当前的平移偏移, 默认为 [480, 250]. 平移偏移量决定了 [投影中心](#projection_center) 的像素坐标. 默认的平移偏移量将 ⟨0°,0°⟩ 放置在 960×500 的区域中心.

<a href="#projection_center" name="projection_center">#</a> <i>projection</i>.<b>center</b>([<i>center</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *center* 则将投影的中心设置为指定的以度为单位的 longitude 和 latitude 组成的二元数组 *center* 并返回投影. 如果没有指定 *center* 则返回当前的投影中心, 默认为 ⟨0°,0°⟩.

<a href="#projection_angle" name="projection_angle">#</a> <i>projection</i>.<b>angle</b>([<i>angle</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *angle* 则将投影后的旋转角度设置为指定的以度为单位的角度并返回投影. 如果没有指定 *angle* 则返回当前投影角度, 默认为 0°. 注意, 在渲染期间旋转(比如使用 [*context*.rotate](https://developer.mozilla.org/docs/Web/API/CanvasRenderingContext2D/rotate)) 可能比通过投影旋转更快.

<a href="#projection_rotate" name="projection_rotate">#</a> <i>projection</i>.<b>rotate</b>([<i>angles</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *rotation* 则将投影的 [三轴球形旋转](https://bl.ocks.org/mbostock/4282586) 设置为指定的 *angles*, 角度必须是包含两个或三个元素的表示旋转角度的数组 [*lambda*, *phi*, *gamma*] 用以表示 [每个球面轴](https://bl.ocks.org/mbostock/4282586). (分别对应 [偏航，俯仰和横滚](http://en.wikipedia.org/wiki/Aircraft_principal_axes)) 如果 *gamma* 没有指定则默认为 `0`. [d3.geoRotation](#geoRotation) 也同理. 如果没有指定 *rotation* 则返回当前的角度默认为 [0, 0, 0].

<a href="#projection_precision" name="projection_precision">#</a> <i>projection</i>.<b>precision</b>([<i>precision</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

如果指定了 *precision* 则将投影的 [自适应重采样]((https://bl.ocks.org/mbostock/3795544)) 阈值设置为指定的像素值并返回投影. 这个值对应 [Douglas–Peucker](http://en.wikipedia.org/wiki/Ramer–Douglas–Peucker_algorithm) 距离. 如果 *precision* 没有指定则返回投影当前的采样精度, 默认为 √0.5 ≅ 0.70710…

<a href="#projection_fitExtent" name="projection_fitExtent">#</a> <i>projection</i>.<b>fitExtent</b>(<i>extent</i>, <i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

调整投影的 [scale](#projection_scale) 和 [translate](#projection_translate) 使给定的
GeoJSON *object* 适应在 *extent* 范围中. *extent* 以 \[\[x₀, y₀\], \[x₁, y₁\]\] 的形式指定, 其中 x₀ 为包裹框的左边界, y₀ 为上边界, x₁ 为右边界, y₁ 为下边界. 返回投影.

例如, 对 [New Jersey State Plane projection](https://bl.ocks.org/mbostock/5126418) 进行缩放和平移以将 GeoJSON 对象 *nj* 位于 960×500 的包裹框内, 并设置边距为 20:

```js
var projection = d3.geoTransverseMercator()
    .rotate([74 + 30 / 60, -38 - 50 / 60])
    .fitExtent([[20, 20], [940, 480]], nj);
```

在确定了缩放和平移之后, 任何 [clip extent](#projection_clipExtent) 将会被忽略. 用来计算给定对象包裹框的 [precision](#projection_precision) 计算的有效等级为 150.

<a href="#projection_fitSize" name="projection_fitSize">#</a> <i>projection</i>.<b>fitSize</b>(<i>size</i>, <i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

[*projection*.fitExtent](#projection_fitExtent) 的便捷用法, 其中左上角坐标为 [0, 0]. 下面两个用法是等效的:

```js
projection.fitExtent([[0, 0], [width, height]], object);
projection.fitSize([width, height], object);
```

<a href="#projection_fitWidth" name="projection_fitWidth">#</a> <i>projection</i>.<b>fitWidth</b>(<i>width</i>, <i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

[*projection*.fitSize](#projection_fitSize) 的便捷用法, 其中高度是根据对象的宽高比和宽度自动计算的.

<a href="#projection_fitHeight" name="projection_fitHeight">#</a> <i>projection</i>.<b>fitHeight</b>(<i>height</i>, <i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

[*projection*.fitSize](#projection_fitSize) 的便捷用法, 其中宽度是根据对象的宽高比和高度自动计算的.

#### Azimuthal Projections

Azimuthal projections project the sphere directly onto a plane.

<a href="#geoAzimuthalEqualArea" name="geoAzimuthalEqualArea">#</a> d3.<b>geoAzimuthalEqualArea</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/azimuthalEqualArea.js "Source")
<br><a href="#geoAzimuthalEqualAreaRaw" name="geoAzimuthalEqualAreaRaw">#</a> d3.<b>geoAzimuthalEqualAreaRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/azimuthalEqualArea.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757101)

The azimuthal equal-area projection.

<a href="#geoAzimuthalEquidistant" name="geoAzimuthalEquidistant">#</a> d3.<b>geoAzimuthalEquidistant</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/azimuthalEquidistant.js "Source")
<br><a href="#geoAzimuthalEquidistantRaw" name="geoAzimuthalEquidistantRaw">#</a> d3.<b>geoAzimuthalEquidistantRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/azimuthalEquidistant.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757110)

The azimuthal equidistant projection.

<a href="#geoGnomonic" name="geoGnomonic">#</a> d3.<b>geoGnomonic</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/gnomonic.js "Source")
<br><a href="#geoGnomonicRaw" name="geoGnomonicRaw">#</a> d3.<b>geoGnomonicRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/gnomonic.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757349)

The gnomonic projection.

<a href="#geoOrthographic" name="geoOrthographic">#</a> d3.<b>geoOrthographic</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/orthographic.js "Source")
<br><a href="#geoOrthographicRaw" name="geoOrthographicRaw">#</a> d3.<b>geoOrthographicRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/orthographic.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757125)

The orthographic projection.

<a href="#geoStereographic" name="geoStereographic">#</a> d3.<b>geoStereographic</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/stereographic.js "Source")
<br><a href="#geoStereographicRaw" name="geoStereographicRaw">#</a> d3.<b>geoStereographicRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/stereographic.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757137)

The stereographic projection.

#### Composite Projections

Composite consist of several projections that are composed into a single display. The constituent projections have fixed clip, center and rotation, and thus composite projections do not support [*projection*.center](#projection_center), [*projection*.rotate](#projection_rotate), [*projection*.clipAngle](#projection_clipAngle), or [*projection*.clipExtent](#projection_clipExtent).

<a href="#geoAlbersUsa" name="geoAlbersUsa">#</a> d3.<b>geoAlbersUsa</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/albersUsa.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/albersUsa.png" width="480" height="250">](https://bl.ocks.org/mbostock/4090848)

This is a U.S.-centric composite projection of three [d3.geoConicEqualArea](#geoConicEqualArea) projections: [d3.geoAlbers](#geoAlbers) is used for the lower forty-eight states, and separate conic equal-area projections are used for Alaska and Hawaii. Note that the scale for Alaska is diminished: it is projected at 0.35× its true relative area. This diagram by Philippe Rivière illustrates how this projection uses two rectangular insets for Alaska and Hawaii:

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/albersUsa-parameters.png" width="480" height="250">](https://bl.ocks.org/Fil/7723167596af40d9159be34ffbf8d36b)

See [d3-composite-projections](http://geoexamples.com/d3-composite-projections/) for more examples.

#### Conic Projections

Conic projections project the sphere onto a cone, and then unroll the cone onto the plane. Conic projections have [two standard parallels](#conic_parallels).

<a href="#conic_parallels" name="conic_parallels">#</a> <i>conic</i>.<b>parallels</b>([<i>parallels</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conic.js "Source")

The [two standard parallels](https://en.wikipedia.org/wiki/Map_projection#Conic) that define the map layout in conic projections.

<a href="#geoAlbers" name="geoAlbers">#</a> d3.<b>geoAlbers</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/albers.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/albers.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734308)

The Albers’ equal area-conic projection. This is a U.S.-centric configuration of [d3.geoConicEqualArea](#geoConicEqualArea).

<a href="#geoConicConformal" name="geoConicConformal">#</a> d3.<b>geoConicConformal</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicConformal.js "Source")
<br><a href="#geoConicConformalRaw" name="geoConicConformalRaw">#</a> d3.<b>geoConicConformalRaw</b>(<i>phi0</i>, <i>phi1</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicConformal.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/conicConformal.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734321)

The conic conformal projection. The parallels default to [30°, 30°] resulting in flat top. See also [*conic*.parallels](#conic_parallels).

<a href="#geoConicEqualArea" name="geoConicEqualArea">#</a> d3.<b>geoConicEqualArea</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEqualArea.js "Source")
<br><a href="#geoConicEqualAreaRaw" name="geoConicEqualAreaRaw">#</a> d3.<b>geoConicEqualAreaRaw</b>(<i>phi0</i>, <i>phi1</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEqualArea.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/conicEqualArea.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734308)

The Albers’ equal-area conic projection. See also [*conic*.parallels](#conic_parallels).

<a href="#geoConicEquidistant" name="geoConicEquidistant">#</a> d3.<b>geoConicEquidistant</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEquidistant.js "Source")
<br><a href="#geoConicEquidistantRaw" name="geoConicEquidistantRaw">#</a> d3.<b>geoConicEquidistantRaw</b>(<i>phi0</i>, <i>phi1</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEquidistant.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/conicEquidistant.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734317)

The conic equidistant projection. See also [*conic*.parallels](#conic_parallels).

#### Cylindrical Projections

Cylindrical projections project the sphere onto a containing cylinder, and then unroll the cylinder onto the plane. [Pseudocylindrical projections](http://www.progonos.com/furuti/MapProj/Normal/ProjPCyl/projPCyl.html) are a generalization of cylindrical projections.

<a href="#geoEquirectangular" name="geoEquirectangular">#</a> d3.<b>geoEquirectangular</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/equirectangular.js "Source")
<br><a href="#geoEquirectangularRaw" name="geoEquirectangularRaw">#</a> d3.<b>geoEquirectangularRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/equirectangular.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757119)

The equirectangular (plate carrée) projection.

<a href="#geoMercator" name="geoMercator">#</a> d3.<b>geoMercator</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/mercator.js "Source")
<br><a href="#geoMercatorRaw" name="geoMercatorRaw">#</a> d3.<b>geoMercatorRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/mercator.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757132)

The spherical Mercator projection. Defines a default [*projection*.clipExtent](#projection_clipExtent) such that the world is projected to a square, clipped to approximately ±85° latitude.

<a href="#geoTransverseMercator" name="geoTransverseMercator">#</a> d3.<b>geoTransverseMercator</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/transverseMercator.js "Source")
<br><a href="#geoTransverseMercatorRaw" name="geoTransverseMercatorRaw">#</a> d3.<b>geoTransverseMercatorRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/transverseMercator.png" width="480" height="250">](https://bl.ocks.org/mbostock/4695821)

The transverse spherical Mercator projection. Defines a default [*projection*.clipExtent](#projection_clipExtent) such that the world is projected to a square, clipped to approximately ±85° latitude.

<a href="#geoNaturalEarth1" name="geoNaturalEarth1">#</a> d3.<b>geoNaturalEarth1</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/naturalEarth1.js "Source")
<br><a href="#geoNaturalEarth1Raw" name="geoNaturalEarth1Raw">#</a> d3.<b>geoNaturalEarth1Raw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/naturalEarth1.png" width="480" height="250">](https://bl.ocks.org/mbostock/4479477)

The [Natural Earth projection](http://www.shadedrelief.com/NE_proj/) is a pseudocylindrical projection designed by Tom Patterson. It is neither conformal nor equal-area, but appealing to the eye for small-scale maps of the whole world.

### Raw Projections

Raw projections are point transformation functions that are used to implement custom projections; they typically passed to [d3.geoProjection](#geoProjection) or [d3.geoProjectionMutator](#geoProjectionMutator). They are exposed here to facilitate the derivation of related projections. Raw projections take spherical coordinates \[*lambda*, *phi*\] in radians (not degrees!) and return a point \[*x*, *y*\], typically in the unit square centered around the origin.

<a href="#_project" name="_project">#</a> <i>project</i>(<i>lambda</i>, <i>phi</i>)

Projects the specified point [<i>lambda</i>, <i>phi</i>] in radians, returning a new point \[*x*, *y*\] in unitless coordinates.

<a href="#project_invert" name="project_invert">#</a> <i>project</i>.<b>invert</b>(<i>x</i>, <i>y</i>)

The inverse of [*project*](#_project).

<a href="#geoProjection" name="geoProjection">#</a> d3.<b>geoProjection</b>(<i>project</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

Constructs a new projection from the specified [raw projection](#_project), *project*. The *project* function takes the *longitude* and *latitude* of a given point in [radians](http://mathworld.wolfram.com/Radian.html), often referred to as *lambda* (λ) and *phi* (φ), and returns a two-element array \[*x*, *y*\] representing its unit projection. The *project* function does not need to scale or translate the point, as these are applied automatically by [*projection*.scale](#projection_scale), [*projection*.translate](#projection_translate), and [*projection*.center](#projection_center). Likewise, the *project* function does not need to perform any spherical rotation, as [*projection*.rotate](#projection_rotate) is applied prior to projection.

For example, a spherical Mercator projection can be implemented as:

```js
var mercator = d3.geoProjection(function(x, y) {
  return [x, Math.log(Math.tan(Math.PI / 4 + y / 2))];
});
```

If the *project* function exposes an *invert* method, the returned projection will also expose [*projection*.invert](#projection_invert).

<a href="#geoProjectionMutator" name="geoProjectionMutator">#</a> d3.<b>geoProjectionMutator</b>(<i>factory</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

Constructs a new projection from the specified [raw projection](#_project) *factory* and returns a *mutate* function to call whenever the raw projection changes. The *factory* must return a raw projection. The returned *mutate* function returns the wrapped projection. For example, a conic projection typically has two configurable parallels. A suitable *factory* function, such as [d3.geoConicEqualAreaRaw](#geoConicEqualAreaRaw), would have the form:

```js
// y0 and y1 represent two parallels
function conicFactory(phi0, phi1) {
  return function conicRaw(lambda, phi) {
    return […, …];
  };
}
```

Using d3.geoProjectionMutator, you can implement a standard projection that allows the parallels to be changed, reassigning the raw projection used internally by [d3.geoProjection](#geoProjection):

```js
function conicCustom() {
  var phi0 = 29.5,
      phi1 = 45.5,
      mutate = d3.geoProjectionMutator(conicFactory),
      projection = mutate(phi0, phi1);

  projection.parallels = function(_) {
    return arguments.length ? mutate(phi0 = +_[0], phi1 = +_[1]) : [phi0, phi1];
  };

  return projection;
}
```

When creating a mutable projection, the *mutate* function is typically not exposed.

### Spherical Math

<a name="geoArea" href="#geoArea">#</a> d3.<b>geoArea</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/area.js "Source")

Returns the spherical area of the specified GeoJSON *object* in [steradians](http://mathworld.wolfram.com/Steradian.html). This is the spherical equivalent of [*path*.area](#path_area).

<a name="geoBounds" href="#geoBounds">#</a> d3.<b>geoBounds</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/bounds.js "Source")

Returns the [spherical bounding box](https://www.jasondavies.com/maps/bounds/) for the specified GeoJSON *object*. The bounding box is represented by a two-dimensional array: \[\[*left*, *bottom*], \[*right*, *top*\]\], where *left* is the minimum longitude, *bottom* is the minimum latitude, *right* is maximum longitude, and *top* is the maximum latitude. All coordinates are given in degrees. (Note that in projected planar coordinates, the minimum latitude is typically the maximum *y*-value, and the maximum latitude is typically the minimum *y*-value.) This is the spherical equivalent of [*path*.bounds](#path_bounds).

<a name="geoCentroid" href="#geoCentroid">#</a> d3.<b>geoCentroid</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/centroid.js "Source")

Returns the spherical centroid of the specified GeoJSON *object*. This is the spherical equivalent of [*path*.centroid](#path_centroid).

<a name="geoDistance" href="#geoDistance">#</a> d3.<b>geoDistance</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/distance.js "Source")

Returns the great-arc distance in [radians](http://mathworld.wolfram.com/Radian.html) between the two points *a* and *b*. Each point must be specified as a two-element array \[*longitude*, *latitude*\] in degrees. This is the spherical equivalent of [*path*.measure](#path_measure) given a LineString of two points.

<a name="geoLength" href="#geoLength">#</a> d3.<b>geoLength</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/length.js "Source")

Returns the great-arc length of the specified GeoJSON *object* in [radians](http://mathworld.wolfram.com/Radian.html). For polygons, returns the perimeter of the exterior ring plus that of any interior rings. This is the spherical equivalent of [*path*.measure](#path_measure).

<a name="geoInterpolate" href="#geoInterpolate">#</a> d3.<b>geoInterpolate</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/interpolate.js "Source")

Returns an interpolator function given two points *a* and *b*. Each point must be specified as a two-element array \[*longitude*, *latitude*\] in degrees. The returned interpolator function takes a single argument *t*, where *t* is a number ranging from 0 to 1; a value of 0 returns the point *a*, while a value of 1 returns the point *b*. Intermediate values interpolate from *a* to *b* along the great arc that passes through both *a* and *b*. If *a* and *b* are antipodes, an arbitrary great arc is chosen.

<a name="geoContains" href="#geoContains">#</a> d3.<b>geoContains</b>(<i>object</i>, <i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/contains.js "Source")

Returns true if and only if the specified GeoJSON *object* contains the specified *point*, or false if the *object* does not contain the *point*. The point must be specified as a two-element array \[*longitude*, *latitude*\] in degrees. For Point and MultiPoint geometries, an exact test is used; for a Sphere, true is always returned; for other geometries, an epsilon threshold is applied.

<a name="geoRotation" href="#geoRotation">#</a> d3.<b>geoRotation</b>(<i>angles</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/rotation.js "Source")

Returns a [rotation function](#_rotation) for the given *angles*, which must be a two- or three-element array of numbers [*lambda*, *phi*, *gamma*] specifying the rotation angles in degrees about [each spherical axis](https://bl.ocks.org/mbostock/4282586). (These correspond to [yaw, pitch and roll](http://en.wikipedia.org/wiki/Aircraft_principal_axes).) If the rotation angle *gamma* is omitted, it defaults to 0. See also [*projection*.rotate](#projection_rotate).

<a name="_rotation" href="#_rotation">#</a> <i>rotation</i>(<i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/rotation.js "Source")

Returns a new array \[*longitude*, *latitude*\] in degrees representing the rotated point of the given *point*. The point must be specified as a two-element array \[*longitude*, *latitude*\] in degrees.

<a name="rotation_invert" href="#rotation_invert">#</a> <i>rotation</i>.<b>invert</b>(<i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/rotation.js "Source")

Returns a new array \[*longitude*, *latitude*\] in degrees representing the point of the given rotated *point*; the inverse of [*rotation*](#_rotation). The point must be specified as a two-element array \[*longitude*, *latitude*\] in degrees.

### Spherical Shapes

To generate a [great arc](https://en.wikipedia.org/wiki/Great-circle_distance) (a segment of a great circle), simply pass a GeoJSON LineString geometry object to a [d3.geoPath](#geoPath). D3’s projections use great-arc interpolation for intermediate points, so there’s no need for a great arc shape generator.

<a name="geoCircle" href="#geoCircle">#</a> d3.<b>geoCircle</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

Returns a new circle generator.

<a name="_circle" href="#_circle">#</a> <i>circle</i>(<i>arguments…</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

Returns a new GeoJSON geometry object of type “Polygon” approximating a circle on the surface of a sphere, with the current [center](#circle_center), [radius](#circle_radius) and [precision](#circle_precision). Any *arguments* are passed to the accessors.

<a name="circle_center" href="#circle_center">#</a> <i>circle</i>.<b>center</b>([<i>center</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

If *center* is specified, sets the circle center to the specified point \[*longitude*, *latitude*\] in degrees, and returns this circle generator. The center may also be specified as a function; this function will be invoked whenever a circle is [generated](#_circle), being passed any arguments passed to the circle generator. If *center* is not specified, returns the current center accessor, which defaults to:

```js
function center() {
  return [0, 0];
}
```

<a name="circle_radius" href="#circle_radius">#</a> <i>circle</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

If *radius* is specified, sets the circle radius to the specified angle in degrees, and returns this circle generator. The radius may also be specified as a function; this function will be invoked whenever a circle is [generated](#_circle), being passed any arguments passed to the circle generator. If *radius* is not specified, returns the current radius accessor, which defaults to:

```js
function radius() {
  return 90;
}
```

<a name="circle_precision" href="#circle_precision">#</a> <i>circle</i>.<b>precision</b>([<i>angle</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

If *precision* is specified, sets the circle precision to the specified angle in degrees, and returns this circle generator. The precision may also be specified as a function; this function will be invoked whenever a circle is [generated](#_circle), being passed any arguments passed to the circle generator. If *precision* is not specified, returns the current precision accessor, which defaults to:

```js
function precision() {
  return 6;
}
```

Small circles do not follow great arcs and thus the generated polygon is only an approximation. Specifying a smaller precision angle improves the accuracy of the approximate polygon, but also increase the cost to generate and render it.

<a name="geoGraticule" href="#geoGraticule">#</a> d3.<b>geoGraticule</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

Constructs a geometry generator for creating graticules: a uniform grid of [meridians](https://en.wikipedia.org/wiki/Meridian_\(geography\)) and [parallels](https://en.wikipedia.org/wiki/Circle_of_latitude) for showing projection distortion. The default graticule has meridians and parallels every 10° between ±80° latitude; for the polar regions, there are meridians every 90°.

<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/graticule.png" width="480" height="360">

<a name="_graticule" href="#_graticule">#</a> <i>graticule</i>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

Returns a GeoJSON MultiLineString geometry object representing all meridians and parallels for this graticule.

<a name="graticule_lines" href="#graticule_lines">#</a> <i>graticule</i>.<b>lines</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

Returns an array of GeoJSON LineString geometry objects, one for each meridian or parallel for this graticule.

<a name="graticule_outline" href="#graticule_outline">#</a> <i>graticule</i>.<b>outline</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

Returns a GeoJSON Polygon geometry object representing the outline of this graticule, i.e. along the meridians and parallels defining its extent.

<a name="graticule_extent" href="#graticule_extent">#</a> <i>graticule</i>.<b>extent</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

If *extent* is specified, sets the major and minor extents of this graticule. If *extent* is not specified, returns the current minor extent, which defaults to ⟨⟨-180°, -80° - ε⟩, ⟨180°, 80° + ε⟩⟩.

<a name="graticule_extentMajor" href="#graticule_extentMajor">#</a> <i>graticule</i>.<b>extentMajor</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

If *extent* is specified, sets the major extent of this graticule. If *extent* is not specified, returns the current major extent, which defaults to ⟨⟨-180°, -90° + ε⟩, ⟨180°, 90° - ε⟩⟩.

<a name="graticule_extentMinor" href="#graticule_extentMinor">#</a> <i>graticule</i>.<b>extentMinor</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

If *extent* is specified, sets the minor extent of this graticule. If *extent* is not specified, returns the current minor extent, which defaults to ⟨⟨-180°, -80° - ε⟩, ⟨180°, 80° + ε⟩⟩.

<a name="graticule_step" href="#graticule_step">#</a> <i>graticule</i>.<b>step</b>([<i>step</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

If *step* is specified, sets the major and minor step for this graticule. If *step* is not specified, returns the current minor step, which defaults to ⟨10°, 10°⟩.

<a name="graticule_stepMajor" href="#graticule_stepMajor">#</a> <i>graticule</i>.<b>stepMajor</b>([<i>step</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

If *step* is specified, sets the major step for this graticule. If *step* is not specified, returns the current major step, which defaults to ⟨90°, 360°⟩.

<a name="graticule_stepMinor" href="#graticule_stepMinor">#</a> <i>graticule</i>.<b>stepMinor</b>([<i>step</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

If *step* is specified, sets the minor step for this graticule. If *step* is not specified, returns the current minor step, which defaults to ⟨10°, 10°⟩.

<a name="graticule_precision" href="#graticule_precision">#</a> <i>graticule</i>.<b>precision</b>([<i>angle</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

If *precision* is specified, sets the precision for this graticule, in degrees. If *precision* is not specified, returns the current precision, which defaults to 2.5°.

<a name="geoGraticule10" href="#geoGraticule10">#</a> d3.<b>geoGraticule10</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

A convenience method for directly generating the default 10° global graticule as a GeoJSON MultiLineString geometry object. Equivalent to:

```js
function geoGraticule10() {
  return d3.geoGraticule()();
}
```

### Streams

D3 transforms geometry using a sequence of function calls, rather than materializing intermediate representations, to minimize overhead. Streams must implement several methods to receive input geometry. Streams are inherently stateful; the meaning of a [point](#point) depends on whether the point is inside of a [line](#lineStart), and likewise a line is distinguished from a ring by a [polygon](#polygonStart). Despite the name “stream”, these method calls are currently synchronous.

<a href="#geoStream" name="geoStream">#</a> d3.<b>geoStream</b>(<i>object</i>, <i>stream</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/stream.js "Source")

Streams the specified [GeoJSON](http://geojson.org) *object* to the specified [projection *stream*](#projection-streams). While both features and geometry objects are supported as input, the stream interface only describes the geometry, and thus additional feature properties are not visible to streams.

<a name="stream_point" href="#stream_point">#</a> <i>stream</i>.<b>point</b>(<i>x</i>, <i>y</i>[, <i>z</i>])

Indicates a point with the specified coordinates *x* and *y* (and optionally *z*). The coordinate system is unspecified and implementation-dependent; for example, [projection streams](https://github.com/d3/d3-geo-projection) require spherical coordinates in degrees as input. Outside the context of a polygon or line, a point indicates a point geometry object ([Point](http://www.geojson.org/geojson-spec.html#point) or [MultiPoint](http://www.geojson.org/geojson-spec.html#multipoint)). Within a line or polygon ring, the point indicates a control point.

<a name="stream_lineStart" href="#stream_lineStart">#</a> <i>stream</i>.<b>lineStart</b>()

Indicates the start of a line or ring. Within a polygon, indicates the start of a ring. The first ring of a polygon is the exterior ring, and is typically clockwise. Any subsequent rings indicate holes in the polygon, and are typically counterclockwise.

<a name="stream_lineEnd" href="#stream_lineEnd">#</a> <i>stream</i>.<b>lineEnd</b>()

Indicates the end of a line or ring. Within a polygon, indicates the end of a ring. Unlike GeoJSON, the redundant closing coordinate of a ring is *not* indicated via [point](#point), and instead is implied via lineEnd within a polygon. Thus, the given polygon input:

```json
{
  "type": "Polygon",
  "coordinates": [
    [[0, 0], [0, 1], [1, 1], [1, 0], [0, 0]]
  ]
}
```

Will produce the following series of method calls on the stream:

```js
stream.polygonStart();
stream.lineStart();
stream.point(0, 0);
stream.point(0, 1);
stream.point(1, 1);
stream.point(1, 0);
stream.lineEnd();
stream.polygonEnd();
```

<a name="stream_polygonStart" href="#stream_polygonStart">#</a> <i>stream</i>.<b>polygonStart</b>()

Indicates the start of a polygon. The first line of a polygon indicates the exterior ring, and any subsequent lines indicate interior holes.

<a name="stream_polygonEnd" href="#stream_polygonEnd">#</a> <i>stream</i>.<b>polygonEnd</b>()

Indicates the end of a polygon.

<a name="stream_sphere" href="#stream_sphere">#</a> <i>stream</i>.<b>sphere</b>()

Indicates the sphere (the globe; the unit sphere centered at ⟨0,0,0⟩).

### Transforms

Transforms are a generalization of projections. Transform implement [*projection*.stream](#projection_stream) and can be passed to [*path*.projection](#path_projection). However, they only implement a subset of the other projection methods, and represent arbitrary geometric transformations rather than projections from spherical to planar coordinates.

<a href="#geoTransform" name="geoTransform">#</a> d3.<b>geoTransform</b>(<i>methods</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/transform.js "Source")

Defines an arbitrary transform using the methods defined on the specified *methods* object. Any undefined methods will use pass-through methods that propagate inputs to the output stream. For example, to reflect the *y*-dimension (see also [*identity*.reflectY](#identity_reflectY)):

```js
var reflectY = d3.geoTransform({
  point: function(x, y) {
    this.stream.point(x, -y);
  }
});
```

Or to define an affine matrix transformation:

```js
function matrix(a, b, c, d, tx, ty) {
  return d3.geoTransform({
    point: function(x, y) {
      this.stream.point(a * x + b * y + tx, c * x + d * y + ty);
    }
  });
}
```

<a href="#geoIdentity" name="geoIdentity">#</a> d3.<b>geoIdentity</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/identity.js "Source")

The identity transform can be used to scale, translate and clip planar geometry. It implements [*projection*.scale](#projection_scale), [*projection*.translate](#projection_translate), [*projection*.fitExtent](#projection_fitExtent), [*projection*.fitSize](#projection_fitSize), [*projection*.fitWidth](#projection_fitWidth), [*projection*.fitHeight](#projection_fitHeight) and [*projection*.clipExtent](#projection_clipExtent).

<a href="#identity_reflectX" name="identity_reflectX">#</a> <i>identity</i>.<b>reflectX</b>([<i>reflect</i>])

If *reflect* is specified, sets whether or not the *x*-dimension is reflected (negated) in the output. If *reflect* is not specified, returns true if *x*-reflection is enabled, which defaults to false.

<a href="#identity_reflectY" name="identity_reflectY">#</a> <i>identity</i>.<b>reflectY</b>([<i>reflect</i>])

If *reflect* is specified, sets whether or not the *y*-dimension is reflected (negated) in the output. If *reflect* is not specified, returns true if *y*-reflection is enabled, which defaults to false. This is especially useful for transforming from standard [spatial reference systems](https://en.wikipedia.org/wiki/Spatial_reference_system), which treat positive *y* as pointing up, to display coordinate systems such as Canvas and SVG, which treat positive *y* as pointing down.

### Clipping

Projections perform cutting or clipping of geometries in two stages.

<a name="preclip" href="#preclip">#</a> <i>preclip</i>(<i>stream</i>)

Pre-clipping occurs in geographic coordinates. Cutting along the antimeridian line, or clipping along a small circle are the most common strategies.

See [*projection*.preclip](#projection_preclip).

<a name="postclip" href="#postclip">#</a> <i>postclip</i>(<i>stream</i>)

Post-clipping occurs on the plane, when a projection is bounded to a certain extent such as a rectangle.

See [*projection*.postclip](#projection_postclip).

Clipping functions are implemented as transformations of a [projection stream](#streams). Pre-clipping operates on spherical coordinates, in radians. Post-clipping operates on planar coordinates, in pixels.

<a name="geoClipAntimeridian" href="#geoClipAntimeridian">#</a> d3.<b>geoClipAntimeridian</b>

A clipping function which transforms a stream such that geometries (lines or polygons) that cross the antimeridian line are cut in two, one on each side. Typically used for pre-clipping.

<a name="geoClipCircle" href="#geoClipCircle">#</a> d3.<b>geoClipCircle</b>(<i>angle</i>)

Generates a clipping function which transforms a stream such that geometries are bounded by a small circle of radius *angle* around the projection’s [center](#projection_center). Typically used for pre-clipping.

<a name="geoClipRectangle" href="#geoClipRectangle">#</a> d3.<b>geoClipRectangle</b>(<i>x0</i>, <i>y0</i>, <i>x1</i>, <i>y1</i>)

Generates a clipping function which transforms a stream such that geometries are bounded by a rectangle of coordinates [[<i>x0</i>, <i>y0</i>], [<i>x1</i>, <i>y1</i>]]. Typically used for post-clipping.
