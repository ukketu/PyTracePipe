# FullCI：数式ベースの厳密解説

作成日: 2026-06-09

## このノートの目的

このノートは、PCMCI論文で比較対象として出てくる **FullCI**、つまり **full conditional independence testing** を、数式ベースで厳密に理解するための解説です。

FullCIは一言でいうと、候補リンク

$$
X^i_{t-\tau} \to X^j_t
$$

が存在するかを、**他のすべての過去変数で条件づけて直接検定する方法**です。

関連ノート:

- [[条件付き独立性]]
- [[CMI_GPDC_ParCorr|CMI / GPDC / ParCorr]]
- [[解説シート_レジーム_CI検定_Granger因果|解説シート：レジーム・CI検定・Granger因果]]
- [[従来の因果構造探索アルゴリズム比較表]]
- [[../01. 論文読解/04_手法の流れ|PCMCI 手法の流れ]]
- [[../03. 課題と発展研究/05_PCMCIへの批判と検証論文|PCMCIへの批判と検証論文]]

---

## 1. FullCIの一言定義

PCMCI論文のSupplementaryでは、FullCIは次の条件付き独立性検定として定義されています。

$$
X^i_{t-\tau} \perp X^j_t
\mid
\mathbf{X}^{-}_t \setminus \{X^i_{t-\tau}\}.
$$

これは日本語でいうと、

> $X^i_{t-\tau}$ 以外の「全変数の過去」をすべて条件に入れたうえで、それでも $X^i_{t-\tau}$ と $X^j_t$ が依存しているかを検定する。

という意味です。

---

## 2. 記号の定義

### 2.1 多変量時系列

$N$個の時系列変数からなる多変量過程を

$$
\mathbf{X}_t = \left(X^1_t, X^2_t, \ldots, X^N_t\right)
$$

とします。

| 記号 | 意味 |
|---|---|
| $t$ | 時刻 |
| $N$ | 観測される時系列変数の数 |
| $X^i_t$ | 時刻 $t$ における変数 $i$ の値 |
| $\mathbf{X}_t$ | 時刻 $t$ の全変数ベクトル |

### 2.2 全過去集合

時刻 $t$ から見た全変数の過去を

$$
\mathbf{X}^{-}_t
=
\left(\mathbf{X}_{t-1}, \mathbf{X}_{t-2}, \ldots\right)
$$

と書きます。

実際のデータ分析では無限の過去は使えないので、最大ラグ $\tau_{\max}$ で打ち切ります。

$$
\mathbf{X}^{-}_{t, \tau_{\max}}
=
\left\{X^k_{t-s}: k=1,\ldots,N,\ s=1,\ldots,\tau_{\max}\right\}.
$$

この集合の大きさは、単純には

$$
\left|\mathbf{X}^{-}_{t, \tau_{\max}}\right|
=
N\tau_{\max}
$$

です。

候補原因 $X^i_{t-\tau}$ 自身は条件集合から除くので、FullCIの条件集合の次元は

$$
N\tau_{\max} - 1
$$

になります。

これがFullCIの最大の問題です。$N$ や $\tau_{\max}$ が大きいと、条件づけ集合がすぐ巨大になります。

---

## 3. FullCIの帰無仮説と対立仮説

候補リンクを

$$
X^i_{t-\tau} \to X^j_t
$$

とします。

FullCIでは、次の帰無仮説を検定します。

$$
H_0:
X^i_{t-\tau} \perp X^j_t
\mid
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}.
$$

対立仮説は、

$$
H_1:
X^i_{t-\tau} \not\perp X^j_t
\mid
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}.
$$

| 仮説 | 意味 |
|---|---|
| $H_0$ | 他のすべての過去を知ると、$X^i_{t-\tau}$ は $X^j_t$ に追加情報を持たない。つまりリンクなし。 |
| $H_1$ | 他のすべての過去を知っても、$X^i_{t-\tau}$ は $X^j_t$ に追加情報を持つ。つまりリンク候補あり。 |

---

## 4. なぜこれで因果リンクを検定できるのか

### 4.1 時系列構造因果モデル

PCMCI論文では、時系列の構造因果過程を大まかに次のように考えます。

$$
X^j_t = f_j\left(\mathcal{P}(X^j_t), \varepsilon^j_t\right).
$$

| 記号 | 意味 |
|---|---|
| $X^j_t$ | 結果側の変数 |
| $f_j$ | 変数 $j$ の生成関数 |
| $\mathcal{P}(X^j_t)$ | $X^j_t$ の親集合、直接原因の集合 |
| $\varepsilon^j_t$ | 外生ノイズ |

候補変数 $X^i_{t-\tau}$ が本当に $X^j_t$ の親なら、

$$
X^i_{t-\tau} \in \mathcal{P}(X^j_t)
$$

です。

逆に親でなければ、

$$
X^i_{t-\tau} \notin \mathcal{P}(X^j_t)
$$

です。

### 4.2 親でない変数は、全過去で条件づければ独立になる

因果Markov条件とFaithfulnessを仮定すると、候補変数 $X^i_{t-\tau}$ が $X^j_t$ の直接原因でない場合、適切な条件集合で条件づけると独立になります。

FullCIは、その「適切な条件集合」として、かなり強引に

$$
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
$$

を使います。

これは、$X^j_t$ に影響しそうな過去変数を全部入れてしまうという発想です。

```text
候補原因以外の過去を全部条件に入れる
  ↓
それでも依存が残るか？
  ↓
残るなら直接リンクの候補
```

---

## 5. 条件付き相互情報量によるFullCI

FullCIは特定の検定方法の名前ではなく、**どの条件集合で条件付き独立性を検定するか**を表す方法です。

CMIを使う場合、FullCIの検定統計量は次のように書けます。

$$
I^{\operatorname{FullCI}}_{i \to j}(\tau)
=
I\left(
X^i_{t-\tau};
X^j_t
\mid
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
\right).
$$

| 記号 | 意味 |
|---|---|
| $I(\cdot;\cdot\mid\cdot)$ | 条件付き相互情報量 |
| $I^{\operatorname{FullCI}}_{i \to j}(\tau)$ | ラグ $\tau$ における $i \to j$ 候補リンクのFullCI効果量 |
| $X^i_{t-\tau}$ | 候補原因 |
| $X^j_t$ | 候補結果 |
| $\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}$ | 候補原因を除く全過去 |

CMIの定義を書くと、一般に

$$
I(X;Y\mid Z)
=
\int p(x,y,z)
\log
\frac{p(x,y\mid z)}{p(x\mid z)p(y\mid z)}
\, dx\, dy\, dz.
$$

FullCIでは、

$$
X = X^i_{t-\tau},
\quad
Y = X^j_t,
\quad
Z = \mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
$$

と置いたものです。

帰無仮説は、

$$
H_0:
I^{\operatorname{FullCI}}_{i \to j}(\tau)=0
$$

です。対立仮説は、

$$
H_1:
I^{\operatorname{FullCI}}_{i \to j}(\tau)>0
$$

です。

---

## 6. ParCorrによるFullCI：線形ガウスの場合

PCMCI論文の気候例では、FullCIを偏相関、つまりParCorrで実装しています。

候補原因を

$$
X = X^i_{t-\tau}
$$

候補結果を

$$
Y = X^j_t
$$

条件集合を

$$
Z = \mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
$$

とします。

ParCorrでは、まず $X$ と $Y$ それぞれから $Z$ で線形に説明できる部分を取り除きます。

$$
X = a_X^\top Z + r_X,
$$

$$
Y = a_Y^\top Z + r_Y.
$$

| 記号 | 意味 |
|---|---|
| $a_X$ | $Z$ から $X$ を予測する回帰係数ベクトル |
| $a_Y$ | $Z$ から $Y$ を予測する回帰係数ベクトル |
| $r_X$ | $Z$ では説明できない $X$ の残差 |
| $r_Y$ | $Z$ では説明できない $Y$ の残差 |

FullCI-ParCorrの効果量は、残差同士の相関です。

$$
\rho^{\operatorname{FullCI}}_{i \to j}(\tau)
=
\operatorname{corr}(r_X,r_Y).
$$

帰無仮説は、

$$
H_0:
\rho^{\operatorname{FullCI}}_{i \to j}(\tau)=0.
$$

サンプルサイズを $n$、条件集合の次元を $k$ とすると、典型的な$t$統計量は

$$
t
=
\rho^{\operatorname{FullCI}}_{i \to j}(\tau)
\sqrt{
\frac{n-k-2}{1-\left(\rho^{\operatorname{FullCI}}_{i \to j}(\tau)\right)^2}
}.
$$

ここでFullCIでは

$$
k = N\tau_{\max}-1
$$

なので、$N$ や $\tau_{\max}$ が大きいと、自由度

$$
n-k-2
$$

が急速に小さくなります。

これが、FullCIの検出力が低下する数式上の理由です。

---

## 7. VARモデルとしてのFullCI

線形時系列モデルでは、FullCIはVARモデルの係数検定としても書けます。

結果変数 $X^j_t$ に対して、最大ラグ $\tau_{\max}$ のVARを考えます。

$$
X^j_t
=
\sum_{k=1}^{N}\sum_{s=1}^{\tau_{\max}}
\beta^{j}_{k,s}X^k_{t-s}
+
\varepsilon^j_t.
$$

| 記号 | 意味 |
|---|---|
| $\beta^{j}_{k,s}$ | 変数 $k$ の $s$ ラグ前が $X^j_t$ に与える線形係数 |
| $\varepsilon^j_t$ | 予測誤差、構造ノイズとは限らない残差 |
| $N\tau_{\max}$ | 説明変数の総数 |

候補リンク

$$
X^i_{t-\tau} \to X^j_t
$$

に対応する係数は

$$
\beta^{j}_{i,\tau}
$$

です。

FullCIの帰無仮説は、線形VARでは

$$
H_0:
\beta^{j}_{i,\tau}=0
$$

と書けます。対立仮説は、

$$
H_1:
\beta^{j}_{i,\tau}\ne 0
$$

です。

この意味で、FullCIは「ラグごとに細かく見たGranger causality検定」と読むこともできます。

ただし、標準的なGranger検定では、しばしば

$$
H_0:
\beta^{j}_{i,1}=\beta^{j}_{i,2}=\cdots=\beta^{j}_{i,\tau_{\max}}=0
$$

のように、変数 $i$ の全ラグをまとめて検定します。

一方、FullCIは

$$
\beta^{j}_{i,\tau}=0
$$

をラグ $\tau$ ごとに見る、よりラグ特異的な検定として理解できます。

---

## 8. FullCIのアルゴリズム手順

FullCIを擬似コードで書くと、次のようになります。

```text
入力:
  多変量時系列 X_t, t=1,...,T
  変数数 N
  最大ラグ tau_max
  条件付き独立性検定 CItest
  有意水準 alpha

for j in 1,...,N:
  for i in 1,...,N:
    for tau in 1,...,tau_max:
      X = X^i_{t-tau}
      Y = X^j_t
      Z = 全過去 {X^k_{t-s}: k=1,...,N, s=1,...,tau_max} から X を除いたもの
      p_value = CItest(X, Y | Z)
      if p_value < alpha:
        X^i_{t-tau} -> X^j_t を採択候補にする
      else:
        リンクなしと判断する
```

候補リンク数は、自己リンクを含めるなら

$$
N^2\tau_{\max}
$$

です。

自己リンクを除くクロスリンクだけなら、

$$
N(N-1)\tau_{\max}
$$

です。

各検定で条件集合の次元はおよそ

$$
N\tau_{\max}-1
$$

です。

したがって、FullCIは

> 多数の候補リンクそれぞれについて、高次元条件付き独立性検定を行う

方法です。

---

## 9. FullCIの理論的な長所

### 9.1 定義に忠実

FullCIは、リンク定義にかなり忠実です。

候補リンク以外の過去をすべて条件づけて、なお依存が残るかを見るからです。

$$
X^i_{t-\tau} \not\perp X^j_t
\mid
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
$$

なら、候補リンクが直接的な情報を持つと解釈しやすいです。

### 9.2 偽陽性を抑えやすい

十分なサンプルがあり、CI検定が正しく働くなら、FullCIは他の過去変数をすべて条件づけるため、間接経路や共通原因による見かけの依存を取り除きやすいです。

たとえば、

```text
X_{t-2} -> Z_{t-1} -> Y_t
```

という間接経路がある場合、$Z_{t-1}$ を条件集合に含めれば、$X_{t-2}$ と $Y_t$ の直接リンクは消えやすくなります。

---

## 10. FullCIの実践的な弱点

### 10.1 条件集合が大きすぎる

FullCIの条件集合は、

$$
Z_{\operatorname{FullCI}}
=
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
$$

です。

次元は、

$$
\dim(Z_{\operatorname{FullCI}})=N\tau_{\max}-1.
$$

たとえば、

| $N$ | $\tau_{\max}$ | 条件集合の次元 |
|---:|---:|---:|
| 5 | 5 | 24 |
| 20 | 5 | 99 |
| 100 | 5 | 499 |
| 100 | 10 | 999 |

サンプルサイズ $T$ が数百程度なら、これはかなり厳しいです。

### 10.2 検出力が落ちる

CI検定の検出力は、一般に次の要因で下がります。

- 条件集合の次元が大きい。
- サンプルサイズが小さい。
- 因果効果が弱い。
- 自己相関が強い。
- ノイズが大きい。
- 非線形CI検定で密度推定や距離推定が不安定。

FullCIは条件集合が最大級に大きいので、特に検出力が落ちやすいです。

論文の数値実験でも、FullCIは $N$ が大きくなると検出力が大きく低下することが示されています。

### 10.3 不要な変数に条件づけて効果量が小さくなる

条件づけは常に良いわけではありません。

不要な変数、特に候補原因や結果と強く相関する変数に条件づけると、残る依存の効果量が小さくなることがあります。

FullCIは「全過去」を入れるため、直接リンクの検定に不要な変数まで条件に入れます。

その結果、FullCIの効果量

$$
I^{\operatorname{FullCI}}_{i \to j}(\tau)
$$

や

$$
\rho^{\operatorname{FullCI}}_{i \to j}(\tau)
$$

が小さくなり、真のリンクを見落としやすくなります。

---

## 11. PCMCIのMCIとの厳密な比較

### 11.1 MCIの条件集合

PCMCIのMCI、つまり momentary conditional independence は、候補リンク

$$
X^i_{t-\tau} \to X^j_t
$$

に対して、主に次のような条件集合を使います。

$$
\mathcal{P}(X^j_t) \setminus \{X^i_{t-\tau}\}
\quad \text{and} \quad
\mathcal{P}(X^i_{t-\tau}).
$$

つまり、MCIの検定は概念的には

$$
X^i_{t-\tau} \perp X^j_t
\mid
\left(\mathcal{P}(X^j_t) \setminus \{X^i_{t-\tau}\}\right),
\mathcal{P}(X^i_{t-\tau})
$$

です。

FullCIは全過去を条件づけますが、MCIはPC1ステップで推定した親集合だけを条件づけます。

### 11.2 条件集合の包含関係

簡略化して、

$$
Z_{\operatorname{MCI}}
=
\left(\mathcal{P}(X^j_t) \setminus \{X^i_{t-\tau}\}\right)
\cup
\mathcal{P}(X^i_{t-\tau})
$$

と書きます。

FullCIの条件集合は、

$$
Z_{\operatorname{FullCI}}
=
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
$$

です。

多くの場合、

$$
Z_{\operatorname{MCI}} \subseteq Z_{\operatorname{FullCI}}
$$

です。

FullCIにだけ含まれる余分な条件変数を

$$
R = Z_{\operatorname{FullCI}} \setminus Z_{\operatorname{MCI}}
$$

と書くと、

$$
Z_{\operatorname{FullCI}} = Z_{\operatorname{MCI}} \cup R.
$$

### 11.3 MCIの効果量はFullCI以上になりやすい

PCMCI論文のSupplementaryでは、条件付き相互情報量を用いて、MCIの効果量がFullCI以上になることが示されています。

概念的には、次の関係です。

$$
I^{\operatorname{MCI}}_{i \to j}(\tau)
\ge
I^{\operatorname{FullCI}}_{i \to j}(\tau).
$$

なぜなら、FullCIはMCIに加えて余分な $R$ まで条件づけるからです。

条件付き相互情報量のチェインルールを使うと、概略として

$$
I(X;Y\mid Z_{\operatorname{MCI}})
=
I(X;Y\mid Z_{\operatorname{MCI}},R)
+
\text{追加項}
$$

のように分解できます。

ここで、

$$
I(X;Y\mid Z_{\operatorname{MCI}},R)
=
I^{\operatorname{FullCI}}_{i \to j}(\tau)
$$

であり、追加項が非負なら、

$$
I^{\operatorname{MCI}}_{i \to j}(\tau)
\ge
I^{\operatorname{FullCI}}_{i \to j}(\tau)
$$

となります。

### 11.4 直感

FullCIは、直接リンクを検定するために、必要以上に多くの変数を条件に入れます。

MCIは、理論上必要な親集合に近いものだけを条件に入れます。

そのため、MCIは

1. 条件集合の次元が小さい。
2. 効果量がFullCIより大きくなりやすい。
3. したがって検出力が高い。

という利点を持ちます。

---

## 12. FullCIとGranger causalityの関係

FullCIは、時系列における条件付き独立性検定なので、Granger causalityと近い関係があります。

標準的な線形Granger causalityでは、

$$
X^i \text{ does not Granger-cause } X^j
$$

を、VARモデルの係数制約として検定します。

$$
H_0:
\beta^j_{i,1}=\beta^j_{i,2}=\cdots=\beta^j_{i,\tau_{\max}}=0.
$$

FullCIは、これをラグごとに分解して、

$$
H_0:
\beta^j_{i,\tau}=0
$$

を検定する方法に近いです。

また、非線形CI検定を使えば、線形VARに限らないGranger的依存も扱えます。

その意味でFullCIは、

> ラグ特異的で、CI検定として一般化されたGranger因果検定

と理解できます。

ただし、Granger causalityと同様に、FullCIの結果を介入的な因果と読むには、因果十分性、定常性、適切な最大ラグ、Faithfulnessなどの仮定が必要です。

---

## 13. FullCIが失敗しやすい具体例

### 13.1 高次元でサンプルが足りない場合

たとえば、

$$
N=50,
\quad
\tau_{\max}=5
$$

なら、条件集合の次元は

$$
50\times 5 - 1 = 249
$$

です。

サンプルサイズが $T=300$ 程度だと、線形回帰でもかなり不安定です。非線形CMIやGPDCならさらに難しくなります。

### 13.2 強い自己相関がある場合

自己相関が強いと、たとえば

$$
X^i_{t-\tau}
$$

と

$$
X^i_{t-\tau-1}
$$

が強く相関します。

FullCIは $X^i_{t-\tau-1}$ なども条件集合に入れるため、候補原因 $X^i_{t-\tau}$ の独自の寄与が小さく見えやすいです。

これは多重共線性に近い問題です。

### 13.3 非線形検定で条件次元が高い場合

CMIやGPDCは非線形依存を検出できますが、条件集合が高次元だと推定が不安定になります。

FullCIでは条件集合が

$$
N\tau_{\max}-1
$$

なので、非線形FullCIは特に厳しくなります。

PCMCI論文でも、GPDCやCMIを用いたFullCIは高次元で性能が悪化することが示されています。

---

## 14. FullCIの正しい位置づけ

FullCIは、理論的には自然で、定義に忠実な方法です。

しかし、実用上は

```text
条件集合が大きすぎる
  ↓
効果量が小さくなる
  ↓
CI検定の検出力が落ちる
  ↓
真のリンクを見落としやすい
```

という問題があります。

そのため、PCMCI論文ではFullCIは主に比較対象として使われます。

PCMCIの主張は、ざっくり言うと、

> FullCIのように全過去で条件づけなくても、親集合をうまく選べば偽陽性を抑えつつ検出力を上げられる。

というものです。

---

## 15. FullCI・MCI・相関の比較

| 方法 | 検定する量 | 条件集合 | 長所 | 弱点 |
|---|---|---|---|---|
| 相関 | $X^i_{t-\tau}$ と $X^j_t$ の無条件依存 | なし | 検出力は高いことがある。単純。 | 間接経路・共通原因・自己相関による偽陽性が多い。 |
| FullCI | $X^i_{t-\tau} \perp X^j_t \mid \mathbf{X}^{-}_t \setminus \{X^i_{t-\tau}\}$ | 候補原因を除く全過去 | 定義に忠実。偽陽性を抑えやすい。 | 条件次元が大きく、検出力が低い。 |
| MCI | $X^i_{t-\tau} \perp X^j_t \mid \mathcal{P}(X^j_t)\setminus\{X^i_{t-\tau}\},\mathcal{P}(X^i_{t-\tau})$ | 推定された親集合 | 条件次元が小さく、効果量が大きい。 | 親集合推定が間違うと影響を受ける。 |

---

## 16. FullCIを読むときのチェックリスト

- [ ] 最大ラグ $\tau_{\max}$ はいくつか。
- [ ] 条件集合の次元 $N\tau_{\max}-1$ はサンプルサイズに対して大きすぎないか。
- [ ] CI検定は ParCorr, GPDC, CMI のどれか。
- [ ] 線形仮定でよいのか、非線形依存がありそうか。
- [ ] 自己相関による効果量低下が起きそうか。
- [ ] FullCIで非検出だったリンクを「因果なし」と断定していないか。
- [ ] PCMCI/MCIとの違いは、条件集合の違いとして説明できるか。

---

## 17. 超短いまとめ

FullCIは、候補リンク

$$
X^i_{t-\tau} \to X^j_t
$$

を検定するために、

$$
X^i_{t-\tau} \perp X^j_t
\mid
\mathbf{X}^{-}_{t,\tau_{\max}} \setminus \{X^i_{t-\tau}\}
$$

を直接調べる方法です。

定義としては素直ですが、条件集合の次元が

$$
N\tau_{\max}-1
$$

まで大きくなるため、高次元・短い時系列・強い自己相関・非線形検定では検出力が大きく落ちます。

PCMCIのMCIは、FullCIの「全部条件づける」という発想をやめて、必要な親集合だけを条件づけることで、より低次元で効果量の大きい検定を目指す方法です。

---

## 18. 出典・確認元

- Runge, J. et al. (2019). `Detecting causal associations in large nonlinear time series datasets`. Science Advances. arXiv:1702.07007. https://arxiv.org/abs/1702.07007
- Supplementary Section `S1.2.1 FullCI`: FullCI is defined as testing $X^i_{t-\tau} \perp X^j_t \mid \mathbf{X}^{-}_t \setminus \{X^i_{t-\tau}\}$.
- Supplementary discussion around Theorem 3: MCI has larger or equal conditional mutual information effect size than FullCI under the stated assumptions.

---

## 関連ノート

- [[条件付き独立性]]
- [[CMI_GPDC_ParCorr|CMI / GPDC / ParCorr]]
- [[偏相関と検出力]]
- [[解説シート_レジーム_CI検定_Granger因果|解説シート：レジーム・CI検定・Granger因果]]
- [[時系列因果探索]]
- [[../01. 論文読解/03_式から読む|PCMCI 式から読む]]
- [[../01. 論文読解/04_手法の流れ|PCMCI 手法の流れ]]
