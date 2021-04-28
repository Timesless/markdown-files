|  层次  |
| :----: |
| 应用层 |
| 传输层 |
| 网际层 |
| 链路层 |

> + 应用层是用户进程，传输层、网际层、链路层由内核执行「由操作系统实现」
> + IP 协议是不可靠的，它只是尽可能传输数据
> + TCP 建设在 IP 上层，提供可靠传输「1. 校验和；2. 确认机制；3. 超时重传

用户进程	用户进程	用户进程			【应用层】

​		:arrow_lower_right:			:arrow_down:			:arrow_lower_left:

​				TCP			UDP					   【传输层】

​						:arrow_lower_right::arrow_lower_left:

​			ICMP	  IP			IGMP 			【网络层】

​						  :arrow_down:

​			ARP	 硬件接口		RAP		  【链路层】

> ICMP 是 IP 协议的附属协议，用户进程 :arrow_right: ICMP「ping」
>
> + 以太网帧长度在 46 ~ 1500 B
>
> **TCP 层：数据段；IP 层：数据报，报文；链路层：帧；物理层：比特流**



### 链路层：以太网和 IEE 802

以太网协议和 IEEE 802 协议不是同一个协议

**PC 与 路由器之间数据传输为以太网协议（抓包基本都是以太网协议），路由器与路由器之间数据传输为 IEE 802.3 协议 「？」**



+ 两种帧格式都采用 48B 源 MAC 和目的 MAC
+ ARP 和 RARP 协议对 32B IP 地址和 48B MAC 地址进行映射
+ 802.3 规定数据部分至少 38B，而以太网协议要求至少 46B（46 + 6 + 6 + 2 + 4）

<table>
  <tr>
  	<td>802协议</td>
		<td>目的MAC 6B</td>
    <td>源MAC 6B</td>
    <td>长度 2B</td>
    <td>1B</td>
		<td>1B</td>
    <td>1B</td>
    <td>3B</td>
    <td>类型 2B</td>
		<td>数据 38 - 1492</td>
    <td>CRC 4B</td>
  </tr>
  <tr>
  	<td>以太网协议</td>
		<td>目的MAC 6B</td>
    <td>源MAC 6B</td>
    <td>类型 2B</td>
    <td colspan="6">数据 46 ~ 1500</td>
		<td>CRC 4B</td>
  </tr>
</table>



### IP 协议

> + 面向无连接，不可靠的协议



1. 第一行的 4B

4b 版本号、4b 首部长度、8b 服务类型（TOS，差别服务）、16b 总长度

> 其中**实际首部长度 = 首部长度 x 4**（最大 15 x 4 = 60B）

2. 第二行 4B

16b 标识（ID）、3b 标志（**仅使用 2b：df（dont fragment），mf（more fragment）**）、13b 片偏移

3. 第三行 4B

8b TTL（每个 hop 都 - 1）、8b 协议（**ICMP：1，TCP：6，UDP：17**）、16b 首部校验和

4. 第四行 4B

32b 源 IP 地址

5. 第五行 4B

32b 目标 IP 地址

6. 选项【如果有，最大 60 - 20 = 40B】
7. 数据