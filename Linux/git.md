### rebase与merge区别
rebase含义：将当前分支相对于目标分支的变化，在目标分支上重放一遍，只改变当前分支  
merge含义：将目标分支和当前分支代码合并，形成新的提交，也是只改变当前分支  
### 换行配置
默认情况下，Windows上使用CRLF作为换行，提交时会将其改为LF，签出时会替换为CRLF  
`git config --global core.autocrlf true`  
当该选项为input时，提交时把CRLF转换成LF，签出时不转换，为false时也是不作转换  
**Linux上bash脚本，使用CRLF会报错，必须用LF**  
### 恢复
从暂存区恢复指定文件  
`git checkout -- 文件名`  
从已提交的commit恢复指定文件  
`git checkout HEAD  文件名`  