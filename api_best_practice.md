&nbsp;&nbsp;在为公司开发Restful Web Service API中,根据我自己的经验, 还有网上的一些资料,总结出了一些best practice.

##使用标准纯粹的Rest风格
&nbsp;&nbsp;对于什么是Rest风格的API, 大家可以参阅维基百科的定义,基本上讲得比较清楚.一般来说,推荐
*  所有资源命名使用名词而非动词,更为具体的说, 应该使用复数名词.
*  使用http method表示对资源的操作.同时各个http method中, GET是幂等的,也就是说,它不会对后台状态有影响.
*  对于不是CRUD的操作,可以有如下做法:
   1.  将该操作, 映射为资源的一个属性.比如如果对一个资源的active操作,可以实现成资源中的一个布尔型activated属性,通过PUT/PATCH方法更新资源时,即可更新该属性.这种实现方式,在操作无需参数时,比较方便.
   2.  将操作当成资源的一个子资源.操作本质上也可以是一个资源.例如Github API中的`star a gist`操作,是县城了star:`PUT /gists/:id/star`, unstar:`DELETE /gists/:id/star`
   3.  有时一些操作很难套成REST风格,例如search多种类型的资源操作,这个操作很难做成某个资源的action.此时`/search`更为合理,虽然search并不是一种资源.只要在文档中写明,并且从API用户的角度看不怪异,就行.
*  在有层级关系的资源中使用嵌套资源url. 具体来说,就是使用`GET /cars/711/drivers/4`之类的嵌套url. 这个标准也不是绝对的,如果子资源的id在系统中是独一无二,那么为了更短更易读的url和更简单的实现, 有人推荐不使用嵌套的[操作](http://weblog.jamisbuck.org/2007/2/5/nesting-resources).

##为结果集提供filter, sort, field selection, pagination功能
&nbsp;&nbsp;对于资源的index等操作,通常需要提供filter, sort, field selection, pagination等功能.这些功能的参数一般是作为url的查询参数.
*  filter: 可以依据属性的值对结果集进行过滤.例如列出tickets资源时,我们只需要状态为open的资源, 这时就需要`GET /tickets?state=open`对结果集进行过滤.
*  sort: 一般是提供一个通用的`sort`参数,该参数接收一个以`,`分隔的列表,依据列表中的属性对结果集进行排序.
   - `GET /tickets?sort=-priority`: 对结果集按priority降序排序
   - `GET /tickets?sort=-pripority, created_at`: 对结果集按priority降序排序,相同priority的记录项, 按created_at降序排序.

*  field selection:允许用户指定每个记录中包含哪些属性,能够有效的降低网络带宽,也能提高API的速度.`GET /tickets?fields=suject,date,location`
*  pagination: pagination在结果集非常大时是很有用的.分页一般使用offset+limit,或者page+per_page的方式.返回的结果集中, 应该告诉用户结果集的总记录数等信息.


##好的文档最重要
&nbsp;&nbsp;一个API最重要的部分就是文档,文档是API用户最为依赖的工具.API的文档应该风格统一,便于用户查找,还需要有具体的API使用示例.
&nbsp;&nbsp;更新不及时的文档不是好文档,API功能的更新,文档也要随之更新,并通过blog或者change log之类的手段及时通知用户.

##指明版本
&nbsp;&nbsp;API总是应该有版本的,版本的好处是显而易见的.对于API版本是否应该在url中还是http header中指明这个问题, 学院派的看法是应该在http header中,但是把版本包含在url中有一个很实用的好处, 就是可以直接通过浏览器访问api url,调试时很直观.    
&nbsp;&nbsp;有人推荐使用[Stripe API](https://stripe.com/docs/api#versioning)式的版本控制.在url中指明大版本,在http header中使用一个字段指明了小版本.这样大版本不变就是API结构保持稳定, 小版本则是在有一些小的改动时进行变化.     
&nbsp;&nbsp;API不会是一成不变的,因此对变动的管理就尤为重要.在deprecate一个API时,如果有良好的文档说明,提前足够久向客户打招呼,那么就不是难以接受的.

##使用JSON
&nbsp;&nbsp;相比XML, JSON作为API的返回格式,有很多优点, 更容易解析, 更易于阅读, 更容易和大多数编程语言中的数据模型对应.XML最大的有点是可扩展性更好,但是对于Web Service API, JSON已经足够用了.关于这俩点的优缺点之争,整个业界已经开始用脚投票了,使用JSON的企业和用户越来越多, 而XML由于其复杂性,越来越少.   
&nbsp;&nbsp;对于返回Content-Type/接受数据的格式,永远都记得要在http header的Content-Type/Accept里指明

##对返回数据pretty print并开启gzip
&nbsp;&nbsp;对返回数据pretty print的一个好处是极大的方便了调试.虽然pretty print可能会增大返回数据的大小,但在量级上来说,这个量一般不大(10%以下).gzip可以大大减少传输数据的量,因此节省带宽,twitter发现使用gzip后,他们的某些API中,带宽节省达到了80%.

##Authentication
&nbsp;&nbsp;无论什么时候, 对API的访问都应该使用SSL.
&nbsp;&nbsp;由于Restful API是无状态的,因此每次请求都应该携带鉴权信息.

##Cache

##Rate limiting

##错误处理
&nbsp;&nbsp;错误处理是API中至关重要的一环.API的返回包括两个部分, 一个是http状态码,一个是消息体.返回http状态码要严格遵照http协议中的定义(200/300/400/500, 具体可参见相关文档).   
&nbsp;&nbsp;返回的错误消息体中, 需要有一个消息错误码指明错误类型, 还有一个消息正文指明具体错误消息.这些都需要在文档中详细说明.

