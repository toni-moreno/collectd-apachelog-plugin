collectd-apachelog-plugin
=========================


Description
--------------

A plugin for collectd which efficiently parses apache log files to get global and extended ( per HTTP CODE)  metrics. It defines a http_perf ( performance ) new type, which defines the new metrics.

__hit_rate__ ( hits /second)
__hit_x_interval__ ( #hits in the collectd interval time=> will be used to count hits across time )
__rt_avg__  ( average response time in the elapsed interval time in "ms" )
__rt_max__ ( max response time in the elapsed interval time in "ms")
__rt_min__ ( min response time in the elapsed interval time in "ms")

In extended mode gather performance statistics  for :

* __global__ : all requests
* __1XX__ : all HTTP 1XX code request
* __2XX__ : all HTTP 2XX code request
* __3XX__ : all HTTP 3XX code request
* __4XX__: all HTTP 4XX code request
* __5XX__: all HTTP 5XX code request


It can parse apache logs and supports for rotatelogs tool ( by checking the last created file with a pattern)



Build the new plugin
====================

You can add to collectd main project by only patching it ( until official pull will be accepted https://github.com/collectd/collectd/pull/576)



* Download the collectd-apachelog-plugin patch

```
# git clone https://github.com/toni-moreno/collectd-apachelog-plugin.git
```

* Download original collectd repository sources

```
# git clone https://github.com/collectd/collectd.git
```

* patch the original collectd source

```
# cd collectd

# cp ~/collectd-apachelog-plugin/apache_log_plugin_v0.1.patch .
# patch -p0 < apache_log_plugin_v0.1.patch
# cp ~/collectd-apachelog-plugin/apache-log.c ./src/

```

You can now rebuild collecd project.

```
# ./build.sh
# ./configure  --enable-apachelog [other_configure_options]
# ./make
# ./make install
```


Configure The Plugin
====================

* Instance: Apache instance name (mandatory)
* RenamePluginAs: (default:none)Used to put apachelog metrics beside de apache metrics in case of organize metrics by product better than by plugin source.
* UseApacheRotatedLogs: (default:false) when true , apachelog plugin tracks always the last modified file with pattern in <File ""> section.  
	See: http://httpd.apache.org/docs/2.2/programs/rotatelogs.html
* ExtendedMetrics: (default:false) when true , apachelog plugin collect global and per http code ( 1XX, 2XX, 3XX, 4XX, 5XX ) performance data.
* SetRespTimeField: ( default: 0/last) set the %D apachelog Field position ( with blanks as field separators).
* SetHTTPCodeField: ( default: 9 ) set where to look for apache status code  (Only used on ExtendedMetrics=true )

	NOTE: the field positions are always specified as:
		0=last
		1=first
		2=second
		..
		N=last(also)

	NOTE2: these positions are the default for the folloging apache logFormat

```
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\" %D" combined
```

* Configuration example File:

```
  LoadPlugin apachelog
  <Plugin apachelog>

    <File "/var/log/apache2/access.log*">  
      Instance "www_misite_com"
      RenamePluginAs "apache"
      UseApacheRotatedLogs "false"
      ExtendedMetrics "true"
      SetRespTimeField 0 
      SetHTTPCodeField  9  
    </File>

    <File "/var/log/apache2/access.log">  //filename Name on a fixed log name
      Instance "www_misite_com"
      RenamePluginAs "apache"
      UseApacheRotatedLogs "false"
    </File>

  </Plugin>

```



Customized configuration for Graphite/Collectd users 
------------------------------------------------------------------

Add these storage-aggregation rules the (/opt/graphite/conf/storage-aggregation.conf) file.

```
[http_perf_hits]
pattern = \.hit_x_interval$
xFilesFactor = 0.5
aggregationMethod = sum

[http_perf_rt_avg]
pattern = \.rt_avg$
xFilesFactor = 0
aggregationMethod = avg

[http_perf_rt_min]
pattern = \.rt_min$
xFilesFactor = 0
aggregationMethod = min

[http_perf_rt_max]
pattern = \.rt_max$
xFilesFactor = 0
aggregationMethod = max
```




