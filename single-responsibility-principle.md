# S … Single Responsibility Principle: 単一責任の原則

## 概要

個々の部品が責任を負う対象は、たったひとつににするべきであるという原則。 ある対象の仕様または実装を変更したときに、別の対象の仕様/実装に影響が及んでしまうような場合、それは単一責任の原則に違反している可能性があり、変更がしにくいソフトウェアを生み出す原因になってしまう。

### 「部品が責任を負う対象」とは

ソフトウェアの既存の機能に変更を加えたり、新たな機能を追加する理由は、そのソフトウェアのユーザーやステークホルダーを満足させるためである。この「ソフトウェアのユーザーやステークホルダー」が、単一責任の原則における「部品が責任を負う対象」である。

### 「部品が責任を負う対象」にあわせて、部品を分割する

複数の対象に対して責任を負っている部品がある場合、その部品を、責任を負う対象の数に分割してあげることで、アンチパターンを回避することができる。TypeScriptのサンプルコードで、実例を解説する。

## 単一責任の原則に違反している例

たとえば、以下のような`Employee`クラスはSRPに違反しているといえる。

```typescript
class Employee {
  public name: string;
  public department: string;
  // etc...

  public constructor(...) {...};

  /**
   * 給与計算のメソッド
   * 経理部門に対して責任を負っている
   */
  public calculatePay = (): Money => {...};

  /**
   * 労働時間レポートを出力するメソッド
   * 人事部門に対して責任を負っている
   */
  public reportHours = (): string => {...};

  /**
   * 従業員情報をDBに保存するメソッド
   * データベース管理者に対して責任を負っている
   */
  public save = (): void => {...};
  
  /**
   * 所定労働時間を算出するメソッド
   * `calculatePay`と`reportHours`の両方で必要な処理のため、メソッドに切り出して共通化している
   */
  private getRegularHours = (): number => {...};
}
```

### 違反している理由

このクラスは、経理部門、人事部門、データベース管理者の3つの対象について責任を負っている。共通の処理を`regularHours`メソッドとして切り出しており、うまくコードの重複を回避しているように見えるが、これが、バグの原因になることも考えられる。

たとえば、経理部門から、所定労働時間の算出方法を変更したい依頼があったとする。改修担当者は、`calculatePay`から`getRegularHours`を呼んで所定労働時間を変更していることを確認し、`getRegularHours`に変更を加える。ユニットテストをパスし、経理部門の担当者に動作確認もしてもらい問題なかったため、この変更は本番環境にデプロイされる。

ここで問題なのは、改修担当者は`getRegularHours`が`reportHours`からも呼ばれていることを確認していないことである。もし、経理部門が扱う所定労働時間と人事部門が扱う所定労働時間の算出方法が異なるものだった場合、人事部門は所定労働時間の算出方法が誤ったものに変更されていることに気づかないまま、`getRegularHours`が算出した値を使い続けることになる。

### 解決策

「メソッドに変更を加える際は、そのメソッドがどこから呼ばれているかしっかり確認する」では、ケアレスミスを防ぐことが出来ず、根本的な解決策にはならない。

たとえば、`Employee`クラスを`PayCalculator`クラス・`HourReporter`クラス・`EmployeeSaver`クラス・`EmployeeData`クラスに分割することで、上記のようなミスを防ぐことができる。

```typescript
/**
 * 従業員に関するデータのみをプロパティとして持つ
 * メソッドは持たない
 */
class EmployeeData {
  public name: string;
  public department: string;
  // etc...

  public constructor(...) {...};
}

class PayCalculator {
  public constructor (
    private readonly employeeData: EmployeeData,
  ) {};

  public execute = (): Money => {...};
    
  private getRegularHours = (): number => {...};
}

class HourReporter {
  public constructor (
    private readonly employeeData: EmployeeData,
  ) {};

  public execute = (): string => {...};
  
  private getRegularHours = (): number => {...};
}

class EmployeeSaver {
  public constructor (
    private readonly employeeData: EmployeeData,
  ) {};

  public execute = (): void => {...};
}
```

`Employee`クラスを「責任を負う対象」の数に分割している。この場合、経理部門から所定労働時間の算出方法を変更したい依頼があった場合、`PayCalculator`の`getRegularHours`を変更しても他の部門に影響がないことは明らかである。
