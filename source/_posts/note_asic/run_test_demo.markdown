title: 一个工厂小demo实现UVM的run_test
Status: Public
Tags: UVM 
date: 2017-3-28 
---

UVM的

<!--more-->

```
virtual class TestPrototype;
    pure virtual task test ();
endclass: TestPrototype

class TestFactory extends LogBase;
    static local TestPrototype tests[string];
    static function void register (TestPrototype obj, string test_name);
        assert(obj != null) 
        else begin
            $display("Can't register null object in factory!!");
            return;
        end
        if (tests.exists(test_name)) begin
            $display("Override obj registered with '%s'!!", test_name);
        end
        tests[test_name] = obj;
    endfunction: register
    static task run_test (string test_name);
        if (!tests.exists(test_name)) begin
            $display("Can't find %s in factory!!", test_name);
            return;
        end
        tests[test_name].test();
    endfunction: run_test
endclass

`define __register(T) \
static local T reg_obj = get(); \
static local function T get(); \
    if (reg_obj == null) begin \
        reg_obj = new();  \
        TestFactory::register(reg_obj, `"T`"); \
    end \
    return reg_obj; \
endfunction
```

