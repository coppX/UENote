# 相关性
游戏中的场景很大，有很多actor，但是并不是玩家能看到所有actor，所以其他看不见的actor就不需要从服务器复制到这个玩家客户端，极大的优化了网络传输压力，服务器就只需要同步和当前客户端actor相关的actor数据，哪些actor需要同步，这就是相关性。
# 同步范围
- AlwaysRelevant
- OnlyRelevantToOwner