---
title: 数据连接与Mapbox Boundaires
description: Join data with v2 of the Mapbox Boundaries tileset.
level: 3
topics:
- data
language:
- JavaScript
thumbnail: dataJoinsWithEnterpriseBoundaries
prereq: Familiarity with front-end development and access to Mapbox Boundaries.
prependJs:
  - "import * as constants from '../../constants';"
  - "import Note from '@mapbox/dr-ui/note';"
  - "import BookImage from '@mapbox/dr-ui/book-image';"
  - "import UserAccessToken from '../../components/user-access-token';"
contentType: tutorial
---

{{
<Note
  title="Access to Mapbox Boundaries"
  imageComponent={<BookImage />}
>
  <p>Access to the Boundaries tilesets are controlled by Mapbox account access token. If you do not have access on your account, <a href='https://www.mapbox.com/contact/'>contact a Mapbox sales representative</a> to request access to Boundaries tilesets.</p>
</Note>
}}

Mapbox企业用户可以为其地图和数据可视化添加全局管理，邮政和统计边界。本指南介绍了如何使用Mapbox Boundaries和Mapbox GL JS创建数据连接，以便为等值区域地图设计样式。

![final choropleth map using Boundaries, local data, and expressions](/help/img/data/eb-data-join-final.png)

## 开始

Mapbox Boundaries 是企业计划的一部分。如果您没有企业计划，或者如果您有企业计划，并想要获取Mapbox边界的访问权限 <a href='https://www.mapbox.com/contact/'>请联系Mapbox销售代表</a> 。对边界图块集的访问权限由您的Mapbox账户访问令牌控制。

## 关于数据连接

数据连接技术使用数据驱动的样式表示法，将您的自定义本地数据（例如美国各州的失业率）与矢量瓦片要素（例如适当的Mapbox Boundaries图块集中的州边界）之间进行内部连接。

## 使用Mapbox Boundaries 创建数据连接

下面，您将使用Mapbox GL JS，本地数据，Mapbox Boundaries，要素状态和数据驱动样式的表达式把本地数据连接到矢量瓦片源，并为等值区域地图设计样式。

###  使用Mapbox GL JS 创建一个地图

首先，使用Mapbox GL JS初始化地图，请确保您使用的访问令牌来自您的账户， **该账户需要拥有Mapbox边界的访问权限**。

这是此示例的初始代码：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset='utf-8' />
    <title>Join local JSON data with Boundaries</title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <script src='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.js'></script>
    <link href='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.css' rel='stylesheet' />
    <style>
      body {
        margin: 0;
        padding: 0;
      }

      #map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
      }
    </style>
  </head>
  <body>


  <div id='map'>
  </div>
  <script>
    mapboxgl.accessToken = '{{ <UserAccessToken /> }}';

    // Initialize a map
    var map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/light-v{{constants.VERSION_LIGHT_STYLE}}',
      center: [-99.9, 41.5],
      zoom: 1
    });

    // Local data code from the next step will go here.
  </script>

  </body>
</html>
```

### 本地数据
在本示例中，您将把美国的失业数据连接到admin-1 Boundaries 图块集。在这里，将对象数组设置为一个名为`localData`的变量. 在您自己的应用中，您可以根据喜好将本地数据添加入应用中。

这个示例中用到的数据是一个名为`localData` 的JavaScript变量，此数据可以被直接添加到HTML文件的`script`标签里，与代码一同初始化地图。

```js
var localData = [
  { STATE_ID: '01', unemployment: 13.17 },
  { STATE_ID: '02', unemployment: 9.5 },
  { STATE_ID: '04', unemployment: 12.15 },
  { STATE_ID: '05', unemployment: 8.99 },
  { STATE_ID: '06', unemployment: 11.83 },
  { STATE_ID: '08', unemployment: 7.52 },
  { STATE_ID: '09', unemployment: 6.44 },
  { STATE_ID: '10', unemployment: 5.17 },
  { STATE_ID: '12', unemployment: 9.67 },
  { STATE_ID: '13', unemployment: 10.64 },
  { STATE_ID: '15', unemployment: 12.38 },
  { STATE_ID: '16', unemployment: 10.13 },
  { STATE_ID: '17', unemployment: 9.58 },
  { STATE_ID: '18', unemployment: 10.63 },
  { STATE_ID: '19', unemployment: 8.09 },
  { STATE_ID: '20', unemployment: 5.93 },
  { STATE_ID: '21', unemployment: 9.86 },
  { STATE_ID: '22', unemployment: 9.81 },
  { STATE_ID: '23', unemployment: 7.82 },
  { STATE_ID: '24', unemployment: 8.35 },
  { STATE_ID: '25', unemployment: 9.1 },
  { STATE_ID: '26', unemployment: 10.69 },
  { STATE_ID: '27', unemployment: 11.53 },
  { STATE_ID: '28', unemployment: 9.29 },
  { STATE_ID: '29', unemployment: 9.94 },
  { STATE_ID: '30', unemployment: 9.29 },
  { STATE_ID: '31', unemployment: 5.45 },
  { STATE_ID: '32', unemployment: 4.21 },
  { STATE_ID: '33', unemployment: 4.27 },
  { STATE_ID: '34', unemployment: 4.09 },
  { STATE_ID: '35', unemployment: 7.83 },
  { STATE_ID: '36', unemployment: 8.01 },
  { STATE_ID: '37', unemployment: 9.34 },
  { STATE_ID: '38', unemployment: 11.23 },
  { STATE_ID: '39', unemployment: 7.08 },
  { STATE_ID: '40', unemployment: 11.22 },
  { STATE_ID: '41', unemployment: 6.2 },
  { STATE_ID: '42', unemployment: 9.11 },
  { STATE_ID: '44', unemployment: 10.42 },
  { STATE_ID: '45', unemployment: 8.89 },
  { STATE_ID: '46', unemployment: 11.03 },
  { STATE_ID: '47', unemployment: 7.35 },
  { STATE_ID: '48', unemployment: 8.92 },
  { STATE_ID: '49', unemployment: 7.65 },
  { STATE_ID: '50', unemployment: 8.01 },
  { STATE_ID: '51', unemployment: 7.62 },
  { STATE_ID: '53', unemployment: 7.77 },
  { STATE_ID: '54', unemployment: 8.49 },
  { STATE_ID: '55', unemployment: 9.42 },
  { STATE_ID: '56', unemployment: 7.59 }
];
```

### Mapbox Boundaries查询表

每个Boundaries图块集都有自己的要素查询表， 每个要素都能被编入索引。查询表被设计在您的本地应用中使用。您可以在 [Mapbox Boundaries入门](/help/tutorials/get-started-enterprise-boundaries/#about-feature-lookup-tables) 里阅读更多有关要素查询表的信息。

以下的代码片段阐释了如何从应用文件中导入要素查询表，然后在导入地图时发出API请求，检索查询表的内容，并标记控制台的响应。 您需要将`./path/to/lookup/table` 替换为到适当查询表的路径，该路径可在Boundaries访问中提供的参考文件找到。

```js
const lookupTable = require('./path/to/lookup/table')
map.on('load', function () {
  createViz(lookupTable);
});

function createViz(lookupData) {
  var dataValues = lookupData.data;
  console.log(dataValues);
}
```

在控制台探索响应，以了解更多有关查询表中包含的内容，并更好地了解在下一步中您将要怎样运用它。

### 要素特征

**要素特征** 是一组可以动态地分配给地图上的要素的属性。[Mapbox GL JS 要素特征](https://www.mapbox.com/mapbox-gl-js/api/#map#setfeaturestate) API  可用于对矢量或GeoJSON源的要素进行动态样式化，从而实现处理地图交互，数据连接，和时间序列动画效果的方法。

在前面步骤中定义的 `createViz` 函数的基础上, 将 Mapbox Boundaries admin-1 图块集添加为名为 `statesData`的源。

{{
<Note
  imageComponent={<BookImage />}
>
  <p>您可以在 <a href="https://www.mapbox.com/vector-tiles/enterprise-boundaries-v2">参考文件</a> 中找到完整的Mapbox Boundaries 图块集列表.</p>
</Note>
}}

然后，在 `createViz` 里创建一个名为`setStates`的新函数来设置要素特征. 要将本地失业数据连接到矢量瓦片边界数据，您需要一个可用于匹配本地数据和矢量数据之间相似特征的属性。 在Boundaries要素查询列表中，每个美国州的信息都被用一组唯一的字符串表示： `US` (表示美国) + `A1` (表示 Admin 1) + 某个数字(表示某个州)。在本地数据中，每个州都用一个数字表示。 这个数字与特征查询列表里的数字一一对应，所以可以将JSON失业数据与对应（对象标识符 === `USA1` + `STATE_ID`）的矢量要素相连接.

要素特征要获知每个矢量要素的 `id`。 此唯一的标识符必须仅包含整数。 在Mapbox Boundaries查询表中， `id_int` 是唯一的整数标识符。 要获取矢量瓦片要素`id` 以设置特征状态，您最终会得到 `id: dataValues["USA1" + row["STATE_ID"]].id_int`从查询列表中寻 `dataValues` 中的对象，其标识符等于 `"USA1" + row["STATE_ID"]` 然后得到 `id_int`属性的值.

最后，在调用自定义 `setState` 函数来设置特征状态前，您将要等待直到 `statesData` 源已经被添加到地图。

```js
function createViz(lookupData) {
  var dataValues = lookupData.data;

  // Add Mapbox Boundaries source for state polygons.
  map.addSource('statesData', {
    type: 'vector',
    url: 'mapbox://mapbox.enterprise-boundaries-a1-v2'
  });

  // Join the JSON unemployment data with the corresponding vector features where
  // feautre.id === 'USA1' + `STATE_ID`.
  function setStates(e) {
    localData.forEach(function(row) {
      map.setFeatureState({ source: 'statesData', sourceLayer: 'boundaries_admin_1', id: dataValues['USA1' + row.STATE_ID].id_int }, { unemployment: row.unemployment });
    });
  }

  // Check if `statesData` source is loaded.
  function setAfterLoad(e) {
    if (e.sourceId === 'statesData' && e.isSourceLoaded) {
      setStates();
      map.off('sourcedata', setAfterLoad);
    }
  }

  // If `statesData` source is loaded, call `setStates()`.
  if (map.isSourceLoaded('statesData')) {
    setStates();
  } else {
    map.on('sourcedata', setAfterLoad);
  }
}
```

### 数据驱动样式表达式

现在已经连接了Mapbox Boundaries图块集里的本地数据和矢量数据，您可以根据本地失业数据的值，在Boundaries图块集里设置要素的样式。在 `createViz` 函数中，添加一个新的名为 `map.addLayer()`的图层。源将是 `statesData` (已在之前的步骤中添加), 并且 `source-layer` 将是 `boundaries_admin_1`.

您可以使用表达式，根据要素特征中失业数据的值，来设置每个要素的填充色。Mapbox GL JS表达式使用类似Lisp的语法，使用JSON数组表示。表达式遵循以下格式：
```
[expression_name, argument_0, argument_1, ...]
```

`expression_name` 是 **表达式运算符**, 例如，您可以使用 [`'*'`](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions-*)乘以两个参数，或使用 [`'case'`](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions-case)来创建条件逻辑。 有关所有可用表达式的完整列表，请参阅[Mapbox样式规范](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions).

 **参数** 可以是文字（数字，字符串或布尔值），也可以是自身表达式。 参数的数量因表达式而异。

在此例中，您将要使用混合的表达式来设置等值区域地图的数据。

- [`case`](https://www.mapbox.com/mapbox-gl-js/style-spec#expressions-case):  使用 `case` 表达式（1）检查失业特征状态属性是否为空，（2）如果失业不为空，则根据失业价值分配填充颜色，（3）如果为空， 您将指定一个填充色rgba `rgba(255, 255, 255, 0)`.
- [`feature-state`](https://www.mapbox.com/mapbox-gl-js/style-spec#expressions-feature-state): 使用 `feature-state` 表达式检索当前要素状态中失业属性的值。
- [`!=`](https://www.mapbox.com/mapbox-gl-js/style-spec#expressions-!=): 使用 `!=` 表达式检查要素状态失业属性是否不为空。
- [`interpolate`](https://www.mapbox.com/mapbox-gl-js/style-spec#expressions-interpolate): 使用 `interpolate` 表达式将填充颜色分配给两个不同的失业值，并推断出站点之间连续的、平滑的填充颜色集。

当您把所有表达式放在一起，您的代码将如下所示：
```js
map.addLayer({
  id: 'states-join',
  type: 'fill',
  source: 'statesData',
  'source-layer': 'boundaries_admin_1',
  paint: {
    'fill-color':
    ['case',
      ['!=', ['feature-state', 'unemployment'], null],
      ['interpolate', ['linear'], ['feature-state', 'unemployment'], 4, 'rgba(222,235,247,1)', 14, 'rgba(49,130,189,1)'],
      'rgba(255, 255, 255, 0)'
    ]
  }
}, 'waterway-label');
```

## 最终成果

您用数据连接和Mapbox Boundaries创建了一个等值区域地图。

![final choropleth map using Boundaries, local data, and expressions](/help/img/data/eb-data-join-final.png)

这是完整的代码：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset='utf-8' />
    <title>Join local JSON data with vector tile geometries</title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <script src='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.js'></script>
    <link href='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.css' rel='stylesheet' />
    <style>
      body {
        margin: 0;
        padding: 0;
      }

      #map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
      }
    </style>
  </head>
  <body>
    <div id='map'></div>
    <script>
      mapboxgl.accessToken = '{{ <UserAccessToken /> }}';
      var map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/light-v{{constants.VERSION_LIGHT_STYLE}}',
        center: [-99.9, 41.5],
        zoom: 1
      });

      // Join local JSON data with vector tile geometry
      // unemployment rate in 2009
      // Source https://data.bls.gov/timeseries/LNS14000000
      var maxValue = 14;
      var localData = [
        { STATE_ID: '01', unemployment: 13.17 },
        { STATE_ID: '02', unemployment: 9.5 },
        { STATE_ID: '04', unemployment: 12.15 },
        { STATE_ID: '05', unemployment: 8.99 },
        { STATE_ID: '06', unemployment: 11.83 },
        { STATE_ID: '08', unemployment: 7.52 },
        { STATE_ID: '09', unemployment: 6.44 },
        { STATE_ID: '10', unemployment: 5.17 },
        { STATE_ID: '12', unemployment: 9.67 },
        { STATE_ID: '13', unemployment: 10.64 },
        { STATE_ID: '15', unemployment: 12.38 },
        { STATE_ID: '16', unemployment: 10.13 },
        { STATE_ID: '17', unemployment: 9.58 },
        { STATE_ID: '18', unemployment: 10.63 },
        { STATE_ID: '19', unemployment: 8.09 },
        { STATE_ID: '20', unemployment: 5.93 },
        { STATE_ID: '21', unemployment: 9.86 },
        { STATE_ID: '22', unemployment: 9.81 },
        { STATE_ID: '23', unemployment: 7.82 },
        { STATE_ID: '24', unemployment: 8.35 },
        { STATE_ID: '25', unemployment: 9.1 },
        { STATE_ID: '26', unemployment: 10.69 },
        { STATE_ID: '27', unemployment: 11.53 },
        { STATE_ID: '28', unemployment: 9.29 },
        { STATE_ID: '29', unemployment: 9.94 },
        { STATE_ID: '30', unemployment: 9.29 },
        { STATE_ID: '31', unemployment: 5.45 },
        { STATE_ID: '32', unemployment: 4.21 },
        { STATE_ID: '33', unemployment: 4.27 },
        { STATE_ID: '34', unemployment: 4.09 },
        { STATE_ID: '35', unemployment: 7.83 },
        { STATE_ID: '36', unemployment: 8.01 },
        { STATE_ID: '37', unemployment: 9.34 },
        { STATE_ID: '38', unemployment: 11.23 },
        { STATE_ID: '39', unemployment: 7.08 },
        { STATE_ID: '40', unemployment: 11.22 },
        { STATE_ID: '41', unemployment: 6.2 },
        { STATE_ID: '42', unemployment: 9.11 },
        { STATE_ID: '44', unemployment: 10.42 },
        { STATE_ID: '45', unemployment: 8.89 },
        { STATE_ID: '46', unemployment: 11.03 },
        { STATE_ID: '47', unemployment: 7.35 },
        { STATE_ID: '48', unemployment: 8.92 },
        { STATE_ID: '49', unemployment: 7.65 },
        { STATE_ID: '50', unemployment: 8.01 },
        { STATE_ID: '51', unemployment: 7.62 },
        { STATE_ID: '53', unemployment: 7.77 },
        { STATE_ID: '54', unemployment: 8.49 },
        { STATE_ID: '55', unemployment: 9.42 },
        { STATE_ID: '56', unemployment: 7.59 }
      ];

      const lookupTable = require('./path/to/lookup/table')
      map.on('load', function () {
        createViz(lookupTable);
      });

      function createViz(lookupData) {
        var dataValues = lookupData.data;
        // Add Mapbox Boundaries source for state polygons.
        map.addSource('statesData', {
          type: 'vector',
          url: 'mapbox://mapbox.enterprise-boundaries-a1-v2'
        });

        // Add layer from the vector tile source with data-driven style
        // Use a feature-state dependent expression to compute the green color band based on
        //  the unemployment percentage
        map.addLayer({
          id: 'states-join',
          type: 'fill',
          source: 'statesData',
          'source-layer': 'boundaries_admin_1',
          paint: {
            'fill-color':
            ['case',
              ['!=', ['feature-state', 'unemployment'], null],
              ['interpolate', ['linear'], ['feature-state', 'unemployment'], 4, 'rgba(222,235,247,1)', 14, 'rgba(49,130,189,1)'],
              'rgba(255, 255, 255, 0)'
            ]
          }
        }, 'waterway-label');

        // Join the JSON unemployment data with the corresponding vector features where
        // feautre.id === 'USA1' + `STATE_ID`.
        function setStates(e) {
          localData.forEach(function(row) {
            map.setFeatureState({ source: 'statesData', sourceLayer: 'boundaries_admin_1', id: dataValues['USA1' + row.STATE_ID].id_int }, { unemployment: row.unemployment });
          });
        }

        // Check if `statesData` source is loaded.
        function setAfterLoad(e) {
          if (e.sourceId === 'statesData' && e.isSourceLoaded) {
            setStates();
            map.off('sourcedata', setAfterLoad);
          }
        }

        // If `statesData` source is loaded, call `setStates()`.
        if (map.isSourceLoaded('statesData')) {
          setStates();
        } else {
          map.on('sourcedata', setAfterLoad);
        }
      }
    </script>
  </body>
</html>
```

## 下一步

了解更多如何使用Mapbox Boundaries

### 更多Mapbox Boundaries 教程

探索我们的其他Mapbox Boundaries教程

- [使用 Mapbox Boundaries点对面查询](/help/tutorials/point-in-polygon-query-with-enterprise-boundaries/): 使用 Mapbox Boundaries Tilequery API计算单个点上存在的面。
- [扩展 Mapbox Boundaries ](/help/tutorials/extend-enterprise-boundaries/): 您可以使用应用程序所需的任何自定义数据扩展mapbox boundaries。这可能意味着添加学校区域，城市，市场，或者所有权边界到您的应用程序中，都具有与原产品相同的性能和API特性。

### 高级用例

Y您还可以探索此 [示例](https://www.mapbox.com/labs-sandbox-demos/vt_polygons/), 该示例使用了本地数据连接教程和 [点对面查询](/help/tutorials/point-in-polygon-query-with-enterprise-boundaries/) 教程中概述的概念，创建了一个具有交互式等值区域地图的应用程序。

![an end to end example using the data-join technique and Tilequery API](/help/img/data/enterprise-boundaries-choropleth-demo.gif)
