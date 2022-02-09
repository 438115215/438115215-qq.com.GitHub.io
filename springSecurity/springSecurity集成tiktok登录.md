# 申请tiktok开发者账号

## 前置条件

https://medium.com/tekraze/adding-apple-sign-in-to-spring-boot-app-java-backend-part-e053da331a（参考java的jwt）

https://www.baeldung.com/spring-security-custom-oauth-requests（这篇文章主要是参考springsecurity的oauth2的自定义流程）

### google邮箱

由于google邮箱可以使用中国号码注册，所以使用tiktok的第三方登录来登录google账号，用以登录tiktok开发者

### 域名

tiktok开发者申请app需要有一个审核过程，该审核过程需要一个域名和一个隐私条款，推荐使用wordpress，wordpress建站自带隐私条款

### 回调地址

由于使用的是spring security这个框架 所以回调地址 一般填 

`https://你的域名/login/oauth2/code/tiktok`

### 获取key和secret

申请完成之后tiktok会给你一个key还有secret,是用来验证sso的

# 配置文件配置

## oauth2

一般来说sso都是符合oauth2协议的,所以会有三个请求地址需要填写

### 认证地址

这个地址一般是用来跳进tiktok的登录页面的，登录成功会返回一个code，code可以用来请求access_token

`https://open-api.tiktok.com/platform/oauth/connect/`

### 获取token地址

这个地址一般是用来获取access_token的，需要在请求参数里面加入code

`https://open-api.tiktok.com/oauth/access_token/`

### 获取用户信息地址

这个地址是用来获取tiktok用户的信息的，需要在请求参数里面加入access_token

`https://open-api.tiktok.com/user/info/`

## spring的配置文件配置

## yml格式的provider和registration配置

由于spring Security并没有内置tiktok的登录配置所以我们需要手动配置provider

配置参考如下，使用的是yml格式

```
spring:  
  security:
    oauth2:
      client:
        registration:
          tiktok:
            client-id: 你的client_id
            client-secret: 你的client_secret
            authorization-grant-type: authorization_code
            redirect-uri: "https://域名/login/oauth2/code/tiktok"
            scope:
              - user.info.basic
        provider:
          tiktok:
            authorization-uri: https://open-api.tiktok.com/platform/oauth/connect/
            token-uri: https://open-api.tiktok.com/oauth/access_token/
            user-info-uri: https://open-api.tiktok.com/user/info/
            user-name-attribute : sub
```

## websecurityconfig的配置

由于我们是自己添加的privider所以我们需要在配置类做一些配置

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    private static final Logger LOGGER = LoggerFactory.getLogger(WebSecurityConfig.class);

	//tiktok流程完成后做的一些事的类，如将用户信息存入数据库
    @Autowired
    OAuth2Service oauth2Service;

    @Autowired
    private ClientRegistrationRepository clientRegistrationRepository;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.csrf()
                .disable()
                .authorizeRequests()
                .antMatchers("/oauth/**", "/actuator/**", "/login", "/user/**", "/static/**", "/v2/api-docs", "/logout") //匹配这些path的
                .permitAll()  //全部放行
                .anyRequest() //其他请求
                .authenticated() //需要认证
                .and()
                .formLogin() //自定义登录
                .loginProcessingUrl("/login") //登录处理url
                .loginPage("/oauth/login") //自定义登录页面
                .failureUrl("/oauth/login?error") //失败登录页面
                .and()
                .logout() // 启用登出
                .deleteCookies("JSESSIONID")
                .invalidateHttpSession(true)
                .logoutUrl("/oauth/logout") // 登录请求url
                .logoutSuccessHandler(new logoutSuccessHandle()) //传入的是登出成功后进行的处理类
                .and().oauth2Login()
                .authorizationEndpoint() //认证端点
                .authorizationRequestResolver(
                        new CustomAuthorizationRequestResolver(
                                this.clientRegistrationRepository)) //传入的是自定义的认证端点处理类
                .and()
                .tokenEndpoint().accessTokenResponseClient(accessTokenResponseClient())//传入的是自定义获取token端点处理类
                .and()
                .loginPage("/oauth/login")
                .userInfoEndpoint()
                .userService(oauth2Service)传入的是用户信息获取后的处理类
                ;
    }
    
    
    private GrantedAuthoritiesMapper userAuthoritiesMapper() {
        return (authorities) -> {
            LOGGER.info("进入userAuthoritiesMapper");
            Set<GrantedAuthority> mappedAuthorities = new HashSet<>();
            authorities.forEach(authority -> {
                if (OidcUserAuthority.class.isInstance(authority)) {
                    OidcUserAuthority oidcUserAuthority = (OidcUserAuthority)authority;

                    OidcIdToken idToken = oidcUserAuthority.getIdToken();
                    OidcUserInfo userInfo = oidcUserAuthority.getUserInfo();
                    LOGGER.info("idToken:{}",idToken);
                    LOGGER.info("userinfo:{}",userInfo);
                    // Map the claims found in idToken and/or userInfo
                    // to one or more GrantedAuthority's and add it to mappedAuthorities

                } else if (OAuth2UserAuthority.class.isInstance(authority)) {
                    OAuth2UserAuthority oauth2UserAuthority = (OAuth2UserAuthority)authority;

                    Map<String, Object> userAttributes = oauth2UserAuthority.getAttributes();

                    // Map the attributes found in userAttributes
                    // to one or more GrantedAuthority's and add it to mappedAuthorities
                    LOGGER.info("userAttributes{}", userAttributes);
                }
            });
            return mappedAuthorities;
        };
    }

	//用户信息处理
    @Bean
    public OAuth2AccessTokenResponseClient<OAuth2AuthorizationCodeGrantRequest> accessTokenResponseClient(){
    //创建一个获取accessToken的类
        DefaultAuthorizationCodeTokenResponseClient accessTokenResponseClient = new DefaultAuthorizationCodeTokenResponseClient();
        //将自定义获取的accesstoken的类传入
        accessTokenResponseClient.setRequestEntityConverter(new CustomRequestEntityConverter());
        LOGGER.info("accessTokenResponseClient{}",accessTokenResponseClient);
        //创建一个token的response的类
        OAuth2AccessTokenResponseHttpMessageConverter tokenResponseHttpMessageConverter =
                new OAuth2AccessTokenResponseHttpMessageConverter();
                //设置自定义的token的response的类
        tokenResponseHttpMessageConverter.setTokenResponseConverter(new CustomTokenResponseConverter());
        RestTemplate restTemplate = new RestTemplate(Arrays.asList(
                new FormHttpMessageConverter(), tokenResponseHttpMessageConverter));
        LOGGER.info("tokenResponseHttpMessageConverter{}",tokenResponseHttpMessageConverter);
        //设置默认处理handler
        restTemplate.setErrorHandler(new OAuth2ErrorResponseErrorHandler());
        //将返回传入系统
        accessTokenResponseClient.setRestOperations(restTemplate);
        
        LOGGER.info("accessTokenResponseClient{}",accessTokenResponseClient);
        // 返回accesstoken客户端
        return accessTokenResponseClient;
    }
    // 处理登出url回调 / 问题，重定向到login
    class logoutSuccessHandle implements LogoutSuccessHandler{
        @Override
        public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
            System.out.println(request.getRequestURI());
            String client_id = request.getParameter("client_id");
            String scope = request.getParameter("scope");
            String redirect_uri = request.getParameter("redirect_uri");
            String response_type = request.getParameter("response_type");
            String state = request.getParameter("state");
            String query = "?client_id=" + client_id +
                               "&scope=" + scope     +
                        "&redirect_uri=" + redirect_uri +
                       "&response_type=" + response_type +
                               "&state=" + state;
            LOGGER.info("authentication:{}",authentication);
            response.sendRedirect("/oauth/authorize"+query);
        }
    }
}

```

# 编写自定义处理sso

## 首先发起authorization请求获取code码

在websecurityconfig的同级目录下创建CustomAuthorizationRequestResolver类，处理在spring security过程中的authorization过程，该类最重要的是重写的resolve方法，这个方法可以在认证过程中获取一些信息，以便我们进行修改，由于tiktok，authorization需要的key和github、google、facebook不同（他们需要带入client_id），tiktok需要的是client_key才能显示登录页面，所以这里我们主要思路就是如果传入的是tiktok我们就把client_id替换成client_key

```
//为了介入security的sso过程我们需要实现OAuth2AuthorizationRequestResolver这个接口的resolve方法
public class CustomAuthorizationRequestResolver implements OAuth2AuthorizationRequestResolver {
    private final OAuth2AuthorizationRequestResolver defaultAuthorizationRequestResolver;

    private static final Logger LOGGER = LoggerFactory.getLogger(AuthController.class);

    public CustomAuthorizationRequestResolver(ClientRegistrationRepository clientRegistrationRepository) {
        this.defaultAuthorizationRequestResolver =
                new DefaultOAuth2AuthorizationRequestResolver(clientRegistrationRepository, "/oauth2/authorization");
    }

    @Override
    public OAuth2AuthorizationRequest resolve(HttpServletRequest request) {
        OAuth2AuthorizationRequest authorizationRequest = this.defaultAuthorizationRequestResolver.resolve(request);
        return authorizationRequest != null ? customAuthorizationRequest(authorizationRequest) : null;
    }

    @Override
    public OAuth2AuthorizationRequest resolve(
            HttpServletRequest request, String clientRegistrationId) {
        OAuth2AuthorizationRequest authorizationRequest =
                this.defaultAuthorizationRequestResolver.resolve(
                        request, clientRegistrationId);

        return authorizationRequest != null ?
                customAuthorizationRequest(authorizationRequest) :
                null;
    }

//这个方法就是我们自定义的用来介入的主要逻辑，会被上面的resolve方法调用
    private OAuth2AuthorizationRequest customAuthorizationRequest(
            OAuth2AuthorizationRequest authorizationRequest) {
        LOGGER.info("resolve request:{}", authorizationRequest);
        LOGGER.info("resolve getAuthorizationRequestUri:{}", authorizationRequest.getAuthorizationRequestUri());
        LOGGER.info("resolve getAdditionalParameters:{}", authorizationRequest.getAdditionalParameters());
        LOGGER.info("resolve getAttributes:{}", authorizationRequest.getAttributes());
        LOGGER.info("resolve getAuthorizationUri:{}", authorizationRequest.getAuthorizationUri());
        //将getAdditionalParameters的参数取出来放入map中
        Map<String, Object> additionalParameters =
                new LinkedHashMap<>(authorizationRequest.getAdditionalParameters());
                //我们只对tiktok的clientid做处理，所以这里是一个判断是否是tiktok的authorization
        if (authorizationRequest.getAttribute("registration_id").equals("tiktok")){
            LOGGER.info("tiktok命中");
            //在参数里面添加clientkey
            additionalParameters.put("client_key", authorizationRequest.getClientId());
            //移除clientId
            additionalParameters.remove("client_id");
        }
        //让security继续下一步，传入参数和认证请求
        return OAuth2AuthorizationRequest.from(authorizationRequest)
                .additionalParameters(additionalParameters)
                .build();
    }
}
```

## 其次使用获取的code码去获取access_token

上面的步骤后面如果用户选择授权登录，那么我们这边将会获取到一个code码，接下来就是要使用code码去发起请求获取access_token,我们同样要编写一个自定义的类来发起获取access_token的请求，还是和上面一样tiktok是不接收clientId的所以还是要修改成clientKey

```
//这个要接入security我们要实现Converter这个接口
public class CustomRequestEntityConverter implements Converter<OAuth2AuthorizationCodeGrantRequest, RequestEntity<?>> {

    private static final Logger LOGGER = LoggerFactory.getLogger(CustomRequestEntityConverter.class);

    private OAuth2AuthorizationCodeGrantRequestEntityConverter defaultConverter;

    public CustomRequestEntityConverter() {
        defaultConverter = new OAuth2AuthorizationCodeGrantRequestEntityConverter();
    }

//主要的是重写convert方法，在这个方法内，我们就可以修改参数和请求
    @Override
    public RequestEntity<?> convert(OAuth2AuthorizationCodeGrantRequest req) {
    //获取是那个sso来源
        String from = req.getClientRegistration().getRegistrationId();
        LOGGER.info("req.getClientRegistration().getRegistrationId():{}",req.getClientRegistration().getRegistrationId());
        LOGGER.info("req.getGrantType().getValue():{}",req.getGrantType().getValue());
        LOGGER.info("req.getAuthorizationExchange().getAuthorizationRequest().getAttributes():{}",req.getAuthorizationExchange().getAuthorizationRequest().getAttributes());
        //获取参数
        RequestEntity<?> entity = defaultConverter.convert(req);
        //将body的参数存起来等会使用
        MultiValueMap<String, String> params = (MultiValueMap<String,String>) entity.getBody();
        LOGGER.info("params:{}",params);
        LOGGER.info("entity.getHeaders():{}", entity.getHeaders());
        LOGGER.info("entity.getMethod():{}",entity.getMethod());
        LOGGER.info("entity.getUrl():{}",entity.getUrl());
        LOGGER.info("client_key:{}", req.getClientRegistration().getClientId());
        LOGGER.info("client_secret:{}", req.getClientRegistration().getClientSecret());
        LOGGER.info("属性：{}", req.getAuthorizationExchange().getAuthorizationRequest().getAttributes());
        LOGGER.info("属性：{}", req.getAuthorizationExchange().getAuthorizationRequest().getAdditionalParameters());
        LOGGER.info("属性：{}", req);
        //判断是否是tiktok，如果是tiktok我们做特殊处理
        if ("tiktok".equals(from)) {
            LOGGER.info("来自TIKTOK");
            在参数里面加入clientkey和clientSecret
            params.add("client_key",req.getClientRegistration().getClientId());
            params.add("client_secret",req.getClientRegistration().getClientSecret());
        }
        LOGGER.info("params:{}",params);
        //继续securtiy的sso流程
        return new RequestEntity<>(params, entity.getHeaders(), entity.getMethod(), entity.getUrl());
    }
}
```

## 根据accessToken获取用户信息

由于tiktok传回的信息格式和springsecurity的处理是不同的，tiktok传入的格式一般是

```
{access_token=你的token, captcha=, desc_url=, description=, error_code=0, expires_in=86400, log_id=你的id, open_id=你的id, refresh_expires_in=31536000, refresh_token=你的refresh_token, scope=user.info.basic}
```

而springsecurity处理过程中接收的是Map<String,String>的格式，这样的话只能获取一层的信息，所以我们需要进行一个处理

```
public class CustomTokenResponseConverter implements Converter<Map<String, String>, OAuth2AccessTokenResponse> {

    private static final Logger LOGGER = LoggerFactory.getLogger(CustomTokenResponseConverter.class);

//设置一个set来存储上面获取到的信息
    private static final Set<String> TOKEN_RESPONSE_PARAMETER_NAMES = Stream.of(
            OAuth2ParameterNames.ACCESS_TOKEN,
            OAuth2ParameterNames.TOKEN_TYPE,
            OAuth2ParameterNames.EXPIRES_IN,
            OAuth2ParameterNames.REFRESH_TOKEN,
            OAuth2ParameterNames.SCOPE) .collect(Collectors.toSet());

//将string转成map
    public static Map<String, String> getStringToMap(String str) {
        // 判断str是否有值
        if (null == str || "".equals(str)) {
            return null;
        }
        // 根据&截取
        String[] strings = str.split(", ");
        // 设置HashMap长度

        int mapLength = strings.length;
        Map<String, String> map = new HashMap<>(mapLength);
        // 循环加入map集合
        for (String string : strings) {
            // 截取一组字符串
            String[] strArray = string.split("=");
            String key = strArray[0];
            String value = "";
            if (strArray.length >= 2) {
                value = strArray[1];
            }
            map.put(key, value);
        }
        return map;
    }
    

//这个方法就是主要的介入security的方法，这个Map很可惜只能是<String,String>的格式
    @Override
    public OAuth2AccessTokenResponse convert(Map<String, String> tokenResponseParameters) {
        LOGGER.info("进入OAuth2AccessTokenResponse");
        LOGGER.info("tokenResponseParameters{}", tokenResponseParameters);
        LOGGER.info("tokenResponseParameters.keySet(){}", tokenResponseParameters.keySet());
        LOGGER.info("tokenResponseParameters.data{}", tokenResponseParameters.get("data"));
        //我们判断如果map的key存在data和message那就判断是tiktok
        if (tokenResponseParameters.keySet().contains("data") && tokenResponseParameters.keySet().contains("message")){
            LOGGER.info("message与data命中，是tiktok认证");
            //我们将{号和}号去除然后调用自定义的string转map
            String data = tokenResponseParameters.get("data").substring(1, tokenResponseParameters.get("data").length()-1);
            调用自定义方法转map
            Map<String, String> stringToMap = getStringToMap(data);
            获取map的参数存入请求参数
            tokenResponseParameters.put("access_token",stringToMap.get("access_token"));
            tokenResponseParameters.put("expires_in",stringToMap.get("expires_in"));
            tokenResponseParameters.put("scope",stringToMap.get("scope"));
            tokenResponseParameters.put("refresh_token",stringToMap.get("refresh_token"));
            tokenResponseParameters.put("open_id",stringToMap.get("open_id"));
        }

        String accessToken = tokenResponseParameters.get(OAuth2ParameterNames.ACCESS_TOKEN);
        LOGGER.info("获取accessToken的值:{}", accessToken);
        OAuth2AccessToken.TokenType accessTokenType = OAuth2AccessToken.TokenType.BEARER;
        LOGGER.info("获取accessTokenType的值:{}", accessTokenType.getValue());
        long expiresIn = 0;
        if (tokenResponseParameters.containsKey(OAuth2ParameterNames.EXPIRES_IN)) {
            try {
                expiresIn = Long.valueOf(tokenResponseParameters.get(OAuth2ParameterNames.EXPIRES_IN));
            } catch (NumberFormatException ex) { }
        }
        LOGGER.info("expiresIn:{}", expiresIn);

        Set<String> scopes = Collections.emptySet();
        if (tokenResponseParameters.containsKey(OAuth2ParameterNames.SCOPE)) {
            String scope = tokenResponseParameters.get(OAuth2ParameterNames.SCOPE);
            scopes = Arrays.stream(StringUtils.delimitedListToStringArray(scope, " ")).collect(Collectors.toSet());
        }
        LOGGER.info("scopes:{}", scopes);
        String refreshToken = tokenResponseParameters.get(OAuth2ParameterNames.REFRESH_TOKEN);
        LOGGER.info("refreshToken:{}", refreshToken);
        Map<String, Object> additionalParameters = new LinkedHashMap<>();
        //将tokenResponseParameters的参数遍历加入additionalParameters中
        tokenResponseParameters.entrySet()
                .stream()
                .filter(e -> !TOKEN_RESPONSE_PARAMETER_NAMES.contains(e.getKey()))
                .forEach(e -> additionalParameters.put(e.getKey(), e.getValue()));
        LOGGER.info("tokenResponseParameters:{}", tokenResponseParameters);
        //发起请求
        return OAuth2AccessTokenResponse.withToken(accessToken)
                .tokenType(accessTokenType)
                .expiresIn(expiresIn)
                .scopes(scopes)
                .refreshToken(refreshToken)
                .additionalParameters(additionalParameters)
                .build();
    }
}
```

## 最后编写继承DefaultOAuth2UserService的类

最重要的就是实现loadUser方法

```
public class OAuth2Service extends DefaultOAuth2UserService {

    private static final Logger LOGGER = LoggerFactory.getLogger(OAuth2Service.class);

    @Autowired
    ThirdPartyService thirdPartyService;

    private RestTemplate restTemplate = new RestTemplate();

//将获取到用户做一个处理
    public Object getResponse(OAuth2UserRequest userRequest) {
        HttpHeaders httpHeaders = new HttpHeaders();

        httpHeaders.set(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
        httpHeaders.set(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE);

        String uri = userRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUri();

        HashMap requestBody = new HashMap<String, Object>();
        requestBody.put("open_id", userRequest.getAdditionalParameters().get("open_id"));
        requestBody.put("access_token", userRequest.getAccessToken().getTokenValue());

        Set Set = new HashSet<>();
        Set.add("open_id");
        Set.add("display_name");
        Set.add("avatar_url");
        Set.add("union_id");
        requestBody.put("fields", Set);
        String json = JSONObject.toJSONString(requestBody);
        try {
            LOGGER.info("打标");
            ResponseEntity<String> responseEntity = restTemplate.postForEntity(uri, new HttpEntity<>(json, httpHeaders), String.class);
            LOGGER.info("responseEntity:{}",responseEntity);
            // 将响应体转化成jsonObject类
            JSONObject jsonObject = JSONObject.parseObject(responseEntity.getBody());
            LOGGER.info("jsonObject:{}",jsonObject);
            return jsonObject;
        }catch (Exception e){
            LOGGER.info("error:{}", e.getMessage());
            return ""+e.getMessage();
        }
    }

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        LOGGER.info("第三方登录认证完成！");
        LOGGER.info("getAdditionalParameters:{}",userRequest.getAdditionalParameters());
        LOGGER.info("getAccessToken:{}",userRequest.getAccessToken());
        //我们判断是否是tiktok的sso
        if (userRequest.getClientRegistration().getRegistrationId().equals("tiktok")) {
            LOGGER.info("tiktok信息获取");
            //处理用户信息
            JSONObject response = (JSONObject) getResponse(userRequest);
            JSONObject data = (JSONObject) response.get("data");
            JSONObject user = (JSONObject) data.get("user");
            Map<String, Object> userAttributes = new HashMap<>();
            userAttributes.put("avatar_url", user.get("avatar_url"));
            userAttributes.put("open_id", user.get("open_id"));
            userAttributes.put("display_name", user.get("display_name"));
            userAttributes.put("name", user.get("display_name"));
            userAttributes.put("union_id", user.get("union_id"));
            Set<GrantedAuthority> authorities = new LinkedHashSet<>();
            authorities.add(new OAuth2UserAuthority(userAttributes));
            OAuth2AccessToken token = userRequest.getAccessToken();
            for (String authority : token.getScopes()) {
                authorities.add(new SimpleGrantedAuthority("SCOPE_" + authority));
            }
            thirdPartyService.associateWithSystemUser((String) userAttributes.get("name"),"null",userRequest.getClientRegistration().getRegistrationId(),(String) userAttributes.get("union_id"));
            返回
            return new DefaultOAuth2User(authorities, userAttributes, "display_name");
        }
        try {
            OAuth2User oAuth2User = super.loadUser(userRequest);
            LOGGER.info("通过loadUser获取第三方登录用户信息:{}",oAuth2User);
            thirdPartyService.associateWithSystemUser(oAuth2User.getAttribute("name"),oAuth2User.getAttribute("email"),userRequest.getClientRegistration().getRegistrationId(),oAuth2User.getAttribute("id").toString());
            LOGGER.info("存储第三方登录用户信息!");
            return oAuth2User;
        }catch (Exception e) {
            LOGGER.info("error:{}",e.getMessage());
        }
            return super.loadUser(userRequest);
    }
}
```



