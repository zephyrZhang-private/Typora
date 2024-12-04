# premise

* 安装Git

> https://git-scm.com/downloads
>
> 启动Git：git bash here

* 创建rsa密钥

> 进入ssh：cd ~/.ssh（找不到文件夹手动创建即可）
>
> 创建密钥：ssh-keygen.exe -t rsa -C "19932726377@163.com"

* 添加认证

> * GitHub仓库settings
> * SSH and GPG keys
> * New SSH key
> * 复制~/.ssh生成的id.rsa_pub密钥文件添加到key中创建

* Git测试

> ssh -T git@github.com

* 添加用户、邮箱

> git config --global user.name "zephyrZhang" 【尽力与GitHub一致】
>
> git config --global user.email "19932726399@163.com"
