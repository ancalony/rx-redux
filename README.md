[![npm version](https://img.shields.io/npm/v/rx-redux.svg?style=flat-square)](https://www.npmjs.com/package/rx-redux)

rx-redux
========

A reimplementation of [redux](https://github.com/gaearon/redux) using [RxJS](https://github.com/Reactive-Extensions/RxJS).

## Why?
Reactive by default, [this makes difference](./examples/universal-counter-rx).

> If you just want to use redux with RxJS and don't care about API compatibility, see [redux-core](https://github.com/jas-chen/redux-core).

## Features
- All of the [redux APIs](https://github.com/gaearon/redux/blob/rewrite-docs/docs/Reference/API.md) implemented.
- Additionally, `store` provides 2 rx objects you can utilize:
    - `dispatcher$` is a [Subject](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/subjects/subject.md) that you can pass actions in.
    - `state$` is an [Observable](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md), a stream of states.
- And a helper function `connectAction` to stream actions to store (see example below).

## What does it look like?
``` javascript
import {createStore, combineReducers, applyMiddleware, connectAction} from 'rx-redux'
import thunkMiddleware from 'redux-thunk'
import * as reducers from './reducers'
import { render, getActionStream } from './view'

const action$ = getActionStream();

const newCreateStore = applyMiddleware(thunkMiddleware)(createStore);
const reducer = combineReducers(reducers);
const store = newCreateStore(reducer);

// stream states to view
store.state$.subscribe(state => render(state));

// stream actions to dispatcher
action$.subscribe(action => store.dispatcher$.onNext(action));
// or you can write this way
// connectAction(action$, store);
```

## Best practice to make your app all the way reactive
**Don't** do async in `Middleware`, create `RxMiddleware` instead.

This will [ease the pain to build universal apps](./examples/universal-counter-rx).

### RxMiddleware
Which wraps action stream, look like this:
```javascript
import Rx from 'rx';

export default function thunkMiddleware(getState) {
  return action => {
    if(typeof action === 'function') {
      return Rx.Observable.just(action(getState));
    }

    // Don't know how to handle this thing, pass to next rx-middleware
    return Rx.Observable.just(action);
  };
}

```

How to design `RxMiddleware`
- Get action, return [Observable](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md).
- **Must** return Observable.
  - If you don't want to return an action (eg. if counter is not odd), return a Rx.Observable.empty().

[See a basic RxMiddleware example](./examples/counter-rx)

## WIP
- Figure out how to test a Rx project (No experience before).
- Work with Hot Module Replacement.
- Work with [redux-devtools](https://github.com/gaearon/redux-devtools).
- More examples.

## Thanks
- [@xgrommx](https://github.com/xgrommx) for  submitting pull requests and suggestions.

> *Feel free to [ask questions](./issues/) or submit pull requests!*

## Inspiration
- [redux](https://github.com/gaearon/redux), learn a lot through the source code.
- [Cycle.js](http://cycle.js.org/) for the cool MVI flow.

## License
[The MIT License (MIT)](./LICENSE)
