---
title: HttpServer配置
abbrlink: 394816549
date: 2021-12-12 15:41:58
tags: [Guide,Springboot,Netty,源码阅读]
categories: [Guide,Springboot,Netty,源码阅读]
---
HttpServer配置

```java
package com.github.jsoncat.server;

import com.github.jsoncat.core.springmvc.factory.FullHttpResponseFactory;
import com.github.jsoncat.core.springmvc.factory.RequestHandlerFactory;
import com.github.jsoncat.core.springmvc.handler.RequestHandler;
import com.github.jsoncat.core.springmvc.util.UrlUtil;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.FullHttpRequest;
import io.netty.handler.codec.http.FullHttpResponse;
import io.netty.handler.codec.http.HttpResponseStatus;
import io.netty.handler.codec.http.HttpUtil;
import io.netty.util.AsciiString;
import lombok.extern.slf4j.Slf4j;

/**
 * @author shuang.kou
 * @createTime 2020年09月23日 17:33:00
 **/
@Slf4j
public class HttpServerHandler extends SimpleChannelInboundHandler<FullHttpRequest> {
    private static final String FAVICON_ICO = "/favicon.ico";
    private static final AsciiString CONNECTION = AsciiString.cached("Connection");
    private static final AsciiString KEEP_ALIVE = AsciiString.cached("keep-alive");

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest fullHttpRequest) {
        log.info("Handle http request:{}", fullHttpRequest);
        String uri = fullHttpRequest.uri();
        if (uri.equals(FAVICON_ICO)) {
            return;
        }
        RequestHandler requestHandler = RequestHandlerFactory.get(fullHttpRequest.method());
        FullHttpResponse fullHttpResponse;
        try {
            fullHttpResponse = requestHandler.handle(fullHttpRequest);
        } catch (Throwable e) {
            log.error("Caught an unexpected error.", e);
            String requestPath = UrlUtil.getRequestPath(fullHttpRequest.uri());
            fullHttpResponse = FullHttpResponseFactory.getErrorResponse(requestPath, e.toString(), HttpResponseStatus.INTERNAL_SERVER_ERROR);
        }
        boolean keepAlive = HttpUtil.isKeepAlive(fullHttpRequest);
        if (!keepAlive) {
            ctx.write(fullHttpResponse).addListener(ChannelFutureListener.CLOSE);
        } else {
            fullHttpResponse.headers().set(CONNECTION, KEEP_ALIVE);
            ctx.write(fullHttpResponse);
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.flush();
    }
}
```

这里是主要的配置条件  但目前我还并未成功运用



![image-20211212094038015](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211212094038015.png?x-oss-process=style/qingyun)

获取到了主要的请求Url



```java
RequestHandler requestHandler = RequestHandlerFactory.get(fullHttpRequest.method());
```

确认请求方式为Get请求



### 当为Get请求时，会进入  GetRequestHandler 类

