# Spring-Security
**![](https://lh7-us.googleusercontent.com/4umSICBtOtnBp4UUGf2vsSf0Bdkl3sUDXK6FjLzxZD57KQF0vtEI_nlZHzT3WY-7I2rYsRQZy2aSvIFkoggHeum34uptEKVdIo4FJTqEKIN0FBfIYSr52FGXeWD_i6L0-lEPORwHZ_Rz8RmOncj9-pw)**

## 1. Spring Security Filter Chain 

> What is a filter? 
> A filter is an object used to intercept the HTTP requests and responses of your application.
   By using filter, we can perform two operations at two instances: 
> - Before sending the request to the controller
> - Before sending a response to the client.

>Chain of responsibility design pattern
>Chain of Responsibility is a behavioral design pattern that lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.

**![](https://lh7-us.googleusercontent.com/4umSICBtOtnBp4UUGf2vsSf0Bdkl3sUDXK6FjLzxZD57KQF0vtEI_nlZHzT3WY-7I2rYsRQZy2aSvIFkoggHeum34uptEKVdIo4FJTqEKIN0FBfIYSr52FGXeWD_i6L0-lEPORwHZ_Rz8RmOncj9-pw)**

## 1. Spring Security Filter Chain 

> What is a filter? 
> A filter is an object used to intercept the HTTP requests and responses of your application.
   By using filter, we can perform two operations at two instances: 
> - Before sending the request to the controller
> - Before sending a response to the client.

>Chain of responsibility design pattern
>Chain of Responsibility is a behavioral design pattern that lets you pass requests along a chain of handlers. Upon receiving a request, each handler decides either to process the request or to pass it to the next handler in the chain.

![Pasted image 20240426201213](https://github.com/DohaRamadan/Spring-Security/assets/77820526/aecca017-5e5c-4164-ab9f-6e43f808b1df)
![Pasted image 20240426201328](https://github.com/DohaRamadan/Spring-Security/assets/77820526/0a05e118-47c3-4944-93c7-274e12a1d9f2)

When we have an authentication request, it goes to Spring Security Filter Chain. It visits all the filters one by one and finally hit the authentication filter. Authentication filter then calls Authentication Manager. Authentication Manager's responsibility is going through all these providers and try to get at least one success to authenticate the user. Authentication Provider fetches user by communicating with User Details service and returns success state if the user exists with given credentials. Otherwise, Authentication Provider is supposed to throw an exception. By this, Spring Security knows this specific Authentication Provider failed to find the user.

> The default filter designed for the default form login is `UsernamePasswordAuthenticationFilter`

`UsernamePasswordAuthenticationFilter` will extract username and password from the authentication request and send them to Authentication Manager. Authentication Manager then send these username and password to `DaoAuthenticationProvider`, which is default provider. And this provider will go to `InMemoryUserDetailsManager`, which is default user details service, and check if user exists with given credentials

## 2. Spring Security Context 

The SecurityContextHolder is where Spring Security stores the details of who is authenticated. Spring Security does not care how the SecurityContextHolder is populated. 
If it contains a value, it is used as the currently authenticated user. 
The simplest way to indicate a user is authenticated is to set the SecurityContextHolder directly. 

```java
SecurityContext context = SecurityContextHolder.createEmptyContext(); Authentication authentication = new TestingAuthenticationToken("username", "password", "ROLE_USER"); 
context.setAuthentication(authentication); SecurityContextHolder.setContext(context);
```

> You should create a new `SecurityContext` instance instead of using `SecurityContextHolder.getContext().setAuthentication(authentication)` to avoid race conditions across multiple threads.

The `SecurityContext` contains an [Authentication](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication) object.

## 3. Authentication Manager 

[`AuthenticationManager`](https://docs.spring.io/spring-security/site/docs/6.2.4/api/org/springframework/security/authentication/AuthenticationManager.html) is the API that defines how Spring Security’s Filters perform [authentication](https://docs.spring.io/spring-security/reference/features/authentication/index.html#authentication). The [`Authentication`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authentication) that is returned is then set on the [SecurityContextHolder](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-securitycontextholder) by the controller (that is, by [Spring Security’s `Filters` instances](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters)) that invoked the `AuthenticationManager`.

## 3.1. Authentication Interface

The [`Authentication`](https://docs.spring.io/spring-security/site/docs/6.2.4/api/org/springframework/security/core/Authentication.html) interface serves two main purposes within Spring Security:
- An input to [`AuthenticationManager`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationmanager) to provide the credentials a user has provided to authenticate. When used in this scenario, `isAuthenticated()` returns `false`
``` java
authenticationManager.authenticate(  
        new UsernamePasswordAuthenticationToken(  
                input.getEmail(),  
                input.getPassword()  
        )
```
- Represent the currently authenticated user. You can obtain the current `Authentication` from the SecurityContext.
The `Authentication` contains:
- `principal`: Identifies the user. When authenticating with a username/password this is often an instance of [`UserDetails`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/user-details.html#servlet-authentication-userdetails).
- `credentials`: Often a password. In many cases, this is cleared after the user is authenticated, to ensure that it is not leaked.
- `authorities`: The [`GrantedAuthority`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-granted-authority) instances are high-level permissions the user is granted. Two examples are roles and scopes.

## 4. Authentication Provider 

You can inject multiple [`AuthenticationProvider`s](https://docs.spring.io/spring-security/site/docs/6.2.4/api/org/springframework/security/authentication/AuthenticationProvider.html) instances into [`ProviderManager`](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-providermanager). Each `AuthenticationProvider` performs a specific type of authentication. For example, [`DaoAuthenticationProvider`](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/dao-authentication-provider.html#servlet-authentication-daoauthenticationprovider) supports username/password-based authentication, while `JwtAuthenticationProvider` supports authenticating a JWT token.

## 5. UserDetailsService 

The UserDetailsService interface has a single method called `loadUserByUsername()`, which takes a username as a parameter and returns a fully populated UserDetails object. The UserDetails object represents the authenticated user in the Spring Security framework and contains details such as the user’s username, password, authorities (roles), and additional attributes.

In Spring Security, the UserDetailsService interface is a core component used for loading user-specific data. It is responsible for retrieving user information from a backend data source, such as a database or an external service, and returning an instance of the UserDetails interface.
For example: 
```java
@Service  
public class UserDetailsServiceImpl implements UserDetailsService {  
    @Autowired  
    UserRepository userRepository;  
  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
        return userRepository.findByEmail(username)  
                                         .orElseThrow(() -> new UsernameNotFoundException("User not found"));   
    }  
}
```




## References and Additional Resources

1. [Servlet Authentication Architecture :: Spring Security](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-authenticationprovider)
2. [Spring Security - UserDetailsService and UserDetails with Example - GeeksforGeeks](https://www.geeksforgeeks.org/spring-security-userdetailsservice-and-userdetails-with-example/)
3. [Chain of Responsibility (refactoring.guru)](https://refactoring.guru/design-patterns/chain-of-responsibility#:~:text=Chain%20of%20Responsibility%20is%20a,next%20handler%20in%20the%20chain.)
4. [Spring Security: Authentication Architecture Explained in Depth (youtube.com)](https://www.youtube.com/watch?v=ElY3rjtukig)
5. [Spring Security: Authentication Architecture Explained In Depth (backendstory.com)](https://backendstory.com/spring-security-authentication-architecture-explained-in-depth/)

