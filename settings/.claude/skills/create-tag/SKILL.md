---
name: 建立 tag
description: '建立並推送 git tag：計算下一個版號、列出將被包含的 commit、確認後才建立。Triggers: "打tag", "建tag", "打個tag", "tag一下", "建版", "打版", "發版本", "create tag", "出版號"'
argument-hint: '[版號]'
allowed-tools: Bash(git *)
---

<!-- 範本檔案。<...> 標示處請替換成團隊實際值，替換完刪掉本行。 -->

## 目前狀態

- 目前分支：!`git branch --show-current`
- 最近 10 個 tag：!`git tag --sort=-v:refname | head -10`
- 工作區狀態：!`git status --short`
- 最新的 commit：!`git log --oneline -15`

---

# 建立 git tag

**這個 skill 有不可逆的副作用（推送 tag 到遠端）。以下每個確認點都不可略過。**

## 1. 確認來源分支

**明確詢問要從哪個分支打 tag，不自行代入上方顯示的目前分支**——熱修可能從 `hotfix/*`、正式版從 `main`。

若確認的分支不是目前分支，先檢查是否已同步：

```bash
git fetch origin
git log origin/{分支}..{分支} --oneline
```

## 2. 檢查工作區

上方「工作區狀態」若非空白，先告知使用者有未提交的變更，並確認是否繼續。

## 3. 決定版號

依序嘗試：

1. 叫用時帶的參數：`$1`
2. 從上方 tag 清單推算（預設 patch +1，例：`v1.4.2` → `v1.4.3`）

**推算結果一律是建議值，必須讓使用者確認或改寫**——minor 還是 patch 由人決定。

## 4. 列出這個 tag 會包含什麼

```bash
git log {上一個 tag}..{來源分支} --oneline
git diff {上一個 tag}..{來源分支} --stat
```

把結果完整呈現給使用者。

若 commit 數為 0，明確告知「與上一個 tag 沒有差異」並詢問是否仍要建立。

## 5. 預覽並確認

完整呈現：

```
來源分支：{分支}
新版號：  {版號}
上一個：  {上一個 tag}
包含：    {n} 個 commit
```

詢問：「確認建立並推送？」**得到明確肯定才進行下一步。**

## 6. 建立並推送

```bash
git tag {版號} {來源分支}
git push origin {版號}
```

完成後回報 tag 名稱與 commit SHA。

---

## 貫穿全程的規則

以下規則在整個流程中持續有效：

- **步驟 1、3、5 三個確認點不可略過。** 即使使用者一開始就說「打個 v1.4.3」，
  仍要走完步驟 4 的內容確認再推送
- 推送後就不可逆。若使用者反悔，說明需要 `git push --delete origin {版號}`，
  但**不要自己執行**——刪 tag 是另一個需要確認的破壞性操作
- 不要修改任何檔案，這個 skill 只負責標記

