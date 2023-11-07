# 笔记：A reinforcement learning approach for the scheduling of live migration from under utilised hosts

- contribution：
  - network aware live migration strategy 通过监控网络带宽
  - agent to learn an optimal time to schedule a virtual machine migration

- introduction:

  - host处于under-utilised或者over-utilised时VM就应该被迁移了
  - 但是由于迁移VM有不可忽视的overhead，host会停留在这种under/over-utilised状态一段时间
  - 网络资源的饱和使得迁移VM的时间成本增大
  - 什么时候去做VM的迁移是一个重要的因素；分析网络的traffic也更有利于高效利用集群内有限的资源

- 论文主要工作：

  - 通过监控当前demand level of a network，利用RL agent更好的利用live migration
  - 该自动agent会挑选适合当前网络资源消耗的最佳迁移VM的时间
  - 以此来减轻网络资源的压力，降低迁移VM的时间，避免未来的网络拥堵

- 论文前提：

  - When a host becomes under-utilised a migration will not take place unless specific data centre constraints are approved.（同样适用于container和kubernetes吗？）

- 建模：

  - 多环境：
    - s_t: 当前VM对于host上RAM的利用率(%)
    - b_t: 当前数据中心的带宽使用数据(越大迁移VM越快)
    - a_t: wait--->nothing happens/migrate--->scheduled group of VM migrations
  - 奖励函数：
    - R(s_t,b_t,a): SLAV and power penalty
      - Migrate: SLAV
      - Wait: power penalty

  