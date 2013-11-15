下载Solr后
 
1. 单机模式启动

**启动应用**

到example目录下

    java -jar start.jar

这样会启动一个8983端口的jetty应用，访问[http://localhost:8983/solr](http://localhost:8983/solr)

**创建索引**

如果要向Solr添加数据进行索引，到example/exampledocs目录下，执行
    
    java -jar post.jar solr.xml monitor.xml

这样会把这两个xml文件post给Solr进行索引，当然也可以

    java -jar post.jar *.xml

把该目录下所有的xml文件post过去进行索引

**查询**

有了索引就可以进行查询了：

*可以直接以查询关键字*：[http://localhost:8983/solr/#/collection1/query?q=video](http://localhost:8983/solr/#/collection1/query?q=video)

*可以指定field查询*：[http://localhost:8983/solr/#/collection1/query?q=name:video](http://localhost:8983/solr/#/collection1/query?q=name:video)

*可以组合多个条件查询*：[http://localhost:8983/solr/#/collection1/query?q=%2Bvideo%20%2Bprice%3A[*%20TO%20400]](http://localhost:8983/solr/#/collection1/query?q=%2Bvideo%20%2Bprice%3A[*%20TO%20400])

> 只索引solr.xml, monitor.xml，上面的查询是没有结果的，因为这两个文档中没有video



**更新索引数据**

如果执行了上面两条，那solr.xml, monitor.xml会被重复提交，但你查询solr仍然只会得到一条结果，这是因为在schema.xml`因为我们现在在example目录下跑solr，这个文件在example\solr\collection1\conf目录下`里指定了id作为唯一键，这样，当你提交了一个唯一键值与之前提交的文档的唯一键相同的文档时，solr会自动替换原来的文档. 你可以通过统计页面的“CORE”/searcher 部分numDocs, maxDoc的值看到这一点，[http://localhost:8983/solr/#/collection1/plugins/core?entry=searcher](http://localhost:8983/solr/#/collection1/plugins/core?entry=searcher)

**删除数据**

通过向更新数据URL发送一个删除指令，同时指定要删除的文档的唯一键field，可都通过查询指定多个文档即可. 指令可以放在xml文件中，如果比较小也可以直接在命令行中作为参数带上

    java -Ddata=args -Dcommit=false -jar post.jar "<delete><id>SP2514N</id></delete>"

因为我们在这里指定了commit=false，那么再次搜索[id:SP2514N](http://localhost:8983/solr/#/collection1/query?q=id:SP2514N)，仍然可以查询到结果. 因为example的配置使用了solr的autoCommit功能，solr仍会自动的持久化索引的变动，但这并不会影响搜索的结果，直到一个“openSearcher”的commit显式地执行

通过updateHandler的[统计页面](http://localhost:8983/solr/#/collection1/plugins/updatehandler?entry=updateHandler)，通过查看deletesById的值降到零而cumulative_deletesById和autocommit的值增加可以观察到这个删除动作已经同步到硬盘上了

下面是一个通过使用delete-by-query来删除name中有DDR的文档的命令

    java -Dcommit=false -Ddata=args -jar post.jar "<delete><query>name:DDR</query></delete>"

通过发送一个显式执行commit命令即强制使用一个新的searcher来反应这些改动

    java -jar post.jar -

再次执行前面的查询，应该已经看不到结果了，你也可以到updateHandler的统计页面去查看那些值看一下

为了下面的查询，还是把所有文档都再次添加进来吧
  
    java -jar post.jar *.xml

2. 集群模式启动