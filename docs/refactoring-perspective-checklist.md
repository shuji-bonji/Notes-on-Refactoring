# コードレビューで使える「リファクタリング観点チェックリスト」

## レビューの目的

「コードの良し悪し」ではなく「未来の保守性・拡張性を保証できるか」こそがレビューの目的です。


## リファクタリング観点チェックリスト
PRテンプレートやレビューガイドに入れる想定です。

```markdown
### リファクタリング観点チェックリスト

#### コードの可読性・明瞭性
- [ ] ロジックが自然な流れで書かれているか（脳内デバッガーが不要か）
- [ ] メソッドや変数の命名は意図を正しく表しているか
- [ ] 関数やクラスの長さが適切か（長すぎないか）

#### ネスト・分岐構造の適切性
- [ ] if/switch のネストが深すぎないか（2階層以内が理想）
- [ ] else if の連鎖をEnumやStrategyパターンで置き換え可能か
- [ ] ガード節（early return）で分岐を簡素にできるか

#### 関心の分離
- [ ] 関数・クラスに複数の責務が混ざっていないか
- [ ] UIとビジネスロジックが混在していないか
- [ ] 共通ロジックはユーティリティ化・サービス化されているか

#### 重複コードの排除
- [ ] 同様のロジックが複数箇所に存在していないか
- [ ] DRY（Don't Repeat Yourself）原則に違反していないか

#### テスト容易性（TDDの恩恵）
- [ ] 関数の副作用が少なく、テストがしやすい構造になっているか
- [ ] テストコードに過剰なモックが必要になっていないか（密結合の兆候）
```


## 改善例（TypeScript）

### ネストだらけで複雑

#### Before:

```ts
function processUser(user: User) {
  if (user) {
    if (user.isActive) {
      if (user.role === 'admin') {
        // 管理者処理
      } else {
        // 一般ユーザー処理
      }
    } else {
      // 非アクティブ処理
    }
  } else {
    throw new Error('ユーザーが存在しません');
  }
}
```


#### After: Early return ＆ Strategy化
- 分岐を最小限に
- 各責務を別関数に抽出（単一責務＆テスト容易）
- 意図が読みやすくなる（ドメイン構造の反映）
```ts
function processUser(user: User) {
  if (!user) throw new Error('ユーザーが存在しません');
  if (!user.isActive) return handleInactiveUser(user);

  return user.role === 'admin'
    ? handleAdmin(user)
    : handleRegularUser(user);
}
```


### 重複コード
#### Before:

```ts
if (status === 'pending') {
  console.log('申請中です');
}
...
if (status === 'pending') {
  console.log('申請中です');
}
```

#### After: メッセージ定義を集約
- 可読性向上
- 他言語対応やUI側への受け渡しにも再利用可能

```ts
const statusMessages = {
  pending: '申請中です',
  approved: '承認済みです',
  rejected: '却下されました',
};

console.log(statusMessages[status]);
```


### ストしづらい密結合関数
####  Before:
```ts
function saveUser(user: User) {
  const response = fetch('/api/save', {
    method: 'POST',
    body: JSON.stringify(user),
  });
  alert('保存しました');
  return response;
}
```

#### After: I/Oを分離してテスト可能に
- ユニットテストでは fetch のmockだけで済む
- UI側では await saveUserToServer(user); notifySaved(); と分離して利用可能

```ts
function saveUserToServer(user: User): Promise<Response> {
  return fetch('/api/save', {
    method: 'POST',
    body: JSON.stringify(user),
  });
}

function notifySaved() {
  alert('保存しました');
}
```
