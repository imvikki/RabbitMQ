# RabbitMQ

## TraceId not shown after springBoot 3 :

### Problem Statement :

  When migrated springBoot from 2.x to 3.x, there were missing traceId only for the logs that generated while publishing via exchange.

### Solution :

  Usage of brave tracer, solved this problem. This solves the below scenarios too

**Scenarios** :
  1. If a message is getting created synchronously as part of an API call, 
      a. the traceId from incoming request should get propagated if present
      b. new traceId should be generated if not present
  2. If a message is getting created in response to an incoming message, 
      a. the traceId from incoming message should get propagated if present
      b. new traceId should be generated if not present
  3. If a message is getting created in response to a scheduled job (or similar),
      a. the traceId from job should get propagated if present
      b. new traceId should be generated if not present

```

@Bean
  public Tracing tracing() {
    return Tracing.newBuilder()
      .currentTraceContext(ThreadLocalCurrentTraceContext.newBuilder()
        .addScopeDecorator(MDCScopeDecorator.get())
        .build())
      .build();
  }

  @Bean
  public MessagingTracing messagingTracing(Tracing tracing) {
    MessagingTracing messagingTracing = MessagingTracing.create(tracing);
    messagingTracing.tracing().setNoop(true);
    return messagingTracing;
  }

  @Bean
  public SpringRabbitTracing springRabbitTracing(MessagingTracing messagingTracing) {
    return SpringRabbitTracing.newBuilder(messagingTracing)
      .build();
  }

```

### Disabled extra log from barve trace library

```
messagingTracing.tracing().setNoop(true);
```
