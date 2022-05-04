# elasticsearch 对text类型数据进行排序的方法

需要对待排序的设置keyword



1.  新建名字为metadata的index

        PUT    IP:9200/metadata    

2. 设置文档
   
   POST    IP:9200/metadata/objects
   
   ```json
   {
   	"mappings":{
   		"objects":{
   			"properties":{
                   "name": {
                       "type": "text",
                       "fields": {
                         "keyword": {
                           "type": "keyword"
                         }
                       }
                     },
   				"version":{
   					"type":"integer"
   				},
   				"size":{
   					"type":"integer"
   				},
   				"hash":{
   					"type":"string"
   				}
   			}	
   		}
   	}
   }
   ```

    这里需要使用name字段进行排序，所以为他设置keyword，即添加如下内容

    

```json
  "fields": {
       "keyword": {
            "type": "keyword"
       }
```

3. 插入数据

    POST     IP:9200/metadata/objects/test3_1?op_type=create

```json
{
  "name":"test3",
  "version":1,
  "size":13,
  "hash":"2oUvHeq7jQ27Va2y/usI1kSX4cETY9LuevZU9RT+Fuc
="
}
```



4. 使用name排序查询
   
   GET    ip:9200/metadata/_search?sort=name.keyword
   
   这里注意是要用name.keyword排序
