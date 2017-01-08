title: UVM源码探索之Factory(下篇)
Status: Public
Tags: UVM 
date: 2016-8-28 
---

[TOC]

到下篇的时候, 我们来探索一下UVM的Factory的另外一个主要功能, 那就是重载. 说起来, 重载功能跟Factory这个设计模式已经没有关系了. 而且, 这里的"重载"跟常见的重载的意义也不一样, 这里的重载准确的描述的话是类"替换".

在UVM里面使用重载的大致流程如下:

1. 对需要重载的类使用`uvm_*_util`进行类型注册, 原始类和重载类都需要注册;
1. 设置重载, 设置原始类"替换"为重载类;
1. 使用Factory的`create`创建原始类对象, 返回的是重载类对象.

步骤1我们在上篇已经讲过了, 步骤2跟3的在上篇其实也有相关的代码展示, 不过我们略过了. 现在到了下篇, 我们的主要任务就是把上面流程中的2跟3的源码实现给搞清楚.

<!--more-->

# 重载的设置

在上篇, `uvm_*_registry`的类定义里面有这样的代码:


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

`set_type_override`和`set_inst_override`就是设置重载的两个函数了, 这两个函数都是基于类型的重载函数, 第一个参数是需要重载的类型. 后面的函数, 从内容上来看, 是为了支持UVM的component树的特定节点的重载的, 这个就要跟component扯上关系了, 所以先不细究了(UVM的component代码很多的, 光是`uvm_component.svh`这个文件里面, 只有一个类定义, 就有3000多行, 而且内容很杂, 我也很无奈~). 

所以, 我们就把重点放在`set_type_override`上面来吧. 这个函数里面, 调用的其实是Factory里定义的`set_type_override_by_type()`这个方法, 第二个参数就是`uvm_object_registry`里面`set_type_override`方法的第一个参数, 就是重载的类型, 而第一个参数是`uvm_object_registry`里面`get`方法的返回值, 也就是本类型的单例对象. 第三个参数`replace`的默认值是1, 这个参数是什么用的, 这里还体现不出来. `set_type_override`这个函数要做的就是就是通过Factory的`set_type_override_by_type`方法把本类型替换为指定的重载类型, 具体怎么重载的呢就要看Factory的代码了.

这个部分代码有点长, 我们要慢慢看:


```
function void uvm_default_factory::set_type_override_by_type (uvm_object_wrapper original_type,
                                                      uvm_object_wrapper override_type,
                                                      bit replace=1);
  bit replaced;
  // check that old and new are not the same
  if (original_type == override_type) begin
    if (original_type.get_type_name() == "" || original_type.get_type_name() == "<unknown>")
      uvm_report_warning("TYPDUP", {"Original and override type ",
                                    "arguments are identical"}, UVM_NONE);
    else
      uvm_report_warning("TYPDUP", {"Original and override type ",
                                    "arguments are identical: ",
                                    original_type.get_type_name()}, UVM_NONE);
  end
    ...
```

第一部分, 是输入检查, 如果类型相同的话输出一个告警.


```
function void uvm_default_factory::set_type_override_by_type (uvm_object_wrapper original_type,
                                                      uvm_object_wrapper override_type,
                                                      bit replace=1);
  bit replaced;
    ...
  // register the types if not already done so, for the benefit of string-based lookup
  if (!m_types.exists(original_type))
    register(original_type); 
  if (!m_types.exists(override_type))
    register(override_type); 
    ...
endfunction
```

第二部分, 是类型注册检查. 还记得`m_types[$]`这个bit类型的哈希表吗, 上篇中说过, 这个表记录了类型是否已注册. 在此之前, 我们知道类型的注册发生在第一次使用Factory的`create`的时候. 在这里我们要思考一下了, 重载的设置, 其实是应该发生在`create`之前的是不是, 所以这里也是需要保证类型注册的.


```
function void uvm_default_factory::set_type_override_by_type (uvm_object_wrapper original_type,
                                                      uvm_object_wrapper override_type,
                                                      bit replace=1);
  bit replaced;
    ...
  // check for existing type override
  foreach (m_type_overrides[index]) begin
    if (m_type_overrides[index].orig_type == original_type ||
        (m_type_overrides[index].orig_type_name != "<unknown>" &&
         m_type_overrides[index].orig_type_name != "" &&
         m_type_overrides[index].orig_type_name == original_type.get_type_name())) begin
      string msg;
      msg = {"Original object type '",original_type.get_type_name(),
             "' already registered to produce '",
             m_type_overrides[index].ovrd_type_name,"'"};
      if (!replace) begin
        msg = {msg, ".  Set 'replace' argument to replace the existing entry."};
        uvm_report_info("TPREGD", msg, UVM_MEDIUM);
        return;
      end
      msg = {msg, ".  Replacing with override to produce type '",
                  override_type.get_type_name(),"'."};
      uvm_report_info("TPREGR", msg, UVM_MEDIUM);
      replaced = 1;
      m_type_overrides[index].orig_type = original_type; 
      m_type_overrides[index].orig_type_name = original_type.get_type_name(); 
      m_type_overrides[index].ovrd_type = override_type; 
      m_type_overrides[index].ovrd_type_name = override_type.get_type_name(); 
    end
  end
    ...
endfunction
```

接着往下看, 这里是一个表的搜索操作. `m_type_overrides`显然是一个重载的记录表了, 这段代码字面上看是要先查找记录表中是不是已经有输入参数`original_type`的重载记录了, 如果有的话需要把记录表中的`ovrd_type`的信息赋值为输入参数`override_type`的相关信息. 中间的`if`表达式需要对第三个输入参数`replace`做检查, 不为0的时候就直接返回, 不进行后面的`override_type`的记录表赋值操作了. 这样看来, UVM的Factory是支持重载的改写操作的, `replace`参数设为1的话就可以实现先把A类型重载为B类型, 之后再把A类型重新重载为C类型.

这个部分其实挺复杂的, 而且我们对`m_type_overrides`这个数据结构其实一无所知. 现在先假定我们是第一次进入这个方法, 所以这个`m_type_overrides`表是空的, 这样, 我们就可以先看看这个方法的最后一部分是什么:

```
function void uvm_default_factory::set_type_override_by_type (uvm_object_wrapper original_type,
                                                      uvm_object_wrapper override_type,
                                                      bit replace=1);
  bit replaced;
    ...
  // make a new entry
  if (!replaced) begin
    uvm_factory_override override;
    override = new(.orig_type(original_type),
                   .orig_type_name(original_type.get_type_name()),
                   .full_inst_path("*"),
                   .ovrd_type(override_type));
    m_type_overrides.push_back(override);
  end
endfunction
```

这里是一个创建重载对象的操作, 使用`uvm_factory_override`这个类型的数据来保存重载信息: 原始类型和重载类型. 然后把这个记录插入到`m_type_overrides`这个表里. `repalced`这个变量在上面的重载改写的时候会置1, 有重复记录的表项不设置改写的时候会直接返回, 所以, 插入的表项都是之前没有重载记录的表项.

# 重载的支持数据结构

到这里来讲, 我们的重载设置这块任务已经算完成的差不多了. 重载设置函数内容上看还是挺简单的, 不过出现了之前没有见过的数据结构. 在重载应用的时候, 这些数据结构也是要发挥作用的, 这里我们还是先了解一下这些数据结构为好.

`uvm_default_factory`类里面的主要数据成员如下:

```
  protected bit                  m_types[uvm_object_wrapper];
  protected bit                  m_lookup_strs[string];
  protected uvm_object_wrapper   m_type_names[string];

  protected uvm_factory_override m_type_overrides[$];

  protected uvm_factory_queue_class m_inst_override_queues[uvm_object_wrapper];
  protected uvm_factory_queue_class m_inst_override_name_queues[string];
  protected uvm_factory_override    m_wildcard_inst_overrides[$];

  local uvm_factory_override     m_override_info[$];
```

里面有老相识`m_types`和`m_type_names`, 还有之前见过的列表`m_type_overrides`和他的类型`uvm_factory_override`, 这里可以看到还有新的类型`uvm_factory_queue_class`和这个类型的3个成员`m_inst_override_queues`, `m_inst_override_name_queues`和`m_wildcard_inst_overrides`, 这三个成员分别是key为`uvm_object_wrapper`的联合数组, key为字符串的联合数组和队列.

`uvm_factory_queue_class`的定义在`uvm_factory.svh`文件的开头部分, 可以先瞄一眼.

```
class uvm_factory_queue_class;
  uvm_factory_override queue[$];
endclass
```

这个`uvm_factory_queue_class`这个数据竟然还是`uvm_factory_override`的队列, 这三个`m_inst_*`成员还真是复杂的很啊.

这里没有注释, 不过从变量的命名和空行分隔排列来看, `uvm_factory_queue_class`的3个成员是用来管理component树节点重载的数据. 之前说过先不管基于component节点的重载真是先见之明啊.

然后我们来看看`uvm_factory_override`的类定义:

```
class uvm_factory_override;
  string full_inst_path;
  string orig_type_name;
  string ovrd_type_name;
  bit selected;
  int unsigned used;
  uvm_object_wrapper orig_type;
  uvm_object_wrapper ovrd_type;
  function new (string full_inst_path="",
                string orig_type_name="",
                uvm_object_wrapper orig_type=null,
                uvm_object_wrapper ovrd_type);
    if (ovrd_type == null) begin
      uvm_report_fatal ("NULLWR", "Attempting to register a null override object with the factory", UVM_NONE);
    end
    this.full_inst_path= full_inst_path;
    this.orig_type_name = orig_type == null ? orig_type_name : orig_type.get_type_name();
    this.orig_type      = orig_type;
    this.ovrd_type_name = ovrd_type.get_type_name();
    this.ovrd_type      = ovrd_type;
  endfunction
endclass
```

就是一些基本的数据, 两个类型数据`orig_type`和`ovrd_type`和这两个类型对应的类型名字符串`orig_type_name`和`ovrd_type_name`. 虽然我们是不管component树节点重载了, 但是`uvm_factory_override`还是要给它提供支持的, `full_inst_path`看起来就是支持这个的. 然后`selected`和`used`是之前没有出现过的, 这个是怎么用的要看后面了.

这个类的数据成员都是公用的, 也只提供了`new`这一个方法, `new`方法也只是用来给成员赋值用的, 只是做了一些小小的空指针检查. 这个类看起来是一副要做大事的样子啊...

# 重载的实现

好了, 现在我们开始研究重载的最后一步吧. 在上篇中我们讲到:

> 用Factory创建对象的调用栈为: `user_object::type_id::create("name")`-->`uvm_object_registry#(user_object, "uvm_object")::create("name")`-->`factory.create_object_by_type()`, 最后返回了对象.

重载的实现就隐藏在`factory.create_object_by_type()`里了, 我们再来恢复一下现场看看:

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

这里的`requested_type = find_override_by_type(requested_type, full_inst_path);`这一句就是重载函数了, 字面上看就是在重载表里对`requested_type`进行查找, 然后返回重载记录表的结果, 然后用返回的结果来创建对象. 按照之前的参数传递, 这里传入的`full_inst_path`是传入的字符串名`name`, 而传入的`requested_type`是`uvm_object_registry#(user_object, "user_object")`的单例对象. 剩下的秘密, 尽在`find_override_by_type`这个方法了.

这个方法的代码:

```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);

  uvm_object_wrapper override;
  uvm_factory_override lindex;
  
  uvm_factory_queue_class qc;
  qc = m_inst_override_queues.exists(requested_type) ?
       m_inst_override_queues[requested_type] : null;

  foreach (m_override_info[index]) begin
    if ( //index != m_override_info.size()-1 &&
       m_override_info[index].orig_type == requested_type) begin
      uvm_report_error("OVRDLOOP", "Recursive loop detected while finding override.", UVM_NONE);
      if (!m_debug_pass)
        debug_create_by_type (requested_type, full_inst_path);

	  m_override_info[index].used++;
      return requested_type;
    end
  end

  // inst override; return first match; takes precedence over type overrides
  if (full_inst_path != "" && qc != null)
    for (int index = 0; index < qc.queue.size(); ++index) begin
      if ((qc.queue[index].orig_type == requested_type ||
           (qc.queue[index].orig_type_name != "<unknown>" &&
            qc.queue[index].orig_type_name != "" &&
            qc.queue[index].orig_type_name == requested_type.get_type_name())) &&
          uvm_is_match(qc.queue[index].full_inst_path, full_inst_path)) begin
        m_override_info.push_back(qc.queue[index]);
        if (m_debug_pass) begin
          if (override == null) begin
            override = qc.queue[index].ovrd_type;
            qc.queue[index].selected = 1;
            lindex=qc.queue[index];
          end
        end
        else begin
	      qc.queue[index].used++;	        
          if (qc.queue[index].ovrd_type == requested_type)
            return requested_type;
          else
            return find_override_by_type(qc.queue[index].ovrd_type,full_inst_path);
        end
      end
    end

  // type override - exact match
  foreach (m_type_overrides[index]) begin
    if (m_type_overrides[index].orig_type == requested_type ||
        (m_type_overrides[index].orig_type_name != "<unknown>" &&
         m_type_overrides[index].orig_type_name != "" &&
         requested_type != null &&
         m_type_overrides[index].orig_type_name == requested_type.get_type_name())) begin
      m_override_info.push_back(m_type_overrides[index]);
      if (m_debug_pass) begin
        if (override == null) begin
          override = m_type_overrides[index].ovrd_type;
          m_type_overrides[index].selected = 1;
          lindex=m_type_overrides[index];
        end
      end
      else begin
	    m_type_overrides[index].used++;	      
        if (m_type_overrides[index].ovrd_type == requested_type)
          return requested_type;
        else
          return find_override_by_type(m_type_overrides[index].ovrd_type,full_inst_path);
      end
    end
  end

  // type override with wildcard match
  //foreach (m_type_overrides[index])
  //  if (uvm_is_match(index,requested_type.get_type_name())) begin
  //    m_override_info.push_back(m_inst_overrides[index]);
  //    return find_override_by_type(m_type_overrides[index],full_inst_path);
  //  end

  if (m_debug_pass && override != null) begin
    lindex.used++;	  
    if (override == requested_type) begin
      return requested_type;
    end	
    else
      return find_override_by_type(override,full_inst_path);
  end	
  
  return requested_type;

endfunction
```

这个代码量也是相当的恐怖, 我们还是来分段看看:

```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);
  uvm_object_wrapper override;
  uvm_factory_override lindex;
  uvm_factory_queue_class qc;

  qc = m_inst_override_queues.exists(requested_type) ?
       m_inst_override_queues[requested_type] : null;
  foreach (m_override_info[index]) begin
    if ( //index != m_override_info.size()-1 &&
       m_override_info[index].orig_type == requested_type) begin
      uvm_report_error("OVRDLOOP", "Recursive loop detected while finding override.", UVM_NONE);
      if (!m_debug_pass)
        debug_create_by_type (requested_type, full_inst_path);

	  m_override_info[index].used++;
      return requested_type;
    end
  end
    ...
endfunction
```

最前面的部分, 就出现了`uvm_factory_queue_class`类型的`qc`和之前只提到过的`m_override_info`的操作. 这真是尴尬, 现在我们只好先假定我们用的很克制, `m_inst_override_queues`里面什么都没有, 然后`qc`就是个空指针啦. 上篇出现的`m_override_info`的时候, 都是只有`delete`操作, 然后这里也就假定这个列表是空吧. 蛤蛤, 先硬着头皮继续往下看.

```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);
  uvm_object_wrapper override;
  uvm_factory_override lindex;
  uvm_factory_queue_class qc;
    ...
  // inst override; return first match; takes precedence over type overrides
  if (full_inst_path != "" && qc != null)
    for (int index = 0; index < qc.queue.size(); ++index) begin
      if ((qc.queue[index].orig_type == requested_type ||
           (qc.queue[index].orig_type_name != "<unknown>" &&
            qc.queue[index].orig_type_name != "" &&
            qc.queue[index].orig_type_name == requested_type.get_type_name())) &&
          uvm_is_match(qc.queue[index].full_inst_path, full_inst_path)) begin
        m_override_info.push_back(qc.queue[index]);
        if (m_debug_pass) begin
          if (override == null) begin
            override = qc.queue[index].ovrd_type;
            qc.queue[index].selected = 1;
            lindex=qc.queue[index];
          end
        end
        else begin
	      qc.queue[index].used++;	        
          if (qc.queue[index].ovrd_type == requested_type)
            return requested_type;
          else
            return find_override_by_type(qc.queue[index].ovrd_type,full_inst_path);
        end
      end
    end
    ...
endfunction
```

第二部分看起来也是来者不善啊, 不过好在这一段全是基于`qc`非空的行为, 所以, 还是可以再跳一段...

```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);
  uvm_object_wrapper override;
  uvm_factory_override lindex;
  uvm_factory_queue_class qc;
    ...
  // type override - exact match
  foreach (m_type_overrides[index]) begin
    if (m_type_overrides[index].orig_type == requested_type ||
        (m_type_overrides[index].orig_type_name != "<unknown>" &&
         m_type_overrides[index].orig_type_name != "" &&
         requested_type != null &&
         m_type_overrides[index].orig_type_name == requested_type.get_type_name())) begin
      m_override_info.push_back(m_type_overrides[index]);
      if (m_debug_pass) begin
        if (override == null) begin
          override = m_type_overrides[index].ovrd_type;
          m_type_overrides[index].selected = 1;
          lindex=m_type_overrides[index];
        end
      end
      else begin
	    m_type_overrides[index].used++;	      
        if (m_type_overrides[index].ovrd_type == requested_type)
          return requested_type;
        else
          return find_override_by_type(m_type_overrides[index].ovrd_type,full_inst_path);
      end
    end
  end
    ...
endfunction
```

终于好像到了我们最关注的部分了. 这里是对`m_type_overrides`的搜索操作, 成功条件是记录表中一个成员的`orig_type`为`requested_type`, 或者记录表中成员的`orig_type_name`跟`requested_type`的`type_name`相等. 咦, 会出现这种情况吗?

这里面的第一步就是把这条记录插入到`m_override_info`这个表里面了, 这~~~看来还是不能不管`m_override_info`这个东西啊. 不过还是先把这部分看完先, 下面出现了`m_debug_pass`, 这个变量也先假定为0好了, 然后看后面的`else`分支. 这里先是把记录表里面的`used`变量加1, 然后判断`requested_type`是不是跟记录表里的`ovrd_type`一致, 一致的话就返回`requested_type`.

我们的输入就是`requested_type`啊, 这里的意思似乎是说设置的重载类型就是原始类型? 会这么傻吗? 不过再看到后面的分支, 这还是个递归函数呢, 牛逼. 现在假定我们是一个正常的调用, 也就是说`requested_type`跟`ovrd_type`是不一样的, 这里就会使用`m_type_overrides`表里的`ovrd_type`作为`requested_type`参数进行递归, 再到开头的那里, 就不能跳过`m_override_info`的处理啦:

```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);
  uvm_object_wrapper override;
  uvm_factory_override lindex;
    ...
  foreach (m_override_info[index]) begin
    if ( //index != m_override_info.size()-1 &&
       m_override_info[index].orig_type == requested_type) begin
      uvm_report_error("OVRDLOOP", "Recursive loop detected while finding override.", UVM_NONE);
      if (!m_debug_pass)
        debug_create_by_type (requested_type, full_inst_path);

	  m_override_info[index].used++;
      return requested_type;
    end
  end
    ...
```

这里要的搜索操作要检查的是记录表`m_override_info`的成员的`orig_type`是不是`requested_type`, 这里的`requested_type`就是之前`ovrd_type`, 这个检查为真的话说明真的是有重载类和原始类相同的重载设置操作啦, 然后这里就先输出一个Error. 后面的`m_debug_pass`值是0, 所有要先做一个`debug_create_by_type`的操作, 不过我过代码, 这里没有对requested_type做改动. 之后就是对`used`计数加1, 然后返回啦. 

因为正常调用的话这里是不会执行到了(蛤蛤, 尴尬), 所以还是要继续往下执行: 

```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);
  uvm_object_wrapper override;
  uvm_factory_override lindex;
    ...
  // type override - exact match
  foreach (m_type_overrides[index]) begin
    if (m_type_overrides[index].orig_type == requested_type ||
        (m_type_overrides[index].orig_type_name != "<unknown>" &&
         m_type_overrides[index].orig_type_name != "" &&
         requested_type != null &&
         m_type_overrides[index].orig_type_name == requested_type.get_type_name())) begin
        ...
      else begin
	    m_type_overrides[index].used++;	      
        if (m_type_overrides[index].ovrd_type == requested_type)
          return requested_type;
        else
          return find_override_by_type(m_type_overrides[index].ovrd_type,full_inst_path);
      end
    end
  end
    ...
endfunction
```

还是在这里, 这里的`requested_type`其实是之前的记录表中的`ovrd_type`, 所以跟所有的`orig_type`是不相等的, 而且`orig_type_name`跟`requested_type`的`type_name`也是不一样的, 所以这里就要跳过了...


```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);
  uvm_object_wrapper override;
  uvm_factory_override lindex;
    ...
  // type override with wildcard match
  //foreach (m_type_overrides[index])
  //  if (uvm_is_match(index,requested_type.get_type_name())) begin
  //    m_override_info.push_back(m_inst_overrides[index]);
  //    return find_override_by_type(m_type_overrides[index],full_inst_path);
  //  end
  if (m_debug_pass && override != null) begin
    lindex.used++;	  
    if (override == requested_type) begin
      return requested_type;
    end	
    else
      return find_override_by_type(override,full_inst_path);
  end	
  return requested_type;
endfunction
```

然后就到了最后了, 中间的一段注释不管了, 然后`m_debug_pass`的分支也不管了, 直接就返回了`requested_type`, 也就是之前传入的`ovrd_type`. 看起来一次重载查找操作至少要调用两次这个方法呢...

## 插播一点点, 关于链式重载

回到第二次调用, 这里的`requested_type`其实是之前的记录表中的`ovrd_type`, 我们之前是假定这个`requested_type`是不会跟任何的`orig_type`相等的, 然后就往下执行了. 正常使用的时候会不会出现`requested_type`跟`orig_type`相等呢? 

仔细考虑一下其实是会出现的. 比如过, 我们先设置A重载为B, 再设置B重载为C, 然后我们调用Factory的`create`创建A的句柄, 这个时候, 在调用到`find_override_by_type`的时候, 就会出现`orig_type`为B类型的重载记录, 这个时候需要继续执行, 这一段代码:

```
function uvm_object_wrapper uvm_default_factory::find_override_by_type(uvm_object_wrapper requested_type,
                                                               string full_inst_path);
  uvm_object_wrapper override;
  uvm_factory_override lindex;
    ...
  // type override - exact match
  foreach (m_type_overrides[index]) begin
    if (m_type_overrides[index].orig_type == requested_type ||
        (m_type_overrides[index].orig_type_name != "<unknown>" &&
         m_type_overrides[index].orig_type_name != "" &&
         requested_type != null &&
         m_type_overrides[index].orig_type_name == requested_type.get_type_name())) begin
        ...
      else begin
	    m_type_overrides[index].used++;	      
        if (m_type_overrides[index].ovrd_type == requested_type)
          return requested_type;
        else
          return find_override_by_type(m_type_overrides[index].ovrd_type,full_inst_path);
      end
    end
  end
    ...
endfunction
```

也就是说, 需要传入B类型的`requested_type`再次调用`find_override_by_type`. 

总共经过3次调用之后, Factory实际上生成的是C类型的对象, 这样就实现了类型的链式重载.


# 总结

到这里为止, Factory的重载功能也介绍完了, 我们来总结一下吧:

1. UVM里面使用Factory重载的流程为: 使用`uvm_*_util`进行注册, 设置重载, 使用`create`创建对象;
1. 为重载提供支持的数据结构是`uvm_factory_override`类型;
1. UVM的Factory支持多级链式重载.

然后, 我们的Factory源码探索也就结束了. Factory源码的内容不止这么多, 比如我们在上篇只讲了`uvm_object`的Factory实现没讲`uvm_component`的, 在下篇只讲了`type_override`没有将基于component的`inst_override`. 另外, 从接口上调用不到的很多代码也没有讲到. 不过, 这两篇读下来, 然后再看剩下的代码相信就没什么难度了.



