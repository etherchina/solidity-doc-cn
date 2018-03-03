# Solidity中文官方文档 

## 工作指南

### Github使用基础

参见廖雪峰的[Git和Github指南](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001373962845513aefd77a99f4145f0a2c7a7ca057e7570000)

### RST文档基础

RST文档格式的一个[中文说明](http://www.cnblogs.com/seayxu/p/5603876.html)

### 工作流

1. 注册Github帐号
2. 加入[Solidity翻译团队](https://github.com/orgs/etherchina/teams/solidity-translation-team/)
3. fork仓库: https://github.com/etherchina/solidity-doc-cn
4. develop: admin负责README更新，记录需要翻译的章节，请务必加入commit hash值，命令见常见问题。
5. develop: 译者翻译，完成后提交PR
6. develop: 在提交PR或git push的时候注意：一定要先同步主仓库源，见常见问题
7. develop: 一名Reviewer审核之后, 有问题退回修改，提交后另一名Reviewer再审核
8. master: 确认质量后由admin合并入master

详细说明：第4步及后续步骤关于翻译工作的认领，请遵循如下流程：管理员把译者加入Github上的翻译团队，翻译的内容在Github上issue上跟踪，可由管理员或译者开issue，可以在管理员开的issue下回复认领翻译工作，然后管理员更新Readme（注意commit hash），译者开始翻译，翻译初稿完成后提交PR，提交PR时注明对应的issue＃，然后进入两轮审校，译者修改并最终审校完成后提交到develop，由管理员提交PR合并到master分支

## 工作进度

示例：<文件名>: [@github-id] [翻译中|翻译完成] [审校人] [未审校|审校中|审校完成] (英文源文件commit hash)  

1. `index.rst`: @hongbinzuo 翻译完成 @toyab 审校中 （01ba8b7e1fbdcb1b26820fd7b8908e43fb367e82）
2. `introduction-to-smart-contracts.rst`: @oldcodeoberyn @ysqi 翻译完成 @hongbinzuo 审校完成 @toyab 审校中 (f58024b9744f557dbc77d5f7bfbc4319bde2e0c7)
3. `structure-of-a-contract.rst`: @bobjiang 翻译中 (f58024b9744f557dbc77d5f7bfbc4319bde2e0c7)
4. `frequently-asked-questions.rst`: @ghostrd 翻译中（5770458826bfe72201393f4f02729a970bac926e） 
5. `abi-spec.rst`: @riversyang 翻译中（5770458826bfe72201393f4f02729a970bac926e）

## 贡献者列表

1. 姜信宝 
2. 左洪斌
3. 侯伯薇
4. toyab
5. 李捷
6. 虞是乎ysqi
7. 周锷
8. 杨镇

## 常见问题
1. fork出的仓库如何同步源的内容：https://www.zhihu.com/question/28676261
2. 如何检查文件(参考[Solidity英文文档仓库](https://github.com/ethereum/solidity))的提交哈希：`git log filename.rst`
3. 文档如何构建：https://solidity-cn.rtfd.io 是我们的托管地址，readthedocument这个网站是免费的，可以关联多个仓库，并且可以由git push触发自动构建，以达到文档更新的目的。
4. 原英文文档更新怎么办：我们需要人去定期检查英文文档的更新，使用commit hash来比较更新，如果有更新，我们发起翻译请求，翻译后提交。后续我们会开发跟踪脚本，每天检查文档文件的更新，以确定是否有新的翻译工作。
5. rst文档中中文斜体无法展示，此问题尚待解决

## 术语表
参见glossaries.md

## 参考资料
参见references.md
