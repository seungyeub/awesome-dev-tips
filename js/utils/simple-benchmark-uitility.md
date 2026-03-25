# ⏱️ JS/TS Simple Benchmark Utility

> "Don't just guess, measure it."
> 자바스크립트와 타입스크립트 환경에서 누구나 쉽고 정확하게 성능을 측정할 수 있도록 설계된 초경량 벤치마크 툴킷입니다.

---

## ✨ Key Features

* **JIT Warm-up**: 자바스크립트 엔진(V8)의 최적화 시간을 고려한 예열 기능을 내장하여 정확도를 높였습니다.
* **Statistical Analysis**: 단순 평균(Avg)뿐만 아니라 최소(Min), 최대(Max) 값을 산출하여 성능의 안정성을 확인합니다.
* **Universal Support**: 브라우저(Console)와 Node.js 환경에서 모두 동작합니다.
* **Visual Output**: `console.table`을 사용하여 측정 결과를 시각적으로 한눈에 비교할 수 있습니다.

---

## 🚀 Quick Start

### 1. 코드 복사 (Benchmark.js / .ts)

이 클래스를 프로젝트의 `utils` 폴더 등에 저장하세요.

```javascript
class SimpleBenchmark {
    constructor() {
        this.tests = [];
    }

    /**
     * 테스트 케이스 등록
     * @param {string} name - 테스트 이름
     * @param {Function} fn - 실행할 함수
     */
    add(name, fn) {
        this.tests.push({ name, fn });
        return this;
    }

    /**
     * 벤치마크 실행
     * @param {number} iterations - 반복 횟수 (기본 100)
     * @param {number} warmup - 예열 횟수 (기본 10)
     */
    async run(iterations = 100, warmup = 10) {
        console.log(`\n🏃 Benchmark Start... (Iter: ${iterations}, Warmup: ${warmup})`);
        const results = [];

        for (const test of this.tests) {
            // Warm-up
            for (let i = 0; i < warmup; i++) test.fn();

            // Measurement
            const times = [];
            for (let i = 0; i < iterations; i++) {
                const start = performance.now();
                test.fn();
                const end = performance.now();
                times.push(end - start);
            }

            const total = times.reduce((a, b) => a + b, 0);
            results.push({
                "Test Name": test.name,
                "Avg (ms)": (total / iterations).toFixed(4),
                "Min (ms)": Math.min(...times).toFixed(4),
                "Max (ms)": Math.max(...times).toFixed(4),
                "Total (ms)": total.toFixed(2)
            });
        }

        console.table(results);
        this.tests = [];
    }
}
```

---

### 2. 사용 예시
```javascipt
const bench = new SimpleBenchmark();
const items = Array.from({ length: 100000 }, (_, i) => i);

bench
    .add('Filter & Map', () => {
        items.filter(x => x % 2 === 0).map(x => x * 2);
    })
    .add('Reduce Only', () => {
        items.reduce((acc, cur) => {
            if (cur % 2 === 0) acc.push(cur * 2);
            return acc;
        }, []);
    })
    .run(50); // 50번 반복 측정
```

---

### 3. 왜 이렇게 작성해야 하나요? (Key Points)
이 코드가 "알아보기 쉽고 전문적인" 이유는 세 가지 핵심 원칙을 지켰기 때문입니다.

1. **Warm-up (예열) 과정:** 자바스크립트 엔진(V8)은 코드가 '뜨거워질(자주 실행될)' 때 비로소 최적화(JIT Compilation)를 시작합니다. 예열 없이 바로 측정하면 최적화 전의 느린 속도가 찍히게 됩니다.
2. **`console.table` 활용:** 결과를 객체 배열로 만들어 `console.table`로 출력하면 별도의 UI 없이도 한눈에 비교가 가능합니다.
3. **최소/최대값 포함:** 평균(`Avg`)만 보면 가끔 튀는 값(가비지 컬렉션 등에 의한 지연)을 놓칠 수 있습니다. `Min`과 `Max`를 같이 봐야 성능의 안정성을 알 수 있습니다.

---

## 💡 Why use this instead of console.time?
1. **JIT Compilation:** 자바스크립트는 코드가 여러 번 실행되어야 비로소 최적화됩니다.<br>`console.time`을 한 번만 쓰면 최적화 전의 속도를 재게 될 확률이 높습니다. 이 툴은 `warmup` 옵션을 통해 이 문제를 해결합니다.
2. **Outliers:** 한 번의 측정은 가비지 컬렉션(GC)이나 시스템 부하로 인해 느려질 수 있습니다.<br>여러 번의 반복을 통해 얻은 평균과 최소값이 진짜 성능입니다.

---



## 📑 올바른 벤치마킹을 위한 체크리스트

> "단 한 번의 실행 결과는 운일 수도 있습니다. 통계로 증명하세요."

### 1. 시간 측정 도구의 선택
- `Date.now()`: 밀리초 단위, 정밀도가 낮음.
- `performance.now()`: **권장**. 마이크로초(1/1000ms) 단위까지 측정 가능하며 시스템 시각 변화에 영향을 받지 않음.

### 2. 외부 요인 통제
- **가비지 컬렉션(GC)**: 루프 중간에 GC가 발생하면 시간이 확 튈 수 있음. 이를 위해 여러 번 실행 후 평균을 내는 것이 중요함.
- **백그라운드 프로세스**: 테스트 중에는 크롬 탭을 최소화하거나 다른 무거운 프로그램을 끄는 것이 좋음.

### 3. 통계의 함정 피하기
- **Outlier(이상치)**: 유독 하나만 너무 느리게 나왔다면 평균을 깎아먹습니다. `Min` 값이 실제 그 로직이 낼 수 있는 '최고의 잠재력'에 가깝습니다.

### 4. 요약
1. 예열(Warm-up)은 선택이 아닌 필수다.
2. 평균, 최소, 최대값을 모두 확인하자.
3. 시각적으로 비교하기 위해 `console.table`을 애용하자.