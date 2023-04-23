---
title: SpringBoot项目下实践设计模式之工厂模式
toc: true
date: 2023-04-23 16:50:50
tags:
	- 设计模式
	- 工程模式
	- SpringBoot
categories:
	- 代码人生
feature: 微信截图_20230423164322.png
---


## 示例结构（红框内的）

工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是复杂对象适合使用工厂模式，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。

<!-- more -->

{% asset_image 微信截图_20230423161023.png %}

## 接口
`com.springcloud.business.service.ICarService`

```java
package com.springcloud.business.service;

/**
 * @author xbronze
 */
public interface ICarService {
    public String run();
}
```

## 实现类
`com.springcloud.business.service.impl.BusServiceImpl`
```java
package com.springcloud.business.service.impl;

import com.springcloud.business.service.ICarService;
import org.springframework.stereotype.Service;

/**
 * @author: xbronze
 * @date: 2023-04-23 14:50
 * @description: TODO
 */
@Service
public class BusServiceImpl implements ICarService {
    @Override
    public String run() {
        return "大巴车一般要求时速控制在每小时80公里";
    }
}
```

`com.springcloud.business.service.impl.SuperCarServiceImpl`

```java
package com.springcloud.business.service.impl;

import com.springcloud.business.service.ICarService;
import org.springframework.stereotype.Service;

/**
 * @author: xbronze
 * @date: 2023-04-23 14:51
 * @description: TODO
 */
@Service
public class SuperCarServiceImpl implements ICarService {
    @Override
    public String run() {
        return "超跑的车速轻松能达到每小时200公里";
    }
}

```

## 一个工厂注册类
`com.springcloud.business.service.impl.CarServiceContent`

```java
package com.springcloud.business.service.impl;

import com.springcloud.business.service.ICarService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Map;

/**
 * @author: xbronze
 * @date: 2023-04-23 15:07
 * @description: TODO
 */
@Service
public class CarServiceContent {

    @Autowired
    private Map<String, ICarService> carServiceMap;

    public ICarService getCarService(String type) {
        if (carServiceMap.isEmpty()) {
            return null;
        }
        return this.carServiceMap.get(type);
    }
}

```
> 项目启动，系统会把`ICarService`的实现类都注入到`carServiceMap`，key值为实现类上@Service注解定义的value，如果没有显式的设置value，如示例上所示，那么默认value值为类名（首字母小写）。
## Controller类VehicleController

```java

/**
 * @author: xbronze
 * @date: 2023-04-23 14:54
 * @description: TODO
 */
@RestController
@RequestMapping("/vehicle")
public class VehicleController {

    @Autowired
    private IVehicleService vehicleService;

    @GetMapping("/{type}")
    public String vehicle(@PathVariable("type") String type){
        return vehicleService.choose(type);
    }
}

```

## 接口IVehicleService

```java
package com.springcloud.business.service;

/**
 * @author: xbronze
 * @date: 2023-04-23 14:54
 * @description: TODO
 */
public interface IVehicleService {

    String choose(String type);
}

```

## 实现类VehicleServiceImpl
```java
package com.springcloud.business.service.impl;

import com.springcloud.business.service.ICarService;
import com.springcloud.business.service.IVehicleService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * @author: xbronze
 * @date: 2023-04-23 15:17
 * @description: TODO
 */
@Service
public class VehicleServiceImpl implements IVehicleService {

    @Autowired
    private CarServiceContent carServiceContent;

    @Override
    public String choose(String type) {
        ICarService carService = carServiceContent.getCarService(type);
        return carService.run();
    }
}

```


## 测试


{% asset_image 微信截图_20230423164156.png %}

<br>

{% asset_image 微信截图_20230423164134.png %}

<br>

{% asset_image 微信截图_20230423164322.png %}

<br>

-----

完毕