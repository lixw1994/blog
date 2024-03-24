---
draft: false
comments: true
slug: golang-gitlab-private-repo
date: 2021-06-18
authors: [lixw1994]
title: Golang 处理 Gitlab 私有仓库依赖
description: >
  Test Blog Comments
categories:
  - General
  - Tech
tags:
  - Go
  - Golang
  - Gitlab
---

# Golang 处理 Gitlab 私有仓库依赖

1. 在gitlab设置中创建一个`access token`，只读权限即可，命名为 `go-get` 。
<!-- more -->
2. 修改本地`~/.gitconfig`
	```
	[url "git@gitlab.com:"]
		insteadOf = https://gitlab.com
	[http]
		extraheader = PRIVATE-TOKEN:YOUR-GO-GET-TOKEN
	```
3. 创建`~/.netrc`
	```
	machine gitlab.com
		login gitlab-user
		password xxxxxxxxxxxxxxxxxx
	```
