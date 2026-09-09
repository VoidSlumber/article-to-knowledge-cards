# 分布式系统中的共识问题

## 什么是共识

分布式系统中的多个节点需要对某件事达成一致的值，这就是共识问题。最常见的场景包括：选主（leader election）、复制日志（replicated log）、原子广播（atomic broadcast）。共识算法要解决的核心问题是：在部分节点故障或消息丢失/延迟的情况下，剩余节点仍能就某个值达成一致。

## FLP 不可能定理

1985 年，Fischer、Lynch 和 Paterson 证明了：在网络不可靠（即使只可能丢一条消息）且至少一个进程可能故障的异步分布式系统中，不存在一个能在有限时间内保证所有正确进程达成一致的确定性算法。这就是 FLP 不可能定理。它不是说不能做共识，而是说在异步模型下不能保证有限时间内终止。

实践中绕过 FLP 的方法有两种：超时假设（把异步系统近似成同步系统，给消息一个超时）和随机化（让算法以概率 1 终止）。

## Paxos 协议

Paxos 是 Lamport 在 1989 年提出的经典共识算法。Basic Paxos 包含两个阶段：

阶段一（Prepare）：Proposer 选择一个全局递增的提案号 n，向多数派 Acceptor 发送 Prepare(n)。Acceptor 收到后承诺不再接受任何编号小于 n 的提案，并返回它已接受的编号最大的提案（如果有）。

阶段二（Accept）：如果 Proposer 收到多数派的 Prepare 响应，就发出 Accept(n, value) 请求，其中 value 是响应中编号最大的提案的值（如果没有则自选）。Acceptor 收到 Accept 后，只要没承诺过更高的编号就接受它。

Paxos 的难点不是协议本身，而是工程实现：多 Proposer 之间的活锁、Proposer 角色漂移、日志空洞等。

## Raft 协议

Raft 是为了可理解性而设计的共识算法。它把共识问题拆成三个子问题：领导选举（leader election）、日志复制（log replication）、安全性（safety）。

Raft 中任何时刻最多有一个 Leader，由 Follower 在选举超时（election timeout）内没收到心跳时变成 Candidate 并发起选举。Candidate 的票数超过半数即成为 Leader。

日志复制由 Leader 负责：客户端请求到达 Leader 后，Leader 把日志条目追加到本地日志并复制到所有 Follower；当日志条目在多数派节点上被复制后即视为已提交。

Raft 的关键不变量是：Leader 在某个任期内只会追加日志条目，且日志匹配性质保证 Leader 和 Follower 在某个索引上的日志条目相同时，之前所有日志都相同。

## 共识 vs 一致性

注意区分"共识"（consensus，节点对某个值达成一致）和"一致性"（consistency，分布式系统对外呈现的状态模型）。前者是底层协议问题，后者是面向用户的数据模型。强一致性往往需要共识或类似机制来保证。