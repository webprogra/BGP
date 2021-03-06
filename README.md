# BGP
## 基本概念
1. BGP概念  
  全称是Border Gateway Protocol, 对应中文是边界网关协议，最新版本是BGPv4。BGP是互联网上一个核心的互联网去中心化自治路由协议。  
  属于应用层协议，其传输层使用TCP，连接是可靠的，默认端口号是179。
2. AS自治系统  
  指在一个（有时是多个）组织管辖下的所有IP网络和路由器的全体，它们对互联网执行共同的路由策略。也就是说，对于互联网来说，一个AS是一个独立的整体网络。每个AS有自己唯一的编号。
3. 其他概念  
  **AS PATH**:路由每通过一个AS范围都会产生一个记录。(路由防环机制)。  
  **EBGP**:外部BGP协议(EBGP)的主要作用是向外部路由器或AS提供更多信息。  
  **IBGP**:内部BGP协议(IBGP)的主要作用是向AS内部路由器提供更多信息。      
  **ISP**: 网络服务供应商。  
  **peer As, Customer AS, Provider AS**：零个或多个 c2p 链接，后跟零个或一个 p2p 链接，后跟零个或多个 p2c 链接。此外，s2s 链接可以出现在路径中任何位置的任意数量。  
  
  
  
## BGP的三张表  
1. 邻居表(adjancy table):保存所有的BGP邻居信息。  
2. BGP表(forwarding database):保存从每一个邻居学到的路由信息。  
3. 路由表(routing table):BGP默认不做负载均衡，会从BGP表中选出一条到达各个目标网络最优的路由，放入路由表保存。路由器只需按路由表保存的路由条目转发数据即可。  

## BGP基本三原则  
1. 当BGP路由建立邻居连接后，彼此将路由条目发送给邻居。  
2. 当目的网络确定时，AS_PATH最短路径具有路由优先权。
3. 当目的网络确定时，网络通告地址越具体(掩码越长)越有路由优先权。   
## BGP攻击（hijack）ps:分别对应BGP基本三原则
1. 闲置AS抢夺  
对外宣告不属于自己，但属于其他机构合法且*未被宣告*的网络  
2. 近邻AS通告抢夺  
利用物理位置*近邻特性*（更接近目标路由，盗用目标路由的地址），就近宣告不属于自己的网络，劫持近邻网络链路
3. 长掩码抢夺（吸虹效应）  
利用BGP选路长掩码优先的特性劫持所有可达网段全流量。使用比目标路由更长的掩码。  
4. AS_PATH hijack (沙丁鱼捕术)  
利用AS_PATH prepend可任意修改的特点,通过增加AS_PATH穿越AS数量抑制其路由优先级，将数据流向赶向目标网络。达到控制网络流量的目的。  
## BGP安全漏洞
1. BGP路由泄露  
BGP路由条目在不同的角色都有其合理通告范围，一旦BGP路由通告传播到其原本预期通告范围之外称之为路由泄露，而这会产生难以准确预料的结果。
发生泄漏造成的结果：  
a. 造成源网络中断  
b. 造成源网络和被指向网络中断  
c. 造成AS穿越/ISP穿越/MIMT等问题  

## 检测和防御  
### 检测  
1. 用traceroute命令查看TTL相关信息。并与正常情况下进行比对。多数情况下可通过TTL值增减和经过的AS路径来判断是否存在劫持。  
2. 自建平台实时同步全球权威机构、组织的全量BGP路由表，并与本地收集到的BGP进行比对。发现异常并实时告警。  
3. 选取合理的采集周期，在周期内对BGP正常状态下路由更新条目的数量进行统计。选取合理的更新条目总数阈值范围，实时监控AS内BGP路由条目更新数目，发现异常实时告警。  
4. 使用商业化BGP路由监控告警平台。  
### 防御  
 1. 梳理清楚AS范围内BGP、IGP全局路由策略哪些路由通告是允许的、哪些是禁止的。并合理使用ACL、Route-map或BGP Prefx Filtering控制路由的宣告、传播范围。  
 2. 运营商、服务提供商应当依据以下原则，对不同商业角色路由器进行路由通告制定详细的BGP Prefx Filtering并启用。  
![image](https://user-images.githubusercontent.com/29565385/128840297-de1fcf25-e56e-42a5-aa68-7e2cfaac1cf4.png)  

### 算法模型  
#### 检测
##### 域间路由的中间人攻击模型  
域间路由的中间人攻击通常基于前缀劫持来实现。  
![image](https://user-images.githubusercontent.com/29565385/128955929-a3ae9bcf-5e5e-4383-bea5-6e612626b3bd.png)
![image](https://user-images.githubusercontent.com/29565385/128955958-dab6b986-3e47-43b9-8c92-b23acd7eb037.png)  
在攻击发起前从Y到f的AS_PATH为：AS2 AS1; 在攻击发生后，从Y到f的AS_PATH为：AS6 AS2 AS1  
1) 受污染网络在攻击之后，其到达受害前缀的数据平面的转发路径比控制平面的AS-path要延长(换言之，控制平面的
AS-path是其数据平面的转发路径的子路径)。
2) 受污染网络在攻击之后到达受害前缀的转发路径的终点正是攻击之前到达该前缀的AS-path的终点。  
###### 解决方案1：RPKI（以从RPKI依赖方那里得到验证后的IP前缀和起源AS号的绑定关系，即路由起源认证）  
能提供路由起源认证，而不能提供路由全路径认证，所以RPKI能够避免以上这种前缀劫持攻击。  
###### 解决方案2：BGPsec（RFC 8206）  
BGPsec要求每一个AS都对它接收到的路由通告消息进行签名，签名内容作为路由消息的路径属性BGPsec_Path_Signatures通告给邻居AS。  
签名内容：  
(1)上一个AS的BGPsec_Path_Signatures；
(2)本自治系统的AS号；
(3)路由消息要发往的自治网络的AS号；
(4)本AS的RPKI路由器公钥。  
BGPsec_Path_Signatures实际上代表了AS路径中的前一AS对后一AS继续通告该路由的授权。  
当一个AS收到路由通告消息后，会依次验证AS路径中每一个AS对应的BGPsec签名（包含在BGPsec_Path_Signatures中）。如果全部验证成功，说明该AS路径是真实有效的，当前AS才可能采用并继续向外广播该路由通告。BGPsec可以有效避免**BGP路径缩短攻击**。  

## 我国网络安全现状  
全球端口暴露最严重的十个国家为美国、中国、加拿大、韩国、英国、法国、荷兰、日本、德国和墨西哥，其中美国暴露的情况最严重，中国位列第二。  
近几年来虽然IETF已经有RPKI、BGPsec等安全的BGP标准推出,但由于种种原因（硬件资源消耗、迭代成本、过度风险等）这些标准的推进部署进度依旧十分缓慢。  
 
## 基于SVM的BGP异常流量检测 
### SVM定义 
        支持向量机（英语：support vector machine，常简称为SVM，又名支持向量网络）是在分类与回归分析中分析数据的监督式学习
    模型与相关的学习算法。 给定一组训练实例，每个训练实例被标记为属于两个类别中的一个或另一个，SVM训练算法创建一个将新的
    实例分配给两个类别之一的模型，使其成为非概率二元线性分类器。
        SVM模型是将实例表示为空间中的点，这样映射就使得单独类别的实例被尽可能宽的明显的间隔分开。然后，将新的实例映射到同一空
    间，并基于它们落在间隔的哪一侧来预测所属类别。 
        其核心思想是利用满足Mercer条件的核函数代替一个非线性映射，使得输入空间中的样本点能映射到一个高维的特征空间，并使之在
    该空间中线性可分，然后构造一个最优超平面来逼近理想分类结果即
        1. 使用非线性映射，将数据映射到F。
        2. 在特征空间中使用线性分类器。
### BGP特性 
     对BGP异常检测领域中获得的数据具有多变性、小样本和高维性等特性。
     而SVM是专门处理有限样本条件下的机器学习问题，其目标是得到在现有信息下的最优解而不仅是样本数量趋于无穷大的最优值。
