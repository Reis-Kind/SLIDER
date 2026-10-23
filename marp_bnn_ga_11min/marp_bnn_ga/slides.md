---
marp: true
theme: default
size: 16:9
paginate: true
lang: ja
style: |
  section {
    background: white;
    color: black;
    font-family: "Yu Gothic", "Noto Sans CJK JP", sans-serif;
    font-size: 28px;
    padding: 40px 58px;
  }
  section h1 {
    font-size: 40px;
    color: black;
    margin-bottom: 24px;
  }
  section h3 {
    font-size: 29px;
    color: black;
    margin: 24px 0 8px;
  }
  section strong {
    color: #c00000;
  }
  section p {
    margin: 12px 0;
  }
  section li {
    margin: 10px 0;
  }
  section img {
    object-fit: contain;
    max-width: 100%;
  }
  section > p:has(img) {
    text-align: center;
  }
  section table {
    display: table;
    width: 85%;
    margin: 24px auto;
    font-size: 34px;
  }
  section th,
  section td {
    padding: 12px 28px;
    color: black;
    background: white;
    text-align: center;
  }
  section.title {
    text-align: center;
  }
  section.title h1 {
    font-size: 43px;
    line-height: 1.5;
  }
  section.title p {
    margin-top: 36px;
  }
  section::after {
    color: black;
    font-size: 18px;
  }
---

<!-- _class: title -->

# 遺伝的アルゴリズムを用いた<br>2値化重みを持つ<br>畳み込みニューラルネットワークの学習

米子工業高等専門学校  
廣芳大二朗　内田雅人

---

# 畳み込みニューラルネットワーク（CNN）

![w:1130](images/cnn.png)

---

# 研究背景・目的

### CNNの軽量化
重みを **±1** に限定し，保存量と演算を削減

### 2値重みの学習
BinaryConnectは，実数重みを保持して勾配で更新

### 本研究の目的
**GAによる直接探索**と，**BP学習後の追加探索**を検証

---

# BPによる学習：BinaryConnect

### ① 順伝播
実数重みを **±1に2値化**して，分類結果と損失を計算

### ② 逆伝播・更新
近似した勾配で **実数重みを更新**

### ③ 次の計算
更新した実数重みを再び2値化し，①を繰り返す

実数重みに小さな更新を蓄積し，符号の変化につなげる

---

# GA：個体の表現と評価

![w:1130](images/ga_evaluation.png)

<!-- 図中の正答率は説明用の例 -->

---

# GA：交叉

![w:1130](images/crossover.png)

---

# GA：突然変異

![w:1130](images/mutation.png)

---

# 実験方法

同じネットワーク構造で，**3手法**を比較

### BPのみ
BinaryConnectで重みを学習

### GAのみ
ランダムな2値重みから探索

### BP＋GA
BPで学習した重みを出発点に，GAで追加探索

---

# ネットワーク構成

![h:380](images/network.png)

畳み込み3層・全結合1層の重みを2値化  
探索対象：**113,504個の2値重み**

---

# データの分割とモデル選択

CIFAR-10：10クラスの画像分類

### 学習用　45,000枚
BPの重み更新・GAの個体評価に使用

### モデル選択用　5,000枚
正答率が最も高いモデルを採用

### テスト用　10,000枚
採用したモデルの性能を比較

---

# 学習条件

**BP**：学習率 0.001，300エポック

**GA共通**：500体，上位50体を保存，突然変異率 0.0001

- GAのみ：ランダム500体から **10,000世代**
- BP＋GA：BP重み1体＋変異449体＋ランダム50体から **1,000世代**

### GAの適応度は正答率
世代ごとに学習用から **2,000枚** を抽出  
同じ世代の全個体を，同じ画像で評価

---

# 実験結果

| 手法 | テスト正答率 |
| :---: | :---: |
| BPのみ | 71.14% |
| GAのみ | 60.20% |
| **BP＋GA** | **72.41%** |

**BP＋GAは，BPのみより1.27ポイント高い**

単一試行の結果であり，再現性は今後検証する

---

# 学習の推移：BP

![h:420](images/bp_accuracy.png)

学習用は約83%まで上昇，テストは約70%で停滞

---

# 学習の推移：GAのみ

![h:420](images/ga_accuracy.png)

学習用・テストとも緩やかに上昇（約66%，約60%）

---

# 学習の推移：BP＋GA

![h:420](images/bp_ga_accuracy.png)

序盤にテスト正答率が向上し，その後は停滞・緩やかに低下

---

# 考察

### BP＋GAの改善
学習済み重みにも，GAで改善できる組合せがあった

### 後半の停滞・低下
評価する2,000枚の違いや，学習画像への適応が影響した可能性

### 今後確かめること
複数試行での再現性と，追加計算時間に対する改善幅

局所解からの脱出や，低下の原因は未確認

---

# まとめ

### GAのみ
2値重みCNNを学習できたが，BPのみには及ばなかった

### BP＋GA
今回の試行では，BPのみから **1.27ポイント改善**

### 今後の課題
- 乱数シードを変えて再現性を検証
- 評価枚数・交叉方法・個体群の多様性を検討