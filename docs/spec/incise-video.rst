.. spec_incise-video:

================
视频打点规范说明
================

.. _audio-transliterate:

1.打点工具说明
--------------------

为"PopSub0.75测试版.rar"，使用"手册请参考PopSub软件使用示范说明.docx".

2.打点文件说明
---------------

打点文件必须为xlsx格式的excel文件

文件命名必须为对应视频名除去后缀的名称.

- 示例如下：

::

	eg:原视频名为"Iron Man 3 2013 1080p BRRip x264 AC3.mp4"，对应打点文件名为"Iron Man 3 2013 1080p BRRip x264 AC3.xlsx"；
	

3.视频格式说明
----------------

被截取视频必须为MP4格式.

4.打点excel文件内容说明
-------------------------

打点excel文件中必须依次提供"编号"、"起始时间"和"终止时间"三个字段数据，后面为自己扩展字段。时间格式必须为"X:X:X.XX"，不能有中文符号.

- 详见"PopSub软件使用示范说明.docx"中的截图。

- 示例如下：

.. figure:: /_static/spec/incise-video/properties.jpg
	:align: center

5.打点时间格式说明
-------------------

打点时间格式必须为"h:mm:ss.00".

- 示例如下：
  
  1.选中B列(start time),点击鼠标右键，在下拉框中选择"设置单元格格式"。
  
	.. figure:: /_static/spec/incise-video/time-norm1.jpg
		:align: center

  2.选择“自定义”，在“类型”中输入“h:mm:ss.00”，点击确认。
  
	.. figure:: /_static/spec/incise-video/time-norm2.jpg
		:align: center
	
  3.C列(end time)同B列操作。
