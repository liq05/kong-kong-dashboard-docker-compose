# docker compose of kong and kong dashboard

- postgresql volume mapping 

- logging:fluentd+elasticsearch+kibana

#### reference

1.logging

https://docs.fluentd.org/v0.12/articles/docker-logging-efk-compose

2.kong

https://github.com/Kong/docker-kong/blob/master/compose/docker-compose.yml

3.kong dashboard

https://gist.github.com/oogali/0a3555b0f766dcecc104717203130f6e

4.kong oauth2(notice parameter:grant_type)

https://github.com/Kong/kong-oauth2-hello-world

grant_type details

https://docs.konghq.com/hub/kong-inc/oauth2/

##### grant_type as follows(four type):
config.enable_authorization_code  details:https://tools.ietf.org/html/rfc6749#section-4.1

config.enable_client_credentials  details:https://tools.ietf.org/html/rfc6749#section-4.4

config.enable_implicit_grant details:https://tools.ietf.org/html/rfc6749#section-4.2

config.enable_password_grant(the easiest type,personal recommendation--test first)

datails:https://tools.ietf.org/html/rfc6749#section-4.3 




