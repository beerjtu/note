创建虚拟环境： conda create -n py36django python=3.6

conda env --help    #查看帮助
conda env list  #列出所有的虚拟环境
conda list --name [虚拟环境名]   #查看指定虚拟环境下的package
#创建
conda create --name [虚拟环境名] [python的版本] [需要的包]
eg:
conda create --name myenv
conda create --name myenv python=2.7
conda create --name myenv pytohon=2.7 numpy scipy


#克隆
conda create --name [虚拟环境名] -- clone [colne的环境]
eg:
#创建一个和原python环境一样的虚拟环境
conda create --name mybase --clone base

#删除
conda remove --name [虚拟环境名] -all


#激活取消（默认的环境是base）
activate [虚拟环境名]
deactivate [虚拟环境名]
