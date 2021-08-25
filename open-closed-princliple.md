# O … Open-Closed Principle: 開放閉鎖の原則

## 概要

ソフトウェアを構成する個々の部品は、拡張に対して開いていて（Open）、修正に対して閉じている（Closed）べきであるという原則。つまり、ソフトウェアに新しく機能を追加するとき、既存のコードを変更せず新しいコードを追加するだけで済むようにしておくべきであるという意味。

### 修正に対して閉じていなければいけない理由

既存のコードを変更するということは、すでに動作しているコードに変更を加えるということである。すでに動作しているコードに変更を加えると、バグを生んでしまう可能性があり、バグを生まないために動作確認を行うなどのコストを支払う必要がある。たとえば`switch`文に新しい条件分岐を追加するという簡単な修正であっても、`break`の書き忘れですでに動作しているコードが動作しなくなってしまう可能性がある。そういったケアレスミスを避けるためにも、新しい機能を追加する際は、既存のコードを修正しなくてもいいようにしておくべきである。

### 拡張に対して開いていなければいけない理由

既存のコードを変更せず新しいコードを追加するだけで済むようにしておくことで、既存の動作している機能を破壊することを恐れることなく、新しい機能を追加できるようになる。具体的な方法として、用意した`interface`を実装する方法を、TypeScriptのサンプルコードとともに紹介する。

## 違反した例

社員の等級と手当から、社員に支払う給与の支払額を計算する`CalculateSalaryService`クラスの例。

```typescript
type Position = "Intern" | "Staff" | "Manager";

class Employee {
  public constructor(public name: string, public position: Position) {}
}

class CaluculatePaymentService {
  private readonly BASE = 100;
  private totalPayment = 0;

  public constructor(
    private readonly employee: Employee,
    private readonly allowance: number = 0,
  ) {}

  public execute = (): number => {
    this.addSalaryToTotalPayment();
    this.addAllowanceToTotalPayment();

    return this.totalPayment;
  };

  private addSalaryToTotalPayment = (): void => {
    switch (this.employee.position) {
      case "Intern":
        this.totalPayment += this.BASE * 0.5;
        break;
      case "Staff":
        this.totalPayment += this.BASE;
        break;
      case "Manager":
        this.totalPayment += this.BASE * 2;
        break;
      default:
        const _check: never = this.employee.position;
    }
  };

  private addAllowanceToTotalPayment = (): void => {
    this.totalPayment += this.allowance;
  };
}
```

`Staff`であるBob（今月は`10`の手当がある）の給与の支払額を計算する場合は以下のようになる。

```typescript
const employee = new Employee("Bob", "Staff")
const totalPayment = new CaluculatePaymentService(employee, 10).execute()
console.log(totalPayment);
// => 110
```

### 違反している理由

### 解決策
