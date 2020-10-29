---
title: Too many open files
date: 2020-10-29 17:09:00 Z
---

Recently I faced "Too many open files" exception. But what is this message about?
This means that for each HTTP connection we open in a Java application, the operating system will allocate a file descriptor to relate the file to our Java process. Once the JVM finishes with the connection, it releases the descriptor.

In my current project (Spring app) we use WebSockets to notify client about new important events in our system. So for every opened browser tab SockJS will create connection to server. Let's assume that every user has opened 2 tabs in their browsers. Let's multiply this by 1000 active users. We end up with 2000 WebSockets connections.

By default, Linux systems limit the number of file descriptors that anyone process may open to 1024 per process. You can increase this limit, but sometimes it's not enough.

If you want to control and close WebSocket connections from server side, just follow this steps. Firstly in your WebSocketMessageBrokerConfigurer implementation decorate WebSocketHandler with example sessionHandler.

{% highlight java %}
@Override
public void configureWebSocketTransport(WebSocketTransportRegistration registry) {
    registry.addDecoratorFactory(new WebSocketHandlerDecoratorFactory() {
        @Override
        public WebSocketHandler decorate(WebSocketHandler handler) {
                return new WebSocketHandlerDecorator(handler) {
                    @Override
                    public void afterConnectionEstablished(final WebSocketSession session) throws Exception {
                        super.afterConnectionEstablished(session);
                        sessionHandler.register(session);
                    }
                };
        }
    });
}
{% endhighlight %}

Let's see how example WebSocketSessionHandler can look like:
{% highlight java %}
public class WebSocketSessionHandler {

    private static final Map<String, WebSocketAudit> SOCKET_AUDIT_MAP = new ConcurrentHashMap<>();

    public void register(WebSocketSession session) {
        SOCKET_AUDIT_MAP.put(session.getId(), new WebSocketAudit(session, LocalDateTime.now()));
    }

}
{% endhighlight %}

Now we have references to WebSocket sessions, so we can close them when we want.
You can schedule removing old sessions or implement any other logic which suites your needs.

{% highlight java %}
try {
    session.close();
} catch (IOException e) {
    log.error("Error while closing websocket session " + sessionId, e);
}
{% endhighlight %}

Example WebSocketAudit containing reference to WebSocketSession and information about session creation time.

{% highlight java %}
class WebSocketAudit {
    WebSocketSession webSocketSession;
    LocalDateTime createdAt;
}
{% endhighlight %}

Hope you enjoy this post and it will be helpful for you. If you have any questions or problems leave a comment or send email.

See You soon!