

在/proc/sys/net/ipv4/目录下，包含的是和tcp/ip协议相关的各种参数，下面我们就对这些网络参数加以详细的说明。

参数名 参数类型
参数值(如无特别标注,内存类的单位为byte,关于时间的单位为秒)

* ip_forward ：BOOLEAN

0 - 关闭(默认值)
非0值 - 打开ip转发
在网络本地接口之间转发数据报。该参数非常特殊，对该参数的修改将导致其它所有相关配置参数恢复其默认值(对于主机参阅RFC1122，对于路由器参见 RFC1812)(在其他一些操作系统中,这个参数不是boolean型,而是INTEGER型,设置为0为不转发,1为根据接口情形决定是否转发,2是始终转发)
* ip_default_ttl ：INTEGER
默认值为 64
表示IP数据报的Time To Live值(在网络传递中,每经过一"跳",该值减少1,当ttl为0的时候，丢弃该包.该值越大,即在网络上可以经过的路由器设备的数量越多,但一个错误的包，也会越发浪费生存周期.根据目前的实际情形而看，设置为32已经足够普通网络访问Internet的需求了)
* ip_no_pmtu_disc ：BOOLEAN
默认值为FALSE(0)
关闭路径MTU探测(典型的瓶颈原理,一次成功的传输中,mtu是由网络上最"窄"的位置决定的.如果IP层有一个数据报要传，而且数据的长度比链路层的MTU还大，那么IP层就需要进行分片（fragmentation），把数据报分成若干片，这样每一片都小于MTU。
几种常见网络的MTU值:

超通道　　　　　　　　 65535
16Mb/ s令牌网(IBM)　　　17914
4Mb/ s令牌网(IEEE 802.5)　4464
FDDI　　　　　　　　　 4352
以太网　　　　　　　　 1500
IEEE 802.3/802.2　　　　　1492
X.25　　　　　　　　　　576
点对点(低延时)　　　　　296
 
* ipfrag_high_thresh ：INTEGER
默认值为262144
用来组装分段的IP包的最大内存量。两个文件分别表示用于重组IP分段的内存分配最低值和最高值，一旦达到最高内存分配值，其它分段将被丢弃，直到达到最低内存(ipfrag_low_thresh 见下文)分配值。(根据我个人理解,就是达到最高后,就"关门打狗",直到处理到最低值的时候才又开门放分段的ip包进来处理.如果最高/最低差距过小, 很可能很快又达到限制又开始丢弃包;而设置过大,又会造成某段时间丢包时间持续过久.因此需要适当地考虑,默认值中给出的最低/最高比率值为3/4.此外补充说明, kernel中,对内存的使用单位,都是以byte为单位的.当TCP数据包传输发生错误时，开始碎片整理。有效的数据包保留在内存，同时损坏的数据包被转发。我在1G内存的NAT机器上,分别设置最低为262144,最高为393216)
* ipfrag_low_thresh ：INTEGER
默认值为196608
参见ipfrag_high_thresh。
* ipfrag_time ：INTEGER
默认值为30
保存一个IP分片在内存中的时间。
* inet_peer_threshold ：INTEGER
默认值为65664
INET 对端存储器某个合适值，当超过该阀值条目将被丢弃。该阀值同样决定生存时间以及废物收集通过的时间间隔。条目越多?存活期越低?GC 间隔越短(GC=Grabage Collection 废物收集?默认值65664=65536 + 128 是怎么得来的呢?看include/net/inetpeer.h struct inet_peer的内容,是为了IP ROUTE更快,缓冲对方IP的信息,一个对方IP一个记录.该值与
* inet_peer_gc_maxtime
* inet_peer_gc_mintime
* inet_peer_maxttl
* inet_peer_minttl
* inet_peer_threshold
参数都是用来控制这个cache的大小的。似乎这个cache消耗比较大,在CU上有朋友提到过在一个26M的嵌入式Linux中,这个cache就用到了1M多内存)
* inet_peer_minttl ：INTEGER
默认值为120
条目的最低存活期。在重组端必须要有足够的碎片(fragment)存活期。www.linuxidc.com这个最低存活期必须保证缓冲池容积是否少于 inet_peer_threshold。该值以 jiffies为单位测量。(每次整理的时候,会考虑小于inet_peer_minttl 的ip条目一定保存,而大于inet_peer_maxttl时间设置的ip条目会被释放)
* inet_peer_maxttl ：INTEGER
默认值为600
条目的最大存活期。在此期限到达之后?如果缓冲池没有耗尽压力的话(例如?缓冲池中的条目数目非常少)?不使用的条目将会超时。该值以 jiffies为单位测量。
* inet_peer_gc_mintime ：INTEGER
默认值为10
废物收集(GC)通过的最短间隔。这个间隔会影响到缓冲池中内存的高压力。 该值以 jiffies为单位测量。(如果长期不整理,会cache很多条目,而整理的时间太频繁,又会给系统造成压力,这个值就是确定最小整理周期间隔的)
* inet_peer_gc_maxtime ：INTEGER
默认值为120
废物收集(GC)通过的最大间隔，这个间隔会影响到缓冲池中内存的低压力。 该值以 jiffies为单位测量。Jiffie: 内核使用的内部时间单位，在i386系统上大小为1/100s，在Alpha中为1/1024S。在/usr/include/asm/param.h中的HZ定义有特定系统的值。
##TCP 参数
tcp_syn_retries ：INTEGER
默认值是5
对于一个新建连接，内核要发送多少个 SYN 连接请求才决定放弃。不应该大于255，默认值是5，对应于180秒左右时间。(对于大负载而物理通信良好的网络而言,这个值偏高,可修改为2.这个值仅仅是针对对外的连接,对进来的连接,是由tcp_retries1 决定的)
* tcp_synack_retries ：INTEGER
默认值是5
对于远端的连接请求SYN，内核会发送SYN ＋ ACK数据报，以确认收到上一个 SYN连接请求包。这是所谓的三次握手( threeway handshake)机制的第二个步骤。这里决定内核在放弃连接之前所送出的 SYN+ACK 数目。不应该大于255，默认值是5，对应于180秒左右时间。(可以根据上面的 tcp_syn_retries 来决定这个值)
* tcp_keepalive_time ：INTEGER
默认值是7200(2小时)
当 keepalive打开的情况下，TCP发送keepalive消息的频率。(由于目前网络攻击等因素,造成了利用这个进行的攻击很频繁,曾经也有cu 的朋友提到过,说如果2边建立了连接,然后不发送任何数据或者rst/fin消息,那么持续的时间是不是就是2小时,空连接攻击? tcp_keepalive_time就是预防此情形的.我个人在做nat服务的时候的修改值为1800秒)
* tcp_keepalive_probes：INTEGER
默认值是9
TCP发送keepalive探测以确定该连接已经断开的次数。(注意:保持连接仅在SO_KEEPALIVE套接字选项被打开是才发送.次数默认不需要修改,当然根据情形也可以适当地缩短此值.设置为5比较合适)
* tcp_keepalive_intvl：INTEGER
默认值为75
探测消息发送的频率，乘以tcp_keepalive_probes就得到对于从开始探测以来没有响应的连接杀除的时间。默认值为75秒，也就是没有活动的连接将在大约11分钟以后将被丢弃。(对于普通应用来说,这个值有一些偏大,可以根据需要改小.特别是web类服务器需要改小该值,15是个比较合适的值)
* tcp_retries1 ：INTEGER
默认值是3
放弃回应一个TCP连接请求前?需要进行多少次重试。RFC 规定最低的数值是3?这也是默认值?根据RTO的值大约在3秒 - 8分钟之间。(注意:这个值同时还决定进入的syn连接)
* tcp_retries2 ：INTEGER
默认值为15
在丢弃激活(已建立通讯状况)的TCP连接之前?需要进行多少次重试。默认值为15，根据RTO的值来决定，相当于13-30分钟(RFC1122规定，必须大于100秒).(这个值根据目前的网络设置,可以适当地改小,我的网络内修改为了5)
* tcp_orphan_retries ：INTEGER
默认值是7
在近端丢弃TCP连接之前?要进行多少次重试。默认值是7个?相当于 50秒 - 16分钟?视 RTO 而定。如果您的系统是负载很大的web服务器?那么也许需要降低该值?这类 sockets 可能会耗费大量的资源。另外参的考 tcp_max_orphans 。(事实上做NAT的时候,降低该值也是好处显著的,我本人的网络环境中降低该值为3)
* tcp_fin_timeout ：INTEGER
默认值是 60
对于本端断开的socket连接，TCP保持在FIN-WAIT-2状态的时间。对方可能会断开连接或一直不结束连接或不可预料的进程死亡。默认值为 60 秒。过去在2.2版本的内核中是 180 秒。您可以设置该值?但需要注意?如果您的机器为负载很重的web服务器?您可能要冒内存被大量无效数据报填满的风险?FIN-WAIT-2 sockets 的危险性低于 FIN-WAIT-1 ?因为它们最多只吃 1.5K 的内存?但是它们存在时间更长。另外参考 tcp_max_orphans。(事实上做NAT的时候,降低该值也是好处显著的,我本人的网络环境中降低该值为30)
* tcp_max_tw_buckets ：INTEGER
默认值是180000
系统在同时所处理的最大 timewait sockets 数目。如果超过此数的话?time-wait socket 会被立即砍除并且显示警告信息。之所以要设定这个限制?纯粹为了抵御那些简单的 DoS 攻击?千万不要人为的降低这个限制?不过?如果网络条件需要比默认值更多?则可以提高它(或许还要增加内存)。(事实上做NAT的时候最好可以适当地增加该值)
* tcp_tw_recycle ：BOOLEAN
默认值是0
打开快速 TIME-WAIT sockets 回收。除非得到技术专家的建议或要求?请不要随意修改这个值。(做NAT的时候，建议打开它)
* tcp_tw_reuse：BOOLEAN
默认值是0
该文件表示是否允许重新应用处于TIME-WAIT状态的socket用于新的TCP连接(这个对快速重启动某些服务,而启动后提示端口已经被使用的情形非常有帮助)
* tcp_max_orphans ：INTEGER
缺省值是8192
系统所能处理不属于任何进程的TCP sockets最大数量。假如超过这个数量?那么不属于任何进程的连接会被立即reset，并同时显示警告信息。之所以要设定这个限制?纯粹为了抵御那些简单的 DoS 攻击?千万不要依赖这个或是人为的降低这个限制(这个值RedHat AS版本中设置为32768,但是很多防火墙修改的时候,建议该值修改为2000)
* tcp_abort_on_overflow ：BOOLEAN
缺省值是0
当守护进程太忙而不能接受新的连接，就象对方发送reset消息，默认值是false。这意味着当溢出的原因是因为一个偶然的猝发，那么连接将恢复状态。只有在你确信守护进程真的不能完成连接请求时才打开该选项，该选项会影响客户的使用。(对待已经满载的sendmail,apache这类服务的时候,这个可以很快让客户端终止连接,可以给予服务程序处理已有连接的缓冲机会,所以很多防火墙上推荐打开它)
* tcp_syncookies ：BOOLEAN
默认值是0
只有在内核编译时选择了CONFIG_SYNCOOKIES时才会发生作用。当出现syn等候队列出现溢出时象对方发送syncookies。目的是为了防止syn flood攻击。
注意：该选项千万不能用于那些没有收到攻击的高负载服务器，如果在日志中出现synflood消息，但是调查发现没有收到synflood攻击，而是合法用户的连接负载过高的原因，你应该调整其它参数来提高服务器性能。参考:
* tcp_max_syn_backlog
* tcp_synack_retries
* tcp_abort_on_overflow
syncookie 严重的违背TCP协议，不允许使用TCP扩展，可能对某些服务导致严重的性能影响(如SMTP转发)。(注意,该实现与BSD上面使用的tcp proxy一样,是违反了RFC中关于tcp连接的三次握手实现的,但是对于防御syn-flood的确很有用.)
* tcp_stdurg ：BOOLEAN
默认值为0
使用 TCP urg pointer 字段中的主机请求解释功能。大部份的主机都使用老旧的 BSD解释，因此如果您在 Linux 打开它?或会导致不能和它们正确沟通。
* tcp_max_syn_backlog ：INTEGER
对于那些依然还未获得客户端确认的连接请求?需要保存在队列中最大数目。对于超过 128Mb 内存的系统?默认值是 1024 ?低于 128Mb 的则为 128。如果服务器经常出现过载?可以尝试增加这个数字。警告?假如您将此值设为大于 1024?最好修改 include/net/tcp.h 里面的 TCP_SYNQ_HSIZE ?以保持 TCP_SYNQ_HSIZE*16<=tcp_max_syn_backlog ?并且编进核心之内。(SYN Flood攻击利用TCP协议散布握手的缺陷，伪造虚假源IP地址发送大量TCP-SYN半打开连接到目标系统，最终导致目标系统Socket队列资源耗尽而无法接受新的连接。为了应付这种攻击，现代Unix系统中普遍采用多连接队列处理的方式来缓冲(而不是解决)这种攻击，是用一个基本队列处理正常的完全连接应用(Connect()和Accept() )，是用另一个队列单独存放半打开连接。这种双队列处理方式和其他一些系统内核措施(例如Syn-Cookies/Caches)联合应用时，能够比较有效的缓解小规模的SYN Flood攻击(事实证明<1000p/s)加大SYN队列长度可以容纳更多等待连接的网络连接数，所以对Server来说可以考虑增大该值.)
* tcp_window_scaling ：INTEGER
缺省值为1
该文件表示设置tcp/ip会话的滑动窗口大小是否可变。参数值为布尔值，为1时表示可变，为0时表示不可变。tcp/ip通常使用的窗口最大可达到 65535 字节，对于高速网络，该值可能太小，这时候如果启用了该功能，可以使tcp/ip滑动窗口大小增大数个数量级，从而提高数据传输的能力(RFC 1323)。（对普通地百M网络而言，关闭会降低开销，所以如果不是高速网络，可以考虑设置为0）
* tcp_timestamps ：BOOLEAN
缺省值为1
Timestamps 用在其它一些东西中?可以防范那些伪造的 sequence 号码。一条1G的宽带线路或许会重遇到带 out-of-line数值的旧sequence 号码(假如它是由于上次产生的)。Timestamp 会让它知道这是个 '旧封包'。(该文件表示是否启用以一种比超时重发更精确的方法（RFC 1323）来启用对 RTT 的计算；为了实现更好的性能应该启用这个选项。)
* tcp_sack ：BOOLEAN
缺省值为1
使用 Selective ACK?它可以用来查找特定的遗失的数据报--- 因此有助于快速恢复状态。该文件表示是否启用有选择的应答（Selective Acknowledgment），这可以通过有选择地应答乱序接收到的报文来提高性能（这样可以让发送者只发送丢失的报文段）。(对于广域网通信来说这个选项应该启用，但是这会增加对 CPU 的占用。)
* tcp_fack ：BOOLEAN
缺省值为1
打开FACK拥塞避免和快速重传功能。(注意，当tcp_sack设置为0的时候，这个值即使设置为1也无效)
* tcp_dsack ：BOOLEAN
缺省值为1
允许TCP发送"两个完全相同"的SACK。
* tcp_ecn ：BOOLEAN
缺省值为0
打开TCP的直接拥塞通告功能。
* tcp_reordering ：INTEGER
默认值是3
TCP流中重排序的数据报最大数量 。 (一般有看到推荐把这个数值略微调整大一些,比如5)
* tcp_retrans_collapse ：BOOLEAN
缺省值为1
对于某些有bug的打印机提供针对其bug的兼容性。(一般不需要这个支持,可以关闭它)
* tcp_wmem(3个INTEGER变量)： min, default, max
min：为TCP socket预留用于发送缓冲的内存最小值。每个tcp socket都可以在建议以后都可以使用它。默认值为4096(4K)。
default：为TCP socket预留用于发送缓冲的内存数量，默认情况下该值会影响其它协议使用的net.core.wmem_default 值，一般要低于net.core.wmem_default的值。默认值为16384(16K)。
max: 用于TCP socket发送缓冲的内存最大值。该值不会影响net.core.wmem_max，"静态"选择参数SO_SNDBUF则不受该值影响。默认值为 131072(128K)。（对于服务器而言，增加这个参数的值对于发送数据很有帮助,在我的网络环境中,修改为了51200 131072 204800）
* tcp_rmem (3个INTEGER变量)： min, default, max
min：为TCP socket预留用于接收缓冲的内存数量，即使在内存出现紧张情况下tcp socket都至少会有这么多数量的内存用于接收缓冲，默认值为8K。
default：为TCP socket预留用于接收缓冲的内存数量，默认情况下该值影响其它协议使用的 net.core.wmem_default 值。该值决定了在tcp_adv_win_scale、tcp_app_win和tcp_app_win=0默认值情况下，TCP窗口大小为65535。默认值为87380
max：用于TCP socket接收缓冲的内存最大值。该值不会影响 net.core.wmem_max，"静态"选择参数 SO_SNDBUF则不受该值影响。默认值为 128K。默认值为87380*2 bytes。（可以看出，.max的设置最好是default的两倍,对于NAT来说主要该增加它,我的网络里为 51200 131072 204800）
* tcp_mem(3个INTEGER变量)：low, pressure, high
low：当TCP使用了低于该值的内存页面数时，TCP不会考虑释放内存。(理想情况下，这个值应与指定给 tcp_wmem 的第 2 个值相匹配 - 这第 2 个值表明，最大页面大小乘以最大并发请求数除以页大小 (131072 * 300 / 4096)。 )
pressure：当TCP使用了超过该值的内存页面数量时，TCP试图稳定其内存使用，进入pressure模式，当内存消耗低于low值时则退出 pressure状态。(理想情况下这个值应该是 TCP 可以使用的总缓冲区大小的最大值 (204800 * 300 / 4096)。 )
high：允许所有tcp sockets用于排队缓冲数据报的页面量。(如果超过这个值，TCP 连接将被拒绝，这就是为什么不要令其过于保守 (512000 * 300 / 4096) 的原因了。 在这种情况下，提供的价值很大，它能处理很多连接，是所预期的 2.5 倍；或者使现有连接能够传输 2.5 倍的数据。 我的网络里为192000 300000 732000)
一般情况下这些值是在系统启动时根据系统内存数量计算得到的。
* tcp_app_win : INTEGER
默认值是31
保留max(window/2^tcp_app_win, mss)数量的窗口由于应用缓冲。当为0时表示不需要缓冲。
* tcp_adv_win_scale : INTEGER
默认值为2
计算缓冲开销bytes/2^tcp_adv_win_scale(如果tcp_adv_win_scale > 0)或者bytes-bytes/2^(-tcp_adv_win_scale)(如果tcp_adv_win_scale <= 0）。
* tcp_rfc1337 :BOOLEAN
缺省值为0
这个开关可以启动对于在RFC1337中描述的"tcp 的time-wait暗杀危机"问题的修复。启用后，内核将丢弃那些发往time-wait状态TCP套接字的RST 包.
* tcp_low_latency : BOOLEAN
缺省值为0
允许 TCP/IP 栈适应在高吞吐量情况下低延时的情况；这个选项一般情形是的禁用。(但在构建Beowulf 集群的时候,打开它很有帮助)
* tcp_westwood :BOOLEAN
缺省值为0
启用发送者端的拥塞控制算法，它可以维护对吞吐量的评估，并试图对带宽的整体利用情况进行优化；对于 WAN 通信来说应该启用这个选项。
* tcp_bic :BOOLEAN
缺省值为0
为快速长距离网络启用 Binary Increase Congestion；这样可以更好地利用以 GB 速度进行操作的链接；对于 WAN 通信应该启用这个选项。
##IP、ICMP
* ip_local_port_range: (两个INTEGER)
定于TCP和UDP使用的本地端口范围，第一个数是开始，第二个数是最后端口号，默认值依赖于系统中可用的内存数：
> 128Mb 32768-61000
< 128Mb 1024-4999 or even less.
该值决定了活动连接的数量，也就是系统可以并发的连接数(做nat的时候,我将它设置为了1024 65530 工作正常)
* ip_nonlocal_bind : BOOLEAN
默认值是0
如果您想让应用程式能够捆绑到一个不属於该系统的位址?就需要设定这?。(当机器使
用非固定/动态的网络连接的时候,或者离线调试程序的时候,当线路断掉之后?该服务仍可?动而且捆绑到特定的位址之上。)
* ip_dynaddr : BOOLEAN
默认值是0
假如甚至为非0值,那么将支持动态地址.如果是设置为>1的值,将在动态地址改写的时候发一条内核消息。(如要用动态界面位址做 dail-on-demand ?那就设定它。一旦请求界面起来之
后?所有看不到回应的本地 TCP socket 都会重新捆绑(rebound)?以获得正确的位址。
假如遇到该网络界面的连线不工作?但重新再试一次却又可以的情形?设定这个可解决这
个问题。)
* icmp_echo_ignore_all ： BOOLEAN
* icmp_echo_ignore_broadcasts ： BOOLEAN
默认值是0
如果任何一个设置为true(>0)则系统将忽略所有发送给自己的ICMP ECHO请求或那些广播地址的请求。(现在网络上很多病毒/木马自动发起感染攻击是先用icmp的echo方式判断对方是否存活，因此开启该值，会降低一些被骚扰的可能性。但由于禁止了 icmp，就无法ping到该机器了，因此网络管理员也没有办法判断机器是否存活了，所以可以考虑用netfilter/iptables来完成该工作会更有所选择针对性.icmp_echo_ignore_all 是禁止所有的icmp包,而icmp_echo_ignore_broadcasts是禁止了所有的广播包)
* icmp_ratelimit : INTEGER
默认值是100 Jiffie
限制发向特定目标的匹配icmp_ratemask的ICMP数据报的最大速率。0表示没有任何限制，否则表示jiffies数据单位中允许发送的个数。 (如果在icmp_ratemask进行相应的设置Echo Request的标志位掩码设置为1,那么就可以很容易地做到ping回应的速度限制了)
* icmp_ratemask : INTEGER
在这里匹配的ICMP被icmp_ratelimit参数限制速率.
匹配的标志位: ＩＨＧＦＥＤＣＢＡ９８７６５４３２１０
默认的掩码值: ００００００１１００００００１１０００ (6168)
关于标志位的设置,可参考 源程序目录/include/linux/icmp.h

0 Echo Reply
3 Destination Unreachable *
4 Source Quench *
5 Redirect
8 Echo Request
B Time Exceeded *
C Parameter Problem *
D Timestamp Request
E Timestamp Reply
F Info Request
G Info Reply
H Address Mask Request
I Address Mask Reply
* 号的被默认限速(见上表mask)
icmp_ignore_bogus_error_responses : BOOLEAN
默认值是0
某些路由器违背RFC1122标准，其对广播帧发送伪造的响应来应答。这种违背行为通常会被以告警的方式记录在系统日志中。如果该选项设置为True，内核不会记录这种警告信息。(我个人而言推荐设置为1)
##网络接口界面(比如lo,eth0,eth1)参数
* /proc/sys/net/ipv4/conf/{interface}/* :
在/proc/sys/net/ipv4/conf/ 下可以发现类似 all,eth0,eth1,default,lo 等网络接口界面,每一个都是目录,他们下属的文件中,每个文件对应该界面下某些可以设置的选项设置.(all/是特定的，用来修改所有接口的设置, default/ 表示缺省设置,lo/表示本地接口设置,eth0/表示第一块网卡,eth1/表示第2块网卡.注意:下面有的参数,是需要all和该界面下同时为 ture才生效,而某些则是只需要该界面下为true即可,注意区别!!)
* log_martians : BOOLEAN
记录带有不允许的地址的数据报到内核日志中。all/ 或者{interface}/ 下至少有一个为True即可生效.
* accept_redirects : BOOLEAN
对于主机来说默认为True，对于用作路由器时默认值为False
收发接收ICMP重定向消息。all/ 和{interface}/ 下两者同时为True方可生效.
(如果不熟悉所在网络的结构.推荐不修改,因为在有多个出口的网络的时候,如果有2个出口路由器,由于作为主机的时候默认只指认一个网关,出口路由可能有策略设置转到另一个路由器上.)
* forwarding : BOOLEAN
在该接口打开转发功能 (在3块或以上的网卡的时候很实用,有时候只想让其中一外一内,另一块做服务,就可以让这块做服务的网卡不转发数据进出)
* mc_forwarding :BOOLEAN
是否进行多播路由。只有内核编译有CONFIG_MROUTE并且有路由服务程序在运行该参数才有效。
* medium_id :INTEGER
默认值是0
通常,这个参数用来区分不同媒介.两个网络设备可以使用不同的值,使他们只有其中之一接收到广播包.默认值为0表示各个网络介质接受他们自己介质上的媒介,值-1表示该媒介未知。通常,这个参数被用来配合proxy_arp实现roxy_arp的特性即是允许arp报文在两个不同的网络介质中转发.(第一段 Integer value used to differentiate the devices by the medium they
are attached to. Two devices can have different id values when
the broadcast packets are received only on one of them.
The default value 0 means that the device is the only interface
to its medium, value of -1 means that medium is not known.没读懂,去cu问人，以后更正)
* proxy_arp : BOOLEAN
打开arp代理功能。all/ 或者{interface}/ 下至少有一个为True即可生效
* shared_media : BOOLEAN
默认为True
发送(路由器)或接收(主机) RFC1620 共享媒体重定向。覆盖ip_secure_redirects的值。all/ 或者{interface}/ 下至少有一个为True即可生效
* secure_redirects : BOOLEAN
默认为True
仅仅接收发给默认网关列表中网关的ICMP重定向消息，默认值是TRUE。all/ 或者{interface}/ 下至少有一个为True即可生效。 (这个参数一般情形请不要修改，可以有效地防止来自同网段的非网关机器发出恶意ICMP重定向攻击行为)
send_redirects : BOOLEAN
默认为True
如果是router，允许发送重定向消息.all/ 或者{interface}/ 下至少有一个为True即可生效。(根据网络而定，如果是做NAT,并且网内只有此一个网关的时候，其实是可以关闭掉它的,事实上目前而言,IP Redirects是TCP/IP协议产生早期为了解决网络持续性而提出的一种方法，后来事实证明这种措施不太实用而且具有很大的安全风险，可能引起各种可能的网络风险产生 - 拒绝服务攻击，中间人攻击，会话劫持等等,所以很多安全文档是推荐关闭它.)
* bootp_relay : BOOLEAN
默认为False
接收源地址为0.b.c.d，目的地址不是本机的数据报。用来支持BOOTP转发服务进程，该进程将捕获并转发该包。目前还没有实现。
* accept_source_route : BOOLEAN
对于主机来说默认为False，对于用作路由器时默认值为True
接收带有SRR选项的数据报。all/ 和{interface}/ 下两者同时为True方可生效.(IP 源路由选项，也是TCP/IP协议早期的一个实现缺陷，允许IP包自身携带路由选择选项，这将允许攻击者绕过某些安全检验的网关，或者被用来探测网络环境。在企业网关上强烈建议设置关闭或过绿丢弃IP源路由选项数据包。这个功能在调试网络的时候很有用,但是在真正的实际应用中,有可能造成一些麻烦和危险)
* rp_filter : BOOLEAN
默认值为False
1 - 通过反向路径回溯进行源地址验证(在RFC1812中定义)。对于单穴主机和stub网络路由器推荐使用该选项。
0 - 不通过反向路径回溯进行源地址验证。
默认值为0，但某些发布在启动时自动将其打开。 (router默认会路由所有东西?就算该封包'显然'不属於我们的网路的。常见的例子?莫过於将私有 IP 泄漏到 internet 上去。假如某个界面?其上设定的网络地址段?
195.96.96.0/24? 那么理论上不会有212.64.94.1 这样的地址段封包会到达这个界面上。许多人都不想转发非本网段的数据包?因此核心设计者也打开了方便之门。在 /proc ?面有些档案?透过它们您可以让核心?您做到这点。此方法被称? "逆向路径过滤(Reverse Path Filtering)"。基本上?假如对此封包作出的回应?不是循其进入的界面送出去?那它就被置之不理。)
* arp_filter : BOOLEAN
默认值为False
1 -允许多个网络介质位于同一子网段内，每个网络界面依据是否内核指派路由该数据包经过此界面来确认是否回答ARP查询(这个实现是由来源地址确定路由的时候决定的),换句话说，允许控制使用某一块网卡（通常是第一块）回应arp询问。(做负载均衡的时候,可以考虑用
echo 1 > /proc/sys/net/ipv4/conf/all/arp_filter
这样的方式就可以解决,当然 利用
echo 2 /proc/sys/net/ipv4/conf/all/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
两条命令配合使用更好,因为arp_announce 和arp_ignore 似乎是对arp_filter的更细节控制的实现。)
0 -默认值，内核设置每个网络界面各自应答其地址上的arp询问。这项看似会错误的设置却经常能非常有效，因为它增加了成功通讯的机会。在Linux主机上，每个IP地址是网络界面独立的，而非一个复合的接口。只有在一些特殊的设置的时候，比如负载均衡的时候会带来麻烦。
all/ 或者{interface}/ 下至少有一个为True即可生效。（简单来说，就是同一Linux上，如果有某些原因，有2块网卡必须设置为同一网段，那么默认情况下，会有一块工作，而另外一块不工作或者内核频繁报告错误，这个时候就需要打开这个选项了）
* arp_announce : INTEGER
默认为0
对网络接口上本地IP地址发出的ARP回应作出相应级别的限制:
确定不同程度的限制,宣布对来自本地源IP地址发出Arp请求的接口
0 - (默认) 在任意网络接口上的任何本地地址
1 -尽量避免不在该网络接口子网段的本地地址. 当发起ARP请求的源IP地址是被设置应该经由路由达到此网络接口的时候很有用.此时会检查来访IP是否为所有接口上的子网段内ip之一.如果改来访IP 不属于各个网络接口上的子网段内,那么将采用级别2的方式来进行处理.

2 - 对查询目标使用最适当的本地地址.在此模式下将忽略这个IP数据包的源地址并尝试选择与能与该地址通信的本地地址.首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址. 如果没有合适的地址被发现,将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接口来进行发送
all/ 和{interface}/ 下两者同时比较，取较大一个值生效.
提高约束级别有益于从指定的目标接受应答,而降低级别可以给予更多的arp查询者以反馈信息(关于arp代理这一段我普遍翻译地不好，去啃一下tcp/ip bible的卷一，然后再翻译吧)
* arp_ignore : INTEGER
默认为0
定义对目标地址为本地IP的ARP询问不同的应答模式
0 - (默认值): 回应任何网络接口上对任何本地IP地址的arp查询请求（比如eth0=192.168.0.1/24,eth1=10.1.1.1/24,那么即使 eth0收到来自10.1.1.2这样地址发起的对10.1.1.1 的arp查询也会回应--而原本这个请求该是出现在eth1上，也该有eth1回应的）
1 - 只回答目标IP地址是来访网络接口本地地址的ARP查询请求（比如eth0=192.168.0.1/24,eth1=10.1.1.1/24,那么即使 eth0收到来自10.1.1.2这样地址发起的对192.168.0.1的查询会回答，而对10.1.1.1 的arp查询不会回应）
2 -只回答目标IP地址是来访网络接口本地地址的ARP查询请求,且来访IP必须在该网络接口的子网段内（比如eth0=192.168.0.1/24, eth1=10.1.1.1/24,eth1收到来自10.1.1.2这样地址发起的对192.168.0.1的查询不会回答，而对 192.168.0.2发起的对192.168.0.1的arp查询会回应）
3 - 不回应该网络界面的arp请求，而只对设置的唯一和连接地址做出回应(do not reply for local addresses configured with scope host,only resolutions for global and link addresses are replied 翻译地似乎不好，这个我的去问问人)
4-7 - 保留未使用
8 -不回应所有（本地地址）的arp查询
all/ 和{interface}/ 下两者同时比较，取较大一个值生效.
* tag : INTEGER
默认为0
