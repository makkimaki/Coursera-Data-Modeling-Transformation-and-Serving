# DBTを使ったスター・スキーマの実装手順

## 1. DBTプロジェクトのセットアップ

### (1) モデルフォルダの作成
DBTの `models` フォルダ内に新しく `star_schema_models` ディレクトリを作成します。

```sh
rm -r models/example  # 自動生成された example フォルダを削除
mkdir models/star_schema_models  # 新しいフォルダを作成
```

---

## 2. スター・スキーマのSQLモデル作成

### (1) `dim_stores.sql` の作成
```sql
SELECT 
    store_id AS store_key, 
    store_name, 
    store_city, 
    store_zipcode
FROM staging.stores;
```

- `staging` は、PostgreSQLデータベース内のスキーマ名。
- `store_id` を `store_key` として主キー（サロゲートキー）に使用。

### (2) `schema.yml` の作成
DBTでは、YAMLファイルを使って **テーブルのドキュメント化とテスト設定** が可能です。

```yaml
models:
  - name: dim_stores
    description: "Stores dimension table."
    columns:
      - name: store_key
        description: "Primary key for dim_stores table."
        tests:
          - unique
          - not_null
```

### (3) その他のモデル作成
```sql
-- dim_items.sql
SELECT 
    sku AS item_key, 
    name, 
    brand
FROM staging.items;
```

```sql
-- fact_orders.sql
SELECT 
    order_id AS fact_order_key, 
    store_id AS store_key,
    sku AS item_key,
    order_date AS date_key,
    quantity,
    price
FROM staging.orders;
```

---

## 3. DBTの設定ファイル更新

### (1) `dbt_project.yml` の更新
デフォルト設定を変更し、新しい `star_schema_models` ディレクトリを指定。

```yaml
models:
  star_schema_models:
    +materialized: table
```

- `materialized: table` を指定し、データベース内に**物理テーブル** を作成。

---

## 4. DBTの実行

### (1) DBTモデルのビルド
```sh
dbt run
```
または、特定のサブディレクトリを選択して実行：
```sh
dbt run -S star_schema_models
```

### (2) PostgreSQL接続とテーブル確認
```sh
psql -d my_database
```
```sql
SELECT schema_name FROM information_schema.schemata;
SELECT table_name FROM information_schema.tables WHERE table_schema = 'star_schema';
```

### (3) データのテストと検証
```sh
dbt test
```

---

## 5. DBTの追加機能

### (1) サロゲートキーの生成
```sql
SELECT 
    dbt_utils.generate_surrogate_key([store_id]) AS store_key
FROM staging.stores;
```

### (2) `dbt_date` パッケージの利用
```sql
SELECT * FROM dbt_date.get_date_dimension('2020-01-01', '2025-01-01');
```

---

## 6. まとめ
✅ **DBTを使ってスター・スキーマを構築**  
1. **モデルフォルダ (`star_schema_models`) を作成**
2. **各テーブルのSQL (`dim_stores.sql` など) を記述**
3. **スキーマファイル (`schema.yml`) を作成**
4. **DBTプロジェクト設定 (`dbt_project.yml`) を更新**
5. **`dbt run` でデータベースにテーブルを作成**
6. **`dbt test` でデータの整合性を検証**
7. **サロゲートキーや `dbt_date` などの機能を活用**

🚀 **次のステップ：実際にDBTを使ってスター・スキーマを構築するラボに挑戦！**
