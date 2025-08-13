---
title: SpringAI之HelloWorld
date: 2025-08-09 21:32:55
tags:
categories: SpringAI
---

前期准备：引入依赖

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

# 1. ChatClient 对话

```java
import org.springframework.ai.chat.ChatClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class ChatController {

    @Autowired
    private ChatClient chatClient;

    @GetMapping("/chat")
    public String generate(@RequestParam String message) {
        return chatClient.call(message);
    }

}
```

# 2. ImageClient 生成图片

```java
@Autowired
private ImageClient imageClient;

@GetMapping("/image")
public String image(@RequestParam String message) {
    ImagePrompt imagePrompt = new ImagePrompt(message);
    ImageResponse imageResponse = imageClient.call(imagePrompt);
    return imageResponse.getResult().getOutput().getUrl();
}
```

# 3. OpenAiAudioTranscriptionClient 语音转文字

```java
@Value("classpath:/jfk.flac")
private Resource audioFile;

@Autowired
private OpenAiAudioTranscriptionClient audioTranscriptionClient;


@GetMapping("/audio")
public String audio() {
    AudioTranscriptionPrompt transcriptionRequest = new AudioTranscriptionPrompt(audioFile);
    AudioTranscriptionResponse response = audioTranscriptionClient.call(transcriptionRequest);
    return response.getResult().getOutput();
}
```

**传统语音识别** 和 **AI（深度学习）语音识别**的区别  

- **传统方法**：

- - 流程

- - - **先把声音拆开** → 分析成一个个音节（就像拼音或音标）
    - **查字典** → 根据规则把音节组合成单词
    - **套模板** → 用预先写好的语言规则拼成一句话

- - 缺点：

- - - 口音一重就懵了（比如四川话说“是”听成“死”）
    - 背景吵就听不清
    - 必须提前喂给它很多针对性语料（方言就得专门训练）

- **AI 方法**：

- - 流程

- - - 直接听完整句话（不拆音节）
    - 用大脑里的**神经网络**在海量真实录音中学过的经验来猜你说的内容

- - 优点：

- - - 不怕口音、方言（它见过太多了）
    - 背景有音乐、街道噪声也能识别
    - 一次训练就能识别多种语言
    - 还能自动加标点、断句，读起来更像人写的

# 4. EmbeddingClient 文本进行向量化

```java
@Autowired
private EmbeddingClient embeddingClient;

@GetMapping("/embedding")
public List<Double> embedding(@RequestParam String message) {
    return embeddingClient.embed(message);
}
```

# 5. Function 工具

- 定义工具

```java
import io.swagger.v3.oas.annotations.media.Schema;
import org.springframework.context.annotation.Description;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.util.function.Function;


@Component
@Description("获取指定地点的当前时间")
public class DateService implements Function<DateService.Request, DateService.Response> {

    public record Request(@Schema(description = "地点") String address) { }

    public record Response(String date) { }

    @Override
    public Response apply(Request request) {
        System.out.println(request.address);
        return new Response(String.format("%s的当前时间是%s", request.address, LocalDateTime.now()));
    }
}
```

- 调用方式一

```java
@GetMapping("/function")
public String function(@RequestParam String message) {
    Prompt prompt = new Prompt(message, OpenAiChatOptions.builder().withFunction("dateService").build());
    Generation generation = chatClient.call(prompt).getResult();
    return (generation != null) ? generation.getOutput().getContent() : "";
}
```

- 调用方式二

```java
@GetMapping("/functionCallback")
public String functionCallback(@RequestParam String message) {
    Prompt prompt = new Prompt(message, OpenAiChatOptions.builder().withFunctionCallbacks(
        List.of(FunctionCallbackWrapper.builder(new DateService())
                .withName("dateService")
                .withDescription("获取指定地点的当前时间").build())
    ).build());
    Generation generation = chatClient.call(prompt).getResult();
    return (generation != null) ? generation.getOutput().getContent() : "";
}
```

- 设置系统提示词后调用

```java
@GetMapping("/functionCallback")
public String functionCallback(@RequestParam String message) {

    SystemMessage systemMessage = new SystemMessage("请用中文回答我");
    UserMessage userMessage = new UserMessage(message);

    Prompt prompt = new Prompt(List.of(systemMessage, userMessage), OpenAiChatOptions.builder().withFunctionCallbacks(
        List.of(FunctionCallbackWrapper.builder(new DateService())
                .withName("dateService")
                .withDescription("获取指定地点的当前时间").build())
    ).build());
    Generation generation = chatClient.call(prompt).getResult();
    return (generation != null) ? generation.getOutput().getContent() : "";
}
```



交互流程图

我们通过第一次请求告诉OpenAi我的需求任务，以及我们提供了哪些工具。然后由OpenAi：

1. 理解任务，制定策略，也就是OpenAi分析需要调用哪些工具，并且调用这些工具的具体参数是什么，调用工具的顺序是什么
2. OpenAi就向OpenAiChatClient发送工具调用请求，并得到工具执行结果
3. OpenAi再基于任务和工具执行结果进行分析，看是否能完成任务了，还是需要继续调用工具。
4. 如果能完成任务了，那就直接把任务的执行结果返回给OpenAiChatClient。

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1755054911992-e482ce31-2c15-4724-8cd7-e9ff9036243e.png)
