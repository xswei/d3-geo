# d3-geo

地图投影有时作为点的转换实现。比如球形墨卡托:

```js
function mercator(x, y) {
  return [x, Math.log(Math.tan(Math.PI / 4 + y / 2))];
}
```

如果你的几何包含连续无限的点集时，这是一个合理的 *数学* 方法。然而，计算机并没有无限的内存，所以我们必须使用离散几何，比如多边形和折线!

离散几何使得从球面投影到平面更加困难。球面多边形的边缘是[geodesics(测地线)](https://en.wikipedia.org/wiki/Geodesic)[great arc](https://en.wikipedia.org/wiki/Great-circle_distance)()的部分)，而不是直线。测地线投射到平面上，除了 [gnomonic](#geoGnomonic) 之外所有的地图投影都是曲线，因此精确的投影需要沿每条弧插值。`D3` 使用 [adaptive sampling(自适应采样)](https://bl.ocks.org/mbostock/3795544)灵感来 [line simplification method(线简化方法)](https://bost.ocks.org/mike/simplify/)，以平衡精度和性能。

多边形和折线的投影还必须处理球面与平面的拓扑差异。一些投影需要切割 [对向子午线](https://bl.ocks.org/mbostock/3788999) 的几何图形，而另一些投影则需要将几何图形 [裁剪为一[great arc](https://en.wikipedia.org/wiki/Great-circle_distance)()](https://bl.ocks.org/mbostock/3021474)。

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

方位投影把球体直接投射到平面上.

<a href="#geoAzimuthalEqualArea" name="geoAzimuthalEqualArea">#</a> d3.<b>geoAzimuthalEqualArea</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/azimuthalEqualArea.js "Source")
<br><a href="#geoAzimuthalEqualAreaRaw" name="geoAzimuthalEqualAreaRaw">#</a> d3.<b>geoAzimuthalEqualAreaRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/azimuthalEqualArea.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757101)

等面积方位投影.

<a href="#geoAzimuthalEquidistant" name="geoAzimuthalEquidistant">#</a> d3.<b>geoAzimuthalEquidistant</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/azimuthalEquidistant.js "Source")
<br><a href="#geoAzimuthalEquidistantRaw" name="geoAzimuthalEquidistantRaw">#</a> d3.<b>geoAzimuthalEquidistantRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/azimuthalEquidistant.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757110)

等距方位投影.

<a href="#geoGnomonic" name="geoGnomonic">#</a> d3.<b>geoGnomonic</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/gnomonic.js "Source")
<br><a href="#geoGnomonicRaw" name="geoGnomonicRaw">#</a> d3.<b>geoGnomonicRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/gnomonic.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757349)

诺蒙尼日投影.

<a href="#geoOrthographic" name="geoOrthographic">#</a> d3.<b>geoOrthographic</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/orthographic.js "Source")
<br><a href="#geoOrthographicRaw" name="geoOrthographicRaw">#</a> d3.<b>geoOrthographicRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/orthographic.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757125)

正交投影.

<a href="#geoStereographic" name="geoStereographic">#</a> d3.<b>geoStereographic</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/stereographic.js "Source")
<br><a href="#geoStereographicRaw" name="geoStereographicRaw">#</a> d3.<b>geoStereographicRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/stereographic.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757137)

赤平投影.

#### Composite Projections

复合投影由多个投影组成. 复合投影具有固定的裁剪, 中心以及旋转, 并且复合投影不支持 [*projection*.center](#projection_center), [*projection*.rotate](#projection_rotate), [*projection*.clipAngle](#projection_clipAngle), 和 [*projection*.clipExtent](#projection_clipExtent).

<a href="#geoAlbersUsa" name="geoAlbersUsa">#</a> d3.<b>geoAlbersUsa</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/albersUsa.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/albersUsa.png" width="480" height="250">](https://bl.ocks.org/mbostock/4090848)

这是以美国为中心的三个 [d3.geoConicEqualArea](#geoConicEqualArea) 组成的投影: [d3.geoAlbers](#geoAlbers) 用来投影较低的 48 个州, 圆锥等面积投影用来投影 `Alaska` 和 `Hawaii`. 请注意 `Alaska` 被缩小了: 相对被缩小到原来的 0.35 倍. `Philippe Rivière` 的这张图用来展示这个投影如何处理 `Alaska` 和 `Hawaii`：

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/albersUsa-parameters.png" width="480" height="250">](https://bl.ocks.org/Fil/7723167596af40d9159be34ffbf8d36b)

See [d3-composite-projections](http://geoexamples.com/d3-composite-projections/) for more examples.

#### Conic Projections

圆锥投影将球体投射到圆锥上, 然后将圆锥展开到平面上. 圆锥投影有两条 [标准平行线](#conic_parallels).

<a href="#conic_parallels" name="conic_parallels">#</a> <i>conic</i>.<b>parallels</b>([<i>parallels</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conic.js "Source")

在圆锥投影中定义地图布局的 [两个标准平行线](https://en.wikipedia.org/wiki/Map_projection#Conic).

<a href="#geoAlbers" name="geoAlbers">#</a> d3.<b>geoAlbers</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/albers.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/albers.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734308)

`Albers` 的等面积圆锥投影. 这是以美国为中心的 [d3.geoConicEqualArea](#geoConicEqualArea).

<a href="#geoConicConformal" name="geoConicConformal">#</a> d3.<b>geoConicConformal</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicConformal.js "Source")
<br><a href="#geoConicConformalRaw" name="geoConicConformalRaw">#</a> d3.<b>geoConicConformalRaw</b>(<i>phi0</i>, <i>phi1</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicConformal.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/conicConformal.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734321)

圆锥保角投影. 平行线默认为 [30°, 30°] 导致平顶. 参考 [*conic*.parallels](#conic_parallels).

<a href="#geoConicEqualArea" name="geoConicEqualArea">#</a> d3.<b>geoConicEqualArea</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEqualArea.js "Source")
<br><a href="#geoConicEqualAreaRaw" name="geoConicEqualAreaRaw">#</a> d3.<b>geoConicEqualAreaRaw</b>(<i>phi0</i>, <i>phi1</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEqualArea.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/conicEqualArea.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734308)

`Albers` 等面积圆锥投影. 参考 [*conic*.parallels](#conic_parallels).

<a href="#geoConicEquidistant" name="geoConicEquidistant">#</a> d3.<b>geoConicEquidistant</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEquidistant.js "Source")
<br><a href="#geoConicEquidistantRaw" name="geoConicEquidistantRaw">#</a> d3.<b>geoConicEquidistantRaw</b>(<i>phi0</i>, <i>phi1</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/conicEquidistant.js "Source")

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/conicEquidistant.png" width="480" height="250">](https://bl.ocks.org/mbostock/3734317)

圆锥等距投影. 参考 [*conic*.parallels](#conic_parallels).

#### Cylindrical Projections

圆柱投影将球体投射到一个圆柱体上, 然后将圆柱体展开到平面上. [Pseudocylindrical projections(伪圆柱投影)](http://www.progonos.com/furuti/MapProj/Normal/ProjPCyl/projPCyl.html) 是圆柱投影的延伸.

<a href="#geoEquirectangular" name="geoEquirectangular">#</a> d3.<b>geoEquirectangular</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/equirectangular.js "Source")
<br><a href="#geoEquirectangularRaw" name="geoEquirectangularRaw">#</a> d3.<b>geoEquirectangularRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/equirectangular.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757119)

等矩形(圆柱)投影.

<a href="#geoMercator" name="geoMercator">#</a> d3.<b>geoMercator</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/mercator.js "Source")
<br><a href="#geoMercatorRaw" name="geoMercatorRaw">#</a> d3.<b>geoMercatorRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/mercator.png" width="480" height="250">](https://bl.ocks.org/mbostock/3757132)

球面墨卡托投影. 定义了默认的 [*projection*.clipExtent](#projection_clipExtent): 世界被投射到一个正方形上, 裁剪到大约 `±85°` 纬度.

<a href="#geoTransverseMercator" name="geoTransverseMercator">#</a> d3.<b>geoTransverseMercator</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/transverseMercator.js "Source")
<br><a href="#geoTransverseMercatorRaw" name="geoTransverseMercatorRaw">#</a> d3.<b>geoTransverseMercatorRaw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/transverseMercator.png" width="480" height="250">](https://bl.ocks.org/mbostock/4695821)

横向球面墨卡托投影, 定义了默认的 [*projection*.clipExtent](#projection_clipExtent): 世界被投射到一个正方形上, 裁剪到大约 `±85°` 纬度.

<a href="#geoNaturalEarth1" name="geoNaturalEarth1">#</a> d3.<b>geoNaturalEarth1</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/projection/naturalEarth1.js "Source")
<br><a href="#geoNaturalEarth1Raw" name="geoNaturalEarth1Raw">#</a> d3.<b>geoNaturalEarth1Raw</b>

[<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/naturalEarth1.png" width="480" height="250">](https://bl.ocks.org/mbostock/4479477)

[Natural Earth projection(自然地球投影)](http://www.shadedrelief.com/NE_proj/) 是由 `Tom Patterson` 设计的伪圆柱投影. 它既不是等角也不是等角的, 但是看起来很吸引人, 像是一个缩小的世界.

### Raw Projections

原始投影是一个用来实现自定义点转换功能的函数; 它们通常被传递给 [d3.geoProjection](#geoProjection) 或 [d3.geoProjectionMutator](#geoProjectionMutator). 这个接口的暴露方便了相关投影的推导. 原始投影接收球面坐标  \[*lambda*, *phi*\] (弧度, 不是角度) 返回点坐标 \[*x*, *y*\], 通常是在以原点为中心的单位正方形中.

<a href="#_project" name="_project">#</a> <i>project</i>(<i>lambda</i>, <i>phi</i>)

计算给定以弧度为单位的点 [<i>lambda</i>, <i>phi</i>] 对应的投影后的无单位坐标 \[*x*, *y*\].

<a href="#project_invert" name="project_invert">#</a> <i>project</i>.<b>invert</b>(<i>x</i>, <i>y</i>)

[*project*](#_project) 的逆计算.

<a href="#geoProjection" name="geoProjection">#</a> d3.<b>geoProjection</b>(<i>project</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

根据指定的 [raw projection](#_project) 构造一个新的投影 *project*. *project* 函数以 [弧度](http://mathworld.wolfram.com/Radian.html) 为单位接收 *longitude* and *latitude*, 通常被称为 *lambda* (λ) and *phi* (φ), 返回二元数组  \[*x*, *y*\] 表示其单位投影. *project* 函数不需要缩放或平移点, 因为它们是通过 [*projection*.scale](#projection_scale), [*projection*.translate](#projection_translate), 和 [*projection*.center](#projection_center) 投影自动应用的. 同样的, 也不需要像投影那样执行任何球面旋转, 比如在投影之前使用 [*projection*.rotate](#projection_rotate) 进行球面旋转.

例如, 球面墨卡托投影可以实现为:

```js
var mercator = d3.geoProjection(function(x, y) {
  return [x, Math.log(Math.tan(Math.PI / 4 + y / 2))];
});
```

如果 *project* 函数暴露 *invert* 方法, 则返回的投影会暴露 [*projection*.invert](#projection_invert).

<a href="#geoProjectionMutator" name="geoProjectionMutator">#</a> d3.<b>geoProjectionMutator</b>(<i>factory</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/projection/index.js "Source")

根据指定的 [raw projection](#_project) *factory* 构造一个新的投影并返回一个当原始投影变化时调用的 *mutate* 函数. *factory* 必须返回一个原始投影. 返回的 *mutate* 函数返回一个投影函数. 例如, 圆锥投影通常有两个可配的平行线. 合适的 *factory* 比如 [d3.geoConicEqualAreaRaw](#geoConicEqualAreaRaw) 应该为:

```js
// y0 and y1 represent two parallels
function conicFactory(phi0, phi1) {
  return function conicRaw(lambda, phi) {
    return […, …];
  };
}
```

适用 `d3.geoProjectionMutator` 时, 你可以实现一个允许平行线改变的标准投影, 将内部使用的原始投影重新分配给 [d3.geoProjection](#geoProjection):

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

在创建可变投影时, 通常不会暴露 `mutate` 函数.

### Spherical Math

<a name="geoArea" href="#geoArea">#</a> d3.<b>geoArea</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/area.js "Source")

返回指定的 `GeoJSON` 对象的球形区域的[立体角](http://mathworld.wolfram.com/Steradian.html). 等价于 [*path*.area](#path_area) 的球面形式.

<a name="geoBounds" href="#geoBounds">#</a> d3.<b>geoBounds</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/bounds.js "Source")

返回指定 `GeoJSON` 对象的球面包围框. 包裹框以二维数组形式表示: \[\[*left*, *bottom*], \[*right*, *top*\]\], 其中  *left* 为最小经度, *bottom* 为最小纬度,  *right* 为最大经度, *top* 为最大纬度. 所有坐标都以度为单位. (注意, 在投影平面坐标中, 最小纬度通常是最大 `y` 值, 而最大纬度通常是最小 `y` 值) 等价于 [*path*.bounds](#path_bounds) 的球面形式.

<a name="geoCentroid" href="#geoCentroid">#</a> d3.<b>geoCentroid</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/centroid.js "Source")

返回指定 `GeoJSON` 对象的球形质心. 等价于 [*path*.centroid](#path_centroid) 的球面形式.

<a name="geoDistance" href="#geoDistance">#</a> d3.<b>geoDistance</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/distance.js "Source")

以[弧度](http://mathworld.wolfram.com/Radian.html)为单位返回两点 *a* 和 *b* 之间的弧长. 每个点都必须被指定为以度为单位的二元数组的形式: \[*longitude*, *latitude*\]. 等价于两点之间计算 [*path*.measure](#path_measure) 的球面形式.

<a name="geoLength" href="#geoLength">#</a> d3.<b>geoLength</b>(<i>object</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/length.js "Source")

以[弧度](http://mathworld.wolfram.com/Radian.html)为单位返回指定 `GeoJSON` 对象的弧长. 如果是多边形则返回外环的周长加上任何内环的周长. 等价于 [*path*.measure](#path_measure) 的球面形式.

<a name="geoInterpolate" href="#geoInterpolate">#</a> d3.<b>geoInterpolate</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/interpolate.js "Source")

在给定的两个点 *a* 和 *b* 之间返回一个插值函数. 每个点都必须被指定为以度为单位的二元数组的形式: \[*longitude*, *latitude*\]. 返回的插值函数接收单个参数 *t*, 其中 *t* 为从 `0` 到 `1` 的数值. *t* 为 `0` 时插值函数返回点 *a*, *t* 为 `1` 时插值函数返回点 *b*. 中间值沿着经过 *a* 和 *b* [great arc](https://en.wikipedia.org/wiki/Great-circle_distance)()弧从 *a* 插值到 *b*. 如果 *a* 和 *b* 正相对应, 则选择任意[great arc](https://en.wikipedia.org/wiki/Great-circle_distance)()弧.

<a name="geoContains" href="#geoContains">#</a> d3.<b>geoContains</b>(<i>object</i>, <i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/contains.js "Source")

判断一个点是否被包含在指定的 `GeoJSON` 对象中, 如果是则返回 `true`, 否则返回 `false`. 指定的点必须被指定为以度为单位的二元数组的形式: \[*longitude*, *latitude*\]. 如果 `GeoJSON` 为点或者多个点则启用精确测试, 如果是球则总是返回 `true`; 对于其他的几何类型, 则使用一个很小的阈值来判断是否被包含.

<a name="geoRotation" href="#geoRotation">#</a> d3.<b>geoRotation</b>(<i>angles</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/rotation.js "Source")

根据指定的 *angles* 返回一个[旋转函数](#_rotation), 其中 *angles* 必须是一个二元或三元数值数组 [*lambda*, *phi*, *gamma*] 用来表示在[每个球面轴](https://bl.ocks.org/mbostock/4282586) 上的旋转角度. (分别对应 [偏航，俯仰和横滚](http://en.wikipedia.org/wiki/Aircraft_principal_axes)) 如果省略了  *gamma* 则默认为 `0`. 参考 [*projection*.rotate](#projection_rotate).

<a name="_rotation" href="#_rotation">#</a> <i>rotation</i>(<i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/rotation.js "Source")

以角度为单位返回将给定的点经过旋转后新的点: \[*longitude*, *latitude*\]. 指定的点必须以度为单位的二元数组形式: \[*longitude*, *latitude*\].

<a name="rotation_invert" href="#rotation_invert">#</a> <i>rotation</i>.<b>invert</b>(<i>point</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/rotation.js "Source")

根据指定的点返回该点经过旋转之前的点的坐标:  \[*longitude*, *latitude*\]. 是 [*rotation*](#_rotation)
 的逆运算, 同理指定的点也必须是以度为单位的二元数组形式.

### Spherical Shapes

在生成 [great arc](https://en.wikipedia.org/wiki/Great-circle_distance)(大圆弧的一部分)时, 只需要给 [d3.geoPath](#geoPath) 传入 `GeoJSON LineString` 对象即可. `D3` 的插值器使用大圆插值进行内部点插值, 因此不需要大圆弧生成器. 

<a name="geoCircle" href="#geoCircle">#</a> d3.<b>geoCircle</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

返回一个新的圆生成器。

<a name="_circle" href="#_circle">#</a> <i>circle</i>(<i>arguments…</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

使用当前的 [center](#circle_center), [radius](#circle_radius) 和 [precision](#circle_precision) 返回一个新的 `GeoJSON` 几何对象, 类型为 `Polygon`, 近似于球体表面上的一个圆. 任何参数都传递给访问器.

<a name="circle_center" href="#circle_center">#</a> <i>circle</i>.<b>center</b>([<i>center</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

如果指定了 *center* 则将当前圆生成器的中心设置为指定的以度为单位的点: \[*longitude*, *latitude*\] 并返回圆生成器. 中心点可以指定为一个函数; 无论何时生成一个圆, 都会调用此函数, 并将传递给圆生成器的任何参数. 如果 *center* 没有被指定则返回当前的中心访问器, 默认为:

```js
function center() {
  return [0, 0];
}
```

<a name="circle_radius" href="#circle_radius">#</a> <i>circle</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

如果指定了 *radius* 则将当前圆生成器的半径设置为指定的角度(以角度为单位)并返回圆生成器. *radius* 可以指定为一个函数; 无论何时生成一个圆, 都会调用此函数, 并将传递给圆生成器的任何参数. 如果 *radius* 没有被指定则返回当前的半径访问器, 默认为:

```js
function radius() {
  return 90;
}
```

<a name="circle_precision" href="#circle_precision">#</a> <i>circle</i>.<b>precision</b>([<i>angle</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/circle.js "Source")

如果指定的 *precision* 则将圆生成器的精度设置为指定的度并返回圆生成器. 精度可以指定为函数形式; 无论何时生成一个圆, 都会调用此函数, 并将传递给圆生成器的任何参数. 如果没有指定 *precision* 则返回当前的精度访问器, 默认为:

```js
function precision() {
  return 6;
}
```

小圆不遵循大圆弧，因此生成的多边形只是一个近似, 指定一个较小的精确角度可以提高多边形的精度, 但也增加了生成和渲染它的成本.

<a name="geoGraticule" href="#geoGraticule">#</a> d3.<b>geoGraticule</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

构造用于创建经纬网的几何生成器: 一种由[经线](https://en.wikipedia.org/wiki/Meridian_\(geography\))和[纬线](https://en.wikipedia.org/wiki/Circle_of_latitude)组成的均匀网格, 用于显示投影变形. 默认的经纬网的经纬线在 ±80° 之间每隔 10° 设置一条线. 对于极地地区则是 90°.

<img src="https://raw.githubusercontent.com/d3/d3-geo/master/img/graticule.png" width="480" height="360">

<a name="_graticule" href="#_graticule">#</a> <i>graticule</i>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

返回一个 `GeoJSON MultiLineString` 几何对象, 该对象表示此分划线的所有经纬线.

<a name="graticule_lines" href="#graticule_lines">#</a> <i>graticule</i>.<b>lines</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

返回一个 `GeoJSON LineString` 几何对象数组, 每个经线对应一个对象.

<a name="graticule_outline" href="#graticule_outline">#</a> <i>graticule</i>.<b>outline</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

返回一个 `GeoJSON` 多边形几何对象, 表示这个经纬网的轮廓, 也就是经纬网的边界.

<a name="graticule_extent" href="#graticule_extent">#</a> <i>graticule</i>.<b>extent</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

如果指定了 *extent* 则设置经纬网的主要和次要边界. 如果没有指定 *extent* 则返回当前次要边界, 默认为 ⟨⟨-180°, -80° - ε⟩, ⟨180°, 80° + ε⟩⟩.

<a name="graticule_extentMajor" href="#graticule_extentMajor">#</a> <i>graticule</i>.<b>extentMajor</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

如果指定了 *extent* 则设置经纬网的主要边界. 如果没有指定 *extent* 则返回当前经纬网主要边界, 默认为 ⟨⟨-180°, -90° + ε⟩, ⟨180°, 90° - ε⟩⟩.

<a name="graticule_extentMinor" href="#graticule_extentMinor">#</a> <i>graticule</i>.<b>extentMinor</b>([<i>extent</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

如果指定了 *extent* 则设置经纬网的次要边界. 如果没有指定 *extent* 则返回当前次要边界, 默认为 ⟨⟨-180°, -80° - ε⟩, ⟨180°, 80° + ε⟩⟩.

<a name="graticule_step" href="#graticule_step">#</a> <i>graticule</i>.<b>step</b>([<i>step</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

如果指定了 *step* 则设置经纬网的主要和次要步长. 如果没有指定 *step* 则返回经纬网的次要步长, 默认为 ⟨10°, 10°⟩.

<a name="graticule_stepMajor" href="#graticule_stepMajor">#</a> <i>graticule</i>.<b>stepMajor</b>([<i>step</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

如果指定了 *step* 则设置经纬网的主要步长. 如果没有指定 *step* 则返回经纬网的主要步长, 默认为 ⟨90°, 360°⟩.

<a name="graticule_stepMinor" href="#graticule_stepMinor">#</a> <i>graticule</i>.<b>stepMinor</b>([<i>step</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

如果指定了 *step* 则设置经纬网的次要步长. 如果没有指定 *step* 则返回经纬网的主要步长, 默认为 ⟨10°, 10°⟩.

<a name="graticule_precision" href="#graticule_precision">#</a> <i>graticule</i>.<b>precision</b>([<i>angle</i>]) [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

如果指定了 *precision* 则以度为单位设置经纬网的精度. 如果没有指定 *precision* 则返回当前精度, 默认为 2.5°.

<a name="geoGraticule10" href="#geoGraticule10">#</a> d3.<b>geoGraticule10</b>() [<>](https://github.com/d3/d3-geo/blob/master/src/graticule.js "Source")

一个方便的方法, 直接生成默认的 10° 的全球经纬网, 以 `GeoJSON MultiLineString` 对象形式返回。等价于:

```js
function geoGraticule10() {
  return d3.geoGraticule()();
}
```

### Streams

`D3` 使用一系列函数调用进行几何转换, 过渡过程中的中间值不被表现出来, 以最小化开销. 流必须实现多个方法来接收输入几何图形. 流本质上是有状态的; [point](#point) 的定义取决于该点是否在 [line](#lineStart) 内, 同样地线与环的区别是由 [polygon](#polygonStart) 决定的. 尽管被命名为 “流” 但是这些方法是被同步调用的.

<a href="#geoStream" name="geoStream">#</a> d3.<b>geoStream</b>(<i>object</i>, <i>stream</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/stream.js "Source")

将指定的 `GeoJSON` 对象流到指定的投影流. 虽然特性和几何对象都支持作为输入, 但是流接口只描述几何形状, 因此其他特性属性对流来说是不可见的.

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

变换是投影的推广. 变换实现了 [*projection*.stream](#projection_stream) 并且可以被传递给 [*path*.projection](#path_projection). 然而, 变换仅仅实现的是一个投影方法子集, 并且表示任意的几何变换, 而不仅仅是球面到平面的投影.

Transforms are a generalization of projections. Transform implement [*projection*.stream](#projection_stream) and can be passed to [*path*.projection](#path_projection). However, they only implement a subset of the other projection methods, and represent arbitrary geometric transformations rather than projections from spherical to planar coordinates.

<a href="#geoTransform" name="geoTransform">#</a> d3.<b>geoTransform</b>(<i>methods</i>) [<>](https://github.com/d3/d3-geo/blob/master/src/transform.js "Source")

使用指定的方法对象定义一个任意的转换. 任何未定义的方法都将使用直通的方式将输入到输出流. 例如, 反转 `y` 维度(参考 [*identity*.reflectY](#identity_reflectY)):

```js
var reflectY = d3.geoTransform({
  point: function(x, y) {
    this.stream.point(x, -y);
  }
});
```

或者定义仿射矩阵变换:

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

单位变换可用于缩放、平移和剪切平面几何图形. 它实现了 [*projection*.scale](#projection_scale), [*projection*.translate](#projection_translate), [*projection*.fitExtent](#projection_fitExtent), [*projection*.fitSize](#projection_fitSize), [*projection*.fitWidth](#projection_fitWidth), [*projection*.fitHeight](#projection_fitHeight) 和 [*projection*.clipExtent](#projection_clipExtent).

<a href="#identity_reflectX" name="identity_reflectX">#</a> <i>identity</i>.<b>reflectX</b>([<i>reflect</i>])

如果指定了 *reflect* , 则设置是否在输出中反转 `x` 维度. 如果没有指定 *reflect* 则返回当前是否反转 `x`, 默认为 `false`.

<a href="#identity_reflectY" name="identity_reflectY">#</a> <i>identity</i>.<b>reflectY</b>([<i>reflect</i>])

如果指定了 *reflect* 则设置是否在输出中反转 `x` 维度. 如果没有指定 *reflect* 则返回当前是否反转 `x`, 默认为 `false`. 这在从标准的 [spatial reference systems](https://en.wikipedia.org/wiki/Spatial_reference_system) 转换时非常有用, 在此类系统中, *y* 方向是向上的, 而显示的时候比如 `Canvas` 和 `SVG` 则 *y* 是向下的.

### Clipping

投影分两个阶段完成几何图形的切割或剪裁.

<a name="preclip" href="#preclip">#</a> <i>preclip</i>(<i>stream</i>)

预剪切发生在地理坐标中. 沿着180度经线切割. 或者沿着一个纬线切割是最常见的策略.

参考 [*projection*.preclip](#projection_preclip).

<a name="postclip" href="#postclip">#</a> <i>postclip</i>(<i>stream</i>)

后期剪切发生在平面上, 当一个投影有一定的边界时, 例如一个将投影限制在一个矩形框中.

参考 [*projection*.postclip](#projection_postclip).

裁剪函数以 [projection stream](#streams) 转换的形式实现. 预剪切操作在球面坐标上，以弧度为单位. 后剪接操作在平面坐标上，以像素为单位.

<a name="geoClipAntimeridian" href="#geoClipAntimeridian">#</a> d3.<b>geoClipAntimeridian</b>

一个剪切函数, 使跨越180度经线的几何图形流(线或多边形)被切成两半，每一边各一个. 通常用于预剪切.

<a name="geoClipCircle" href="#geoClipCircle">#</a> d3.<b>geoClipCircle</b>(<i>angle</i>)

生成一个剪切函数, 该函数转换流使得几何图形以围绕投影 [center](#projection_center) 的小半径角为界. 通常用于预剪切.

<a name="geoClipRectangle" href="#geoClipRectangle">#</a> d3.<b>geoClipRectangle</b>(<i>x0</i>, <i>y0</i>, <i>x1</i>, <i>y1</i>)

生成一个剪切函数, 该函数转换流使几何图形以矩形坐标框 [[x0, y0]， [x1, y1]] 为边界. 通常用于后期剪切.
