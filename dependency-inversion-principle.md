# D … Dependency Inversion Principle: 依存性逆転の原則

## 概要

あるモジュールの別のモジュールを利用するとき、モジュールはお互いに直接依存すべきではなく、どちらのモジュールも、共有された抽象（インターフェイスや抽象クラスなど）に依存すべきであるという原則。

たとえば、関数Aが処理の内部で関数Bを直接読み込んで利用している場合、関数Aは関数Bの実装に依存しているといえる。

```puml
@startuml
object 関数A {
  関数Bを利用した処理
}
object 関数B {
  何かしらの処理
}

関数A -> 関数B
@enduml
```

依存性逆転の原則では、関数BのインターフェイスIを用意し、関数Aは関数Bの実装を参照するのではなくインターフェイスIを参照し、関数BはインターフェイスIを実装する形にすることが望ましいとされている。

```puml
@startuml
object 関数A {
  インターフェイスIを利用した処理
}
object 関数B {
  インターフェイスIの具体的な処理
}
object インターフェイスI {
  関数Bの入出力を表している
}

関数A -> インターフェイスI
インターフェイスI <|- 関数B
@enduml
```

## 原則に違反してはいけない理由

モジュールAがモジュールBの実装を参照していた場合、モジュールBの変更がモジュールAに影響を及ぼす可能性がある。そのため、モジュールBの改修を行う際は、モジュールBの実装に依存しているモジュールAに影響がないかなどの調査を行わねばならず、そのぶん工数がかかってしまう。

モジュールAとモジュールBの間にインターフェイスIを挟むことで、（インターフェイスIを正しく実装していさえすれば）モジュールAを意識することなくモジュールBを改修することができるようになる。

## 依存性逆転の原則の適用例

```typescript
export type User = {
  id: string;
  name: string;
  // etc...
};
```

```typescript
import { fetchUser } from "path/to/fetch-user";

const getUserName = async () => {
  const user = await fetchUser();

  return user.name;
};
```

```typescript
export const fetchUser = async (): Promise<User> => {
  try {
    const response = await fetch("/api/user");
    // fetchの返り値に型をつける自前のutil
    const user = validateType<User>(response.json());

    return user;
  } catch (error) {
    throw new Error(error);
  }
};
```

### 違反している理由

### 解決策
