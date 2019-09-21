# java8从list集合中取出某一属性的值的集合

List<Order> list = new ArrayList<User>();

        Order o1 = new Order("1","MCS-2019-1123");
        list.add(o1 );
        Order o2= new Order("2","MCS-2019-1124");
        list.add(o2);
        Order o3= new Order("3","MCS-2019-1125");
        list.add(o3);
        List<String> orderNoList=list.stream().map(User::getOrderNo).collect(Collectors.toList());
        System.out.println("输出单号集合："+orderNoList);
        List<String> idList=list.stream().map(Order::getId()).collect(Collectors.toList());
        System.out.println(idList)
```
/*对一 二级list的名字进行去重处理,得到一二级全部的菜单*/
List<SysMenu> unique = sysMenuFirst.stream().collect(
        Collectors.collectingAndThen(
                Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(SysMenu::getName))), ArrayList::new)
);
```
```
 List<SysMenu> sysMenus = new ArrayList<>();
/*这个里面都是一级目录的菜单,将图片路径进行替换*/
sysMenus.forEach(pl -> {
    pl.setMenuIcon(path + pl.getMenuIcon());
});

/*得到的是一级目录的几个消息,进行排序处理*/
//        sysMenus.sort((o1, o2) -> o1.getMenuSort().compareTo(o2.getMenuSort()));

遍历操作
//            unique.forEach(sysMenu ->{
//                System.out.println(sysMenu.getMenuName());});
//
//                unique1.forEach(MenuBtn ->{
//                    System.out.println(MenuBtn.getBtnName());});
//            return tree;
```