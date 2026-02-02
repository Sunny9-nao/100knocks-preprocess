# Pandas データ操作 公式集 (P-001～P-026)

## 概要
データサイエンス100本ノック（P-001～P-026）から抽出した、Pandasを使ったデータ操作の実践的な公式集です。

---

## 1. データの基本操作 【P-001～P-003】

### データの表示
```python
df.head(10)          # 先頭10件を表示
df.tail(10)          # 末尾10件を表示
```

### 列の選択
```python
# 単一列の選択（Seriesとして取得）
df['column_name']

# 複数列の選択（DataFrameとして取得）
df[['col1', 'col2', 'col3']]

# 1列でもDataFrameとして取得
df[['column_name']]
```

### 列名の変更
```python
# 特定の列名を変更
df.rename(columns={'old_name': 'new_name'})

# 複数の列名を一度に変更
df.rename(columns={'old1': 'new1', 'old2': 'new2'})

# 列名を直接代入
df.columns = ['new1', 'new2', 'new3']
```

---

## 2. 条件抽出とフィルタリング 【P-004～P-009】

### query()メソッドの基本
```python
# 単一条件
df.query('column == "value"')
df.query('amount >= 1000')

# AND条件（すべて満たす）
df.query('column1 == "value" & column2 >= 100')

# OR条件（いずれかを満たす）
df.query('amount >= 1000 | quantity >= 5')

# 範囲条件
df.query('1000 <= amount <= 2000')

# 否定条件
df.query('product_cd != "P071401019"')
```

### ド・モルガンの法則
```python
# not(A | B) = (not A) & (not B)
# not(A & B) = (not A) | (not B)

# 例: 以下の2つは等価
df.query('not(prefecture_cd == "13" | floor_area > 900)')
df.query('prefecture_cd != "13" & floor_area <= 900')
```

---

## 3. 文字列操作 【P-010～P-016】

### 基本的な文字列検索（必ず.strを使用）
```python
# 前方一致（XXXで始まる）
df.query('column.str.startswith("XXX")')

# 後方一致（XXXで終わる）
df.query('column.str.endswith("XXX")')

# 部分一致（XXXを含む）
df.query('column.str.contains("XXX")')
```

### 正規表現を使った文字列検索
```python
# Raw String（r''）を使用するとエスケープが不要で便利

# 先頭が特定の文字で始まる（A～F）
df.query("column.str.contains(r'^[A-F]')")

# 末尾が特定の文字で終わる（数字0～9）
df.query("column.str.contains(r'[0-9]$')")

# 複合パターン（先頭A～F、末尾0～9）
df.query("column.str.contains(r'^[A-F].*[0-9]$')")

# 数字のパターンマッチング（3桁-3桁-4桁の電話番号）
df.query('tel_no.str.contains(r"^\d{3}-\d{3}-\d{4}$")')
```

### 正規表現の基本記号
| 記号 | 意味 | 例 |
|------|------|-----|
| `^` | 行の先頭 | `r'^ABC'` : ABCで始まる |
| `$` | 行の末尾 | `r'XYZ$'` : XYZで終わる |
| `.` | 任意の1文字 | `r'a.c'` : aとcの間に任意の1文字 |
| `*` | 直前の0回以上の繰り返し | `r'ab*c'` : ac, abc, abbc... |
| `+` | 直前の1回以上の繰り返し | `r'ab+c'` : abc, abbc... |
| `{n}` | 直前の正確にn回の繰り返し | `r'\d{3}'` : 3桁の数字 |
| `[ABC]` | A, B, Cのいずれか | `r'[A-F]'` : A～Fのいずれか |
| `\d` | 数字（0～9） | `r'\d{4}'` : 4桁の数字 |
| `\w` | 英数字とアンダースコア | `r'\w+'` : 1文字以上の英数字 |

---

## 4. ソート 【P-017～P-018】

```python
# 昇順ソート（デフォルト）
df.sort_values('column_name')

# 降順ソート
df.sort_values('column_name', ascending=False)

# 複数列でソート
df.sort_values(['col1', 'col2'], ascending=[True, False])

# インデックスをリセット
df.sort_values('column_name').reset_index(drop=True)
```

---

## 5. ランキング 【P-019～P-020】

### rank()メソッドのオプション
```python
# 基本形
df['ranking'] = df['amount'].rank(ascending=False)

# method オプション
df['rank_min'] = df['amount'].rank(method='min', ascending=False)
df['rank_max'] = df['amount'].rank(method='max', ascending=False)
df['rank_first'] = df['amount'].rank(method='first', ascending=False)
df['rank_dense'] = df['amount'].rank(method='dense', ascending=False)
df['rank_avg'] = df['amount'].rank(method='average', ascending=False)
```

### methodオプションの動作（値: [10, 20, 20, 30] の場合）
| method | 結果 | 説明 |
|--------|------|------|
| `'min'` | [1, 2, 2, 4] | 同値には最小順位を割り当て |
| `'max'` | [1, 3, 3, 4] | 同値には最大順位を割り当て |
| `'first'` | [1, 2, 3, 4] | 出現順に異なる順位を割り当て |
| `'dense'` | [1, 2, 2, 3] | 同値は同順位、次は連続（ギャップなし） |
| `'average'` | [1, 2.5, 2.5, 4] | 同値には平均順位を割り当て（デフォルト） |

### ランキング列の追加例
```python
# DataFrameに新しい列としてランキングを追加
df_with_rank = pd.concat([
    df[['customer_id', 'amount']],
    df['amount'].rank(method='min', ascending=False)
], axis=1)
df_with_rank.columns = ['customer_id', 'amount', 'ranking']
```

---

## 6. 集計処理 【P-021～P-026】

### 基本的なカウント
```python
# 全体の件数
len(df)
df.shape[0]

# ユニーク件数
len(df['column'].unique())
df['column'].nunique()
```

### groupby()による集計
```python
# 単一の集計関数
df.groupby('column')['amount'].sum()
df.groupby('column')[['amount', 'quantity']].sum()

# agg()で複数の集計関数を指定
df.groupby('column').agg({'amount': 'sum', 'quantity': 'mean'})

# 同じ列に複数の集計関数を適用
df.groupby('column').agg({'sales_ymd': ['max', 'min']})

# 集計後にインデックスを列に変換
df.groupby('column').agg({'amount': 'sum'}).reset_index()
```

### 主な集計関数
| 関数 | 説明 | 使用例 |
|------|------|--------|
| `sum()` | 合計 | `df.groupby('col')['amount'].sum()` |
| `mean()` | 平均 | `df.groupby('col')['amount'].mean()` |
| `median()` | 中央値 | `df.groupby('col')['amount'].median()` |
| `min()` | 最小値 | `df.groupby('col')['date'].min()` |
| `max()` | 最大値 | `df.groupby('col')['date'].max()` |
| `count()` | 件数 | `df.groupby('col')['id'].count()` |
| `std()` | 標準偏差 | `df.groupby('col')['amount'].std()` |
| `var()` | 分散 | `df.groupby('col')['amount'].var()` |

### 集計後のフィルタリングとソート
```python
# 集計後にさらに条件抽出
df_agg = df.groupby('customer_id').agg({'sales_ymd': ['max', 'min']}).reset_index()
df_agg.columns = ['customer_id', 'max_date', 'min_date']
df_agg.query('max_date != min_date')

# 降順ソートしてTOP5を表示
df.groupby('store_cd')['amount'].mean().sort_values(ascending=False).head(5)
```

---

## クイックリファレンス

### よく使う操作の組み合わせ
```python
# 条件抽出 → 列選択 → 表示
df.query('amount >= 1000')[['customer_id', 'amount']].head(10)

# グループ化 → 集計 → ソート → 上位表示
df.groupby('store_cd')['amount'].sum().sort_values(ascending=False).head(5)

# 列追加（ランキング） → ソート → 表示
df['rank'] = df['amount'].rank(method='min', ascending=False)
df.sort_values('rank').head(10)

# 複数列の集計 → 列名変更 → フィルタリング
df_agg = df.groupby('id').agg({'date': ['max', 'min']}).reset_index()
df_agg.columns = ['id', 'max_date', 'min_date']
df_agg.query('max_date != min_date')
```

---

**更新日**: 2026年2月2日  
**対象範囲**: データサイエンス100本ノック P-001～P-026
