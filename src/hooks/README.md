# hooks ディレクトリ

カスタムフックが格納されています。

## カスタムフック一覧

### useTestCases.ts
テストケースの取得・管理を行うカスタムフック。
- テストケース一覧の取得
- テストケースの作成・更新・削除
- データのキャッシング
- エラーハンドリング

### useTestResults.ts
テスト結果の取得を行うカスタムフック。
- テスト結果一覧の取得
- テスト実行状態の管理
- リアルタイム更新対応

## 設計方針

- SWRまたはReact Queryを活用したデータフェッチング
- エラー状態とローディング状態の適切な管理
- 型安全性の確保
- コンポーネントからのビジネスロジック分離

## 使用例

```typescript
const { testCases, isLoading, error, createTestCase } = useTestCases();
const { testResults, runTest } = useTestResults();
```