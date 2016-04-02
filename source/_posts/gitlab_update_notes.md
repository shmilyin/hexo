title: Gitlab 源码安装版 升级笔记
date: 2016-03-29 15:00
categories:
- notes
tags:
- Gitlab
---


# Gitlab 升级笔记
>适用于使用源码安装的Gitlab

>GitLab Community Edition 8.3.2 fbb8b6e>

#### 0. Backup
```
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:backup:create RAILS_ENV=production
```
备份完成后会在`/home/git/gitlab/backup`文件夹下生成已时间戳命名的文件`1459217332_gitlab_backup.tar`

#### 1. Stop server
```
sudo service gitlab stop
```
<!--more-->

#### 2.Run GitLab upgrade tool
Please replace X.X.X with the [latest GitLab release](https://packages.gitlab.com/gitlab/gitlab-ce).

GitLab 7.9 adds `nodejs` as a dependency. GitLab 7.6 adds `libkrb5-dev` as a dependency (installed by default on Ubuntu and OSX). GitLab 7.2 adds `pkg-config` and `cmake` as dependency. Please check the dependencies in the [installation guide](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/install/installation.md#1-packages-dependencies).

```
cd /home/git/gitlab
sudo -u git -H ruby -Ilib -e 'require "gitlab/upgrader"' -e 'class Gitlab::Upgrader' -e 'def latest_version_raw' -e '"vX.X.X"' -e 'end' -e 'end' -e 'Gitlab::Upgrader.new.execute'

# to perform a non-interactive install (no user input required) you can add -y
# sudo -u git -H ruby -Ilib -e 'require "gitlab/upgrader"' -e 'class Gitlab::Upgrader' -e 'def latest_version_raw' -e '"vX.X.X"' -e 'end' -e 'end' -e 'Gitlab::Upgrader.new.execute' -- -y
```

*小插曲：*执行命令时报错

```
from /home/git/gitlab/lib/gitlab/upgrader.rb:74:in `upgrade'

    from /home/git/gitlab/lib/gitlab/upgrader.rb:22:in `execute'

    from -e:7:in `<main>'
```
解决办法

```
Open /home/git/gitlab/lib/gitlab/upgrader.rb in editor
Replace all #{Gitlab.config.git.bin_path} by git and save
Run upgrader.rb as usual
```


I have the same problem upgrading from 8.5.1 to 8.5.5. Replacing `Gitlab.config.git.bin_path` with `git` fixed it.
This is the search and replace command to run in vi: `%s/#{Gitlab.config.git.bin_path}/git/g`

#### 3. Start application
```
sudo service gitlab start
sudo service nginx restart
```
#### 4. Check application status
Check if GitLab and its dependencies are configured correctly:

```
sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
```
If all items are green, then congratulations upgrade is complete!

#### 5. Upgrade GitLab Shell
GitLab Shell might be outdated, running the commands below ensures you're using a compatible version:

```
cd /home/git/gitlab-shell
sudo -u git -H git fetch
sudo -u git -H git checkout v`cat /home/git/gitlab/GITLAB_SHELL_VERSION`
```
#### One line upgrade command

You've read through the entire guide and probably already did all the steps one by one.
Below is a one line command with step 1 to 5 for the next time you upgrade.
Please replace X.X.X with the [latest GitLab release](https://packages.gitlab.com/gitlab/gitlab-ce).

```
cd /home/git/gitlab; \
  sudo -u git -H bundle exec rake gitlab:backup:create RAILS_ENV=production; \
  sudo service gitlab stop; \
  sudo -u git -H ruby -Ilib -e 'require "gitlab/upgrader"' -e 'class Gitlab::Upgrader' -e 'def latest_version_raw' -e '"vX.X.X"' -e 'end' -e 'end' -e 'Gitlab::Upgrader.new.execute' -- -y; \
  cd /home/git/gitlab-shell; \
  sudo -u git -H git fetch; \
  sudo -u git -H git checkout v`cat /home/git/gitlab/GITLAB_SHELL_VERSION`; \
  cd /home/git/gitlab; \
  sudo service gitlab start; \
  sudo service nginx restart; \
  sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
```
####Note:
阿里云服务器ruby源不稳定。换国内的源

```
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
gem sources -l
# 请确保只有 ruby.taobao.org
gem install rails
```

```
sudo -u git bundle install
```

安装完成之后 `update init script`

```
rm /etc/init.d/gitlab
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
```

升级是由于是从7.x 到8.x 样式变化很大，所以执行完后样式错乱。后来找了些文档才发现要重新生产asset文件。

```
sudo -u git -H bundle exec rake assets:clean assets:precompile cache:clear RAILS_ENV=production
```
重启

```
service gitlab start
service nginx restart
```
Done!


