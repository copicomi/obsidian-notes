[[【分布式】spanner.pdf]]
# Abstract
spanner 是 Google 开发的大规模、多版本、globle分布式、复制同步的数据库

本文描述 spanner 的架构、特性与设计选择，以及独特又强大的 time clock API

# 1 Introduction
spanner 脱胎于 bigtable的版本化KV服务，是一个多版本时态数据库

spanner 的优势在于
1. 全球范围内的数据分片
2. 客户端之间可以自动进行数据转移与故障转移
3. 基于 TrueTime API 提供的时间戳，实现高度的线性一致性
4. 同时可以监测数据流的使用情况，在不同center间进行负载平衡

# 2 Implementation

