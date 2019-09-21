2、Lombock的使用

主要是基于标注来进行信息的封装和使用：

​    @Setter 注解在类或字段，注解在类时为所有字段生成setter方法，注解在字段上时只为该字段生成setter方法。

​    @Getter 使用方法同上，区别在于生成的是getter方法。

​    @ToString 注解在类，添加toString方法。

​    @EqualsAndHashCode 注解在类，生成hashCode和equals方法。

​    @NoArgsConstructor 注解在类，生成无参的构造方法。

​    @RequiredArgsConstructor 注解在类，为类中需要特殊处理的字段生成构造方法，比如final和被@NonNull注解的字段。

​    @AllArgsConstructor 注解在类，生成包含类中所有字段的构造方法。

​    @Data 注解在类，为类的所有字段注解@ToString、@EqualsAndHashCode、@Getter的便捷方法，同时为所有非final字段注解@Setter。