---
layout: post
title: Logstash介绍 - 日志查询、分析工具
date: 2015-04-01 15:44:10
category: "logstash"
---

在工作中，经常需要查询日志，通常需要在后台用命令查询日志，比较低效，尤其是对部署在很多服务器的分布式服务来说，非常不便。

我推荐使用Logstash + Kibana + Elasticsearch搭建日志服务，可以非常方便地查询、分析日志。


### 安装相应组件


Logstash 　　　　　- 收集和解析日志，并发送给elasticsearch。
Elasticsearch      - 存储日志。
Kibanna 　　　　　 - 日志服务的前端。


{% highlight bash %}
#Logstash 1.4.2
wget https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz


#Kibana 4.0.1
wget https://download.elasticsearch.org/kibana/kibana/kibana-4.0.1-linux-x64.tar.gz

#Elasticsearch 1.5.0
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.5.0.tar.gz
{% endhighlight %}


### 配置

主要需要配置Logstash

{% highlight bash %}
input {
  file {
    path => "/tmp/input.txt"
    type => "test"
  }
}

output {
  stdout { }
  elasticsearch {
    cluster => "elasticsearch"
  }
}
{% endhighlight %}

input，标示日志的来源，这里采用了file模式，读取"/tmp/input.txt"这个文件。

logstash支持很多来源接口，例如file,redis, rabbitmq等等，具体请参考http://logstash.net/docs/1.4.2/

output，表示日志需要存储到哪里，这里采用了elasticsearch。


### 启动服务

{% highlight bash %}
./bin/elasticsearch

./bin/kibana 

./bin/logstash --verbose -f ./test.conf
{% endhighlight %}
　
向测试文件中写一些数据试一试。

{% highlight bash %}
echo test1 >> /tmp/input.txt 
echo test2 >> /tmp/input.txt 
echo test3 >> /tmp/input.txt 
{% endhighlight %}

访问[http://localhost:5601](http://localhost:5601){:target="_blank"}
![Kibanan parse tree]({{ site.url }}/images/assets/logstash.png)


这样，一个最简单的日志服务建立起来了。

Logstash的功能非常强大，这里仅仅展示了最简单的功能，有兴趣的话，可以进一步参考
[Logstash文档](http://logstash.net/docs/1.4.2/){:target="_blank"}

   





