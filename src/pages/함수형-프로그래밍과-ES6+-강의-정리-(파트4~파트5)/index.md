---
title: 함수형 프로그래밍과 ES6+ 강의 정리 (파트4~파트5)
date: '2019-02-15'
spoiler: The other kind of technical debt.
---

유인동님의 [함수형 프로그래밍과 ES6+ 강의](https://programmers.co.kr/learn/courses/7637?utm_source=programmers&utm_medium=banner&utm_campaign=course7637)를 보고 정리한 내용입니다.

### 파트4. map, filter, reduce

#### map

```javascript
const map = (f, iter) => {
  let res = [];
  for(const a of iter) {
    res.push(f(a));
  }
  return res;
}
```
map은 보조함수를 통해서 iterator안의 특정값을 전달하여 새로운 결과를 반환합니다.

#### 이터러블 프로토콜을 따른 map의 다형성

```javascript
[1, 2, 3].map(a => a + 1);
map(el => el.nodeName, document.querySelectorAll('*')); // 배열의 map과는 다르게 이터러블 프로토콜을 따르는 돔 리스트도 인자로 받아 map을 사용할 수 있다.

function *gen() {
  yield 2;
  if(false) yield 3;
  yield 4;
}

console.log(map(a => a * a, gen()));
// 제너레이터 함수의 결과들도 map을 할 수 있기  때문에 사실상 모든 것들을 map함수에 적용할 수 있다고 볼 수 있다.

let m = new Map();
m.set('a', 10);
m.set('b', 20);
console.log(new Map(map(([k, a]) => [k, a * 2], m)));
// {"a" => 20, "b" => 40}
```

#### filter
```javascript
const filter = (f, iter) => {
  let res = [];
  for(const a of iter) {
    if(f(a)) res.push(a);
  }
  return res;
}

console.log(filter(n => n % 2, function *() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
} ()));

// [1, 3, 5]
```

#### reduce
값을 하나로 축약하는 것

```javascript
const reduce = (f, acc, iter) => {
  if(!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for(const a of iter) {
    acc = f(acc, a);
  }
  return acc;
}

const add = (a, b) => a + b;
console.log(reduce(add, 0, [1, 2, 3, 4, 5];
// 15
console.log(add(add(add(add(add(0, 1), 2), 3), 4), 5));
// 15   
console.log(reduce(add, [1, 2, 3, 4, 5];
// 15                   
console.log(reduce(add, 1, [2, 3, 4, 5];                   
```
### 파트5. 코드를 값으로 다루어 표현력 높이기

#### go, pipe

```javascript
const go = (...args) => reduce((a, f) => f(a), args);
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);

go(
  0,
  a => a + 1,
  a => a + 10,
  a => a + 100,
  console.log
)

const f = pipe(
  a => a + 1,
  a => a + 10,
  a => a + 100);

console.log(f(0));
```
#### curry
함수를 값으로 다루면서 받아둔 함수를 원하는 시점에 평가시키는 기법

```javascript
const curry = f =>
	(a, ..._) => _.length ? f(a, ..._) : ( ..._) => f(a, ..._);
// 함수를 받아서 함수를 리턴하고 리턴된 함수가 실행되었을 때 인자가 2개 이상이라면
// 받아둔 함수를 즉시 실행하고 2개 이하라면 함수를 다시 리턴하고 그 이후에 받은 인자를 합쳐서 다시 함수를 수행한다.

const mult = curry((a, b) => a * b);
console.log(mult(3)(2));
// 6

const mult3 = mult(3);
console.log(mult3(10));
// 30
console.log(mult3(5));
// 15

```

#### go + curry를 적용하여 더 읽기 좋은 코드로 만들기

```javascript
const map = curry((f, iter) => {
  let res = [];
  for(const a of iter) {
    res.push(f(a));
  }
  return res;
});

const filter = curry((f, iter) => {
  let res = [];
  for(const a of iter) {
    if(f(a)) res.push(a);
  }
  return res;
});

const reduce = curry(f, acc, iter) => {
  if(!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for(const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});

const products = [
  {name: '반팔티', price: 15000 },
  {name: '긴팔티', price: 20000 },
  {name: '핸드폰케이스', price: 15000 },
  {name: '후드티', price: 30000 },
  {name: '바지', price: 25000 },

const add = (a, b) => a + b;
console.log(
  reduce(
    add,
    map(p => p.price,
        filter(p => p.price < 20000, products))));
// 30000
go(
  products,
  products => filter(p => p.price < 20000, products),
  products => map(p => p.price, products),
  prices => reduce(add, prices),
  console.log);
// go를 이용해 순서를 반대로 뒤집기
// 30000

go(
  products,
  filter(p => p.price < 20000),
  map(p => p.price),
  reduce(add),
  console.log);
// 함수를 부분적으로 실행하는 curry를 이용해 보다 간결한 코드만들기
// 30000
```
