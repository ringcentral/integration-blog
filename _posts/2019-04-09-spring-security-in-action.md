---
layout: post
title:  "Spring Security In Action"
date:   2019-04-09 23:31:07 +0800
tags: [JAVA, Spring-Security]
categories: [blog]
author: Jakob He

---

## **What is Spring-Security?**
Spring-Security is a highly customizable authentication and access-control framework for java applications. Especially for spring based applications. This is widely used in Java developing field. 

## **Why do we need to use Spring-Security?**
Spring-Security is highly integrated with the most popular framework Spring-Boot. It supports both Authentication and Authorization, which are also the most popular ways to deal with the security issues between server and client end. Like all Spring based projects, the real power of Spring-Security is found in how easily it can be extended to meet customer requirements.

## **Spring-Security in Action**
In this example we will go through a very basic Spring-Security application. There are four important classes to be introduced (HttpSecurity, WebSecurityConfigurer, UserDetailsService, AuthenticationManager). And the final application will cover following features. 
1. username and password verification
2. role control
3. token distribution - using JWT
4. token verification
5. password crypto

> ![spring-security-scope.jpg](/integration-blog/assets/2019-04-09-spring-security-in-action/spring-security-scope.jpg)

Through the process of implementation, we will cover some fundamental principles of Spring-Security.

### I. Include Spring-Security Dependencies 
```sh
compile "org.springframework.boot:spring-boot-starter-security"
```

### II. Token Distrubtion
The first thing to do is to define a way to grant tokens. There are several grant types to accomplish this, including "password", "authorization_code", "implicit", and "client_credentials". In our example, we used "password" as the grant type.

```java
@Override
public void configure(ClientDetailsServiceConfigurer configurer) throws Exception {
    configurer
            .inMemory()//jdbc()
            .withClient(clientId)
            .secret(passwordEncoder.encode(clientSecret))
            .authorizedGrantTypes("password")
            .scopes("read", "write")
            .resourceIds(resourceIds);
}
```

Normally, we don't use in-memory client id and secret. In production, we can configure to read all clients from database as shown below:
```java
configurer.jdbc()
```

After that, we need to configure token settings, including:
1. Token store
2. Token converter
3. Token manager 
4. Enhancer chain. 

In this example we are using JWT to manage our tokens.

### III. Token Verification
Now you are able to implement your own security policy. A popular way is to extend WebSecurityConfigurerAdapter and rewrite security control functions based on customer's requirements. Listed below are the important functions:
1. The passwordEncoder() function defines the way to encode and compare the passwords.
2. The configure(HttpSecurity http) function sets the resource strategy.
```java
@Override
public void configure(HttpSecurity http) throws Exception {
    //@formatter:off
    http
            .requestMatchers()
        .and()
            .authorizeRequests()
            .antMatchers("/public/**").permitAll() //no authorization required here
            .antMatchers("/demo/**").authenticated(); //need authorization
    //@formatter:on
}
```

3. The configure(ResourceServerSecurityConfigurer resources) function defines the security strategy. 
In this config() function we need to assign a token service to explain tokens. A typical token service is defining as below. 
```java
@Bean
@Primary
public DefaultTokenServices tokenServices() {
    DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
    defaultTokenServices.setTokenStore(tokenStore());
    defaultTokenServices.setSupportRefreshToken(true);
    return defaultTokenServices;
}
```

4. JWT provides convenient APIs that intergrated with Spring-Security closely. There is a converter to convert and decode tokens. All you need to do is to set a signing key and then set them in the config(AuthorizationServerEndpointsConfigurer endpoints) function.

```java
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey(signingKey);
    return converter;
}

@Bean
public TokenStore tokenStore() {
    return new JwtTokenStore(accessTokenConverter());
}
```

```java
@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    TokenEnhancerChain enhancerChain = new TokenEnhancerChain();
    enhancerChain.setTokenEnhancers(Collections.singletonList(accessTokenConverter));
    endpoints.tokenStore(tokenStore)
            .accessTokenConverter(accessTokenConverter)
            .tokenEnhancer(enhancerChain)
            .authenticationManager(authenticationManager);
}
```
Remember that the tokenService and authenticationManager must be the same one in token verication, so that the token can be decoded properly.

5. The authenticationManager() function defines the token verification logic. This class will be use to check the user authentication when a token is refreshed.

### IV. Optional: Custom Token Authorization Verification Logic
If the default token manager does not meet your requirements, which is happening all the time, you could use your own authenticationProvider by using the code below:
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.authenticationProvider(jwtAuthenticationProvider());
}
```

And the provider can be like this
```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    DecodedJWT jwt = ((JwtAuthenticationToken) authentication).getToken();
    //I want a token never expired.
    //if (jwt.getExpiresAt().before(Calendar.getInstance().getTime()))
        //throw new NonceExpiredException("Token expires");
    String clientId = jwt.getSubject();
    UserDetails user = userService.loadUserByUsername(clientId);
    if (user == null || user.getPassword() == null)
        throw new NonceExpiredException("Token expires");
    Algorithm algorithm = Algorithm.HMAC256(user.getPassword());
    JWTVerifier verifier = JWT.require(algorithm)
            .withSubject(clientId)
            .build();
    try {
        verifier.verify(jwt.getToken());
    } catch (Exception e) {
        throw new BadCredentialsException("JWT token verify fail", e);
    }
    return new JwtAuthenticationToken(user, jwt, user.getAuthorities());
}
```
### V. Optional: Custom Verification Chain
If the authenticate() function throws any exception, we may handle it or just record it in the database. To implement it we can just set tokenValidSuccessHandler and tokenValidFailureHandler. In the handler, you can rewrite onAuthenticationSuccess and onAuthenticationFailure with your own logic.
```java
...
    .and()
        .apply(new MyValidateConfigure<>())
            .tokenValidSuccessHandler(myVerifySuccessHandler())
            .tokenValidFailureHandler(myVerifyFailureHandler())
...
```
```java
public class JwtAuthenticationFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
                                        AuthenticationException exception) {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
    }
}
```

### VI. UserDetailService
UserDetailService is the core interface which loads user-specific data. We must realize loadUserByUsername() function to locate user and user's role.

### VII. REST APIs
Once all the authorization configurations have been finished, you can start to enjoy developing your project. To control your role and access you can simply add @PreAuthorize("hasAuthority('ADMIN_USER')") in your rest api declaration.
```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
@PreAuthorize("hasAuthority('ADMIN_USER')") //user with ADMIN_USER role have this access.
public ResponseEntity<List<User>> getUsers() {
    return new ResponseEntity<>(userService.findAllUsers(), HttpStatus.OK);
}
```

## **Demo**

### I. Apply for a token
Client end can either use postman or curl to get a token.

```sh
curl client-id:client-password@localhost:8080/oauth/token -d grant_type=password -d username=admin.admin -d password=Test!123
```

You will get:
```json
{"access_token":"eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOlsic3ByaW5nLXNlY3VyaXR5LWRlbW8tcmVzb3VyY2UtaWQiXSwidXNlcl9uYW1lIjoiYWRtaW4uYWRtaW4iLCJzY29wZSI6WyJyZWFkIiwid3JpdGUiXSwiZXhwIjoxNTU0ODQ0NTQxLCJhdXRob3JpdGllcyI6WyJTVEFOREFSRF9VU0VSIiwiQURNSU5fVVNFUiJdLCJqdGkiOiI4MTM3Y2Q4OS0wMWMyLTRkMTgtYjA4YS05MjNkOTcxYjNhYzQiLCJjbGllbnRfaWQiOiJjbGllbnQtaWQifQ.1t_4xVT8xaAtisHaNT_nMRBLKfpiI0SZQ2bbEGxu6mk","token_type":"bearer","expires_in":43199,"scope":"read write","jti":"8137cd89-01c2-4d18-b08a-923d971b3ac4"}
```

### II. Use the token in role control
Then use the token above to post a request which need authorition.
```sh
curl http://localhost:8080/demo/users -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOlsic3ByaW5nLXNlY3VyaXR5LWRlbW8tcmVzb3VyY2UtaWQiXSwidXNlcl9uYW1lIjoiYWRtaW4uYWRtaW4iLCJzY29wZSI6WyJyZWFkIiwid3JpdGUiXSwiZXhwIjoxNTU0ODQ0NTQxLCJhdXRob3JpdGllcyI6WyJTVEFOREFSRF9VU0VSIiwiQURNSU5fVVNFUiJdLCJqdGkiOiI4MTM3Y2Q4OS0wMWMyLTRkMTgtYjA4YS05MjNkOTcxYjNhYzQiLCJjbGllbnRfaWQiOiJjbGllbnQtaWQifQ.1t_4xVT8xaAtisHaNT_nMRBLKfpiI0SZQ2bbEGxu6mk"
```
It will return:
```json
[{"id":1,"username":"jakob.he","firstName":"Jakob","lastName":"He","roles":[{"id":1,"roleName":"STANDARD_USER","description":"Standard User"}]},{"id":2,"username":"admin.admin","firstName":"Admin","lastName":"Admin","roles":[{"id":1,"roleName":"STANDARD_USER","description":"Standard User"},{"id":2,"roleName":"ADMIN_USER","description":"Admin User"}]}]
```

### III. Verify the token by JWT
Put your token and signing key in [jwt.io](https://jwt.io), you will get the following result.

> ![jwt-verification.jpg](/integration-blog/assets/2019-04-09-spring-security-in-action/jwt-verification.jpg)

Let's quickly go over what we have done: we have introduced what is Spring-Security and why we need to use it. We have also implemented a complete Spring-Security application that included token management, token distribution, and REST APIs that are required for web authorization.
I hope this article is helpful.

#### **Links**
-   [Spring Security](https://spring.io/projects/spring-security)
-   [JWT](https://jwt.io/)
-   [Source Code](https://github.com/jakob-lewei/spring-security-demo)
