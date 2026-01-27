## テスト計画

### テスト対象
- `src/components/features/TestCaseForm.tsx` (テストケースフォーム)
- `src/lib/testRunner.ts` (テスト実行処理)

### テスト観点

#### TestCaseForm (テストケースフォーム)
- **正常系**:
    - 正しい値が入力された場合、onSubmit propで渡された関数が呼ばれる。
    - 初期値が渡された場合、フォームに初期値が設定される。
    - 各フィールドの入力値が適切にstateに反映される。
    - JSON Editorの内容が適切に反映される。
- **異常系**:
    - 必須項目が未入力の場合、エラーメッセージが表示される。
    - 無効なJSON形式の文字列が入力された場合、エラーメッセージが表示される。
    - onSubmit propで渡された関数がエラーをスローした場合、エラーメッセージが表示される。
- **境界値**:
    - 各フィールドの最大文字数を超えた場合、入力が制限される。
    - inputDataとexpectedOutputに非常に大きなJSONを入力した場合の動作確認。
    - AssertionTypeの各値が正常に動作する。
- **パフォーマンス**:
    - 特に入力フィールドが多い場合や、ReactJsonの表示が遅延しないか確認。

#### TestRunner (テスト実行処理)
- **正常系**:
    - テストケースが正常に実行され、期待される結果が得られた場合、`TestStatus.SUCCESS`となる。
    - AssertionTypeがEQUAL, CONTAINS, REGEXの場合に、それぞれ正しい判定が行われる。
    - テスト実行後、TestResultがAPI経由で保存される。
- **異常系**:
    - 存在しない`testCaseId`が指定された場合、エラーが適切に処理される。
    - 存在しない`moduleName`、`className`、`functionName`が指定された場合、エラーが適切に処理される。
    - テスト対象の関数がエラーをスローした場合、`TestStatus.ERROR`となり、エラーメッセージとスタックトレースが記録される。
    - 無効なJSON形式の`inputData`または`expectedOutput`が指定された場合、エラーが適切に処理される。
    - API呼び出し(テストケース取得、テスト結果保存)が失敗した場合、エラーが適切に処理される。
- **境界値**:
    - `inputData`に非常に大きなJSONを入力した場合の動作確認。
    - テスト対象関数が非常に長い実行時間を要する場合のタイムアウト処理。
    - `expectedOutput`に空文字や特殊文字のみの文字列を指定した場合の動作確認。
- **パフォーマンス**:
    - テスト対象関数の実行時間が長い場合に、UIがフリーズしないか確認。
    - 大量のテストケースを連続して実行した場合のパフォーマンス劣化。

### テストケース一覧

#### TestCaseForm (テストケースフォーム)
| ID | 分類 | テスト内容 | 入力 | 期待結果 |
|----|------|-----------|------|---------|
| TC01 | 正常系 | 全ての必須項目に正しい値を入力してSubmit | name: "Test Case 1", functionName: "add", moduleName: "./math", inputData: `{"a": 1, "b": 2}`, expectedOutput: `3`, assertionType: "EQUAL" | onSubmit propで渡された関数が、上記の入力値で呼び出される |
| TC02 | 正常系 | 初期値が設定された状態でフォームを表示 | initialValues: { id: "123", name: "Existing Test Case", ...} | フォームの各フィールドに初期値が表示される |
| TC03 | 正常系 | inputDataにJSONを編集し、onSubmitで正しいJSONが渡されることを確認 | inputData: {"a":1} -> {"a":2} | onSubmit propで渡された関数が、inputData={"a":2} で呼び出される |
| TC04 | 正常系 | expectedOutputにJSONを編集し、onSubmitで正しいJSONが渡されることを確認 | expectedOutput: {"a":1} -> {"a":2} | onSubmit propで渡された関数が、expectedOutput={"a":2} で呼び出される |
| TC05 | 異常系 | 必須項目(name)が空の状態でSubmit | name: "", functionName: "add", moduleName: "./math" | エラーメッセージが表示される |
| TC06 | 異常系 | 無効なJSON形式のinputDataを入力してSubmit | name: "Test Case 1", inputData: "invalid json" | エラーメッセージが表示される |
| TC07 | 異常系 | 無効なJSON形式のexpectedOutputを入力してSubmit | name: "Test Case 1", expectedOutput: "invalid json" | エラーメッセージが表示される |
| TC08 | 異常系 | onSubmitで渡された関数がエラーをスローする | - | エラーメッセージが表示される |
| TC09 | 境界値 | 各フィールドに最大文字数を超える値を入力 | name: (256文字), description: (1024文字), ... | 最大文字数を超えた分の入力が制限される |
| TC10 | 境界値 | inputDataに非常に大きなJSONを入力 | inputData: (非常に大きなJSON) | フォームが正常に動作する |
| TC11 | 境界値 | expectedOutputに非常に大きなJSONを入力 | expectedOutput: (非常に大きなJSON) | フォームが正常に動作する |

#### TestRunner (テスト実行処理)
| ID | 分類 | テスト内容 | 入力 | 期待結果 |
|----|------|-----------|------|---------|
| TC20 | 正常系 | テストケースが正常に実行され、期待される結果が得られる (EQUAL) | testCaseId: "TC01" (add関数, 1 + 2 = 3) | TestResult.statusがSUCCESSとなる |
| TC21 | 正常系 | テストケースが正常に実行され、期待される結果が得られる (CONTAINS) | testCaseId: "TC02" (文字列結合関数, "hello" + "world" contains "world") | TestResult.statusがSUCCESSとなる |
| TC22 | 正常系 | テストケースが正常に実行され、期待される結果が得られる (REGEX) | testCaseId: "TC03" (メールアドレス検証関数, "test@example.com" matches regex) | TestResult.statusがSUCCESSとなる |
| TC23 | 正常系 | テスト実行後、TestResultがAPI経由で保存されることを確認 | testCaseId: "TC01" | API(`/api/tests/results`)にTestResultがPOSTされる |
| TC24 | 異常系 | 存在しないtestCaseIdを指定してテストを実行 | testCaseId: "INVALID_ID" | TestResult.statusがERRORとなり、エラーメッセージに "Test case with ID INVALID_ID not found." が含まれる |
| TC25 | 異常系 | 存在しないmoduleNameを指定してテストを実行 | testCaseId: "TC04" (存在しないモジュールを参照) | TestResult.statusがERRORとなり、エラーメッセージに "Cannot find module" が含まれる |
| TC26 | 異常系 | 存在しないfunctionNameを指定してテストを実行 | testCaseId: "TC05" (存在しない関数を参照) | TestResult.statusがERRORとなり、エラーメッセージに "Function ... not found in module ..." が含まれる |
| TC27 | 異常系 | テスト対象の関数がエラーをスローする | testCaseId: "TC06" (例外をスローする関数) | TestResult.statusがERRORとなり、エラーメッセージにテスト対象関数がスローしたエラーが含まれる |
| TC28 | 異常系 | 無効なJSON形式のinputDataを指定してテストを実行 | testCaseId: "TC07" (inputDataがinvalid JSON) | JSONパースエラーが発生し、TestResult.statusがERRORになる |
| TC29 | 異常系 | API呼び出し(テストケース取得)が失敗する | - | TestResult.statusがERRORとなり、エラーメッセージにAPIエラーに関する情報が含まれる |
| TC30 | 異常系 | assertionに失敗する場合 | testCaseId: "TC08" (1 + 1 = 3を期待するテストケース) | TestResult.statusがFAILUREとなる |
| TC31 | 境界値 | inputDataに非常に大きなJSONを入力 | testCaseId: "TC09" (inputDataが巨大なJSON) | 正常にテストが実行され、期待通りの結果になる |
| TC32 | 境界値 | テスト対象関数が非常に長い実行時間を要する場合 | testCaseId: "TC10" (長時間処理を行う関数) | タイムアウトが発生するか、正常にテストが終了する |

### カバレッジ目標
- ライン カバレッジ: 80%
- ブランチ カバレッジ: 70%

### Claudeへの実装指示
- 使用するテストフレームワーク: Jest
- テストファイルの配置場所: 各コンポーネント/モジュールの隣に `*.test.tsx`または`*.test.ts` ファイルを配置する。例: `src/components/features/TestCaseForm.test.tsx`、`src/lib/testRunner.test.ts`
- モック・スタブの方針:
    - API呼び出し (`api.ts`) は `jest.mock` でモックする。
    - 動的インポート (`testRunner.ts`) は `jest.mock` と `webpackIgnore: true` を組み合わせてモックする。
    - UIコンポーネントのテスト (`TestCaseForm.tsx`) では、必要に応じて `jest.fn` でpropsの関数をモックする。

```typescript
// 例: src/components/features/TestCaseForm.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import TestCaseForm from './TestCaseForm';
import * as api from '@/lib/api';

jest.mock('@/lib/api'); // APIモジュール全体をモック

describe('TestCaseForm', () => {
  const mockOnSubmit = jest.fn();
  const mockOnCancel = jest.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
    mockOnCancel.mockClear();
    (api.createTestCase as jest.Mock).mockResolvedValue({ id: '123', name: 'Test Case', ... }); // モックAPIの戻り値を設定
  });

  it('TC01: Should call onSubmit with correct values when all required fields are filled', async () => {
    render(<TestCaseForm onSubmit={mockOnSubmit} onCancel={mockOnCancel} />);

    fireEvent.change(screen.getByLabelText('Name:'), { target: { value: 'Test Case 1' } });
    fireEvent.change(screen.getByLabelText('Function Name:'), { target: { value: 'add' } });
    fireEvent.change(screen.getByLabelText('Module Name:'), { target: { value: './math' } });

    // ReactJson コンポーネントへの入力イベントの発火は少し複雑になる可能性があります。
    // 今回は割愛しますが、必要に応じて、ReactJsonコンポーネントが提供するAPIを利用してJSONを編集するテストを実装してください。

    fireEvent.click(screen.getByText('Submit'));

    expect(mockOnSubmit).toHaveBeenCalled();
    // onSubmitに渡される引数の詳細な検証は、ここでは省略
  });

  // 他のテストケースも同様に実装
});
```

```typescript
// 例: src/lib/testRunner.test.ts
import runTest from './testRunner';
import * as api from './api';
import { TestStatus } from '@/types/testcase';

jest.mock('./api'); // APIモジュール全体をモック

// 動的インポートをモックするための設定
jest.mock('./math', () => ({ // モックするモジュール名を指定
  add: (input: { a: number; b: number }) => input.a + input.b, // モック関数の実装
}), { virtual: true }); // virtual: true はモックがwebpackのバンドルに含まれないようにするオプション

describe('runTest', () => {
  beforeEach(() => {
    (api.getTestCase as jest.Mock).mockClear();
    (api.createTestResult as jest.Mock).mockClear();
    jest.clearAllMocks(); // モックをクリア
  });

  it('TC20: Should return SUCCESS when test case passes with EQUAL assertion', async () => {
    // モックのテストケース
    (api.getTestCase as jest.Mock).mockResolvedValue({
      id: 'TC01',
      name: 'Add Test',
      functionName: 'add',
      moduleName: './math',
      inputData: '{"a": 1, "b": 2}',
      expectedOutput: '3',
      assertionType: 'EQUAL',
    });

    const result = await runTest('TC01');

    expect(result.status).toBe(TestStatus.SUCCESS);
    expect(api.createTestResult).toHaveBeenCalled();
  });

  it('TC24: Should return ERROR when test case ID is invalid', async () => {
    (api.getTestCase as jest.Mock).mockRejectedValue(new Error('Test case not found')); // APIがエラーを返すようにモック

    const result = await runTest('INVALID_ID');

    expect(result.status).toBe(TestStatus.ERROR);
    expect(result.errorMessage).toBe('Test case not found');
  });
});
```

**補足:**

*   上記のテストケース一覧はあくまで例です。詳細設計書と実装コードを元に、より網羅的なテストケースを設計してください。
*   カバレッジ目標はあくまで目安です。重要なロジックや複雑な処理は、より高いカバレッジを目指してください。
*   APIモックにおけるエラーケースの再現には `mockRejectedValue`を利用する。
*   動的インポートをモックする際は、`webpackIgnore: true` と `jest.mock` を組み合わせることで、Jestがモジュールを解決できない問題を回避できる。
*   `ReactJson` のテストは、ライブラリのAPI仕様に依存するため、必要に応じて別途調査し、適切なテスト方法を検討してください。(onChangeイベントの発火方法など)

このテスト計画に基づいて、テストコードを実装し、継続的にテストを実行することで、コードの品質を向上させることができます。
