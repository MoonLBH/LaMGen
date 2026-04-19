# LaMGen ligand / protein 输入机制（代码导向）

本文件基于当前仓库代码路径梳理训练、推理与分子重建流程，重点覆盖：

- `scripts/train_dual.py`, `scripts/train_triple.py`
- `scripts/gen_dual.py`, `scripts/gen_triple.py`
- `model/lamgen_model.py`
- `utils/smi_torsion_2_molobj.py`
- `scripts/calc_property.py`

结论要点：

1. 训练输入来自 CSV 的 `smiles` + `geos` + target ID，protein 侧读取的是本地 `.npy` ESM-C embedding，而非运行时从序列推理。  
2. ligand 侧在模型里是一个统一 token 序列（SMILES 字符 + `GEO` + torsion 数值 token），模型做 next-token 自回归。  
3. 多靶点融合方式是“分别将每个 protein embedding 与 ligand hidden 拼接后做同一个 `CrossSelfAttention`，再把多路结果相加回残差”，未见独立 `TriCoupleAttention` 类实现。  
4. 推理输出也是 token 序列；3D 结构重建在后处理脚本里通过 `SMILES + torsion` 调 RDKit 构象并设置二面角实现。  

