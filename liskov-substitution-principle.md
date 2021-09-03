# L … Liskov Substitution Principle: リスコフの置換原則

## 概要

部品Tとその派生型である部品Sがあるとき、部品Tが使われている箇所はすべて部品Sで置換可能になるように部品Sはつくられているべきであるという原則。

### 「置換可能である」とは

オブジェクト指向のクラス設計でよくある部品Tと部品Sの関係として、インターフェイスとそれを実装するクラスの関係などがある。この関係のことをスーパータイプとサブタイプと呼ぶこともある。

リスコフの置換原則では、スーパータイプとサブタイプが置換可能であるとき、以下の2つのルールに則っているとされている。

> - サブタイプの事前条件はスーパータイプと同一か、それよりも弱めることができる（事前条件をスーパータイプより強めることは出来ない）
> - サブタイプの事後条件はスーパータイプと同一か、それよりも強めることができる（事後条件をスーパータイプより弱めることは出来ない）

事前条件とは、ある操作が実行される直前の状態で満たすべき条件のこと。「事前条件を弱める」というのは、具体的にはインスタンス生成のために必要な引数の数を少なくするなどがある。

事後条件とは、ある操作が実行された直後の状態で満たすべき条件のこと。「事後条件を強める」というのは、具体的にはメソッド実行後に変更されていなければならないプロパティの数を増やすなどがある。

### 原則に違反してはいけない理由

スーパータイプとサブタイプの関係を置換可能なものにしていない場合、ソフトウェアの拡張性というオブジェクト指向設計の大きなメリットを享受できなくなる。また、置換可能でないスーパークラスとサブクラスの関係を作ってしまったことによりバグが生まれる可能性もある。

## 違反している例

長方形を表すinterface`IRectangle`と、それを実装した長方形クラス`Rectangle`がある。

```typescript
interface IRectangle {
  setWidth: (width: number) => IRectangle;
  setHeight: (height: number) => IRectangle;
  getArea: () => number;
}

class Rectangle implements IRectangle {
  private width: number;
  private height: number;

  public constructor() {
    this.width = 0;
    this.height = 0;
  }

  public setWidth = (width: number) => {
    this.width = width;

    return this;
  };

  public setHeight = (height: number) => {
    this.height = height;

    return this;
  };

  public getArea = () => this.width * this.height;
}
```

`IRectangle`型のクラスは、長方形の横の長さ/縦の長さを設定するメソッド`setWidth`/`setHeight`と、長方形の面積を計算する`getArea`を実装しなければならない。サブタイプ`Rectangle`はスーパータイプ`IRectangle`を正しく実装できている。

そして、`IRectangle`型のクラスを利用する関数として、`IRectangle[]`型のインスタンスを受け取って2x4の`IRectangle[]`を返す関数`getTwoByFourRectangleList`がある。

```typescript
const getTwoByFourRectangleList = (rectangleList: IRectangle[]): IRectangle[] =>
  rectangleList.map((rectangle) => rectangle.setWidth(2).setHeight(4));
```

`getTwoByFourRectangleList`のユニットテストは以下のようになっている。

```typescript
describe("2x4の長方形を生成する", () => {
  test("生成された長方形の面積はすべて8である", () => {
    const twoByFourRectangleList = getTwoByFourRectangleList([
      new Rectangle(),
      new Rectangle(),
    ]);

    const expectedArea = 8;

    expect(
      twoByFourRectangleList.every(
        (rectangle) => rectangle.getArea() === expectedArea,
      ),
    ).toBe(true);
  });
});
```

`Rectangle`が`IRectangle`を置換可能になっており、リスコフの置換原則を満たしているため、このユニットテストは問題なくパスする。

ここに、`IRectangle`の実装として、正方形を表す`Square`クラスを追加してみる。

```typescript
class Square implements IRectangle {
  private length: number;

  public constructor() {
    this.length = 0;
  }

  public setWidth = (width: number) => {
    this.length = width;

    return this;
  };

  public setHeight = (height: number) => {
    this.length = height;

    return this;
  };

  public getArea = () => this.length * this.length;
}
```

正方形はすべての辺の長さが同じのため、プロパティを`length`のみにし、`IRectangle`型が実装しなければならないメソッド`setWidth`、`setHeight`、`getArea`をそれに合わせて実装している。

一見、すべてのメソッドが揃っておりサブタイプ`Square`はスーパータイプ`IRectangle`を正しく実装しているように見える。実際、`Square`クラス単体では何も問題はないが、スーパークラス`IRectangle`との関係を考えたとき、この関係はリスコフの置換原則に違反している。

### 違反している理由

`Square`は、`IRectangle`を置換可能になっておらず、`getTwoByFourRectangleList`のユニットテストはパスしない。

```typescript
const getTwoByFourRectangleList = (rectangleList: IRectangle[]): IRectangle[] =>
  // Squareのインスタンスが来た場合、4x4の正方形が生成されてしまう
  rectangleList.map((rectangle) => rectangle.setWidth(2).setHeight(4));

describe("2x4の長方形を生成する", () => {
  test("生成された長方形の面積はすべて8である", () => {
    const twoByFourRectangleList = getTwoByFourRectangleList([
      new Rectangle(),
      new Rectangle(),
      new Square(),
    ]);

    const expectedArea = 8;

    expect(
      twoByFourRectangleList.every(
        (rectangle) => rectangle.getArea() === expectedArea,
      ),
    ).toBe(true); // => FAIL
  });
});
```

`Rectangle`の`setWidth`は、事後条件として、「`width`が変更されていること」、「`height`が変更されていないこと」などがあると考えられます。しかし、`Square`の`setWidth`は「`length`が変更されていること」という1つの事後条件しかなく、事後条件が弱まっていると考えることができる。

### 解決策
