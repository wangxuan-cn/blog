---
title: jdk8新特性
date: 2018-04-24 16:07:29
tags:
    - jdk8
categories:
    - jdk8
---
# 1. stream
## 1.1 stream分组合并
```
List<Buss> bussList = new ArrayList<>();
bussList.add(new Buss("a",10,0.3));
bussList.add(new Buss("b",3,0.8));
bussList.add(new Buss("c",5,2.0));
bussList.add(new Buss("b",30,3.2));
bussList.add(new Buss("c",20,0.1));

List<Buss> st = new ArrayList<>();
bussList.stream().collect(Collectors.groupingBy(Buss::getName)) //分组(Name can't be null)
.forEach((k,v) -> {
      Optional<Buss> sum = v.stream().reduce((v1,v2) -> { //合并
          v1.setCount(v1.getCount() + v2.getCount());
          v1.setValue(v1.getValue() + v2.getValue());
          return v1;
      });
      st.add(sum.orElse(new Buss("other",0,0.0)));
});

System.out.println(st);
}

class Buss {
    private String name;
    private int count;
    private double value;

    public Buss(String name, int count, double value) {
        this.name = name;
        this.count = count;
        this.value = value;
    }
    //省略get、set、toString方法
}
```

输出：  
`[Buss{name='a', count=10, value=0.3}, Buss{name='b', count=33, value=4.0}, Buss{name='c', count=25, value=2.1}]`  

如何根据两个字段排序，可以再model里面，写一个类似的方法即可，下例，分组的key 就是name+value
```
public String groupField() {
      return getName()+"-"+getValue();
}
```
