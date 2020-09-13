翻译流程
==================

本项目进行的翻译遵循以下原则：

- 尊重原文档的结构、格式
- 采用译、校、审流程
- 保持对上游文档的更新

## 译者加入流程

1. 添加Tiny微信 ：xlbxiong
2. 进行试译流程，就是翻译一个段落，以 1000 字为限，时间限制为 1 天。
3. 如果通过就进入 solidity 翻译小组，进入翻译流程。


## 流程说明

翻译流程主要分为以下几个环节：
- 未认领，[当前状态](status_cn.md)
- 翻译，包括译者初译、修改等工作
	- 选择尚未被认领的工单，备注认领信息，记得跟新[状态](status_cn.md)
	- 进行翻译和自我校对……
	- 将翻译成果通过 PR（包含文档的commit hash） 的方式提交到仓库，等待校对意见
- 校对，对译者完成的翻译进行初步校对，主要涉及：
	- 版式检查，链接检查，用语检查，内容错乱
  如果初校过程中发现译文问题较多，可驳回要求译者修改，并指出重点问题。
- 完成，对进行过校对的文章整体审阅，符合项目质量、风格要求的译文，提交进入 develop 分支管理员发布，记得跟新[状态](status_cn.md)


## 翻译方法

### 拉取官方文档
翻译项目为 [Solidity 官方文档](https://github.com/ethereum/solidity/) 0.6.4 版本，获取官方文档0.6.4 版本的方法为：

```
git clone git@github.com:ethereum/solidity.git
git checkout v0.6.4 -b v0.6.4  # 检出 v0.6.4版本
cd docs   # docs 为文档所在目录
```

这里有一份[Git和Github指南](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001373962845513aefd77a99f4145f0a2c7a7ca057e7570000)

### 拉取翻译文档

```
git clone git@github.com:lbc-team/solidity-doc-cn.git
```

由于文档是sphinx 生成，最好安装下：

```
sudo pip install -U Sphinx
pip install sphinx_rtd_theme
pip install pygments-lexer-solidity
pip install sphinx-a4doc

make html

```

之后就可以打编辑器对照英文0.6.4版本进行翻译，翻译完成后提交pull request。

RST文档格式的一个[中文说明](http://www.cnblogs.com/seayxu/p/5603876.html)

### 术语库
文档一个术语库，用于统一文档翻译中的术语使用。

术语库内的术语使用 RAW 格式的 HTML 5 RUBY 标签，鉴于主流浏览器和微信都支持它。格式如下：

```
.. |glossary| raw:: html
  <ruby>术语<rp>（</rp><rt>glossary</rt><rp>）</rp></ruby>
```

而其它页面则在首部采用 `.. include :: glossary.rst` 标签统一引入术语库，并在使用该术语处采用 `|glossary|` 来引用该术语。


## 常见问题

0. [中文文案排版指北](https://github.com/mzlogin/chinese-copywriting-guidelines)
1. fork出的仓库如何同步源的内容：https://www.zhihu.com/question/28676261
2. 如何检查文件(参考[Solidity英文文档仓库](https://github.com/ethereum/solidity))的提交哈希：`git log filename.rst`
3. 文档如何构建：目前文档托管在[登链社区](https://learnblockchain.cn/docs/solidity/)的CDN上，在管理员校对后，自动发布。
4. 原英文文档更新怎么办：我们需要人去 tag 对比，查看更新内容，进行补充。


