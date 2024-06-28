# 贡献指南

在提交贡献之前，请务必花点时间阅读下面的项目贡献指南。这将让你能更快速的参与到项目中，同时也感谢您的贡献！

## 透明化开发流程

所有的开发提交都直接透明地在 GitHub 上进行，确保代码质量的同时也便于追溯每次的提交变动。
`FEHHair` 团队成员和外部贡献者的 PR 都会经过相同的流程。

- 需保证 PR 与最新主分支保持同步
- 通过 CI/CD 流程后才能合并
- 两人以上的 Reviewer 审核通过后才能合并

> 克隆-切分支-开发-提交-创建PR-Review(代码审查)-合并

## 报告问题

我们使用 [Github issues](https://github.com/FEHHair/mojo/issues) 采集问题 bug 和新建议 feature。

⚠️ 注意在提 issues 之前，请确保搜索过类似的问题，因为它们可能已经得到解答或正在被修复。

## 开发

1. Fork [此仓库](https://github.com/FEHHair/mojo.git)
2.  `pnpm install` 安装依赖
3.  `pnpm run dev` 启动项目
4. 从 `main` 创建分支进行开发。分支命名：`类型/开发功能内容`例：`feature/add-login-page`

## 提交 PR

1. 确认代码正常运行且能通过各项检查

2. 提交 git commit, 请同时遵守 [Commit 规范](#commit-指南)
```bash
git add .
git commit -m 'feat: 新增xxxx功能'
```

3. 发布分支
```bash
git push origin xxx分支
```
4.  提交 pull request, 如果有对应的 issue，请进行[关联](https://docs.github.com/zh/get-started/writing-on-github/working-with-advanced-formatting/autolinked-references-and-urls#issues-and-pull-requests)。
5.  等待 CI/CD 流程完成，等待 Reviewer 审核。

### Commit 指南

本项目遵循[conventional-changelog 标准](https://www.conventionalcommits.org/en/v1.0.0/)    

以下是 commit 类型列表:
* feat: 新功能、新特性
* fix: 修改 bug
* perf: 更改代码，性能优化
* refactor: 代码重构（在不影响代码内部行为、功能下的代码修改）
* docs: MD 文档相关修改
* style: 代码格式修改, 注意不是 css 样式修改（例如分号修改）
* test: 测试用例新增、修改
* build: 影响项目构建或依赖项修改
* revert: 恢复上一次提交
* ci: 持续集成相关文件修改
* release: 发布新版本
* workflow: 工作流相关文件修改
* chore: 其他修改（不在上述类型中的修改）

详见[commitlint.config.js](./commitlint.config.js)文件  

⚠️ 我们的每次发版都将根据 commit 自动化生成 changelog 更新日志故需严格遵循以上规范。
## License

[MIT 协议](./LICENSE)
