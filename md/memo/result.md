

## 1. コンティグ数（断片化の程度）

* **Raven：28**
* **Flye：33**
* **Shasta：36**
* **Canu：14**

> 少ないほど断片化が少なく、一連の大きな配列になっていることを示します。
> → **Canu** が最少（14本）で最も断片化が少ない

---

## 2. 最大コンティグ長（最大連続配列の長さ）

* **Raven：5.8 Mb**
* **Flye：5.0 Mb**
* **Shasta：12.0 Mb**
* **Canu：8.7 Mb**

> 長いほど大規模な染色体断片を一塊で組み立てられたことを示す。
> → **Shasta** が断トツで最大（12 Mb）

---

## 3. 総アセンブリ長（網羅性／過剰・欠損のチェック）

* **Raven：35.5 Mb**
* **Flye：34.3 Mb**
* **Shasta：33.5 Mb**
* **Canu：34.6 Mb**

> 既知のゲノム長（約33.8 Mb）に近いほど“ちょうど良い”網羅性。

* Raven はややオーバー（冗長 or 重複の可能性）
* Shasta は若干短め（欠落のリスク）
* Flye・Canu は ±1% 程度で良好

---

## 4. N50（連続性の代表値）

* **Raven：2.7 Mb**
* **Flye：2.8 Mb**
* **Shasta：6.7 Mb**
* **Canu：3.6 Mb**

> N50 が大きいほど「上位の長いコンティグだけでゲノム長の 50% を占める」＝高い連続性。
> → **Shasta** が最も優秀（6.7 Mb）

---

## 5. GC含有率（塩基組成の妥当性）

* Raven：22.8%
* Flye：23.0%
* Shasta：22.9%
* Canu：23.1%

> 既知の GC ≒ 23% とほぼ一致。いずれも問題なし。

---

## 6. プロットから読み取る傾向

* **Nxプロット**（x％の累積長を担うコンティグ長 vs x％）

  * Shasta ≫ Canu > Flye > Raven の順で曲線が高い → Shasta の連続性が突出
* **累積長プロット**（コンティグインデックス vs 累積長）

  * Shasta・Canu は少数の長いコンティグで急峻に立ち上がり、早期に総長に到達

---

## 総合的な選び方

1. **最高の連続性（大きな断片が欲しい）** → **Shasta**
2. **染色体本数に近い少数のコンティグかつ網羅性重視** → **Canu**
3. **冗長・重複を避けつつ最大限網羅したい** → **Flye**
4. **最大限の網羅性（多少冗長でも全領域をカバー）** → **Raven**

---

**結論**

* contiguity（連続性）最優先 → **Shasta**
* contig数＋誤差少なめのバランス型 → **Canu**

