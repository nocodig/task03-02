# task03-02

# 1、简述Vue首次渲染的过程


Vue首先执行初始化实例成员、静态成员，执行Vue构造函数，在构造函数中调用了this._init，在该函数中，调用了第一个

vm.$mount：该方法时入口文件(src\platforms\web\entry-runtime-with-compiler.js)中的$mount，即重写后的$mount，核心作用的将模板转换成render函数，首先会判断有没有传入render，如果没有传入，会获取template选项，如果也没有template选项，会将el中的内容作为模板，转换成render函数，通过compileToFunctions生成render函数，当把render函数编译好后，存放在options.render中。(运行时版本不会执行该部分)
上述过程结束后

然后调用第二个$mount方法，即 src\platforms\web\runtime\index.js文件中的$mount方法，然后重新获取el，调用mountComponent方法，该方法在src\core\instance\lifecycle.js文件中，首先判断是否有render选项，如果没有render选项，且传入了模板，而且又是开发环境，会爆出警告，然后出发了beforemount钩子函数，定义了updateComponent，此时仅仅定义该函数，在该函数中，调用了vm._render()方法以及vm._update()方法，_render方法是渲染虚拟DOm，_update是更新，将虚拟DOM转换成真实DOM。

在然后创建了Watcher实例，将上面声明的updateComponent函数传入watcher对象，然后调用watcher内部的get方法，当创建完watcher是，首先会调用一次get方法，在get方法中会调用updateComponent
最后触发mounted方法

# 2、请简述Vue响应式原理

- 首先Vue的init方法中调用initState，初始化Vue实例的状态，在该函数中调用了initDate，在initData中调用Observe，将data对象转换成响应式对象

- observe是响应式的入口, 在observe(value)中，首先判断传入的参数value是否是对象，如果不是对象直接返回。再判断value对象是否有__ob__这个属性，如果有说明做过了响应式处理，则直接返回，如果没有，创建observer对象，并且返回observer`对象。


- 在创建observer对象时，给当前的value对象定义不可枚举的__ob__属性，记录当前的observer对象，然后再进行数组的响应式处理和对象的响应式处理，数组的响应式处理就是拦截数组的几个特殊的方法，push、pop、shift等，然后找到数组对象中的__ob__对象中的dep,调用dep的notify()方法，再遍历数组中每一个成员，对每个成员调用observer()，如果这个成员是对象的话，也会转换成响应式对象。对象的响应式处理，就是调用walk方法，walk方法就是遍历对象的每一个属性，对每个属性调用defineReactive方法


- defineReactive会为每一个属性创建对应的dep对象，让dep去收集依赖，如果当前属性的值是对象，会调用observe。defineReactive中最核心的方法是getter 和 setter。getter 的作用是收集依赖，收集依赖时, 为每一个属性收集依赖，如果这个属性的值是对象，那也要为子对象收集依赖，最后返回属性的值。在setter 中，先保存新值，如果新值是对象，也要调用 observe ，把新设置的对象也转换成响应式的对象,然后派发更新（发送通知），调用dep.notify()


- 收集依赖时，在watcher对象的get方法中调用pushTarget,记录Dep.target属性，访问data中的成员的时候收集依赖，defineReactive的getter中收集依赖，把属性对应的 watcher 对象添加到dep的subs数组中，给childOb收集依赖，目的是子对象添加和删除成员时发送通知。


- 在数据发生变化的时候，会调用dep.notify()发送通知，dep.notify()会调用watcher对象的update()方法，update()中的调用的queueWatcher()会去判断watcher是否被处理，如果这个watcher对象没有的话添加到queue队列中，并调用flushScheduleQueue()，flushScheduleQueue()触发beforeUpdate钩子函数调用watcher.run()： run()-->get() --> getter() --> updateComponent()

# 3、简述虚拟DOM中Key的作用和好处

v-for遍历的时候，能够追踪每个节点的身份，在进行新旧虚拟DOM节点比较的时候，会基于key的变化重新排列元素的顺序，从而重用和重新排序现有的元素，并且移除key不存在的元素，方便在diff过程中找到对应的节点，然后复用，从而减少dom的操作。

# 4、请简述Vue 模板编译的过程

- 模版编译入口函数compileToFunctions
  - 内部首先从缓存加载编译好的render函数
  - 如果缓存中没有，调用compile开始编译
- 在 compile 函数中
  - 首先合并选项options
调用 baseCompile 编译模版
compile的核心是合并选项options， 真正处理是在basCompile中完成的，把模版和合并好的选项传递给baseCompile, 这里面完成了模版编译的核心三件事情
  - parse()
    - 把模版字符串转化为AST 对象，也就是抽象语法树
  - optimize()
    - 对抽象语法树进行优化，标记静态语法树中的所以静态根节点
    - 检测到静态子树，设置为静态，不需要在每次重新渲染的时候重新生成节点
    - patch的过程中会跳过静态根节点
- generator()
  - 把优化过的AST对象，转化为字符串形式的代码
- 执行完成之后，会回到入口函数
  - complieToFunctions
compileToFunction会继续把字符串代码转化为函数
  - 调用createFunction
  - 当 render 和 staticRenderFns初始化完毕，最终会挂在到Vue实例的options对应的属性中