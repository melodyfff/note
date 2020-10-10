---
title: 【Java】 重复读取request中的body - Warp ServletRequest
tags: [java]
date: 2018-9-29
---

# 重复读取request中的body - Warp ServletRequest

**BodyReaderHttpServletRequestWrapper.java**
```java
import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.BufferedReader;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;

/**
 *
 * wrap {@link HttpServletRequest}
 *
 * make body can read
 *
 * @author xinchen
 * @version 1.0
 * @date 10/10/2020 13:36
 */
public class BodyReaderHttpServletRequestWrapper extends HttpServletRequestWrapper {
    /**存储body的值，在需要使用的时候，通过设置InputStream设置进去*/
    private final byte[] body;
    private final String bodyString;

    /**
     * wrapper request
     * @param request HttpServletRequest
     * @throws IOException IOException
     */
    BodyReaderHttpServletRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        // 重设body值
        bodyString = cloneBody(request.getInputStream());
        body =bodyString.getBytes(StandardCharsets.UTF_8);
    }

    @Override
    public BufferedReader getReader() throws IOException {
        return new BufferedReader(new InputStreamReader(getInputStream()));
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream delegate = new ByteArrayInputStream(body);
        return new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return delegate.read();
            }
            @Override
            public boolean isFinished() {
                return false;
            }
            @Override
            public boolean isReady() {
                return false;
            }
            @Override
            public void setReadListener(ReadListener readListener) {
            }

            /**
             * Closes this input stream and releases any system resources associated
             * with the stream.
             *
             * @throws IOException IOException
             */
            @Override
            public void close() throws IOException {
                delegate.close();
                super.close();
            }
        };
    }

    private String cloneBody(ServletInputStream inputStream) throws IOException {
        // get byte data
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int len;
        while ((len = inputStream.read(buffer)) > -1) {
            byteArrayOutputStream.write(buffer, 0, len);
        }
        byteArrayOutputStream.flush();

        // trans to string
        String line;
        StringBuilder sb = new StringBuilder();
        // 特性关闭相关的流
        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(
                        new ByteArrayInputStream(byteArrayOutputStream.toByteArray()), StandardCharsets.UTF_8)
        )) {
            while ((line = reader.readLine()) != null) {
                sb.append(line);
            }
        }
        return sb.toString();
    }

    String getBodyString() {
        return bodyString;
    }
}
```

假设在`ServletRequest`中
```java
public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
    if (request instanceof HttpServletRequest && response instanceof HttpServletResponse) {
        BodyReaderHttpServletRequestWrapper wrapper = new BodyReaderHttpServletRequestWrapper(request);
        chain.doFilter(wrapper, response);
    }
}
```