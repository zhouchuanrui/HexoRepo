title: UVM源码探索之Factory(上篇)
Status: Public
Tags: UVM 
date: 2015-8-27 22:04:39
---

[TOC]

Factory机制是UVM的一个重要的底层机制, 内容不多, 源码来讲大概2000行左右, 跟sequence, 寄存器模型, TLM这些大部头来讲很少了. 配合这篇笔记再加上源码我估计一个下午就可以了解掌握的差不多了. 

这篇笔记的是我在学习UVM的Factory源码的时候的一些记录跟总结, 整理为分为上下两篇. 上篇讲述的是`uvm_object`的Factory实现, 下篇再讲一些UVM Factory的其他功能. 使用的源码是`uvm-1.2`. 这篇笔记写的很浅, 讲的比较细很容易看懂, 要是读者有OOP的基础(多态和泛型)那就更轻松了. 

# 我眼中的Factory

讲述工厂模式的书会告诉你, 工厂模式可以让你动态的生成对象(这其实是句废话啦, 对象其实都是动态也就是运行时生成的嘛), 比如用字符串为参数生成指定的对象.

根据这个描述, UVM的Factory在我想象中是这样的:

<!--more-->

```
class uvm_factory#(type T=uvm_object);
    static T queue[string];
    static void function register(T obj, string name);
        queue[name] = obj;
    endfunction
    static T function create(string name);
        if (queue.exists(name))
            return queue[name];
        else
            return null;
    endfunction
endclass 
```

非常简单, 配置了类型参数`T`, 而且设置默认值为`uvm_object`. 跟其他OOP系统一样, object是所有其他对象的祖类, 这个对UVM用户来说是成立的, 因为用户要么用的是`uvm_object`(比如`uvm_sequence_item`, `uvm_sequence`), 要么是继承自`uvm_object`的`uvm_component`(比如`uvm_driver`, `uvm_test`). 不过如果你看过一些书你会知道`uvm_object`还有个父类`uvm_void`, 要是你看源码你会发现还有很多其他的事情(嗯~). `queue`是用来存放对象的哈希表, 用字符串索引, 然后方法`register`和`create`用来注册类型和生成类型. 这些成员和对象设置为静态保证这个Factory是唯一的用类名直接使用的.

假定现在我们对类型为`uvm_user_type`的对象进行操作, 注册的时候是这样的:

```
uvm_user_type obj = new();
uvm_factory::register(obj.clone(), "uvm_user_type");
```

保险起见还是用个clone出来的对象来注册, 其中`clone()`方法需要自行实现, 而且返回值是`uvm_user_type`对象的, Systemverilog是静态类型语言, 用起来要麻烦一点. 然后`uvm_user_type`的对象就跟字符串`"uvm_user_type"`绑定了, 需要生成对象的时候就这样:

```
uvm_user_type obj, tmp;
$cast(tmp, uvm_factory::create("uvm_user_type"))
obj = tmp.clone();
```

实现的简单用起来就有点麻烦了对吧. 来看看UVM里面是怎么实现Factory的吧.

# UVM中的Factory

基本上, 大家在使用UVM做验证平台的时候, 关于Factory的操作有三处: 

1. top中的`run_test("test_name")`;
1. 创建对象时使用的`uvm_user_type::type_id::create("<name>")`;
1. 类定义中使用的宏`uvm_object_util(<class_name>)`和`uvm_component_util(<class_name>)`;

其中`run_test()`是最符合工厂的特性的, 它根据传入的字符串或者从运行参数`+UVM_TESTNAME="test_name"`读取字符串来生成`uvm_test`对象, 并运行这个对象的全部`phase`函数来执行一次测试用例的运行. 

然后是`create()`的函数, 仔细看的话, 你会发现这个函数是跟你要生成的类名绑定的, 是这个类下面的一个静态方法; 而且它的参数也不是跟类绑定的什么字符串, 可以是任意的字符串, 有点奇怪对不对.

至于注册宏, 这是什么鬼(UVM中很多宏都会给我这个感觉). 不过答案可能就在这个宏里面了. 我们就从`uvm_object_util()`宏开始, 来探索UVM中的Factory实现吧.

## 从Factory注册宏开始

`uvm_object_util()`的定义在文件`/src/macros/uvm_object_defines.svh`里, 看看里面是什么吧:

```
`define uvm_object_utils(T) \
  `uvm_object_utils_begin(T) \
  `uvm_object_utils_end
```

是两个宏, 还要继续继续往下翻才行:

```
`define uvm_object_utils_begin(T) \
   `m_uvm_object_registry_internal(T,T)  \
   `m_uvm_object_create_func(T) \
   `m_uvm_get_type_name_func(T) \
   `uvm_field_utils_begin(T) 
```

又出来四个宏, 看来强行看源码需要容量不小的人脑堆栈才行, 这个时候做笔记的好处就出来了. 继续来看吧:

```
`define m_uvm_object_registry_internal(T,S) \
   typedef uvm_object_registry#(T,`"S`") type_id; \
   static function type_id get_type(); \
     return type_id::get(); \
   endfunction \
   virtual function uvm_object_wrapper get_object_type(); \
     return type_id::get(); \
   endfunction 
```

这个宏做了三件事情(嗯):

1. `type_id`的类型定义. 再回头看看生成对象的`create`函数的`::type_id`, 其实就是引用的这个类`uvm_object_registry`. 这个类从名字上看就知道是用来注册的类啦, 现在看不懂也没关系. 这个类的参数设置的有点意思`#(T,"S")`, 从调用看这里的`S`其实是`T`, 也就是注册的类, 这里用这种方式生成了同名的字符串, 这样似乎就把类型跟字符串绑定到一起了, 略显巧妙啊.
1. `get_type`函数, 直接引用的`uvm_object_registry`的`get`函数, 是个静态方法, 后面再看吧.
1. `get_object_type`函数, 还是引用的`uvm_object_registry`的`get`函数, 不过返回值是`uvm_object_wrapper`(这个类型想必是object的子类吧? 是不是呢?), 后面再看吧.

再来看看`m_uvm_object_create_func`这个宏:

```
`define m_uvm_object_create_func(T) \
   function uvm_object create (string name=""); \
     T tmp; \
`ifdef UVM_OBJECT_DO_NOT_NEED_CONSTRUCTOR \
     tmp = new(); \
     if (name!="") \
       tmp.set_name(name); \
`else \
     if (name=="") tmp = new(); \
     else tmp = new(name); \
`endif \
     return tmp; \
   endfunction
```

这个宏看来就是`create`函数了, 而且返回值就是`uvm_object`, 不过这个`name`参数果然是跟类型没有关系的. 然而这个函数还不是`type_id::create`这个函数呢, 这两个函数有什么关系呢?

然后是`m_uvm_get_type_name_func`这个宏:

```
`define m_uvm_get_type_name_func(T) \
   const static string type_name = `"T`"; \
   virtual function string get_type_name (); \
     return type_name; \
   endfunction 
```

这里又用到了把类型转换为字符串的技巧, 这种东西只能用宏来实现, 果然宏才是大boss. 这个函数返回了类型字符串.

然后是`uvm_field_utils_begin`这个宏, 这个宏其实已经是field automation功能的宏了:

```
`define uvm_field_utils_begin(T) \
   function void __m_uvm_field_automation (uvm_object tmp_data__, \
                                     int what__, \
                                     string str__); \
   begin \
     T local_data__; /* Used for copy and compare */ \
     typedef T ___local_type____; \
     string string_aa_key; /* Used for associative array lookups */ \
     uvm_object __current_scopes[$]; \
     if(what__ inside {UVM_SETINT,UVM_SETSTR,UVM_SETOBJ}) begin \
        if(__m_uvm_status_container.m_do_cycle_check(this)) begin \
            return; \
        end \
        else \
            __current_scopes=__m_uvm_status_container.m_uvm_cycle_scopes; \
     end \
     super.__m_uvm_field_automation(tmp_data__, what__, str__); \
     /* Type is verified by uvm_object::compare() */ \
     if(tmp_data__ != null) \
       /* Allow objects in same hierarchy to be copied/compared */ \
       if(!$cast(local_data__, tmp_data__)) return;
```

你看是不是很复杂而且跟Factory没有关系, 很多资料都指出field automation有性能问题, 大家还是不要用的好.

再跳回上一级调用堆栈看看最后一个宏:

```
`define uvm_object_utils_end \
     end \
   endfunction \
```

完全是为了关闭field automation的函数定义而已.

假设我们使用这个宏的形式是这样的:

```
class user_object extends uvm_transaction;
    `uvm_object_util(user_object)
    ...
endclass
```

现在人肉展开一下(并且忽略field函数), 就可以看出来这个宏做了哪些事情了:

```
   typedef uvm_object_registry#(user_object,"user_object") type_id; 
   static function type_id get_type(); 
     return type_id::get(); 
   endfunction 
   virtual function uvm_object_wrapper get_object_type(); 
     return type_id::get(); 
   endfunction 

   function uvm_object create (string name=""); 
     user_object tmp; 
     tmp = new(); 
     if (name!="") 
       tmp.set_name(name); 
     return tmp; 
   endfunction

   const static string type_name = "user_object"; 
   virtual function string get_type_name (); 
     return type_name; 
   endfunction 
```

为你的类定义生成了上面显示的这些成员和方法. 这个宏的内容就是这些了, 剩下的疑问都要从`uvm_object_registry`这个类里面找答案了.

## 探索uvm_object_registry

grep一下可以发现, `uvm_object_registry`这个类的定义在`src/base/uvm_registry.svh`里面, 不是`uvm_factory.svh`哦, 是不是感觉就有点蹊跷呢? 这个文件里面定义了定义了`uvm_object_registry`和`uvm_component_registry`两个类, 我们要关注的是前者, 至于后者就`uvm_component_registry`下篇再讲吧(那是什么时候?).

这两个类都是比较大的定义了, 具体的`uvm_object_registry`的定义在第189行到第316行, 其实大部分是注释啦, 把注释去掉看看:

```
class uvm_object_registry #(type T=uvm_object, string Tname="<unknown>")
                                        extends uvm_object_wrapper;
  typedef uvm_object_registry #(T,Tname) this_type;
  virtual function uvm_object create_object(string name="");
    T obj;
`ifdef UVM_OBJECT_DO_NOT_NEED_CONSTRUCTOR
    obj = new();
    if (name!="")
      obj.set_name(name);
`else
    if (name=="") obj = new();
    else obj = new(name);
`endif
    return obj;
  endfunction

  const static string type_name = Tname;
  virtual function string get_type_name();
    return type_name;
  endfunction
    ...    
endclass
```

(注意到这个类的父类了吗?)先看最前面的部分, 有点眼熟对不对, `this_type`的定义跟`m_uvm_object_registry_internal`中`type_id`的定义是一样的. 而`create_object`跟`m_uvm_object_create_func`中的`create`函数是一模一样的, 这是为什么呢? 然后是`type_name`字符串跟`get_type_name`, 这些又跟`m_uvm_get_type_name_func`宏做了一样的事情, 为什么`registry`类里定义了, 又要用宏在用户定义类里再定义一次呢? 不过这些东西还没到最需要关注的地方. 再往下看看吧:

```
class uvm_object_registry #(type T=uvm_object, string Tname="<unknown>")
                                        extends uvm_object_wrapper;
    ...
  local static this_type me = get();
  static function this_type get();
    if (me == null) begin
      uvm_coreservice_t cs = uvm_coreservice_t::get();
      uvm_factory factory=cs.get_factory();
      me = new;
      factory.register(me);
    end
    return me;
  endfunction
    ...
endclass
```

这个`me`表示的是本类型的一个静态对象, 是调用`get`来返回的. `get`里面出现了没见过的东西了, `uvm_coreservice_t`, 这个是什么? 不过强行看下来, 可以发现`get`做的工作就是, `me`对象不为空的话就创建, 然后用`factory`的`register`方法来注册这个`me`对象, 最后将`me`这个对象返回, 这些工作可以保证最多执行一次. 总的说起来, `get`函数`new`出了`me`, 然后用`factory`注册了`me`, 有点眉目了对不对. 再往下看:

```
class uvm_object_registry #(type T=uvm_object, string Tname="<unknown>")
                                        extends uvm_object_wrapper;
    ...
  static function T create (string name="", uvm_component parent=null,
                            string contxt="");
    uvm_object obj;
    uvm_coreservice_t cs = uvm_coreservice_t::get(); 
    uvm_factory factory=cs.get_factory();
    if (contxt == "" && parent != null)
      contxt = parent.get_full_name();
    obj = factory.create_object_by_type(get(),contxt,name);
    if (!$cast(create, obj)) begin
      string msg;
      msg = {"Factory did not return an object of type '",type_name,
        "'. A component of type '",obj == null ? "null" : obj.get_type_name(),
        "' was returned instead. Name=",name," Parent=",
        parent==null?"null":parent.get_type_name()," contxt=",contxt};
      uvm_report_fatal("FCTTYP", msg, UVM_NONE);
    end
  endfunction
    ...
endclass
```

总算看到`create`这个静态函数了对不对. 不过, 这个函数参数有点复杂, `parent`是怎么回事, `context`是什么; 而且`uvm_coreservice_t`这个东西又出现了. 

从之前的调用来看, `parent`和`context`都是空的. 然后这个函数的核心尽在:

```
    obj = factory.create_object_by_type(get(),"",name);
    if (!$cast(create, obj)) begin
```

这两行, 一个是类型的创建, 一个是类型cast. 还记得前面的`get`吗, 这里直接调用了`get`保证直接调用`create`时候Factory已经注册了这个类型. 这就是我们想看到的东西了对不对. 不过看起来谜底都要去`uvm_factory`里面找了.

```
class uvm_object_registry #(type T=uvm_object, string Tname="<unknown>")
                                        extends uvm_object_wrapper;
    ...
  static function void set_type_override (uvm_object_wrapper override_type,
                                          bit replace=1);
    uvm_coreservice_t cs = uvm_coreservice_t::get();
    uvm_factory factory=cs.get_factory();
    factory.set_type_override_by_type(get(),override_type,replace);
  endfunction

  static function void set_inst_override(uvm_object_wrapper override_type,
                                         string inst_path,
                                         uvm_component parent=null);
    string full_inst_path;
    uvm_coreservice_t cs = uvm_coreservice_t::get();
    uvm_factory factory=cs.get_factory();
    if (parent != null) begin
      if (inst_path == "")
        inst_path = parent.get_full_name();
      else
        inst_path = {parent.get_full_name(),".",inst_path};
    end
    factory.set_inst_override_by_type(get(),override_type,inst_path);
  endfunction
endclass
```

最后面是两个用于重载的函数, 这两个也是用`factory`来实现的. 重载也是UVM里面的Factory的一大功能, 不过普世的纯粹的Factory是不用关注这些的. 这里我们需要关注的是`uvm_factory`里的`register`方法和`create_object_by_type`方法.

不过在此之前, 我觉得需要先看看`uvm_coreservice_t`, 去探索一下为什么要用这种方式来使用Factory.

### 插播uvm_coreservice_t

`uvm_coreservice_t`的位置在`src/base/uvm_coreservice.svh`, 从名字来看似乎是UVM的基础设施呢. 来看看定义:

```
virtual class uvm_coreservice_t;
	pure virtual function uvm_factory get_factory();
	pure virtual function void set_factory(uvm_factory f);
	pure virtual function uvm_report_server get_report_server();
	pure virtual function void set_report_server(uvm_report_server server);
        pure virtual function uvm_tr_database get_default_tr_database();
        pure virtual function void set_default_tr_database(uvm_tr_database db);
	pure virtual function void set_component_visitor(uvm_visitor#(uvm_component) v);
	pure virtual function uvm_visitor#(uvm_component) get_component_visitor();
	pure virtual function uvm_root get_root();

	local static `UVM_CORESERVICE_TYPE inst;
	static function uvm_coreservice_t get();
		if(inst==null)
			inst=new;
		return inst;
	endfunction 
endclass
```

这竟然是一个抽象类, 里面的成员都是纯虚函数, 从名字上看返回的果然都是一些UVM的基础设施; 只在最后部分实现了一个`get`方法, 这个方法返回了`UVM_CORESERVICE_TYPE`这个类型的`inst`对象, 而且创建方式是跟之前的`registry`是一样的. 还有个地方不知道大家注意到没有, 这里的代码缩进是4个空格, 而之前的部分代码缩进都是2个空格, 似乎是不同公司的人写的呢.

`inst`的类型定义在`uvm_coreservice_t`的上方:

```
`ifndef UVM_CORESERVICE_TYPE
`define UVM_CORESERVICE_TYPE uvm_default_coreservice_t
`endif
```

也就是说`uvm_coreservice_t`的默认实现是`uvm_default_coreservice_t`, 也支持其他实现. 来看看"uvm_default_coreservice_t":

```
class uvm_default_coreservice_t extends uvm_coreservice_t;
	local uvm_factory factory;
	virtual function uvm_factory get_factory();
		if(factory==null) begin
			uvm_default_factory f;
			f=new;
			factory=f;
		end 
		return factory;
	endfunction
    ...
```

关于`factory`的部分是这样的, 其他的组件是一模一样的形式. 你看看这里又是这个套路, 再来回顾一下之前`factory`的使用方式:

```
    uvm_coreservice_t cs = uvm_coreservice_t::get();
    uvm_factory factory=cs.get_factory();
```

可以发现每个地方这么用的`factory`其实是同一个对象, `cs`也是这样. 这个就是设计模式中的单例模式. 了解了这个情况之后, 我们就知道`registry`中调用`register`方法和`create_object_by_type`方法是同一个`factory`对象了. 不过需要注意的是, 这里返回的其实是`uvm_default_factory`的对象, 猜测一下, `uvm_factory`应该会是个抽象类.

## 向uvm_factory进发

`uvm_factory`的位置在`src/base/uvm_factory.svh`, 来看看定义:

```
virtual class uvm_factory;
  static function uvm_factory get();
	  	uvm_coreservice_t s;
	  	s = uvm_coreservice_t::get();
	  	return s.get_factory();
  endfunction	
  pure virtual function void register (uvm_object_wrapper obj);
  pure virtual function
      void set_inst_override_by_type (uvm_object_wrapper original_type,
                                      uvm_object_wrapper override_type,
                                      string full_inst_path);
  pure virtual function
      void set_inst_override_by_name (string original_type_name,
                                      string override_type_name,
                                      string full_inst_path);
    ...
  pure virtual function
      uvm_object    create_object_by_type    (uvm_object_wrapper requested_type,  
                                              string parent_inst_path="",
                                              string name=""); 
  pure virtual function
      uvm_object    create_object_by_name    (string requested_type_name,  
                                              string parent_inst_path="",
                                              string name=""); 
    ...
  pure  virtual function void print (int all_types=1);
endclass 
```

果然`uvm_factory`又是一个抽象类, 除了`get`之外都是纯虚函数原型, 这里的`get`返回的也是`uvm_default_factory`类型的单例对象. 根据之前的线索, 我们关注的是`register`和`create_object_by_type`这两个方法, 不过这里还有`create_object_by_name`这个方法, 而且从名字来看这个方法更加的Factory, 还是放到一起看吧. 先从`register`开始:

```
function void uvm_default_factory::register (uvm_object_wrapper obj);
  if (obj == null) begin
    uvm_report_fatal ("NULLWR", "Attempting to register a null object with the factory", UVM_NONE);
  end
  if (obj.get_type_name() != "" && obj.get_type_name() != "<unknown>") begin
    if (m_type_names.exists(obj.get_type_name()))
      uvm_report_warning("TPRGED", {"Type name '",obj.get_type_name(),
        "' already registered with factory. No string-based lookup ",
        "support for multiple types with the same type name."}, UVM_NONE);
    else 
      m_type_names[obj.get_type_name()] = obj;
  end
    ...
endfunction
```

函数的参数类型是`uvm_object_wrapper`, 这个类型又出现了, 从这里看这个应该是`uvm_object_registry`的父类. 最前面是空指针处理和注册处理. `get_type_name`大家在`uvm_object_registry`里见过了, 是跟类名绑定的字符串, 不过这个方法在祖类`uvm_object`里也是有的, 默认值就是`<unknown>`这个字符串. `m_type_names`想必就是保存对象的哈希表了, 但是有点奇怪, 为什么是`uvm_object_wrapper`的对象直接存进去了, 而不是我们定义的类型. 先来看看这个哈希表:

```
  protected uvm_object_wrapper   m_type_names[string];
```

直到这里, 我们终于知道UVM的Factory的保存的类型是`uvm_object_wrapper`了, 但是这个类是怎么回事, 它是从`uvm_object`继承过来的吗? 我觉得需要紧急插播一下这个类定义.

### 紧急插播uvm_object_wrapper

好在这个类的定义跟Factory是在同个文件的, 来看看吧:

```
virtual class uvm_object_wrapper;
  virtual function uvm_object create_object (string name="");
    return null;
  endfunction
  virtual function uvm_component create_component (string name, 
                                                   uvm_component parent); 
    return null;
  endfunction
  pure virtual function string get_type_name();
endclass
```

竟然是个抽象类, 而且没有从任何的类继承, 是个悬空类! 从注释信息看, 这个类还是`uvm_component_registry`的父类. `uvm_object_registry`实现了`create_object`, `uvm_component_registry`实现了`create_component`, 然后两个类都需要实现`get_type_name`这个纯虚方法. 父类对象句柄用来存放子类对象, 这就是所谓的多态啦.

所以回到`m_type_names`这个哈希表上, 因为`uvm_object_wrapper`是个抽象类, 所以这个哈希表里存放的是`uvm_object_registry`或者是`uvm_component_registry`对象. 这两类对象不是我们注册的类本身, 但是这两个类型对象都是泛型类(参数化类), 可以根据绑定的类型参数来生成我们定义的类的对象, 也就是说, **UVM的Factory注册的不是我们定义的类, 而是我们定义的类的object或者component生成器对象.**

这点是很重要的, 了解了这点, 然后再转换一下思维, 我相信UVM的Factory的一切真相就都要水落石出了.

我们再来跳回上一级调用栈, 继续往下看Factory的`register`方法:

```
function void uvm_default_factory::register (uvm_object_wrapper obj);
    ...
  if (m_types.exists(obj)) begin
    if (obj.get_type_name() != "" && obj.get_type_name() != "<unknown>")
      uvm_report_warning("TPRGED", {"Object type '",obj.get_type_name(),
                         "' already registered with factory. "}, UVM_NONE);
  end
  else begin
    m_types[obj] = 1;
    ... // override dealing
  end
endfunction
```

这里的`m_types`是一个bit哈希表:

```
  protected bit                  m_types[uvm_object_wrapper];
```

为注册的对象置1, 后面的就是一些重载信息处理了, 重载的内容我们在下篇再关注吧~

接下来我们来看看最后一个方法`create_object_by_type`:

```
function uvm_object uvm_default_factory::create_object_by_type (uvm_object_wrapper requested_type,  
                                                        string parent_inst_path="",  
                                                        string name=""); 
  string full_inst_path;
  if (parent_inst_path == "")
    full_inst_path = name;
  else if (name != "")
    full_inst_path = {parent_inst_path,".",name};
  else
    full_inst_path = parent_inst_path;
  m_override_info.delete();
  requested_type = find_override_by_type(requested_type, full_inst_path);
  return requested_type.create_object(name);
endfunction
```

联系到我们之前的调用参数:

```
    obj = factory.create_object_by_type(get(),"","user_object");
```

这里的`requested_type`类型是`uvm_object_registry#(user_object, "user_object")`的, 里面的`full_inst_path`就是"user_object". 中间的`requested_type`还经历了`find_override_by_type`这个方法的调用. 这个方法是重载相关的方法, 非常的复杂, 而且是个递归函数. 不过它的功能可以描述为"在Fatory中寻找`requested_type`的重载信息, 如果找到了就返回重载的registry对象, 否则就返回原输入对象"(是不是很水的感觉, 没办法, 重载的内容我打算放到下篇). 然后, 函数的最后返回的是`requested_type`的方法`create_object`创建的对象, 也就是调用`uvm_object_registry #(user_object, "user_object")`对象中的`create_object`方法(父类句柄调用子类的方法, 这就是所谓的多态之动态绑定)创建的对象, 再往上翻翻吧, 创建的对象就是`user_object`类型的.

### 番外篇之create_object_by_name

讲到这里, 我们的Factory创建`uvm_object`对象探索其实已经可以算结束了. 不过呢, 之前出现了`create_object_by_name`这个从名字来看最符合Factory精神的函数, 不来了解一下说不过去的嘛对不对:

```
function uvm_object uvm_default_factory::create_object_by_name (string requested_type_name,  
                                                        string parent_inst_path="",  
                                                        string name=""); 
  uvm_object_wrapper wrapper;
  string inst_path;
  if (parent_inst_path == "")
    inst_path = name;
  else if (name != "")
    inst_path = {parent_inst_path,".",name};
  else
    inst_path = parent_inst_path;
  m_override_info.delete();
  wrapper = find_override_by_name(requested_type_name, inst_path);
    ...
endfunction
```

第一个参数变成了类型字符串, 然后前面的部分跟`create_object_by_type`的处理其实是一样的, 也加入了重载处理, 再来看看最后面:

```
function uvm_object uvm_default_factory::create_object_by_name (string requested_type_name,  
                                                        string parent_inst_path="",  
                                                        string name=""); 
    ...
  if (wrapper==null) begin
    if(!m_type_names.exists(requested_type_name)) begin
      uvm_report_warning("BDTYP",{"Cannot create an object of type '",
      requested_type_name,"' because it is not registered with the factory."}, UVM_NONE);
      return null;
    end
    wrapper = m_type_names[requested_type_name];
  end
  return wrapper.create_object(name);
endfunction
```

从哈希表里取出对象生成器`uvm_object_registry`然后用来创建对象, 最想看到的东西总算看到了, 真好.

# 总结

到这里, 我们的UVM源码探索之Factory上篇, 也就是`uvm_object`的Factory实现算是正式完结了, 来总结梳理一下我们探索了哪些内容吧:

1. 在`user_object`类定义中使用`uvm_object_util`宏为用户添加了一系列成员和方法, 其中最重要的是创建了`uvm_object_registry#(user_object, "uvm_object")`这个绑定了类名和类名字符串的registry类型;
1. `uvm_factory`作为UVM的基础设施, 使用了单例模式来保证其对象唯一性;
1. `uvm_factory`中实际注册的不是`user_object`对象, 而是用于生成`user_object`对象的`uvm_object_registry#(user_object, "uvm_object")`对象;
1. 用Factory创建对象的调用栈为: `user_object::type_id::create("name")`-->`uvm_object_registry#(user_object, "uvm_object")::create("name")`-->`factory.create_object_by_type()`, 最后返回了对象.

总的说来, UVM的Factory是通过多态和泛型来实现的, 跟普通的Factory比起来要复杂很多. 平心而论, 如果不需要支持重载的话, Factory实现的起来会简单很多. 我在最前面实现了一个"乞丐版"Factory, 大家可以考虑下加入泛型特性, 写出一个比较实用的Factory来, 不考虑重载的话应该也是挺简单的.

