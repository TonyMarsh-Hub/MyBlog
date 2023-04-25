---
layout: post
title: "Spring中的转换器 To be continued"
date: 2023-04-25 22:29 +0800
category: [ 读书笔记, "spring实战第六版" ]
tag: [ Converter, 转换器 ]
---

顾名思义，转换器就是将A类型的对象转换为B类型的对象。
在Spring中，很多地方都会用到转换器，比如在SpringMVC中，将请求参数转换为Controller中的方法参数，将方法返回值转换为响应体,还有很多小地方。

发生最频繁的是Json与java对象的转换,前后端往往会统一使用JSON格式的数据进行交互，这时候就需要将Java对象转换为JSON对象，或者将JSON对象转换为Java对象。
不过先不谈论这个复杂的转换器，而是从简单的转换器开始。

## 简单的转换器使用

在Spring中，转换器是一个接口，它有两个泛型参数，分别是源类型和目标类型。

在书中第二节，遇到了这样的场景是 html发送的表单中，包含了Taco的name和一个Ingredient的ID列表。处理器中将表单数据绑定为Taco对象进行处理。
虽然name属性可以和Taco对象中的name属性进行绑定，但是Ingredient的ID列表无法和Taco对象中的Ingredient列表进行绑定。
因为传递的Id只是一个String，而Ingredient是一个复杂的对象，
所以在书本的案例中，使用了一个自定义的转换器，来将请求携带的ID转换为java对象。

```java
@Component
public class IngredientByIdConverter implements Converter<String, Ingredient> {

  private Map<String, Ingredient> ingredientMap = new HashMap<>();

  public IngredientByIdConverter() {
    ingredientMap.put("FLTO", 
        new Ingredient("FLTO", "Flour Tortilla", Type.WRAP));
    ingredientMap.put("COTO", 
        new Ingredient("COTO", "Corn Tortilla", Type.WRAP));
    ingredientMap.put("GRBF", 
        new Ingredient("GRBF", "Ground Beef", Type.PROTEIN));
    ingredientMap.put("CARN", 
        new Ingredient("CARN", "Carnitas", Type.PROTEIN));
    ingredientMap.put("TMTO", 
        new Ingredient("TMTO", "Diced Tomatoes", Type.VEGGIES));
    ingredientMap.put("LETC", 
        new Ingredient("LETC", "Lettuce", Type.VEGGIES));
    ingredientMap.put("CHED", 
        new Ingredient("CHED", "Cheddar", Type.CHEESE));
    ingredientMap.put("JACK", 
        new Ingredient("JACK", "Monterrey Jack", Type.CHEESE));
    ingredientMap.put("SLSA", 
        new Ingredient("SLSA", "Salsa", Type.SAUCE));
    ingredientMap.put("SRCR", 
        new Ingredient("SRCR", "Sour Cream", Type.SAUCE));
  }

  @Override
  public Ingredient convert(String id) {
    return ingredientMap.get(id);
  }

}
```

convert操作的本质将就是根据ID从数据库中查询出对应的对象，然后返回。（此时还有没到数据库部分，所以手动维护了一个map）

---

此时一个很重要的问题是，这个转换器是如何被Spring管理的呢？
我们在Spring中注册了一个转换器，但是没有在需要使用的地方显式的使用这个转换器，Spring是如何知道我们需要使用这个转换器的呢？

其实Spring在启动的时候，会扫描所有的转换器，然后将其注册到一个转换器注册表中，当需要使用转换器的时候，就会从转换器注册表中获取对应的转换器。
获取的根据就是判定的类型是否匹配，如果匹配就会使用这个转换器。

具体来说，所有实现了Converter接口的组件，会在启动时统一注册到ConversionService中，
所有的convert操作统一仅由ConversionService来完成，其先会使用默认的转换器，然后再使用自定义的转换器。如果仍然无法转换，就会抛出异常。
如果存在多个转换器，就会使用优先级高的那一个，优先级的设定可以在注册转换器组件的使用，添加注解@Order来指定。值越小，优先级越高。


