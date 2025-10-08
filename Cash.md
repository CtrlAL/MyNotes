```js
using Microsoft.Extensions.Caching.Memory;
using System.Collections.Concurrent;

namespace BL.Implementations;
public class WaitToFinishMemoryCache<TItem>
{
    private readonly IMemoryCache _cache;
    private readonly TimeSpan _chachingTime;
    private readonly ConcurrentDictionary<object, SemaphoreSlim> _locks;

    public WaitToFinishMemoryCache(IMemoryCache memoryCache)
    {
        _cache = memoryCache;
        _chachingTime = TimeSpan.FromMinutes(2);
        _locks = new ConcurrentDictionary<object, SemaphoreSlim>();
    }

    public async Task<TItem> GetOrCreate(object key, Func<Task<TItem>> createItem)
    {
        TItem cacheEntry;

        if (!_cache.TryGetValue(key, out cacheEntry))
        {
            SemaphoreSlim mylock = _locks.GetOrAdd(key, k => new SemaphoreSlim(1, 1));

            await mylock.WaitAsync();
            try
            {
                if (!_cache.TryGetValue(key, out cacheEntry))
                {
                    cacheEntry = await createItem();
                    _cache.Set(key, cacheEntry, _chachingTime);
                }
            }
            finally
            {
                mylock.Release();
            }
        }

        return cacheEntry;
    }
}
```
