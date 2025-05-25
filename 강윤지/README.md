### **1. 클로저 (Closure)**

### **1.1 클로저 개념의 정석**

- 클로저란?
  함수가 선언될 당시의 외부 변수 환경을 기억하고, 그 환경에 계속 접근할 수 있는 함수.
  즉, 함수가 자신의 렉시컬 환경을 ‘기억’하는 것.
- 렉시컬 스코프 vs 다이나믹 스코프
  - 렉시컬 스코프: 선언된 위치 기준으로 스코프가 결정됨 (자바스크립트는 이 방식 사용)
  - 다이나믹 스코프: 호출된 위치 기준으로 스코프가 결정됨
- 실행 컨텍스트와 클로저
  - 실행 컨텍스트가 종료되어도 해당 컨텍스트의 변수 환경이 참조되고 있다면 GC 대상이 아님
  - 클로저는 이러한 실행 컨텍스트의 환경 정보를 유지함

### **1.2 실행 컨텍스트와 스코프 체인**

- [[Scope]], [[Environment]] 구조
  - 함수는 생성 시점에 [[Environment]] 내부 슬롯에 자신이 선언된 환경을 저장
  - 실행 시 이 환경은 스코프 체인을 따라 변수 탐색에 사용됨
- scope chain vs prototype chain 비교
  - 스코프 체인: 변수 식별자 탐색을 위한 체인 (렉시컬 환경)
  - 프로토타입 체인: 객체의 속성 탐색 체인 (상속 구조)
- 함수가 참조하는 외부 변수는 어떻게 살아있는가?
  - 클로저가 외부 변수를 참조하면, JS 엔진은 해당 변수를 힙 메모리에 유지함 (GC 대상 제외)

### **1.3 클로저의 메모리와 가비지 컬렉션**

- 클로저는 외부 변수에 대한 참조를 유지하여 변수 수명 연장
- 메모리 누수는 클로저가 더 이상 필요하지 않지만 참조를 유지할 경우 발생
- 해결 전략
  - 클로저 내부 참조를 null로 제거
  - 사용하지 않는 이벤트 리스너 해제

### **1.4 실전 예제 및 면접 질문**

- 클로저가 왜 메모리 누수를 유발할 수 있나요?
  클로저는 함수가 선언될 당시의 외부 변수 환경(Lexical Environment)을 참조 상태로 유지합니다. 이때 클로저 내부에서 외부 변수를 계속 참조하고 있으면, 해당 변수가 가비지 컬렉션 대상이 되지 않아서 메모리가 해제되지 않습니다.
  예를 들어, 이벤트 리스너나 타이머 함수 안에서 클로저로 외부 변수를 참조한 뒤, 해당 리스너를 제거하지 않으면 필요 없는 메모리도 계속 유지되며 누수가 발생합니다.
  이를 해결하기 위해 이벤트를 제거하거나, 클로저 참조를 명시적으로 null로 초기화 하거나, WeakMap, WeakRef 등을 활용할 수 있습니다.
- 클로저와 캡슐화의 관계를 설명해보세요.
  자바스크립트는 클래스의 private 키워드가 도입되기 전까지 접근 제한자가 없었기 때문에, 클로저를 활용한 정보 은닉이 중요한 패턴이었습니다.
  클로저는 외부에서 직접 접근할 수 없는 지역 변수를 함수 내부에 유지할 수 있으므로, 이 변수를 외부에서 조작하지 못하도록 감출 수 있습니다. 이를 통해 상태 보존과 캡슐화를 구현할 수 있습니다.
  이때 변수는 클로저를 통해 은닉된 상태로 유지되며, getter/setter 역할을 하는 메서드만 노출함으로써 객체지향의 캡슐화 효과를 구현할 수 있습니다.

### **2. 이벤트 루프 (Event Loop)**

### **2.1 이벤트 루프 동작 원리**

![image.png](attachment:1fa3199a-7ba0-4c12-997f-a8cba89f93a9:image.png)

- JS는 싱글스레드이기에, 동시에 하나의 작업만 실행 가능
- 주요 구성 요소:
  - Call Stack: 실행 중인 함수의 스택
  - Web APIs: 타이머, DOM, AJAX 등 비동기 API
  - Callback Queue (Task Queue = Macrotask Queue): setTimeout, setInterval 등의 콜백 저장소
  - Microtask Queue: Promise.then, queueMicrotask, MutationObserver 등

### **2.2 이벤트 루프의 순환 구조**

1. Call Stack 에서 하나씩 실행
2. Call Stack이 비면 → MicroTask Queue에서 하나 실행
3. MicroTask Queue이 비면 → Microtask Queue를 에서 하나 실행

- **우선순위 예시**
  ```jsx
  setTimeout(() => console.log("timeout"), 0);
  Promise.resolve().then(() => console.log("promise"));
  ```
  출력 순서: promise → timeout
  https://wikidocs.net/251949

### **2.3 브라우저와 Node.js의 차이**

- 브라우저의 Event Loop 구조 → 2.1 참고
- Node.js의 libuv 기반 Event Loop 구조
  - Node.js는 브라우저와 달리 내부적으로 libuv라는 C++ 기반 라이브러리를 사용하여 이벤트 루프를 관리함
  - I/O 중심의 서버 환경에 맞게 더 복잡한 구조로 이루어져 있으며, **총 6단계(Phase)**로 구분됩니다:
  | Phase                 | 설명                                           |
  | --------------------- | ---------------------------------------------- |
  | **timers**            | `setTimeout`, `setInterval`의 콜백 실행        |
  | **pending callbacks** | 일부 시스템 작업의 콜백 처리 (예: TCP 오류 등) |
  | **idle, prepare**     | 내부적으로 사용됨 (사용자 코드에서 사용 불가)  |
  | **poll**              | I/O 이벤트 체크 및 대기                        |
  | **check**             | `setImmediate` 콜백 실행                       |
  | **close callbacks**   | 소켓 등 종료된 리소스의 콜백 실행              |
  - Microtask (Next Tick vs Promise)
    - process.nextTick()은 모든 Phase보다 먼저 실행됨 (Node.js 전용)
    - Promise.then()은 각 Phase 뒤에 실행되는 Microtask로 처리됨
- 비교
  | 항목             | 브라우저                     | Node.js                         |
  | ---------------- | ---------------------------- | ------------------------------- |
  | 실행 환경        | 사용자 UI 중심               | 서버 사이드, I/O 중심           |
  | Task 종류        | setTimeout, setInterval 등   | setTimeout, setImmediate 등     |
  | Microtask 종류   | Promise.then, queueMicrotask | Promise.then, process.nextTick  |
  | 이벤트 루프 구조 | 비교적 단순                  | 6가지 Phase로 구성된 libuv 구조 |
  | 렌더링 단계      | 있음 (UI 업데이트 포함)      | 없음 (UI 렌더링 불필요)         |

### **2.4 실전 예제 및 면접 질문**

- setTimeout(fn, 0)이 바로 실행되지 않는 이유는?
  setTimeout(fn, 0)은 즉시 실행되는 것처럼 보이지만, 실제로는 JS 이벤트 루프의 기본 순서 때문에 Macrotask Queue에 등록되어 Call Stack이 완전히 비워진 후에 실행됩니다. 따라서 동기 코드, Microtask(Promise 등)가 먼저 처리된 후 실행됩니다.
- Promise의 then과 setTimeout의 실행 순서는?
  Promise.then()은 Microtask Queue에 등록되고, setTimeout은 Macrotask Queue에 등록됩니다. 이벤트 루프는 한 Task가 끝날 때마다 Microtask Queue를 모두 처리하고 다음 Task로 넘어가기 때문에, Promise.then()이 항상 setTimeout보다 먼저 실행됩니다.
- Node.js의 이벤트 루프 단계 중 poll phase란?
  Node.js에서 poll 단계는 I/O 이벤트를 기다리고 처리하는 단계입니다. 예를 들어, 파일 읽기, 네트워크 요청 같은 작업의 콜백을 이 단계에서 실행합니다. 또한, 이 단계에서 setTimeout이 아직 도래하지 않은 경우, poll 단계가 일정 시간 동안 block되어 대기 할 수도 있습니다.
- async/await 가 이벤트 루프, 콜스택 등등에서 어떻게 동작하는지
  async 함수는 항상 Promise를 반환하며, 내부에서 await을 만나면 실행을 일시 중단하고, 해당 Promise가 해결되면 Microtask Queue에 등록되어 재개됩니다. 따라서 await 이후 코드는 콜스택이 비워지고 Microtask가 처리되는 타이밍에 실행됩니다.
  ```jsx
  console.log('1');

  async function asyncFunc() {
    console.log('2');
    await Promise.resolve();
    console.log('3');
  }

  asyncFunc();

  console.log('4');

  -> 1 2 4 3
  ```
- Promise + setTimeout + async/await 혼합 실행 순서 예측
  ```jsx
  console.log('1');

  setTimeout(() => console.log('2'), 0);

  Promise.resolve()
    .then(() => console.log('3'))
    .then(() => console.log('4'));

  async function asyncFunc() {
    console.log('5');
    await Promise.resolve();
    console.log('6');
  }

  asyncFunc();

  console.log('7');

  -> 1 5 7 3 4 6 2
  ```

### **3. 가비지 컬렉터 (Garbage Collector)**

### **3.1 JS의 메모리 구조**

- 스택(Stack): 원시값, 실행 컨텍스트 저장 (고속 접근)
- 힙(Heap): 객체, 배열, 함수 등 참조 타입 저장
- 실행 컨텍스트가 종료되면 스택 메모리는 해제되지만, 참조가 남아 있으면 힙은 유지

### **3.2 GC 알고리즘**

- **Reference Counting(참조 카운팅)**
  - 객체가 몇 개의 참조(reference)를 받고 있는지 계산
  - 참조 수가 0이 되면 해당 객체는 더 이상 사용되지 않는 것으로 판단 → 해제
  - 장점: 구현이 간단하고 빠름
  - 단점: 순환 잠초 문제를 해결하지 못함
  ```jsx
  function foo() {
    const a = {};
    const b = {};
    a.b = b;
    b.a = a; // 순환 참조 발생 → 해제되지 않음 (옛날 방식 기준)
  }
  ```
- **Mark and Sweep(마크 앤 스윕)**
  - 도달 가능한 객체를 “마크”하고, 나머지를 “쓸어냄”
  - 단계:
    1. 루트 객체(예: 글로벌 객체, 실행 컨텍스트 등)에서 시작
    2. 루트에서 **참조 가능한 객체를 마크(Mark)**함
    3. 마크되지 않은 객체는 쓸어냄(Sweep) → GC 대상
  - 장점:
    - 참조 카운팅의 순환 참조 문제 해결 가능
  - 단점:
    - 마크와 스윕 시 전체 힙을 탐색하므로 GC 동안 멈춤(Pause) 발생
  - 요약:
    - 현재 대부분의 JS 엔진(V8 포함)은 이 방식을 기반으로 최적화된 방식 사용
- Generational GC (V8)
  - 객체를 **수명(lifetime)**에 따라 세대로 분류하고, 세대별로 다른 전략을 적용하여 성능을 최적화
  | 세대                         | 설명                           |
  | ---------------------------- | ------------------------------ |
  | **New (Young) Generation**   | 새로 생성된 객체 (수명이 짧음) |
  | **Old (Tenured) Generation** | 오래 살아남은 객체 (수명이 김) |
  V8에서는 다음과 같은 전략을 사용합니다:
  - Scavenge (New Generation)
    - Mark-and-Copy 기반의 알고리즘
    - 대부분 금방 수거 가능하므로 빠르고 효율적
    - "Minor GC"라 불림
  - Mark-Compact (Old Generation)
    - Mark 후 사용 중인 객체를 힙 앞쪽으로 압축(compact)
    - "Major GC"로, 더 무겁고 멈춤 시간도 더 긺

### **3.3 실전 예제 및 면접 질문**

- JS에서 메모리 누수가 발생하는 사례는?
  - 클로저 내부에서 외부 변수를 계속 참조할 때, 이벤트 리스너를 제거하지 않았을 때, DOM 요소를 제거했지만 JS 변수로 계속 참조할 때, 전역 변수 오용으로 불필요한 객체가 계속 살아있을 때
- GC가 발생하는 타이밍은 언제인가요?
  - GC는 명확한 시점 없이, JS 엔진이 메모리가 부족하다고 판단할 때 자동으로 발생합니다. 일반적으로는 다음과 같은 시점에 트리거될 수 있습니다:
    - NewSpace가 가득 찼을 때 (Minor GC)
    - 오래된 객체가 누적되어 OldSpace가 커졌을 때 (Major GC)
    - JS 엔진이 메모리 압박을 감지했을 때
  - 개발자가 직접 GC를 호출할 수는 없고, 대신 메모리 구조를 최적화하여 GC 발생 빈도를 줄일 수 있습니다.
