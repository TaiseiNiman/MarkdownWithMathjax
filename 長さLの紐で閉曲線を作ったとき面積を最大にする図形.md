## 長さLの紐で閉曲線を作ったとき面積を最大にする図形
長さLの紐を用意して、いろんな平面上の輪を作ったとしよう.
このとき,どのような輪がその面積を最大にさせるのかを考えたことはないだろうか.
例えば,正方形なら一辺の長さは$\frac{L}{4}$だから,その面積は$\frac{L^2}{16}$となる.
一方,円なら円周の長さがLということなので,その面積は$(\frac{L}{2\pi})^2\times \pi = \frac{L^2}{4\pi}$となる.
これは円の面積が正方形の面積より$\frac{4}{\pi}$だけ大きいことを意味している.
周の長さが一定の元で面積を最大にする閉曲線を求める問題は,
典型的な制約付き変分問題として考えることが出来る.
変分問題として定式化すると以下のようになる.
$$
G[r,\theta] = \int_{0}^{2\pi} \frac{r(\theta+d\theta)r(\theta)\sin{d\theta}}{2}= \int_{0}^{2\pi} \frac{\left(r(\theta)+\frac{\mathrm{d}r}{\mathrm{d}\theta}d\theta\right)r(\theta)d\theta}{2} = \int_{0}^{2\pi} \frac{r^2(\theta)}{2}d\theta \tag{1} 
$$
$$
\int_{0}^{2\pi} r(\theta)d\theta = l \tag{2}
$$
式(2)の下で,式(1)を最大にする$\theta$の関数$r(\theta)$を求めればよい.
[解答]
まず式(1),(2)をラグラジアン$L[r,\lambda,\theta]$に置き換える.すると,
$$
L[r,\lambda,\theta] = G[r,\theta] - \lambda\left(\int_{0}^{2\pi} r(\theta)d\theta - l\right) \tag{3}
$$
Lの変分を取ると,
$$
\delta L = \int_{0}^{2\pi} r(\theta)\delta r d\theta - \int_{0}^{2\pi} \lambda \delta r d\theta - \delta\lambda\left(\int_{0}^{2\pi} r(\theta)d\theta - l\right) = \int_{0}^{2\pi} \left(r(\theta)-\lambda\right)\delta r d\theta = 0 \tag{4}
$$
を得る.これより,
$$
r(\theta) = \lambda \tag{5}
$$
式(5)より$r(\theta)$は定数だから,式(2)の$r(\theta)$を積分の外に置くと,
$$
r(\theta) = \frac{l}{2\pi} \tag{6}
$$
を得る.式(6)は周の長さが$l$のときの円の方程式に他ならない.
よって,一本の紐を用いて輪を作るとき、面積を最大にする平面図形は円である.
