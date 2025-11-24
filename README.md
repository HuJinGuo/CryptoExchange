# CryptoExchange 项目介绍

## 项目概述
CryptoExchange 是一个基于 Go 语言的加密货币交易所后端系统，专注于 ETH/USDT 交易对的撮合引擎和流动性管理。项目源于 Anthony 的教学仓库（anthonygg/cryptoExchange），由 HuJinGuo  fork 并优化，用于模拟真实交易所环境，包括自动做市商机器人和链上结算。仓库地址：https://github.com/HuJinGuo/CryptoExchange。当前为开发/测试阶段，适合学习 Web3 开发和量化交易。

## 核心功能
- **订单簿撮合引擎**：支持限价单（Limit Order）和市价单（Market Order），价格优先 + 时间优先匹配。
- **自动做市商（Market Maker）**：机器人提供双边流动性，紧贴最优报价挂单，防止盘口空盘，支持 Binance 价格锚定。
- **链上结算**：撮合后自动转账 ETH（基于 Ethereum/Anvil 本地链），模拟真实资金流动。
- **API 接口**：RESTful HTTP API（Echo 框架），包括下单、查询盘口、撤单、交易历史。
- **测试模拟**：内置吃盘机器人，模拟市场波动和压力测试。

## 技术栈
- **语言/框架**：Go 1.20+，Echo v4（Web 框架）。
- **区块链**：go-ethereum（ethclient），支持 Hardhat/Anvil 本地链。
- **订单管理**：纯内存 Orderbook，带 RWMutex 并发安全。
- **外部依赖**：logrus（日志）、crypto（签名）、Binance API（价格拉取）。
- **模块结构**：
  - `orderbook/`：撮合核心。
  - `server/`：HTTP API + Exchange 逻辑。
  - `client/`：SDK 客户端。
  - `maker/`：做市商机器人。

## 安装与运行
1. **环境准备**：
   - Go 1.20+。
   - Node.js + Hardhat/Anvil（本地链）：`npx anvil` 启动（默认端口 8545）。
   - 依赖：`go mod tidy`。

2. **克隆仓库**：
   ```
   git clone https://github.com/HuJinGuo/CryptoExchange.git
   cd CryptoExchange
   ```

3. **运行**：
   ```
   make run
   ```
   - 启动服务器（:3000）。
   - 自动注册测试用户（ID: 7, 8, 666）。
   - 启动做市商（UserID=666，提供 15 ETH 深度）。
   - 启动吃盘机器人（UserID=7，模拟交易）。

4. **测试 API**：
   - 下限价单：`POST /order`（JSON: `{ "UserID": 7, "Type": "LIMIT", "Bid": true, "Size": 1.0, "Price": 2000, "Market": "ETH" }`）。
   - 查询盘口：`GET /book/ETH`。
   - 撤单：`DELETE /order/{id}`。

## 已知问题与优化
- **链上转账**：测试环境易耗尽 gas（建议注释 `transferETH` 仅模拟撮合）。
- **Panic 处理**：市价单空盘时 panic 已修复为返回错误。
- **扩展建议**：添加余额系统、手续费、USDT 支持、多交易对。

## 仓库统计
- Stars/Forks：0（新 fork 项目）。
- 最后更新：2025-11-24。
- License：MIT（[继承原仓库](https://github.com/anthdm/crypto-exchange)）。

后续改进计划：
### CryptoExchange 项目后续改进路线图（从玩具 → 可商用）

| 优先级 | 功能模块 | 具体改进点 | 预计难度 | 预计耗时 | 备注 |
|------|----------|----------|----------|----------|------|
| ★★★★★ | 核心安全 | 1. 加入用户余额系统（ETH 余额表）<br>2. 下单前检查余额 + 挂单锁仓<br>3. 市价单禁止超额成交 | 中 | 1-2 天 | 杜绝负余额、刷钱漏洞 |
| ★★★★★ | 稳定性 | 4. 彻底移除 `panic("not enough volume")`，全部改为返回错误<br>5. 做市商每次循环前撤掉自己所有旧单 | 低 | 4 小时 | 彻底永不崩溃 |
| ★★★★ | 链上交互 | 6. 链上转账金额单位修复（×1e18）<br>7. 可开关真实链上转账（开发时关闭）<br>8. 增加交易手续费（0.05%-0.2%） | 低 | 4 小时 | 防止 gas 耗尽 |
| ★★★★ | 做市商增强 | 9. 做市商支持多档挂单（3~5 档深度）<br>10. 动态调整 offset（波动率越大，价差越大）<br>11. 支持从多个交易所拉价格（Coinbase、OKX） | 中 | 2-3 天 | 更真实流动性 |
| ★★★★ | 多交易对 | 12. 支持 BTC/USDT、SOL/USDT 等<br>13. Market 枚举化，订单簿 map[Market] | 中 | 1-2 天 | 从玩具变真交易所 |
| ★★★ | 用户系统 | 14. 注册/登录 API（JWT）<br>15. API Key 认证<br>16. 私有订单查询 | 中 | 2-3 天 |  |
| ★★★ | 持久化 | 17. 用 Redis/PostgreSQL 存储订单、余额、交易历史<br>18. 重启后恢复订单簿 | 高 | 5-7 天 |  |
| ★★★ | WebSocket | 19. 实时推送深度、成交、用户订单状态 | 高 | 3-5 天 | 前端实时行情必备 |
| ★★ | 前端 | 20. React/Vue 简易交易界面（盘口、下单、K 线） | 中 | 5-10 天 | 可视化展示 |
| ★★ | 风控 | 21. 单用户频率限制<br>22. 最大单笔限制<br>23. 异常撤单监控 | 中 | 2 天 | 防止机器人刷 |
| ★★ | 监控告警 | 24. Prometheus + Grafana 监控<br>25. 盘口深度、价差、gas 异常告警 | 中 | 3 天 |  |
| ★ | 高级功能 | 26. 支持 Maker 费率优惠<br>27. 止损/止盈单<br>28. 资金费率（永续合约） | 高 | 10+ 天 | 接近真实交易所 |


