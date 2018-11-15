#### JsonInclude

```
@JsonSerialize(include = JsonSerialize.Inclusion.NON_NULL)
```

+ Include.NON_NULL  属性为null不序列化
+ Include.NON_DEFAULT  属性为默认值不序列化
+ Include.NON_EMPTY  属性为空或者为NULL都不序列化
+ Include.AWAYS  属性都序列化  默认





+ NotNull  集合不能是null
+ NotEmpty  集合不能是null，且size>0
+ NotBlank  String不能是null，且trim length>0

使用@Valid修饰要校验的对象，否则不起作用