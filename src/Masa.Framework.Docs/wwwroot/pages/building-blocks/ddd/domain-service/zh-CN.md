## 领域服务

是领域模型的操作者, 被用来处理业务逻辑, 它是无状态的, 状态由领域对象来保存, 提供面向应用层的服务, 完成封装领域知识, 供应用层使用。

与应用服务不同的是, 应用服务仅负责编排和转发, 它将要实现的功能委托给一个或多个领域对象来实现, 它本身只负责处理业务用例的执行顺序以及结果的拼装, 在应用服务中不应该包含业务逻辑

继承`IDomainService`的类被标记为领域服务, 领域服务支持从DI获取, 其中提供了`EventBus` (用于提供发送领域事件)

```csharp
public class PaymentDomainService : DomainService
{
    private readonly ILogger<PaymentDomainService> _logger;

    public PaymentDomainService(IDomainEventBus eventBus, ILogger<PaymentDomainService> logger) : base(eventBus)
    {
        _logger = logger;
    }

    public async Task StatusChangedAsync(Aggregate.Payment payment)
    {
        IIntegrationDomainEvent orderPaymentDomainEvent = payment.Succeeded ? 
            new OrderPaymentSucceededDomainEvent(payment.OrderId): 
            new OrderPaymentFailedDomainEvent(payment.OrderId);

        _logger.LogInformation(
            "----- Publishing integration event: {IntegrationEventId} from {AppName} - ({@IntegrationEvent})", 
            orderPaymentDomainEvent.GetEventId(), 
            Program.AppName, 
            orderPaymentDomainEvent);

        await EventBus.PublishAsync(orderPaymentDomainEvent);
    }
}
```

> 继承`DomainService`的类会被自动注入, 其生命周期为`Scoped`, 它可以在应用服务的构造函数中被注入使用