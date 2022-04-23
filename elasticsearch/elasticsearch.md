# elasticsearch

#### 数据类型

1. 结构化数据（表格）

2. 非结构化数据（视频）

3. 半结构化数据（xml）es解决的查询内容

学习自

https://blog.csdn.net/smilehappiness/article/details/118466378

## 安装

1. 获取es安装包

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.2-linux-x86_64.tar.gz
```

2. 执行解压命令

```shell
tar -zxvf elasticsearch-7.13.2-linux-x86_64.tar.gz -C /usr/local
```

## 解决es强依赖jdk问题

由于es和jdk是一个强依赖的关系，所以当我们在新版本的ElasticSearch压缩包中包含有自带的jdk，但是当我们的Linux中已经安装了jdk之后，就会发现启动es的时候优先去找的是Linux中已经装好的jdk，此时如果jdk的版本不一致，就会造成jdk不能正常运行，报错如下：

注：如果Linux服务本来没有配置jdk，则会直接使用es目录下默认的jdk，反而不会报错

```shell
warning: usage of JAVA_HOME is deprecated, use ES_JAVA_HOME
Future versions of Elasticsearch will require Java 11; your Java version from [/usr/local/jdk1.8.0_291/jre] does not meet this requirement. Consider switching to a distribution of Elasticsearch with a bundled JDK. If you are already using a distribution with a bundled JDK, ensure the JAVA_HOME environment variable is not set.

```

**解决办法：**

- 进入bin目录  
  `cd /usr/local/elasticsearch-7.13.2/bin`

- 修改elasticsearch配置  
  `vim ./elasticsearch`

```shell
############# 添加配置解决jdk版本问题 ##############
# 将jdk修改为es中自带jdk的配置目录
export JAVA_HOME=/usr/local/elasticsearch-7.13.2/jdk
export PATH=$JAVA_HOME/bin:$PATH

if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/usr/local/elasticsearch-7.13.2/jdk/bin/java"
else
        JAVA=`which java`
fi


```



修改后结果为

```shell
#!/bin/bash

# CONTROLLING STARTUP:
#
# This script relies on a few environment variables to determine startup
# behavior, those variables are:
#
#   ES_PATH_CONF -- Path to config directory
#   ES_JAVA_OPTS -- External Java Opts on top of the defaults set
#
# Optionally, exact memory values can be set using the `ES_JAVA_OPTS`. Example
# values are "512m", and "10g".
#
#   ES_JAVA_OPTS="-Xms8g -Xmx8g" ./bin/elasticsearch
#=======添加配置解决jdk版本问题=====
export JAVA_HOME=/usr/local/elasticsearch-7.13.2/jdk     # （将原目录修改为es中自带jdk的配置目录）
export PATH=$JAVA_HOME/bin:$PATH


source "`dirname "$0"`"/elasticsearch-env

CHECK_KEYSTORE=true
DAEMONIZE=false
for option in "$@"; do
  case "$option" in
    -h|--help|-V|--version)
      CHECK_KEYSTORE=false
      ;;
    -d|--daemonize)
      DAEMONIZE=true
      ;;
  esac
done
if [ -z "$ES_TMPDIR" ]; then
  ES_TMPDIR=`"$JAVA" "$XSHARE" -cp "$ES_CLASSPATH" org.elasticsearch.tools.launchers.TempDirectory`
fi

# get keystore password before setting java options to avoid
# conflicting GC configurations for the keystore tools
unset KEYSTORE_PASSWORD
KEYSTORE_PASSWORD=
if [[ $CHECK_KEYSTORE = true ]] \
    && bin/elasticsearch-keystore has-passwd --silent
then
  if ! read -s -r -p "Elasticsearch keystore password: " KEYSTORE_PASSWORD ; then
    echo "Failed to read keystore password on console" 1>&2
    exit 1
  fi
fi

# The JVM options parser produces the final JVM options to start Elasticsearch.
# It does this by incorporating JVM options in the following way:
#   - first, system JVM options are applied (these are hardcoded options in the
#     parser)
#   - second, JVM options are read from jvm.options and jvm.options.d/*.options
#   - third, JVM options from ES_JAVA_OPTS are applied
#   - fourth, ergonomic JVM options are applied
ES_JAVA_OPTS=`export ES_TMPDIR; "$JAVA" "$XSHARE" -cp "$ES_CLASSPATH" org.elasticsearch.tools.launchers.JvmOptionsParser "$ES_PATH_CONF" "$ES_HOME/plugins"`

#=======添加配置解决jdk版本问题=====

if [ -x "$JAVA_HOME/bin/java" ]; then
        JAVA="/usr/local/elasticsearch-7.13.2/jdk/bin/java"
else
        JAVA=`which java`
fi
#============

# manual parsing to find out, if process should be detached
if [[ $DAEMONIZE = false ]]; then
  exec \
    "$JAVA" \
    "$XSHARE" \
    $ES_JAVA_OPTS \
    -Des.path.home="$ES_HOME" \
    -Des.path.conf="$ES_PATH_CONF" \
    -Des.distribution.flavor="$ES_DISTRIBUTION_FLAVOR" \
    -Des.distribution.type="$ES_DISTRIBUTION_TYPE" \
    -Des.bundled_jdk="$ES_BUNDLED_JDK" \
    -cp "$ES_CLASSPATH" \
    org.elasticsearch.bootstrap.Elasticsearch \
    "$@" <<<"$KEYSTORE_PASSWORD"
else
  exec \
    "$JAVA" \
    "$XSHARE" \
    $ES_JAVA_OPTS \
    -Des.path.home="$ES_HOME" \
    -Des.path.conf="$ES_PATH_CONF" \
    -Des.distribution.flavor="$ES_DISTRIBUTION_FLAVOR" \
    -Des.distribution.type="$ES_DISTRIBUTION_TYPE" \
    -Des.bundled_jdk="$ES_BUNDLED_JDK" \
    -cp "$ES_CLASSPATH" \
    org.elasticsearch.bootstrap.Elasticsearch \
    "$@" \
    <<<"$KEYSTORE_PASSWORD" &
  retval=$?
  pid=$!
  [ $retval -eq 0 ] || exit $retval
  if [ ! -z "$ES_STARTUP_SLEEP_TIME" ]; then
    sleep $ES_STARTUP_SLEEP_TIME
  fi
  if ! ps -p $pid > /dev/null ; then
    exit 1
  fi
  exit 0
fi
exit $?
~            oded options in the
#     parser)  



```

## 解决内存不足问题

由于 elasticsearch 默认分配 jvm空间大小为2g，修改 jvm空间，如果Linux服务器本来配置就很高，可以不用修改。

```shell
error:
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c6a00000, 962592768, 0) failed; error='Not enough space' (errno=12)
        at org.elasticsearch.tools.launchers.JvmOption.flagsFinal(JvmOption.java:119)
        at org.elasticsearch.tools.launchers.JvmOption.findFinalOptions(JvmOption.java:81)
        at org.elasticsearch.tools.launchers.JvmErgonomics.choose(JvmErgonomics.java:38)
        at org.elasticsearch.tools.launchers.JvmOptionsParser.jvmOptions(JvmOptionsParser.java:13

```

**进入config文件夹开始配置，编辑jvm.options：**  
`vim /usr/local/elasticsearch-7.13.2/config/jvm.options`

```shell
默认配置如下：
-Xms2g
-Xmx2g
默认的配置占用内存太多了，调小一些：
-Xms256m
-Xmx256m

```

## 创建专用用户启动ES

root用户不能直接启动Elasticsearch，所以需要创建一个专用用户，来启动ES

```shell
java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:101)
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:168)
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:397)
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159)
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150)
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:75)
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:116)
        at org.elasticsearch.cli.Command.main(Command.java:79)
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:115)
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:81)

```

创建用户
`useradd user-es`

创建所属组：
`chown user-es:user-es -R /usr/local/elasticsearch-7.13.2`

切换到user-es用户
`su user-es`

进入bin目录
`cd /usr/local/elasticsearch-7.13.2/bin`

启动elasticsearch
`./elasticsearch`

## 修改ES核心配置信息

执行命令修改elasticsearch.yml文件内容  
`vim /usr/local/elasticsearch-7.13.2/config/elasticsearch.yml`

修改绑定的ip允许远程访问

```shell
#默认只允许本机访问，修改为0.0.0.0后则可以远程访问
# 绑定到0.0.0.0，允许任何ip来访问
network.host: 0.0.0.0 

```

初始化节点名称

```shell
cluster.name: elasticsearch 
node.name: es-node0
cluster.initial_master_nodes: ["es-node0"]

```

## vm.max_map_count [65530] is too low问题

出现

```shell
ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
bootstrap check failure [1] of [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```

**elasticsearch用户拥有的内存权限太小，至少需要262144，解决办法：**  
在 `/etc/sysctl.conf` 文件最后添加如下内容，即可永久修改

切换到root用户  
`执行命令：su root`

执行命令  
`vim /etc/sysctl.conf`

添加如下内容  
`vm.max_map_count=262144`

保存退出，刷新配置文件  
`sysctl -p`

切换user-es用户，继续启动  
`su user-es`

启动es服务  
`/usr/local/elasticsearch-7.13.2/bin/elasticsearch`

## ES服务的启动与停止

前台运行，Ctrl + C 则程序终止  
`/usr/local/elasticsearch-7.13.2/bin/elasticsearch`

后台运行  
`/usr/local/elasticsearch-7.13.2/bin/elasticsearch -d`  
出现started时启动完成

关闭ES服务  
`kill pid`

**说明：**  
Elasticsearch端口9300、9200，其中：  
`9300是tcp通讯端口，集群ES节点之间通讯使用，9200是http协议的RESTful接口`

## 使用

启动后使用postman进行操作

![](C:\Users\zjw\AppData\Roaming\marktext\images\2022-04-22-09-46-22-image.png)

##### 创建index

put：http://101.43.155.248:9200/shopping

成功返回

```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "shopping2"
}
```

##### 获取某个index信息

get: http://101.43.155.248:9200/shopping

返回

```json
{
    "shopping": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "shopping",
                "creation_date": "1650591766042",
                "number_of_replicas": "1",
                "uuid": "uxcXblqqQ9uvAbycFfe57g",
                "version": {
                    "created": "7130299"
                }
            }
        }
    }
}
```

##### 获取全部index信息

get:http://101.43.155.248:9200/_cat/indices?v

返回

```json
health status index     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   shopping2 ha96_dVOTeuw6XsbqKlrmQ   1   1          0            0       208b           208b
yellow open   shopping  uxcXblqqQ9uvAbycFfe57g   1   1          0            0       208b           208b

```

##### 删除某一索引

delete：http://101.43.155.248:9200/shopping

```json
{
 "acknowledged": true
}
```

##### 添加type

post:http://101.43.155.248:9200/shopping/_doc

body中加json数据

```json
{
	"title":"小米手机",
	"category":"小米",
	"price":3999.00
}
```

返回

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "-N75ToAB0U04MuS2yENg",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

post:http://101.43.155.248:9200/shopping/_doc/1001 

该方法可以将数据的id设置为1001

##### 获取index的所有数据

get： http://101.43.155.248:9200/shopping/_search

返回

```json
{
    "took": 883,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1,
        "hits": [
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "-N75ToAB0U04MuS2yENg",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米",
                    "price": 3999
                }
            },
            {
                "_index": "shopping",
                "_type": "_doc",
                "_id": "-d77ToAB0U04MuS2wEO1",
                "_score": 1,
                "_source": {
                    "title": "小米手机",
                    "category": "小米"
                }
            }
        ]
    }
}
```

##### 修改数据

put: http://101.43.155.248:9200/shopping/doc/N75ToAB0U04MuS2yENg

body:

```json
{
	"title": "华为手机",
    "category": "华为",
    "price": 3999
}
```

返回

```json
{
    "_index": "shopping",
    "_type": "_doc",
    "_id": "N75ToAB0U04MuS2yENg",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```


