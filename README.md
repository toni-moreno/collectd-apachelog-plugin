collectd-apachelog-plugin
=========================

A plugin for collectd which efficiently parses apache log files to get global and extended ( per HTTP CODE) hits count and response time it also supports the rotatelogs tools

You can add to collectd main project by only patching and 


Build the new plugin
====================

* Download the collectd-apachelog-plugin pathc

```
 git clone https://github.com/toni-moreno/collectd-apachelog-plugin.git
```

* Download original collectd repository sources

```
git clone https://github.com/collectd/collectd.git
```

* patch the original collectd source

```
 cd collectd

 mv ~/collectd-apachelog-plugin/apache_log_plugin_v0.1.patch .
 patch -p0 < apache_log_plugin_v0.1.patch
 mv ~/collectd-apachelog-plugin/apache-log.c ./src/
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







