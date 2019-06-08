# git简介
## 基本命令
* 初始化git并添加一个readme：
```git
echo "# XXXX.github.io" >> README.md
git init
git add README.md
```
* 提交至本地仓库
```git
git commit -m "first commit"
```
* 配置git账户
git config --global user.email "XXXX@163.com"
git config --global user.name "XXXX"

* 添加远端仓库
```git
git commit -m "first commit"
git remote add origin https://github.com/XXXX/XXXX.github.io.git
```
* 推送至远端仓库
```git
git push -u origin master
```
* 克隆远端仓库
```git
git clone https://github.com/XXXX/XXXX.github.io
```
* 添加所有本目录下的文件以后续提交至本地仓库，推送至远端
```git
cd XXXX.github.io/
git add --all
git commit -m "Initial commit"
git push -u origin master
```
* 查看发生变化的文件
```git
git status
```
* 添加单个文件（可以是新文件，也可以是发生变化的文件）
```git
git add index.html 
git commit -m "3rd commit"
```
* 默认状态即为推送至远程origin master分支
```git
git push
```
* 查看变化的文件
```git
git diff A B
```

----


## 地址
博客地址：    
[https://huajingze.github.io/](https://huajingze.github.io/)