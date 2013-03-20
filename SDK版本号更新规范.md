# SDK 版本号更新规范

## 版本号

版本号命名规则：`<struct>.<feature>.<patch>`

- 结构性大改动，`<struct>` +1，`<feature>` 和 `<patch>` 归零
- 功能调整，`<feature>` +1，`<struct>` 不变，`<patch>` 归零
- 修复bug，`<patch>` +1，`<struct>` 和 `<feature>` 不变

## 更新日志

需同步更新 CHANGELOG.md，注明SDK新版本所更新的内容。