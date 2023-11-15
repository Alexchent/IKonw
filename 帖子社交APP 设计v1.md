# 帖子社交APP 设计v1

## mongodb 持久化存储

### 用户标签集合
```
userid:     [userid]
schoolid:   [schoolid]
provinceid: [provinceid]
friends:    {[userid],[userid]}
tags:       {[tag]=>[weight], [tag]=>[weight]}
```

### 用户7日内已推集合
userid: [userid]
newids: {[newid] => [time], [newid] => [time]}

### 用户已读集合
userid: [userid]
newids: {[newid] => [time], [newid] => [time]}

### 新闻标签集合
```
newid:  [newid]
userid: [userid]
schoolid:   [schoolid]
provinceid: [provinceid]
tags:       {[tag], [tag]}
```

## redis维护以下集合
{} 为hash标签，其内部均为实际变量

| key | 用途 |
|---|---|
| {userid}_history_push | 用户**已推新闻**集合（7日内）|
| hot_{channel} | 热门**频道**集合 |
| {userid}_new_[分类] | 用户个标签未推(7日内)news集合 |
| [userid]_new_friend_write | 用户的好友为作者(7日内)的news集合 |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |
|  |  |