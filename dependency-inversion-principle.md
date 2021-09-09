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

## 依存性逆転の原則に違反してはいけない理由

モジュールAがモジュールBの実装を参照していた場合、モジュールBの変更がモジュールAに影響を及ぼす可能性がある。そのため、モジュールBの改修を行う際は、モジュールBの実装に依存しているモジュールAに影響がないかなどの調査を行わねばならず、そのぶん工数がかかってしまう。

モジュールAとモジュールBの間にインターフェイスIを挟むことで、（インターフェイスIを正しく実装していさえすれば）モジュールAを意識することなくモジュールBを改修することができるようになる。

## 原則に違反している例

TypeScriptのサンプルコードで、依存性逆転の原則に違反している例を紹介する。

`User`型のユーザー情報をAPIから取得してユーザー名を返す関数`getUserName`について。

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
  const response = await fetchUser();
  // fetchの`response.json()`の型がPromise<any>であるため、自前のwrapperで型をつける
  const user: User = await validateType<User>(response.json());

  return user.name;
};
```

```typescript
export const fetchUser = async () => {
  try {
    const response = await fetch("/api/user");

    return response;
  } catch (error) {
    throw new Error(error);
  }
};
```

`getUserName`関数はデータの取得に`fetchUser`関数を使用しており、さらに`getUserName`関数は`fetchUser`が内部で`fetch`を利用していることを知っている（`response.json()`を使用している）。これは、`getUserName`が`fetchUser`の実装に依存しているといえる。図で示すと、以下のような関係になっている。

```puml
@startuml
object getUserName {
  fetchUserを利用した処理
}
object fetchUser {
  User取得処理
}

getUserName -> fetchUser
@enduml
```

データの取得に`fetch`ではなくライブラリ`axios`を使いたくなった場合を考えてみる。`fetchUser`関数を以下のように修正する。

```typescript
import axios from "axios";

export const fetchUser = async (): Promise<User> => {
  try {
    const response = await axios.get<User>("/api/user");
    const user = response.data;

    return user;
  } catch (error) {
    throw new Error(error);
  }
};
```

新しい`fetchUser`から返ってくる値は`json()`メソッドを持っていないため、`getUserName`関数ではエラーが発生する。

```typescript
const getUserName = async () => {
  const response = await fetchUser();
  // fetchの`response.json()`の型がPromise<any>であるため、自前のwrapperで型をつける
  const user: User = await validateType<User>(response.json());
  //                                                   ~~~~
  //                                                   ^Property 'json' does not exist on type 'User'.

  return user.name;
};
```

`getUserName`以外にも`fetchUser`を使っている関数などがあった場合、それらもすべて修正する必要があり、工数がかかる上にバグを埋め込んでしまう可能性も高い。

## 解決策

これを避けるために、`getUserName`が抽象的なインターフェイスに依存するような設計で実装してみる。

まず、`fetchUser`関数の型を表現した`IFetchUser`を用意する。

```typescript
interface IFetchUser {
  fetchUser: () => Promise<User>;
}
```

次に、`getUserName`関数を、`IFetchUser`を利用する形で実装する。

```typescript
const getUserName = async ({ fetchUser }: IFetchUser) => {
  const user: User = await fetchUser();

  return user.name;
};
```

このように、`fetchUser`を`import`ではなく関数の引数として受け取ることで、インターフェイスに依存させることができる。

そして、`fetchUser`を`IFetchUser`のインターフェイス通りに実装する。`fetch`を使う場合は以下のようになる。

```typescript
export const fetchUser = async (): Promise<User> => {
  try {
    const response = await fetch("/api/user");
    // fetchの`response.json()`の型がPromise<any>であるため、自前のwrapperで型をつける
    const user = validateType<User>(response.json());

    return user;
  } catch (error) {
    throw new Error(error);
  }
};
```

図で示すと、以下のような関係になっている。

```puml
@startuml
object getUserName {
  IFetchUserを利用した処理
}
object fetchUser {
  IFetchUserの具体的な処理
}
object IFetchUser {
  fetchUserの入出力を表している
}

getUserName -> IFetchUser
IFetchUser <|- fetchUser
@enduml
```

この抽象と実装の関係ができていれば、`fetch`のかわりに`axios`を使いたくなった場合でも、`IFetchUser`を実装できていさえすれば`getUserName`などを気にせず`fetchUser`を変更することができるようになる。
