# app ディレクトリ

Next.js App Routerのページコンポーネントが格納されています。

## ディレクトリ構成

```
app/
├── testcases/           # テストケース管理画面
│   ├── page.tsx          # テストケース一覧ページ
│   └── [id]/            # テストケース詳細・編集ページ
│       └── page.tsx
├── tests/               # テスト実行画面
│   └── page.tsx
└── reports/             # レポート表示画面
    └── page.tsx
```

## 各ページの役割

- **testcases/**: テストケースの作成、編集、一覧表示を行う
- **tests/**: テストの実行と結果表示を行う
- **reports/**: テスト結果のレポートとカバレッジ情報を表示する

## ルーティング

- `/testcases` - テストケース一覧
- `/testcases/[id]` - テストケース詳細・編集
- `/tests` - テスト実行
- `/reports` - レポート表示