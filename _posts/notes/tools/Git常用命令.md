# Git常用命令

## 一、标签 tag

### 打标签
	git tag -a v1.1.4 -m "tagging version 1.1.4"
### 推送标签
	git push origin <tagName>
### 删除本地标签
	git tag -d <tagName>
### 删除远程标签
	git push origin :refs/tags/<tagName>
### 列出标签
	git tag


## 二、分支 branch

### 建立新分支
	git checkout -b v1.1.0-stable
### 推送分支
	git push origin v1.1.0-stable
### 删除本地分支
	git branch -d v1.1.0-stable
### 删除远程分支
	git push origin :v1.1.0-stable
