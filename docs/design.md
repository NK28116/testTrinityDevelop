# 詳細設計書

## 1. ディレクトリ構造

```
src/
├── unit/                 # Pythonユニットのソースコード
│   ├── email_validator.py   # メールアドレスバリデーションユニット
│   └── ...
├── tests/                # テストスクリプト
│   ├── test_email_validator.py # メールアドレスバリデーションユニットのテスト
│   └── ...
├── README.md              # ユニットの説明と使い方
└── requirements.txt       # 依存ライブラリ
```

## 2. 型定義・インターフェース

```python
# Pythonでは明示的なインターフェース定義は少ないですが、ドキュメンテーションで型を明確化します。

# email_validator.py

def validate_email(email: str) -> bool:
  """
  メールアドレスのバリデーションを行う関数

  Args:
    email (str): バリデーション対象のメールアドレス

  Returns:
    bool: 有効なメールアドレスの場合はTrue、そうでない場合はFalse
  """
  # ... (実装)

```

## 3. コンポーネント詳細仕様

### 3.1 メールアドレスバリデーションユニット

- **ファイル**: `src/unit/email_validator.py`
- **関数**: `validate_email(email: str) -> bool`
- **引数**:
  | 引数 | 型 | 説明 |
  |---|---|---|
  | `email` | `str` | バリデーション対象のメールアドレス文字列 |
- **戻り値**:
  | 戻り値 | 型 | 説明 |
  |---|---|---|
  | `bool` | `bool` | メールアドレスが有効な形式の場合 `True`、無効な形式の場合 `False` |
- **処理フロー**:
  1.  入力された文字列`email`がNoneまたは空文字列の場合、`False`を返却する。
  2.  正規表現パターン`r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"`を用いて、`email`がメールアドレスの形式に合致するかチェックする。
  3.  `re.match()`の結果が`True`であれば`True`を、そうでなければ`False`を返却する。
- **エラーハンドリング**:
    - 入力がNoneまたは空文字列の場合、明示的なエラーは発生させず、`False`を返却する。
    - 正規表現のマッチングでエラーが発生する可能性は低いが、念のためtry-exceptブロックで囲む（loggingは後述のエラーハンドリング方針参照）。

## 4. API/関数仕様

### 4.1 `validate_email`

- **ファイル**: `src/unit/email_validator.py`
- **シグネチャ**: `def validate_email(email: str) -> bool:`
- **処理フロー**:
  1.  入力`email`がNoneまたは空文字列であるか確認。
  2.  正規表現`r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"`を用いて、`email`がメールアドレスとして有効な形式かどうかを判定する。
  3.  正規表現にマッチした場合、`True`を返す。マッチしなかった場合、`False`を返す。
- **エラーハンドリング**:
    - 入力値がNoneまたは空文字列の場合、エラーとはみなさず、Falseを返す。
    - 正規表現コンパイルエラー（通常は発生しない）が発生した場合は、例外をキャッチし、ログに記録し、Falseを返す。
```python
import re
import logging

# ロガーの設定
logging.basicConfig(level=logging.ERROR)

def validate_email(email: str) -> bool:
    """
    メールアドレスのバリデーションを行う関数

    Args:
      email (str): バリデーション対象のメールアドレス

    Returns:
      bool: 有効なメールアドレスの場合はTrue、そうでない場合はFalse
    """
    if not email:
        return False

    try:
        pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
        return bool(re.match(pattern, email))
    except re.error as e:
        logging.error(f"Regex compilation error: {e}")
        return False
    except Exception as e:
        logging.exception(f"Unexpected error during email validation: {e}")
        return False
```

## 5. 状態管理設計

### 5.1 グローバル状態

このユニットは独立して動作するため、グローバル状態は持ちません。

### 5.2 ローカル状態

このユニットはローカル状態を持ちません。

## 6. データフロー

```
[メールアドレス文字列 (入力)] --> [validate_email関数] --> [真偽値 (出力)]
```

## 7. エラーハンドリング方針

| エラー種別 | 対応方法 |
|---|---|
| 入力値がNoneまたは空文字列 | `False`を返却 |
| 正規表現関連のエラー | 例外をキャッチし、ログに記録。`False`を返却 |
| その他の予期せぬエラー | 例外をキャッチし、ログに記録。`False`を返却 |

## 8. テスト設計

### 8.1 テストケース

以下のテストケースを含むpytestスクリプトを作成します。

- **正常系**:
    - 有効なメールアドレス (`test@example.com`)
    - サブドメインを含むメールアドレス (`test@sub.example.com`)
    - `+`記号を含むメールアドレス (`test+alias@example.com`)
    - `_`記号を含むメールアドレス (`test_user@example.com`)
- **異常系**:
    - 無効なメールアドレス (`invalid-email`)
    - `@`記号を含まないメールアドレス (`test.example.com`)
    - ドメイン名がないメールアドレス (`test@.com`)
    - トップレベルドメインがないメールアドレス (`test@example`)
    - 空文字列 (`""`)
    - None

### 8.2 テストコード例 (`tests/test_email_validator.py`)

```python
import pytest
from src.unit.email_validator import validate_email

def test_valid_email():
    assert validate_email("test@example.com") == True
    assert validate_email("test@sub.example.com") == True
    assert validate_email("test+alias@example.com") == True
    assert validate_email("test_user@example.com") == True

def test_invalid_email():
    assert validate_email("invalid-email") == False
    assert validate_email("test.example.com") == False
    assert validate_email("test@.com") == False
    assert validate_email("test@example") == False

def test_empty_email():
    assert validate_email("") == False

def test_none_email():
    assert validate_email(None) == False
```

## 9. デプロイメント

このユニットは、独立したPythonモジュールとしてデプロイされます。
他のシステムから呼び出す際には、`src/unit/email_validator.py`をインポートして`validate_email`関数を呼び出すことで利用できます。

## 10. その他の考慮事項

- 正規表現パターンは、メールアドレスの形式を厳密に検証するために、必要に応じて調整してください。
- パフォーマンスが重要な場合は、正規表現のコンパイルを一度だけ行い、再利用することを検討してください。
- より高度なバリデーションが必要な場合は、DNSルックアップによるドメインの存在確認などを検討してください。

この詳細設計書に基づき、実装を開始してください。
