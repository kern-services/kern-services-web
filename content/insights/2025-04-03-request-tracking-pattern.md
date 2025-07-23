---
title: "Request Tracking Pattern: Preventing Duplicate API Calls in .NET"
date: 2025-04-03
description: "Learn how to implement the Request Tracking Pattern in .NET to prevent duplicate API calls and optimize resource usage in distributed systems"
tags: ["concurrency", "dotnet", "performance", "patterns", "api"]
featuredImage: "/images/insights/jonathan-chng-HgoKvtKpyHA-unsplash.jpg"
author: "Mattanja Kern"
photoCredit:
  photographer: "Jonathan Chng"
  photographerUrl: "https://unsplash.com/@jon_chng?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
  source: "Unsplash"
  sourceUrl: "https://unsplash.com/photos/group-of-men-running-in-track-field-HgoKvtKpyHA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
---

In modern distributed systems, handling concurrent requests efficiently is crucial for both performance and cost optimization. One common challenge is dealing with multiple simultaneous requests for the same resource. Without proper handling, this can lead to redundant database queries, unnecessary API calls to external services, and increased costs.

In this article, we'll explore the Request Tracking Pattern - a powerful technique to prevent duplicate requests while maintaining system responsiveness. We'll implement this pattern in C# using .NET's `ConcurrentDictionary` and `TaskCompletionSource`.

## Why Request Tracking?

Consider a scenario where your application receives multiple requests for the same cell tower location within milliseconds of each other. Without request tracking:

1. Each request would independently query the database
2. If the location isn't cached, each request would trigger external API calls
3. This leads to:
   - Increased database load
   - Unnecessary API calls to external services
   - Higher costs
   - Potential rate limiting issues
   - Inconsistent results if the external service returns different data

The Request Tracking Pattern solves these issues by ensuring that only one request is processed while others wait for the result.

## Implementation in C#

Let's look at a practical implementation using C#:

```csharp
public class InMemoryRequestTracker : IRequestTracker
{
    private readonly ConcurrentDictionary<string, Task> _inFlightRequests = new();
    private readonly ILogger<InMemoryRequestTracker> _logger;

    public async Task<T> TrackRequest<T>(string key, Func<Task<T>> requestFunc)
    {
        var taskCompletionSource = new TaskCompletionSource<T>();

        var existingTask = _inFlightRequests.GetOrAdd(key, taskCompletionSource.Task);

        if (existingTask == taskCompletionSource.Task)
        {
            // This is the first request for this key
            try
            {
                var result = await requestFunc();
                taskCompletionSource.SetResult(result);
                _inFlightRequests.TryRemove(key, out _);
                return result;
            }
            catch (Exception ex)
            {
                taskCompletionSource.SetException(ex);
                _inFlightRequests.TryRemove(key, out _);
                throw;
            }
        }

        // Another request is already processing this key
        return await (Task<T>)existingTask;
    }
}
```

### Key Components

1. **ConcurrentDictionary**: Thread-safe collection to store in-flight requests
2. **TaskCompletionSource**: Allows creating a Task that can be completed manually
3. **GetOrAdd**: Atomic operation to either get an existing task or add a new one

## Usage Example

Here's how to use the request tracker in a real-world scenario:

```csharp
public class CellTowerLocationService
{
    private readonly IRequestTracker _requestTracker;
    private readonly ILocationProvider _locationProvider;

    public async Task<LocationResult> GetLocationAsync(CellTower cellTower)
    {
        var key = $"{cellTower.MobileCountryCode}-{cellTower.MobileNetworkCode}-{cellTower.LocationAreaCode}-{cellTower.CellId}";

        return await _requestTracker.TrackRequest(key, async () =>
        {
            // This code will only execute once for concurrent requests with the same key
            return await _locationProvider.GetLocationAsync(cellTower);
        });
    }
}
```

## Benefits

1. **Resource Optimization**: Reduces database and API calls
2. **Cost Reduction**: Fewer external API calls mean lower costs
3. **Consistency**: All concurrent requests get the same result
4. **Rate Limiting**: Helps stay within external API rate limits
5. **Performance**: Reduces system load and improves response times

## Testing with Load Tests

To verify the effectiveness of request tracking, we can use load testing tools like `hey` or JMeter. Here's how to set up a test:

### Using hey (Simple Load Testing)

```bash
hey -n 100 -c 10 -m POST \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key" \
  -d '{"cellId": 170402199,"locationAreaCode": 35632,"mobileCountryCode": 310,"mobileNetworkCode": 410}' \
  http://localhost:5093/api/CellTower/location
```

This test will:
- Send 100 total requests
- With 10 concurrent users
- All requesting the same cell tower location

### Expected Results

With request tracking:
- Only one database query will be executed
- Only one external API call will be made
- All 100 requests will complete successfully
- Response times will be consistent

Without request tracking:
- Multiple database queries would occur
- Multiple external API calls would be made
- Potential rate limiting issues
- Inconsistent response times

## Best Practices

1. **Key Generation**: Use meaningful, unique keys for tracking requests
2. **Error Handling**: Ensure proper cleanup in case of failures
3. **Timeouts**: Consider adding timeout mechanisms
4. **Monitoring**: Track metrics for request tracking effectiveness
5. **Cleanup**: Implement cleanup for stale entries

## Conclusion

The Request Tracking Pattern is a powerful tool in the .NET developer's arsenal for building efficient, scalable applications. By preventing duplicate requests, it helps optimize resource usage, reduce costs, and improve system reliability.

Implementing this pattern is straightforward with C#'s built-in concurrent collections and task-based programming model. Combined with proper load testing, it can significantly improve your application's performance and cost-effectiveness.

## Further Reading

- [Data Structures for Parallel Programming](https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/data-structures-for-parallel-programming)
- [TaskCompletionSource Documentation](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.taskcompletionsource)
- [ASP.NET Core Performance Best Practices](https://docs.microsoft.com/en-us/aspnet/core/performance/performance-best-practices)
