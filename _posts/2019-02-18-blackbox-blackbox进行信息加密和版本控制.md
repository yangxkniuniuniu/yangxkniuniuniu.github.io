---
layout:     post
title:      blackbox配合git进行信息加密与版本管理
subtitle:   blackbox
date:       2019-04-18
author:     owl city
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - blackbox
    - git
    - encrypt
    - gitlab ci
---

#### 背景
**存在问题:**
    - edw配置文件(db连接信息)统计管理较为繁琐, 团队某一成员修改配置后, 其他成员需要手动进行同步

    - 升级后的airflow在同步配置文件时, 也需要人为手动更新,与自动化部署思想相左

**解决方案:**
    - 使用git版本控制工具统一化管理配置文件

    - 对上传到git的文件进行加密, 并实现让指定人员才能进行加密与解密

    - 进行自动化部署配置, 在airflow每次重新部署的时候自动加载并解密配置文件, 同时进行配置文件的解析, 读取并写入Variables和Connections.


#### 安装blackbox, gpg, git
- `git install git`

- `git install gpg`

- `git install blackbox`

#### blackbox介绍
> **[Safely store secrets in a VCS repo !](https://github.com/StackExchange/blackbox)**

**before:**
- git : 版本控制工具

- Gnupg : 可以使用RSA算法对信息进行加密和解密的工具, 是目前最流行, 最好用的加密工具之一.

**blackbox:** blackbox使用GPG对文件进行加密和解密, 并使用git进行版本管理
> blackbox支持多种VCS仓库, 如Git, Mercurial, Subversion or Perforce

blackbox使用可以指定一组秘钥来对信息进行加密解密的模式. 只需要将admins的秘钥添加到keychain中, blackbox会使用keychain中的每个秘钥对信息加密, 这样任何一个admin都能够对加密后的信息进行解密.


#### 配置blackbox

- 在git项目中进行blackbox初始化: `blackbox_initialize`  (**只需要在首次创建项目或者为已有项目添加blackbox时进行**)

#### 简单使用
- 生成自己的秘钥: `gpg --gen-key` (按默认值设置就可以)

- 添加到管理员列表: `blackbox_addadmin user_id` (user_id可以是上一步输入的用户名, 也可以是邮箱地址)

- 选择需要加密的文件: `blackbox_register_new_file  xxx.yml`

- 上传加密后的文件到git

- 本地查看或编辑加密文件

    - 第一种方式: `blackbox_edit xxx.gpg`

    - 第二种方式: `blackbox_edit_start xxx.gpg`,这样会生成一个xxx.yml的文件,可以使用各种编辑器打开并进行修改,修改完成之后保存并再次加密`blackbox_edit_end xxx.yml`


#### 添加新的使用者
- 新用户需要做的:
    - 使用gpg生成自己的秘钥: `gpg --gen-key`
    - 把自己添加到管理员列表中(相当于申请权限): `blackbox_addadmin user_id`
    - 按照提示的信息提交修改到远程仓库

- 已经是admin的成员同意申请
    - `git pull`
    - `gpg --homedir=.blackbox --list-keys` 查看已有的Keys
    - 导入新用户: `gpg --keyring .blackbox/pubring.kbx  --export | gpg --import`
    - 重新加密所有文件 : `blackbox_update_all_files`
    - 提交更新


#### 移除使用者
- 移除用户: `blackbox_removeadmin user_id`
- 重新加密所有文件: `blackbox_update_all_files`

> 移除使用者之后,虽然这个用户不再有查看和修改加密文件的权限,但是用户的key仍然存在于仓库的keychains中,如果需要完全移除:

    1. `gpg --homedir=.blackbox --list-keys`

    2. `gpg --homedir=.blackbox --delete-key user_id`

    3. `git commit -m'Cleaned olduser@example.com from keyring'  .blackbox/*`


#### 添加git diff
- 在项目的顶层目录下添加`.gitattributes`文件
- 将下列内容写入.gitattributes文件中: `*.gpg diff=blackbox`
- 将下列内容添加到.git/config文件中:
```
[diff "blackbox"]
    textconv = gpg --use-agent -q --batch --decrypt
```
- 使用 `git log -p etl_config.gpg`进行验证

#### 在自动化部署中自动解密加密文件 **Gitlab-ci && blackbox**
因为GPG密钥必须有密码。但是，密码在子项上是可选的。因此，我们将创建一个带密码的密钥，然后创建一个不带密码的子密钥.

[具体实现步骤](https://medium.com/@mipselaer/gitlab-ci-blackbox-526c7ad7bec0)


#### 放在最后 (命令大全)

| Name:                               | Description:                                                            |
|-------------------------------------|-------------------------------------------------------------------------|
| `blackbox_edit <file>`              | Decrypt, run $EDITOR, re-encrypt a file                                 |
| `blackbox_edit_start <file>`        | Decrypt a file so it can be updated                                     |
| `blackbox_edit_end <file>`          | Encrypt a file after blackbox_edit_start was used                       |
| `blackbox_cat <file>`               | Decrypt and view the contents of a file                                 |
| `blackbox_view <file>`              | Like blackbox_cat but pipes to `less` or $PAGER                         |
| `blackbox_diff`                     | Diff decrypted files against their original crypted version             |
| `blackbox_initialize`               | Enable blackbox for a GIT or HG repo                                    |
| `blackbox_register_new_file <file>` | Encrypt a file for the first time                                       |
| `blackbox_deregister_file <file>`   | Remove a file from blackbox                                             |
| `blackbox_list_files`               | List the files maintained by blackbox                                   |
| `blackbox_list_admins`              | List admins currently authorized for blackbox                           |
| `blackbox_decrypt_file <file>`      | Decrypt a file                                                          |
| `blackbox_decrypt_all_files`        | Decrypt all managed files (INTERACTIVE)                                 |
| `blackbox_postdeploy`               | Decrypt all managed files (batch)                                       |
| `blackbox_addadmin <gpg-key>`       | Add someone to the list of people that can encrypt/decrypt secrets      |
| `blackbox_removeadmin <gpg-key>`    | Remove someone from the list of people that can encrypt/decrypt secrets |
| `blackbox_shred_all_files`          | Safely delete any decrypted files                                       |
| `blackbox_update_all_files`         | Decrypt then re-encrypt all files. Useful after keys are changed        |
| `blackbox_whatsnew <file>`          | show what has changed in the last commit for a given file               |
