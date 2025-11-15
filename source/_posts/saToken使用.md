---
title: saToken使用
date: 2025-09-15 19:01:33
tags:
categories: JavaWes
---

官网：https://sa-token.cc/v/v1.9.0/doc/index.html#/

# 1. 接口配置需要的权限名

```java
@SaCheckPermission("system:user:list")
@GetMapping("/list")
public TableDataInfo<SysUserVo> list(SysUserBo user, PageQuery pageQuery) {
    return userService.selectPageUserList(user, pageQuery);
}
```

# 2. 设置用户权限

用户登录接口，登录验证通过，设置用户的MenuPermission & RolePermission

```java
    private LoginUser buildLoginUser(SysUserVo user) {
        LoginUser loginUser = new LoginUser();
        loginUser.setTenantId(user.getTenantId());
        loginUser.setUserId(user.getUserId());
        loginUser.setDeptId(user.getDeptId());
        loginUser.setUsername(user.getUserName());
        loginUser.setNickName(user.getNickName());
        loginUser.setAvatar(user.getAvatar());
        loginUser.setUserType(user.getUserType());
        loginUser.setMenuPermission(permissionService.getMenuPermission(user.getUserId()));
        loginUser.setRolePermission(permissionService.getRolePermission(user.getUserId()));
        List<RoleDTO> roles = BeanUtil.copyToList(user.getRoles(), RoleDTO.class);
        loginUser.setRoles(roles);
        return loginUser;
    }
```

对应执行的SQL

```sql
   select distinct m.perms
        from sys_menu m
                 left join sys_role_menu rm on m.menu_id = rm.menu_id
                 left join sys_user_role sur on rm.role_id = sur.role_id
                 left join sys_role r on r.role_id = sur.role_id
        where m.status = '0'
          and r.status = '0'
          and sur.user_id = #{userId}
  select distinct r.role_id,
                  r.role_name,
                  r.role_key,
                  r.role_sort,
                  r.data_scope,
                  r.menu_check_strictly,
                  r.dept_check_strictly,
                  r.status,
                  r.del_flag,
                  r.create_time,
                  r.remark
  from sys_role r
           left join sys_user_role sur on sur.role_id = r.role_id
           left join sys_user u on u.user_id = sur.user_id
           left join sys_dept d on u.dept_id = d.dept_id
           WHERE r.del_flag = '0' and sur.user_id = #{userId}
```

# 3. 获取用户权限

实现StpInterface，获取当前账号权限码集合

```java
public class SaPermissionImpl implements StpInterface {

    /**
     * 获取菜单权限列表
     */
    @Override
    public List<String> getPermissionList(Object loginId, String loginType) {
        LoginUser loginUser = LoginHelper.getLoginUser();
        UserType userType = UserType.getUserType(loginUser.getUserType());
        if (userType == UserType.SYS_USER) {
            return new ArrayList<>(loginUser.getMenuPermission());
        } else if (userType == UserType.APP_USER) {
            // 其他端 自行根据业务编写
        }
        return new ArrayList<>();
    }

    /**
     * 获取角色权限列表
     */
    @Override
    public List<String> getRoleList(Object loginId, String loginType) {
        LoginUser loginUser = LoginHelper.getLoginUser();
        UserType userType = UserType.getUserType(loginUser.getUserType());
        if (userType == UserType.SYS_USER) {
            return new ArrayList<>(loginUser.getRolePermission());
        } else if (userType == UserType.APP_USER) {
            // 其他端 自行根据业务编写
        }
        return new ArrayList<>();
    }
}
```

## 4. 处理异常

注册全局异常处理satoken 的aop权限检查抛出的异常

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 权限码异常
     */
    @ExceptionHandler(NotPermissionException.class)
    public R<Void> handleNotPermissionException(NotPermissionException e, HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        log.error("请求地址'{}',权限码校验失败'{}'", requestURI, e.getMessage());
        return R.fail(HttpStatus.HTTP_FORBIDDEN, "没有访问权限，请联系管理员授权");
    }

    /**
     * 角色权限异常
     */
    @ExceptionHandler(NotRoleException.class)
    public R<Void> handleNotRoleException(NotRoleException e, HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        log.error("请求地址'{}',角色权限校验失败'{}'", requestURI, e.getMessage());
        return R.fail(HttpStatus.HTTP_FORBIDDEN, "没有访问权限，请联系管理员授权");
    }
}
```
