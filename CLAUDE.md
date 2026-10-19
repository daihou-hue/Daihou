# 贷后管理看板 — 给 Claude 的项目说明

用户是银行对公客户经理，不懂编程，用中文沟通。回答要通俗，改完直接告诉用户结果和怎么用。

## 现在在用的是哪个版本
- **在用：Claude 私密看板** https://claude.ai/artifact/BRownHGVgzLjb75GixrL8W
  - 源码：`artifact/dashboard.html`（单文件，无依赖）。改完用 Artifact 工具发布到上面这个 URL（传 `url`，先 read 再发布），不要新建另一个。
  - 能力：`db`（云端数据）+ `downloads`（导出 Excel）。
- 旧版：根目录 `index.html` 是早期 GitHub Pages 版（数据存手机浏览器 localStorage），已停用，不再维护。

## 数据（在看板的 db 里，用 ArtifactData 工具读写）
- `clients/{id}`：`name`, `order`, `credit{limit 万元, start, end, guarantee, renew 0未启动/1准备材料/2已上报/3已批复}`
- `loans/{id}`：借据 `clientId, amount 万元, start, end, method, plan(''|repay 还本转贷|norepay 无还本续贷|close 到期结清|report 重新上报授信), note, closed`
- `records/{2026Q4_客户id}`：季度进度 `visit{s 0未开始/1已约/2已完成, date, note}`, `loan{s 0正常/1关注/2有风险, note}`, `report{s 0未写/1草稿/2已提交}`
- `tasks/{id}`：待办 `clientId, title, due, note, done`
- `settings/main`：`loanWarn`(30) `creditWarn`(90) `approvalDays`(60) `exceedMonths`(0)
- 写已有文档要先读拿 version（if_version）。用户发来授信批复/借据截图时，录入对应字段。

## 业务规则（用户确认过的）
- 两层期限：授信期限 + 单笔（借据）期限；默认单笔到期日不能超过授信到期日。
- 转贷两种：还本转贷（用户常做）和无还本续贷；关注类/有风险客户不能直接转贷，需上报。
- 到期判断逻辑在 `adviseLoan()` / `creditAlert()`；季末前 15 天未完成访问或报告的客户标红置顶。

## 重要约束
- **仓库是公开的：绝不能把真实客户名称或任何客户数据写进代码、提交或 commit 信息。** 客户数据只放在看板 db 里。
- 改功能后先用 Playwright + 模拟 `window.claude`（db/downloads）在手机尺寸下测一遍再发布。
