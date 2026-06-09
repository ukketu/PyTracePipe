# GPDCと「独立性検定が因果を表すとき確率モデルになる理由」

作成日: 2026-06-09

## このノートの目的

このノートでは、PCMCI論文で出てくる **GPDC**、つまり **Gaussian Process Distance Correlation** を数式ベースで説明します。

さらに、ユーザーの疑問である、

> 独立性検定が因果を表すとき、なぜ確率モデルの話になるのか？

も整理します。

結論を先に言うと、因果探索で独立性検定を使う理由は、

> 因果構造モデルが観測データの同時確率分布 $P(\mathbf{X})$ に条件付き独立関係を刻み込むから

です。

ただし、独立性検定だけで因果が証明されるわけではありません。因果Markov条件、Faithfulness、因果十分性、時間順序などの仮定によって、確率分布上の独立性を因果グラフに読み替えます。

関連ノート:

- [[条件付き独立性検定_厳密解説|条件付き独立性検定：厳密解説]]
- [[CMI_GPDC_ParCorr|CMI / GPDC / ParCorr]]
- [[FullCI_数式ベース厳密解説|FullCI：数式ベースの厳密解説]]
- [[PCアルゴリズム]]
- [[偏相関と検出力]]
- [[../01. 論文読解/03_式から読む|PCMCI 式から読む]]

---

# Part A. GPDCの厳密解説

## 1. GPDCは何をする検定か

GPDCは、条件付き独立性

$$
X \perp Y \mid Z
$$

を検定するための方法です。

ただし、CMIのように $p(x,y,z)$ を直接推定するのではなく、次の2段階で検定します。

```text
Step 1. X と Y から Z で説明できる非線形成分を取り除く
Step 2. 残差同士が独立かを distance correlation で調べる
```

つまり、GPDCは **残差ベースの条件付き独立性検定** です。

---

## 2. GPDCが仮定するモデル

GPDCの中心仮定は、$Z$ への依存が加法的に分離できることです。

代表的には、次のようなモデルを考えます。

$$
X = f_X(Z) + \varepsilon_X
$$

$$
Y = f_Y(Z) + \varepsilon_Y
$$

ここで、

| 記号 | 意味 |
|---|---|
| $X$ | 条件付き独立性を調べたい一方の変数 |
| $Y$ | 条件付き独立性を調べたいもう一方の変数 |
| $Z$ | 条件づける変数、ふつうは多変量 |
| $f_X$ | $Z$ から $X$ を予測する非線形関数 |
| $f_Y$ | $Z$ から $Y$ を予測する非線形関数 |
| $\varepsilon_X$ | $Z$ では説明できない $X$ の残差、ノイズ |
| $\varepsilon_Y$ | $Z$ では説明できない $Y$ の残差、ノイズ |

このモデルのもとで、もし

$$
\varepsilon_X \perp \varepsilon_Y
$$

なら、$Z$ で説明できる部分を取り除いたあと、$X$ と $Y$ には依存が残らないと考えます。

したがって、GPDCでは近似的に、

$$
X \perp Y \mid Z
\quad \Longleftrightarrow \quad
\varepsilon_X \perp \varepsilon_Y
$$

という読み替えをします。

重要なのは、この同値は無条件に成り立つわけではないことです。加法ノイズ構造が崩れると、残差独立性だけでは条件付き独立性を完全には表せません。

---

## 3. ParCorrとの違い

ParCorrは、線形回帰で残差化します。

$$
X = a_X^\top Z + r_X
$$

$$
Y = a_Y^\top Z + r_Y
$$

そして、

$$
\operatorname{corr}(r_X,r_Y)=0
$$

かどうかを見ます。

GPDCでは、この線形回帰をGaussian Process回帰に置き換えます。

$$
X = f_X(Z) + r_X
$$

$$
Y = f_Y(Z) + r_Y
$$

したがって、ParCorrよりも柔軟に、非線形な $Z$ の影響を取り除けます。

---

## 4. Gaussian Process回帰とは何か

Gaussian Process、つまりGPは、関数そのものに確率分布を置く非パラメトリック回帰です。

通常の回帰では、たとえば

$$
f(z)=a^\top z
$$

のように関数形を決めます。

一方、GP回帰では、関数 $f$ がガウス過程に従うと仮定します。

$$
f \sim \mathcal{GP}(m, k)
$$

ここで、

| 記号 | 意味 |
|---|---|
| $m(z)$ | 平均関数 |
| $k(z,z')$ | カーネル関数、2点 $z,z'$ の関数値の共分散 |
| $\mathcal{GP}$ | Gaussian Process |

これは、任意の有限個の入力 $z_1,\ldots,z_n$ に対して、

$$
\left(f(z_1),\ldots,f(z_n)\right)
$$

が多変量正規分布に従う、という意味です。

$$
\left(f(z_1),\ldots,f(z_n)\right)
\sim
\mathcal{N}(m, K)
$$

ただし、

$$
K_{ab}=k(z_a,z_b)
$$

です。

GPDCでは、このGP回帰を使って、$Z$ から $X$ と $Y$ をそれぞれ非線形に予測します。

---

## 5. GPDCのStep 1：GP回帰で残差を作る

観測データを

$$
\{(x_t,y_t,z_t)\}_{t=1}^{n}
$$

とします。

まず、$X$ を $Z$ でGP回帰します。

$$
\hat{f}_X = \operatorname{GPRegression}(Z,X)
$$

そして残差を作ります。

$$
\hat{\varepsilon}_{X,t}=x_t-\hat{f}_X(z_t)
$$

同様に、

$$
\hat{f}_Y = \operatorname{GPRegression}(Z,Y)
$$

$$
\hat{\varepsilon}_{Y,t}=y_t-\hat{f}_Y(z_t)
$$

を作ります。

この残差は、

> $Z$ から説明できる非線形成分を取り除いたあとの $X,Y$

です。

---

## 6. GPDCのStep 2：残差をuniformizeする

PCMCI論文の説明では、GPDCはGP回帰後、残差を **uniform marginals** に変換してからdistance correlationを計算します。

これは、残差の周辺分布を一様分布にそろえる操作です。

経験分布関数を使って、残差 $\hat{\varepsilon}_{X,t}$ を

$$
\tilde{u}_{X,t}
=
\hat{F}_X(\hat{\varepsilon}_{X,t})
$$

に変換します。

同様に、

$$
\tilde{u}_{Y,t}
=
\hat{F}_Y(\hat{\varepsilon}_{Y,t})
$$

です。

ここで、

| 記号 | 意味 |
|---|---|
| $\hat{F}_X$ | $X$ 側残差の経験分布関数 |
| $\hat{F}_Y$ | $Y$ 側残差の経験分布関数 |
| $\tilde{u}_{X,t}$ | 一様化された $X$ 側残差 |
| $\tilde{u}_{Y,t}$ | 一様化された $Y$ 側残差 |

確率積分変換により、連続分布なら

$$
F_R(R) \sim \operatorname{Uniform}(0,1)
$$

です。

この変換によって、周辺分布の形の違いに検定が引きずられにくくなります。

---

## 7. GPDCのStep 3：distance correlationを計算する

GPDCの最後のステップは、一様化された残差

$$
\tilde{U}_X = (\tilde{u}_{X,1},\ldots,\tilde{u}_{X,n})
$$

と

$$
\tilde{U}_Y = (\tilde{u}_{Y,1},\ldots,\tilde{u}_{Y,n})
$$

の独立性をdistance correlationで検定することです。

距離相関は、線形相関では見えない非線形依存も検出できます。

母集団レベルでは、distance correlationは

$$
\operatorname{dCor}(X,Y)=0
\quad \Longleftrightarrow \quad
X \perp Y
$$

という性質を持ちます。

---

## 8. distance covarianceの標本定義

観測値 $x_1,
\ldots,x_n$ と $y_1,\ldots,y_n$ を考えます。

まず、距離行列を作ります。

$$
a_{jk}=\lVert x_j-x_k\rVert
$$

$$
b_{jk}=\lVert y_j-y_k\rVert
$$

次に、二重中心化します。

$$
A_{jk}=a_{jk}-\bar{a}_{j\cdot}-\bar{a}_{\cdot k}+\bar{a}_{\cdot\cdot}
$$

$$
B_{jk}=b_{jk}-\bar{b}_{j\cdot}-\bar{b}_{\cdot k}+\bar{b}_{\cdot\cdot}
$$

ここで、

$$
\bar{a}_{j\cdot}=\frac{1}{n}\sum_{k=1}^{n}a_{jk}
$$

$$
\bar{a}_{\cdot k}=\frac{1}{n}\sum_{j=1}^{n}a_{jk}
$$

$$
\bar{a}_{\cdot\cdot}=\frac{1}{n^2}\sum_{j=1}^{n}\sum_{k=1}^{n}a_{jk}
$$

です。$B$ についても同様です。

標本distance covarianceは、

$$
\operatorname{dCov}^2(X,Y)
=
\frac{1}{n^2}\sum_{j=1}^{n}\sum_{k=1}^{n}A_{jk}B_{jk}
$$

です。

distance varianceは、

$$
\operatorname{dVar}^2(X)
=
\operatorname{dCov}^2(X,X)
$$

$$
\operatorname{dVar}^2(Y)
=
\operatorname{dCov}^2(Y,Y)
$$

です。

そして、distance correlationは、概念的には

$$
\operatorname{dCor}(X,Y)
=
\frac{\operatorname{dCov}(X,Y)}
{\sqrt{\operatorname{dVar}(X)\operatorname{dVar}(Y)}}
$$

です。

PCMCI論文のSupplementaryでは、GPDCの距離相関はこのdistance covariance / varianceに基づいて定義されています。

---

## 9. GPDCの帰無仮説

GPDCの条件付き独立性検定では、本来の帰無仮説は

$$
H_0: X \perp Y \mid Z
$$

です。

GPDCでは、加法ノイズ仮定のもとでこれを残差独立性に変換し、

$$
H_0^{\operatorname{GPDC}}:
\tilde{U}_X \perp \tilde{U}_Y
$$

をdistance correlationで検定します。

つまり、実際に検定しているのは、

$$
\operatorname{dCor}(\tilde{U}_X,\tilde{U}_Y)=0
$$

かどうかです。

$p$値が小さければ、

$$
\operatorname{dCor}(\tilde{U}_X,\tilde{U}_Y)>0
$$

と判断し、

$$
X \not\perp Y \mid Z
$$

の証拠とします。

---

## 10. GPDCのアルゴリズム

```text
入力:
  サンプル {(x_t, y_t, z_t)}_{t=1}^n
  有意水準 alpha

1. X を Z でGP回帰する
   x_t = f_X(z_t) + residual_X_t

2. Y を Z でGP回帰する
   y_t = f_Y(z_t) + residual_Y_t

3. 残差を経験分布関数で一様化する
   u_X_t = F_hat_X(residual_X_t)
   u_Y_t = F_hat_Y(residual_Y_t)

4. u_X と u_Y のdistance correlationを計算する

5. independence nullのもとでp値を計算する

6. p値 < alpha なら X not independent Y | Z と判断する
```

PCMCIの文脈では、この $X,Y,Z$ に、たとえばMCIなら

$$
X = X^i_{t-\tau}
$$

$$
Y = X^j_t
$$

$$
Z=
\left(\mathcal{P}(X^j_t)\setminus\{X^i_{t-\tau}\}\right)
\cup
\mathcal{P}(X^i_{t-\tau})
$$

を入れます。

---

## 11. GPDCが得意なケース

GPDCが得意なのは、$Z$ の影響が非線形だが、加法的に分離できる場合です。

たとえば、

$$
X = Z^2 + \varepsilon_X
$$

$$
Y = -Z^2 + \varepsilon_Y
$$

のようなケースです。

このとき、$X$ と $Y$ は無条件では強く関係します。どちらも $Z^2$ に依存するからです。

しかし、$Z$ を非線形に考慮すれば、残差は

$$
\varepsilon_X
$$

と

$$
\varepsilon_Y
$$

になります。

もしノイズ同士が独立なら、

$$
\varepsilon_X \perp \varepsilon_Y
$$

なので、GPDCは

$$
X \perp Y \mid Z
$$

を検出できます。

ParCorrは線形にしか $Z$ の影響を取り除けないため、このような二次関数的な依存を取り除くのが苦手です。

---

## 12. GPDCが苦手なケース

GPDCは加法ノイズ型の残差化に依存します。

そのため、次のような乗法的構造では失敗しやすくなります。

$$
X = Z\varepsilon_X
$$

$$
Y = -Z\varepsilon_Y
$$

この場合、$Z$ への依存が単純な

$$
f(Z)+\varepsilon
$$

の形ではありません。

残差化しても、$Z$ による分散構造や分布形の変化が残ります。

そのため、残差同士の無条件独立性を調べるだけでは、真の条件付き独立性をうまく表せないことがあります。

このような場合は、CMIのようなより一般的な非パラメトリック条件付き独立性検定の方が原理的には適しています。

---

## 13. GPDCとCMIの違い

| 方法 | 基本発想 | 仮定 | 得意 | 弱点 |
|---|---|---|---|---|
| ParCorr | 線形回帰で残差化し、残差相関を見る | 線形・ガウス的 | 線形関係、小サンプル | 非線形に弱い |
| GPDC | GP回帰で非線形残差化し、残差のdistance correlationを見る | 加法ノイズ | 滑らかな非線形加法関係 | 非加法・高次元に弱い |
| CMI | $I(X;Y\mid Z)$ を直接推定 | 少ない構造仮定 | 一般的な非線形依存 | 推定が難しく、サンプルが多く必要 |

GPDCは、ParCorrとCMIの中間のような位置づけです。

- ParCorrより柔軟。
- CMIより仮定が強い。
- 仮定が合えば、CMIより検出力が高くなることがある。
- ただし高次元条件集合ではGP回帰もdistance correlationも不安定になる。

---

# Part B. 独立性検定が因果を表すとき、なぜ確率モデルになるのか

## 14. 観測データは確率変数の実現値だから

因果探索で扱うデータは、通常、確定的な表ではなく、確率変数の実現値として扱います。

たとえば、時系列なら

$$
\mathbf{X}_t = (X^1_t,\ldots,X^N_t)
$$

は確率過程です。

観測データ

$$
\{\mathbf{x}_1,\ldots,\mathbf{x}_T\}
$$

は、その確率過程から得られた1つのサンプルです。

したがって、独立性や条件付き独立性は、データ表そのものではなく、背後にある分布

$$
P(\mathbf{X})
$$

の性質として定義されます。

たとえば、

$$
X \perp Y \mid Z
$$

とは、分布が

$$
p(x,y\mid z)=p(x\mid z)p(y\mid z)
$$

を満たすという意味です。

つまり、独立性検定は最初から確率モデル上の仮説検定です。

---

## 15. 因果モデルは分布を生成するモデルだから

構造的因果モデル、SCMでは、各変数を次のような構造方程式で表します。

$$
X_j = f_j(\operatorname{PA}_j, \varepsilon_j)
$$

ここで、

| 記号 | 意味 |
|---|---|
| $X_j$ | 変数 $j$ |
| $\operatorname{PA}_j$ | $X_j$ の親、直接原因 |
| $f_j$ | 生成関数 |
| $\varepsilon_j$ | 外生ノイズ |

時系列なら、

$$
X^j_t = f_j\left(\mathcal{P}(X^j_t), \varepsilon^j_t\right)
$$

と書けます。

ここでノイズ

$$
\varepsilon^1_t,\ldots,\varepsilon^N_t
$$

に確率分布を置くと、変数全体の同時分布

$$
P(\mathbf{X})
$$

が誘導されます。

つまり因果モデルは、単なる矢印の絵ではなく、

> どの変数がどの変数とノイズから生成されるかを表す確率的生成モデル

です。

だから、因果と確率分布は切り離せません。

---

## 16. 因果グラフは同時分布の分解を与える

DAGの因果モデルでは、同時分布は親集合によって分解されます。

$$
p(x_1,\ldots,x_N)
=
\prod_{j=1}^{N}p(x_j\mid \operatorname{pa}_j)
$$

これは、各変数 $X_j$ は、親 $\operatorname{PA}_j$ を与えれば、それ以外の非子孫とは独立になる、という意味です。

時系列グラフでも同様に、各時刻の変数は過去の親によって生成されると考えます。

$$
p(\mathbf{x}_{1:T})
=
\prod_{t=1}^{T}\prod_{j=1}^{N}
p\left(x^j_t \mid \mathcal{P}(X^j_t)\right)
$$

定常性などを仮定すると、同じ構造が時刻をまたいで繰り返されます。

この分解こそが、因果グラフと確率モデルを結びつけます。

---

## 17. 因果Markov条件：グラフから独立性が出る

因果Markov条件は、ざっくり言うと、

> 各変数は、自分の直接原因を知れば、それ以外の非子孫とは独立になる

という仮定です。

数式で書くと、DAGでは概念的に

$$
X_j \perp \operatorname{NonDescendants}(X_j)
\mid \operatorname{PA}_j
$$

です。

この仮定により、因果グラフから条件付き独立性が導かれます。

例:

```text
X -> Z -> Y
```

このグラフでは、$Z$ を条件づけると経路が遮断されるので、

$$
X \perp Y \mid Z
$$

が期待されます。

つまり、因果構造は確率分布上の条件付き独立性として現れます。

---

## 18. Faithfulness：独立性からグラフを読むための逆向き仮定

因果Markov条件は、

```text
グラフ上で分離される
  ↓
確率分布上で条件付き独立になる
```

という向きの仮定です。

しかし、因果探索では逆に、データから

$$
X \perp Y \mid Z
$$

を見つけて、グラフ構造を推定したいです。

そのためには、Faithfulnessが必要です。

Faithfulnessは、ざっくり言うと、

> 観測される条件付き独立性は、グラフ構造による分離から生じている。係数の偶然の相殺で独立に見えているわけではない。

という仮定です。

例として、

```text
X -> Y
X -> Z -> Y
```

があるとします。

このとき、$X$ から $Y$ への直接効果と、$X \to Z \to Y$ の間接効果が偶然打ち消し合うと、データ上は

$$
X \perp Y
$$

のように見えるかもしれません。

Faithfulnessは、このような偶然の相殺を基本的に除外します。

---

## 19. 独立性検定が因果に変換される論理

独立性検定から因果構造を読む論理は、次の流れです。

```text
1. 因果構造モデルを仮定する
   X_j = f_j(PA_j, epsilon_j)

2. ノイズに確率分布を置く
   これにより同時分布 P(X) が生じる

3. 因果Markov条件により
   グラフの分離 => 条件付き独立

4. Faithfulnessにより
   条件付き独立 => グラフの分離

5. 観測データから条件付き独立性を検定する

6. 検定結果をグラフの辺の有無に読み替える
```

数式で極端に短く書けば、

$$
\text{causal graph } G
\Rightarrow
P(\mathbf{X})
\Rightarrow
\{X \perp Y \mid Z\}
$$

であり、Faithfulnessのもとで、

$$
\{X \perp Y \mid Z\}
\Rightarrow
\text{constraints on } G
$$

と読み戻します。

だから、独立性検定が因果を表すときは、必ず確率モデルの話になります。

---

## 20. PCMCIではどうなるか

PCMCIでは、時系列構造因果モデルを考えます。

$$
X^j_t = f_j\left(\mathcal{P}(X^j_t),\varepsilon^j_t\right)
$$

このモデルが、時系列全体の確率分布を生成します。

候補リンク

$$
X^i_{t-\tau} \to X^j_t
$$

がないなら、適切な親集合で条件づけたとき、

$$
X^i_{t-\tau} \perp X^j_t
\mid
\mathcal{P}(X^j_t)\setminus\{X^i_{t-\tau}\},
\mathcal{P}(X^i_{t-\tau})
$$

が成り立つはずだ、というのがPCMCIの考え方です。

逆に、この条件付き独立性が棄却されるなら、

$$
X^i_{t-\tau} \not\perp X^j_t
\mid
\mathcal{P}(X^j_t)\setminus\{X^i_{t-\tau}\},
\mathcal{P}(X^i_{t-\tau})
$$

なので、時間順序と因果仮定のもとで、ラグ付き因果リンクの候補として残します。

ここでGPDCは、この条件付き独立性を調べるための検定器の1つです。

---

## 21. 「確率モデルになる」の正確な意味

「独立性検定が因果を表すとき確率モデルになる」というより、より正確には次の順番です。

```text
因果モデルがある
  ↓
外生ノイズがある
  ↓
確率分布 P(X) が誘導される
  ↓
因果グラフの構造が P(X) の条件付き独立性として現れる
  ↓
観測データから P(X) の条件付き独立性を検定する
  ↓
仮定のもとでグラフ構造に読み戻す
```

つまり、独立性検定が因果そのものなのではありません。

独立性検定は、

> 因果モデルが生み出す確率分布の影を観測データから調べる道具

です。

---

## 22. なぜ決定論ではなく確率が必要か

現実のデータでは、同じ原因条件でも常に同じ結果になるとは限りません。

たとえば気候データでは、

$$
\text{BCT}_t = f(\text{Nino}_{t-2}, \text{他の大気海洋状態}) + \varepsilon_t
$$

のように、観測できない要因や測定誤差があります。

この未観測要因を

$$
\varepsilon_t
$$

として確率的に扱います。

すると、因果関係は

> $X$ を知ると $Y$ の分布が変わるか

という形になります。

すなわち、

$$
p(y\mid x,z) \ne p(y\mid z)
$$

なら、$Z$ を調整しても $X$ は $Y$ の分布に追加情報を持ちます。

逆に、

$$
p(y\mid x,z)=p(y\mid z)
$$

なら、$Z$ を知ったあとは $X$ は $Y$ の分布を変えません。

これが条件付き独立性です。

---

## 23. 重要な注意：確率的依存は因果と同じではない

確率モデルを使うからといって、依存がそのまま因果になるわけではありません。

たとえば、

```text
Z -> X
Z -> Y
```

では、

$$
X \not\perp Y
$$

ですが、$X$ が $Y$ の原因とは限りません。

また、隠れ共通原因 $U$ があると、

```text
U -> X
U -> Y
```

なのに、観測上は

$$
X \not\perp Y
$$

が出ます。

したがって、独立性検定を因果として読むには、少なくとも次の仮定が必要です。

| 仮定 | 役割 |
|---|---|
| 因果十分性 | 重要な共通原因が観測されているとみなす |
| 因果Markov条件 | グラフ構造から条件付き独立性を導く |
| Faithfulness | 条件付き独立性からグラフ構造を読み戻す |
| 時間順序 | 過去から未来への向きを制約する |
| 定常性 | 時刻をまたいで同じ構造が繰り返されるとみなす |
| 正しい最大ラグ | 必要な過去変数が候補集合に含まれる |

---

## 24. GPDCと確率モデルの接点

GPDCも確率モデルの上にあります。

まず、条件付き独立性

$$
X \perp Y \mid Z
$$

は、分布上の性質です。

$$
p(x,y\mid z)=p(x\mid z)p(y\mid z)
$$

GPDCは、この分布を直接推定せず、加法ノイズモデル

$$
X=f_X(Z)+\varepsilon_X
$$

$$
Y=f_Y(Z)+\varepsilon_Y
$$

を仮定し、

$$
\varepsilon_X \perp \varepsilon_Y
$$

かどうかを調べます。

そして $f_X,f_Y$ の推定にGaussian Processという確率的回帰モデルを使います。

つまりGPDCは、

```text
条件付き独立性という分布上の仮説
  ↓
加法ノイズモデルで残差独立性に変換
  ↓
GP回帰で残差を推定
  ↓
distance correlationで残差独立性を検定
```

という方法です。

---

## 25. まとめ

GPDCは、条件付き独立性

$$
X \perp Y \mid Z
$$

を直接密度推定するのではなく、

$$
X=f_X(Z)+\varepsilon_X
$$

$$
Y=f_Y(Z)+\varepsilon_Y
$$

という加法ノイズ構造を仮定して、GP回帰で $Z$ の影響を取り除き、残差同士のdistance correlationを検定する方法です。

独立性検定が因果探索で使える理由は、因果モデルが観測変数の同時確率分布を生成し、その因果グラフの構造が条件付き独立性として分布に現れるからです。

ただし、

$$
X \not\perp Y \mid Z
$$

は、それだけで因果を証明するものではありません。

因果として読むには、因果Markov条件、Faithfulness、因果十分性、時間順序、定常性などの仮定が必要です。

PCMCIでは、時間順序で向きを制約し、条件付き独立性検定でラグ付きリンクの有無を判定します。GPDCはその検定器の一つです。

---

## 出典・確認元

- Runge, J. et al. (2019). `Detecting causal associations in large nonlinear time series datasets`. Science Advances. arXiv:1702.07007. https://arxiv.org/abs/1702.07007
- Supplementary Section S2.2 GPDC: GPDC is described as GP regression followed by distance correlation testing on uniformized residuals.
- Supplementary Section S3.2 Consistency: PCMCI consistency relies on causal sufficiency, causal Markov condition, faithfulness, stationarity, and consistent conditional independence tests.
- Székely et al. distance correlation: distance correlation is zero iff the variables are independent at the population level.

---

## 関連ノート

- [[条件付き独立性検定_厳密解説|条件付き独立性検定：厳密解説]]
- [[CMI_GPDC_ParCorr|CMI / GPDC / ParCorr]]
- [[FullCI_数式ベース厳密解説|FullCI：数式ベースの厳密解説]]
- [[PCアルゴリズム]]
- [[偏相関と検出力]]
- [[次元の呪い]]
- [[../01. 論文読解/03_式から読む|PCMCI 式から読む]]
