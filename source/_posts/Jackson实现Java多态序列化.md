---
title: Jackson实现Java多态序列化
date: 2025-08-07 20:02:00
tags:
categories: Java基础
---

在开发 RESTful API 或微服务架构时，我们经常会遇到这样的需求：**同一个接口返回不同类型的数据对象，但这些对象具有共同的基类**。例如，聊天系统中的不同消息类型、AI 平台中的多种模型请求等。

这时，就需要使用 **多态序列化/反序列化** 来正确处理这些对象。而 **Jackson** 作为 Java 生态中最主流的 JSON 处理库，提供了强大的多态支持。

# 1. 问题场景

假设我们有一个聊天系统，支持普通聊天和 ESG 问答两种模式：

```java
// 基础请求参数
public abstract class BaseChatRequest {
    private String model;
    private String userId;
    private Boolean stream;
}

 // 普通聊天
public class ChatRequest extends BaseChatRequest {
    private String question;
    private List<Long> attachmentIds;
}
 // ESG 问答
public class ESGChatRequest extends BaseChatRequest {
    private String question;
    private String processId;
}
```



现在，前端发送一个 JSON 请求：

```json
{
  "model": "chat",
  "userId": 1,
  "stream": true,
  "question": "这是啥",
  "processId": "1953357916506259457",
}
```

后端如何知道这个 JSON 应该反序列化为 `ESGChatRequest` 而不是 `ChatRequest`？



# 2. 核心注解

## 2.1. `@JsonTypeInfo` —— 定义类型识别方式

```java
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.EXISTING_PROPERTY,
    property = "model",
    visible = true
)
```

- `use`: 使用哪种方式标识类型

- - `NAME`: 使用逻辑名称（推荐）
  - `CLASS`: 使用完整类名（不安全）

- `include`: 类型信息如何包含在 JSON 中

- - `EXISTING_PROPERTY`: 使用已有字段（如 `model`）
  - `PROPERTY`: 作为额外字段添加

- `property`: 用于存储类型标识的字段名（如 `"model"`）
- `visible`: 是否在序列化时保留该字段（`true` 表示保留）, 不设置的话，序列化时会默认移除这个字段

## 2.2.  `@JsonSubTypes` —— 声明所有子类型

```java
@JsonSubTypes({
    @JsonSubTypes.Type(value = ChatRequest.class, name = "chat"),
    @JsonSubTypes.Type(value = ESGChatRequest.class, name = "esg_faqs")
})
```

- 将逻辑名称（如 `"esg_faqs"`）映射到具体 Java 类。

# 3. 完整代码

```java
public interface ChatModeConstants {
    String JASOLAR_CHAT_CODE = "chat";
    String ESG_FAQS = "esg-faqs";
}
@JsonTypeInfo(
        use = JsonTypeInfo.Id.NAME,
        include = JsonTypeInfo.As.EXISTING_PROPERTY,
        property = "model",
        visible = true
)
@JsonSubTypes({
        @JsonSubTypes.Type(value = ChatRequest.class, name = ChatModeConstants.CHAT_CODE),
        @JsonSubTypes.Type(value = ESGChatRequest.class, name = ChatModeConstants.ESG_FAQS)
})
@Data
public class BaseChatRequest implements IChatRequest {
    @NotEmpty(message = "传入的模型不能为空")
    private String model;
    private Long userId;
    private Boolean stream = Boolean.TRUE;
    private Long sessionId;
    private boolean enableThinking;
    private boolean searchEnabled;

    public String getCategory() {
        return ChatModeConstants.CHAT_CODE;
    }

    public String getQuestion() {
        return EMPTY;
    }
}
@Controller
@Slf4j
@RequiredArgsConstructor
@RequestMapping("/chat")
public class ChatController {

    private final ISseService sseService;
    
    @PostMapping("/send")
    @ResponseBody
    public SseEmitter sseChat(@RequestBody @Valid BaseChatRequest baseChatRequest, HttpServletRequest request) {
        return sseService.sseChat(baseChatRequest, request);
    }
}
@Service
@Slf4j
@RequiredArgsConstructor
public class SseServiceImpl implements ISseService {
    @Override
    public SseEmitter sseChat(BaseChatRequest chatRequest, HttpServletRequest request) {
        SseEmitter sseEmitter = new SseEmitter(0L);
        // 根据模型分类调用不同的处理逻辑
        IChatService<BaseChatRequest> chatService = chatServiceFactory.getChatService(chatModelVo.getCategory());
        chatService.chat(chatRequest, sseEmitter);
        return sseEmitter;
    }
}
public interface IChatService<T extends BaseChatRequest> {
    SseEmitter chat(T chatRequest, SseEmitter emitter);
}
@Service
@Slf4j
public abstract class BaseChatServiceImpl<T extends BaseChatRequest> implements IChatService<T> {

    @Transactional
    @Override
    public SseEmitter chat(T chatRequest, SseEmitter emitter) {
        beforeChat(chatRequest);
        OpenAiStreamClient openAiStreamClient = createOpenAiStreamClient(chatRequest);
        ChatSSEEventSourceListener listener = new ChatSSEEventSourceListener(emitter, chatRequest.getUserId(), chatRequest.getSessionId());
        BaseChatCompletion completion = buildChatCompletion(chatRequest);
        openAiStreamClient.streamChatCompletion(completion, listener);
        return emitter;
    }

    public void beforeChat(T chatRequest) {
        // This method can be overridden in subclasses to perform actions before the chat starts
    }
}
@Service
@Slf4j
public class ChatServiceImpl extends BaseChatServiceImpl<ChatRequest> {

    public ChatServiceImpl(IChatModelService chatModelService) {
        super(chatModelService);
    }

    @Override
    public void beforeChat(ChatRequest chatRequest) {
        saveMessageAttachments(chatRequest);
    }
}
@Service
@Slf4j
public class ESGChatServiceImpl extends BaseChatServiceImpl<ESGChatRequest> {

    public ESGChatServiceImpl(IChatModelService chatModelService) {
        super(chatModelService);
    }
}
@Component
public class ChatServiceFactory  implements ApplicationContextAware {
    private final Map<String, IChatService<BaseChatRequest>> chatServiceMap = new ConcurrentHashMap<>();

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        // 初始化时收集所有IChatService的实现
        Map<String, IChatService> serviceMap = applicationContext.getBeansOfType(IChatService.class);
        for (IChatService service : serviceMap.values()) {
            if (service != null) {
                chatServiceMap.put(service.getCategory(), service);
            }
        }
    }

    /**
     * 根据模型类别获取对应的聊天服务实现
     */
    public IChatService<BaseChatRequest> getChatService(String category) {
        IChatService<BaseChatRequest> service = chatServiceMap.get(category);
        if (service == null) {
            throw new IllegalArgumentException("不支持的模型类别: " + category);
        }
        return service;
    }
}
```
