# Kaggle-Titanic-Machine-Learning-REPO

学校のAIシステム開発授業向けに、タイタニック生存者分析の基本構成を用意したリポジトリです。  
今回の授業では **Train.csv の一部を分割してテスト用（評価用）データ** に使います。  
**Test.csv はモデル評価には使わず、欠損値データの保管用** として扱います。

## ディレクトリ構造

```text
.
├── README.md
├── data
│   ├── raw
│   │   ├── Train.csv            # 元の学習データ
│   │   └── Test.csv             # 受領データ（評価には使わない）
│   ├── split_from_train
│   │   ├── train_split.csv      # Train.csvから作る学習用データ
│   │   └── valid_split.csv      # Train.csvから作る評価用データ
│   └── missing_store
│       └── Test.csv             # 欠損値処理・保管用
├── models                        # 学習済みモデル保存先
└── notebooks                     # 解析ノートブック
```

> 実ファイル（CSV）は授業環境で配置してください。  
> このリポジトリではフォルダ構成のみを先に作成しています。

## データの使い方（授業ルール）

1. `data/raw/Train.csv` を読み込む  
2. Train.csv を学習用と評価用に分割する（例: 80:20）  
3. 学習用データでモデルを学習する  
4. 評価用データでモデル精度を確認する  
5. `data/raw/Test.csv` は評価には使わず、`data/missing_store/Test.csv` として欠損値管理に利用する

## 代表的な特徴量（Feature）解説

- `Pclass`：チケットクラス（1, 2, 3）  
- `Sex`：性別  
- `Age`：年齢（欠損が多く前処理対象）  
- `SibSp`：同乗した兄弟/配偶者の数  
- `Parch`：同乗した親/子供の数  
- `Fare`：運賃  
- `Embarked`：乗船港（C/Q/S）  

## 最低限の前処理の例

- `Age`, `Fare`, `Embarked` の欠損補完  
- `Sex`, `Embarked` のカテゴリ変数エンコード  
- 必要に応じて `FamilySize = SibSp + Parch + 1` などの特徴量作成