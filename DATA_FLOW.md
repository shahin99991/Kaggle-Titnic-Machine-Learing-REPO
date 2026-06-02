# データフロー図

## 全体の流れ

```mermaid
flowchart TD
    subgraph RAW["📁 data/raw/"]
        TRAIN_RAW["train.csv\n891件・12列\n✅ 正解ラベルあり（Survived）"]
        TEST_RAW["test.csv\n418件・11列\n❌ 正解ラベルなし"]
    end

    subgraph REF["参照のみ"]
        MEDIAN["中央値の計算\nAge中央値 ≈ 28歳\nFare中央値 ≈ 14.5ポンド"]
    end

    subgraph PREPROCESS["前処理（全データ共通）"]
        FE["特徴量エンジニアリング\n・Title（氏名から敬称抽出）\n・FamilySize = SibSp + Parch + 1\n・IsAlone（単独乗船フラグ）\n・Sex_enc（女性=1/男性=0）\n・Embarked_enc（S=0/C=1/Q=2）"]
        FILL["欠損値補完\n・Age → 中央値\n・Fare → 中央値\n・Embarked → 最頻値'S'"]
    end

    subgraph SPLIT["分割 train_test_split()"]
        direction LR
        TRAIN_SPLIT["train_split.csv\n712件（80%）\n学習用"]
        VALID_SPLIT["valid_split.csv\n179件（20%）\n評価用"]
    end

    subgraph MISSING["📁 data/missing_store/"]
        TEST_STORE["test.csv\n欠損値参照用として保管\n※ モデル評価には使わない"]
    end

    subgraph TRAIN["学習（STEP 3）"]
        DT["決定木\nDecisionTreeClassifier\nrandom_state=42"]
        RF["ランダムフォレスト\nRandomForestClassifier\nn_estimators=200\nrandom_state=42"]
        GB["GBDT\nGradientBoostingClassifier\nn_estimators=200\nlearning_rate=0.05\nmax_depth=4"]
    end

    subgraph MODELS["📁 models/"]
        DT_PKL["決定木.pkl"]
        RF_PKL["ランダムフォレスト.pkl"]
        GB_PKL["GBDT.pkl"]
    end

    subgraph EVAL["評価（STEP 4）"]
        PREDICT["valid_splitで予測\nmodel.predict(X_valid)"]
        METRICS["評価指標の算出\n・accuracy_score\n・confusion_matrix\n・classification_report\n　(precision / recall / F1)"]
        RESULT["最良モデル選定\nランダムフォレスト\n評価精度 83.8%"]
    end

    subgraph OUTPUT["📁 data/ 出力画像"]
        IMG1["model_evaluation_result.png\n精度比較・混同行列・特徴量重要度"]
    end

    %% データの流れ
    TRAIN_RAW -->|"concat して中央値算出"| MEDIAN
    TEST_RAW  -->|"concat して中央値算出"| MEDIAN
    TEST_RAW  -->|"保管"| TEST_STORE

    TRAIN_RAW --> FE
    MEDIAN    --> FILL
    FE --> FILL

    FILL -->|"stratify=Survived\nrandom_state=42"| TRAIN_SPLIT
    FILL -->|"stratify=Survived\nrandom_state=42"| VALID_SPLIT

    TRAIN_SPLIT --> DT
    TRAIN_SPLIT --> RF
    TRAIN_SPLIT --> GB

    DT --> DT_PKL
    RF --> RF_PKL
    GB --> GB_PKL

    VALID_SPLIT --> PREDICT
    DT_PKL --> PREDICT
    RF_PKL --> PREDICT
    GB_PKL --> PREDICT

    PREDICT --> METRICS
    METRICS --> RESULT
    RESULT  --> IMG1

    %% スタイル
    classDef rawStyle    fill:#1a3a5c,stroke:#5b9bd5,color:#cce0ff
    classDef splitStyle  fill:#1a4a2a,stroke:#4caf50,color:#c8e6c9
    classDef modelStyle  fill:#3a1a4a,stroke:#9c27b0,color:#e1bee7
    classDef evalStyle   fill:#4a2a00,stroke:#ff9800,color:#ffe0b2
    classDef outStyle    fill:#2a2a00,stroke:#ffeb3b,color:#fff9c4

    class TRAIN_RAW,TEST_RAW rawStyle
    class TRAIN_SPLIT,VALID_SPLIT splitStyle
    class DT,RF,GB,DT_PKL,RF_PKL,GB_PKL modelStyle
    class PREDICT,METRICS,RESULT evalStyle
    class IMG1 outStyle
```

---

## 特徴量一覧（モデルへの入力）

| # | 特徴量 | 元の列 | 種別 |
|---|---|---|---|
| 1 | `Pclass` | そのまま | 元の特徴量 |
| 2 | `Sex_enc` | Sex | エンコード |
| 3 | `Age` | Age（欠損補完済み） | 元の特徴量 |
| 4 | `Fare` | Fare（欠損補完済み） | 元の特徴量 |
| 5 | `FamilySize` | SibSp + Parch + 1 | エンジニアリング |
| 6 | `IsAlone` | FamilySize == 1 | エンジニアリング |
| 7 | `Title` | Name から抽出 | エンジニアリング |
| 8 | `Embarked_enc` | Embarked | エンコード |
| 9 | `SibSp` | そのまま | 元の特徴量 |
| 10 | `Parch` | そのまま | 元の特徴量 |

目的変数（正解ラベル）: `Survived`（0=死亡 / 1=生存）

---

## 評価結果サマリー

| モデル | 学習精度 | 評価精度 | 差（過学習） |
|---|---|---|---|
| 決定木 | 98.3% | 83.2% | −15.1% |
| **ランダムフォレスト** | 98.3% | **83.8%** | −14.5% |
| GBDT | 93.3% | 83.2% | **−10.1%**（最安定） |
