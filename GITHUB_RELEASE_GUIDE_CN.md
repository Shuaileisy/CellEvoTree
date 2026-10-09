# CellEvoTree 0.1.0 GitHub 发布步骤

## 1. 先补齐身份信息

先打开 `GITHUB_SETUP_REQUIRED.md`，再按 `RELEASE_CHECKLIST.md` 修改
`DESCRIPTION`、`LICENSE`、`LICENSE.md`、`CITATION.cff` 和
`.zenodo.json.template`。作者、维护者邮箱、GitHub 用户名和版权归属不能
使用占位信息公开发布。

## 2. 创建空 GitHub 仓库

仓库名建议为 `CellEvoTree`。因为本地已经有 README 和 LICENSE，创建
远程仓库时不要让 GitHub 自动生成这些文件。

## 3. 初始化与首次提交

```powershell
cd F:\codex\CellEvoTree_GitHub_Ready_20260923\CellEvoTree

git init
git branch -M main
git add .
git status
git diff --cached
git commit -m "Initial release of CellEvoTree 0.1.0"
```

## 4. 连接并推送远程仓库

```powershell
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/CellEvoTree.git
git push -u origin main
```

## 5. 创建版本标签

```powershell
git tag -a v0.1.0 -m "CellEvoTree 0.1.0"
git push origin v0.1.0
```

## 6. 从 GitHub 安装

```r
install.packages("pak")
pak::pak("YOUR_GITHUB_USERNAME/CellEvoTree")
```

或者：

```r
install.packages("remotes")
remotes::install_github("YOUR_GITHUB_USERNAME/CellEvoTree")
```

## 7. 发布原则

- 不提交真实 Seurat、SCENIC、RNA velocity 或 RData 对象。
- 不提交本机绝对路径、密码、token 或 `.Renviron`。
- 真实研究结果和软件源代码使用独立目录或独立数据仓库。
- CellEvoTree 输出应称为 regulatory cell-state tree，不应声称为遗传
  系统发育树或克隆谱系树。
