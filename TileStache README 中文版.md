# TileStache

_一种很酷炫的缓存地图瓦片的方式_


**TileStache**是一个基于Python的服务器应用程序，能够基于渲染地理信息数据提供地图瓦片服务。你可能熟悉来自MetaCarta的Web地图服务器TileCache。TileStache与其很类似，但是我们希望它更简单并且对于设计人员和制图人员更好用。

## 概要

    import TileStache
    import ModestMaps

    config = {
      "cache": {"name": "Test"},
      "layers": {
        "example": {
            "provider": {"name": "mapnik", "mapfile": "examples/style.xml"},
            "projection": "spherical mercator"
        }
      }
    }

    # like http://tile.openstreetmap.org/1/0/0.png
    coord = ModestMaps.Core.Coordinate(0, 0, 1)
    config = TileStache.Config.buildConfiguration(config)
    type, bytes = TileStache.getTile(config.layers['example'], coord, 'png')

    open('tile.png', 'w').write(bytes)



## 相关依赖

### 必须要有:

- ModestMaps: http://modestmaps.com, http://github.com/migurski/modestmaps-py
- Python图片处理库 (Pillow): https://python-pillow.org

### 可选的:

- Simplejson: https://github.com/simplejson/simplejson (可选的 if using >= python 2.6)
- mapnik: http://mapnik.org (可选的)
- werkzeug: http://werkzeug.pocoo.org/ (可选的)

使用pip（是python的包管理工具，类似于java的maven）安装纯python模块:

    sudo pip install -U python-pil modestmaps simplejson werkzeug uuid

像这样安装pip (http://www.pip-installer.org/)（其实不用，ubuntu上直接sudo apt-get install python-pip -y）:

    curl -O -L https://raw.github.com/pypa/pip/master/contrib/get-pip.py
    sudo python get-pip.py

按照下面的介绍安装mapnik:

    http://mapnik.org/pages/downloads.html


## 安装

TileStache可以直接从下载目录运行，比如像下面这些脚本:

    tilestache-render.py tilestache-seed.py tilestache-server.py

能够像这样在本地运行:

    ./scripts/tilestache-server.py

可以这样进行全局安装:

    python setup.py install

  * 注意: 你可能需要在命令前面加上sudo以获取完整权限进行安装。


## 快速开始

让TileStache开始工作在一个开发服务器上:

    ./scripts/tilestache-server.py

然后打开一个现代浏览器，你应该可以在下面地址浏览瓦片:

    http://localhost:8080/osm/preview.html


这是一个预览，通过默认配置文件tilestache.cfg中的配置使用来自http://tile.osm.org的ModestMaps和OpenStreetMap瓦片


## 文档

下一步是学习怎样构建自定义层以及如何使用它们提供服务。

详细阅读[这个文档](http://tilestache.org/doc/)。


## 特性

渲染Providers:
* Mapnik
* Proxy
* Vector
* 模版URLs

缓存后端:
* 本地磁盘
* Test
* Memcache
* S3（这是Amazon S3）


## 设计目标

TileStache的设计焦点在于易于使用，从而牺牲了自由度和完整性。我们希望任何一个人都可以简单的使用它为他们的城市构建一张地图，发布一个新的世界视窗，或者甚至构建下一个8-Bit NYC（http://8bitnyc.com）。

* 小巧

TileStache的核心试图做的尽量精简。基于像CGI脚本这样的入口点，人们应该能够简单快速的了解什么库是干什么的，并且为什么是这样。在可能的情况下，尽量不要使用动态编程这种魔法，而是使用最简单，流程化，以及拥有大量文档的Python编写。

* 可插拔的

我们想要从TileStache外部接受插件和扩展，并将TileStache自身作为一个扩展提供给其他系统。它必须能够在不修改核心包的情况下编写和使用额外的caches或者renderers，可以从包内部扩展类，或者导航类的依赖关系。鸭子类型（弱接口类型）和稳定的接口是最好的了。

* 明确的默认值

一个TileStache实例的默认配置动作应该允许大多数主流交互形式：一个世界范围的球形莫卡托左上角瓦片布局兼容OpenStreetMap，Google，Bing Maps，Yahoo!和其他等等。让TileStache支持所有的扩展系统应该是可能的，但是我们降低复杂度，遵循标准，这样对于基本的web客户更务实和便捷。


## 许可证

BSD, 请看LICENSE文件.
