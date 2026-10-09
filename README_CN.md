# CellEvoTree

CellEvoTree 是用于重建单细胞调控状态树的 R 包。表达数据负责确定主要的
细胞状态邻近关系，regulon 与功能模块用于受约束的局部优化，拟时序和 RNA
velocity 用于定根与方向判断，基因、转录因子和模块用于解释姐妹分支差异。

该树表示转录状态之间的层次关系，不代表基于突变或克隆信息的遗传谱系。

## 安装

将下面的 `YOUR_GITHUB_USERNAME` 替换为实际 GitHub 用户名或组织名：

```r
install.packages("pak")
pak::pak("YOUR_GITHUB_USERNAME/CellEvoTree")
```

也可以安装 GitHub Release 中的源码包：

```r
install.packages("CellEvoTree_0.1.0.tar.gz", repos = NULL, type = "source")
```

## 仓库内容

- `R/`：包函数源码。
- `man/`：函数帮助文档。
- `tests/testthat/`：自动测试。
- `inst/docs/`：中文教程、方法说明和验证说明。
- `inst/examples/`：运行模板。
- `paper/figures/`：论文图件。
- `paper/source_data/`：能够公开且体积较小的绘图源数据。
- `paper/scripts/`：论文分析脚本快照，运行前需按说明配置数据路径。

真实 Seurat 对象、SCENIC 大矩阵、RNA velocity 对象、FASTQ/BAM 文件和
未投稿 Word 稿不进入 GitHub。大鼠损伤队列的公开编号为 GEO GSE305964；
其他数据来源见 `paper/sample_metadata/`。

## 上传前

先完成 `GITHUB_SETUP_REQUIRED.md` 中的作者、邮箱、ORCID、GitHub 用户名和
版权信息，再运行：

```powershell
powershell -ExecutionPolicy Bypass -File tools/preflight_github.ps1
```
