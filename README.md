# Solidity中文官方文档 

## 工作指南

### Github使用基础

参见廖雪峰的[Git和Github指南](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001373962845513aefd77a99f4145f0a2c7a7ca057e7570000)

### RST文档基础

RST文档格式的一个[中文说明](http://www.cnblogs.com/seayxu/p/5603876.html)

### 工作流

1. 注册Github帐号
2. 申请加入[Solidity翻译团队](https://github.com/orgs/etherchina/teams/solidity-translation-team/)
3. fork仓库: https://github.com/etherchina/solidity-doc-cn
4. 在README中更新，认领需要翻译的章节，请务必加入commit hash值，命令见常见问题。
5. 翻译，完成后提交PR
6. admin审核之后，由校对人员校对
7. 确认质量后由admin合并入master

## 工作进度

1. index.rst: 翻译完成，未校对 （commit 01ba8b7e1fbdcb1b26820fd7b8908e43fb367e82）


## 翻译认领

1. index.rst: 左洪斌
2. ...


## 贡献者列表

1. 姜信宝 
2. 左洪斌

## 常见问题
1. fork出的仓库如何同步源的内容：https://www.zhihu.com/question/28676261
2. 如何检查文件的提交哈希：`git log filename.rst`
3. 文档如何构建：https://solidity-cn.rtfd.io 是我们的托管地址，readthedocument这个网站是免费的，可以关联多个仓库，并且可以由git push触发自动构建，以达到文档更新的目的。
4. 原英文文档更新怎么办：我们需要人去定期检查英文文档的更新，使用commit hash来比较更新，如果有更新，我们发起翻译请求，翻译后提交。后续我们会开发跟踪脚本，每天检查文档文件的更新，以确定是否有新的翻译工作。

## 术语表
此表需要单独文件给出