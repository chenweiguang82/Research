# VASP声子谱计算
* 优化结构
* 扩胞
* 计算
* 处理
## 处理
1. 执行phonopy --fc vasprun.xml
2. 创建band.conf文件
```
ATOM_NAME = Ge
DIM = 1 1 1
FORCE_CONSTANTS = READ
BAND = 0.5 0.25 0.75  0.5 0.5 0.5  0.0 0.0 0.0  0.5 0.0 0.5  0.5 0.25 0.75  0.375 0.375 0.75
```
3. 生成声子谱
```
phonopy -p -s band.conf
```
4. 输出声子谱数据
