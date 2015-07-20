##引言
&nbsp;&nbsp;最近一直在看Clean Code,关注整洁代码编写方面的实践.之前组里有推广Rubocop,也一直在强调Ruby代码要合乎规范.我在github上找到了他们内部styleguide,地址可戳[https://github.com/styleguide](https://github.com/styleguide).Github在里面提供了他们内部开发时的CSS/JS/Ruby代码规范,上次公司邀请了外国友人来讲解CSS开发时需要避免的一些误区,我觉得参考这份CSS代码规范,应该会有更深刻的理解.<br/>
&nbsp;&nbsp;闲话不多说,我们主要关注ruby的代码规范,github的ruby代码规范是来对ruby社区代码规范的一个精简,此代码规范可戳[https://github.com/bbatsov/ruby-style-guide](https://github.com/bbatsov/ruby-style-guide).Rubocop也是基于此代码规范实现的,这份规范被翻译成了多种语言,简体中文的版本可见[link](https://github.com/JuanitoFatas/ruby-style-guide/blob/master/README-zhCN.md).规范有点长,但我强烈建议大家通读一遍.我在通读的时候就发现了目前UI组内代码中一些不符合规范或者没有注意的地方,下面主要就摘抄出一些个人认为目前UI组内代码没有严格遵守的规范,并加了一点个人的理解,希望对大家有帮助.还是那句话,规范只有大家都去了解阅读,都去遵守,代码风格统一才有可能,没人了解实行的规范,即使再好,也发挥不了它的价值.<br/>

## 源代码排版

* 对于没有成员的类，尽可能使用单行类定义。`LUO:这个场景主要是自定义异常类时`

    ```Ruby
    # 差
    class FooError < StandardError
    end

    # 勉强可以
    class FooError < StandardError; end

    # 好
    FooError = Class.new(StandardError)
    ```

* 当赋值一个条件表达式的结果给一个变量时，保持分支的缩排在同一层。`LUO:目前组内代码这块做得不是很统一,例子中的1,2种用法比较多,个人建议统一成第3种,行宽更短,结构也更清晰`

    ```Ruby
    # 差 - 非常复杂
    kind = case year
    when 1850..1889 then 'Blues'
    when 1890..1909 then 'Ragtime'
    when 1910..1929 then 'New Orleans Jazz'
    when 1930..1939 then 'Swing'
    when 1940..1950 then 'Bebop'
    else 'Jazz'
    end

    result = if some_cond
      calc_something
    else
      calc_something_else
    end

    # 好 - 结构很清晰
    kind = case year
           when 1850..1889 then 'Blues'
           when 1890..1909 then 'Ragtime'
           when 1910..1929 then 'New Orleans Jazz'
           when 1930..1939 then 'Swing'
           when 1940..1950 then 'Bebop'
           else 'Jazz'
           end

    result = if some_cond
               calc_something
             else
               calc_something_else
             end

    # 好 ( 避免代码让行宽过长 )
    kind =
      case year
      when 1850..1889 then 'Blues'
      when 1890..1909 then 'Ragtime'
      when 1910..1929 then 'New Orleans Jazz'
      when 1930..1939 then 'Swing'
      when 1940..1950 then 'Bebop'
      else 'Jazz'
      end

    result =
      if some_cond
        calc_something
      else
        calc_something_else
      end
    ```

* 在 `def` 之间使用空行，并且用空行把方法分成合乎逻辑的段落。`LUO:在方法定义中应该随时记住对逻辑不同的代码段用空行隔开,对阅读很有帮助,这是对可读性提升很大的一个简单技巧.一个方法通常都有初始化,处理,返回三个逻辑清晰明显的部分`

    ```Ruby
    def some_method
      data = initialize(options)

      data.manipulate!

      data.result
    end

    def some_method
      result
    end
    ```

* 当给方法的参数赋默认值时，在 `=` 两边使用空格：`LUO:这个我平时还没有注意过,不过可读性确实因此增加了不少.对于方法默认参数有时我们可以使用ruby2.0推出的关键字参数,它是个很有用的小技巧.[http://www.oschina.net/question/12_72725](http://www.oschina.net/question/12_72725)`

    ```Ruby
    # 差
    def some_method(arg1=:default, arg2=nil, arg3=[])
      # 做一些任务...
    end

    # 好
    def some_method(arg1 = :default, arg2 = nil, arg3 = [])
      # 做一些任务...
    end
    ```

    虽然几本 Ruby 书建议用第一个风格，不过第二个风格在实践中更为常见（并可争议地可读性更高一点）。

* 使用链式方法时风格统一。社区认为前引点号和末端点号都是好的风格。`LUO:目前组内并没有对这个进行统一,不过好像也没有多少地方有链式方法,个人觉得还是要定一个convention,遵守就好`

   * （可选 A）和当一个链式方法调用需要在另一行继续时，将 `.` 放在第二行。

    ```Ruby
    # 差 - 为了理解第二行需要去查阅第一行
    one.two.three.
      four

    # 好 - 第二行在做什么立刻变得很清晰
    one.two.three
      .four
    ```


    * （可选 B）末尾用点号表示表达式没有结束

    ```Ruby
    # 差 - 需要读到第二行才能确定表达式没有结束
    one.two.three
      .four

    # 好 - 从第一行就可以立即明白表达式没有结束
    one.two.three.
      four
    ```

两种方法各自优点参阅[这里](https://github.com/bbatsov/ruby-style-guide/pull/176)。

* 方法参数过长时，将它对齐排列在多行。当对齐的参数由于线宽不适合对齐时, 简单的在第一行之后缩进也是可以接受的。`LUO:此种情况目前组内代码也没有统一,2,3的做法都可以,重要的是风格统一`

    ```Ruby
    # 初始（行太长了）
    def send_mail(source)
      Mailer.deliver(to: 'bob@example.com', from: 'us@example.com', subject: 'Important message', body: source.text)
    end

    # 差（两倍缩排）
    def send_mail(source)
      Mailer.deliver(
          to: 'bob@example.com',
          from: 'us@example.com',
          subject: 'Important message',
          body: source.text)
    end

    # 好
    def send_mail(source)
      Mailer.deliver(to: 'bob@example.com',
                     from: 'us@example.com',
                     subject: 'Important message',
                     body: source.text)
    end

    # 好（普通缩排）
    def send_mail(source)
      Mailer.deliver(
        to: 'bob@example.com',
        from: 'us@example.com',
        subject: 'Important message',
        body: source.text)
    end
  ```

* 大数字添加下划线来改善可读性。`LUO:这是一个很小的技巧,却真的避免了人脑的许多工作,强烈推荐`

    ```Ruby
    # 差 - 有几个零？
    num = 1000000

    # 好 - 更容易被人脑解析。
    num = 1_000_000
    ```

* 使用 RDoc 以及它的惯例来撰写 API 文档。注解区块及 `def` 不要用空行隔开。`LUO:组内注释时很少用RDoc方式,不过觉得不是什么大问题,最重要的还是代码清晰易读,注释只需要阐述关键信息即可.RDoc还增加了代码维护负担`

## 语法
* 永远不要使用 `for` ，除非你很清楚为什么。大部分情况应该使用迭代器。`for` 是由 `each` 实现的。所以你绕弯了，而且 `for` 没有包含一个新的作用域 (`each` 有 ），因此它区块中定义的变量将会被外部所看到。`LUO:for中作用域和each等不一样我还真没有注意过.个人推荐少用for为好,一个原因是此种描述的作用域问题,另一个是使用each更容易和map等方法联合使用,更为清晰`

    ```Ruby
    arr = [1, 2, 3]

    # 差
    for elem in arr do
      puts elem
    end

    # 注意 elem 会被外部所看到
    elem #=> 3

    # 好
    arr.each { |elem| puts elem }

    # elem 不会被外部所看到
    elem #=> NameError: undefined local variable or method `elem'
    ```

* 利用 if 和 case 是表达式的特性。`LUO: 建议尽量利用ruby中大多数都是表达式的特性`

    ```Ruby
    # 差
    if condition
      result = x
    else
      result = y
    end

    # 好
    result =
      if condition
        x
      else
        y
      end
      ```

* 使用 `!` 替代 `not`。

    ```Ruby
    # 差 - 因为操作符有优先级，需要用括号。
    x = (not something)

    # 好
    x = !something
    ```

* 避免使用 `!!`。`LUO:有些代码仍然有这样的用法`

    ```Ruby
    # 差
    x = 'test'
    # obscure nil check
    if !!x
      # body omitted
    end

    x = false
    # double negation is useless on booleans
    !!x # => false

    # 好
    x = 'test'
    unless x.nil?
      # body omitted
    end
    ```

* `and` 和 `or` 这两个关键字被禁止使用了。 总是使用 `&&` 和 `||` 来取代。

    ```Ruby
    # 差
    # 布尔表达式
    if some_condition and some_other_condition
      do_something
    end

    # 控制流程
    document.saved? or document.save!

    # 好
    # 布尔表达式
    if some_condition && some_other_condition
      do_something
    end

    # 控制流程
    document.saved? || document.save!
    ```

* 单行主体用 `if/unless` 修饰符。另一个好的方法是使用 `&&/||` 控制流程。`LUO:2,3种方法用的都比较多`

    ```Ruby
    # 差
    if some_condition
      do_something
    end

    # 好
    do_something if some_condition

    # 另一个好方法
    some_condition && do_something
    ```

* 避免在多行区块后使用 `if` 或 `unless`。`LUO:有时我会写出这样的代码,最大的坏处就是在阅读时很容易忽视if条件`

    ```Ruby
    # 差
    10.times do
      # multi-line body omitted
    end if some_condition

    # 好
    if some_condition
      10.times do
        # multi-line body omitted
      end
    end
    ```

* 不要使用括号围绕 `if/unless/while` 的条件式。

    ```Ruby
    # 差
    if (x > 10)
      # 此处省略语句体
    end

    # 好
    if x > 10
      # 此处省略语句体
    end
    ```

* 无限循环用 `Kernel#loop`，不用 `while/until` 。`LUO:既然ruby提供了一个更易读更表意的loop方法,就应该用它`

    ```Ruby
    # 差
    while true
      do_something
    end

    until false
      do_something
    end

    # 好
    loop do
      do_something
    end
    ```

* 循环后条件判断使用 `Kernel#loop` 和 `break`，而不是 `begin/end/until` 或者 `begin/end/while`。`LUO:这样确实更为易读,因为阅读begin/end/while代码时,只有读到while时你会意识到这时一个循环,思维的方向和阅读的方向相反.`

   ```Ruby
   # 差
   begin
     puts val
     val += 1
   end while val < 0

   # 好
   loop do
     puts val
     val += 1
     break unless val < 0
   end
   ```

* 忽略围绕方法参数的括号，如内部 DSL (如：Rake, Rails, RSpec)，Ruby 中带有“关键字”状态的方法（如：`attr_reader`，`puts`）以及属性存取方法。所有其他的方法呼叫使用括号围绕参数。`LUO:这区分了关键字方法和其他方法`

    ```Ruby
    class Person
      attr_reader :name, :age

      # 忽略
    end

    temperance = Person.new('Temperance', 30)
    temperance.name

    puts temperance.age

    x = Math.sin(y)
    array.delete(e)

    bowling.score.should == 0
    ```

* 如果方法是内部 DSL 的一部分，那么省略外层的花括号和圆括号。

    ```Ruby
    class Person < ActiveRecord::Base
      # 差
      validates(:name, { presence: true, length: { within: 1..10 } })

      # 好
      validates :name, presence: true, length: { within: 1..10 }
    end
    ```

* 单行区块倾向使用 `{...}` 而不是 `do...end`。多行区块避免使用 `{...}`（多行串连总是​​丑陋）。在 `do...end` 、 “控制流程”及“方法定义”，永远使用 `do...end` （如 Rakefile 及某些 DSL）。串连时避免使用 `do...end`。`LUO: 串联时使用do...end特别丑陋,使用{...}可读性会好一点,但是阅读的负担还是很重,应该考虑抽成方法`

    ```Ruby
    names = ['Bozhidar', 'Steve', 'Sarah']

    # 差
    names.each do |name|
      puts name
    end

    # 好
    names.each { |name| puts name }

    # 差
    names.select do |name|
      name.start_with?('S')
    end.map { |name| name.upcase }

    # 好
    names.select { |name| name.start_with?('S') }.map { |name| name.upcase }
    ```

    某些人会争论多行串连时，使用 `{...}` 看起来还可以，但他们应该扪心自问——这样代码真的可读吗？难道不能把区块内容取出来放到小巧的方法里吗？

* 显性使用区块参数而不是用创建区块字面量的方式传递参数给区块。此规则对性能有所影响，因为区块先被转化为 `Proc`。

    ```Ruby
    require 'tempfile'

    # 差
    def with_tmp_dir
      Dir.mktmpdir do |tmp_dir|
        Dir.chdir(tmp_dir) { |dir| yield dir }  # block just passes arguments
      end
    end

    # 好
    def with_tmp_dir(&block)
      Dir.mktmpdir do |tmp_dir|
        Dir.chdir(tmp_dir, &block)
      end
    end

    with_tmp_dir do |dir| # 使用上面的方法
      puts "dir is accessible as a parameter and pwd is set: #{dir}"
    end
    ```

* 避免在不需要控制流程的场合时使用 `return` 。

    ```Ruby
    # 差
    def some_method(some_arr)
      return some_arr.size
    end

    # 好
    def some_method(some_arr)
      some_arr.size
    end
    ```

* 避免在不需要的情况使用 `self` 。（只有在调用一个 self write 访问器时会需要用到。）`LUO: 在很多代码中我看见了self到处出现`

    ```Ruby
    # 差
    def ready?
      if self.last_reviewed_at > self.last_updated_at
        self.worker.update(self.content, self.options)
        self.status = :in_progress
      end
      self.status == :verified
    end

    # 好
    def ready?
      if last_reviewed_at > last_updated_at
        worker.update(content, options)
        self.status = :in_progress
      end
      status == :verified
    end
    ```

* 变量自赋值用简写方式。

    ```Ruby
    # 差
    x = x + y
    x = x * y
    x = x**y
    x = x / y
    x = x || y
    x = x && y

    # 好
    x += y
    x *= y
    x **= y
    x /= y
    x ||= y
    x &&= y
    ```

* 如果变量未被初始化过，用 `||=` 来初始化变量并赋值。`LUO: ||=和&&=是很有用的语法糖,代码越短,理解越容易`

    ```Ruby
    # 差
    name = name ? name : 'Bozhidar'

    # 差
    name = 'Bozhidar' unless name

    # 好 仅在 name 为 nil 或 false 时，把名字设为 Bozhidar。
    name ||= 'Bozhidar'
    ```

* 不要使用 `||=` 来初始化布尔变量。 （想看看如果现在的值刚好是 `false` 时会发生什么。）`LUO:使用||=需要牢记这一点`

    ```Ruby
    # 差——会把 `enabled` 设成真，即便它本来是假。
    enabled ||= true

    # 好
    enabled = true if enabled.nil?
    ```

* 使用 &&= 可先检查是否存在变量，如果存在则做相应动作。这样就无需用 `if` 检查变量是否存在了。`LUO:这个技巧在代码中很少见到,感觉很好用`

    ```Ruby
    # 差
    if something
      something = something.downcase
    end

    # 差
    something = something ? something.downcase : nil

    # 可以
    something = something.downcase if something

    # 好
    something = something && something.downcase

    # 更好
    something &&= something.downcase
    ```

* 总是使用 `-w` 来执行 Ruby 解释器，如果你忘了某个上述的规则，它就会警告你！`LUO: 这个选项很有用`

* 用新的 lambda 字面语法定义单行区块，用 `lambda` 方法定义多行区块。`LUO: lambda,->,proc其实是有差别的`

    ```Ruby
    # 差
    lambda = lambda { |a, b| a + b }
    lambda.call(1, 2)

    # 正确，但看着怪怪的
    l = ->(a, b) do
    tmp = a * 7
    tmp * b / 50
    end

    # 好
    l = ->(a, b) { a + b }
    l.call(1, 2)

    l = lambda do |a, b|
      tmp = a * 7
      tmp * b / 50
    end
    ```

* 用 `proc` 而不是 `Proc.new`。

    ```Ruby
    # 差
    p = Proc.new { |n| puts n }

    # 好
    p = proc { |n| puts n }
    ```

* 未使用的区块参数和局部变量使用 `_` 前缀或直接使用 `_`（虽然表意性差些） 。Ruby解释器和RuboCop都能辨认此规则，并会抑制相关地有变量未使用的警告。`LUO:_当做占位符,别人就能知道其未被使用`

    ```Ruby
    # 差
    result = hash.map { |k, v| v + 1 }

    def something(x)
      unused_var, used_var = something_else(x)
      # ...
    end

    # 好
    result = hash.map { |_k, v| v + 1 }

    def something(x)
      _unused_var, used_var = something_else(x)
      # ...
    end

    # 好
    result = hash.map { |_, v| v + 1 }

    def something(x)
      _, used_var = something_else(x)
      # ...
    end
    ```

* 当处理你希望将变量作为数组使用，但不确定它是不是数组时，`LUO:我在代码中经常看见很多数组判断,这个技巧拯救这这种丑陋的代码`
  使用 `[*var]` 或 `Array()` 而不是显式的 `Array` 检查。

    ```Ruby
    # 差
    paths = [paths] unless paths.is_a? Array
    paths.each { |path| do_something(path) }

    # 好
    [*paths].each { |path| do_something(path) }

    # 好（而且更具易读性一点）
    Array(paths).each { |path| do_something(path) }
    ```

* 尽量使用范围或 `Comparable#between?` 来替换复杂的逻辑比较。`LUO:选择表意性更好的方式`

    ```Ruby
    # 差
    do_something if x >= 1000 && x < 2000

    # 好
    do_something if (1000...2000).include?(x)

    # 好
    do_something if x.between?(1000, 2000)

    ```

* 尽量用判断方法而不是使用 `==` 。比较数字除外。`LUO:这看起来是有些小题大做,但代码表意性却明显提升了.阅读代码时头脑无需再去思考代码的逻辑意义`

    ```Ruby
    # 差
    if x % 2 == 0
    end

    if x % 2 == 1
    end

    if x == nil
    end

    # 好
    if x.even?
    end

    if x.odd?
    end

    if x.nil?
    end

    if x.zero?
    end

    if x == 0
    end
    ```

* 除非是布尔值，不用显示检查它是否不是 `nil` 。`LUO: 有时会写很多nil?判断,但是根本不需要`

    ```Ruby
    # 差
    do_something if !something.nil?
    do_something if something != nil

    # 好
    do_something if something

    # 好——检查的是布尔值
    def value_set?
      !@some_boolean.nil?
    end
    ```

* 避免使用嵌套的条件来控制流程。
  当你可能断言不合法的数据，使用一个防御从句。一个防御从句是一个在函数顶部的条件声明，这样如果数据不合法就能尽快的跳出函数。`LUO:防御从句是个让代码变得清晰许多的技巧,对比下面的例子,相信你会有很深体会`
    ```Ruby
    # 差
      def compute_thing(thing)
        if thing[:foo]
          update_with_bar(thing)
          if thing[:foo][:bar]
            partial_compute(thing)
          else
            re_compute(thing)
          end
        end
      end

    # 好
      def compute_thing(thing)
        return unless thing[:foo]
        update_with_bar(thing[:foo])
        return re_compute(thing) unless thing[:foo][:bar]
        partial_compute(thing)
      end
    ```

*  使用 `next` 而不是条件区块。`LUO:相比于所有代码都包含在if-end中,next明显代码编排易于阅读`

    ```Ruby
    # 差
    [0, 1, 2, 3].each do |item|
      if item > 1
        puts item
      end
    end

    # 好
    [0, 1, 2, 3].each do |item|
      next unless item > 1
      puts item
    end
    ```

* 倾向使用 `map` 而不是 `collect` ， `find` 而不是 `detect` ， `select` 而不是 `find_all` ， `reduce` 而不是 `inject` 以及 `size` 而不是 `length` 。这不是一个硬性要求；如果使用别名增加了可读性，使用它没关系。这些有押韵的方法名是从 Smalltalk 继承而来，在别的语言不通用。鼓励使用 `select` 而不是 `find_all` 的理由是它跟 `reject` 搭配起来是一目了然的。

* 不要用 `count` 代替 `size`。除了`Array`其它`Enumerable`对象都需要遍历整个集合才能得到大小。`LUO:原来count是遍历集合才能得到结果`

    ```Ruby
    # 差
    some_hash.count

    # 好
    some_hash.size
    ```

* 倾向使用 `flat_map` 而不是 `map` + `flatten` 的组合。
  这并不适用于深度大于 2 的数组，举个例子，如果 `users.first.songs == ['a', ['b', 'c']]` ，则使用 `map + flatten` 的组合，而不是使用 `flat_map`。
  `flat_map` 将数组变平坦一个层级，而 `flatten` 会将整个数组变平坦。`LUO:flat_map少敲了许多代码`

    ```Ruby
    # 差
    all_songs = users.map(&:songs).flatten.uniq

    # 好
    all_songs = users.flat_map(&:songs).uniq
    ```

## 命名

> 程式设计的真正难题是替事物命名及使缓存失效。<br>
> ——Phil Karlton(`LUO:totally agree`)
* 判断式方法的名字（返回布尔值的方法）应以问号结尾。 (例如： `Array#empty?` )。不返回布尔值的方法不应用问号结尾。

* 有潜在**危险性**的方法，若此**危险**方法有安全版本存在时，应以安全版本名加上惊叹号结尾（例如：改动 `self` 或参数、 `exit!` （不会向 `exit` 那样运行 finalizers）, 等等方法）。

* 如果存在潜在的**危险**方法（即修改 `self` 或者参数的方法，不像 `exit` 那样运行 finalizers 的 `exit!`，等等）的安全版本，那么 *危险* 方法的名字应该以惊叹号结尾。`LUO:危险这个词的含义没有统一定义,个人认为是含有让调用者惊异的行为.调用者可能根据方法名望文生义,猜测这个方法背后的逻辑,因此定义方法时,需要以调用者的视角检视方法,如果有可能和其望文生义的猜想不一致,要么就在方法定义时加入注释,要么就以!结尾`

    ```Ruby
    # 不好 - 没有对应的安全方法
    class Person
      def update!
      end
    end

    # 好
    class Person
      def update
      end
    end

    # 好
    class Person
      def update!
      end

      def update
      end
    end
    ```

* 如果可能的话，根据危险方法（bang）来定义对应的安全方法（non-bang）。`LUO:同时提供俩个版本,bang方法的存在才有意义`

    ```Ruby
    class Array
      def flatten_once!
        res = []

        each do |e|
          [*e].each { |f| res << f }
        end

        replace(res)
      end

      def flatten_once
        dup.flatten_once!
      end
    end
    ```

* 在简短区块中使用 `reduce` 时，把参数命名为 `|a, e|` (累加器（`accumulator`），元素（`element`）)`LUO:目前我们对于reduce/map等方法中得参数命名五花八门,个人建议是优先选择符合业务逻辑的名字`

* 在定义二元操作符时，把参数命名为 `other` （`<<` 与 `[]` 是这条规则的例外，因为它们的语义不同）。

    ```Ruby
    def +(other)
      # body omitted
    end
    ```

## 注释

> 良好的代码是最佳的文档。当你要加一个注释时，扪心自问，“如何改善代码让它不需要注释？” 改善代码，再写相应文档使之更清楚。<br>
> ——Steve McConnell

* 编写让人一目了然的代码然后忽略这一节的其它部分。我是认真的！`LUO:优美的代码本身就是一目了然,相比于烂代码+好注释,好的代码永远都更易于阅读`
* 及时更新注释。过时的注解比没有注解还差。

> 好代码就像是好的笑话 - 它不需要解释 <br>
> ——Russ Olsen

* 避免替烂代码写注释。重构代码让它们看起来一目了然。（要嘛就做，要嘛不做——不要只是试试看。——Yoda）

### 注解

* 注解应该直接写在相关代码那行之前。
* 注解关键字后面，跟着一个冒号及空格，接着是描述问题的文字。
* 如果需要用多行来描述问题，后续行要放在 `#` 号后面并缩排两个空格。

    ```Ruby
    def bar
      # FIXME: 这在 v3.2.1 版本之后会异常崩溃，或许与
      #   BarBazUtil 的版本更新有关
      baz(:quux)
    end
    ```
* 在问题是显而易见的情况下，任何的文档会是多余的，注解应放在有问题的那行的最后，并且不需更多说明。这个用法应该是例外而不是规则。

    ```Ruby
    def bar
      sleep 100 # OPTIMIZE
    end
    ```

* 使用 `TODO` 标记以后应加入的特征与功能。
* 使用 `FIXME` 标记需要修复的代码。
* 使用 `OPTIMIZE` 标记可能影响性能的缓慢或效率低下的代码。
* 使用 `HACK` 标记代码异味，即那些应该被重构的可疑编码习惯。
* 使用 `REVIEW` 标记需要确认其编码意图是否正确的代码。举例来说：`REVIEW: 我们确定用户现在是这么做的吗？`
* 如果你觉得恰当的话，可以使用其他定制的注解关键字，但别忘记录在项目的 `README` 或类似文档中。

## 类与模块

* 在类别定义里使用一致的结构。`LUO:有些实践认为类别定义应该达到让人像阅读报纸一样阅读代码的效果,也就是说一个方法中使用到的其他方法定义应该就在它之后,而无需跳转到其他地方阅读这些方法定义.`

    ```Ruby
    class Person
      # 首先是 extend 与 include
      extend SomeModule
      include AnotherModule

      # 接着是常量
      SOME_CONSTANT = 20

      # 接下来是属性宏
      attr_reader :name

      # 跟着是其它的宏（如果有的话）
      validates :name

      # 公开的类别方法接在下一行
      def self.some_method
      end

      # 跟着是公开的实例方法
      def some_method
      end

      # 受保护及私有的方法，一起放在接近结尾的地方
      protected

      def some_protected_method
      end

      private

      def some_private_method
      end
    end
    ```

* 如果某个类需要多行代码，则不要嵌套在其它类中。应将其独立写在文件中，存放以包含它的类的的名字命名的文件夹中。`LUO:是的,代码应该和房间一样,每样东西都放在它该放的地方`

    ```Ruby
    # 差

    # foo.rb
    class Foo
      class Bar
        # 30个方法
      end

      class Car
        # 20个方法
      end

      # 30个方法
    end

    # 好

    # foo.rb
    class Foo
      # 30个方法
    end

    # foo/bar.rb
    class Foo
      class Bar
        # 30个方法
      end
    end

    # foo/car.rb
    class Foo
      class Car
        # 20个方法
      end
    end
    ```

* 倾向使用模块，而不是只有类别方法的类。类别应该只在产生实例是合理的时候使用。`LUO:module和class在此时实现功能没什么区别,但是却明显的有一个好处:调用者知道module不能被生成实例,会避免误用`

    ```Ruby
    # 差
    class SomeClass
      def self.some_method
        # 省略函数体
      end

      def self.some_other_method
      end
    end

    # 好
    module SomeClass
      module_function

      def some_method
        # 省略函数体
      end

      def some_other_method
      end
    end
    ```

* 当你想将模块的实例方法变成类别方法时，偏爱使用 `module_function` 胜过 `extend self`。

    ```Ruby
    # 差
    module Utilities
      extend self

      def parse_something(string)
        # 做一些事
      end

      def other_utility_method(number, string)
        # 做另一些事
      end
    end

    # 好
    module Utilities
      module_function

      def parse_something(string)
        # 做一些事
      end

      def other_utility_method(number, string)
        # 做另一些事
      end
    end
    ```

* 当设计类型层级时，确认它们符合 [Liskov 替换原则](http://en.wikipedia.org/wiki/Liskov_substitution_principle)。`LUO:面向对象设计都应该符合里氏替换原则`
* 尽可能让你的类型越 [SOLID](http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) 越好。
* 永远替类型提供一个适当的 `to_s` 方法给来表示领域模型。`LUO:在rails中这条可能不是那么重要`

    ```Ruby
    class Person
      attr_reader :first_name, :last_name

      def initialize(first_name, last_name)
        @first_name = first_name
        @last_name = last_name
      end

      def to_s
        "#{@first_name #@last_name}"
      end
    end
    ```

* 倾向使用[鸭子类型](http://en.wikipedia.org/wiki/Duck_typing) 而不是继承。`LUO:是的,虽然我们在面向对象编程,但是继承的使用场景其实没那么广阔`

    ```Ruby
    ## 差
    class Animal
      # 抽象方法
      def speak
      end
    end

    # 继承超类
    class Duck < Animal
      def speak
        puts 'Quack! Quack'
      end
    end

    # 继承超类
    class Dog < Animal
      def speak
        puts 'Bau! Bau!'
      end
    end

    ## 好
    class Duck
      def speak
        puts 'Quack! Quack'
      end
    end

    class Dog
      def speak
        puts 'Bau! Bau!'
      end
    end
    ```

* 由于类变量在继承中产生的“讨厌的”行为，避免使用类变量（`@@`）。`LUO:是的,尽量避免使用类变量`

    ```Ruby
    class Parent
      @@class_var = 'parent'

      def self.print_class_var
        puts @@class_var
      end
    end

    class Child < Parent
      @@class_var = 'child'
    end

    Parent.print_class_var # => will print "child"
    ```

    如同你所看到的，在类型层级中的所有类其实都共享单独一个类变量。通常情况下应该倾向使用实例变量而不是类变量。

## 异常

* 使用 `fail` 方法来抛出异常。仅在捕捉到异常时使用 `raise` 来重新抛出异常（因为没有失败，所以只是显式地有目的性地抛出一个异常）`LUO:我们的代码几乎不用fail,fail关键字的表意性更好`

    ```Ruby
    begin
     fail 'Oops';
    rescue => error
      raise if error.message != 'Oops'
    end
    ```

* 尽可能使用隐式的 `begin` 区块。`LUO:是的,这样代码编排更漂亮,也很容易看出来方法有正常逻辑和错误处理俩部分`

    ```Ruby
    # 差
    def foo
      begin
        # 此处放主要逻辑
      rescue
        # 错误处理放在此处
      end
    end

    # 好
    def foo
      # 此处放主要逻辑
    rescue
      # 错误处理放在此处
    end
    ```

* 通过 *contingency* 方法 (一个由 Avdi Grimm 创造的词) 来减少 `begin` 区块的使用。`LUO:这个是很多语言上都会使用的一个技巧,最大的好处就是错误处理和业务逻辑代码分开了,阅读时再也不会被烦人的异常处理代码吸引走注意力,程序员可以完全关注在业务逻辑上`

    ```Ruby
    # 差
    begin
      something_that_might_fail
    rescue IOError
      # 处理 IOError
    end

    begin
      something_else_that_might_fail
    rescue IOError
      # 处理 IOError
    end

    # 好
    def with_io_error_handling
       yield
    rescue IOError
      # 处理 IOError
    end

    with_io_error_handling { something_that_might_fail }

    with_io_error_handling { something_else_that_might_fail }
    ```

* 不要抑制异常。

    ```Ruby
    begin
      # 这里发生了一个异常
    rescue SomeError
      # 拯救子句完全没有做事
    end

    # 差
    do_something rescue nil
    ```

* 避免使用 `rescue` 的修饰符形式。

    ```Ruby
    # 差 - 这捕捉了所有的 StandardError 异常。
    do_something rescue nil
    ```

* 不要为了控制流程而使用异常。`LUO:控制流程不应该和异常混淆`

    ```Ruby
    # 差
    begin
      n / d
    rescue ZeroDivisionError
      puts 'Cannot divide by 0!'
    end

    # 好
    if d.zero?
      puts 'Cannot divide by 0!'
    else
      n / d
    end
    ```

* 避免救援 `Exception` 类别。这会把信号困住，并呼叫 `exit`，导致你需要 `kill -9` 进程。`LUO:永远都记得,细粒度的异常rescue,代码更容易读,从异常类名,我们就能知道什么错误发生了`

    ```Ruby
    # 差
    begin
      # 呼叫 exit 及杀掉信号会被捕捉（除了 kill -9）
      exit
    rescue Exception
      puts "you didn't really want to exit, right?"
      # 异常处理
    end

    # 好
    begin
      # 一个不明确的 rescue 子句捕捉的是 StandardError，
      #   而不是许多编程者所设想的 Exception。
    rescue => e
      # 异常处理
    end

    # 也好
    begin
      # 这里发生一个异常

    rescue StandardError => e
      # 异常处理
    end
    ```

* 把较具体的异常放在救援串连的较上层，不然它们永远不会被拯救。

    ```Ruby
    # 差
    begin
      # 一些代码
    rescue Exception => e
      # 一些处理
    rescue StandardError => e
      # 一些处理
    end

    # 好
    begin
      # 一些代码
    rescue StandardError => e
      # 一些处理
    rescue Exception => e
      # 一些处理
    end
    ```
* 在 `ensure` 区块中释放你的程式的外部资源。

    ```Ruby
    f = File.open('testfile')
    begin
      # .. 处理
    rescue
      # .. 错误处理
    ensure
      f.close unless f.nil?
    end
    ```

* 倾向使用标准库的异常类而不是导入新的异常类。`LUO:相信我,标准库考虑的比大多数人都多`

## 集合

* 倾向数组及哈希的字面表示法（除非你需要给构造器传入参数）。`LUO: 我有见过一些Hash.new的用法,阅读时真的会因此而思维一顿`

    ```Ruby
    # 差
    arr = Array.new
    hash = Hash.new

    # 好
    arr = []
    hash = {}
    ```

* 当你需要一个符号的数组（并且不需要保持 Ruby 1.9 兼容性）时，使用 `%i`。仅当数组只有两个及以上元素时才应用这个规则。`LUO:真的可以少键入很多:`

    ```Ruby
    # 差
    STATES = [:draft, :open, :closed]

    # 好
    STATES = %i(draft open closed)
    ```

* 避免在数组中创造巨大的间隔。`LUO:ruby中的数组并没有想象的那么智能`

    ```Ruby
    arr = []
    arr[100] = 1 # 现在你有一个很多 nil 的数组
    ```

* 当访问数组的首元素或尾元素时，尽量使用 `first` 或 `last`， 而非 `[0]` 或 `[-1]`。

* 当处理的元素没有重复时，使用 `Set` 来替代 `Array` 。 `Set` 实现了无序、无重复值的集合。 `Set` 的方法同数组类一样直观，还可像哈希中那样快速查找元素。`LUO:是的,Set性能高很多很多`

* 避免使用可变的对象作为键值。`LUO:如果可变对象当键值,那就会有很多可怕地事情发生了`

* 当哈希的键为symbol时，使用 Ruby 1.9 的哈希的字面语法。`LUO:推荐使用这种方式,更为简练`

    ```Ruby
    # 差
    hash = { :one => 1, :two => 2, :three => 3 }

    # 好
    hash = { one: 1, two: 2, three: 3 }
    ```

* 用 `Hash#key?`。不用 `Hash#has_key?`。用 `Hash#value?`。不用 `Hash#has_value?`。松本提到过已经不推荐使用较长的形式了。`LUO:算是一个小小的改变吧,个人觉得俩种都可读性差不多`

    ```Ruby
    # 差
    hash.has_key?(:test)
    hash.has_value?(value)

    # 好
    hash.key?(:test)
    hash.value?(value)
    ```

* 在处理应该存在的哈希键时，使用 `Hash#fetch`。

    ```Ruby
    heroes = { batman: 'Bruce Wayne', superman: 'Clark Kent' }
    # 差 - 如果我们打错字的话，我们就无法找到对的英雄了
    heroes[:batman] # => "Bruce Wayne"
    heroes[:supermen] # => nil

    # 好 - fetch 会抛出一个 KeyError 来使这个问题明显
    heroes.fetch(:supermen)
    ```

* 在使用 `Hash#fetch` 时，使用第二个参数设置默认值。

   ```Ruby
   batman = { name: 'Bruce Wayne', is_evil: false }

   # 差 - 如果我们仅仅使用 || 操作符，那么当值为假时，我们不会得到预期的结果
   batman[:is_evil] || true # => true

   # 好 - fetch 在遇到假值时依然正确
   batman.fetch(:is_evil, true) # => false
   ```

* 当需要从哈希中同时获取多个键值时，使用 `Hash#values_at`。`LUO:记住代码越少,越容易阅读,越不容易出错`

  ```Ruby
  # 差
  email = data['email']
  username = data['nickname']

  # 好
  email, username = data.values_at('email', 'nickname')
  ```

* Ruby 1.9 的哈希是有序的，利用这个特性。

* 在遍历一个集合时，不要改动它。

* 当访问集合中的元素时，避免通过 `[n]` 直接访问，尽量使用提供的方法。这样可以防止你对 `nil` 调用 `[]`。`LUO:nil是个很烦的东西,判断nil真的很烦`

  ```Ruby
  # 差
  Regexp.last_match[1]

  # 好
  Regexp.last_match(1)
  ```

* 为集合提供存取器时，在访问元素之前采用一种替代的形式，从而防止用户访问的下标是 `nil`。

  ```Ruby
  # 差
  def awesome_things
    @awesome_things
  end

  # 好
  def awesome_things(index = nil)
    if index && @awesome_things
      @awesome_things[index]
    else
      @awesome_things
    end
  end
  ```

## 字符串

* 选定一个字符串字面量创建的风格。Ruby 社区认可两种分割，默认用单引号（风格 A）和默认用双引号（风格 B）

  * **（风格 A）**当你不需要插入特殊符号如 `\t`, `\n`, `'`, 等等时，尽量使用单引号的字符串。`LUO:单引号会抑制转义`

    ```Ruby
    # 差
    name = "Bozhidar"

    # 好
    name = 'Bozhidar'
    ```

  * **（风格 B）** 用双引号。除非字符串中含有双引号，或者含有你希望抑制的逃逸字符。

    ```Ruby
    # 差
    name = 'Bozhidar'

    # 好
    name = "Bozhidar"
    ```

  有争议的是，第二种风格在 Ruby 社区里更受欢迎一些。但是本指南中字符串采用第一种风格。

* 不要用 `?x`。从 Ruby 1.9 开始， `?x` 和 `'x'` 是等价的（只包括一个字符的字符串）。`LUO:Ruby 1.9的新变化`

    ```Ruby
    # 差
    char = ?c

    # 好
    char = 'c'
    ```

* 当你需要建构庞大的数据块（chunk）时，避免使用 `String#+` 。
  使用 `String#<<` 来替代。`<<` 就地改变字符串实例，因此比 `String#+` 来得快。`String#+` 创造了一堆新的字符串对象。`LUO:<<性能更好`

    ```Ruby
    # 好也比较快
    html = ''
    html << '<h1>Page title</h1>'

    paragraphs.each do |paragraph|
      html << "<p>#{paragraph}</p>"
    end
    ```

* heredocs 中的多行文字会保留前缀空白。因此做好如何缩进的规划。

    ```Ruby
    code = <<-END.gsub(/^\s+\|/, '')
      |def test
      |  some_method
      |  other_method
      |end
    END
    #=> "def\n  some_method\n  \nother_method\nend"
    ```

## 正则表达式

> 有些人在面对问题时，不经大脑便认为，「我知道，这里该用正则表达式」。现在他要面对两个问题了。<br>
> ——Jamie Zawinski

* 如果只需要在字符串中简单的搜索文字，不要使用正则表达式：`string['text']`。`LUO:正则表达式的代价比较大`

* 针对简单的字符串查询，可以直接在字符串索引中直接使用正则表达式。`LUO:使用的小技巧,也更为可读`

    ```Ruby
    match = string[/regexp/] # 获得匹配正则表达式的内容
    first_group = string[/text(grp)/, 1] # 或得分组的内容
    string[/text (grp)/, 1] = 'replace' # string => 'text replace'
    ```
* 当你不需要替结果分组时，使用非分组的群组。`LUO:分组的成本很高,需要记住分组内容,具体可参见正则表达式实现相关文章`

    ```Ruby
    /(first|second)/ # 差
    /(?:first|second)/ # 好
    ```

* 避免使用数字来获取分组。因为很难明白他们代表的意思。应该使用命名群组来替代。`LUO:命名比1,2等等更为可读`

    ```Ruby
    # 差
    /(regexp)/ =~ string
    ...
    process Regexp.last_match[1]

    # 好
    /(?<meaningful_var>regexp)/ =~ string
    ...
    process meaningful_var
    ```

* 字符类别只有几个你需要关心的特殊字符：`^`、`-`、`\`、`]`，所以你不用转义 `[]` 中的 `.` 或中括号。

* 小心使用 `^` 与 `$` ，它们匹配的是一行的开始与结束，不是字符串的开始与结束。如果你想要匹配整个字符串，使用 `\A` 与 `\z`。(译注：`\Z` 实为 `/\n?\z/`，使用 `\z` 才能匹配到有含新行的字符串的结束)`LUO:单行与多行模式时,^/$匹配的内容也不一样`

    ```Ruby
    string = "some injection\nusername"
    string[/^username$/] # 匹配
    string[/\Ausername\z/] # 不匹配
    ```
* 针对复杂的正则表达式，使用 `x` 修饰符。可提高可读性并可以加入有用的注释。只是要注意空白字符会被忽略。

    ```Ruby
    regexp = %r{
      start # 一些文字
      \s # 空白字元
      (group) # 第一组
      (?:alt1|alt2) # 一些替代方案
      end
    }x
    ```

* 针对复杂的替换，`sub` 或 `gsub` 可以与区块或哈希结合使用。

## 元编程

* 避免无谓的元编程。`LUO:虽然元编程很酷炫,但也不要滥用了`

* 写一个函数库时不要使核心类混乱（不要使用 monkey patch）。`LUO:除非迫不得已,否则不要修改标准库中得核心类`

* 倾向使用区块形式的 `class_eval` 而不是字符串插值（string-interpolated）的形式。
  - 当你使用字符串插值形式时，总是提供 `__FILE__` 及 `__LINE__`，使你的 backtrace 看起来有意义：

    ```ruby
    class_eval "def use_relative_model_naming?; true; end", __FILE__, __LINE__
    ```

  - 倾向使用 `define_method` 而不是 `class_eval{ def ... }`

* 当使用 `class_eval` （或其它的 `eval`）搭配字符串插值时，添加一个注解区块，来演示如果做了插值的样子（我从 Rails 代码学来的一个实践）：

    ```ruby
    # activesupport/lib/active_support/core_ext/string/output_safety.rb
    UNSAFE_STRING_METHODS.each do |unsafe_method|
      if 'String'.respond_to?(unsafe_method)
        class_eval <<-EOT, __FILE__, __LINE__ + 1
          def #{unsafe_method}(*args, &block) # def capitalize(*args, &block)
            to_str.#{unsafe_method}(*args, &block) # to_str.capitalize(*args, &block)
          end # end

          def #{unsafe_method}!(*args) # def capitalize!(*args)
            @dirty = true # @dirty = true
            super # super
          end # end
        EOT
      end
    end
    ```

* 元编程避免使用 `method_missing`。会让 Backtraces 变得很凌乱；行为没有列在 `#methods` 里；拼错的方法调用可能默默的工作（`nukes.launch_state = false`）。考虑使用 delegation, proxy, 或是 `define_method` 来取代。如果你必须使用 `method_missing`，
  - 确保 [也定义了 `respond_to_missing?`](http://blog.marc-andre.ca/2010/11/methodmissing-politely.html)
  - 仅捕捉字首定义良好的方法，像是 `find_by_*`——让你的代码愈肯定（assertive） 愈好。
  - 在语句的最后调用 `super`
  - delegate 到确定的、非魔法方法中:

    ```ruby
    # 差
    def method_missing?(meth, *args, &block)
      if /^find_by_(?<prop>.*)/ =~ meth
        # ... lots of code to do a find_by
      else
        super
      end
    end

    # 好
    def method_missing?(meth, *args, &block)
      if /^find_by_(?<prop>.*)/ =~ meth
        find_by(prop, *args, &block)
      else
        super
      end
    end

    # 最好的方式，可能是每个可找到的属性被声明后，使用 define_method。
    ```

## 其它

* `ruby -w` 写安全的代码。
* 避免使用哈希作为可选参数。这个方法是不是做太多事了？（对象初始器是本规则的例外）。
* 避免方法长于 10 行代码（LOC）。理想上，大部分的方法会小于 5 行。空行不算进 LOC 里。
* 避免参数列表长于三或四个参数。
* 如果你真的需要“全局”方法，把它们加到 Kernel 并设为私有的。
* 使用模块变量代替全局变量。

    ```Ruby
    # 差
    $foo_bar = 1

    # 好
    module Foo
      class << self
        attr_accessor :bar
      end
    end

    Foo.bar = 1
    ```

* 使用 `OptionParser` 来解析复杂的命令行选项及 `ruby -s` 来处理琐碎的命令行选项。
* 使用 `Time.now` 而不是 `Time.new` 来获取系统时间。
* 用函数式的方法编程，在有意义的情况下避免赋值 (mutation)。
* 不要改变参数，除非那是方法的目的。
* 避免超过三层的区块嵌套。
* 保持一致性。在理想的世界里，遵循这些准则。
* 使用常识。

## 工具

以下是一些工具，让你自动检查 Ruby 代码是否符合本指南。

### RuboCop

[RuboCop](https://github.com/bbatsov/rubocop) 是一个基于本指南的 Ruby 代码风格检查工具。 RuboCop 涵盖了本指南相当大的部分，支持 MRI 1.9 和 MRI 2.0，而且与 Emacs 整合良好。

### RubyMine

[RubyMine](http://www.jetbrains.com/ruby/) 的代码检查[部分基于](http://confluence.jetbrains.com/display/RUBYDEV/RubyMine+Inspections)本指南。

# 贡献

在本指南所写的每条规则都不是定案。这只是我渴望想与同样对 Ruby 编程风格有兴趣的大家一起工作，以致于最终我们可以替整个 Ruby 社区创造一个有益的资源。

欢迎 open tickets 或 push 一个带有改进的更新请求。在此提前感谢你的帮助！

## 如何贡献？
很简单，只需要参考 [贡献准则](https://github.com/bbatsov/ruby-style-guide/blob/master/CONTRIBUTING.md)。

# 授权

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/deed.zh)

# 口耳相传

一份社区驱动的风格指南，如果没多少人知道，对一个社区来说就没有多少用处。微博转发这份指南，分享给你的朋友或同事。我们得到的每个评价、建议或意见都可以让这份指南变得更好一点。而我们想要拥有的是最好的指南，不是吗？

共勉之，<br>
[Bozhidar](https://twitter.com/bbatsov)