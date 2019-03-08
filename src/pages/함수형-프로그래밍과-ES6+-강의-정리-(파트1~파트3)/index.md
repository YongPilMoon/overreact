---
title: '함수형 프로그래밍과 ES6+ 강의 정리 (파트1~파트3)'
date: '2018-12-02'
spoiler: We talk about classes, new, instanceof, prototype chains, and API design.
---

유인동님의 [함수형 프로그래밍과 ES6+ 강의](https://programmers.co.kr/learn/courses/7637?utm_source=programmers&utm_medium=banner&utm_campaign=course7637)를 보고 정리한 내용입니다.

### 파트1. 함수형 자바스크립트 기본기

#### 평가 
코드가 계산(Evaluation) 되어 값을 만드는 것
 
```javascript 
// ex)
3 + 4
=> 7

[1, 2 + 5]
=> [1, 7]
```

#### 일급
- 값으로 다룰 수 있다.
- 변수에 담을 수 있다.
- 함수의 인자로 사용될 수 있다.
- 함수의 결과로 사용될 수 있다.

```javascript
const a = 10; // 변수에 담을 수 있다.
const add10 = a => a + 10;
console.log(add10(10)); // 함수의 인자로 사용될 수 있다.
// 20  함수의 결과로 사용될 수 있다.
```

#### 일급함수
- 함수를 값으로 다룰 수 있다.
- 조합성과 추상화의 도구

자바스크립트에서 함수는 일급입니다.
```javascript
const add5 = a => a + 5; // 함수를 변수에 담을 수 있다.
const f1 = () => () => 1;
console.log(f1());
// () => 1  함수의 결과로 함수를 반환할 수 있다.

const f2 = f1(); // 함수의 결과를 다른 변수에 담을 수 있다.
```

#### 고차 함수 
함수를 값으로 다루는 함수로 크게 2가지로 분류할 수 있습니다.

##### 함수를 인자로 받아서 실행하는 함수
- apply1
```javascript
const apply1 = f => f(1);
const add2 = a => a + 2;
console.log(apply1(add2))
// 3
```
- times

```javascript
const times = (f, n) => {
  let i = -1;
  while (++i < n) f(i);
};

times(a => console.log(a + 10), 3)
// 10
// 11
// 12
```
##### 함수를 만들어 리턴하는 함수 (클로저를 만들어 리턴하는 함수)
- addMaker
```javascript
const addMaker = a => b => a + b;
const add10 = addMaker(10);
console.log(add10(5))
// 15
console.log(add10(10))
// 20
```

### 파트2. ES6에서의 순회와 이터러블/이터레이터 프로토콜

#### 기존과 달라진 ES6에서의 리스트 순회
- for i++
- for of
```javascript
const list = [1, 2, 3];
// 기존 리스트 순회
for (var i = 0; i < list.length; i++) {
  console.log(list[i]);
}
// 기존 유사배열 순회
const str = 'abc';
for (var i = 0; i < str.length; i++) {
  console.log(str[i]);
}
// ES6
for (const a of list) {
  console.log(a);
}
for (const a of str) {
  console.log(a);
}  
```

#### Array를 통해 알아보기
```javascript
const arr = [1, 2, 3];
for (const a of arr) console.log(a);
console.log(arr[Symbol.iterator]);
// f values() { [native code] }
```

#### Set을 통해 알아보기
```javascript
const set = new Set([1, 2, 3]);
for (const a of set) console.log(a);
console.log(set[Symbol.iterator]);
// f values() { [native code] }
```

#### Map을 통해 알아보기
```javascript
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
for (const a of map) console.log(a);
console.log(map[Symbol.iterator]);
// f values() { [native code] }
for (const a of map.keys()) console.log(a);
// a
// b
// c
for (const a of map.values()) console.log(a);
// 1
// 2
// 3
for (const a of map.entries()) console.log(a);
// ["a", 1]
// ["b", 2]
// ["c", 3]
```
keys, values. entries는 iterator를 리턴합니다


Array, Set, Map은 javascript 내장객체로써 이터러블/이터레이터 프로토콜을 따르고 있습니다.

#### 이터러블/이터레이터 프로토콜
- 이터러블: 이터레이터를 리턴하는 [Symbol.iterator]() 를 가진 값
- 이터레이터: { value, done } 객체를 리턴하는 next() 를 가진 값
- 이터러블/이터레이터 프로토콜: 이터러블을 for...of, 전개 연산자 등과 함께 동작하도록한 규약

```javascript
const arr = [1, 2, 3];
console.log(arr[Symbol.iterator]);
// f values() { [native code] } (Array는 이터러블 입니다)
let iterator = arr[Symbol.iterator]();
console.log(iterator.next());
// {value: 1, done: false}
console.log(iterator.next());
// {value: 2, done: false}
console.log(iterator.next());
// {value: 3, done: false}
console.log(iterator.next());
// {value: undefined, done: true}
```
Array는 [Symbol.iterator]를 키로 함수를 가지고 있고, 이 함수를 수행하면 {value, done}을 리턴하는 next()를 가진 객체(itorator)가 리턴되기 때문에 for...of 문에서 잘동작하고 이터러블/이터레이터 프로토콜을 따른다고 할 수 있습니다.
for...of 문에서는 value에 있는 값을 변수에 담아 사용하고 done 이 true이면 for 문을 빠져 나옵니다.

#### 사용자 정의 이터러블을 통해 알아보기
```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
	return {
      next() {
        return i === 0 ? { done: true} : { value: i--, done: false };
    	}
	}
  }
};

let iterator = iterable[Symbol.iterator]();
for (const a of iterable) console.log(a);
// 3
// 2
// 1
```
위에 구현된 이터러블에서 반환된 iterator는 다시 for...of 구문을 통해 순회 할 수 없습니다. 이는 well formed iterator가 아니기 때문입니다.
```javascript
for(const a of iterator) console.log(a);
// Uncaught TypeError: iterator is not iterable
```

well formed iterator로 만들어 주기 위해선 리턴되는 iterator에 [Symbol.itorator] 키로 자기자신을 리턴하는 함수를 구현해 주면 됩니다.

```javascript
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
	return {
      next() {
        return i === 0 ? { done: true} : { value: i--, done: false };
    	}
      [Symbol.iterator]() { return this } // well formed iterator
	}
  }
};
```

이렇게 하면 반환된 iterator로 순회를 할 수 있고 value의 값이 그대로 보존되기 때문에 iterator.next()를 수행하여 일정 부분을 진행 한 후에도 for...of문 구문에서 순회할 수 있습니다.

#### 전개연산자
```javascript
const a = [1, 2];
log([...a, ...[3, 4]);
// [1, 2, 3, 4]
```
전개연산자 역시 이터러블 프로토콜을 따르는 객체를 펼칩니다.

### 파트3. 제너레이터와 이터레이터

#### 제너레이터/이터레이터
제너레이터: 이터레이터이자 이터러블을 생성하는 함수(well formed 이터레이터를 리턴하는 함수)

```javascript
function *gen() {
  yield 1;
  yield 2;
  yield 3;
  // return 100;
  // 마지막 done이 ture일 때 value로 return 값을 전달 할 수 있다.
}

let iter = gen();
console.log(iter[Symbol.iterator]() == iter); // true
console.log(iter.next()); // {value:1, done: false}
console.log(iter.next()); // {value:2, done: false}
console.log(iter.next()); // {value:3, done: false}
console.log(iter.next()); // {value:undefined, done: true}

for(const a of gen()) log(a);
```
제너레이터는 `if(false) yield 2`처럼 순회하는 값을 문장으로 표현한것이라고도 볼 수 있습니다. 
이는 자바스크립트에서 어떠한 상태나 값도 제너레이터를 통해  순회할 수 있는 값으로 만들 수 있다는 것을 의미합니다.

#### odds
```javascript
function *infinity(i = 0) {
  while(true) yield i++
}
function *limit(l, iter) {
  for (const a of iter) {
    yield a;
    if(a == l) return;
  }
}
function *odds(l) {
  for (const a of limit(l, infinity(1)) {
  	if(i % 2) yield i;
  }
}

let iter2 = odds(10);
console.log(iter2.next());
console.log(iter2.next());
console.log(iter2.next());
console.log(iter2.next());
```

#### for of, 전개 연산자, 구조 분해, 나머지 연산자
```javascript
console.log(...odds(10)); // 1 3 5 7 9 
console.log([...odds(10), ...odds(20)]);
// [1, 3, 5, 7, 9, 1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
const [head, ...tail] = odds(5);
console.log(head); // 1
console.log(tail); // [3, 5]
```
