ipad+unity+pythonによるinstant-ngp実装
ipadのARカメラを用いて,リアルタイムに学習させ3D再構成を行う.
まず理論について述べる.
nerfでは内部パラメータを固定し,入力として外部パラメータ、出力として静止画のみを学習データとして用いる.
外部パラメータは,カメラの位置とカメラの視線情報を持ち,
これらを入力として,各点の放射輝度場と密度を予測し,これらをもとにボリュームレンダリングを行うことで、静止画を生成する.
生成された静止画と学習データとして与えられた静止画の差分を最小二乗法によって最適化することで,各層の重み係数を決定し,任意の位置と視線からの静止画を再構成するものである.
次に,どのようにして実装するか説明する.
まず,被写体をコンピュータビジョンの技術を用いて自動的に識別したり,手動で人力で静止画を選別する方法を取らない.
空間上の同一の点には同一の物体が時間的に変化せず存在し,かつどの視線からでも遮るものがないことを仮定した上で,
カメラが同一の位置にある物体を静止画として写すときは,常に同じ物体であることを仮定する.
この仮定はいずれ,非剛体物体や時間変化する場合を考慮することになるので取り払うことにはなるが,今はこの仮定の元で実装することにする.
そこで大事になるのは,カメラの外部パラメータからどの位置の物体を写しているかを判断することである。
ここでは,式を簡単にするためにカメラのロールつまり静止画を回転するような回転を無視して,
カメラの視線ベクトルとカメラの位置のみを対象とする.
カメラの水平方向の視野角をh,垂直方向の視野角をvとする.
まずカメラの視野角と被写体との深度距離zから,学習データとして用いる静止画数を決定する.
単に半径zの球面を考えて,カメラの画像面が切り取るその球面を切り取る表面積S_0を考えればよい.
球面の面積をこのS_0で割ったものを必要な静止画数Nとすればよいが,重なりがあればあるほど精度が上がるので,精度係数aを乗じることにする.
つまり,
$$
N = \frac{4\pi z^2}{S_0}a = \frac{4\pi z^2}{2z\tan{h}\cdot 2z\tan{v}}a = \frac{\pi}{\tan{h}\tan{v}}a  \tag{1}
$$
これは,空間の中心にカメラがある場合の話で,極端に壁側にいたりするとこの式は成り立たない.
ここでより厳密に考えると,半径zの球面の各点は, $(\theta,\phi,z)$ で与えられるから,その面積Sが,
$$
S = \int_{0}^{2\pi}\int_{-\pi/2}^{\pi/2} \left\|\begin{pmatrix} -zd\theta\sin{\phi}\sin{\theta} \\ zd\theta\sin{\phi}\cos{\theta} \\ 0 \end{pmatrix} \times \begin{pmatrix} zd\phi\cos{\phi}\cos{\theta} \\ zd\phi\cos{\phi}\sin{\theta} \\ -zd\phi\sin{\phi} \end{pmatrix}\right\| = 
$$
$$
 \int_{0}^{2\pi}\int_{-\pi/2}^{\pi/2}\left\|\begin{pmatrix} -z^2d\theta d\phi \sin^2{\phi}\cos{\theta} \\ -z^2d\theta d\phi \sin^2{\phi}\sin{\theta} \\ -z^2d\theta d\phi \cos{\phi}\sin{\phi} \end{pmatrix} \right\| =  \int_{0}^{2\pi}\int_{-\pi/2}^{\pi/2}z^2d\theta d\phi |\sin{\phi}| = z^2\cdot2\pi \cdot 2 = 4\pi z^2 \tag{2}
$$
より,厳密にNは以下のように求められる.
$$
N = \frac{4a\pi z^2}{\int_{0}^{2h}\int_{-v}^{v}z^2d\theta d\phi |\sin{\phi}|} = \frac{4a\pi z^2}{4z^2h\left(1-\cos{v}\right)} = \frac{a\pi}{h\left(1-\cos{v}\right)}, 0 \leqq h,v \leqq \frac{\pi}{2} \tag{3} 
$$
ここで,周の長さを常に同じにするように $\theta$ 方向にx分割と $\phi$ 方向にy分割することを考えると,
$$
y=\frac{x}{2} \tag{4}
$$
これよりx,y分割したときの単位球面を切り取る表面積S_0は,
$$
S_0 = \int_{0}^{\frac{2\pi}{x}}\int_{-\pi/x}^{\pi/x}z^2d\theta d\phi |\sin{\phi}| = z^2\cdot \frac{2\pi}{x}\cdot 2\left(1-\cos{\frac{\pi}{x}}\right) \tag{4}
$$
半径zの球面の表面積Sを式(4)で割った値が式(3)に等しければよいから,
$$
\frac{a\pi}{h\left(1-\cos{v}\right)} = \frac{x}{1-\cos{\frac{\pi}{x}}} \tag{5}
$$
式(5)を満たすようにxとyを決定すれば,最適な分割数が求められる.
より適切には,カメラの水平視野角と垂直視野角の比率も考慮してx,yの分割数を定めるべきだが,簡単にするために無視した.
次に,カメラの視線ベクトルをクォータニオンから求めて,視線ベクトルから回転角 $\theta,\phi$ を求める方法を述べる.
ipadのARkitあるいはipadのARカメラをunityから操作するとき,カメラの回転角は,以下のクォータニオン表示で精度よく得ることが出来る.
$$
q = w+ix+jy+kz \tag{6} 
$$
$i,j,k$ はそれぞれ四元数の虚数単位.
四元数の回転公式に従って,z軸を回転させることで以下の視線ベクトルz'が得られる.
ただし,z軸の初期値はARカメラが起動した直後のカメラの視線方向の負の向き.
$$
z' = q(-k)q* \tag{7}
$$
$q*$ は $q$ の共役四元数
次に外部パラメータから得られたカメラの視線ベクトルを回転角 $\theta,\phi$ に分解する.
単位球面上の各点は球面座標系で $(\theta,\phi,1)$ として与えられるが,
これをxyz座標座標系で考えることで,以下の関係を得る
$$
\begin{pmatrix} x \\ y \\ z \end{pmatrix} = \begin{pmatrix} \sin{\phi}\cos{\theta} \\ \sin{\phi}\sin{\theta} \\ \cos{\phi}\end{pmatrix} \tag{8}
$$
式(8)のz座標から,ただちに
$$
\phi = \arccos{z} \tag{9}
$$
式(8)のx座標,y座標から,
$$
\theta = \arccos{\left( \frac{x}{\sin{\phi}}\right)} = \arcsin{\left( \frac{y}{\sin{\phi}}\right)} \tag{10}
$$
あとは分割数に従って, $(\theta\phi)$平面をN個のゾーンに分けて,各ゾーンごとの静止画を得られるまでカメラをユーザが動かすようにすればよい.
視覚的にわかりやすくするために,平面ではなく単位球面をipadのモニターにHUDとして表示して,
静止画の得られたゾーンは色を変えてタスクが完了していることを示す.
