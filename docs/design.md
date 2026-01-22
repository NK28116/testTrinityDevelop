```markdown
# 詳細設計書

## 1. ディレクトリ構造

```
src/
├── app/
│   ├── testcases/            # テストケース管理画面
│   │   ├── page.tsx           # テストケース一覧ページ
│   │   ├── [id]/             # テストケース詳細・編集ページ
│   │   │   └── page.tsx
│   ├── tests/                # テスト実行画面
│   │   └── page.tsx
│   ├── reports/              # レポート表示画面
│   │   └── page.tsx
│   └── ...
├── components/
│   ├── ui/               # 汎用UIコンポーネント
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Table.tsx
│   │   └── ...
│   ├── features/         # 機能別コンポーネント
│   │   ├── TestCaseForm.tsx      # テストケース作成・編集フォーム
│   │   ├── TestResultTable.tsx   # テスト結果表示テーブル
│   │   ├── CoverageReport.tsx  # カバレッジレポート表示
│   │   └── ...
├── hooks/
│   ├── useTestCases.ts     # テストケース取得・管理用カスタムフック
│   ├── useTestResults.ts   # テスト結果取得用カスタムフック
│   └── ...
├── lib/
│   ├── api.ts            # APIクライアント
│   ├── testRunner.ts     # テスト実行処理
│   ├── reportGenerator.ts # レポート生成処理
│   └── utils.ts          # ユーティリティ関数
├── types/
│   ├── testcase.ts
│   ├── testresult.ts
│   ├── report.ts
│   └── ...
└── ...
```

## 2. 型定義・インターフェース

```typescript
// src/types/testcase.ts
interface TestCase {
  id: string;           // UUID
  name: string;         // テストケース名
  description: string;  // テストケースの説明
  functionName: string; // テスト対象の関数名
  className?: string;    // テスト対象のクラス名 (optional)
  moduleName: string;   // テスト対象のモジュール名
  inputData: string;    // 入力データ (JSON文字列)
  expectedOutput: string; // 期待される出力 (JSON文字列)
  assertionType: AssertionType; // 検証方法
  createdAt: Date;      // 作成日
  updatedAt: Date;      // 更新日
}

enum AssertionType {
  EQUAL = "EQUAL",
  CONTAINS = "CONTAINS",
  REGEX = "REGEX",
}

// src/types/testresult.ts
interface TestResult {
  id: string;           // UUID
  testCaseId: string;   // 紐づくテストケースID
  status: TestStatus;   // テスト結果 (SUCCESS, FAILURE, ERROR)
  actualOutput: string; // 実際の出力
  errorMessage?: string; // エラーメッセージ (エラー時のみ)
  stackTrace?: string;   // スタックトレース (エラー時のみ)
  executionTime: number;  // 実行時間 (ミリ秒)
  executedAt: Date;     // 実行日時
  testCodeId: string;   // テスト対象のコードID
}

enum TestStatus {
  SUCCESS = "SUCCESS",
  FAILURE = "FAILURE",
  ERROR = "ERROR",
}

// src/types/report.ts
interface CoverageReport {
  id: string;          // UUID
  reportData: string;    // カバレッジデータ (JSON文字列)
  generatedAt: Date;      // 生成日時
}

```

## 3. コンポーネント詳細仕様

### 3.1 TestCaseForm (テストケースフォーム)
- **ファイル**: `src/components/features/TestCaseForm.tsx`
- **Props**:
  | プロパティ | 型 | 必須 | 説明 |
  |-----------|-----|------|------|
  | initialValues | `TestCase \| null` |  | 初期値（編集時）。新規作成時はnull。 |
  | onSubmit | `(testCase: TestCase) => Promise<void>` | ✓ | フォーム送信時のハンドラ。 |
  | onCancel | `() => void` |  | キャンセルボタン押下時のハンドラ。 |
- **State**:
  | 状態 | 型 | 初期値 | 説明 |
  |------|-----|--------|------|
  | name | `string` | `""` | テストケース名 |
  | description | `string` | `""` | テストケースの説明 |
  | functionName | `string` | `""` | テスト対象の関数名 |
  | className | `string` | `""` | テスト対象のクラス名 |
  | moduleName | `string` | `""` | テスト対象のモジュール名 |
  | inputData | `string` | `"{}"` | 入力データ (JSON文字列) |
  | expectedOutput | `string` | `"{}"` | 期待される出力 (JSON文字列) |
  | assertionType | `AssertionType` | `AssertionType.EQUAL` | 検証方法 |
  | isSubmitting | `boolean` | `false` | 送信中フラグ |
- **イベントハンドラ**:
  - `handleChange(e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>)`: 各入力フィールドの変更を処理し、対応するstateを更新する。
  - `handleSubmit(e: React.FormEvent)`: フォーム送信を処理する。入力バリデーションを行い、`onSubmit` propを呼び出す。送信中は`isSubmitting` を true にする。
- **UI**:
  - ラベル付きのテキスト入力フィールド (name, description, functionName, className, moduleName)
  - JSONエディタ (inputData, expectedOutput) -  `react-json-view` or similar
  - セレクトボックス (assertionType)
  - 送信ボタン
  - キャンセルボタン

```typescript jsx
// src/components/features/TestCaseForm.tsx
import React, { useState } from 'react';
import { TestCase, AssertionType } from '@/types/testcase';
import { Button, Input } from '@/components/ui';  // 例: src/components/ui/Button.tsx, src/components/ui/Input.tsx
import ReactJson from 'react-json-view';

interface TestCaseFormProps {
  initialValues?: TestCase | null;
  onSubmit: (testCase: TestCase) => Promise<void>;
  onCancel: () => void;
}

const TestCaseForm: React.FC<TestCaseFormProps> = ({ initialValues, onSubmit, onCancel }) => {
  const [name, setName] = useState(initialValues?.name || "");
  const [description, setDescription] = useState(initialValues?.description || "");
  const [functionName, setFunctionName] = useState(initialValues?.functionName || "");
  const [className, setClassName] = useState(initialValues?.className || "");
  const [moduleName, setModuleName] = useState(initialValues?.moduleName || "");
  const [inputData, setInputData] = useState(initialValues?.inputData || "{}");
  const [expectedOutput, setExpectedOutput] = useState(initialValues?.expectedOutput || "{}");
  const [assertionType, setAssertionType] = useState<AssertionType>(initialValues?.assertionType || AssertionType.EQUAL);
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => {
    const { name, value } = e.target;
    switch (name) {
      case "name": setName(value); break;
      case "description": setDescription(value); break;
      case "functionName": setFunctionName(value); break;
      case "className": setClassName(value); break;
      case "moduleName": setModuleName(value); break;
      case "inputData": setInputData(value); break;
      case "expectedOutput": setExpectedOutput(value); break;
      case "assertionType": setAssertionType(value as AssertionType); break;
      default: break;
    }
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);

    try {
      // バリデーション
      if (!name || !functionName || !moduleName) {
        alert("Name, Function Name, and Module Name are required.");
        return;
      }

      const testCase: TestCase = {
        id: initialValues?.id || crypto.randomUUID(), // 新規作成時はUUIDを生成
        name,
        description,
        functionName,
        className,
        moduleName,
        inputData,
        expectedOutput,
        assertionType,
        createdAt: initialValues?.createdAt || new Date(),
        updatedAt: new Date(),
      };
      await onSubmit(testCase);
    } catch (error) {
      console.error("Error submitting form:", error);
      alert("Failed to submit the form.");
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name:</label>
        <Input type="text" id="name" name="name" value={name} onChange={handleChange} />
      </div>
      <div>
        <label htmlFor="description">Description:</label>
        <Input type="text" id="description" name="description" value={description} onChange={handleChange} />
      </div>
      <div>
        <label htmlFor="functionName">Function Name:</label>
        <Input type="text" id="functionName" name="functionName" value={functionName} onChange={handleChange} />
      </div>
      <div>
        <label htmlFor="className">Class Name:</label>
        <Input type="text" id="className" name="className" value={className} onChange={handleChange} />
      </div>
      <div>
        <label htmlFor="moduleName">Module Name:</label>
        <Input type="text" id="moduleName" name="moduleName" value={moduleName} onChange={handleChange} />
      </div>
      <div>
        <label htmlFor="inputData">Input Data (JSON):</label>
        {/*
        <Input type="textarea" id="inputData" name="inputData" value={inputData} onChange={handleChange} />
        */}
        <ReactJson src={JSON.parse(inputData)} name="inputData" onEdit={e => setInputData(JSON.stringify(e.updated_src))}  onAdd={e => setInputData(JSON.stringify(e.updated_src))} onDelete={e => setInputData(JSON.stringify(e.updated_src))}/>

      </div>
      <div>
        <label htmlFor="expectedOutput">Expected Output (JSON):</label>
        {/*
        <Input type="textarea" id="expectedOutput" name="expectedOutput" value={expectedOutput} onChange={handleChange} />
        */}
        <ReactJson src={JSON.parse(expectedOutput)} name="expectedOutput" onEdit={e => setExpectedOutput(JSON.stringify(e.updated_src))}  onAdd={e => setExpectedOutput(JSON.stringify(e.updated_src))} onDelete={e => setExpectedOutput(JSON.stringify(e.updated_src))}/>
      </div>
      <div>
        <label htmlFor="assertionType">Assertion Type:</label>
        <select id="assertionType" name="assertionType" value={assertionType} onChange={handleChange}>
          <option value={AssertionType.EQUAL}>EQUAL</option>
          <option value={AssertionType.CONTAINS}>CONTAINS</option>
          <option value={AssertionType.REGEX}>REGEX</option>
        </select>
      </div>

      <div>
        <Button type="submit" disabled={isSubmitting}>{isSubmitting ? "Submitting..." : "Submit"}</Button>
        <Button type="button" onClick={onCancel}>Cancel</Button>
      </div>
    </form>
  );
};

export default TestCaseForm;
```

### 3.2 TestResultTable (テスト結果テーブル)
- **ファイル**: `src/components/features/TestResultTable.tsx`
- **Props**:
  | プロパティ | 型 | 必須 | 説明 |
  |-----------|-----|------|------|
  | testResults | `TestResult[]` | ✓ | 表示するテスト結果の配列。 |
- **State**:
  | 状態 | 型 | 初期値 | 説明 |
  |------|-----|--------|------|
  | (なし) |  |  |  |
- **イベントハンドラ**: (なし)
- **UI**:
  - `Table` コンポーネント (または同様のテーブル表示コンポーネント)
  - 各行は、テスト結果のプロパティを表示する (id, testCaseId, status, executionTime, executedAt など)。
  -  テスト結果が `FAILURE` または `ERROR` の場合、エラーメッセージとスタックトレースを表示するための展開可能な詳細セクション。

## 4. API/関数仕様

### 4.1 createTestCase (テストケース作成)
- **ファイル**: `src/lib/api.ts`
- **シグネチャ**: `async function createTestCase(testCase: Omit<TestCase, 'id' | 'createdAt' | 'updatedAt'>): Promise<TestCase>`
- **処理フロー**:
  1. APIエンドポイント `/api/testcases` へ POST リクエストを送信する。
  2. リクエストボディにテストケースのデータを JSON 形式で含める。
  3. レスポンスを解析し、作成されたテストケースのデータを返す。
- **エラーハンドリング**:
  - API 呼び出しが失敗した場合、例外をスローする。

### 4.2 runTest (テスト実行)
- **ファイル**: `src/lib/testRunner.ts`
- **シグネチャ**: `async function runTest(testCaseId: string): Promise<TestResult>`
- **処理フロー**:
  1.  `testCaseId` をもとに、API(`/api/testcases/{testCaseId}`)からテストケースを取得する。
  2.  テスト対象コード(`testCase.moduleName`,`testCase.className`,`testCase.functionName`)を動的にインポートする。
  3.  テストケースの `inputData` を JSON としてパースし、テスト対象関数に引数として渡す。
  4.  テスト対象関数を実行し、結果を取得する。
  5.  `testCase.expectedOutput` と実際の結果を、`testCase.assertionType` に基づいて比較する。
  6.  テスト結果 (`TestResult`) を生成し、API(`/api/tests/results`)にPOSTリクエストで保存する。
  7.  成功/失敗に関わらず、テスト結果を返す。
- **エラーハンドリング**:
  1.  テストケースの取得失敗：エラーをスロー
  2.  動的インポート失敗：エラーをスロー
  3.  テスト対象関数の実行時エラー：エラーをキャッチし、エラーメッセージとスタックトレースを `TestResult` に含める。
  4.  assertion失敗: `TestResult.status` を `FAILURE`に設定。

```typescript
// src/lib/testRunner.ts
import { TestCase, TestResult, TestStatus, AssertionType } from "@/types/testcase";
import { getTestCase, createTestResult } from "./api"; // APIクライアント関数

async function runTest(testCaseId: string): Promise<TestResult> {
  try {
    // 1. テストケースの取得
    const testCase: TestCase = await getTestCase(testCaseId);
    if (!testCase) {
      throw new Error(`Test case with ID ${testCaseId} not found.`);
    }

    // 2. テスト対象コードの動的インポート
    const module = await import(/* webpackIgnore: true */ `${testCase.moduleName}`); // webpackIgnore: true は動的インポート時にwebpackが処理しないようにする指示
    const targetFunction = testCase.className ? module[testCase.className][testCase.functionName] : module[testCase.functionName];

    if (typeof targetFunction !== 'function') {
      throw new Error(`Function ${testCase.functionName} not found in module ${testCase.moduleName}.`);
    }

    // 3. 入力データのパース
    const inputData = JSON.parse(testCase.inputData);

    // 4. テスト対象関数の実行
    let actualOutput: any;
    try {
      actualOutput = await targetFunction(inputData);  // 関数実行
    } catch (error: any) {
      // エラーハンドリング
      console.error("Error during test execution:", error);
      const testResult: TestResult = {
        id: crypto.randomUUID(),
        testCaseId: testCase.id,
        status: TestStatus.ERROR,
        actualOutput: error.message,  // エラーメッセージを格納
        errorMessage: error.message,
        stackTrace: error.stack,
        executionTime: 0,
        executedAt: new Date(),
        testCodeId: "testCodeId",  // 仮の値。必要に応じて修正
      };
      return testResult;
    }


    // 5. 検証
    let status: TestStatus = TestStatus.SUCCESS;
    let errorMessage: string | undefined = undefined;

    switch (testCase.assertionType) {
      case AssertionType.EQUAL:
        if (JSON.stringify(actualOutput) !== JSON.stringify(JSON.parse(testCase.expectedOutput))) {
          status = TestStatus.FAILURE;
          errorMessage = `Expected ${testCase.expectedOutput}, but got ${JSON.stringify(actualOutput)}`;
        }
        break;
      case AssertionType.CONTAINS:
        if (!JSON.stringify(actualOutput).includes(testCase.expectedOutput)) {
          status = TestStatus.FAILURE;
          errorMessage = `Expected to contain ${testCase.expectedOutput}, but got ${JSON.stringify(actualOutput)}`;
        }
        break;
      case AssertionType.REGEX:
        const regex = new RegExp(testCase.expectedOutput);
        if (!regex.test(JSON.stringify(actualOutput))) {
          status = TestStatus.FAILURE;
          errorMessage = `Expected to match regex ${testCase.expectedOutput}, but got ${JSON.stringify(actualOutput)}`;
        }
        break;
    }

    // 6. テスト結果の生成と保存
    const testResult: TestResult = {
      id: crypto.randomUUID(),
      testCaseId: testCase.id,
      status: status,
      actualOutput: JSON.stringify(actualOutput),
      errorMessage: errorMessage,
      executionTime: 0, // 実行時間を計測する場合は修正
      executedAt: new Date(),
      testCodeId: "testCodeId", // 仮の値。必要に応じて修正
      stackTrace: undefined
    };

    await createTestResult(testResult);
    return testResult;

  } catch (error: any) {
    console.error("Error running test:", error);

    // テストケース取得失敗やインポート失敗時のエラーハンドリング
    const testResult: TestResult = {
      id: crypto.randomUUID(),
      testCaseId: testCaseId,
      status: TestStatus.ERROR,
      actualOutput: error.message,  // エラーメッセージを格納
      errorMessage: error.message,
      stackTrace: error.stack,
      executionTime: 0,
      executedAt: new Date(),
      testCodeId: "testCodeId",  // 仮の値。必要に応じて修正
    };
    return testResult;
  }
}

export default runTest;
```

## 5. 状態管理設計

### 5.1 グローバル状態
- **テストケース**: テストケースの一覧を保持する。`useTestCases`カスタムフックで取得・管理する。`SWR`などでキャッシュするとパフォーマンスが向上する。
- **テスト結果**: テスト実行結果の一覧を保持する。`useTestResults`カスタムフックで取得・管理する。

### 5.2 ローカル状態
- `TestCaseForm`コンポーネント：フォームの入力値を保持する。
- テスト実行画面：実行中のテストケースの状態、実行結果の詳細表示状態などを保持する。

## 6. データフロー

```
[ユーザー操作: テストケース作成・編集] → [TestCaseForm: 入力値変更] → [TestCaseForm: state更新] → [ユーザー操作: フォーム送信] → [TestCaseForm: onSubmitハンドラ] → [api.createTestCase/updateTestCase] → [バックエンド: DB更新] → [グローバル状態: useTestCasesフックで再取得] → [UI再レンダリング: テストケース一覧表示]

[ユーザー操作: テスト実行] → [テスト実行画面: テスト実行ボタンクリック] → [runTest関数呼び出し] → [API呼び出し (testRunner.ts, api.ts)] → [バックエンド: テスト実行, 結果DB保存] → [グローバル状態: useTestResultsフックで再取得] → [UI再レンダリング: テスト結果表示]
```

## 7. エラーハンドリング方針

| エラー種別 | 対応方法 |
|-----------|---------|
| API通信エラー | リトライ (SWRの機能を利用) + ユーザー通知 (トースト通知など) |
| バリデーションエラー | フォームにエラー表示 (必須項目未入力など) |
| 予期せぬエラー | エラーバウンダリでキャッチ + エラーログ出力 |
| テスト実行時のエラー | テスト結果をFAILURE/ERRORとして記録し、エラーメッセージをUIに表示 |
```