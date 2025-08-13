---
title: SpringAI之基于RAG实现一套企业智能客服系统
date: 2025-08-12 21:25:26
tags:
categories: SpringAI
---

# 1. 前期准备

首先，我们需要把现有的常见文件整理成文档，可以是txt、pdf、xlsx、markdown等格式都可以，我们这里将问题和答案转成txt文件，文件为：外卖常见问题.txt

```
Q：在线支付取消订单后钱怎么返还？
订单取消后，款项会在一个工作日内，直接返还到您的美团账户余额。

Q：怎么查看退款是否成功？
退款会在一个工作日之内到美团账户余额，可在“账号管理——我的账号”中查看是否到账。

Q：美团账户里的余额怎么提现？
余额可到美团网（meituan.com）——“我的美团→美团余额”里提取到您的银行卡或者支付宝账号，另外，余额也可直接用于支付外卖订单（限支持在线支付的商家）。

Q：余额提现到账时间是多久？
2-5个工作日内可退回您的支付账户。由于银行处理可能有延迟，具体以账户的到账时间为准。

Q：申请退款后，商家拒绝了怎么办？
申请退款后，如果商家拒绝，此时回到订单页面点击“退款申诉”，美团客服介入处理。
```



# 2. 代码实现

## 2.1. 引入依赖

直接创建一个普通的Maven工程就可以了，然后引入spring-ai的依赖和你选择的大模型依赖，我这里使用open-ai：

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.ai</groupId>
      <artifactId>spring-ai-bom</artifactId>
      <version>0.8.1-SNAPSHOT</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
  </dependency>
</dependencies>
```

配置open AI

```properties
spring.ai.openai.api-key=xxx
spring.ai.openai.base-url=xxx
```



## 2.2. 导入知识库

### 2.2.1. 使用DocumentReader读取txt文件

实现类有：

1. JsonReader：读取JSON格式的文件
2. TextReader：读取txt文件
3. PagePdfDocumentReader：使用Apache PdfBox读取PDF文件
4. TikaDocumentReader：使用Apache Tika来读取PDF, DOC/DOCX, PPT/PPTX, and HTML等文件

```java
import org.springframework.ai.document.Document;
import org.springframework.ai.reader.TextReader;
import org.springframework.ai.transformer.splitter.TextSplitter;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Component;

import java.util.List;


@Component
public class DocumentService {

    @Value("classpath:meituan-qa.txt") 
    private Resource resource;


    public List<Document> loadText() {
        TextReader textReader = new TextReader(resource);
        textReader.getCustomMetadata().put("filename", "meituan-qa.txt");
        List<Document> documents = textReader.get();

        CustomerServiceTextSplitter TextSplitter = new CustomerServiceTextSplitter();
        List<Document> list = TextSplitter.apply(documents);

}
```

使用FileSystemDocumentLoader来加载本地文件，利用TextDocumentParser来解析txt文件，最终得到"meituan-qa.txt"文件所对应的Document对象。

### 2.2.2. 使用TextSplitter切分文件成问答对

```java
import org.springframework.ai.transformer.splitter.TextSplitter;
import java.util.List;


public class CustomerServiceTextSplitter implements TextSplitter {


     @Override
    protected List<String> splitText(String text) {
        return List.of(split(text));
    }

    public String[] split(String text) {
        // 使用正则表达式"\\s*\\R\\s*\\R\\s*"来进行切分
        return text.split("\\s*\\R\\s*\\R\\s*"); 
    }
    
}
```



切分结果为：

### 2.2.3. 基于Embedding+redis文本向量化并存储

- 引入依赖

```xml
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-redis</artifactId>
</dependency>

<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>5.1.0</version>
</dependency>
```

- 代码

```java
@Bean
public RedisVectorStore vectorStore(EmbeddingClient embeddingClient) {
    RedisVectorStore.RedisVectorStoreConfig config = RedisVectorStore.RedisVectorStoreConfig.builder()
        .withURI("redis://localhost:6379")
        .withMetadataFields(
            RedisVectorStore.MetadataField.text("filename"))
        .build();
    
    return new RedisVectorStore(config, embeddingClient);
}
import org.springframework.ai.document.Document;
import org.springframework.ai.reader.TextReader;
import org.springframework.ai.transformer.splitter.TextSplitter;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.Resource;
import org.springframework.stereotype.Component;

import java.util.List;


@Component
public class DocumentService {

    @Value("classpath:meituan-qa.txt") 
    private Resource resource;

        @Autowired
    private VectorStore vectorStore;

    public List<Document> loadText() {
        TextReader textReader = new TextReader(resource);
        textReader.getCustomMetadata().put("filename", "meituan-qa.txt");
        List<Document> documents = textReader.get();

        CustomerServiceTextSplitter TextSplitter = new CustomerServiceTextSplitter();
        List<Document> list = TextSplitter.apply(documents);

        
        // 把问题存到元数据中
        list.forEach(document -> document.getMetadata().put("question", document.getContent().split("\\n")[0]));
            return documents;
        }

        // 向量存储
        vectorStore.add(list)
 }
}
```

## 2.3. 使用vectorStore进行内容查找

定义元数据搜索：

```java
public List<Document> metadataSearch(String message, String question) {
    return vectorStore.similaritySearch(
        SearchRequest
        .query(message)
        .withTopK(5)
        .withSimilarityThreshold(0.1)
        .withFilterExpression(String.format("question in ['%s']", question)));
}
```



测试效果：

```java
@GetMapping("/documentMetadataSearch")
public List<Document> documentMetadataSearch(@RequestParam String message, @RequestParam String question) {
    return documentService.metadataSearch(message, question);
}
```

## 2.4. 组装提示词

```java
@GetMapping("/customerService")
public String customerService(@RequestParam String question) {

    // 向量搜索
    List<Document> documentList = documentService.search(question);

    // 提示词模板
    PromptTemplate promptTemplate = new PromptTemplate("{userMessage}\n\n 用以下信息回答问题:\n {contents}");

    // 组装提示词
    Prompt prompt = promptTemplate.create(Map.of("userMessage", question, "contents", documentList));

    // 调用大模型
    return chatClient.call(prompt).getResult().getOutput().getContent();
}
```
