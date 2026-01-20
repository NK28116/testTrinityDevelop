```markdown
# 詳細設計書

## 1. ディレクトリ構造

```
src/
├── cli.py           # コマンドラインインターフェース (フロントエンド)
└── task_engine.py   # タスク実行エンジン (バックエンド)
```

## 2. 型定義・インターフェース

```python
# src/task_engine.py

from typing import TypedDict, Literal

ExecutionStatus = Literal["success", "failure", "pending"]

class TaskDefinition(TypedDict):
    task_id: int
    task_description: str
    execution_status: ExecutionStatus
```

## 3. コンポーネント詳細仕様

### 3.1 フロントエンド (コマンドラインインターフェース)

- **ファイル**: `src/cli.py`
- **Props**:  コマンドライン引数 (`sys.argv`)
- **State**:
  | 状態         | 型                      | 初期値    | 説明                                                                 |
  |--------------|--------------------------|-----------|----------------------------------------------------------------------|
  | `task_list` | `list[TaskDefinition]` | `[]` | 定義されたタスクのリスト                                                     |
  | `next_task_id` | `int`                 | `1`    | 次に割り当てるタスクID                                                      |
- **イベントハンドラ**: なし (コマンドライン引数に基づいて処理を実行)

- **コマンドの処理詳細**:

  - **`define <task>`**:
    1. コマンドライン引数から `<task>` (タスクの説明) を取得。
    2. 新しい `TaskDefinition` オブジェクトを作成。
       - `task_id`: `next_task_id` を使用。
       - `task_description`: `<task>` の値を使用。
       - `execution_status`: `"pending"` に設定。
    3. `task_list` に新しい `TaskDefinition` オブジェクトを追加。
    4. `next_task_id` をインクリメント。
    5. ユーザーにタスクが定義されたことを通知 (例: "Task 'ファイルを削除する' defined with ID 1")
  - **`execute <task_id>`**:
    1. コマンドライン引数から `<task_id>` を取得し、整数に変換。
    2. `task_list` から `task_id` が一致するタスクを検索。
    3. タスクが見つからない場合は、エラーメッセージを表示して処理を中断。
    4. タスク実行エンジン (`task_engine.execute_task`) を呼び出し、タスクの ID を渡す。
    5. タスク実行エンジンの実行結果 (success/failure) を受け取る。
    6. `task_list` 内の該当タスクの `execution_status` を実行結果で更新。
    7. ユーザーに実行結果を通知 (例: "Task 1 executed successfully.")
  - **`status <task_id>`**:
    1. コマンドライン引数から `<task_id>` を取得し、整数に変換。
    2. `task_list` から `task_id` が一致するタスクを検索。
    3. タスクが見つからない場合は、エラーメッセージを表示して処理を中断。
    4. 該当タスクの `execution_status` を表示 (例: "Task 1 status: pending")
  - **`list`**:
    1. `task_list` の内容をフォーマットして表示。
       - 各タスクについて、`task_id`, `task_description`, `execution_status` を表示。
       - テーブル形式で表示すると見やすい。

## 4. API/関数仕様

### 4.1 `execute_task`
- **ファイル**: `src/task_engine.py`
- **シグネチャ**: `def execute_task(task_id: int) -> Literal["success", "failure"]`
- **処理フロー**:
  1. タスク ID を受け取る。
  2. (本システムでは) 受け取ったタスク ID をコンソールに出力 (シミュレーション)。
  3. 確率 (例: 80%) で `"success"` を、残りの確率で `"failure"` をランダムに返す。
- **エラーハンドリング**:  なし (エラーが発生する可能性がないため)

## 5. 状態管理設計

### 5.1 グローバル状態
- `task_list`: 定義されたタスクのリスト。`cli.py` で管理。
- `next_task_id`: 次に割り当てるタスク ID。 `cli.py` で管理。

### 5.2 ローカル状態
- なし

## 6. データフロー

```
[ユーザー入力 (コマンド)] → [cli.py (コマンド処理)] → [task_engine.py (タスク実行, execute_task)] → [cli.py (結果の表示、状態更新)] → [標準出力 (結果表示)]
```

## 7. エラーハンドリング方針

| エラー種別             | 対応方法                                   |
|-----------------------|--------------------------------------------|
| 無効なコマンド         | エラーメッセージを表示してプログラムを終了 |
| 存在しないタスクID     | エラーメッセージを表示                     |
| 引数の型が異なる場合 | エラーメッセージを表示                     |

## 8. Pythonコード例

**src/cli.py:**

```python
import sys
import random
from task_engine import execute_task, TaskDefinition

task_list: list[TaskDefinition] = []
next_task_id: int = 1

def define_task(task_description: str) -> None:
    """新しいタスクを定義する"""
    global next_task_id, task_list
    new_task: TaskDefinition = {
        "task_id": next_task_id,
        "task_description": task_description,
        "execution_status": "pending",
    }
    task_list.append(new_task)
    print(f"Task '{task_description}' defined with ID {next_task_id}")
    next_task_id += 1

def execute_cli_task(task_id: int) -> None:
    """タスクを実行する"""
    global task_list
    for task in task_list:
        if task["task_id"] == task_id:
            result = execute_task(task_id)
            task["execution_status"] = result
            print(f"Task {task_id} executed {result}.")
            return

    print(f"Error: Task with ID {task_id} not found.")


def status_task(task_id: int) -> None:
    """タスクのステータスを表示する"""
    global task_list
    for task in task_list:
        if task["task_id"] == task_id:
            print(f"Task {task_id} status: {task['execution_status']}")
            return

    print(f"Error: Task with ID {task_id} not found.")

def list_tasks() -> None:
    """タスクの一覧を表示する"""
    if not task_list:
        print("No tasks defined.")
        return

    print("Tasks:")
    print("ID\tDescription\t\tStatus")
    print("----------------------------------------")
    for task in task_list:
        print(f"{task['task_id']}\t{task['task_description']}\t\t{task['execution_status']}")


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: cli.py <command> [arguments]")
        sys.exit(1)

    command = sys.argv[1]

    if command == "define":
        if len(sys.argv) < 3:
            print("Usage: cli.py define <task_description>")
            sys.exit(1)
        define_task(sys.argv[2])
    elif command == "execute":
        if len(sys.argv) < 3:
            print("Usage: cli.py execute <task_id>")
            sys.exit(1)
        try:
            task_id = int(sys.argv[2])
            execute_cli_task(task_id)
        except ValueError:
            print("Error: Task ID must be an integer.")
            sys.exit(1)

    elif command == "status":
        if len(sys.argv) < 3:
            print("Usage: cli.py status <task_id>")
            sys.exit(1)
        try:
            task_id = int(sys.argv[2])
            status_task(task_id)
        except ValueError:
            print("Error: Task ID must be an integer.")
            sys.exit(1)
    elif command == "list":
        list_tasks()
    else:
        print(f"Error: Unknown command '{command}'")
        sys.exit(1)
```

**src/task_engine.py:**

```python
import random
from typing import Literal

def execute_task(task_id: int) -> Literal["success", "failure"]:
    """タスクを実行する (ここでは単に成功/失敗をランダムに返す)"""
    print(f"Executing task with ID: {task_id}")
    if random.random() < 0.8:
        return "success"
    else:
        return "failure"
```

**実行例:**

```bash
python src/cli.py define ファイルを削除する
python src/cli.py define 別のファイルを削除する
python src/cli.py list
python src/cli.py execute 1
python src/cli.py status 1
```
