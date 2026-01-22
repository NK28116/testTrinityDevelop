# 詳細設計書

## 1. ディレクトリ構造

```
src/
├── main.py           # メイン処理
├── logger.py         # ロギング処理
```

## 2. 型定義・インターフェース

(今回は型定義は不要です。Python は動的型付け言語のため、明示的な型定義は必須ではありません。ただし、必要に応じて `typing` モジュールを使用できます。)

## 3. コンポーネント詳細仕様

### 3.1 メイン処理 (main.py)
- **ファイル**: `src/main.py`
- **Props**: (なし)
- **State**: (なし)
- **イベントハンドラ**: (なし)
- **処理内容**:
    1.  `logger.py` のロガーインスタンスを取得します。
    2.  ロガーを使って INFO レベルで起動メッセージを記録します。
    3.  画面に表示するテキストを定義します (例: "Hello, World!")。
    4.  `print()` 関数を使用してテキストをコンソールに出力します。
    5.  ロガーを使って INFO レベルで終了メッセージを記録します。

- **コード例**:

```python
# src/main.py
import logging
from logger import get_logger

logger = get_logger(__name__)

def main():
    logger.info("Application started")
    message = "Hello, World!"
    print(message)
    logger.info("Application finished")

if __name__ == "__main__":
    main()
```

### 3.2 ロギング処理 (logger.py)
- **ファイル**: `src/logger.py`
- **Props**: (なし)
- **State**: (なし)
- **イベントハンドラ**: (なし)
- **処理内容**:
    1.  `logging` モジュールを使用して、ロガーを初期化します。
    2.  ログレベルを設定します (例: `logging.INFO`)。
    3.  ログフォーマットを設定します (例: `%Y-%m-%d %H:%M:%S %levelname %(name)s: %(message)s`)。
    4.  コンソールまたはファイルにログを出力するハンドラーを追加します。
    5.  ロガーインスタンスを返す関数 `get_logger()` を定義します。

- **コード例**:

```python
# src/logger.py
import logging
import os

LOG_LEVEL = os.environ.get('LOG_LEVEL', 'INFO').upper()

def get_logger(name):
    logger = logging.getLogger(name)
    logger.setLevel(LOG_LEVEL)

    # ログフォーマット
    formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(name)s - %(message)s')

    # コンソールハンドラー
    ch = logging.StreamHandler()
    ch.setFormatter(formatter)
    logger.addHandler(ch)

    # ファイルハンドラー (オプション)
    # fh = logging.FileHandler('app.log')
    # fh.setFormatter(formatter)
    # logger.addHandler(fh)

    return logger
```

## 4. API/関数仕様

(API は定義しません。)

### 4.1 get_logger
- **ファイル**: `src/logger.py`
- **シグネチャ**: `def get_logger(name: str) -> logging.Logger`
- **処理フロー**:
  1.  `logging.getLogger(name)` でロガーインスタンスを取得します。
  2.  ロガーのログレベルを設定します (デフォルトは INFO)。
  3.  ログフォーマットを設定します。
  4.  コンソールハンドラーを追加します。
  5.  (オプション) ファイルハンドラーを追加します。
  6.  ロガーインスタンスを返却します。
- **エラーハンドリング**: 特にエラーハンドリングは行いません。

## 5. 状態管理設計

(状態管理は行いません。アプリケーションはステートレスです。)

## 6. データフロー

```
[アプリケーション起動] → [logger.py でロガー初期化] → [main.py でロガー取得] → [コンソールにテキスト出力] → [logger.py でログ出力]
```

## 7. エラーハンドリング方針

| エラー種別 | 対応方法 |
|-----------|---------|
| 予期せぬエラー | 例外をキャッチしてログに出力 (必要に応じて)。今回は最低限の構成のため、詳細なエラーハンドリングは省略します。|

**備考**

*   **ログ出力**: 必要に応じてファイル出力も有効にしてください。`logger.py` のコメントアウトされている箇所を修正することで設定できます。
*   **ログレベル**: 環境変数 `LOG_LEVEL` でログレベルを変更できるようにしています。デフォルトは INFO です。例えば、DEBUG レベルでログを出力する場合は、`LOG_LEVEL=DEBUG` を設定して実行します。
*   **実行方法**: `python src/main.py` コマンドで実行します。

この詳細設計に基づき、`src/main.py` と `src/logger.py` を実装してください。
