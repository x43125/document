# Shiro步骤

#### 1 引入依赖

```xml
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.7.1</version>
        </dependency>
```

#### 2 基础配置

**application.yml**

```yaml
server:
  port: 8080
  servlet:
    context-path: /shiro
```

#### 3 shiroconfig.java

##### 3.1 拦截器 ShiroFilterFactoryBean

```java
    /**
     * Filter工厂，设置对应的过滤条件和跳转条件
     * 负责拦截所有请求
     * @param securityManager
     * @return
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, String> map = new HashMap<>();
        // 登出
        map.put("/logout", "logout");
        // "authc"：认证，"/**"对所有的进行认证
        map.put("/**", "authc");
        // 默认登陆页面
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 首页
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 错误页面，认证不通过跳转
        shiroFilterFactoryBean.setUnauthorizedUrl("/error");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
        return shiroFilterFactoryBean;
    }
```

##### 3.2 安全管理器 DefaultWebSecurityManager

```java
    /**
     * 安全管理器
     * @return
     */
    @Bean
    public DefaultWebSecurityManager defaultWebSecurityManager(Realm realm) {
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(realm);
        return defaultWebSecurityManager;
    }
```

##### 3.3 Realm

```java
    /**
     * 将自己的验证方式加入容器
     * @return shiroRealm
     */
    @Bean
    public Realm getRealm() {
        CustomerRealm customerRealm = new CustomerRealm();
        return customerRealm;
    }
```

#### 4 Realm

