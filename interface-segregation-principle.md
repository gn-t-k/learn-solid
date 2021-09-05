# I … Interface Segregation Principle: インターフェイス分離の原則

## 概要

インターフェイス分離の原則とは、インターフェイスとクライアント（インターフェイスの利用者）があるとき、インターフェイスに用意されてある（利用するクライアントにとって）不必要なメソッドにクライアントが依存しなくてもよいように、分割できるインターフェイスは分割するべきであるという原則。

## 原則に違反してはいけない理由

インターフェイスに用意されている不必要なメソッドにクライアントが依存してしまうと、クライアントでは「例外を返すだけのメソッド」のような意味のないメソッドを用意しなければならない。また、そのようなクライアントが増えると、インターフェイスに変更を加えたときの影響範囲も増えてしまい、バグを生む可能性が高まる。

## 原則に違反している例

タスク管理アプリケーションの「タスク」に関する情報を持つ`Task`オブジェクトと、「タスクのタイトル」を更新する`updateTaskTitle`関数との関係について。`ITaskRepository`はDBなどの永続化層のインターフェイスを表しているが、この例ではあまり重要でないため説明は省略する。

```typescript
type Task = {
  id: string;
  title: string;
  details: string;
  status: "ToDo" | "InProgress" | "Done";
  createdAt: Date;
  updatedAt: Date;
};
```

```typescript
const updateTaskTitle = async (
  task: Task,
  repository: ITaskRepository,
): Promise<void> => {
  await repository.updateTitle({ id: task.id, title: task.title });
};
```

```typescript
interface ITaskRepository {
  registerTask: (props: { title: string; details: string }) => Promise<void>;
  getTask: (props: { id: string }) => Promise<Task>;
  updateTitle: (props: { id: string; title: string }) => Promise<void>;
  updateDetails: (props: { id: string; details: string }) => Promise<void>;
  // etc...
}
```

`updateTaskTitle`関数の仮引数として`Task`型の`task`オブジェクトを設定しているが、この部分がインターフェイス分離の原則に違反している可能性がある。

### 違反している理由

`updateTaskTitle`関数では、`Task`型の`task`オブジェクトを仮引数として設定しているが、関数内で実際に使われているのは、`Task`型のうち`id`と`title`のみである。そのため、たとえば、`Task`型が以下のように変更されてしまった場合、バグが発生する。

```typescript
type Task = {
  id: string;
  taskInfo: {
    title: string;
    details: string;
    status: "ToDo" | "InProgress" | "Done";
  },
  createdAt: Date;
  updatedAt: Date;
};
```

`updateTaskTitle`関数では`task.title`が`undefined`になってしまうため、`repository.updateTitle`が必ず失敗してしまう。

```typescript
const updateTaskTitle = async (
  task: Task,
  repository: ITaskRepository,
): Promise<void> => {
  await repository.updateTitle({ id: task.id, title: task.title });
  // => TS2339: Property 'title' does not exist on type 'Task'.
};
```

### 解決策

`updateTaskTitle`関数ではstring型の`id`とstring型の`title`しか利用しないため、仮引数を`Task`まるごとではなく`id`と`title`に分割することで、`Task`の変更によるバグなどは防ぐことができるようになる。（`updateTaskTitle`関数を利用する側から`task.taskInfo.title`を渡せばよい）

```typescript
const updateTaskTitle = async (
  task: {id: string, title: string},
  repository: ITaskRepository,
): Promise<void> => {
  await repository.updateTitle({ id: task.id, title: task.title });
};
```

※ `Task`が、各プロパティの整合性を保つためのバリデーターメソッドなどを備えたクラス等であった場合、必ずしもインターフェイスを分割すべきであるとは言えないため、注意が必要。
