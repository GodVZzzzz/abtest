### Feature
- 灰度
    - 将多个灰度规则组合成一条灰度策略，支持无限扩展。参见`com.springframework.toolkit.abtest.AbTestFactory.AbTestFacade`类

- 分流
    - 支持取模、哈希取模、随机取模等分流设置，支持自定义分流方法。参见`com.springframework.toolkit.abtest.div.DivMethods`类


### Quick Start
#### Step1: 配置`灰度`和`分流`规则

- 策略1
    - 当且仅当 渠道`source IN ["2","21"] `和 城市`city IN ["1", "5"]`时，命中灰度规则；分流 50%

- 策略2
    - 当且仅当 渠道`source IN ["1"] `和 城市`city IN ["3"]`时，命中灰度规则；分流 100%

```
[
  {
    "layer": {
      "id": "layer1",
      "data": "something1"
    },
    "grayRules": [
      {
        "name": "source",
        "enabled": true,
        "include": [
          "2",
          "21"
        ],
        "exclude": [],
        "global": false
      },
      {
        "name": "city",
        "enabled": true,
        "include": [
          "1",
          "5"
        ],
        "exclude": [],
        "global": false
      }
    ],
    "divRule": {
      "percent": 50
    }
  },
  {
    "layer": {
      "id": "layer2",
      "data": "something2"
    },
    "grayRules": [
      {
        "name": "source",
        "enabled": true,
        "include": [
          "1"
        ],
        "exclude": [],
        "global": false
      },
      {
        "name": "city",
        "enabled": true,
        "include": [
          "3"
        ],
        "exclude": [],
        "global": false
      }
    ],
    "divRule": {
      "percent": 100
    }
  }
]
```

#### Step2: 读取配置、校验规则


```
// 当前请求上下文
String source = "2";
String cityId = "3";
String uid = "abc123"; 

  
// 1、加载配置文件: 可以从配置中心或存储读取
String conf = "[{\"layer\":{\"id\":\"layer1\",\"data\":\"something1\"},\"grayRules\":[{\"name\":\"source\"," +
                "\"enabled\":true,\"include\":[\"2\",\"21\"],\"exclude\":[],\"global\":false},{\"name\":\"city\"," +
                "\"enabled\":true,\"include\":[\"1\",\"5\"],\"exclude\":[],\"global\":false}]," +
                "\"divRule\":{\"percent\":50}},{\"layer\":{\"id\":\"layer2\",\"data\":\"something2\"}," +
                "\"grayRules\":[{\"name\":\"source\",\"enabled\":true,\"include\":[\"1\"],\"exclude\":[]," +
                "\"global\":false},{\"name\":\"city\",\"enabled\":true,\"include\":[\"3\"],\"exclude\":[]," +
                "\"global\":false}],\"divRule\":{\"percent\":100}}]";
                
  
AbTestFactory.AbTestFacade facade = AbTestFactory.build(conf);  

  
// 2、链式校验是否命中灰度规则和分流规则
DivResult result = facade.hitGray(GrayPoints.source(source))
                         .peek(System.out::println)
                         .hitGray(GrayPoints.city(cityId))
                         .peek(System.out::println)
                         .hitDiv(DivMethods.hashMod(uid)); // 按hash(uid) % 100 分流

if (result.isHit()) {
    // do something
}                              
```
