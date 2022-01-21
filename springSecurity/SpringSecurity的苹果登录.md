

# Spring Security的苹果登录

苹果登录这篇文章要结合tiktok登录来阅读，不如许多的知识无法联通起来，推荐先阅读了参考文章，后面的代码就简单的看懂

## 参考文章

https://medium.com/tekraze/adding-apple-sign-in-to-spring-boot-app-java-backend-part-e053da331a（参考java的jwt）

https://www.baeldung.com/spring-security-custom-oauth-requests（这篇文章主要是参考springsecurity的oauth2的自定义流程）

## 获取clientId、teamId、keyId

这里获取的文章推荐apple官方https://appleid.apple.com/

## yml配置文件

```
spring:  
  security:
    oauth2:
      client:
        registration:
          apple:
            client-id: '这个是需要你去你所申请的apple应用中的service id中找到identifier的值'
            client-secret: '不需要填写静态的secret'
            authorization-grant-type: authorization_code
            redirect-uri: "https://你的域名/login/oauth2/code/apple"
            scope:
              - openid
              - name
              - email
            client-name: Apple
            client-authentication-method: post
        provider:
          apple:
            authorization-uri: https://appleid.apple.com/auth/authorize?response_mode=form_post
            token-uri: https://appleid.apple.com/auth/token
            jwk-set-uri: https://appleid.apple.com/auth/keys
            user-name-attribute: sub
```

# pom添加依赖

```
//添加jwt用来生成动态的secret
		<dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
```

# 自定义请求体中生成secret

```
package cn.ortur.auth.config;

import cn.ortur.auth.controller.AuthController;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.convert.converter.Converter;
import org.springframework.http.RequestEntity;
import org.springframework.security.oauth2.client.endpoint.OAuth2AuthorizationCodeGrantRequest;
import org.springframework.security.oauth2.client.endpoint.OAuth2AuthorizationCodeGrantRequestEntityConverter;
import org.springframework.util.MultiValueMap;
import org.bouncycastle.asn1.pkcs.PrivateKeyInfo;
import org.bouncycastle.openssl.PEMParser;
import org.bouncycastle.openssl.jcajce.JcaPEMKeyConverter;

import java.io.FileReader;
import java.security.PrivateKey;
import java.security.PublicKey;
import io.jsonwebtoken.JwsHeader;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import java.util.Date;
import java.io.File;
import org.springframework.util.ResourceUtils;

public class CustomRequestEntityConverter implements Converter<OAuth2AuthorizationCodeGrantRequest, RequestEntity<?>> {

    private static final Logger LOGGER = LoggerFactory.getLogger(CustomRequestEntityConverter.class);

    private OAuth2AuthorizationCodeGrantRequestEntityConverter defaultConverter;

    public CustomRequestEntityConverter() {
        defaultConverter = new OAuth2AuthorizationCodeGrantRequestEntityConverter();
    }

    @Override
    public RequestEntity<?> convert(OAuth2AuthorizationCodeGrantRequest req) {
        String from = req.getClientRegistration().getRegistrationId();
        RequestEntity<?> entity = defaultConverter.convert(req);
        MultiValueMap<String, String> params = (MultiValueMap<String,String>) entity.getBody();
        //如果的apple的认证就替换secret
        if ("apple".equals(from)) {
            LOGGER.info("来自APPLE");
            params.remove("client_secret");
            try {
                params.add("client_secret", generateJWT());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return new RequestEntity<>(params, entity.getHeaders(), entity.getMethod(), entity.getUrl());
    }

    private String generateJWT() throws Exception {
        // Generate a private key for token verification from your end with your creds
        PrivateKey pKey = generatePrivateKey();
        String token = Jwts.builder()
                //key-id
                .setHeaderParam(JwsHeader.KEY_ID, "LZTGDJ323Z")
                //team-id
                .setIssuer("V8793WX47W")
                .setAudience("https://appleid.apple.com")
                //client-id
                .setSubject("com.leadiffer.www")
                .setExpiration(new Date(System.currentTimeMillis() + (1000 * 60 * 5)))
                .setIssuedAt(new Date(System.currentTimeMillis()))
                //jwt use es256
                .signWith(SignatureAlgorithm.ES256, pKey)
                .compact();
        return token;
    }

    // Method to generate private key from certificate you created
    private PrivateKey generatePrivateKey() throws Exception {
        // here i have added cert at resource/apple folder. So if you have added somewhere else, just replace it with your path ofcert
        // this path can be changed depend on your download file(from apple service id download file) path
        //这个文件是你在apple所下载的key文件
        File file = ResourceUtils.getFile("classpath:apple/AuthKey_LZTGDJ323Z.p8");
        final PEMParser pemParser = new PEMParser(new FileReader(file));
        final JcaPEMKeyConverter converter = new JcaPEMKeyConverter();
        final PrivateKeyInfo object = (PrivateKeyInfo) pemParser.readObject();
        final PrivateKey pKey = converter.getPrivateKey(object);
        pemParser.close();
        return pKey;
    }

}
```

## webSecurityConfig

需要引入自定义的CustomAuthorizationRequestResolver、CustomRequestEntityConverter、CustomTokenResponseConverter具体可以阅读https://www.baeldung.com/spring-security-custom-oauth-requests

```
    @Bean
    public OAuth2AccessTokenResponseClient<OAuth2AuthorizationCodeGrantRequest> accessTokenResponseClient(){
        DefaultAuthorizationCodeTokenResponseClient accessTokenResponseClient = new DefaultAuthorizationCodeTokenResponseClient();
        accessTokenResponseClient.setRequestEntityConverter(new CustomRequestEntityConverter());
        LOGGER.info("accessTokenResponseClient{}",accessTokenResponseClient);
        OAuth2AccessTokenResponseHttpMessageConverter tokenResponseHttpMessageConverter =
                new OAuth2AccessTokenResponseHttpMessageConverter();
        tokenResponseHttpMessageConverter.setTokenResponseConverter(new CustomTokenResponseConverter());
        RestTemplate restTemplate = new RestTemplate(Arrays.asList(
                new FormHttpMessageConverter(), tokenResponseHttpMessageConverter));
        LOGGER.info("tokenResponseHttpMessageConverter{}",tokenResponseHttpMessageConverter);
        restTemplate.setErrorHandler(new OAuth2ErrorResponseErrorHandler());
        accessTokenResponseClient.setRestOperations(restTemplate);
        LOGGER.info("accessTokenResponseClient{}",accessTokenResponseClient);
        return accessTokenResponseClient;
    }
```

需要在configure的successHandler方法里面配置如下

```
.successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                        OAuth2AuthenticationToken authenticationToken = (OAuth2AuthenticationToken) authentication;
                if("apple".equals(authenticationToken.getAuthorizedClientRegistrationId())){
                            LOGGER.info("苹果登录认证完成!");
                            OAuth2User oAuth2User = (OAuth2User) authentication.getPrincipal();
                            LOGGER.info("通过loadUser获取第三方登录用户信息:{}",oAuth2User);
                            thirdPartyService.associateWithSystemUser("",oAuth2User.getAttribute("email"),authenticationToken.getAuthorizedClientRegistrationId(),oAuth2User.getAttribute("sub"));
                            LOGGER.info("存储第三方登录用户信息!");
                        }
                        RequestCache requestCache = new HttpSessionRequestCache();
                        SavedRequest savedRequest = requestCache.getRequest(request,response);
                        String redirectUrl = savedRequest.getRedirectUrl();
                        response.sendRedirect(redirectUrl);
                    }
                })
```

