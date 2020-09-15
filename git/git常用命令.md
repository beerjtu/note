* 添加远程地址： git remote add origin <远程地址>
* 查看远程地址： git remote -v
* 生成ssh key: ssh-keygen -t rsa -C "youremail@example.com"
   > 生成后在用户文件夹下的.ssh文件中，复制id_rsa.pub中的内容，放到github的settings->SSH and GPG keys的SHH keys中
* 本地仓库和远程仓库文件冲突，需要强行进行合并，方法是执行: git pull --rebase origin master