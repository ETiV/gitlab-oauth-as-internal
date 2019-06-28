# gitlab-oauth-as-internal
To fix OAuth sign-up defaults to external users

最近在搞 Gitlab + OAuth 登录（Google 企业邮箱），发现 Gitlab 存在这样的 Bug：  
当使用 OAuth 登录时，无论 Web 后台如何配置 Internal 用户的正则表达式，Gitlab 均会给新进用户赋予 External 角色。  
这就导致了新进用户无法创建群组或项目。

比如： https://gitlab.com/gitlab-org/gitlab-ce/issues/51805

所以就产生了修复此 Bug 的动机。

----

几经搜寻，发现日志「saving user xxx@yyy.zzz from login」附近有相关代码。

在多打了几次 log.info 后，发现默认情况 user.external 都为 true。

而他原本的判断逻辑是

```
user.external = true if external_provider? && user&.new_record?
# 以非登录表单登入的新用户，都是 external
```

然后就是修改代码了，改成了 patch 中所写的那样：

```
user.external = user_default_internal_regex_enabled? ? user_default_internal_regex_instance.match(user.email).nil? : false
# 如果设置了 internal 正则，则使用正则判断其是否为 external
# 如果没有配置 internal 正则，则 user.external = false
```

----

本 patch 适用于打在 docker container 内部路径的 `/opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/auth/o_auth/user.rb` 这个文件上  
由于 gitlab 官方镜像内部没有 patch 命令，所以可以将此文件 docker cp 出来、打 patch、再 cp 回去  
操作完使用 `gitlab-crl restart unicorn` 等待重启完毕即可生效

\- DONE -
