# GitHub 网页上传指南

> 用途：不用本地 git，直接把本工程上传到 GitHub 远程仓库。

目标仓库：

```text
https://github.com/lmyykxy/lumen_llmcollab_hub
```

---

## 1. 上传步骤

1. 下载本 zip。
2. 在本地解压。
3. 打开 GitHub 仓库页面。
4. 点击 `Add file`。
5. 点击 `Upload files`。
6. 将解压后的文件夹内容拖入网页。
7. 填写 commit message：

```text
Initialize Lumen LLM collaboration hub
```

8. 点击 `Commit changes`。

---

## 2. 注意事项

不要上传 zip 本身作为仓库内容。  
要上传 zip 解压后的这些文件和目录：

```text
AGENTS.md
README.md
CHANGELOG.md
CODEX_HANDOFF_PROMPT.md
GITHUB_WEB_UPLOAD_GUIDE.md
roles/
docs/
.github/
.gitignore
```

---

## 3. 后续维护

小修改可以用：

```text
github.dev/lmyykxy/lumen_llmcollab_hub
```

或者在仓库页面按键盘：

```text
.
```

进入网页版 VS Code 编辑。

---

## 4. 与 Codex 配合

仓库上传后，切到 Codex 新对话，把 `CODEX_HANDOFF_PROMPT.md` 的内容贴给 Codex。
