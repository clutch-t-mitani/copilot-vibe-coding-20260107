# GitHub Copilot カスタム指示（Laravel 12 & PHP 8.5）

これらの指示は、Copilot が **Laravel 12 の最新標準**、**PHP 8.5 の言語機能**、  
および **ドメイン駆動設計（DDD）** に基づいた  
**保守性・安全性・品質の高いコード**を生成するための制約仕様である。

---

## ✅ 一般的なコーディング規約

- PSR-12 に従う
- 可読性と明確さを最優先する
- 意味の明確な命名を行う（省略しない）
- 複雑なクラス・メソッドには PHPDoc を記述する
- 単一責任原則（SRP）を遵守する
- マジックナンバーやハードコード文字列を禁止する

---

## ✅ PHP 8.5 ベストプラクティス

- 可能な限り `readonly` プロパティを使用する
- 定数の代替として Enum を使用する
- First-class callable 構文を利用する
- コンストラクタプロパティプロモーションを活用する
- Union / Intersection Types を使用する
- 厳密な戻り値型を指定する（`true` / `false` を含む）
- Static Return Type を適切に使用する
- Nullsafe Operator（`?->`）を使用する
- 継承を想定しないクラスは `final` にする
- Named Arguments を使用して可読性を高める
- PHP 8.5 の新機能および非推奨事項を常に考慮する

---

## ✅ アーキテクチャ方針

このプロジェクトは **Domain-Driven Design（DDD）** を採用する。

- ビジネスロジックは **Domain 層に集約**する
- すべての業務処理は **UseCase として実装**する
- Controller にビジネスロジックを書いてはならない

---

## ✅ ディレクトリ構成（DDD）

アプリケーションは以下の構成に従う：

app/
  Domain/
    Entities/
    ValueObjects/
    Services/
    Repositories/        # Repository Interface
    Exceptions/

  Application/
    UseCases/
    DTOs/

  Infrastructure/
    Persistence/
    Repositories/        # Repository Implementation
    Services/

  Presentation/
    Http/
      Controllers/
      Requests/
      Resources/

---

## ✅ レイヤ責務

### Domain Layer
- ビジネスルールの中核
- Laravel / Eloquent / Facade への依存は禁止
- Entity / ValueObject は不変性を優先する
- Setter を持たせない

### Application Layer
- ユースケースを定義する
- ドメインオブジェクトを組み合わせて処理を構成する
- HTTP / DB / 外部 API の詳細を知らない
- 1 UseCase = 1 業務操作

### Infrastructure Layer
- 永続化、外部サービス、Eloquent の実装を担当する
- Repository Interface の実装はこの層に置く
- Laravel への依存はこの層のみ許可する

### Presentation Layer
- HTTP リクエスト／レスポンスの変換のみを行う
- ビジネスロジックを含めない
- Controller は UseCase を呼び出すだけにする

---

## ✅ 依存関係ルール（厳守）

- Domain MUST NOT depend on any other layer
- Application MAY depend on Domain
- Infrastructure MAY depend on Domain and Application
- Presentation MUST depend on Application only

- Presentation MUST NOT access Domain or Infrastructure directly
- Domain MUST NOT use Laravel helpers or facades

---

## ✅ UseCase ルール

- すべての業務処理は UseCase として実装する
- Controller は UseCase のみを呼び出す
- Repository は Interface 経由で使用する
- UseCase はフレームワーク非依存であること

### UseCase 実装例

```php
final class CreateUserUseCase
{
    public function __construct(
        private readonly UserRepositoryInterface $userRepository
    ) {}

    public function execute(CreateUserInput $input): User
    {
        // business rules here
    }
}
