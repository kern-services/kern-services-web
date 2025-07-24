---
title: "Reliable Domain Event Dispatching with Transactions in Entity Framework Core"
date: 2025-07-24
description: "Ideas on how to implement reliable domain event dispatching with transactions in Entity Framework Core"
tags: ["dotnet", "entity-framework", "ddd", "domain-events"]
featuredImage: "/images/insights/bradyn-trollip-pxVOztBa6mY-unsplash.jpg"
author: "Mattanja Kern"
photoCredit:
  photographer: "Bradyn Trollip"
  photographerUrl: "https://unsplash.com/de/@bradyn?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
  source: "Unsplash"
  sourceUrl: "https://unsplash.com/de/fotos/person-die-weisse-und-blaue-plastikblocke-halt-pxVOztBa6mY?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
---

## The Problem: Domain Events and Transactional Consistency

In modern DDD-inspired architectures, domain events are a powerful way to decouple side effects from your core business logic. When an aggregate root changes, it raises domain events, which are then dispatched to handlers that perform additional actions (such as sending emails, publishing integration events, etc).

However, when using Entity Framework Core, a subtle but critical problem arises: **When should you dispatch domain events in relation to your database transaction?**

### The Naive Approach

A common (but flawed) approach is to dispatch domain events immediately after calling `SaveChanges` or `SaveChangesAsync`:

```csharp
public override async Task<int> SaveChangesAsync(bool acceptAllChangesOnSuccess, CancellationToken cancellationToken = default)
{
    var result = await base.SaveChangesAsync(acceptAllChangesOnSuccess, cancellationToken);
    await _dispatcher.DispatchAndClearEvents(entitiesWithEvents);
    return result;
}
```

This works fine **if** you are not using explicit transactions. But if you wrap multiple `SaveChanges` calls in a manual transaction, or if the transaction fails after `SaveChanges` returns, you may end up dispatching events for changes that are never actually committed to the database. This can lead to data inconsistencies and hard-to-debug issues.

## The Solution: Interceptors for Transaction-Aware Event Dispatching

To solve this, we need to ensure that domain events are only dispatched **after** the database transaction has been successfully committed. This is non-trivial, because:

- EF Core creates implicit transactions for each `SaveChanges` call, unless you start your own transaction.
- If you use manual transactions, you might call `SaveChanges` multiple times before committing.
- There is no built-in EF Core event for "transaction committed" at the DbContext level.

### Our Approach: Combining SaveChanges and Transaction Interceptors

We use two EF Core interceptors:

1. **SaveChangesInterceptor**: Collects and clears domain events from entities after each `SaveChanges` call. If no transaction is present, it dispatches the events immediately.
2. **DbTransactionInterceptor**: If a transaction is present, it collects all domain events for the transaction. When the transaction is committed, it dispatches all collected events. If the transaction is rolled back or fails, the events are discarded.

This ensures that:
- **No events are dispatched unless the transaction is committed.**
- **No duplicate events are dispatched, even if SaveChanges is called multiple times in a transaction.**

### Example Implementation

#### 1. Collecting and Clearing Events in SaveChangesInterceptor

```csharp
public class DomainEventsSaveChangesInterceptor : SaveChangesInterceptor
{
    public override async ValueTask<int> SavedChangesAsync(
        SaveChangesCompletedEventData eventData,
        int result,
        CancellationToken cancellationToken = default)
    {
        if (eventData.Context is not null)
        {
            var entitiesWithEvents = eventData.Context.ChangeTracker.Entries<HasDomainEventsBase>()
                .Select(e => e.Entity)
                .Where(e => e.DomainEvents.Any())
                .ToArray();
            if (entitiesWithEvents.Length > 0)
            {
                var allEvents = new List<DomainEventBase>();
                foreach (var entity in entitiesWithEvents)
                {
                    allEvents.AddRange(entity.DomainEvents);
                    entity.ClearDomainEvents(); // Prevents duplicate dispatch
                }
                var dbContext = eventData.Context;
                var dbTransaction = dbContext.Database.CurrentTransaction?.GetDbTransaction();
                if (dbTransaction == null)
                {
                    // No transaction: dispatch immediately
                    await dispatcher.DispatchAndClearEvents(allEvents);
                }
                else
                {
                    // Transaction: collect for later
                    transactionInterceptor.CollectEvents(dbTransaction, allEvents);
                }
            }
        }
        return await base.SavedChangesAsync(eventData, result, cancellationToken);
    }
}
```

#### 2. Dispatching Events After Transaction Commit

```csharp
public class DomainEventsTransactionInterceptor : DbTransactionInterceptor
{
    private readonly ConcurrentDictionary<DbTransaction, List<DomainEventBase>> _transactionEvents = new();

    public void CollectEvents(DbTransaction transaction, IEnumerable<DomainEventBase> events)
    {
        if (!_transactionEvents.TryGetValue(transaction, out var list))
        {
            list = [];
            _transactionEvents[transaction] = list;
        }
        list.AddRange(events);
    }

    public override async Task TransactionCommittedAsync(DbTransaction transaction, TransactionEndEventData eventData, CancellationToken cancellationToken = default)
    {
        if (_transactionEvents.TryRemove(transaction, out var events) && events.Count > 0)
        {
            await dispatcher.DispatchAndClearEvents(events);
        }
    }

    public override Task TransactionFailedAsync(DbTransaction transaction, TransactionErrorEventData eventData, CancellationToken cancellationToken = default)
    {
        _transactionEvents.TryRemove(transaction, out _);
        return Task.CompletedTask;
    }

    public override Task TransactionRolledBackAsync(DbTransaction transaction, TransactionEndEventData eventData, CancellationToken cancellationToken = default)
    {
        _transactionEvents.TryRemove(transaction, out _);
        return Task.CompletedTask;
    }
}
```

#### 3. Registering the Interceptors

In your `DbContext`:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    // ... other config ...
    var dispatcher = serviceProvider.GetRequiredService<IDomainEventDispatcher>();
    var transactionInterceptor = new DomainEventsTransactionInterceptor(dispatcher);
    optionsBuilder.AddInterceptors(
        new DomainEventsSaveChangesInterceptor(dispatcher, transactionInterceptor),
        transactionInterceptor
    );
}
```

### Key Takeaways

- **Always clear domain events from entities after collecting them.** This prevents duplicate dispatch if `SaveChanges` is called multiple times in a transaction.
- **Use both SaveChanges and Transaction interceptors** to ensure events are only dispatched after a successful commit, regardless of how transactions are managed.
- **This approach is robust, future-proof, and works with both implicit and explicit transactions in EF Core.**

## Conclusion

Handling domain events in a transactional, reliable way with Entity Framework Core is tricky, but with the right use of interceptors, you can ensure your side effects are always consistent with your data. This pattern is production-proven and works seamlessly with DDD, CQRS, and Clean Architecture approaches.

If you have questions or want to see more advanced patterns (like outbox/eventual consistency), let us know!
