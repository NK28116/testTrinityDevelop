# 詳細設計書

## 1. ディレクトリ構造

```
src/
├── app/                    # Next.js App Router
│   ├── page.tsx           # ホームページ (登録フォーム)
│   └── api/              # API Routes
│       └── register/     # 登録API
│           └── route.ts
├── components/            # UIコンポーネント
│   ├── ui/               # 汎用UIコンポーネント
│   │   ├── Input.tsx
│   │   ├── Button.tsx
│   │   └── ErrorMessage.tsx
│   └── features/         # 機能別コンポーネント
│       └── RegistrationForm.tsx
├── hooks/                 # カスタムフック
│   └── useRegistration.ts
├── lib/                   # ユーティリティ・APIクライアント
│   ├── api.ts           # APIクライアント関数
│   ├── validation.ts    # バリデーション関数
│   └── logger.ts        # ロギング関数
├── types/                 # 型定義
│   ├── user.ts
│   ├── registration.ts
│   └── setting.ts
└── .env.local            # 環境変数ファイル
```

## 2. 型定義・インターフェース

```typescript
// src/types/user.ts
export interface User {
  user_id: number;
  username: string;
  email: string;
  password: string;
}

// src/types/registration.ts
export interface RegistrationLog {
  log_id: number;
  user_id: number;
  timestamp: string;
  status: 'success' | 'failure';
  message: string;
}

// src/types/setting.ts
export interface RegistrationSetting {
  setting_id: number;
  service_name: string;
  api_endpoint: string;
  request_method: 'POST' | 'GET' | 'PUT' | 'DELETE'; // 必要なメソッドを追加
}

export interface RegistrationFormData {
  username: string;
  email: string;
  password: string;
}
```

## 3. コンポーネント詳細仕様

### 3.1 RegistrationForm
- **ファイル**: `src/components/features/RegistrationForm.tsx`
- **Props**:  なし
- **State**:
  | 状態 | 型 | 初期値 | 説明 |
  |------|-----|--------|------|
  | formData | `RegistrationFormData` | `{ username: '', email: '', password: '' }` | フォームの入力値を保持 |
  | errors | `{[key: string]: string}` | `{}` | 各フィールドのエラーメッセージを保持 |
  | registrationStatus | `'idle' | 'loading' | 'success' | 'error'` | `'idle'` | 登録処理の状態を保持 |
  | successMessage | `string` | `''` | 登録成功時のメッセージ |
- **イベントハンドラ**:
  - `handleChange`: 入力フィールドの値が変更されたときに`formData`を更新する。
  - `handleSubmit`: フォーム送信時に登録処理を実行する。

```typescript jsx
// src/components/features/RegistrationForm.tsx
import React, { useState } from 'react';
import { RegistrationFormData } from '@/types/registration';
import { Input } from '@/components/ui/Input';
import { Button } from '@/components/ui/Button';
import { ErrorMessage } from '@/components/ui/ErrorMessage';
import { useRegistration } from '@/hooks/useRegistration';

const RegistrationForm = () => {
  const [formData, setFormData] = useState<RegistrationFormData>({
    username: '',
    email: '',
    password: '',
  });

  const { errors, registrationStatus, successMessage, register } = useRegistration();

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await register(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <Input
        type="text"
        name="username"
        placeholder="Username"
        value={formData.username}
        onChange={handleChange}
      />
      {errors.username && <ErrorMessage message={errors.username} />}

      <Input
        type="email"
        name="email"
        placeholder="Email"
        value={formData.email}
        onChange={handleChange}
      />
      {errors.email && <ErrorMessage message={errors.email} />}

      <Input
        type="password"
        name="password"
        placeholder="Password"
        value={formData.password}
        onChange={handleChange}
      />
      {errors.password && <ErrorMessage message={errors.password} />}

      <Button type="submit" disabled={registrationStatus === 'loading'}>
        {registrationStatus === 'loading' ? 'Registering...' : 'Register'}
      </Button>

      {registrationStatus === 'success' && <p>{successMessage}</p>}
      {registrationStatus === 'error' && <p>Registration failed. Please try again.</p>}
    </form>
  );
};

export default RegistrationForm;
```

### 3.2 Input (汎用UIコンポーネント)
- **ファイル**: `src/components/ui/Input.tsx`
- **Props**:  標準的な InputProps に加え、`errorMessage?: string`
- **State**:  なし

```typescript jsx
// src/components/ui/Input.tsx
import React from 'react';

interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  errorMessage?: string;
}

export const Input: React.FC<InputProps> = ({ errorMessage, ...props }) => {
  return (
    <div>
      <input {...props} />
      {errorMessage && <p style={{ color: 'red' }}>{errorMessage}</p>}
    </div>
  );
};
```

### 3.3 Button (汎用UIコンポーネント)
- **ファイル**: `src/components/ui/Button.tsx`
- **Props**: 標準的な ButtonProps

```typescript jsx
// src/components/ui/Button.tsx
import React from 'react';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {}

export const Button: React.FC<ButtonProps> = ({ children, ...props }) => {
  return <button {...props}>{children}</button>;
};
```

### 3.4 ErrorMessage (汎用UIコンポーネント)
- **ファイル**: `src/components/ui/ErrorMessage.tsx`
- **Props**: `message: string`
- **State**: なし

```typescript jsx
// src/components/ui/ErrorMessage.tsx
import React from 'react';

interface ErrorMessageProps {
  message: string;
}

export const ErrorMessage: React.FC<ErrorMessageProps> = ({ message }) => {
  return <p style={{ color: 'red' }}>{message}</p>;
};
```

## 4. API/関数仕様

### 4.1 register
- **ファイル**: `src/lib/api.ts`
- **シグネチャ**: `async function register(data: RegistrationFormData): Promise<{ success: boolean; message: string; errors?: {[key: string]: string} }>`
- **処理フロー**:
  1.  `data` を `/api/register` エンドポイントへ POST リクエストとして送信する。
  2.  レスポンスをJSON形式でパースする。
  3.  レスポンスのステータスコードをチェックする。ステータスコードが 200 以外の場合、エラーを投げる。
  4.  レスポンスの `success` プロパティが `true` の場合、成功メッセージとともに成功を返す。
  5.  レスポンスの `success` プロパティが `false` の場合、エラーメッセージとともに失敗を返す（エラーがレスポンスに含まれている場合はそれも返す）。
- **エラーハンドリング**:
  - API通信エラー: `try...catch` ブロックでキャッチし、エラーメッセージを返す。
  - バリデーションエラー: バックエンドから返されたエラーメッセージを`errors`に含めて返す。

```typescript
// src/lib/api.ts
import { RegistrationFormData } from '@/types/registration';

export async function register(data: RegistrationFormData): Promise<{ success: boolean; message: string; errors?: {[key: string]: string} }> {
  try {
    const response = await fetch('/api/register', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });

    const result = await response.json();

    if (!response.ok) {
      throw new Error(result.message || 'Registration failed');
    }

    return result;
  } catch (error: any) {
    console.error("Registration API Error:", error);
    return { success: false, message: error.message || 'An unexpected error occurred' };
  }
}
```

### 4.2 validateRegistrationData
- **ファイル**: `src/lib/validation.ts`
- **シグネチャ**: `function validateRegistrationData(data: RegistrationFormData): {[key: string]: string}`
- **処理フロー**:
  1.  `username` が空でないことを確認する。空の場合、`username` フィールドのエラーメッセージを返す。
  2.  `email` が空でないこと、および有効なメールアドレス形式であることを確認する。そうでない場合、`email` フィールドのエラーメッセージを返す。
  3.  `password` が空でないこと、および 8 文字以上であることを確認する。そうでない場合、`password` フィールドのエラーメッセージを返す。
  4.  エラーがない場合、空のオブジェクトを返す。
- **エラーハンドリング**:
  - 各バリデーション条件が満たされない場合、対応するエラーメッセージを返す。

```typescript
// src/lib/validation.ts
import { RegistrationFormData } from '@/types/registration';

export function validateRegistrationData(data: RegistrationFormData): {[key: string]: string} {
  const errors: {[key: string]: string} = {};

  if (!data.username) {
    errors.username = 'Username is required';
  }

  if (!data.email) {
    errors.email = 'Email is required';
  } else if (!/\S+@\S+\.\S+/.test(data.email)) {
    errors.email = 'Email address is invalid';
  }

  if (!data.password) {
    errors.password = 'Password is required';
  } else if (data.password.length < 8) {
    errors.password = 'Password must be at least 8 characters long';
  }

  return errors;
}
```

### 4.3 logEvent
- **ファイル**: `src/lib/logger.ts`
- **シグネチャ**: `function logEvent(message: string, level: 'info' | 'warn' | 'error'): void`
- **処理フロー**:
  1.  ログメッセージとログレベルを受け取る。
  2.  ログレベルに応じて、コンソールにログを出力する。
  3.  (オプション) ログサーバーにログを送信する (環境変数で制御)。
- **エラーハンドリング**:
  - ログサーバーへの送信に失敗した場合、エラーログを出力する。

```typescript
// src/lib/logger.ts
export function logEvent(message: string, level: 'info' | 'warn' | 'error'): void {
  const timestamp = new Date().toISOString();
  const logMessage = `[${timestamp}] [${level.toUpperCase()}] ${message}`;

  switch (level) {
    case 'error':
      console.error(logMessage);
      break;
    case 'warn':
      console.warn(logMessage);
      break;
    default:
      console.log(logMessage);
  }

  // Optional: Send logs to a logging server
  if (process.env.NEXT_PUBLIC_LOG_SERVER_URL) {
    fetch(process.env.NEXT_PUBLIC_LOG_SERVER_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        message: logMessage,
        level: level,
      }),
    }).catch((error) => {
      console.error('Failed to send log to server:', error);
    });
  }
}

```

## 5. 状態管理設計

### 5.1 グローバル状態
今回のアプリケーションでは、グローバル状態は必要ありません。

### 5.2 ローカル状態

*   `RegistrationForm` コンポーネント内で、`formData` (フォームの入力値)、`errors` (エラーメッセージ), `registrationStatus`(登録処理の状態) を `useState` で管理する。
*   登録処理の結果は、`successMessage` として `useState` で管理する。

## 6. データフロー

```
[ユーザーが登録フォームに入力] → [handleChange イベントハンドラが formData を更新] → [handleSubmit イベントハンドラがフォーム送信時に register 関数を呼び出し] → [register 関数が /api/register エンドポイントへ POST リクエストを送信] → [API Route がバリデーションを行い、データベースを更新し、結果を返す] → [register 関数が結果を処理し、registrationStatus および successMessage を更新] → [UIが再レンダリングされ、結果が表示される]
```

## 7. エラーハンドリング方針

| エラー種別 | 対応方法 |
|-----------|---------|
| API通信エラー | `lib/api.ts` の `register` 関数内で `try...catch` ブロックでキャッチし、ユーザーにエラーメッセージを表示する。  |
| バリデーションエラー | `lib/validation.ts` でバリデーションを行い、エラーメッセージを `RegistrationForm` コンポーネントに渡し、フォームにエラー表示する。 |
| バックエンドエラー | `/api/register` エンドポイントで発生したエラーをキャッチし、適切なエラーメッセージをJSON形式でフロントエンドに返す。 |
| 予期せぬエラー | エラーバウンダリ (必要に応じて実装) でキャッチし、ユーザーにエラーメッセージを表示する。 `_error.tsx` もしくはグローバルな Error Boundaryを検討。|

## 8. API Route (Backend)

`/app/api/register/route.ts`

```typescript
// app/api/register/route.ts
import { NextResponse } from 'next/server';
import { RegistrationFormData } from '@/types/registration';
import { validateRegistrationData } from '@/lib/validation';
import { logEvent } from '@/lib/logger';

// Test Database (SQLite) interaction - placeholder for real ORM integration
const USERS = [
  {user_id: 1, username: 'testuser', email: 'test@example.com', password: 'password123'}
]
let nextUserId = 2; // Simple auto-increment for testing.


export async function POST(request: Request) {
  try {
    const data: RegistrationFormData = await request.json();
    logEvent(`Registration attempt for user: ${data.username}`, 'info');

    // Input Validation
    const errors = validateRegistrationData(data);
    if (Object.keys(errors).length > 0) {
      logEvent(`Validation errors: ${JSON.stringify(errors)}`, 'warn');
      return NextResponse.json({ success: false, message: 'Validation failed', errors }, { status: 400 });
    }

    // Simulate Database interaction
    const newUser = {
      user_id: nextUserId++, // Increment User ID
      username: data.username,
      email: data.email,
      password: data.password, // Store password securely in a real app - this is just a demo
    }
    USERS.push(newUser);

    logEvent(`User registered successfully: ${data.username}`, 'info');

    return NextResponse.json({ success: true, message: 'Registration successful!' }, { status: 200 });
  } catch (error: any) {
    logEvent(`Registration failed: ${error.message}`, 'error');
    return NextResponse.json({ success: false, message: error.message || 'An unexpected error occurred' }, { status: 500 });
  }
}
```

## 9. Hooks

### 9.1 useRegistration
- **ファイル**: `src/hooks/useRegistration.ts`
- **説明**: 登録フォームのロジックをカプセル化するためのカスタムフック。
- **State**:
  | 状態 | 型 | 初期値 | 説明 |
  |---|---|---|---|
  | `errors` | `{[key: string]: string}` | `{}` | フォームの入力エラーを保持 |
  | `registrationStatus` | `'idle' | 'loading' | 'success' | 'error'` | `'idle'` | 登録処理の状態を保持 |
  | `successMessage` | `string` | `''` | 登録成功時のメッセージ |
- **関数**:
  | 関数名 | 引数 | 戻り値 | 説明 |
  |---|---|---|---|
  | `register` | `RegistrationFormData` | `Promise<void>` | 登録処理を実行し、状態を更新する |

```typescript
// src/hooks/useRegistration.ts
import { useState } from 'react';
import { RegistrationFormData } from '@/types/registration';
import { register as registerAPI } from '@/lib/api';
import { validateRegistrationData } from '@/lib/validation';

export const useRegistration = () => {
  const [errors, setErrors] = useState<{[key: string]: string}>({});
  const [registrationStatus, setRegistrationStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [successMessage, setSuccessMessage] = useState<string>('');

  const register = async (data: RegistrationFormData) => {
    setRegistrationStatus('loading');
    setErrors({}); // Clear previous errors

    const validationErrors = validateRegistrationData(data);
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      setRegistrationStatus('idle'); // Reset status after validation fails
      return;
    }

    try {
      const result = await registerAPI(data);

      if (result.success) {
        setSuccessMessage(result.message);
        setRegistrationStatus('success');
      } else {
        setErrors(result.errors || {}); // Set backend validation errors, if any
        setRegistrationStatus('error');
      }
    } catch (error: any) {
      // Handle unexpected API errors
      console.error("Registration Hook Error:", error);
      setErrors({ general: error.message || 'An unexpected error occurred' });
      setRegistrationStatus('error');
    }
  };

  return { errors, registrationStatus, successMessage, register };
};
```

## 10. 環境変数

`.env.local` ファイルに、必要に応じて環境変数を定義します。

```
NEXT_PUBLIC_LOG_SERVER_URL=http://localhost:8080/logs # Optional log server URL
```
