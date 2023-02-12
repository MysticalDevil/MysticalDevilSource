---
title: "如何批量修改 Git 提交记录中的用户名和邮箱"
date: 2023-02-12T15:35:20+08:00
draft: true
categories: [
"Git"
]
"tags": [
"Git",
"Skill"
]
---

### 适用场景

>  已经提交了N个commit才发现用的配置(user.name 和 user.email)错了，比如要用个人邮箱的，用成了公司邮箱。 基于隐私考虑，我们需要把公司邮箱和昵称替换掉。

主要是用到 [git-filter-repo](https://github.com/newren/git-filter-repo)的**CALLBACKS**功能

参考文档 https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#CALLBACKS

### 使用方法

```shell
#将所有用户名中包含的 foo 替换成 devil （注意，不支持中文)
git filter-repo --name-callback 'return name.replace(b"foo", b"devil")'

#将所有commit信息的email中包含的 foo@example.com 替换成 devil@example.com
git filter-repo --email-callback 'return email.replace(b"foo@example.com", b"devil@example.com")'
```

注意：

> 1. 替换是部分匹配的，因此注意使用尽可能长的子串
> 2. 操作会重写所有被匹配到的commit, 因此，如果repo已经push到了远程仓库时，操作要慎重
