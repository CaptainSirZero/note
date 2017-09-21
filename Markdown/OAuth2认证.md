###OAuth2.0认证

####历史 

移动 App 的开发是基于现有的 Web 开发的基础上产生的，所以网络通信一般都是基于 HTTP 协议通信，而 HTTP 是一种无状态协议，所以针对 HTTP 协议状态保存一直都是永恒的话题。对于传统 Web 开发来讲，Cookie 和 Session 是最好的选择，在最早的时候，只有 Cookie 一种方案，但是这种方案存在缺陷，也就是容易被修改，所以结合 Cookie 就提出了 Session 这种服务端存储状态的方法。

但是移动 App 开发和传统 Web 开发是存在区别的，相比 Web 开发被局限于一个域中，移动 App 开发更加灵活，所以就需要更方便的机制用于授权认证。当然，并不是说移动 App 开发做不到 Session 这种方式，只需要在 HTTP 部分填充服务端返回的 Cookie字段，自然就能做到 Session。

HTTP 现有很多种认证机制，原生 HTTP 就有 `Basic` `Auth`、`Digest`，在 HTTP 基础上提出的认证也有很多，但是其中最知名最广泛的就是 OAuth2.0 认证。

####OAuth 历史
引用百度百科上的定义：

>OAuth 协议为用户资源的授权提供了一个安全的、开放而又简易的标准。与以往的授权方式不同之处是 OAuth 的授权不会使第三方触及到用户的帐号信息（如用户名与密码），即第三方无需使用用户的用户名与密码就可以申请获得该用户资源的授权，因此 OAuth 是安全的。OAuth 是 Open Authorization 的简写。
OAuth 协议实际上不是一个专门为了移动客户端提出的协议，它的本来意义是隔离授权和认证，方便第三方应用存取资源，但是实际上由于 OAuth 的便捷性，已经成为实质上的移动客户端认证方式。

OAuth 有 1.0 和 2.0 两个版本，实际内容差不多，2.0 版本是对 1.0 版本的扩充和修复，但是 2.0 版本不向下兼容 1.0 版本，所以目前使用的基本都是 2.0 版本。

OAuth 本身不存在一个标准的实现，后端开发者自己根据实际的需求和标准的规定实现。其步骤一般如下：

1. 客户端要求用户给予授权
2. 用户同意给予授权
3. 根据上一步获得的授权，向认证服务器请求令牌（token）
4. 认证服务器对授权进行认证，确认无误后发放令牌
5. 客户端使用令牌向资源服务器请求资源
6. 资源服务器使用令牌向认证服务器确认令牌的正确性，确认无误后提供资源


授权可以是不同的内容和方式，OAuth2.0 定义了四种授权方式

1. 授权码模式
2. 简化模式
3. 密码模式
4. 客户端 OAuth 实践

####授权码模式
授权码模式是目前功能最为完备使用最广泛的 OAuth 认证方式，目前市场上大部分的针对第三方应用的开放平台都是这种形式。阮一峰大神在自己的博客中已经有了很多讲述，但是估计太过于深，所以很多人都是看的云里雾里，这里就拿通常情况下的认证模式打比方。

对于客户端来说，最终的要求就是访问到资源服务器，并且从资源服务器获取用户的资源，但是资源服务器需要令牌（AccessToken），所以就需要向认证服务器获得令牌，由于授权模式不允许客户端代替用户提交用户名密码，所以就需要使用链接跳转到认证服务器的认证界面，但是，需要在 QueryString 附上 ClientID 和 RedirectUri，ClientID 用于标识客户端，从认证服务器注册后获得，RedirectUri 则是客户端后台服务器，然后用户在认证服务器提供的页面上填写用户名密码。注意，这里的页面是认证服务器提供的，也就是说，客户端无从插手用户名密码的输入，这最大限度的保障了用户名密码的安全，然后认证服务器检查用户名密码的正确性，如果正确，则跳转到指定的 RedirectUri，并且在 QueryString 上附带 AuthorizationCode，后台服务器使用 AuthorizationCode 向认证服务器获取 AccessToken，认证服务器则在 Response 域中返回 AccessToken，这样就可以访问资源服务器了。

####简化模式
简化模式和授权码模式基本一样，除了没有客户端的后台服务器作为中转，而是直接在浏览器 Uri 中请求令牌，这里就不多讲，直接百度就行。

####密码模式
密码模式是一种很少见又官方的的模式，它和客户端模式是复用的，它很少在实际开放平台中使用是因为用户需要向客户端提供用户名和密码，由客户端向认证服务器获得 AccessToken，然后使用 AccessToken 向资源服务器请求资源，这种情况实际上非常危险，因为客户端可以以明文的形式获得用户名和密码，所以在其他情况能使用的时候少用这种情况。

####客户端模式
这种情况是目前大部分中小型公司在开发客户端的时候使用最广泛的模式，严格来说，客户端模式不属于 OAuth2.0 规范需要解决的问题，而是一种从密码模式演化而来的模式。它直接传递给认证服务器 ClientID，然后认证服务器返回 AccessToken。但是由于大部分公司不需要向第三方应用开放接口，不需要建立开放平台，在一定程度上是和密码模式复用的。用户在客户端上注册，认证服务器实际上就是后台服务器，然后使用用户名密码返回 AccessToken。

###客户端的 OAuth 实践
在客户端开发中，最常见的就是密码模式，客户端获取用户名密码，向后台服务器请求 AccessToken，使用 AccessToken 向后台服务器其他 API 接口请求数据。对于大部分开发者来说，都是自己实现具体的业务逻辑处理，包括笔者，但是后来笔者发现了 AFNetworking 团队实际上已经自己提供了一套 OAuth2.0 认证机制模块 AFOAuth2Manager，足以适用于大部分情况了，所以这里直接剖析其源码，借鉴其精华。

####内部模块剖析
####文件结构
AFOAuth2Manager 实际上是依托于 AFNetworking 框架的一个扩展模块，实际上代码量非常小，就两个模块 `AFOAuth2Manager` 和 `AFOAuthCredential`，前者包含了所有的网络通信代码，后者则是存储 AccessToken 的模型类，文档介绍非常简单，就介绍了密码认证的流程

#####Authentication 身份验证

```
NSURL *baseURL = [NSURL URLWithString:@"http://example.com/"];
AFOAuth2Manager *OAuth2Manager =
            [[AFOAuth2Manager alloc] initWithBaseURL:baseURL
                                            clientID:kClientID
                                              secret:kClientSecret];

[OAuth2Manager authenticateUsingOAuthWithURLString:@"/oauth/token"
                                          username:@"username"
                                          password:@"password"
                                             scope:@"email"
                                           success:^(AFOAuthCredential *credential) {
                                               NSLog(@"Token: %@", credential.accessToken);
                                           }
                                           failure:^(NSError *error) {
                                               NSLog(@"Error: %@", error);
                                           }];
                                           
```
#####Authorizing Requests 授权请求

```
AFHTTPRequestOperationManager *manager =
    [[AFHTTPRequestOperationManager alloc] initWithBaseURL:baseURL];

[manager.requestSerializer setAuthorizationHeaderFieldWithCredential:credential];

[manager GET:@"/path/to/protected/resource"
  parameters:nil
     success:^(AFHTTPRequestOperation *operation, id responseObject) {
         NSLog(@"Success: %@", responseObject);
     }
     failure:^(AFHTTPRequestOperation *operation, NSError *error) {
         NSLog(@"Failure: %@", error);
     }];
```
#####Storing Credentials

```
[AFOAuthCredential storeCredential:credential
                    withIdentifier:serviceProviderIdentifier];
```

#####Retrieving Credentials
```
AFOAuthCredential *credential =
        [AFOAuthCredential retrieveCredentialWithIdentifier:serviceProviderIdentifier];
```
总共四个方法就概括了所有的 OAuth2.0 密码认证流程。而模块实际山也就只有4个文件，两个头文件两个实现文件。
我们先来看 AFHTTPRequestSerializer+OAuth2 模块，这个模块实际上是 AFNetworking 其中 AFHTTPRequestSerializer 类的分类扩展，里面就声明了一个方法
```
- (void)setAuthorizationHeaderFieldWithCredential:(AFOAuthCredential *)credential;
```

它的实现如下

```
- (void)setAuthorizationHeaderFieldWithCredential:(AFOAuthCredential *)credential {
    if ([credential.tokenType compare:@"Bearer" options:NSCaseInsensitiveSearch] == NSOrderedSame) {
        [self setValue:[NSString stringWithFormat:@"Bearer %@", credential.accessToken] forHTTPHeaderField:@"Authorization"];
    }
}
```

这个方法使用传入的 credential 参数，取出其中的 accessToken 成员，并且和 Bearer 字符串组合在一起，填充到 HTTP 的 Authorization 字段，这个字段是 OAuth2.0 规范规定的，当然，很多情况下我们可能不是传递 Bearer 字符串而是其他，完全可以新增一个方法。

再来看 `AFOAuth2Manager` 模块，里面声明了继承自 `NSObject` 的 `AFOAuthCredential` 类和继承自 `AFHTTPRequestOperationManager` 的 `AFOAuth2Manager`，我们新来看 AFOAuthCredential 类

```
@interface AFOAuthCredential : NSObject <NSCoding>

@property (readonly, nonatomic, copy) NSString *accessToken;
@property (readonly, nonatomic, copy) NSString *tokenType;
@property (readonly, nonatomic, copy) NSString *refreshToken;
@property (readonly, nonatomic, assign, getter = isExpired) BOOL expired;

+ (instancetype)credentialWithOAuthToken:(NSString *)token
                               tokenType:(NSString *)type;

- (id)initWithOAuthToken:(NSString *)token
               tokenType:(NSString *)type;

- (void)setRefreshToken:(NSString *)refreshToken;

- (void)setExpiration:(NSDate *)expiration;

- (void)setRefreshToken:(NSString *)refreshToken
             expiration:(NSDate *)expiration;

+ (BOOL)storeCredential:(AFOAuthCredential *)credential
         withIdentifier:(NSString *)identifier;

+ (BOOL)storeCredential:(AFOAuthCredential *)credential
         withIdentifier:(NSString *)identifier
      withAccessibility:(id)securityAccessibility;

+ (AFOAuthCredential *)retrieveCredentialWithIdentifier:(NSString *)identifier;

+ (BOOL)deleteCredentialWithIdentifier:(NSString *)identifier;

@end
```

这个类实现了 NSCoding 协议，用于持久化，并且有4个成员变量，用于存储 accessToken、令牌类型、refreshToken 和 过期标志，基本没什么要讲的，不过我们在查看源码的时候能发现以下内容

```
+ (BOOL)storeCredential:(AFOAuthCredential *)credential
         withIdentifier:(NSString *)identifier
      withAccessibility:(id)securityAccessibility
{
    NSMutableDictionary *queryDictionary = [AFKeychainQueryDictionaryWithIdentifier(identifier) mutableCopy];
```

很明显，模块使用钥匙串来存储凭证，但是实际上钥匙串不能滥用，做过开发的朋友应该知道，用户无法自行存取钥匙串，应用程序才能使用钥匙串，但是钥匙串不像 NSUserDefault，应用程序卸载的时候钥匙串内容是不会消失的，很容易导致钥匙串内遗留垃圾数据，所以这里不应当使用自带方法存储，可以使用扩展自行实现 NSUserDefault 存储凭证。

```
- (BOOL)isExpired {
    return [self.expiration compare:[NSDate date]] == NSOrderedAscending;
}
```

这里用 Swift 的话来说就是一个计算变量。通过比较过期日期和当前日期来确定是否过期，非常简单的小技巧。
再来看最后一个 `AFOAuth2Manager` 模块

```
@interface AFOAuth2Manager : AFHTTPRequestOperationManager

@property (readonly, nonatomic, copy) NSString *serviceProviderIdentifier;
@property (readonly, nonatomic, copy) NSString *clientID;
@property (nonatomic, assign) BOOL useHTTPBasicAuthentication;

+ (instancetype)clientWithBaseURL:(NSURL *)url
                         clientID:(NSString *)clientID
                           secret:(NSString *)secret;

- (id)initWithBaseURL:(NSURL *)url
             clientID:(NSString *)clientID
               secret:(NSString *)secret;

- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                                   username:(NSString *)username
                                   password:(NSString *)password
                                      scope:(NSString *)scope
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure;

- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                                      scope:(NSString *)scope
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure;

- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                               refreshToken:(NSString *)refreshToken
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure;

- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                                       code:(NSString *)code
                                redirectURI:(NSString *)uri
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure;

- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                                 parameters:(NSDictionary *)parameters
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure;

@end

```

可以看到，这个类是继承自 AFHTTPRequestOperationManager，使用过 AFNetworking 框架的朋友应该不会陌生，这是一个网络通信类，里面有三个成员变量 `serviceProviderIdentifier`、`clientID`、`useHTTPBasicAuthentication`。

`serviceProviderIdentifier` 是用于存储和获取 OAuth 凭证的标识符，`clientID` 就是客户端ID，用于认证服务器标志客户端。最后一个就是是否将 AccessToken 存放在 Authorization 字段，默认为 YES。

所有的初始化函数最终会使用 AFHTTPRequestOperationManager 的初始化函数使用 url 初始化整个网络框架类，然后将 OAuth 认证信息传递给内部成员，最终代码如下

```
- (id)initWithBaseURL:(NSURL *)url
             clientID:(NSString *)clientID
               secret:(NSString *)secret
{
    NSParameterAssert(clientID);

    self = [super initWithBaseURL:url];
    if (!self) {
        return nil;
    }
    
    self.serviceProviderIdentifier = [self.baseURL host];
    self.clientID = clientID;
    self.secret = secret;

    self.useHTTPBasicAuthentication = YES;

    [self.requestSerializer setValue:@"application/json" forHTTPHeaderField:@"Accept"];
    
    return self;
}

```

可以看到，实际上默认 useHTTPBasicAuthentication 为 YES，并且在 HTTP 头字段添加了 application/json=Accept 键值对，表示接受 json 返回。

除了两个初始化函数以外，还有5个请求函数


```
- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
username:(NSString *)username
password:(NSString *)password
scope:(NSString *)scope 
success:(void ( ^ )( AFOAuthCredential *credential ))success 
failure:(void ( ^ ) ( NSError *error ))failure

```

这个函数很好理解，就是使用用户名和密码，并且以指定的 scope 请求 AccessToken。当然 scope 参数也可能是不存在的，因为很多后台不需要这个参数。实际上最终这个函数是根据 OAuth2.0 规范，将 grant_type、username、password、scope 四个参数打包成字典然后传递给

```
- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
parameters:(NSDictionary *)parameters 
success:(void ( ^ ) ( AFOAuthCredential *credential ))success 
failure:(void ( ^ ) ( NSError *error ))failure
```
方法。除此之外

```
- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                                      scope:(NSString *)scope
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure
- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                               refreshToken:(NSString *)refreshToken
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure
- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                                       code:(NSString *)code
                                redirectURI:(NSString *)uri
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure
```

三个函数也是将其打包成字典然后传递给最后的方法，其中第三个函数就是 OAuth 授权码模式的实现。
最后来看最终通信逻辑实现函数

```
- (AFHTTPRequestOperation *)authenticateUsingOAuthWithURLString:(NSString *)URLString
                                 parameters:(NSDictionary *)parameters
                                    success:(void (^)(AFOAuthCredential *credential))success
                                    failure:(void (^)(NSError *error))failure
{
    NSMutableDictionary *mutableParameters = [NSMutableDictionary dictionaryWithDictionary:parameters];
    if (!self.useHTTPBasicAuthentication) {
        mutableParameters[@"client_id"] = self.clientID;
        mutableParameters[@"client_secret"] = self.secret;
    }
    parameters = [NSDictionary dictionaryWithDictionary:mutableParameters];

    AFHTTPRequestOperation *requestOperation = [self POST:URLString parameters:parameters 
success:^(__unused AFHTTPRequestOperation *operation, id responseObject) {
        if (!responseObject) {
            if (failure) {
                failure(nil);
            }

            return;
        }

        if ([responseObject valueForKey:@"error"]) {
            if (failure) {
                failure(AFErrorFromRFC6749Section5_2Error(responseObject));
            }

            return;
        }

        NSString *refreshToken = [responseObject valueForKey:@"refresh_token"];
        if (!refreshToken || [refreshToken isEqual:[NSNull null]]) {
            refreshToken = [parameters valueForKey:@"refresh_token"];
        }

        AFOAuthCredential *credential = 
        [AFOAuthCredential credentialWithOAuthToken:[responseObject valueForKey:@"access_token"] 
tokenType:[responseObject valueForKey:@"token_type"]];


        if (refreshToken) { // refreshToken is optional in the OAuth2 spec
            [credential setRefreshToken:refreshToken];
        }

        // Expiration is optional, but recommended in the OAuth2 spec. 
        // It not provide, assume distantFuture === never expires
        NSDate *expireDate = [NSDate distantFuture];
        id expiresIn = [responseObject valueForKey:@"expires_in"];
        if (expiresIn && ![expiresIn isEqual:[NSNull null]]) {
            expireDate = [NSDate dateWithTimeIntervalSinceNow:[expiresIn doubleValue]];
        }

        if (expireDate) {
            [credential setExpiration:expireDate];
        }

        if (success) {
            success(credential);
        }
    } failure:^(__unused AFHTTPRequestOperation *operation, NSError *error) {
        if (failure) {
            failure(error);
        }
    }];
    
    return requestOperation;
}
```
其中主要是使用了 AFHTTPRequestOperation，最终返回也是这个对象，里面使用 block 包含了具体成功和失败的逻辑过程，包括了拆包然后提取 refresh_token 等参数，需要注意的是在这个函数中，实际上已经调用了

```
AFOAuthCredential *credential = 
[AFOAuthCredential credentialWithOAuthToken:[responseObject valueForKey:@"access_token"] 
tokenType:[responseObject valueForKey:@"token_type"]];
```
代码，也就是说，不需要开发者自己再手动将 accessToken 存储到钥匙串中。而开发者需要做的事情就是在所有的网络通信之前使用

```
[manager.requestSerializer setAuthorizationHeaderFieldWithCredential:credential];
```
将凭证嵌入到 HTTP 头中。

刷新凭证
OAuth1.0 规范中，允许 AccessToken 存在很长时间，或者是 RefreshToken 存在无限长时间，但是在 OAuth2.0 规范中就行不通了，这就需要使用 RefreshToken 刷新凭证，OAuth2.0 规范规定返回 AccessToken 的时候必须制定一个过期时间，一般是一个以秒为单位的时间长度，框架使用 `expireDate = [NSDate dateWithTimeIntervalSinceNow:[expiresIn doubleValue]];` 将其转换为 NSDate 类型存储，一般来说，可以使用 `isExpired` 函数判断是否已经过期，但是非常遗憾的是，很多情况下，后台服务器过期时间根本就是瞎编的，所以也需要注意在过期时间之前，AccessToken 已经过期了的情况，一旦出现过期或者说没有过期但是请求 API 接口返回 AccessToken 已经过期的情况，就需要使用 RefreshToken 刷新凭证，而 RefreshToken 实际上也是有一个过期日期的，但是这个过期日期规范并没有规定后台必须返回，所以就需要自行判断后台返回值，如果 RefreshToken 也已经失效，就需要使用存储的用户名密码重新登录，或者说不存储用户名密码而是弹出登录界面让用户自行填写登录。