## Saga를 실행하기 위해서 해야할 일
1. Sagas 리스트와 함께 Saga 미들웨어를 만들어라
2. Saga 미들웨어를 Redux 스토어에 연결하세요
```javascript
import {createStore,applyMiddleware} from 'redux';
import createSagaMiddleware from 'redux-saga';

const sagaMiddleware=createSagaMiddleware();
const store=createStore(
    reducer,
    applyMiddleware(sagaMiddleware)
)

sagaMiddleware.run(helloSaga);
```


## 비동기 호출하기
main.js
```javascript
function render() {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => action('INCREMENT')}
      onDecrement={() => action('DECREMENT')} 
      onIncrementAsync={() => action('INCREMENT_ASYNC')} />,
    document.getElementById('root')
  )
}
```
redux-thunk는 함수를 dispatch하여 비동기작업을 수행하지만
redux-saga는 순수 액션 오브젝트만 dispatch한다.

sagas.js
```javascript
import {delay}from 'redux-saga';
import {put,takeEvery} from 'redux-saga/effects';

// worker Saga: 비동기 증가 태스크를 수행할겁니다.
export function* incrementAsync() {
  yield delay(1000);
  yield put({ type: 'INCREMENT' })
}

// watcher Saga: 각각의 INCREMENT_ASYNC 에 incrementAsync 태스크를 생성할겁니다.
export function* watchIncrementAsync() {
  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}
```

Saga= 제너레이터 함수로 구현되어서 이벤트를 구독해두는 이벤트리스너이다. 
yield의 상태값으로 이벤트를 알림
yield 뒤의 값들은 미들웨어에 의해서 해석되는 명령의 종류이다.

delay 함수는 설정된 시간 이후에 resolve를 하는 Promise객체를 리턴한다.
이는 제너레이터를 정지하는데에 사용된다.

Promise가 미들웨어에 yield될 때 미들웨어는 Promise가 끝날 떄 까지 Saga를 일시정지 시킨다.

위의 예시에서 `incrementAsync` Saga는 1초 후에 일어날 `delay`의 resolve에 의해 Promise가 리턴될 때 까지 정지되어 있는다.

Promise가 한번 resolve되고나면 미들웨어는 Saga를 다시 작동시키면서, 다음 yield까지 코드를 실행한다.

다음 상태는 미들웨어에게 `INCREMENT` 액션을 dispatch하게 알려주는 put({type:'INCREMENT'})의 결과 객체가 된다.

`put`은 이펙트라고 부르는 예중 하나이다.
이펙트는 미들웨어에 의해 수행되는 명령을 담고있는 자바스크립트 객체이다.
미들웨어가 Saga에 의해 yield된 이펙트를 받을 때, Saga는 이펙트가 수행될 때까지 정지되어 있다.

`watchIncrementAsync`는 dispatch된 `INCREMENT_ASYNC` 액션을 바라보고, 매번 incrementAsync를 실행하기 위해 redux-saga 패키지가 제공하는 takeEvery 헬퍼 함수를 사용했다.

```javascript
// 모든 Saga들을 한번에 시작하기 위한 단일 entry point 입니다.
export default function* rootSaga() {
  yield all([
    helloSaga(),
    watchIncrementAsync()
  ])
}
```
rootSaga는 helloSaga와 watchIncrementAsync Saga가 호출된 결과의 배열을 yield한다.
이것은 rootsaga에서 두 제너레이터의 이벤트를 구독하고 실행해주는 것이다.
또한 생성된 두 제너레이터가 병렬로 시작된다는 것을 의미한다.

```javascript
import rootSaga from './sagas'

const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
sagaMiddleware.run(rootSaga)
```
sagaMiddleware.run을 통해 rootSaga를 등록한다.


## 테스트 가능한 코드 만들기


