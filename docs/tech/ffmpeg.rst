.. tech_ffmpeg:

=============================
FFmpeg 工作原理和常用命令
=============================

.. note:: 本文档基于 `FFmpeg官方文档`_ 形成，旨在对其中原理性的内容进行说明，以帮助英语不好的读者理解基础原理；另外，对其中较常用的命令选项进行总结，帮助新手快速上手。由于译者水平有限，可能存在词不达意，学有余力的读者建议阅读 `FFmpeg官方文档`_ 。


.. _usage:

语法
-------

ffmpeg [`global_options`] {[`input_file_options`] -i input_url} ...{[`output_file_options`] output_url} ...

.. _intro:

简介
-------

**FFmpeg** 是一个快速的视频和音频转换工具，也能够从实时的视频/音频源中抓取流媒体文件。另外，通过一个高质量的多相滤波器，它也可以做到转换视频的采样率和分辨率等功能。

ffmpeg 读取任意数量的输入“文件”（可以是常规的文件，管道，网络流媒体，采集设备等）——通过参数 **-i** 指定，然后写入到任意数量的输出“文件”——通过指定不带任何参数的url。命令行上输入的任何无法被解释为参数的内容，都会被认为是输出地址。

原则上来说，每一个输入\/输出地址，都可以包含任意多的不同格式的流文件（视频\/音频\/字幕\/附件\/数据）。但有时候，允许的流文件数量和\/或类型可能会受具体的容器的格式影响。流文件的输入和输出可以通过 **-map** 选项来指定，如果没有提供的化，将会被自动选择。

你需要通过在命令行中指定其索引（以0开始）来指定被使用的输入文件。举例来说，第一个输入文件的索引是0，第二个是1，依次类推。与此类似，在一个文件中的多个流文件也可以通过他们的索引被引用。比如说，2:3指定了使用第三个输入文件的第四条流文件。可以在官方文档的 `Stream Specifiers`_ 这节中找到更多内容。

总的来说，参数选项被用于输入时紧随其后的文件。因此，输入的顺序尤为重要，也因此，你可以在一条命令中多次输入相同的参数，每个参数也都会在执行时被应用到其后的那个文件中。这条规则仅仅不适用于一些全局选项（例如详细等级（verbosity level）），这种情况下应该将其放在命令的开始位置。

.. note:: 不要混淆输入和输出文件——永远要先指定所有的输入文件，然后再是输出文件。同样，也不要混淆针对不同文件的选项。所有的选项都 **只** 会被用于下一个输入\/输出文件，在处理后续文件时，这些选项会被清空。


.. detailed_description:

工作原理
-----------

对于每一个输出，**FFmpeg** 的转码过程可以以下图描述： ::

     _______              ______________
    |       |            |              |
    | input |  demuxer   | encoded data |   decoder
    | file  | ---------> | packets      | -----+
    |_______|            |______________|      |
                                               v
                                          _________
                                         |         |
                                         | decoded |
                                         | frames  |
                                         |_________|
     ________             ______________       |
    |        |           |              |      |
    | output | <-------- | encoded data | <----+
    | file   |   muxer   | packets      |   encoder
    |________|           |______________|


**FFmpeg** 首先调用 ``libavformat`` 库对输入文件进行解复用（分流）获取编码的数据包（encoded packets）。当存在多个输入文件时，ffmpeg尝试通过追踪所有活动的输入流中最慢的时间戳来保持同步。

接下来，编码的数据包被传入解码器（decoder）中（除非对当前流已选择 `streamcopy` 选项）。解码器产生无压缩的帧（frames，例如raw video\/PCM audio\/...），然后被滤波器（fitering，参考 :ref:`filtering` ）进一步处理。

经滤波器处理后的帧被传入到编码器（encoder），被编码形成数据包，并最终被传入复用器（muxer），以将这些数据包写入到输出文件中。


.. _filtering:

滤波
------------

在编码之前，**FFmpeg** 通过调用 ``libavfilter`` 库的滤波器（filters）来处理原生（raw）音频和视频帧。我们将由多个滤波器（chained filters）组成的链式结构称为滤波图（filter graph）。ffmpeg 中区分两类滤波图：简单的（simple）和复杂的(complex)。

**简单滤波图 Simple filtergraphs**

简单滤波图是指那些有且只有一个并且相同类型的输入和输出的滤波图。这种类型的滤波图可以通过简单地在上图中的解码和编码过程中插入一个额外步骤来表示： ::

     _________                        ______________
    |         |                      |              |
    | decoded |                      | encoded data |
    | frames  |\                   _ | packets      |
    |_________| \                  /||______________|
                 \   __________   /
      simple     _\||          | /  encoder
      filtergraph   | filtered |/
                    | frames   |
                    |__________|

简单滤波图被设置成每一条流（per-stream）一个 **-filter** 选项（ **-vf** 和 **-af** 分别代表视频和音频）。例如，一个视频的简单滤波图可以以下图表示： ::

     _______        _____________        _______        ________
    |       |      |             |      |       |      |        |
    | input | ---> | deinterlace | ---> | scale | ---> | output |
    |_______|      |_____________|      |_______|      |________|


.. note:: 有些滤波器能修改帧的属性而不是内容。比如：上述例子中的 ``fps`` 滤波器会修改帧数但不会修改每帧中的内容。另一个例子是， ``setpts`` 滤波器只会修改每帧的时间戳而对内容保持不变。

**复杂滤波图 Complex filtergraphs**



.. _options:

命令行参数
----------



.. _commands:

常用命令
-----------

* 音频\/视频格式转换： ::

    $ ffmpeg -i INPUT -qscale 0 -acodec codec -ar freq OUTPUT

  * -qscale `q` 采用动态编码率对文件进行编码，q的数值决定了编码质量，取值从0~255，数值越小质量越好；
  * -acodec `codec` 指定了音频的编码格式，例如pcm_s16le指定了采用PCM，16比特位宽，小端的方式编码；
  * -ar `freq` 指定了音频的采样率，例如16000指定了输出文件采用16KHz的采用率。

* 为PCM文件添加头部信息： ::

    $ ffmpeg -f s16le -ar 16000 -ac 1 -i INPUT.pcm OUTPUT.wav

  * -f `fmt` 强制指定输入或输出文件的格式。选项 **s16le -ar 16000 -ac 1** 指定OUTPUT.wav包含的头部信息为，16比特位宽、16KHz采样率、通道数为1。

* 音频\/视频分割： ::

    $ ffmpeg -i INPUT -ss 1.01 -t 2.12 OUTPUT

  * -ss `position` 指定相对于文件开始处的位置；
  * -t `duration` 指定输出文件时长。

* 提取音频流文件： ::

    $ ffmpeg -i INPUT.mp4 -q:a 0 -map a -ar 16000 OUTPUT.wav

  * -q:a `q` 设置音频质量，类似于-qscale；
  * -map a 选取其中的音频流；
  * -ar `freq` -ac `channels` -acodec `codec` 设置产生的输出文件的格式。

* 视频拆帧： ::

    $ ffmpeg -i INPUT.avi -vf fps=N –frames N OUTPUT_%3d.jpg

  * -r `fps=N` 选项告诉ffmpeg 按照每秒N帧来抽取，N可以是小数，例如N为0.2，则每5秒抽取一帧。抽取的帧是该段间隔内最中间的一帧，例如N=0.2，视频帧率是30fps时，那么抽取的第一张将是第75帧；
  * `OUTPUT_%3d.jpg` 定义了生成的图片的命名格式，即OUTPUT_001.jpg，OUTPUT_002.jpg…OUTPUT_999.jpg（根据视频长度决定最后一帧名称）；
  * -frames `N` 指定输出N张后停止输出。

  .. warning:: 不同系统上，当视频中包含丢帧的情况时，具体的处理方式会有所不同。在windows系统上，会跳过丢的那帧使用后续帧来补齐；而在linux系统中，丢的那帧依然会计入其中。假设一段视频共包含100帧，其中第2,3,4,5帧丢失，那在windows平台下全部抽取会产生96张，并且最后一张名称是OUTPUT_096.jpg。而linux平台上会产生100张，但是其中第2-5张是无效的文件。


* 视频合并： ::

    $ ffmpeg -f concat -i filelist.txt -c copy output.mkv

  * -i 指定一个文本，包含所有要合并视频的路径，如下： ::

        file 'input1.mkv'
        file 'input2.mkv'
        file 'input3.mkv'

  .. warning:: 此特性需要 FFmpeg 1.1 以上版本


.. _FFmpeg官方文档: https://ffmpeg.org/ffmpeg.html
.. _Stream Specifiers: https://ffmpeg.org/ffmpeg.html#Stream-specifiers-1
