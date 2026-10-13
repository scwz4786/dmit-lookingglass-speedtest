# DMIT 三网测速：官方测试 IP、Looking Glass 用法与回程路由怎么看，下单前自己先验一遍

买 DMIT 之前先 ping 一下，几乎是国内 VPS 圈的固定动作。但绝大多数人只做了一半：在自己电脑上 ping 一个 IP，看到延迟 140ms 就下单，然后发现晚高峰看视频还是卡。问题出在"三网测速"这四个字被简化成了"ping 一下"。

真正要看的东西分两段——数据从你家宽带出去到机房（去程），以及从机房回到你家宽带（回程）。回程决定你下载、看视频、拉文件爽不爽，而这恰恰是你本地 ping 测不到的部分。DMIT 全程提供官方的 Looking Glass 测速平台，就是来解决这个问题的。

## 三网测速到底在测什么

"三网"指电信、联通、移动。同一个机房，数据包走电信的骨干网和走移动的骨干网，可能完全是两条路：一条直连，一条绕美绕日；一条晚高峰稳如老狗，一条晚上八点开始丢包。

所以三网测速的目标是分别回答三个问题：

- 延迟有多少，全国还是只有华东好
- 丢包和抖动如何，尤其是晚高峰
- 路由是不是直连，中间有没有莫名其妙的绕行

**去程和回程必须分开看。** 去程是你能测的（本地 ping / traceroute），回程是决定实际体验的（下载速度、流媒体拖动进度条）。想测回程只有两个办法：从服务器侧往外发起探测，或者用第三方多地拨测平台。DMIT 的 Looking Glass 属于前者。

## DMIT 官方测速入口：Looking Glass 怎么用

DMIT 的网络测试平台在 `lg.dmit.sh`，覆盖洛杉矶、香港、东京全部在售节点。打开之后你看到的是服务器视角：输入你自己的公网 IP 或者任意目标地址，点击运行，测试从机房发出去。

支持的命令有三类：

- **Ping**：看基础延迟和丢包，判断连通性
- **Traceroute**：看数据包逐跳经过哪些骨干节点
- **MTR**：Ping + 路由追踪的合体，能定位具体是哪一跳在丢包

页面里还能跑带宽测试，直接看这个节点到国际方向的下行/上行吞吐。实测时加载速度还行，结果可以截图保存，方便对比几个节点再决定。

地址栏也支持带上参数直达具体节点，`?server=lax-pro`、`?server=hkg-pro`、`?server=tyo-pro` 这类写法对应 Pro 线路，`lax-t1`、`hkg-t1`、`tyo-t1` 对应国际线路。省得在节点列表里找半天。

## DMIT 各机房测试 IP 一览

如果你只想快速 ping 一下感受延迟，可以直接用下面这些测试 IP。这份清单来自公开的测速节点整理，同一份数据在不同站点上有细微出入（DMIT 会调整测试机），所以**拿不准的时候以 Looking Glass 页面里的实时节点为准**。

| 机房与线路 | 线路特征 | 测试 IPv4 |
| --- | --- | --- |
| 洛杉矶 LAX.Pro | 三网 CN2 GIA 回程 | 154.17.30.96 |
| 洛杉矶 LAX.EB | CMIN2 + 9929 | 154.17.229.24 |
| 洛杉矶 LAX.T1 | 国际 BGP，无大陆优化 | 174.136.204.135 |
| 洛杉矶 Pro 高防 | CN2 GIA + Cloudflare Magic Transit | 45.88.194.187 |
| 香港 HKG.Pro | 三网 CN2 GIA 回程 | 103.117.100.199 |
| 香港 HKG.EB | 三网 CMIN2 | 154.12.188.3 |
| 香港 HKG.T1 | 国际 BGP | 154.12.176.28 |
| 东京 TYO.Pro | 三网 CN2 GIA 回程 | 154.12.190.32 |
| 东京 TYO.EB | 三网 CMI / CMIN2 | 154.31.116.3 |
| 东京 TYO.T1 | 国际线路 | 154.31.112.5 |

三地机房之间目前已经开通内网互通（香港–美国、日本–美国、香港–日本），跨机房传数据走内网，不过一样计入流量额度，这点别忽略。

## 三条线路，测速结果分别应该长什么样

DMIT 的线路分成三档，测出来的东西完全不同。看路由的时候别只盯延迟数字，直接看 AS 号更靠谱。

### Premium（Pro）：三网回程 CN2 GIA

线路构成：电信去程走 AS4809 CN2 GIA，联通去程直连 AS4837，移动去程经香港移动 AS58453，**三网回程统一走 AS4809**。

第三方实测里，洛杉矶 AS3.Pro.TINY 的国内三网延迟大致是上海电信 141ms、联通 139ms、移动 129ms，回程在多地均被识别为 AS906 → CN2 GIA。这就是 Pro 系列的典型特征：数字不算惊艳，但路径干净。

东京 Pro 的表现更好看一些。从服务器侧回国内的追踪数据里，东京到你上海大约 30ms，北京 55–75ms，广东 52–95ms（不同运营商差得比较明显），全程 AS906 → AS23764（电信 CTGNet）→ AS4809 CN2 骨干，再进入各家的国内网络。

香港 Pro 是三地里延迟最低的，公开测评里引用的全国三网平均延迟在 43ms 上下。代价是价格，香港 Pro 是三条产品线中最贵的一档。

### Eyeball（EB）：9929 + CMIN2 的组合拳

EB 的逻辑是"用次一级的价格，换更好的联通/移动体验"：

- 电信：在 AS9929 与 AS58807（CMIN2）之间做负载分配
- 联通：主要走 AS9929（联通精品网 / CUII）
- 移动：主要走 AS58807 CMIN2

如果你的宽带是联通或移动，这条线通常比 Pro 更划算——价格更低，流量和带宽反而给得更多。电信用户的话，EB 的电信回程不是纯 CN2 GIA，对延迟极端敏感的话还是看 Pro。

东京 EB 的架构稍有不同：三网回程走 CMI，电信和联通去程是 CTG GIA，移动去程走 CMI。

### Tier 1（T1）：国际线路，别拿它测国内

T1 是纯国际互联线路，**没有任何针对中国大陆的优化**。给日本 T1 做路由追踪，你会看到电信双程 NTT、联通双程 NTT、偶尔绕美走 Lumen、移动走 CMI。

所以用"三网测速"的思路去评价 T1 是跑偏了。它适合的场景是拿来做落地机、拉美国本土家宽、或者亚太中转。真要直连用，电信用户的体验大概率比美国线路还差。

流量跑完之后 T1 不会停机，而是降到低速继续跑：T1.WEE 里香港和东京限速到 50Mbps，洛杉矶限速到 100Mbps，其余档位按配置给到 200Mbps 甚至 500Mbps。这一点对流量偶尔超额的场景挺实用。

## 在服务器上跑一遍：三网测速脚本

拿到机器之后，下面这几个工具基本能覆盖全部需求。

**NextTrace** 是目前最顺手的路由追踪工具，会直接标出每一跳的 AS 归属：

bash
# 一键安装
curl -sL nxtrace.org/nt | bash

# 四地三网快速回程测试（北京/上海/广州/杭州）
nexttrace -f

# TCP 模式，绕开骨干网对 ICMP 的限速
nexttrace -T -p 443 1.1.1.1


传统 traceroute 遇到最大的一片星号，很多时候是骨干路由器对 ICMP TTL 报文做了限速甚至丢弃。换成 TCP SYN 模式，探测包长得像正常的业务握手，被拦的概率低很多。

想一次把延迟、回程路由、国内三网测速都跑完，可以用 NetQuality：

bash
bash <(curl -Ls https://Check.Place) -N


它给的报告包括中国大陆 31 个省级行政区的三网回程延迟、基于 SPEEDTEST.NET 的国内三网网速，以及完整的 NextTrace 回程路由。支持 IPv4/IPv6 双栈，也能跳过不想要的章节（`-S 1234567`）。

做完路由，别忘了用 iperf3 分别找电信、联通、移动的节点跑一轮实际吞吐。路由路径漂亮不等于带宽一定达标，这两件事要分开验证。

## 测速时最容易踩的几个坑

**别在下午测。** 工作时段的三网线路大多是空闲的，随便一台机器看着都挺漂亮。真正的差别要到晚上八点到十一点才显形，这一步没有捷径。

**区分单线程和多线程。** 美西线路常见单线程 200Mbps 的软限制，实测稳定在 150–180Mbps 就是正常的。如果你跑的是多线程 iperf3 冲到 900Mbps 然后觉得单线程慢是商家虚标，那属于测错指标了。

**ping 不通不代表线路差。** 中间几跳显示星号，多数是被限速或丢弃 ICMP。用 MTR 看丢包是否持续到最后一跳才是有意义的判断。

**测试 IP 会变，测试机也不是你买的那台。** 同一个机房不同 IP 段的质量可能有差异，测试机和你最终拿到的实例也不保证在同一台宿主机上。测速只能帮你判断线路档次，不能帮你预测具体这一台的表现。

**看清算费口径。** DMIT 各系列的流量计算方式不完全一致，下单前在产品页确认一遍是单向还是双向计费、超量后的限速值是多少，比事后投诉有用。

## DMIT 主要在售套餐与价格对比

DMIT 的产品线用固定格式命名：`机房 + 硬件平台 + 线路 + 档位`，比如 `LAX.AN5.Pro.TINY`。硬件平台代号（AS3 = EPYC 7003、AN4 = EPYC 9004、AN5 = EPYC 9005）经常省略，平时直接说"洛杉矶 Pro TINY"就够。

需要提醒的是，DMIT 的 SKU 和库存轮换很快，同一条线路会交替出现 AS3 和 AN5 两个硬件平台，特价限量款补货即秒。下表整理的是主流在售档位，下单前请以官网产品页的实时配置和价格为准。

### Premium 系列（三网 CN2 GIA 回程）

| 套餐 | 配置（CPU/内存/硬盘） | 带宽 / 月流量 | 价格 | 购买 |
| --- | --- | --- | --- | --- |
| LAX.AN5.Pro.TINY | 1核 / 2G / 20G | 1Gbps / 1TB | $12.98/月 | [查看该套餐](https://www.dmit.io/aff.php?aff=12046&pid=100) |
| LAX.AN5.Pro.Pocket | 2核 / 2G / 40G | 4Gbps / 1.5TB | $18.9/月 | [查看该套餐](https://www.dmit.io/aff.php?aff=12046&pid=137) |
| LAX.AN5.Pro.STARTER | 2核 / 2G / 80G | 10Gbps / 3TB | $38.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.Pro.MINI | 2核 / 4G / 80G | 10Gbps / 5TB | $76.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.Pro.MICRO | 4核 / 4G / 160G | 10Gbps / 7TB | $99.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.Pro.MEDIUM | 4核 / 8G / 160G | 10Gbps / 14TB | $219.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.Pro.LARGE | 8核 / 16G / 320G | 10Gbps / 25TB | $459.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.Pro.GIANT | 8核 / 24G / 640G | 10Gbps / 50TB | $839.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.Pro.TINY | 1核 / 1G / 20G | 1Gbps / 500G | $39.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.Pro.STARTER | 1核 / 2G / 40G | 1Gbps / 1TB | $79.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.Pro.MINI | 2核 / 2G / 60G | 1Gbps / 1.5TB | $126.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.Pro.MICRO | 2核 / 4G / 80G | 1Gbps / 2TB | $179.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AS3.Pro.MEDIUM | 4核 / 4G / 160G | 1Gbps / 2.5TB | $239.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.MINI | 2核 / 2G / 80G | 1Gbps / 1.5TB | $149.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.MICRO | 2核 / 4G / 160G | 1Gbps / 2TB | $159.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.MEDIUM | 4核 / 4G / 160G | 1Gbps / 2.5TB | $180/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.LARGE | 4核 / 8G / 320G | 1Gbps / 3TB | $240/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.AN5.Pro.GIANT | 8核 / 16G / 640G | 1Gbps / 6TB | $500/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| TYO.AS3.Pro.TINY | 1核 / 1G / 20G | 1Gbps / 500G | $21.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |

东京 Pro 往上的档位（STARTER / MINI / MICRO / MEDIUM / LARGE / GIANT）同样在售，配置和价格按档递增，具体以产品页为准。这三个机房的 Pro 系列，流量超出额度后不会直接停机。

### Eyeball 系列（CMIN2 + 9929）

| 套餐 | 配置（CPU/内存/硬盘） | 带宽 / 月流量 | 价格 | 购买 |
| --- | --- | --- | --- | --- |
| LAX.AN5.EB.TINY | 1核 / 2G / 20G | 2Gbps / 1.5TB | $12.98/月 | [查看该套餐](https://www.dmit.io/aff.php?aff=12046&pid=189) |
| LAX.AN5.EB.Pocket | 2核 / 2G / 40G | 4Gbps / 3TB | $18.9/月 | [查看该套餐](https://www.dmit.io/aff.php?aff=12046&pid=190) |
| LAX.AN5.EB.STARTER | 2核 / 2G / 80G | 10Gbps / 5TB | $38.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.EB.MINI | 4核 / 4G / 80G | 10Gbps / 10TB | $76.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.EB.MICRO | 4核 / 4G / 160G | 10Gbps / 14TB | $99.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.AN5.EB.MEDIUM | 6核 / 8G / 160G | 10Gbps / 30TB | $199.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |

同价位的 Pro 和 EB（比如都是 $12.98/月、$18.9/月）配置完全相同，差别只在线上：Pro 换 CN2 GIA，EB 换双倍甚至更多流量和更大带宽。这个取舍很直白——你是电信用户且对延迟敏感就选 Pro，联通移动用户或者流量吃得多就选 EB。

此外 DMIT 会不定期放一批特价限量款，常见的有洛杉矶 EB 的 WEE（$39.9/年）、CORONA（$49.9/年）、FONTANA（$88.88/季）以及 Pro 的 MALIBU（$49.9/年）。这类机型补货后往往很快售罄，看到有货基本就该直接下单，犹豫一轮就没了。

### Tier 1 系列（国际线路）

| 套餐 | 配置（CPU/内存/硬盘） | 带宽 / 月流量 | 超量后限速 | 价格 | 购买 |
| --- | --- | --- | --- | --- | --- |
| LAX.T1.WEE | 1核 / 1G / 10G | 4Gbps / 1TB | 100Mbps | $36.9/年 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.T1.TINY | 1核 / 1G / 10G | 4Gbps / 2TB | 200Mbps | $6.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.T1.STARTER | 2核 / 2G / 40G | 10Gbps / 4TB | 200Mbps | $12.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.T1.MINI | 2核 / 4G / 80G | 10Gbps / 8TB | 200Mbps | $21.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| LAX.T1.MICRO | 4核 / 4G / 120G | 10Gbps / 16TB | 500Mbps | $32.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.WEE | 1核 / 1G / 20G | / 1TB | 50Mbps | $36.9/年 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.TINY | 1核 / 1G / 20G | / 2TB | — | $6.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.STARTER | 1核 / 2G / 40G | / 4TB | — | $12.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.MINI | 2核 / 2G / 60G | / 8TB | — | $21.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.MICRO | 4核 / 4G / 80G | / 16TB | — | $32.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.MEDIUM | 4核 / 8G / 160G | / 32TB | — | $49.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| HKG.T1.LARGE | 8核 / 16G / 320G | / 64TB | — | $99.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| TYO.T1.WEE | 1核 / 1G / 20G | 4Gbps / 1TB | 50Mbps | $36.9/年 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| TYO.T1.TINY | 1核 / 1G / 20G | / 2TB | — | $6.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| TYO.T1.STARTER | 1核 / 2G / 40G | / 4TB | — | $12.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |
| TYO.T1.MINI | 2核 / 2G / 60G | / 8TB | — | $21.9/月 | [查看配置与价格](https://www.dmit.io/aff.php?aff=12046) |

支付方式上，支付宝、PayPal、信用卡都能用，对国内用户算比较友好。退款方面，公开测评整理的规则是新购服务 3 天内且流量使用不超过 30GB 可申请全额退款，30 天内可按剩余价值申请部分退款，具体条款以官网为准。

## 优惠码怎么用

下面这些优惠码来自 2026 年公开整理的资料，但优惠码的时效性变化很快，**下单前一定要在结算页点 Validate Code 验证**，验证通过再付款。

| 优惠码 | 适用范围 | 折扣 |
| --- | --- | --- |
| `LAX-EB-LAUNCH-NON-MONTHLY-RECURRING-20OFF` | 洛杉矶 EB，Tiny 及以上，非月付 | 8 折循环 |
| `2025-TYO-T1-HI-GSL-NON-MONTHLY-30OFF` | 东京 T1，Tiny 及以上，季付及以上 | 7 折 |
| `2025-TYO-T1-HI-GSL-MONTHLY-10OFF` | 东京 T1，月付 | 9 折 |
| `202510_HKG_TYO_PRO_20OFF_RECURRING` | 香港 / 东京 Pro，季付及以上 | 8 折循环 |
| `202510_HKG_TYO_T1_30OFF_RECURRING` | 香港 / 东京 T1，季付及以上（不含 WEE） | 7 折循环 |
| `Lite-Annually-Recur-30OFF` | Standard（原 Lite）系列，年付 | 7 折 |
| `Lite-Semi-Annually-Recur-20OFF` | Standard 系列，半年付 | 8 折 |

两个容易搞错的点：一是优惠码和付款周期强绑定，写着"非月付"的码用月付下单不会生效；二是部分码是"循环折扣"，续费时同样享受折扣，算长期成本的时候要把这个算进去。

## 按需求选线的几句话

纯电信宽带、主要用来建站或者对延迟极其敏感：直接看 Pro 系列。洛杉矶 PRO 三网回程 CN2 GIA，香港 Pro 延迟最低但价格最贵，东京 Pro 是折中。

联通或移动宽带、流量吃得多：EB 系列更合适，同价位流量和带宽基本翻倍。

要落地机、拉美国本土家宽、或者做亚太中转：T1 系列，$36.9/年起步，别指望它直连国内好用。

预算优先、能接受折腾：盯特价限量款，WEE / CORONA / MALIBU 这几款是 DMIT 性价比最高的入口，缺点是抢。

## 常见问题

**DMIT 三网测速用哪个 IP？**
按机房和线路对号入座：洛杉矶 Pro 用 154.17.30.96，EB 用 154.17.229.24，T1 用 174.136.204.135；香港 Pro 用 103.117.100.199，T1 用 154.12.176.28；东京 Pro 用 154.12.190.32，T1 用 154.31.112.5。测试 IP 会随库存调整，对不上时用官方 Looking Glass。

**测出来的延迟多少算正常？**
香港 Pro 全国三网平均在 40ms 出头，洛杉矶 Pro 华东方向在 130–140ms 区间，东京 Pro 大致在 30–75ms 之间浮动且运营商差别明显。这些是参考区间，晚高峰的抖动比绝对延迟更值得关注。
