# 🛠️ redux-saga-promise-actions 📦

Simple to use library which allows to return promise after an action is dispatched.

## 📥 Instalation

`npm install tomekkleszcz/redux-saga-promise-actions`

or 

`yarn add tomekkleszcz/redux-saga-promise-actions`

## 🤔 Why this library does even exists?

In most of my projects I use [Formik](https://github.com/formium/formik) for handling the forms
and [Redux saga](https://github.com/redux-saga/redux-saga/) for handling async tasks. Unfortunately
it is not possible to control Formik state from saga, so there is a need to return the information
if the async task was handled successfully or not to the component.

One of the possible solutions is to store the information if the form is submitting and its errors in Redux store.
However it is [not recommended to store the UI state in Redux store](https://redux.js.org/faq/organizing-state#should-i-put-form-state-or-other-ui-state-in-my-store).
If I would store the submitting and errors in the Redux store why would I use Formik?

The second possible solution is to pass the Formik helpers (i.e. setSubmitting or setErrors) as an action props.
Or... You can use this library instead. 🙂

## 🧰 Usage

### Include promise middleware

First of all, you have to add the promise middleware to your Redux store middleware chain.

🚨 The promise middleware should be before the saga middleware in the chain.

```typescript
//Redux store
import {createStore, applyMiddleware} from 'redux';

//Middlewares
import {promiseMiddleware} from 'tomekkleszcz/redux-saga-promise-actions';
import createSagaMiddleware from 'redux-saga';

const sagaMiddleware = createSagaMiddleware();

const store = createStore(rootReducer, {}, applyMiddleware(promiseMiddleware, sagaMiddleware));
```

### Declare promise actions

Then, you can declare the promise action. The library uses [typesafe-actions](https://github.com/piotrwitek/typesafe-actions)
under the hood so declaring actions is easy as it can be.

#### Javascript
```javascript
//Action creators
import {createPromiseAction} from 'tomekkleszcz/redux-saga-promise-actions';

const signUp = createPromiseAction(
    'SIGN_UP_REQUEST', 
    'SIGN_UP_SUCCESS', 
    'SIGN_UP_FAILURE'
)();
```

#### Typescript
```typescript
//Action creators
import {createPromiseAction} from 'tomekkleszcz/redux-saga-promise-actions';

const signUp = createPromiseAction(
    'SIGN_UP_REQUEST', 
    'SIGN_UP_SUCCESS', 
    'SIGN_UP_FAILURE'
)<
    {email: string, password: string},
    {accessToken: string, refreshToken: string},
    {email: string | null, password: string | null}
>();
```

### Handle actions

Finally, you can handle promise actions in your code.

#### Component
```javascript
dispatch(signUp.request({email: 'example@example.org', password: 'TestPassword'}))
    .then(() => alert('Success!'))
    .catch(err => console.error(err));
```

#### Saga

```javascript
//Promise actions
import {resolvePromiseAction, rejectPromiseActions} from "tomekkleszcz/redux-saga-promise-actions";

function* signUp(action) {
    try {
        // Send the request

        yield call(resolvePromiseAction, action, response);
    } catch(err) {
        yield call(rejectPromiseActions, action, err);
    }
}
```

Depending on the result the returned promise from `dispatch(...)` will either resolve or reject.