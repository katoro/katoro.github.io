---
layout: post
title: "Springboot Oauth2 Refresh token 문제"
date: 2017-05-02 15:02:19
image: '/assets/img/'
description:
tags:
- springboot
- oauth2
categories:
- development
twitter_text:
---

# 현상

REST서버와 Auth서버가 분리된 형태이다. 추후에 전체 구조와 초기구성방법에 대해 포스팅해둬야겠다.

Auth서버에서 아이디(이메일)/패스워드의 형태로 로그인하면 access_token이 발급되며 해당 토큰으로 REST Api를 이용할수 있다.

토큰이 발급될때 Refresh_token도 함께 발급되는데, access_token이 만료될 경우 refresh_token을 이용하여 갱신하는 형태이다.

access_token의 유효시간은 10초, refresh_token의 유효시간은 60초로 설정한뒤 테스트해봤다. (실제로 이렇게 짧게하면 안된다.)

첫 발급 후 access토큰의 경우 10초간은 유효하였으며, 10초 이후로는 유효하지 않았다.

refresh_token을 이용하여 갱신하는것은 첫발급이후 60초까지만 유효하였으며 초과한경우 갱신이 되지 않았다. 물론 access토큰도 사라진다.

jdbc token store를 이용하고 있는 상태라 실제 db를 관찰해보니 60초이후 인증관련 행위를 진행하면 access, refresh토큰 컬럼이 삭제된다 (oauth_access_token, oauth_refresh_token)

# 해결방법

- client가 잘못되었다
는 무슨ㅎㅎ 그런건 없었다. 다음 형태로 잘 보내고 있었다.


	@FormUrlEncoded
    @POST("oauth/token")
    Call<AccessTokenResponse> getRefreshAccessToken(
            @Header("Authorization") String authorization,
            @Field("refresh_token") String refreshToken,
            @Field("grant_type") String grantType);


> Header의 경우 base64 인코딩을 해줘야하며, Grant_type은 refresh_token 이다.


- Springboot configure 문제이다
이게맞았다. 


	@Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(userDetailService)
                .tokenStore(jdbcTokenStore());
    }


여기서 한줄이 빠졌다.

	@Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(userDetailService)
                .tokenStore(jdbcTokenStore())
                .reuseRefreshTokens(false);
	}
    
RefreshToken관련 함수를 아무래도 구현해줘야하는것으로 생각된다.

스프링부트 공부를 다시 해야할것같다. 작년에 해둔거를 땜빵해서 쓰는게 여간 힘든게 아니다.
