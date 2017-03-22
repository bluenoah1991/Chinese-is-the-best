# TileStache API

TileStache是一个提供地图瓦片服务的Python服务器。你可能熟悉MetaCarta的开源WMS(Web Maps Server我猜的)[TileCache](http://tilecache.org/)。TileStache是类似的，但是我们希望其更简单，并且更适合设计人员和制图人员使用。
##### 这个文档包括TileStache 1.50.0版
也可以看[详细的模块和类参考](http://tilestache.org/doc/TileStache.html)。

----------
## 请求瓦片
#### 通过HTTP
TileStache的URLs是基于类似Google Maps的结构：  
  
`/{layer name}/{zoom}/{column}/{row}.{extension}`  

类似的例子像这样：

`http://example.org/path/tile.cgi/streets/12/656/1582.png`  

对于返回JSON响应的情况(例如通过[Vector provider](http://tilestache.org/doc/#vector-provider))，URLs能包括一个可选的JSONP回调：

`http://example.org/path/tile.cgi/streets/12/656/1582.json?callback=funcname`  

相对的，也可以预览瓦片的slippy-map：

`/{layer name}/preview.html`  

#### 代码方面

#####TileStache.getTile  
对于一个给定的层瓦片请求，提供一个类型字符串和瓦片二进制数据  
getTile的参数：  
**layer**  
要渲染的Core.Layer的实例。  
**coord**  
一个ModestMaps.Core.Coordinate对应到一个单独的瓦片。  
**ignore_cached**  
可选布尔型：无论是否有缓存，都一律重新渲染瓦片，默认值False。  

getTile的返回值是一个元组，包含一个类似于"image/png"这样的mime-type字符串，和被渲染瓦片的完整二进制字符串数据。  
查看[TileStache.getTile](http://tilestache.org/doc/TileStache.html#-getTile)了解更多信息。
#####TileStache.requestHandler  
为一个给定请求生成一个mime-type和一个响应body体。这个函数用于为TileStache创建新的HTTP接口。  
requestHandler的参数：  
**config**  
必选，是一个JSON配置文件的路径字符串，或者是一个包含cache，layers和dirpath属性的配置对象，例如[TileStache.Config.Configuration](http://tilestache.org/doc/TileStache.Config.html#Configuration)  
**path_info**  
必选，一个请求URL的尾部，包含层名和瓦片坐标，例如  
`"/roads/12/656/1582.png"`  
**query_string**  
可选，查询字符串。当前仅被用于JSONP回调的场景。  
**script_name**  
可选，对应CGI环境变量*SCRIPT_NAME*的脚本名称，被用于正确计算302跳转。  

requestHandler的返回值是一个元组，包含一个类似于"image/png"这样的mime-type字符串，和被渲染瓦片的完整二进制字符串数据。  
查看[TileStache.requestHandler](http://tilestache.org/doc/TileStache.html#-requestHandler)了解更多信息。

## 瓦片服务器  
我们当前为瓦片服务器提供3个脚本：一个是提供给基于WSGI的web服务器，一个是提供给基于CGI的web服务器，还有一个是为Apache的mod_python模块提供的。
##### WSGI
TileStache自身可以作为WSGI应用程序运行，并且使用一个[Werkzeug](http://werkzeug.pocoo.org/)服务器。要使用这个内置服务器，运行tilestache-server.py，它默认会在当前目录下查找一个叫tilestache.cfg的配置文件，并且在http://127.0.0.1:8080/上提供瓦片服务。通过查看tilestache-server.py --help来改变这些默认值。

或者，任意一款可以指向TileStache.WSGITileServer实例的WSGI服务器。下面是如何与[gunicorn](http://gunicorn.org/)服务器一起使用：  
`$ gunicorn "TileStache:WSGITileServer('/path/to/tilestache.cfg')"`  
同样的配置可以像下面这样使用[uWSGI](http://projects.unbit.it/uwsgi/)提供服务：  

    $ uwsgi --http :8080 --eval 'import TileStache; \  
    application = TileStache.WSGITileServer("/path/to/tilestache.cfg")'

查看[TileStache.WSGITileServer](http://tilestache.org/doc/TileStache.html#WSGITileServer)了解更多信息。
##### CGI
通过CGI为TileStache提供最基本的瓦片服务支持，以便用于简单的测试和中低流量的网站。这是一个完整的可工作的CGI脚本，通过从本地查找一个叫tilestache.cfg的配置文件：  

    #!/usr/bin/python  
    import os, TileStache  
    TileStache.cgiHandler(os.environ, 'tilestache.cfg', debug=True)  

查看[TileStache.cgiHandler](http://tilestache.org/doc/TileStache.html#-cgiHandler)了解更多信息。
##### mod_python
使用基于mod_python的TileStache缓存被导入的模块提升性能，但是必须对Apache服务器进行配置。这是一个完整的基于web服务器发布瓦片的例子，基于/etc下面的配置文件：

	<Directory /var/www/tiles>
	  AddHandler mod_python .py
	  PythonHandler TileStache::modpythonHandler
	  PythonOption config /etc/tilestache.cfg
	</Directory>

查看[TileStache.modPythonHandler](http://tilestache.org/doc/TileStache.html#-modPythonHandler)了解更多信息。
## 配置TileStache
TileStache的配置文件是一个JSON文件，并且由2个主要的顶级节点*"cache"*和*"layers"*组成。下面是最小配置的例子：

	{
	  "cache": {"name": "Test"},
	  "layers": {
	    "ex": {
	        "provider": {"name": "mapnik", "mapfile": "style.xml"},
	        "projection": "spherical mercator"
	    } 
	  }
	}

### Caches
Cache在TileStache中是用于存储静态文件提速未来的请求。一些默认的caches展示在这里，额外的cache类定义在[TileStache.Goodies.Caches](http://tilestache.org/doc/TileStache.Goodies.Caches.html)。  
跳转到[Test](http://tilestache.org/doc/#test-cache)，[Disk](http://tilestache.org/doc/#disk-cache)，[Multi](http://tilestache.org/doc/#multi-cache)，[Memcache](http://tilestache.org/doc/#memcache-cache)，[Redis](http://tilestache.org/doc/#redis-cache)，或者[S3](http://tilestache.org/doc/#s3-cache).  
#####Test
最简单的cache实际上什么都不缓存。
尽管活动日志是可选的。
样例配置：

	{
	  "cache": {
	    "name": "Test",
	    "verbose": true
	  },
	  "layers": { … }
	}

Test的cache参数：  
**verbose**  
可选，布尔值标识是否写缓存活动到一个日志函数，如果省略的话默认是False。
查看[TileStache.Caches.Test](http://tilestache.org/doc/TileStache.Caches.html#Test)文档了解更多信息。
#####Disk
