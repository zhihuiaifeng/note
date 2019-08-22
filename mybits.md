@Param("参数名"),表示给参数命名,名称就是括号中的内容，@Param("**cid**")Integer cid 

这个是SQL语句SELECT * FROM carinfo car LEFT JOIN dotinfo dot ON car.dotInfo = dot.dot_id WHERE car.cid = #{**cid**}

```
SELECT COUNT(*) FROM carinfo car WHERE car.dotInfo = #{dotId}
这个是前面mapper层提供的方法
int deleteCarInfo(@Param("cids")int[] cids);
delete from carinfo where cid in
<foreach collection="cids" item="item" index="index" open="(" separator="," close=")">
	#{item}
</foreach>
```

```
# 上传文件大小限制    
  servlet:
    multipart:
      max-file-size: 500Mb
      max-request-size: 500Mb
 pagehelper:
    params: count=countSql
    # 指定分页插件使用哪种方言
    helper-dialect: mysql
    # 分页合理化参数 pageNum<=0时会查询第一页 pageNum>pages(超过总数时) 会查询最后一页
    reasonable: 'true'
    support-methods-arguments: 'true'
    
jackson:
  # 全局配置   返回前端值为 null的属性不会返回
    default-property-inclusion: NON_NULL
    date-format: yyyy-MM-dd HH:mm:ss
    locale: GMT+8
```

