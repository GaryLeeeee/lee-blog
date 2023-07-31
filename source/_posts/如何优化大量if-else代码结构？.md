---
title: 如何优化大量if-else代码结构？
date: 2023-07-31 00:18:38
tags: [代码优化,策略模式,工厂模式]
categories: [代码优化]
---


## 一、提前return
**适用场景**：if-else代码结构包含return语句
**优点**：优化代码阅读体验，如果if处理逻辑复杂不用拉到底才知道else是return逻辑
**例子代码**：
```
优化前：
if(condition) {
    // do sth
} else {
    return;
}

优化后：
if(!condition) {
    return;
}
// do sth

```

## 二、使用三目运算符
**适用场景**：if-else条件都是为某个变量赋值
**优点**：使代码更加简洁
**例子代码**：
```
优化前：
int price;
if(condition) {
    price = 10;  
} else {
    price = 20;
}

优化后：
int price = condition ? 10 : 20;
```

## 三、使用枚举
**适用场景**：condition的判断维度一致，且if-else逻辑都是为某个变量赋值
**优点**：方便扩展
**例子代码**：
```
优化前：
int status;
String remark;
if(status == 0) {
    remark = "正常";
} else if (status == 1) {
    remark = "已隐藏";
} else if (status == 2) {
    remark = "已删除";
}

优化后：
@AllArgsConstructor
@Getter
public enum StatusEnum {
    COMMON(0,"正常"),
    HIDDEN(1,"已隐藏"),
    DELETED(2,"已删除");
    
    private int status;
    private String remark;
    
    public static String getRemarkByStatus(int status) {
        for(StatusEnum statusEnum : StatusEnum.values()) {
            if(status == statusEnum.getStatus()) {
                return statusEnum;
            }
        }
        
        return null;
    }
}

int status;
String remark = StatusEnum.getRemarkByStatus(status);
```

## 四、合并条件表达式
**适用场景**：多个if-else对应一样的逻辑处理
**例子代码**：
```
优化前：
if(age < 0) {
    return -1;
}
if(id < 0) {
    return -1;
}
if(StringUtils.isBlank(name)) {
    return -1;
}

优化后：
if(age < 0 || id < 0 || StringUtils.isBlank(name)) {
    return -1;
}
```

## 五、策略模式+工厂模式
**使用场景**：每个if-else逻辑处理比较复杂，放在一个方法里会不好维护
**优点**：基于接口编程，可读性好，方便维护扩展
**例子代码**：
```
优化前：
int type;
if(type == 1) {
    // do sth1
} else if (type == 2) {
    // do sth2
}

优化后：
@Service
public class TestFactory implements InitializingBean {
    @Autowired
    private List<TestStrategy> strategies;
    private Map<Integer, TestStrategy> strategyMap = new HashMap<>();
    
    @Override
    public void afterPropertiesSet() throws Exception {
        for(TestStrategy testStrategy : strategies) {
            strategyMap.put(testStrategy.getType(), testStrategy); 
        }
    }
    
    public TestStrategy getTestStrategy(int type) {
        return strategyMap.get(type);
    }
}

public interface TestStrategy {
    // 获取类型
    int getType();
    // 执行
    void execute();
}

@Serivce
public class TestStrategyImpl1 implements TestStrategy {
    @Override
    public int getType() {
        return 1; 
    }
    
    @Override
    public void execute() {
        //do sth
    }
}

@Serivce
public class TestStrategyImpl2 implements TestStrategy {
    @Override
    public int getType() {
        return 2; 
    }
    
    @Override
    public void execute() {
        //do sth
    }
}

int type;
TestFactory.getTestStrategy(type).execute();
```