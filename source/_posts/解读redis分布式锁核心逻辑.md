---
title: 解读redis分布式锁核心逻辑
date: 2024-07-30 10:07:27
tags:
---
   ```lua
  local insertable = false; local value = redis.call('hget', KEYS[1], ARGV[5]); if value == false then insertable = true; else local t, val = struct.unpack('dLc0', value); local expireDate = 92233720368547758; local expireDateScore = redis.call('zscore', KEYS[2], ARGV[5]); if expireDateScore ~= false then expireDate = tonumber(expireDateScore) end; if t ~= 0 then local expireIdle = redis.call('zscore', KEYS[3], ARGV[5]); if expireIdle ~= false then expireDate = math.min(expireDate, tonumber(expireIdle)) end; end; if expireDate <= tonumber(ARGV[1]) then insertable = true; end; end; if insertable == true then if tonumber(ARGV[2]) > 0 then redis.call('zadd', KEYS[2], ARGV[2], ARGV[5]); else redis.call('zrem', KEYS[2], ARGV[5]); end; if tonumber(ARGV[3]) > 0 then redis.call('zadd', KEYS[3], ARGV[3], ARGV[5]); else redis.call('zrem', KEYS[3], ARGV[5]); end; local maxSize = tonumber(redis.call('hget', KEYS[7], 'max-size')); if maxSize ~= nil and maxSize ~= 0 then     local currentTime = tonumber(ARGV[1]);     local lastAccessTimeSetName = KEYS[5]; local mode = redis.call('hget', KEYS[7], 'mode'); if mode == false or mode == 'LRU' then redis.call('zadd', lastAccessTimeSetName, currentTime, ARGV[5]); end;     local cacheSize = tonumber(redis.call('hlen', KEYS[1]));     if cacheSize >= maxSize then         local lruItems = redis.call('zrange', lastAccessTimeSetName, 0, cacheSize - maxSize);         for index, lruItem in ipairs(lruItems) do             if lruItem and lruItem ~= ARGV[5] then                 local lruItemValue = redis.call('hget', KEYS[1], lruItem);                 redis.call('hdel', KEYS[1], lruItem);                 redis.call('zrem', KEYS[2], lruItem);                 redis.call('zrem', KEYS[3], lruItem);                 redis.call('zrem', lastAccessTimeSetName, lruItem);                 if lruItemValue ~= false then                 local removedChannelName = KEYS[6]; local ttl, obj = struct.unpack('dLc0', lruItemValue);                    local msg = struct.pack('Lc0Lc0', string.len(lruItem), lruItem, string.len(obj), obj);                redis.call('publish', removedChannelName, msg); end;             end;         end;     end; if mode == 'LFU' then redis.call('zincrby', lastAccessTimeSetName, 1, ARGV[5]); end; end; local val = struct.pack('dLc0', tonumber(ARGV[4]), string.len(ARGV[6]), ARGV[6]); redis.call('hset', KEYS[1], ARGV[5], val); local msg = struct.pack('Lc0Lc0', string.len(ARGV[5]), ARGV[5], string.len(ARGV[6]), ARGV[6]); redis.call('publish', KEYS[4], msg); return nil; else local t, val = struct.unpack('dLc0', value); redis.call('zadd', KEYS[3], t + ARGV[1], ARGV[5]); return val; end; 
   ```

这段Lua脚本用于Redis缓存的管理和插入操作，主要功能包括：
- 判断是否需要插入或更新缓存。
- 处理缓存的过期逻辑。
- 实现基于LRU（最近最少使用）或LFU（最少使用频率）策略的缓存淘汰。
- 在缓存项插入或更新时发布相关消息。

具体的解读如下：

1. **初始化和获取当前值**：
   ```lua
   local insertable = false
   local value = redis.call('hget', KEYS[1], ARGV[5])
   ```

2. **判断当前缓存项是否存在**：
    - 如果不存在，则设置`insertable`为`true`，表示可以插入新的缓存项。
    - 如果存在，继续判断其过期时间。
   ```lua
   if value == false then 
       insertable = true 
   else
       local t, val = struct.unpack('dLc0', value)
       local expireDate = 92233720368547758
       local expireDateScore = redis.call('zscore', KEYS[2], ARGV[5])
       if expireDateScore ~= false then expireDate = tonumber(expireDateScore) end
       if t ~= 0 then
           local expireIdle = redis.call('zscore', KEYS[3], ARGV[5])
           if expireIdle ~= false then expireDate = math.min(expireDate, tonumber(expireIdle)) end
       end
       if expireDate <= tonumber(ARGV[1]) then insertable = true end
   end
   ```

3. **处理可插入的情况**：
    - 更新或删除过期时间。
    - 检查缓存大小，并根据LRU或LFU策略进行淘汰。
   ```lua
   if insertable == true then 
       if tonumber(ARGV[2]) > 0 then 
           redis.call('zadd', KEYS[2], ARGV[2], ARGV[5])
       else 
           redis.call('zrem', KEYS[2], ARGV[5]) 
       end
       if tonumber(ARGV[3]) > 0 then 
           redis.call('zadd', KEYS[3], ARGV[3], ARGV[5])
       else 
           redis.call('zrem', KEYS[3], ARGV[5]) 
       end
       local maxSize = tonumber(redis.call('hget', KEYS[7], 'max-size'))
       if maxSize ~= nil and maxSize ~= 0 then
           local currentTime = tonumber(ARGV[1])
           local lastAccessTimeSetName = KEYS[5]
           local mode = redis.call('hget', KEYS[7], 'mode')
           if mode == false or mode == 'LRU' then 
               redis.call('zadd', lastAccessTimeSetName, currentTime, ARGV[5])
           end
           local cacheSize = tonumber(redis.call('hlen', KEYS[1]))
           if cacheSize >= maxSize then
               local lruItems = redis.call('zrange', lastAccessTimeSetName, 0, cacheSize - maxSize)
               for index, lruItem in ipairs(lruItems) do
                   if lruItem and lruItem ~= ARGV[5] then
                       local lruItemValue = redis.call('hget', KEYS[1], lruItem)
                       redis.call('hdel', KEYS[1], lruItem)
                       redis.call('zrem', KEYS[2], lruItem)
                       redis.call('zrem', KEYS[3], lruItem)
                       redis.call('zrem', lastAccessTimeSetName, lruItem)
                       if lruItemValue ~= false then
                           local removedChannelName = KEYS[6]
                           local ttl, obj = struct.unpack('dLc0', lruItemValue)
                           local msg = struct.pack('Lc0Lc0', string.len(lruItem), lruItem, string.len(obj), obj)
                           redis.call('publish', removedChannelName, msg)
                       end
                   end
               end
           end
           if mode == 'LFU' then 
               redis.call('zincrby', lastAccessTimeSetName, 1, ARGV[5]) 
           end
       end
       local val = struct.pack('dLc0', tonumber(ARGV[4]), string.len(ARGV[6]), ARGV[6])
       redis.call('hset', KEYS[1], ARGV[5], val)
       local msg = struct.pack('Lc0Lc0', string.len(ARGV[5]), ARGV[5], string.len(ARGV[6]), ARGV[6])
       redis.call('publish', KEYS[4], msg)
       return nil
   ```

4. **处理不可插入的情况**：
    - 更新过期时间，并返回现有值。
   ```lua
   else 
       local t, val = struct.unpack('dLc0', value)
       redis.call('zadd', KEYS[3], t + ARGV[1], ARGV[5])
       return val
   end
   ```

### 参数和键值解释

- `KEYS[1]`: 哈希表存储缓存项。
- `KEYS[2]`: 有序集合存储过期时间。
- `KEYS[3]`: 有序集合存储空闲时间。
- `KEYS[4]`: 频道名称，用于发布缓存项变更消息。
- `KEYS[5]`: 有序集合存储最近访问时间（LRU模式）。
- `KEYS[6]`: 频道名称，用于发布被淘汰的缓存项。
- `KEYS[7]`: 存储缓存配置（如最大尺寸、模式等）。

- `ARGV[1]`: 当前时间戳。
- `ARGV[2]`: 新的过期时间。
- `ARGV[3]`: 新的空闲时间。
- `ARGV[4]`: 新缓存项的TTL。
- `ARGV[5]`: 缓存项的键名。
- `ARGV[6]`: 缓存项的值。