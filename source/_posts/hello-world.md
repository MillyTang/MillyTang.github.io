---
title: Hexo & travis bot坑爹的自动部署
date: 2021-10-01
---

## 没有修改主题前的自动部署流程十分顺滑

``` bash
# git:main 分支
$ hexo new "My New Post"
$ git add .
$ git commit msg
$ git push
# 剩下的交给travis.bot发布到gh-pages分支
```

## 一切变故来源于更换了更好看的主题

  同样操作一遍，线上的直接404，在.travis.yml脚本中增加了 npm install,就此成功了一次，当第三次推送时又不行了，本地server完全OK，查看node版本，换成.travis.yml中的10.22.0,再clean掉，不行，第五次，第六次，尝试了多种办法还是不行，一气之下改成私有部署，本地1个命令，真香。

### 推荐私有部署 hexo-deployer-git 再无头秃烦恼

```yml
deploy:
  type: git
  repo: # your-git-repo-url
  token: $GH_TOKEN # 你的token, 跟在travis.com上配置是同一个token,这里写token名：$GH_TOKEN
  branch: gh-pages # 发布分支
```

``` bash
# 配置完成后接下来命令行：直接一个命令
$ npx hexo clean && npx hexo deploy
# 接下来就是vscode-auth => github.com的认证过程，按照提示操作即可
# 如果碰到 关键字中有 LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443错误，更换网络，不要用流量
# 如果碰到 Authentication failed for 'https://github.com/xxx/xxx.github.io/'，那就是travis上配置的$ $GH_TOKEN与.config.yml中的不一致
# 这两个问题都没碰到，基本成功
```

### TroubleShooting

1. 将主题换回默认主题，部署成功，说明问题出在 `Next` 主题包上
2. 在 `.travis.yml` 文件内 `install` 下增加一条安装主题的命令 `git clone --branch v8.0.0 https://github.com/next-theme/hexo-theme-next themes/next` 即可
3. 重新在`main`分支上推一下代码即可安装成功