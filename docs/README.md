# docs/

这个目录放「给人看的东西」，不放会被 GitHub 主页渲染的内容。

## 私有材料（不进版本库）

`docs/private/` 已在 `.gitignore` 里排除，放简历、成绩单、证书原件、个人资料都安全：

```
docs/private/resume.pdf
docs/private/certificates/...
```

把简历 PDF 放进 `docs/private/` 之后告诉我一声，我会读它，并且**不会**提交到仓库。

## 公开文档

如果将来要放对外的说明（例如 `docs/certificates.md`，主页通过链接指向它），
请直接放在 `docs/` 根下 —— 这些是会被提交的。

## 规划产物

- `research/` —— 调研笔记（已有引用来源，会提交）
- `research/profile-spec.md` —— 主页规格（区块、组件、设计系统、验收标准）
