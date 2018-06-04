.. spec_geojson:

================
GeoJSON规范说明
================

.. _intro:

1.GeoJSON简介
=============

``GeoJSON`` 是基于JavaScript Object Notation (JSON) 的地理空间数据可交换格式（geospatial data interchange format）。它定义了多种类型的 **JSON对象** 以及如何通过不同的组合的方式来表示相应的地理特征、属性和空间扩展（spatial extents）。

``GeoJSON`` 使用地理坐标参考系统（geographic coordinate reference system）—— World Geodetic System 1984，并以小数作为表示单元。

数据堂使用 ``GeoJSON`` 作为图像类标注信息的表示格式。


.. geojson_definition:

1.1 GeoJSON定义
-----------------

GeoJSON是一种对各种地理数据结构进行编码的格式。GeoJSON对象可以表示几何、特征或者特征集合。GeoJSON支持下面几何类型：点、线、多边形、多点、多线、多边形集合和几何集合。GeoJSON里的特征包含一个几何对象和其他属性，特征集合表示一系列特征。

一个完整的GeoJSON数据结构总是一个（JSON术语里的）对象。在GeoJSON里，对象由键/值对，也称作成员的集合组成。对每个成员来说，名字总是字符串。成员的值要么是字符串、数字、对象、数组，要么是下面文本常量中的一个：”true"，"false"和"null"。数组是由值是上面所说的元素组成。
我们常用到的类型包括：点（Point）、线（LineString）、多边形（Polygon）、多点（MultiPoint）、多线（MultiLineString）和多边形集合（MultiPolygon）。

JavaScript对象表示和术语对象、名字、值、数组和数字都定义在IETF RFC4627_ 里。


1.2 GeoJSON举例
-----------------

::

    {
    "type": "FeatureCollection",
    "features": [
      {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [102.0, 0.5]
        },
      "properties": {
        "prop0": "value0"
        }
      },
      {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]
          ]
        },
      "properties": {
        "prop0": "value0",
        "prop1": 0.0
        }
      },
      {
      "type": "Feature",
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
            [100.0, 1.0], [100.0, 0.0]
          ]
        ]},
      "properties": {
        "prop0": "value0",
        "prop1": {"this": "that"}
        }
      }
    ]
    }


-----------------------------------------

.. _geojson_object:

2.GeoJSON对象
===============

GeoJSON总是由一个单独的对象组成。这个对象（指的是下面的GeoJSON对象）表示几何、特征或者特征集合。

GeoJSON对象可能有任何数目成员（键/值对），必须由一个名字为 **type** 的成员。这个成员的值是由GeoJSON对象的类型所确定的字符串。type成员的值必须是下面之一：
    - **Point**
    - **MultiPoint**
    - **LineString**
    - **MultiLineString**
    - **Polygon**
    - **MultiPolygon**
    - **GeometryCollection**
    - **Feature**
    - **FeatureCollection**


.. _geometry_object:

2.1 几何对象(Geometry Object)
------------------------------

一个几何对象表示点、曲线和坐标空间中的面。

几何对象的type成员的值是下面字符串之一：**Point** 、 **MultiPoint** 、 **LineString** 、 **MultiLineString** 、 **Polygon** 、 **MultiPolygon** 或者 **GeometryCollection** 。

除了 **GeometryCollection** 外的其他任何类型的 **GeoJSON几何对象** 必须有一个名字为 **coordinates** 的成员。

**coordinates** 成员的值总是数组，这个数组里的元素的结构由几何类型来确定。

- 2.1.1 位置(Position)

    .. todo: rewrite

    位置是基本的几何结构。几何对象的 **coordinates** 成员由一个位置（这儿是几何点）、位置数组（线或者几何多点），位置数组的数组（面、多线）或者位置的多维数组（多面）组成。

    位置由数字数组表示。必须至少两个元素，可以有更多元素。元素的顺序必须遵从 **x，y，z** 顺序（投影坐标参考系统中坐标的东向、北向、高度或者地理坐标参考系统中的坐标长度、纬度、高度）。任何数目的其他元素是允许的---其他元素的说明和意义超出了这篇规格说明的范围。


- 2.1.2 点(Point)

    对类型 **Point** 来说， **coordinates** 成员必须是一个单独的位置。例如： ::

      { "type": "Point", "coordinates": [100.0, 0.0] }


- 2.1.3 多点(MultiPoint)

    对类型 **MultiPoint** 来说， **coordinates** 成员必须是位置数组。例如： ::

    	{ "type": "MultiPoint",
    	  "coordinates": [ [100.0, 0.0], [101.0, 1.0] ]
    	}

- 2.1.4 线(LineString)

    对类型 **LineString** 来说， **coordinates** 成员必须是两个或者多个位置的数组。例如： ::

      { "type": "LineString",
        "coordinates": [ [100.0, 0.0], [101.0, 1.0] ]
      }

- 2.1.5 多线（MultiLineString）

    对类型“MultiLineString"来说，"coordinates"成员必须是一个线坐标数组的数组。例如： ::

      { "type": "MultiLineString",
        "coordinates": [
          [ [100.0, 0.0], [101.0, 1.0] ],
          [ [102.0, 2.0], [103.0, 3.0] ]
          ]
      }

- 2.1.6 多边形（Polygon）

    对类型"Polygon"来说，"coordinates"成员必须是一个线性环坐标数组的数组。对拥有多个环的多边形来说，第一个环必须是外部环，其他的必须是内部环或者孔。

    - 无孔多边形： ::

          { "type": "Polygon",
            "coordinates": [
              [
                [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
                [100.0, 1.0], [100.0, 0.0]
              ]
            ]
          }

    - 含孔多边形： ::

          { "type": "Polygon",
            "coordinates": [
              [
                [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
                [100.0, 1.0], [100.0, 0.0]
              ],
              [
                [100.2, 0.2], [100.8, 0.2], [100.8, 0.8],
                [100.2, 0.8], [100.2, 0.2]
              ]
            ]
          }


- 2.1.7 多边形集合（MultiPolygon）

    对类型"MultiPlygon"来说，"coordinates"成员必须是面坐标数组的数组。例如： ::

        { "type": "MultiPolygon",
          "coordinates": [
                [[102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0], [102.0, 2.0]],
                [[[100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]],
                [[100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8], [100.2, 0.2]]]
            ]
        }

- 2.1.8 几何集合（GeometryCollection）

    类型为"GeometryCollection"的GeoJSON对象是一个集合对象，它表示几何对象的集合。

    几何集合必须有一个名字为"geometries"的成员。与"geometries"相对应的值是一个数组。这个数组中的每个元素都是一个GeoJSON几何对象。示例如下： ::

        { "type": "GeometryCollection",
          "geometries": [
            { "type": "Point",
              "coordinates": [100.0, 0.0]
            },
            { "type": "LineString",
              "coordinates": [ [101.0, 0.0], [102.0, 1.0] ]
            }
          ]
        }

.. _feature_object:

2.2 特征对象(Feature Object)

类型为"Feature"的GeoJSON对象是特征对象。特征对象必须由一个名字为"geometry"的成员，这个几何成员的值是上面定义的几何对象或者JSON的null值。特征对象那个必须有一个名字为“properties"的成员，这个属性成员的值是一个对象（任何JSON对象或者null值）。如果特征是常用的标识符，那么这个标识符应当包含名字为“id”的特征对象成员。

- 示例见2.4边界框的示例。

2.3 特征对象集合(FeatureCollection Object)

类型为"FeatureCollection"的GeoJSON对象是特征集合对象。特征集合对象必须由一个名字为"features"的成员。与“features"相对应的值是一个数组。这个数组中的每个元素都是上面定义的特征对象。

- 示例见2.4边界框的示例。

2.4 边界框(Bounding Box)

为了包含几何、特征或者特征集合的坐标范围信息，GeoJSON对象可能有一个名字为"bbox的成员。bbox成员的值必须是2*n数组，这儿n是所包含几何对象的维数，并且所有坐标轴的最低值后面跟着最高者值。bbox的坐标轴的顺序遵循几何坐标轴的顺序。除此之外，bbox的坐标参考系统假设匹配它所在GeoJSON对象的坐标参考系统。

- 特征对象上的bbox成员的例子：
::

    { "type": "Feature",
      "bbox": [-180.0, -90.0, 180.0, 90.0],
      "geometry": {
		  "type": "Polygon",
		  "coordinates": [[
			    [-180.0, 10.0], [20.0, 90.0], [180.0, -5.0], [-30.0, -90.0]
			]]
		}
      ...
    }
- 特征集合对象bbox成员的例子：
::

    { "type": "FeatureCollection",
      "bbox": [100.0, 0.0, 105.0, 1.0],
      "features": [
            ...
        ]
    }

-----------------------------------------

.. _geojson_cheat_sheet:

3.GeoJSON Cheat Sheet
======================

.. note: 为方便读者理解和查询，我们整合了Wiki上 ``GeoJSON`` 的介绍，以下表格来自 `GeoJSON Wiki`_ 。

.. _geometry_primitives:

3.1 Geometry 基本类型 （Geometry primitives）
------------------------------

.. list-table::
  :widths: 5 10 30
  :header-rows: 1

  * - Tpye
    - Picture
    - Examples
  * - Point
    - .. figure:: /_static/spec/geojson/51px-SFA_Point.svg.png
         :width: 50
    - ::

	{
	  "type": "Point",
	  "coordinates": [30, 10]
	}
  * - LineString
    - .. figure:: /_static/spec/geojson/51px-SFA_LineString.svg.png
         :width: 50
    - ::

	{
	  "type": "LineString",
	  "coordinates": [
		  [30, 10], [10, 30], [40, 40]
		]
	}
  * - Polygon
    - .. figure:: /_static/spec/geojson/SFA_Polygon.svg.png
         :width: 50
    - ::

	{
	  "type": "Polygon",
	  "coordinates": [
		  [[30, 10],[40, 40],
		  [20, 40],[10, 20],[30, 10]]
		]
	}
  * - Polygon (hole)
    - .. figure:: /_static/spec/geojson/SFA_Polygon_with_hole.svg.png
         :width: 50
    - ::

	{
	  "type": "Polygon",
	  "coordinates": [
		  [
		   [35, 10],[45, 45],[15, 40],
		   [10, 20],[35, 10]
		  ],
		  [
		   [20, 30], [35, 35],
		   [30, 20], [20, 30]
		  ]
		]
	}

.. _multipart_geometries:

3.2 Geometry 组合（Multipart geometries）
------------------------------

.. list-table::
  :widths: 5 15 30
  :header-rows: 1

  * - Tpye
    - Picture
    - Examples
  * - Multi-Point
    - .. figure:: /_static/spec/geojson/51px-SFA_MultiPoint.svg.png
         :width: 50
    - ::

	{
	  "type": "MultiPoint",
	  "coordinates": [
		  [10, 40], [40, 30],
		  [20, 20], [30, 10]
		]
	}
  * - Multi-LineString
    - .. figure:: /_static/spec/geojson/51px-SFA_MultiLineString.svg.png
         :width: 50
    - ::

	{
	  "type": "MultiLineString",
	  "coordinates": [
		 [[10, 10],[20, 20],[10, 40]],
		 [[40, 40], [30, 30],
		 [40, 20], [30, 10]]
		]
	}
  * - Multi-Polygon
    - .. figure:: /_static/spec/geojson/SFA_MultiPolygon.svg.png
         :width: 50
    - ::

	{
	  "type": "MultiPolygon",
	  "coordinates": [
		  [
		   [[30, 20], [45, 40],
		   [10, 40], [30, 20]]
		  ],
		  [
		   [[15, 5],[40, 10],[10, 20],
		   [5, 10], [15, 5]]
		  ]
		]
	}
  * - Multi-Polygon (hole)
    - .. figure:: /_static/spec/geojson/SFA_MultiPolygon_with_hole.svg.png
         :width: 50
    - ::

	{
	  "type": "MultiPolygon",
	  "coordinates": [
		  [
		   [[30, 20], [45, 40],
		   [10, 40], [30, 20]]
		  ],
		  [
		   [[15, 5],[40, 10],[10, 20],
		   [5, 10], [15, 5]]
		  ]
		]
	}

.. _RFC4627: http://www.ietf.org/rfc/rfc4627.txt
.. _GeoJSON RFC7946: https://tools.ietf.org/html/rfc7946
.. _GeoJSON Wiki: https://en.wikipedia.org/wiki/GeoJSON
