### 安装

> elasticsearch:

```properties
安装：
     1 解压，
     2 bin
     3 启动
目录：
     bin 启动文件
     config 配置文件
         log4j2 日志配置文件
         jvm.options java 虚拟机相关的配置
         elasticsearch.yml elasticsearch 的配置文件！ 默认 9200 端口！ 跨域！
         lib 相关jar包
         logs 日志！
         modules 功能模块
         plugins 插件！
```

> 安装可视化插件：

```properties
安装：
          1 elasticsearch-head-master，解压
          2 npm install npm run start ， 需要node环境
          3 解决跨域问题，elasticsearch->config->elasticsearch.yml配置：
             http.cors.enabled: true
             http.cors.allow-origin: "*"
```

> Kibana:

```properties
 1、解压后端的目录
 2、启动
 3、访问测试
 4、开发工具！ （Post、curl、head、谷歌浏览器插件测试！）
 5、汉化,i18n.locale: "zh-CN"
```

### ES核心概念

> ![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203156.png)

![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203203.png)

## IK分词器插件(可以添加自己的词典)

### 介绍

```properties
安装：
	1、https://github.com/medcl/elasticsearch-analysis-ik
	2、下载完毕之后，放入到我们的elasticsearch 插件,plugins目录
	3、重启观察ES，可以看到ik分词器被加载了！
	4、elasticsearch-plugin 可以通过这个命令来查看加载进来的插件
	5、使用kibana测试！
其他：
	ik_smart 和 ik_max_word，其中 ik_smart 为最少切分，ik_max_word为最细 粒度划分！
	config/IKAnalyzer.cfg.xml  添加加载分词规则，main.dic默认加载 (必定添加main)
```

| method  | url地址                                          | 描述                   |
| ------- | ------------------------------------------------ | ---------------------- |
| PUT     | localhost:9200/索引名称/类型名称/文档id          | 创建文档（指定文档id） |
| POST    | localhost:9200/索引名称/类型名称                 | 创建文档（随机文档id） |
| _update | llocalhost:9200/索引名称/类型名称/文档id/_update | 修改文档               |
| DELETE  | localhost:9200/索引名称/类型名称/文档id          | 删除文档               |
| GET     | localhost:9200/索引名称/类型名称/文档id          | 查询文档通过文档id     |
| POST    | localhost:9200/索引名称/类型名称/_search         | 查询所有数据           |



### 构建自己的字典

使得狂神说 成为一个词。
5.1新增一个 “cyx.dic”
![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203222.png)![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203237.png)

###  把自己的dic 添加到配置中

![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203302.png)![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203309.png)

### 重启 elasticsearch和kibana

加载了编写的dic
![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203317.png)
重新编译，这两个都把狂神说合在一起作为一个词



## api操作

### 添加

```java
PUT /zgc/user/1    //此处会自动帮我们封装好每个field对于的类型  不需要我们指定类型  比如age就默认封装为long类型
{
"name": "狂神说java",
"age" : 10,
"desc": "还行",
"tags": ["技术宅","帅哥","渣男"]
}
```

### PUT更新

**空的字段会覆盖，不推荐**

```java
PUT /zgc/user/1
{
"name": "狂神说123",
"desc": "一顿操作猛如虎，一看工资2500",
"tags": ["技术宅","温暖","直男"]
}
```

### POST更新

**灵活，推荐**

```java
POST /zgc/user/1/_update
{
  "doc":{
    "name": "狂神说222222"
  }
}
```

### 简单查询

```java
GET /zgc/user/1  //查询zgc中id=1的文档

GET /test  //查询test库的信息
```

### 删除索引

```java
DELETE /test  //删除test库
```



#### 1. 分词并且搜索词拆分

```java
// 查询文档1、2
POST /zgc/user/_search?q=name:狂神说java
```

#### 2. 分词并且搜索词拆分

```java
GET /zgc/user/_search
{
  "query": {
    "match": {
      "name": "狂神说"
    }
  }
}
```

### _source指定字段,sort排序

```java
GET /zgc/user/_search
{
  "query": {
    "match": {
      "name": "狂神说"
    }
  }
  , "_source": ["name","tags","age"]
  ,"sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
  ,"from": 0
  ,"size": 2
}
```

### must （and），should （or），must_not （not）

```java
GET /zgc/user/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "name": "java"
          }
        },
        {
          "match": {
            "age": "10"
          }
        }
      ]
    }
  }
}
```

### 匹配多个值 or并且搜索词拆分

```java
GET /zgc/user/_search
{
  "query": {
    "match": {
      "tags": "男  帅哥"
    }
  }
}
```

### term ”不能把搜索词拆分“(如果该字段的属性是keyword的话，但是如果为text，是可以调用分词器进行拆分后进行查找)

```java
GET /zgc/user/_search
{
  "query": {
    "term": {
      "name": "狂"
    }
  }
}
```

### filter:过滤器

```java
# query->bool->filter
# filter比query快的原因：
# 1 query:会先比较查询条件，然后计算分值，最后返回文档结果；filter: 1 对结果进行缓存 2 避免计算分值
GET /zgc/user/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "age": {
            "gte": 10,
            "lte": 200
          }
        }
      }
    }
    
  }
}
```

### 高亮查询

```java
GET /zgc/user/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  }
  ,"highlight": {
    "fields": {
      "name":{}
    }
  }
}
```

### 自定义高亮查询

```java
GET /zgc/user/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  }
  ,"highlight": {
    "fields": {
      "name":{}
    }
    ,"pre_tags": "<p class='key' style='color:red' >"
    ,"post_tags": "</p>"
  }
}
```



### <font color=red>term和match</font>

- term是精确查询，也就是会根据字段的类型（比如说name:text,desc:keyword）

那么使用term查询name的时候就会调用分词器解析后对name进行查询，但是对于desc查询的时候就会想desc的内容当作整体，不使用分词器进行分词查询，而是作为整体查询

- match

  使用分词器进行查询，不管字段的类型时text还是keyword都使用分词器解析之后进行查询



### 集成SpringBoot

https://www.elastic.co/guide/en/elasticsearch/client/index.html
![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203327.png)
![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203336.png)

### maven

<elasticsearch.version>7.6.1</elasticsearch.version>

### springboot配置

```java
// 新建ElasticSearchClientConfig文件
@Configuration
public class ElasticSearchClientConfig {

    @Bean
    public RestHighLevelClient restHighLevelClient() {

        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("127.0.0.1", 9200, "http")
                )
        );
        return client;
    }
}
```

包，搜autoconfigure->搜elasticearch->ElasticsearchRestClientAutoConfiguration

@Import({RestClientBuilderConfiguration.class, RestHighLevelClientConfiguration.class, RestClientFallbackConfiguration.class})

### 索引操作

```java
@Autowired
private RestHighLevelClient restHighLevelClient;

// 创建索引
@Test
void createIndex() throws IOException {
	CreateIndexRequest createIndexRequest = new CreateIndexRequest("zgc_index");
	CreateIndexResponse createIndexResponse = restHighLevelClient.indices().create(createIndexRequest, RequestOptions.DEFAULT);
	System.out.println(createIndexResponse);
}
//org.elasticsearch.client.indices.CreateIndexResponse@998a69c8

// 判断索引是否存在
@Test
void existIndex() throws IOException {
	GetIndexRequest getIndexRequest = new GetIndexRequest("zgc_index");
	boolean exists = restHighLevelClient.indices().exists(getIndexRequest, RequestOptions.DEFAULT);
	System.out.println(exists);
}
//true

// 删除索引
@Test
void delIndex() throws IOException {
	DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("zgc_index");
	AcknowledgedResponse delete = restHighLevelClient.indices().delete(deleteIndexRequest, RequestOptions.DEFAULT);
	System.out.println(delete);
	System.out.println(delete.isAcknowledged());
}
//org.elasticsearch.action.support.master.AcknowledgedResponse@4ee
//true
```

### 文档操作

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}


//测试类
@SpringBootTest
class ElasticsearchApplicationTests {

    @Autowired
    @Qualifier("restHighLevelClient")  //指明此处是获取的我们自己配置的RestHighLevelClient对象
    RestHighLevelClient restHighLevelClient;

    //测试索引的创建
    @Test
    void contextLoads() throws IOException {
        //获取创建索引的请求
        CreateIndexRequest request = new CreateIndexRequest("cooper_index");
        //客户端执行请求 IndicesClient 请求后获得响应
        CreateIndexResponse response = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
        System.out.println(response);
    }

    //测试获取索引
    @Test
    public void testGetIndex() throws IOException {
        GetIndexRequest request = new GetIndexRequest("cooper_index");
        boolean exists = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }

    //测试删除索引
    @Test
    public void deleteIndex() throws IOException {
        DeleteIndexRequest request = new DeleteIndexRequest("cooper_index");
        AcknowledgedResponse response = restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
        System.out.println(response.isAcknowledged());
    }


    //测试添加文档
    @Test
    public void testAddDocument() throws IOException {
        User user = new User("cooper",22);
        //创建请求
        IndexRequest request = new IndexRequest("cooper_index");

        //规则 put/cooper_index/_doc/1
        request.id("1");
        request.timeout("1s");

        //将数据请求放入到json中
        request.source(JSON.toJSONString(user), XContentType.JSON);

        //客户端发送请求并获取响应结果
        IndexResponse response = restHighLevelClient.index(request, RequestOptions.DEFAULT);

        System.out.println(response.toString());
    }

    //查询是否存在该字段
    @Test
    public void testIsExist() throws IOException {
        GetRequest request = new GetRequest("cooper_index", "1");

        request.fetchSourceContext(new FetchSourceContext(false));//不获取返回的_source的上下文  提高查询的效率
        request.storedFields("_none");
        boolean exists = restHighLevelClient.exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }

    //获取文档信息
    @Test
    public void testGetDocument() throws IOException {
        GetRequest request = new GetRequest("cooper_index", "1");
        GetResponse response = restHighLevelClient.get(request, RequestOptions.DEFAULT);
        System.out.println(response.getSourceAsString());//打印文档内容
        System.out.println(response);
    }

    //更新文档信息
    @Test
    public void testUpdateDocument() throws IOException {
        UpdateRequest request = new UpdateRequest("cooper_index", "1");
        request.timeout("1s");

        User user = new User("Aaron",10);
        request.doc(JSON.toJSONString(user),XContentType.JSON);

        UpdateResponse response = restHighLevelClient.update(request, RequestOptions.DEFAULT);
        System.out.println(response.status());
    }


    //删除文档信息
    @Test
    public void testDeleteDocument() throws IOException {
        DeleteRequest request = new DeleteRequest("cooper_index", "1");
        request.timeout("1s");


        DeleteResponse response = restHighLevelClient.delete(request, RequestOptions.DEFAULT);
        System.out.println(response.status());
    }
}
```

### 批量插入和查询

```java
//批量插入数据
    @Test
    public void testBulkRequest() throws IOException {
        BulkRequest request = new BulkRequest();
        request.timeout("10s");

        ArrayList<User> users = new ArrayList<>();
        users.add(new User("Aaron1",1));
        users.add(new User("Aaron2",2));
        users.add(new User("Aaron3",3));

        for(int i = 0;i<users.size();i++)
        {
            request.add(new IndexRequest("cooper_index").id(""+(i+1)).
                    source(JSON.toJSONString(users.get(i)),XContentType.JSON)); //此处如果不添加id的话，系统会给你随机分配一个id
        }

        BulkResponse bulkResponse = restHighLevelClient.bulk(request, RequestOptions.DEFAULT);
        System.out.println(bulkResponse.hasFailures());
    }

    //批量查询
    @Test
    public void testSearch() throws IOException {
        SearchRequest request = new SearchRequest();
        //构建查询条件
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        //查询条件可以使用QueryBuilder工具来实现
        //termQuery为精确查询

        //此处必须写成name.keyword 不能只写name 因为
        //elasticsearch 里默认的IK分词器是会将每一个中文都进行了分词的切割，所以你直接想查一整个词，或者一整句话是无返回结果的
        TermQueryBuilder termQueryBuilder = QueryBuilders.termQuery("name.keyword", "Aaron1");

        sourceBuilder.query(termQueryBuilder);


        sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));

        request.source(sourceBuilder);

        SearchResponse search = restHighLevelClient.search(request, RequestOptions.DEFAULT);

        System.out.println(JSON.toJSONString(search.getHits()));

        System.out.println("========");

        for (SearchHit hit : search.getHits().getHits()) {
            System.out.println(hit.getSourceAsMap());
        }
    }
```



### 京东搜索实战

#### 访问静态页面

![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203417.png)
**注意:上文修改ElasticSearch版本包的添加config包和其文件，导入静态资源**

```java
server.port=9090
# 关闭thymeleaf的缓存
spring.thymeleaf.cache=false
123
@Controller
public class Jd {
    @GetMapping({"/", "/jd"})
//    @ResponseBody
    public String jd() {
        return "index";
    }
}
// 可以访问首页
```

#### 数据爬取

```xml
//需要导入jsoup包
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.10.2</version>
</dependency>
```

```java
//编写工具类
public class HtmlParseUtil {
    public static void main(String[] args) throws IOException {
        new HtmlParseUtil().parseID("java").forEach(System.out::println);
    }

    public List<Content> parseID(String keyword) throws IOException {
        String url = "https://search.jd.com/Search?keyword="+keyword;
        //解析网页
        Document document = Jsoup.parse(new URL(url),3000);

        Element element = document.getElementById("J_goodsList");

        Elements elements = element.getElementsByTag("li");

        ArrayList<Content> list = new ArrayList<>();

        for (Element el : elements) {
            String img = el.getElementsByTag("img").eq(0).attr("src");
            String price = el.getElementsByClass("p-price").eq(0).text();
            String title = el.getElementsByClass("p-name").eq(0).text();

//            System.out.println("================");
//            System.out.println(img);
//            System.out.println(price);
//            System.out.println(title);
            Content content = new Content();
            content.setImg(img);
            content.setPrice(price);
            content.setTitle(title);

            list.add(content);
        }

        return list;
    }
}
```

#### 数据存进es

```java
package com.zgc.esjd3.service;

import com.alibaba.fastjson.JSON;
import com.zgc.esjd3.pojo.Content;
import com.zgc.esjd3.util.HtmlParseUtil;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.util.ArrayList;

@Service
public class ContentService {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    // Jsoup数据放es
    public boolean parseContent(String keyword) throws IOException {
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.timeout("3m");
        ArrayList<Content> contents = new HtmlParseUtil().parseJd(keyword);
        for (int i = 0; i < contents.size(); i++) {
            bulkRequest.add(new IndexRequest("jd_index").id("" + (i + 1)).source(JSON.toJSONString(contents.get(i)), XContentType.JSON));
        }

        BulkResponse bulk = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        return !bulk.hasFailures();  // hasFailures返回false为成功
    }

}

@GetMapping("/jdTest/{keyword}")
@ResponseBody
public String jdTest(@PathVariable String keyword) throws IOException {
    System.out.println(keyword);
    boolean b = contentService.parseContent(keyword);
    System.out.println(b);
    return keyword;
}
// 访问http://localhost:9090/jdTest/java，成功把数据录入es
```

#### 实现搜索功能

//分页+搜索+高亮业务编写

```java
public List<Map<String, Object>> searchPage(String keyword, int pageNo, int pageSize) throws IOException {
        if (pageNo <= 1) {
            pageNo = 1;
        }
        // 查询请求
        SearchRequest searchRequest = new SearchRequest("jd_index");
        // 查询构造
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        //分页
        searchSourceBuilder.from(pageNo);
        searchSourceBuilder.size(pageSize);

        //精准匹配
        TermQueryBuilder queryBuilder = QueryBuilders.termQuery("title", keyword);
        searchSourceBuilder.query(queryBuilder);

        //高亮
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("title");//设置高亮的字段
        highlightBuilder.requireFieldMatch(false); //多处高亮
        highlightBuilder.preTags("<span style= 'color:red'>");
        highlightBuilder.postTags("</span>");
        searchSourceBuilder.highlighter(highlightBuilder);

        // 执行搜索
        searchRequest.source(searchSourceBuilder);
        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        System.out.println(search);
        ArrayList<Map<String, Object>> list = new ArrayList<>();
        for (SearchHit hit : search.getHits()) {
            //解析高亮的字段，将原来的字段换为我们高亮的字段即可
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            HighlightField title1 = highlightFields.get("title");
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();//原来的结果
            //解析高亮的字段，将原来的字段换为我们高亮的字段即可
            if (title1 != null) {
                Text[] fragments = title1.fragments();
                String n_title = "";
                for (Text text : fragments) {
                    n_title += text;
                }
                sourceAsMap.put("title", n_title);//高亮字段替换掉原来的内容 也就是将原来的title替换为高亮的title
            }
            list.add(sourceAsMap);
        }
        return list;
    }

    @GetMapping("/search/{keyword}/{pageNo}/{pageSize}")
    @ResponseBody
    public List<Map<String, Object>> search(@PathVariable String keyword, @PathVariable Integer pageNo, @PathVariable Integer pageSize) throws IOException {
        return contentService.searchPage(keyword,pageNo,pageSize);
    }
1234567891011
```

#### **下载 vue:**

cnpm install vue
cnpm install axios
把两个文件放入static
![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203435.png)
![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203428.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810191845839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FkbWluNzQxYWRtaW4=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200819203441.png)