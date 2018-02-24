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

示例：<文件名>: [@github-id] [翻译中|翻译完成] [审校人] [未审校|审校中|审校完成] (英文源文件commit hash)  

1. index.rst: @hongbinzuo 翻译完成 未审校 （01ba8b7e1fbdcb1b26820fd7b8908e43fb367e82）
2. structure-of-a-contract.rst: @bobjiang 翻译中 (f58024b9744f557dbc77d5f7bfbc4319bde2e0c7)


## 贡献者列表

1. 姜信宝 
2. 左洪斌

## 常见问题
1. fork出的仓库如何同步源的内容：https://www.zhihu.com/question/28676261
2. 如何检查文件(参考[Solidity英文文档仓库](https://github.com/ethereum/solidity))的提交哈希：`git log filename.rst`

## 术语表
此表需要单独文件给出