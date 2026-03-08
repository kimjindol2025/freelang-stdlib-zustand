# freelang-stdlib-zustand

FreeLang v2 stdlib — npm **zustand** 완전 대체  
경량 상태 관리 (순수 .fl 구현, 네이티브 .ts 별도 없음)

## 핵심 API

```fl
import "stdlib/zustand"

var useStore = create(fn(set, get) {
  return {
    count: 0,
    inc:   fn() { set({ count: get().count + 1 }) },
    dec:   fn() { set({ count: get().count - 1 }) },
    reset: fn() { set({ count: 0 }) }
  }
})

var state = getState(useStore)
state.inc()
println(getState(useStore).count)   // 1

// updater 패턴
setState(useStore, fn(s) { return { count: s.count * 2 } })

// 구독
var unsub = subscribe(useStore, fn(newState, prevState) {
  println(str(prevState.count) + " → " + str(newState.count))
})
state.inc()   // "2 → 3" 출력
unsub()       // 구독 해제

// 특정 키만 구독
var unsubKey = subscribeToKey(useStore, "count", fn(nv, pv) {
  println("count changed: " + str(pv) + " → " + str(nv))
})

// 소멸
destroy(useStore)
```

## 미들웨어

```fl
// persist: 인메모리 저장
var store = persist(create(initializer), { key: "myStore", storage: "memory" })

// devtools: 변경 로그 출력
var store = devtools(create(initializer), "CounterStore")

// immer: 불변성 패턴 (JSON 깊은 복사 방식)
var store = immer(create(initializer))
```

## subscribe 구현 방식

`listeners: map (id→fn)` 기반.  
`subscribe` 호출 시 `_nextId`로 고유 ID 발급 → `listeners[id] = fn` 저장.  
반환된 `unsubscribe` 클로저가 `capturedId`를 캡처하여 `delete(store.listeners, id)` 호출.  
`setState` 시 모든 리스너에게 `(newState, prevState)` 전달.
