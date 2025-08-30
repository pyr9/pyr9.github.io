---
title: 第三方HTTP接口调用方案
date: 2025-08-23 13:05:54
tags: 工具类
categories: 实战问题复盘
---

# 1. 场景

为解决多智能体接口、第三方 API（如翻译服务、AI 解析接口）的调用一致性问题，基于 Spring WebFlux 的 WebClient 封装统一 HTTP 调用框架，支持 GET/POST/PUT 等请求方法，实现 "一处配置、多处复用" 的调用模式。

# 2. 解决方案

## 1. 封装统一HTTP调用框架

1. 统一管理接口超时时间（连接超时 3s、读取超时 10s）、重试策略（最多 3 次重试 ）
2. 响应式处理与异常统一转换：利用 WebClient 的响应式特性（Mono/Flux）处理异步请求，通过`onErrorResume`将不同接口的异常（如 404、500、超时）统一转换为业务异常（`ApiException`），配合全局异常处理器返回标准化错误信息

```java
@Data
@Builder
public class ApiCallHttpRequest {
    private String systemCode;
    private String apiCallCode;                    // 接口调用标识（可选）
    private HttpMethod method;                     // 请求方法
    private String url;                            // 完整URL
    private Map<String, Object> params;            // 请求参数（body 或 query）
    private Map<String, String> headers; // 请求头
    private Long bizId; //业务ID
}
```

```java
import lombok.extern.slf4j.Slf4j;
import org.jetbrains.annotations.Nullable;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatusCode;
import org.springframework.http.MediaType;
import org.springframework.http.client.MultipartBodyBuilder;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

import java.io.File;
import java.io.InputStream;
import java.time.Duration;
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

/**
 * 通用 HTTP 工具类（基于 WebClient）
 */
@Slf4j
public class HttpUtil {

    private final WebClient webClient;

    public HttpUtil(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.build();
    }


    /**
     * GET 请求
     */
    public String get(ApiCallHttpRequest callHttpRequest) {
        callHttpRequest.setMethod(HttpMethod.GET);
        return doRequest(callHttpRequest);
    }

    /**
     * DELETE 请求
     */
    public String delete(ApiCallHttpRequest callHttpRequest) {
        callHttpRequest.setMethod(HttpMethod.DELETE);
        return doRequest(callHttpRequest);
    }

    /**
     * 发送 POST 请求
     */
    public String post(ApiCallHttpRequest callHttpRequest) {
        callHttpRequest.setMethod(HttpMethod.POST);
        return doRequest(callHttpRequest);
    }

    /**
     * 通用请求（GET / DELETE）
     *
     * @return 响应字符串
     */
    public String doRequest(ApiCallHttpRequest callHttpRequest) {
        long startTime = System.currentTimeMillis(); // 记录开始时间
        HttpMethod method = callHttpRequest.getMethod();
        String url = callHttpRequest.getUrl();
        Map<String, Object> params = callHttpRequest.getParams();
        Map<String, String> headers = callHttpRequest.getHeaders();
        WebClient.RequestHeadersSpec<?> request;

        if (method == HttpMethod.GET || method == HttpMethod.DELETE) {
            // GET / DELETE 用 query 参数
            if (params != null && !params.isEmpty()) {
                request = webClient.method(method).uri(uriBuilder -> {
                    uriBuilder.path(url);
                    params.forEach(uriBuilder::queryParam);
                    return uriBuilder.build();
                });
            } else {
                request = webClient.method(method).uri(url);
            }

        } else if (method == HttpMethod.POST || method == HttpMethod.PUT || method == HttpMethod.PATCH) {
            WebClient.RequestBodySpec bodySpec = webClient.method(method).uri(url);

            // 设置 headers
            bodySpec = setHeaders(headers, bodySpec);

            if (params != null && !params.isEmpty()) {
                // 是否包含文件
                boolean hasFile = params.values().stream()
                        .anyMatch(v -> v instanceof Resource || v instanceof java.io.File);

                if (hasFile) {
                    MultipartBodyBuilder bodyBuilder = new MultipartBodyBuilder();
                    for (Map.Entry<String, Object> entry : params.entrySet()) {
                        Object value = entry.getValue();
                        if (value instanceof Resource) {
                            bodyBuilder.part(entry.getKey(), value);
                        } else if (value instanceof java.io.File) {
                            bodyBuilder.part(entry.getKey(), new FileSystemResource((java.io.File) value));
                        } else {
                            bodyBuilder.part(entry.getKey(), value);
                        }
                    }
                    bodySpec.contentType(MediaType.MULTIPART_FORM_DATA)
                            .body(BodyInserters.fromMultipartData(bodyBuilder.build()));
                } else {
                    bodySpec.contentType(MediaType.APPLICATION_JSON)
                            .bodyValue(params);
                }
            }

            request = bodySpec;

        } else {
            // 其他方法，默认无 body
            request = webClient.method(method).uri(url);
        }

        request = setHeaders(headers, request);
        String responseBody = getResponseBody(callHttpRequest, request.retrieve());

        // 计算耗时
        long duration = System.currentTimeMillis() - startTime;
        log.info("请求apicode: {} 请求地址：{}, 参数：{}， Headers: {}, 返回值：{} 耗时: {}ms", callHttpRequest.getApiCallCode(), callHttpRequest.getUrl(), callHttpRequest.getParams(), callHttpRequest.getHeaders(), responseBody, duration);
        return responseBody;
    }

    /**
     * 单独处理参数中的不可序列化的文件对象
     */
    public static Map<String, Object> sanitizeParams(Map<String, Object> params) {
        Map<String, Object> safeParams = new HashMap<>();
        for (Map.Entry<String, Object> entry : params.entrySet()) {
            Object value = entry.getValue();
            if (value instanceof InputStream) {
                safeParams.put(entry.getKey(), "InputStream[ignored]");
            } else if (value instanceof File file) {
                safeParams.put(entry.getKey(), Map.of(
                        "fileName", file.getName(),
                        "fileSize", file.length()
                ));
            } else if (value instanceof Resource resource) {
                safeParams.put(entry.getKey(), Map.of(
                        "resourceName", Objects.requireNonNull(resource.getFilename()),
                        "resourceDescription", resource.getDescription()
                ));
            } else {
                safeParams.put(entry.getKey(), value);
            }
        }
        return safeParams;
    }


    private static WebClient.RequestHeadersSpec<?> setHeaders(Map<String, String> headers, WebClient.RequestHeadersSpec<?> request) {
        if (headers != null && !headers.isEmpty()) {
            request = request.headers(h -> headers.forEach(h::add));

        }
        return request;
    }

    private static WebClient.RequestBodySpec setHeaders(Map<String, String> headers, WebClient.RequestBodySpec request) {
        if (headers != null && !headers.isEmpty()) {
            request = request.headers(h -> headers.forEach(h::add));
        }
        return request;
    }


    @Nullable
    private static String getResponseBody(ApiCallHttpRequest callHttpRequest, WebClient.ResponseSpec request) {
        return request
                .onStatus(HttpStatusCode::isError,
                        clientResponse ->
                                clientResponse.bodyToMono(String.class)
                                        .flatMap(body -> Mono.error(new ApiCallException(
                                                callHttpRequest,
                                                clientResponse.statusCode().value(),
                                                body,
                                                JAI_SERVICE_ERROR
                                        ))))
                .bodyToMono(String.class)
                .retryWhen(
                        reactor.util.retry.Retry.backoff(3, Duration.ofSeconds(1))
                                .maxBackoff(Duration.ofSeconds(10))
                                .filter(ex -> !(ex instanceof java.util.concurrent.TimeoutException))
                                .onRetryExhaustedThrow((spec, signal) -> signal.failure())
                )
                .timeout(Duration.ofMinutes(5))
                .block();
    }
}
```



## 2. 增加统一日志表

增加统一日志表`api_call_log`记录 “请求 - 响应  - 异常栈“ 方便后续定位问题

```java
@NoArgsConstructor
@AllArgsConstructor
@TableName("api_call_log")
public class ApiCallLog extends BaseEntity {

    @Serial
    private static final long serialVersionUID = 1L;

    /**
     * 逐渐
     */
    @TableId
    private Long id;
    /**
     * 调用来源系统编码
     */
    private String systemCode;
    /**
     * 接口名称/标识
     */
    private String apiCode;
    /**
     * 请求参数
     */
    private String requestPayload;
    /**
     * 请求头
     */
    private String requestHeaders;
    /**
     * 响应状态码
     */
    private Integer responseCode;
    /**
     * 错误消息/补充信息
     */
    private String message;
    /**
     * 响应数据
     */
    private String responseData;
    /**
     * 业务ID，可存订单ID、任务ID、会话ID等
     */
    private Long bizId;
}
```

#### 1. 正常调用日志写入（可增加拦截器）

```java
@Override
public ResultBody<commonResponseData> chat(T chatRequest, ChatModelVo chatModelVo) throws IOException {
    beforeChat(chatRequest);
    Map<String, Object> params = buildRequestParams(chatRequest);
    Map<String, String> headers = buildRequestHeaders();
    ApiCallHttpRequest request = buildApiCallHttpRequest(chatRequest, chatModelVo, params, headers);
    Object responseData = jaiHttpUtil.doRequest(request, getResponseDataClass(chatRequest));
    apiCallLogService.saveLog(request, responseData);
    commonResponseData result = rebuild(chatRequest, responseData);
    saveChatMessage(chatRequest.getUserId(), chatRequest.getSessionId(), result);
    return ResultBody.success(result);
}
```

#### 2. 异常状态日志写入

```java
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * 全局异常处理器
 */
@Slf4j
@RestControllerAdvice
public class ApiCallExceptionHandler {

    @Resource
    private IApiCallLogService apiCallLogService;

    @ExceptionHandler(ApiCallException.class)
    public R<Void> handleApiCallException(ApiCallException ex) {
        apiCallLogService.saveLog(ex.getRequest(), ex.getResponseData(), ex);
        return R.fail(JAI_SERVICE_ERROR, ex.getMessage());
    }
}

```



## 3. 全链路日志追踪

1. 集成Spring Cloud Sleuth， 生成traceID [集成Spring Cloud Sleuth ](https://pyr9.github.io/%E8%B7%A8%E5%BA%94%E7%94%A8%E7%9A%84%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA/#2-%E8%B0%83%E7%94%A8%E9%93%BE%E8%B7%AF%E6%A8%A1%E5%9E%8B)

2. 集成ELK实现日志检索.  [docker搭建ELK](https://pyr9.github.io/docker搭建ELK/)
3. 使用链路追踪工具—skywalking/ZIpKin [skywalking](https://pyr9.github.io/Skywalking监控/)/[ZIpKin](https://pyr9.github.io/%E8%B7%A8%E5%BA%94%E7%94%A8%E7%9A%84%E9%93%BE%E8%B7%AF%E8%BF%BD%E8%B8%AA/#3-Sleuth%E4%B8%8EZipkin%E9%9B%86%E6%88%90)

# 3. 项目成果

- 开发效率提升：统一 HTTP 框架支持 10 + 第三方接口的快速接入，新接口集成时间从平均 2 天缩短至 4 小时，代码复用率提升 70%；
- 稳定性增强：通过统一的超时控制与重试策略，第三方接口调用失败率从 12% 降至 3%，配合完整的日志链路，问题排查时间缩短 80%；
