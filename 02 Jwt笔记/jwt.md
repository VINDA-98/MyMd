

获取令牌：

```java
@Test
public void getToken(){
    HashMap<String,Object> map = new HashMap<>();
    Calendar calendar = Calendar.getInstance();
    //一天后失效
    calendar.add(Calendar.HOUR,24);
    //设置token //payload
    String token = JWT.create()
            .withClaim("userid",1 )
            .withClaim("username","Vinda")
            .withExpiresAt(calendar.getTime()) //指定令牌过期时间
            .sign(Algorithm.HMAC256("!@#$%^&*()"));//签名
    System.out.println(token) ;
}
```



验证令牌：

```java
@Test
public void setToken(){
    //创建验证对象
    JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("!@#$%^&*()")).build();
    DecodedJWT verify = jwtVerifier.verify(
            "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9." +
                    "eyJleHAiOjE2MDAzNTY0NTQsInVzZXJpZCI6MSwidXNlcm5hbWUiOiJWaW5kYSJ9." +
                    "l6-lQCbuAE2EcNR_5BtA2LjeKfWshXYRTtWHIk0DTa8");
    System.out.println(verify.getClaim("userid"));
    System.out.println(verify.getClaim("username"));
    System.out.println(verify.getClaim("userid").asInt());
    System.out.println(verify.getClaim("username").asString());
}
```

![(D:\ADaDemo\05 Jwt笔记\token令牌验证.png)