//2.根据类型构造不同的资源对象（菜单，按钮，api）
    int type = perm.getType();
    switch (type) {
      case PermissionConstants.PERMISSION_MENU:
        PermissionMenu menu = BeanMapUtils.mapToBean(map,PermissionMenu.class);
        menu.setId(id);
        permissionMenuDao.save(menu);
        break;
      case PermissionConstants.PERMISSION_POINT:
        PermissionPoint point =
BeanMapUtils.mapToBean(map,PermissionPoint.class);
        point.setId(id);
        permissionPointDao.save(point);
        break;
      case PermissionConstants.PERMISSION_API:
        PermissionApi api = BeanMapUtils.mapToBean(map,PermissionApi.class);
        api.setId(id);
        permissionApiDao.save(api);
        break;
      default:
        throw new CommonException(ResultCode.FAIL);



@Getter
@Setter
@NoArgsConstructor
public class ProfileResult {
  private String mobile;
  private String username;

private String company;
  private Map roles;
  public ProfileResult(User user) {
    this.mobile = user.getMobile();
    this.username = user.getUsername();
    this.company = user.getCompanyName();
    //角色数据
    Set<String> menus = new HashSet<>();
    Set<String> points = new HashSet<>();
    Set<String> apis = new HashSet<>();
    Map rolesMap = new HashMap<>();
    for (Role role : user.getRoles()) {
      for (Permission perm : role.getPermissions()) {
        String code = perm.getCode();
        if(perm.getType() == 1) {
          menus.add(code);
       }else if(perm.getType() == 2) {
          points.add(code);
       }else {
          apis.add(code);
       }
     }
   }
    rolesMap.put("menus",menus);
    rolesMap.put("points",points);
    rolesMap.put("apis",points);
    this.roles = rolesMap;
 }
}



/**

获取个人信息
*/
@RequestMapping(value = "/profile", method = RequestMethod.POST)
public Result profile(HttpServletRequest request) throws Exception {
//临时使用
  String userId = "1";
  User user = userService.findById(userId);
  return new Result(ResultCode.SUCCESS,new ProfileResult(user));
}