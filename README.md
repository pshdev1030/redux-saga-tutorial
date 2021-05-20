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
