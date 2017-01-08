title: UVM Commandline Processor
Status: Public
Tags: UVM 
date: 2016-9-28 
---

UVM的CLP(Commandline Processor)出现在源码文档的倒数第二章, 而最后一章的标题是"GLOBALS", 也就是说UVM Command Processor是源码文档中的最后一个主题类.

这个类从字面上来看, 是处理仿真器的命令行参数的. 在看到这个类的时候, 我的直觉是这个类会不会是用SV的`$plusargs`实现的. 这次就来探索一下这个类吧.

# CLP的API

这些内容都是文档里面直接给出的, 先是基本对象接口:

API|Description
---|---
`get_inst`|Returns the singleton instance of the UVM command line processor.

表明这个类是个单例类, 这个好理解, 因为命令行的处理只需要做一次, 得到的参数保留一份就够了.

<!--more-->

接下来是基本功能接口:

API|Description
---|---
`get_args`|	This function returns a queue with all of the command line arguments that were used to start the simulation.
`get_plusargs`|	This function returns a queue with all of the plus arguments that were used to start the simulation.
`get_uvmargs`|	This function returns a queue with all of the uvm arguments that were used to start the simulation.
`get_arg_matches`|	This function loads a queue with all of the arguments that match the input expression and returns the number of items that matched. 

看描述的话, 这个类可以把所有的参数都提取出来呢, 而`$plusargs`是参数格式要求的, 也就是`+`开头的参数才能读取. 然后这个类提供了`get_arg_matches`这个字符串匹配接口, 看来不是`$plusargs`所能轻易做到的.


API|Description
---|---
`get_arg_value`|	This function finds the first argument which matches the match arg and returns the suffix of the argument.
`get_arg_values`|	This function finds all the arguments which matches the match arg and returns the suffix of the arguments in a list of values. 

这两个API是用来获取参数值的, 参数值的保存跟取值需要分开操作.

API|Description
---|---
`get_tool_name`|	Returns the simulation tool that is executing the simulation.
`get_tool_version`|	Returns the version of the simulation tool that is executing the simulation. 

这两个API返回仿真器的信息, 看来UVM的代码果然还是跟仿真器相关的.

然后是UVM里面所有的内置运行时参数:

API|Description
---|---
`+UVM_TESTNAME	`|`+UVM_TESTNAME=<class name> allows the user to specify which uvm_test (or uvm_component) should be created via the factory and cycled through the UVM phases.`
`+UVM_VERBOSITY	`|`+UVM_VERBOSITY=<verbosity> allows the user to specify the initial verbosity for all components.`
`+uvm_set_verbosity	`|`+uvm_set_verbosity=<comp>,<id>,<verbosity>,<phase> and +uvm_set_verbosity=<comp>,<id>,<verbosity>,time,<time> allow the users to manipulate the verbosity of specific components at specific phases (and times during the “run” phases) of the simulation.`
`+uvm_set_action	`|`+uvm_set_action=<comp>,<id>,<severity>,<action> provides the equivalent of various uvm_report_object’s set_report_*_action APIs.`
`+uvm_set_severity	`|`+uvm_set_severity=<comp>,<id>,<current severity>,<new severity> provides the equivalent of the various uvm_report_object’s set_report_*_severity_override APIs.`
`+UVM_TIMEOUT	`|`+UVM_TIMEOUT=<timeout>,<overridable> allows users to change the global timeout of the UVM framework.`
`+UVM_MAX_QUIT_COUNT	`|`+UVM_MAX_QUIT_COUNT=<count>,<overridable> allows users to change max quit count for the report server.`
`+UVM_PHASE_TRACE	`|`+UVM_PHASE_TRACE turns on tracing of phase executions.`
`+UVM_OBJECTION_TRACE	`|`+UVM_OBJECTION_TRACE turns on tracing of objection activity.`
`+UVM_RESOURCE_DB_TRACE	`|`+UVM_RESOURCE_DB_TRACE turns on tracing of resource DB access.`
`+UVM_CONFIG_DB_TRACE	`|`+UVM_CONFIG_DB_TRACE turns on tracing of configuration DB access.`
`+uvm_set_inst_override, +uvm_set_type_override	`|`+uvm_set_inst_override=<req_type>,<override_type>,<full_inst_path> and +uvm_set_type_override=<req_type>,<override_type>[,<replace>] work like the name based overrides in the factory--factory.set_inst_override_by_name() and factory.set_type_override_by_name().`
`+uvm_set_config_int, +uvm_set_config_string	`|`+uvm_set_config_int=<comp>,<field>,<value> and +uvm_set_config_string=<comp>,<field>,<value> work like their procedural counterparts: set_config_int() and set_config_string().`
`+uvm_set_default_sequence	`|`The +uvm_set_default_sequence=<seqr>,<phase>,<type> plusarg allows the user to define a default sequence from the command line, using the typename of that sequence. `

可以看出内置参数都是`+uvm`或`+UVM`开头的, 其实说起来用`$plusargs`也是可以的. 

# CLP的源码

接着我们来看看UVM Commandline Processor的源码罢, 看看这家伙是怎么实现的.

UVM Commandline Processor的源码文档位置在`src/base/uvm_cmdline_processor.svh`, 没多少行的.

```
class uvm_cmdline_processor extends uvm_report_object;

  static local uvm_cmdline_processor m_inst;

  // Group: Singleton 

  // Function: get_inst
  //
  // Returns the singleton instance of the UVM command line processor.

  static function uvm_cmdline_processor get_inst();
    if(m_inst == null) 
      m_inst = new("uvm_cmdline_proc");
    return m_inst;
  endfunction
    ...
```

开头是CLP的单例接口, 这个已经很眼熟了.


```
    ...
  protected string m_argv[$]; 
  protected string m_plus_argv[$];
  protected string m_uvm_argv[$];
  // Group: Basic Arguments
  
  // Function: get_args
  //
  // This function returns a queue with all of the command line
  // arguments that were used to start the simulation. Note that
  // element 0 of the array will always be the name of the 
  // executable which started the simulation.

  function void get_args (output string args[$]);
    args = m_argv;
  endfunction
    ...
  function void get_plusargs (output string args[$]);
    args = m_plus_argv;
  endfunction
    ...
  function void get_uvm_args (output string args[$]);
    args = m_uvm_argv;
  endfunction
    ...
```

继续往下看, 这里是CLP的基本接口, `get_*args`都是直接从成员`m_argv`, `m_plus_argv`和`m_uvm_argv`直接获取的, 这么简单的.


```
    ...
  // Function: get_arg_matches
  //
  // This function loads a queue with all of the arguments that
  // match the input expression and returns the number of items
  // that matched. If the input expression is bracketed
  // with //, then it is taken as an extended regular expression 
  // otherwise, it is taken as the beginning of an argument to match.
  // For example:
  //
  //| string myargs[$]
  //| initial begin
  //|    void'(uvm_cmdline_proc.get_arg_matches("+foo",myargs)); //matches +foo, +foobar
  //|                                                            //doesn't match +barfoo
  //|    void'(uvm_cmdline_proc.get_arg_matches("/foo/",myargs)); //matches +foo, +foobar,
  //|                                                             //foo.sv, barfoo, etc.
  //|    void'(uvm_cmdline_proc.get_arg_matches("/^foo.*\.sv",myargs)); //matches foo.sv
  //|                                                                   //and foo123.sv,
  //|                                                                   //not barfoo.sv.

  function int get_arg_matches (string match, ref string args[$]);

   `ifndef UVM_CMDLINE_NO_DPI
    chandle exp_h = null;
    int len = match.len();
    args.delete();
    ...
```

在到后面, `get_arg_matches`这个方法. 看注释, 竟然还支持正则表达式匹配, 好家伙! 然后再看实现, 竟然出现了DPI的宏, 看来CLP肯定是用C语言实现的了.
 
再往下翻是`get_arg_value`和`get_arg_values`方法, 其实没什么好看的, 接下来就直入主题, 来看看CLP的最重要的部分:

```
    ...
  // constructor

  function new(string name = "");
    string s;
    string sub;
    int doInit=1;
    super.new(name);
    do begin
      s = uvm_dpi_get_next_arg(doInit);
      doInit=0;
      if(s!="") begin
        m_argv.push_back(s);
        if(s[0] == "+") begin
          m_plus_argv.push_back(s);
        end 
        if(s.len() >= 4 && (s[0]=="-" || s[0]=="+")) begin
          sub = s.substr(1,3);
          sub = sub.toupper();
          if(sub == "UVM")
            m_uvm_argv.push_back(s);
        end 
      end
    end while(s!=""); 
    ...
```

在构造函数里, CLP使用`uvm_dpi_get_next_arg`来获取参数, 每个返回的参数字符串存入`m_argv`队列, 然后以`+`开头的参数存入`m_plus_argv`队列, `+uvm`或'-uvm`(大小无关)开头的参数存入`m_uvm_argv`队列. (题外话, 构造函数不是local或者protected修饰的, 有点不单例啊)所以, 现在的焦点到了`uvm_dpi_get_next_arg`这里.

# CLP与DPI

grep一下, `uvm_dpi_get_next_arg`的声明在`src/dpi/uvm_svcmd_dpi.svh`文件里:

```
`ifndef UVM_CMDLINE_NO_DPI
import "DPI-C" function string uvm_dpi_get_next_arg_c (int init);
import "DPI-C" function string uvm_dpi_get_tool_name_c ();
import "DPI-C" function string uvm_dpi_get_tool_version_c ();

function string uvm_dpi_get_next_arg(int init=0);
  return uvm_dpi_get_next_arg_c(init);
endfunction

function string uvm_dpi_get_tool_name();
  return uvm_dpi_get_tool_name_c();
endfunction

function string uvm_dpi_get_tool_version();
  return uvm_dpi_get_tool_version_c();
endfunction

import "DPI-C" function chandle uvm_dpi_regcomp(string regex);
import "DPI-C" function int uvm_dpi_regexec(chandle preg, string str);
import "DPI-C" function void uvm_dpi_regfree(chandle preg);

`else
function string uvm_dpi_get_next_arg(int init=0);
  return "";
endfunction

function string uvm_dpi_get_tool_name();
  return "?";
endfunction
```

注意到`uvm_dpi_get_next_arg`有两处定义, 无`UVM_CMDLINE_NO_DPI`宏分支下`uvm_dpi_get_next_arg`调用的是C语言函数`uvm_dpi_get_next_arg_c()`; 在另外的分支, `uvm_dpi_get_next_arg`直接返回了空字符串, 要是没有DPI, 这些参数都不能用的意思吗?

`uvm_dpi_get_next_arg_c`的定义在`uvm_svcmd_dpi.c`文件里, 直接看代码吧:

```
#include "uvm_dpi.h"
#include <assert.h>
    ...
const char *uvm_dpi_get_next_arg_c (int init) {
	s_vpi_vlog_info info;
	static int idx=0;

	if(init==1)
	{
		// free if necessary
		free(argv_stack);
		argc_total=0;

		vpi_get_vlog_info(&info);
		walk_level(0,info.argc,info.argv,0);

		argv_stack = (char**) malloc (sizeof(char*)*argc_total);
		argv_ptr=argv_stack;
		walk_level(0,info.argc,info.argv,1);	
		idx=0;	
		argv_ptr=argv_stack;
	}

	if(idx++>=argc_total)
	  return NULL;
	
	return *argv_ptr++;
}

```

一大片的`argc`跟`argv`, 跟`main`函数里的运行参数是一致的. 还有`vpi`前缀的函数, 看着麻烦, 而且查到不到定义, 看看`uvm_dpi.h`这个头文件

```
#ifndef UVM_DPI__H
#define UVM_DPI__H

#include <stdlib.h>
#include "vpi_user.h"
#include "veriuser.h"
#include "svdpi.h"
#include <malloc.h>
#include <string.h>
#include <stdio.h>
#include <regex.h>
#include <limits.h>
```

中间的三个文件源码包里面都没有, 看来要去工具目录下才找得到. 不过到现在, 这个函数的功能都已经清楚了, 也不用再继续找下去了.

# CLP与`$plusargs`

到这里, CLP的功能实现我感觉已经可以得出结论了, 那就是, CLP是用DPI提取了所有的命令行参数. 不过, 我还是想知道UVM里面到底用了`$plusargs`没有.

我试着grep一下, 结果还是找到了:

```
base/uvm_cmdline_processor.svh:164:  // returns the suffix of the argument. This is similar to the $value$plusargs
base/uvm_root.svh:452:  if ($value$plusargs("UVM_TESTNAME=%s", test_name)) begin
base/uvm_root.svh:993:  verb_count = $value$plusargs("UVM_VERBOSITY=%s",verb_string);
```

翻到`uvm_root.svh`文件里看一下:

```
`ifndef UVM_NO_DPI

  // Retrieve the test names provided on the command line.  Command line
  // overrides the argument.
  test_name_count = clp.get_arg_values("+UVM_TESTNAME=", test_names);

  // If at least one, use first in queue.
  if (test_name_count > 0) begin
    test_name = test_names[0];
    testname_plusarg = 1;
  end

  // If multiple, provided the warning giving the number, which one will be
  // used and the complete list.
  if (test_name_count > 1) begin
    ...
  end

`else

     // plusarg overrides argument
  if ($value$plusargs("UVM_TESTNAME=%s", test_name)) begin
    `uvm_info("NO_DPI_TSTNAME", "UVM_NO_DPI defined--getting UVM_TESTNAME directly, without DPI", UVM_NONE)
    testname_plusarg = 1;
  end

`endif
```

这是第一处匹配, 用`UVM_NO_DPI`宏隔开的, 也就是说, 不用DPI的时候, UVM用`$plusargs`来获取`UVM_TESTNAME`这个运行参数.

接下来是第二个匹配:

```
  `ifndef UVM_CMDLINE_NO_DPI
  // Retrieve the verbosities provided on the command line.
  verb_count = clp.get_arg_values("+UVM_VERBOSITY=", verb_settings);
  `else
  verb_count = $value$plusargs("UVM_VERBOSITY=%s",verb_string);
  if (verb_count)
    verb_settings.push_back(verb_string);
  `endif
```

一样的道理, 不用DPI的时候, 用`$plusargs`来获取`UVM_VERBOSITY`这个运行参数. 看起来`UVM_TESTNAME`跟`UVM_VERBOSITY`地位不一样啊.

# 总结

UVM的Commandline Processor的内容不多, 感觉了解的差不多了, 现在来做个总结吧:

1. CLP提供了运行参数的解析API;
1. CLP是单例类, 但是构造函数并没有声明为私有成员;
1. CLP默认使用DPI来实现运行参数的获取;
1. UVM中的`UVM_TESTNAME`参数和`UVM_VERBOSITY`参数比较重要, 在不使用DPI-C的时候也要用`$plusargs`从命令行获取;

另外, CLP跟`$plusargs`非常类似, 在来做个比较吧: 

1. 功能方面CLP更强大, 支持任意格式的参数. 
1. 使用方面来讲, `$plusargs`似乎要跟方便, 因为`$plusargs`可以直接带格式化字符串, 而CLP的字符串获取跟取值需要分两步, 非字符参数的格式化还需要另外的步骤.
1. 兼容性来讲, `$plusargs`是SV的原生语法, 而CLP需要依赖UVM和DPI-C, 可移植性也是`plusargs`更好.

这么一对比, 似乎`$plusargs`是用户更好的选择. 这个其实也好理解, 因此CLP是个单例类, 在是为UVM提供基础功能的, 只能算是个半公开API. 所以, 要做运行参数解析的话, 直接选`$plusargs`好了.

