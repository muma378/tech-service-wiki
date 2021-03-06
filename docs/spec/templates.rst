.. spec_templates:

=================
标注模板简明指南
=================

数据堂标注平台_ 提供了一系列的标注模板以完成各类标注任务，简单来说，包括 **语音内容转写** ， **街景图片标注** ， **人脸关键点标注** 和 **视频物体追踪** 。我们在这篇文档中将集中介绍以上四种类型的标注任务常用的模板。

不同的客户常常有着定制化的需求，即使同一类型的标注任务也会对应十多种模板，我们不会详细列举所有模板的功能，而是选取每类任务中最基础最通用的模板进行说明，以期帮助大家建立一个全局的印象。

对每类模板的说明我们会按照以下几个方面进行：

* 功能介绍。模板实现的业务功能，包括使用场景、操作界面和基本属性；
* 平台格式。平台相关的索引文件格式和标注导出结果格式说明。需要注意的是，导出格式中通常包含着很多通用字段，如 `_guid` 、 `_id` 、 `_personInProjectId` 、 `Workload` 等。为了使格式显示更加清晰，我们在介绍模板的时候去掉了这些通用字段，而在本篇结尾 :ref:`common_fields_intro` 统一进行说明；
* 交付建议。对这类标注数据序列化，交付客户的最佳格式建议；
* 相关扩展。相似模板及可能引入的修改点。

另外，我们提供了一个基于 **Moose1.0** 的项目 datatang-base_ ，提供针对这几类不同的模板编写代码的思路。

-----------------------------------------

.. _audio-transliterate:

语音内容转写
=============

多段落语音标注v2.1.1
-------------------------

**> 功能介绍**

**多段落语音标注v2.1.1** 提供了对一段音频中每句话的内容进行转写、标定开始和结束时间的功能。利用这些标注信息，数据处理工程师可以将一段音频分割成多个单句，并且提供单句对应的文本内容。

**> 平台格式**

- 索引格式： ::

    {
    "url" : "G4297/IOS0-6.wav",
    "urlList" : ["G4297/IOS_G4297_S0000.wav", "G4297/IOS_G4297_S0001.wav", "G4297/IOS_G4297_S0002.wav",],
    }


- 导出格式：

  Source ::

    {
    "url" : "G4297/IOS0-6.wav",
    "urlList" : ["G4297/IOS_G4297_S0000.wav", "G4297/IOS_G4297_S0001.wav", "G4297/IOS_G4297_S0002.wav",],
    },


  Result ::

    {
    "filename" : "G4300/IOS0-4wav",
    "effective" : "true",
    "markResult" : [{
        "start" : 2.703125,
        "end" : 6.53125,
        "title" : "",
        "extend" : {
          "formValue" : [],
          "content" : "妈妈这（ze4）些小树（su4）苗们在跳舞[N]"
        }
      }, {
        "start" : 7.328125,
        "end" : 10.6875,
        "title" : "",
        "extend" : {
        "content" : "[N]就像做错了什（sen3）么事（si4）[N]",
        "formValue" : []
        }
      },
      ...
      ]
      }


**> 交付建议**

在处理这类数据时，我们建议将每段音频按照时间点切割成单句，并提供与音频名称相同的文本文件——仅包含一行转写内容（无换行符）的 **.txt** 文件。

在文件命名上，切割后的文件按照 **原始音频名** + **_S** + **按照起始时间排序从1开始的索引，并用0左补齐** 命名，所有的切割后的文件放在以 **原始音频名（不包含后缀）** 命名的文件夹下，具体文件结构和格式如下：

- 文件结构：

  定制数据 ::

    第XXX期语音标注任务/
      wav_name1/
        wav_name1_S0001.wav
        wav_name1_S0001.txt
        ...
      wav_name2/
        wav_name2_S0001.wav
        wav_name2_S0001.txt
        ...

  自制数据 ::

    data/
      category/
        G0001/
          session_Phone/
            Phone_G0001_S0001.wav
            Phone_G0001_S0001.txt
            Phone_G0001_S0001.metadata
            ...
          session_Mic/
            Mic_G0001_S0001.wav
            Mic_G0001_S0001.txt
            Mic_G0001_S0001.metadata
            ...
          ...
        G0002/
        ...

- 文件格式：

  wav_name1_S0001.txt ::

    妈妈这（ze4）些小树（su4）苗们在跳舞[N]


**> 扩展话题**

* 除了语音内容转写，这类模板也可能包含录音人或音频信息的属性标注。如果包含其他属性的标注，则需要属性名称和值行成的 **.json** 文件。关于属性名称的选择可以参考本章的 `属性命名参考列表`_ ，文件格式如下：

  wav_name1_S0001.json ::

    {
    "sex": "female",
    "age": 5,
    "content": "妈妈这（ze4）些小树（su4）苗们在跳舞[N]"
    }

*

多国语音标注模板V1.0
-------------------------

**> 功能介绍**

**多国语音标注模板V1.0** 是针对于采集转标注语音任务的一个比较完善的模板，它提供了之后进行自有数据入库所需要的说话人信息，原始语料文本，有效音频起止时间，标注后矫正文本以及音频时长信息，数据处理人员可以根据这些信息来对数据进行入库处理和导出操作。

**> 平台格式**

- 具体实现：

  采集转标注主要原理是将采集的数据**（acquisition表）**作为原始数据插入到原始数据表**（DataSource表）**中，我们需要从**acquisition表**中查到我们需要的字段，通过sql语句将这些数据插入到**DataSource表**中，当然在插入数据之前需要对采集数据进行一系列的验证工作，避免出现插错数据的情况，目前也正在做采集转标注的自动化接口，接口还在测试中，后续会更新。
  同时对于我们内部APP采集的数据每一个号段我们都会有一个对应人的信息文件，文件名为**xxxinfo.txt**,这个文件包含了录音人以及其使用的设备，所处的环境等信息，具体内容如下所示：::

    {
    "name": "周某某",
    "age": "24",
    "idcard": "08066695335",
    "local_accent": "北方地区",
    "city": "辽宁省锦州市",
    "sex": "女",
    "version": "2.3",
    "MobileType": "vivo X9s",
    "imei": "866352035390911",
    "gps": "116.353688,40.002153"
    }

  当然，如果是采集转标注的数据，这个文件的信息也会在我们这个标注模板中包含。

- 导出格式：

  Source ::

    {}

  由于是采集转标注，没有上传索引且没有往MongoDB中插入任何数据，所以值为空

  Result ::

    {
    "hdTimeEnd": 5.6,
    "hdTimeStart": 0,
    "Content": "O comboio sai às vinte e três e trinta em ponto.",
    "Effective": "1",
    "duration": "5.60000",
    "userinfo": {
      "name": "Audrey Siqueira",
      "age": "28",
      "idcard": "FR045656",
      "local_accent": "Southeast area",
      "native_place": "Sao Paulo",
      "city": "Beijing",
      "recording_environment": "Quiet",
      "sex": "Male",
      "version": "2.3.1",
      "MobileType": "MHA-AL00",
      "imei": "866229036613964",
      "gps": ""
    },
    "defaultCont": "O comboio sai às 23.30 em ponto.",
    "Workload": {
      "effectiveDuration": 5.6,
      "invalidDuration": 0,
      "taginvalidDuration": 0,
      "workTime": 9.5551333333333339
    },
    "title": "T0503G0016S0193.wav",
    "_personInProjectId": 578441,
    "_createTime": "2018/7/2 16:45:56",
    "_guid": "f208ac7b-945e-44b5-83a0-299c921c5b0e"
    }



**> 交付建议**

在处理这类数据时，我们建议交付每段音频以及并提供与音频名称相同的文本文件——仅包含一行标注音频内容（无换行符）的 **.txt** 文件。所有文件放在一个目录下方。

- 文件结构：

  同 多段落语音标注v2.1.1_ 

- 文件格式：

  同 多段落语音标注v2.1.1_ 


**> 扩展话题**


-----------------------------------------

.. _cityscape_images:

街景图片标注
================

街景矩形框、多边形v3.8
--------------------------------

**> 功能介绍**

**街景矩形框、多边形v3.8** 提供了一张图片中每个标注物的坐标信息、标注物相关的属性以及标
注物互相之间的关联关系，数据处理工程师可以使用这些坐标信息将标注框还原到图片上，生成不同类型的图像，并且知道标注物之间的关系。

.. figure:: /_static/spec/templates/cityscape_template.png
    :alt: 街景矩形框、多边形v3.8
    :align: center

    街景矩形框、多边形v3.8 操作界面

**> 平台格式**

- 索引格式 ::

    url1.jpg
    url2.jpg
    url3.jpg
    ...

- 导出格式

  Source ::

    {
      'dataTitle': 'url1.jpg',
      'url': 'url1.jpg',
    }

  Result ::

    {
      'effective': 1,
      'markResult': {
      'type': 'FeatureCollection',
      'features': [{
        'type': 'Feature',
        'geometry': {
          'type': 'LineString',
          'coordinates': [
            [606.8049645037071, 297.8348053163902],
            [545.8049645037071, 311.3348053163902],
            [327.3049645037071, 356.8348053163902],
            [161.80496450370708, 395.3348053163902],
            [4.804964503707083, 436.3348053163902]
          ]
        },
        'properties': {
          'type': {
            'parentDataKey': 'whitedashed',
            'currentDataKey': 'parallel',
            'currentDataTitle': '纵向',
            'parentDataTitle': '白色虚线'
          },
          'quality': {

          },
          'index': 1
        },
        'title': '1-白色虚线-纵向'
      }]
    }

.. note:: 上述 **导出格式** 的 `Result` 格式是基于 ``GeoJSON`` （RFC7946_）修改形成的，具体细节请参考 :doc:`geojson`

**> 交付建议**

这类数据我们建议提供与图片名称相同的 **.json** 文件，文件内容包含标注的线/矩形/多边形的坐标点和与之相关的必要的属性，**json** 各字段的组织应尽量直接、平坦（flat），在可能的条件下，不应包含过多的嵌套。同时，提供对应的原始图、掩模图以及效果图。
具体文件结构格式如下：

- 文件结构：

  定制数据 ::

    第XXX期图片标注任务/
        dirname1/
            image_name.jpg
            image_name.json
            image_name_mask.jpg
            image_name_blend.jpg
            ...
        dirname2/
            ...

  自制数据 ::

    data/
        category/
            G0001/
                session_jpg/
                    imgname_G0001_S0001.jpg
                    imgname_G0001_S0001.json
                    imgname_G0001_S0001.metadata
                    ...
                ...
            G0002
            ...

- 文件格式：

  image_name.json ::

    [{
      "label": "car",
      "coordinates": [
        [606.8049645037071, 297.8348053163902],
        [545.8049645037071, 311.3348053163902],
        [327.3049645037071, 356.8348053163902],
        [161.80496450370708, 395.3348053163902],
        [4.804964503707083, 436.3348053163902],
        [606.8049645037071, 297.8348053163902]
      ],
      "type": "Polygon",
      "id": 1,
      },
      ...
    ]

- 图片示例：

.. figure:: /_static/spec/templates/cityscape.jpg
    :alt: raw
    :align: center

    原始图

.. figure:: /_static/spec/templates/cityscape_mask.jpg
    :alt: mask
    :align: center

    掩模图

.. figure:: /_static/spec/templates/cityscape_blend.jpg
    :alt: blend
    :align: center

    效果图


**> 扩展话题**

* 这类模板也可能包括各标注物之间的关联关系，一般在导出数据中包含 **relativeAssis** 或者 **link** 这样的属性字段，里面包含其所关联的父标注物id，例如 ::

    "link":{
        "linkId": 10,
        "linkTitle": "轿车",
        "val": "10-轿车
        }

表示其关联id为10的轿车，或者 ::

    "relativaAssis": 4

表示其与id为4的标注对象有关联。

* 同时这类模板可能会被要求导出标注框的总数以及标注的不同类型框的数量，具体信息可以参照本章的 `属性命名参考列表`_, 但是导出结果数据可能不太准确，所以建议自己计算出数量导出。

-----------------------------------------

.. _face_keypoints:

人脸关键点标注
================

人脸106关键点标注v2.0
----------------------

**> 功能介绍**


**> 平台格式**


**> 交付建议**


**> 扩展话题**

-----------------------------------------

.. _objects_tracking:

视频物体追踪
================

单镜头街景标注v1.6
----------------------

**> 功能介绍**


**> 平台格式**


**> 交付建议**


**> 扩展话题**


------------------------------------------


.. _common_fields_intro:

平台通用字段说明
================

每个模板都包含一些通用字段，它们通常由系统自动生成，并与SQL数据库中的某个字段相同，便于交叉查询。在系统中，每条数据对应 **Source** 和 **Result** 两个集合（collection），分别代表这条数据的原始信息（标注前）和标注信息（标注后），因此，我们在每个字段后用等宽文本（monospaced text）来注明这个字段所在的集合。

_guid ``Source`` ``Result``
    每条数据的全局唯一标识符， ``Source`` 中的_guid与SQL数据库中 ``DataSource`` 表中的 `DataGuid` 和 ``DataResult`` 表中的 `SourceGuid` 一致， ``Result`` 中的_guid与SQL数据库中 ``DataResult`` 表中的 `DataGuid` 一致。

_personInProjectId ``Result``
    该数据标注人员的在项目中的ID，与SQL数据库中 ``PersonInProject`` 表中的 `id` 一致。

_id ``Source`` ``Result``
    MongoDB中插入数据时自动生成的唯一值。

_createTime ``Source`` ``Result``
    该条数据被创建的时间。

Workload ``Result``
    对标注各个类别的数量统计（不准，不建议使用）。

markCount ``Result``
    标注数量的统计。

url ``Source``
    原始文件在云平台上面的相对路径

effective ``Result``
    标注结果是否有效(1 表示有效)

markResult ``Result``
    总标注结果数据(包括坐标点和属性值)

geometry ``Result``
    每一个框的坐标点(每一个列表分别有两个坐标值，分别代表 **x** 和 **y** )

properties ``Result``
    每一个框的属性值

-----------------------------------------


.. _naming_reference:

属性命名参考列表
==================

因为客户对标注属性的选择不一，各模板产生的字段也不尽不同，我们无法提供一个统一的格式来覆盖所有的可能数据集。但是，我们在此提供了一个属性命名参考列表，避免不同项目中同一含义的属性使用多套命名规则：

+-------------------+-----------------+
|导出属性名称       | 属性命名        |
+===================+=================+
| filename          | 文件名          |
+-------------------+-----------------+
| prop              | 标注属性列表    |
+-------------------+-----------------+
| arr               | 坐标点列表      |
+-------------------+-----------------+
| id                | 标注框id        |
+-------------------+-----------------+
| sex               | 性别            |
+-------------------+-----------------+
| age               | 年龄            |
+-------------------+-----------------+
| parentId          | 关联id          |
+-------------------+-----------------+
| solid             | 实线            |
+-------------------+-----------------+
| dashed            | 虚线            |
+-------------------+-----------------+
| dotted            | 点线            |
+-------------------+-----------------+
|                   |                 |
+-------------------+-----------------+
| content           | 文本内容        |
+-------------------+-----------------+
|                   |                 |
+-------------------+-----------------+
|                   |                 |
+-------------------+-----------------+
|                   |                 |
+-------------------+-----------------+










.. _数据堂标注平台: http://bz.datatang.com/
.. _datatang-base: http://git.datatang.com/xiaoyang/datatang_base
.. _RFC7946: https://tools.ietf.org/html/rfc7946
