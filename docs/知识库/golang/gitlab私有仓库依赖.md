1. 在gitlab设置中创建一个`access token`，只读权限即可，命名为go-get
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