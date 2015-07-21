## 引言
&nbsp;&nbsp;在2015年的北京QCon上,我听了一个关于微服务(microservices)的演讲,"API先行策略对企业的影响".主讲的哥们没有花时间讲解API相关的主题,全场都在布道微服务架构.之后我在[microservices.io](http://http://microservices.io/)上找到了一篇Martin Fowler关于微服务的详细介绍文章, 有点体会,下面内容是我对这篇文章的一点体会,内容多有雷同.恰好UI目前有意向推动整个架构向微服务方向变化,希望能够对大家能够有所帮助.

## 一体化架构和微服务架构
&nbsp;&nbsp;微服务是近年来兴起的一个软件架构风格方面的名词,有很多公司软件架构在演化中其实最后都变成了微服务或类似微服务,这个名词的出现正好填补了这个指代空缺.我直接引用Martin Fowler的微服务定义如下: <br/>
>In short, the microservice architectural style [1] is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.

&nbsp;&nbsp;定义中的关键词有`一个应用是多个小服务(a single application as a suite of small services)`, `轻量通信方式(lightweight communication mechanisms)`, `围绕业务需求构建(built around business capabilities
)`, `独立部署(independently deployable)`,`去中心化管理(decentralize)`,`灵活地技术栈选择(use different programming languages and data storage technologies)`.<br />
这些关键词很好地描绘了微服务架构的基本特征,关于这些关键字在微服务架构中得详细体现和带来的好处,下面会慢慢介绍.<br />
相对于微服务架构的,是一体化架构(Monolithic architecture).这个架构最常见,一个应用就是一个开发单元.Martin Fowler的定义:<br/>
>a monolithic application built as a single unit. Enterprise Applications are often built in three main parts: a client-side user interface (consisting of HTML pages and javascript running in a browser on the user's machine) a database (consisting of many tables inserted into a common, and usually relational, database management system), and a server-side application. The server-side application will handle HTTP requests, execute domain logic, retrieve and update data from the database, and select and populate HTML views to be sent to the browser. This server-side application is a monolith - a single logical executable[2]. Any changes to the system involve building and deploying a new version of the server-side application.

&nbsp;&nbsp;注意到其中的关键词`单一的可执行单元(a single logical executable)`,相对于微服务的将应用拆分成多个小服务的实现,必然带来的问题就是引用中说到得,每个小改变都要重新编译部署整个应用.

## 微服务的特征
###组件与服务
&nbsp;&nbsp;人们心目中理想的软件开发是这样的,有很多像乐高积木一样的组件,编写应用就是简单地搭积木.在现实应用开发过程中,我们会将很多东西都抽成组件,整个应用是组件的组合.组件简单地来说,就是一个可以独立被替换/更新而不影响整个应用的单元</br>
&nbsp;&nbsp;对于微服务来说,组件是一个个服务.每个服务运行在单独的进程上(不是绝对),服务之间通过远程调用通信.而在一体化架构中,应用是一个大进程,组件们在同一进程内存中.<br />
&nbsp;&nbsp;比较一下一体化架构和微服务架构,一体化架构组件之间的调用, 成本比服务之间远程调用要低很多.但是组件的每次更新,都要求对整个应用进行重新编译部署,服务就不存在这个问题.理想情况下,组件之间的边界是应该足够清晰的,低耦合高内聚.但在一体化架构下,受累于很多语言并没有良好的[公共接口](http://martinfowler.com/bliki/PublishedInterface.html)机制,随着时间的推移,很难维持组件间的边界,最终导致组件间依赖逐渐发生,一切越来越浆糊.微服务架构中,由于远程调用接口天然就是一个明确的边界,也是一个更清晰的接口,可以更容易维持组件之间的独立性.<br/>
&nbsp;&nbsp;微服务架构中各服务之间的远程API一般比内存中直接调用的API要粗糙得多,因为远程调用成本高,为了压缩成本,API的粒度必然要更大<br/>
###机构组织
Conway法则是这样说的:<br/>
>Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.

&nbsp;&nbsp;基于这一法则,一般公司部门的划分,其实就是应用拆分的一个复本.例如UI组,数据库组,服务逻辑组.这带来的问题就是团队协作实在太多了,逻辑散布在各个部门中,小小的变更都会导致跨团队间协作.
<br/>
![image](http://martinfowler.com/articles/microservices/images/conways-law.png)<br/>
&nbsp;&nbsp;微服务架构中服务的组织是基于业务功能的,团队的划分也是基于服务的组织,每个团队负责各自服务的用户接口,持久化存储等各方面.因此团队是跨职能的,负责所有开发相关方面:项目管理,数据库,用户体验.因此变更带来的跨团队协作变得更少了.<br/>
![image](http://martinfowler.com/articles/microservices/images/PreferFunctionalStaffOrganization.png)<br/>
###产品不是项目
&nbsp;&nbsp;大多数应用开发都是采取项目模型:致力提供一个完整地软件,一旦开发完成,软件移交给维护部门,然后就没有开发团队的事情了.<br/>
&nbsp;&nbsp;微服务架构则尽力避免这种开发模型,一个团队应该管理一个产品的全生命周期.Amazon的理念是"you build, you run it",开发团队需要对线上的软件负担全部责任.开发者需要每天都了解线上软件的运行状况,并需要承担一些售后支持,与用户的联系也必然更紧密.开发者经常不关心"软件如何帮助用户强化他们的商业竞争力",这种做法能够让开发者更加关注用户,而不是埋头实现功能.<br/>
&nbsp;&nbsp;当然在一体化架构中,也可以实现"you build, you run it".但是相比整个庞大应用,服务是更为细粒度的东西,更自然的在开发者和用户间构造联系.
###强终端和弱管道
&nbsp;&nbsp;当构建跨进程间的通信架构时,很多产品和设计都会在通信机制中强加入很多东西,ESB(Enterprise Service Bus)就是一个例子,它包含了很多消息路由,编码,传输,业务规则等等机制.<br/>
&nbsp;&nbsp;微服务架构则通常不这么做,而是强终端和弱管道.微服务架构在构造应用时,一个目标就是服务间依赖解耦,并高内聚,因此通信管道不能加入太多逻辑.这有点像Unix的风格,每个服务就是接受一系列输入,做处理,然后输出,简单地通信管道把这一切粘合起来,完成复杂的业务逻辑.<br/>
&nbsp;&nbsp;微服务架构因此多使用建立在HTTP协议上的resource API(Restful API)或轻量级消息中间件.轻量级消息中间件一般就是实现了消息路由即可,甚至没有可靠的异步机制,一切都在终端处理.<br/>
一体化架构中,组件都是在同一进程中,通信就是简单地方法调用.对于微服务架构来说,内存中的方法调用变成了RPC,由此带来了不可靠的问题,由此需要将原来粒度很细的调用,换成粗粒度的调用.
###去中心化管理
&nbsp;&nbsp;在一体化架构中,中心化管理倾向于将技术标准化,解决问题的方案因此也就被限制了.但是对于不同的问题,各自有合适的技术解决方案.C++适合编写实时性能要求高的组件,Ruby适合快速网络应用开发.微服务架构将应用拆分成多个服务,每个服务因此可以采取适合的技术方案.例如,如果某个服务的实现需要采取PostgrelSQL的一个特性,那尽管去使用,对其他服务来说,此服务的内部实现是个黑盒,无需关心.一切都因此更为灵活了.
###分散数据管理
&nbsp;&nbsp;一体化服务中,一个应用使用单一的数据库对数据进行持久化.但微服务架构中,每个服务团队都需要管理自己的数据库,无论是使用相同数据的同一实例,或者是使用不同的数据库系统.<br/>
&nbsp;&nbsp;在管理数据更新时,经常使用数据库事务来保证一致性.但数据如果散布在多个数据库系统上,分布式事务管理系统不仅难以实现,而且成本很高.因此微服务架构强调服务之间的事务协调,一致性也只能是最终一致性,并通过补偿操作来问题出现时进行处理.<br/>
&nbsp;&nbsp;选择这样处理一致性问题对很多开发团队来说是挑战,但这样做其实是一个常见业务实践模式.通常我们会允许一定的不一致性来保证快速响应性能,然后有一个进程专门处理错误.当业务上处理强一致性的消耗比处理错误消耗少时,这样做是值得的.
###基础设施自动化
&nbsp;&nbsp;过去的几年中,基础设施自动化技术已经得到了长足发展,尤其是云的出现,使构建,发布,运维的复杂性大大降低.<br/>
&nbsp;&nbsp;如果没有基础设施自动化技术支持,一个应用分成许许多多个微服务后,构建,测试,发布应用的成本必然高的不可想象.持集部署和持续集成,是目前自动化构建常用的技术.持集部署的一个目标在于让发布变得简单,无论一个还是十个.微服务架构必然更加依赖与基础设施自动化技术.
###容错性设计
&nbsp;&nbsp;使用服务当做组件构建应用的一个影响就是,应用在设计时需要对服务出错时有一定的容错性.当任意一个服务因为某些原因fail时,需要对用户友好的报错.相比于一体化架构,微服务架构在这方面引入了额外的复杂性.微服务团队需要时时刻刻都考虑服务fail时对用户体验的影响.<br/>
&nbsp;&nbsp;Netflix的Simian Army可以测试服务fail甚至数据中心fail时,整个应用的可恢复性以及监控是否发挥作用.这种测试一般就可以做到让大家都可以按时下班.由于服务在任何时候都有可能fail,因此快速地探测服务fail,并在可能时自动恢复就特别重要.微服务架构的应用需要投入很多精力在实时应用监控上,不仅检查架构方面实时情况(数据库每秒接收到多少请求),还要检查业务相关情况(例如有每分钟有多少订单).监控系统这样就可以在系统出错前提供预警,并通知开发团队跟进调查.<br/>
&nbsp;&nbsp;微服务架构中一般需要对每个服务都有一个复杂的监控,并提供一个面板,再次面板上可以查看每个服务的各方面情况.
##参考文献和其他相关链接
* Microservices introduction by Martin Fowler:[link](http://martinfowler.com/articles/microservices.html)
* 关于微服务架构的介绍官网:[link](http://microservices.io)

