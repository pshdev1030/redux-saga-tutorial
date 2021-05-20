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
```javascript sagas.js
// worker Saga: 비동기 증가 태스크를 수행할겁니다.
export function* incrementAsync() {
  yield delay(1000)
  yield put({ type: 'INCREMENT' })
}

// watcher Saga: 각각의 INCREMENT_ASYNC 에 incrementAsync 태스크를 생성할겁니다.
export function* watchIncrementAsync() {
  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
}
```

redux-thunk는 함수를 dispatch하여 비동기작업을 수행하지만
redux-saga는 순수 액션 오브젝트만 dispatch한다.

delay
