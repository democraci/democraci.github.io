&nbsp;&nbsp;Cyclomatic Complexity(CC)和ABC是一些对代码衡量的指标.在这些指标出现之前,人们通常使用LoC(代码行数),方法平均行数等一些粗糙的指标去衡量代码.    
&nbsp;&nbsp;为什么需要这些指标? 在评价一个程序一个方法好坏复杂时,最笨得方法是人工检查代码, 获得一个对程序的大致认识.但是使用人工检查太耗费时间对复杂的程序也无能无力, 如果我们能定义一些可以使用机器分析出来的可靠指标,揭示程序特征,那想必是极好的,对于一切都希望有个衡量数字的人,也能带来极大的安全感.   
&nbsp;&nbsp;事实上,这些指标不是绝对的衡量指数,它们揭示出来的, 是程序的大概特征.如果要在这些指标的指导下查探修改程序中存在的问题,还需要大量的人工介入, 去确定是否真的存在问题真的有必要修改问题.不应该被这些指标驱使着去编程,让自己的程序指标得分高有时是好事, 但就像用代码行数衡量程序员生产率一样,唯指标论则完全脱离了这些发明这些指标的初衷.指标应该是诊断程序的帮助工具, 而不是最终目的. 

##Cyclomatic Complexity定义
&nbsp;&nbsp;Cyclomatic Complexity(CC)是一个衡量程序复杂度的指标,详细的定义可以参见[维基百科](https://en.wikipedia.org/wiki/Cyclomatic_complexity).简单地说, 对于每个方法, 从方法入口(调用), 到方法出口(return返回), 有多条可能执行路径, 这些路径组成一个控制流图,CC就是基于这个图计算的统计指标.对于一个方法,每个statement都是图中的一个点, 如果一个statement可能六成转向另外一个statement, 那么它们之间就会有一条边.     
&nbsp;&nbsp;下面使用一个抄袭的例子来讲解一些CC的具体计算, 计算如下Ruby方法的CC:   
```ruby
def cyclomatic_complexity_of_two(arg)
  if arg        # Node A
    puts "Foo"  # Node B
  else
    puts "Bar"  # Node C
  end
end             # Node D
```
&nbsp;&nbsp;对于上面方法, 可能的流程转向有: `A -> B, B -> D, A -> C, C -> D, D -> entrypoint`, 画出来的控制流图如下:![control_flow_graph](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2012/12/control_flow_graph.png)    
&nbsp;&nbsp;依据上图, 有计算公式`CC = Edges - Nodes + Endpoints`, 计算出的结果就是 5-4 +1 = 2.

##Cyclomatic Complexity指标作用
&nbsp;&nbsp;从定义上可知,CC揭示了一个方法内部可能的执行路径.CC越高, 说明方法越复杂, 内部可能执行路径越多, 测试起来越困难.如果一个方法内部控制语句比较多的话, 这个时候CC得分相对会高一点,此时要本能的嗅到code smell. 对于多高的CC是不正常, 目前没有明确的标准.

##Ruby中Cyclomatic Complexity的测量
&nbsp;&nbsp;一般使用Saikuro,官网[戳此](http://saikuro.rubyforge.org/).Saikuro使用BSD License,在计算CC时, 把所有`if, unless, while, until, for, elsif, when`以及`block`都当做程序控制流分支.
&nbsp;&nbsp;Saikuro可以指定CC数值在多少以上时给出Warning和Error, 这俩个基准值如果有相关经验和理论支持,定制可能更符合项目组中实际情况.
&nbsp;&nbsp;Saikuro目前还是每次人工跑命令获取CC结果, 如果能够和CI集成,在每次代码提交后,计算CC并给与开发者反馈, 则是极好的.

##ABC的定义
&nbsp;&nbsp;ABC也是一个度量代码复杂度的指标, 虽然这个指标原本发明出来是为了度量软件大小的.ABC的计算涉及到程序中三个方面.
*  赋值(Assignment, A): 将值赋予给变量, 例如`=, /=, +=, &=`等
*  分支(Branch, B): 程序分支, `if else switch`等等
*  方法调用(Call,C): 方法调用,将执行流从当前scope转移了.

ABC是上述三个数的平方和的开平方, `ABC = sqrt((A*A) + (B*B) + (C*C))`

##ABC指标作用
&nbsp;&nbsp;ABC代表了代码复杂度, 一个方法ABC得分越高, 则方法越复杂, 就有可能需要进行refactor.

##Ruby中ABC的测量
&nbsp;&nbsp;一般使用flog, github地址[戳此](https://github.com/seattlerb/flog). flog的安装和使用都比较简单,它会计算出每个方法的ABC指数,并默认输出ABC指数前60%的方法.
&nbsp;&nbsp;值得一提的是, flog计算ABC采取的标准非常主观,和标准的ABC定义不太一样, 可以说是为Ruby语言完全定制的ABC定义.flog中把`:and, :case, :if, :or, :rescue, :until, :when, :while`还有其他语句当做分支(Branch).所有赋值(Assignment)都得一分,方法调用(Call)则是将执行流从当前Scope中转移得方法调用.具体的计算过程可以直接阅读flog源码,大概1000行左右,非常精悍短小.
&&nbsp;&&nbsp;对于flog中计算出来的方法ABC指数,Jake Scruggs给出了如下经验法则:     
*  0-10: 无需做什么, 方法逻辑很简单
*  11-21: Good
*  21-40: 可能需要做一些refactor
*  41-60:需要检查一下方法代码, 看看是否需要做refactor
*  61-?: 只要有机会, 就应该做refactor

##其它工具
&nbsp;&nbsp;[PareseTree](http://www.zenspider.com/ZSS/Products/ParseTree/)是一个将Ruby方法/类转为Lisp中使用的S-expression,并可计算ABC指数的工具.   
&nbsp;&nbsp;[PMD](http://pmd.sourceforge.net/)是一个探查源代码缺陷的强大工具.它可以查找Java/JavaScript/PLSQL/XML/XSL中的常见编程错误, 也可以用来探查Java/C/C++/PHP/Ruby/JavaScript/Scala/Go中的重复代码.