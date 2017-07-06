# React & Redux in TypeScript - Static Typing Guide
一个完善的指导, 静态类型定义 "React & Redux " 应用 使用 Typescript

> 如果觉得这个有用, 想知道更多? 给个星吧

## 有关 Typescript v2.3 (https://github.com/Microsoft/TypeScript/wiki/Roadmap)

### 引言

这个指导的目标是给 Typescript 的解释器以严格的设置 , 这贵提供完全的最好的类型安全的 DX.
通过一小部分的努力 添加明确的类型注释在 必须并且大部分时间利用 智能的 类型推断(Type Inference).
我们将会获得:
  - 和强类型语言相同的重构能力(不是类似 Webstorm IDE 提供的 "stringly" 的类型定义)
  - 当你改变你的代码时, 可以准确洞察受影响的整个代码仓库(会展示所有的引用到的代码)
  - 当实现新的功能(编译验证所有的 props 有注入到连接上的 component,action creator 参数, payload 对象结构以及传递给 reducer的 state/action 对象 - 这会展示所有可能的 JS 错误)时,最小化错误影响的代码表面积

这个强力的功能可以帮助你在 通过重构改进你的代码, 最小抽象层级 以及 清除未使用的代码时更加的轻松和简单, 还会增强你的工作不会破坏已有的生产代码的信心.

> NB: 这个 构建体系(setup) 将会带给你最多的好处, 如果你是工作在一个改动非常频繁的项目, 并且没有办法提供单元测试(因为经常更改,它们的花费也会很大). 我成功的从这个 构建体系 中获益, 没有写任何一行单元测试的代码, 在 a decent size 的 React/Redux 单页应用中, 并且 0 生产错误. 唯一可能引起错误的源头则是 网络 ,(比如 失败的 API 调用) 但这是容易控制的.                更多的好处在于提供了一套声明的接口来描述你的 API contracts, 你将从任何可能 从null/undefined 导致的异常中获得解救 , 这些来自你调用的API的响应.

目标:
 * 完全的类型安全, 剔除所有的 `any` type
 * 借助类型推断(Type Inference) 达到最小化手工类型注释
 * 用简单的工具函数,通过泛型和高级类型功能来来减少模板代码

---------------
Table of Contents

----------

---

# React

---


## Stateless Component

- 无状态组件样板代码
  
```javascript
  import * as React from 'react';

    type Props = {
      className?: string,
      style?: React.CSSProperties,
    };

    const MyComponent: React.StatelessComponent<Props> = (props) => {
      const { children, ...restProps } = props;
      return (
        <div {...restProps} >
          {children}
        </div>
      );
    };

  export default MyComponent;
```

---

Class Component

- 类组件样板代码

```tsx
import * as React from 'react';

  type Props = {
    className?: string,
    style?: React.CSSProperties,
    initialCount?: number,
  };

  type State = {
    counter: number,
  };

  class MyComponent extends React.Component<Props, State> {
    // 默认 props 使用 属性初始化工具
    static defaultProps: Partial<Props> = {
      className: 'default-class',
    };

    // 初始化 state 属性
    state: State = {
      counter: this.props.initialCount || 0,
    };

    // 生命周期方法正常声明
    componentDidMount() {
      console.log('Mounted!');
    }

    increaseCounter = () => { this.setState({ counter: this.state.counter + 1 }); };

    render() {
      const { children, initialCount, ...restProps } = this.props;

      return (
        <div {...restProps} onClick={this.increaseCounter} >
          Clicks: {this.state.counter}
          <hr />
          {children}
        </div>
      );
    }
  }

export default MyComponent;
```

---

泛型(Generic) 组件

  - 模板代码

```tsx
type GenericListProps<T> = {
  dataSource: T[],
  itemRenderer: (item: T) => JSX.Element,
};

class GenericList<T> extends React.Component<GenericListProps<T>, {}> {
  render() {
    const { dataSource, itemRenderer } = this.props;

    return (
      <div>
        {dataSource.map(itemRenderer)}      
      </div>
    );
    }
  }

```


  ```tsx
  interface IUser {
  id: string,
  name: string,
  }

  type UserItemProps = {
    dataSource: IUser,
  }; 

  cosnt UserItem: React.StatelessComponent<UserItemProps> = ({id, name}) => {
    return (
      <div>
        <b>{name}</b>
        <small>({id})</small>
      </div>
    );
  }

  ```

  ```tsx
  const UserList = class extends Table<IUser> { };

  const users = [{
    id: 'uuidv4',
    name: 'Dude',
  }];

  ReactDOM.render(
    <UserList
      dataSource={users}
      itemRenderer={(item) => <UserItem key={item.id} dataSource={item} />} // <- notice that "item" has inferred "IUser" type
    >    
    </UserList>
  );

  ```


  ---

# 高阶组件

- 包裹并装饰输入组件, 返回一个新的组件
- 新的 组件 会继承输入 组件和 `HOC` 组合后的 Props
- 使用 类型推断会自动的计算 Props 接口
- 通过Wrapped Component过滤出装饰的 props 并只传递关联的props
- 允许无状态以及普通的组件

```tsx
// controls/button.tsx
import * as React from 'react';
import { Button } from 'antd';

type Props = {
  className?: string,
  autoFocus?: boolean,
  htmlType?: typeof Button.prototype.props.htmlType,
  type?: typeof Button.prototype.props.type,
};

const ButtonControl: React.StatelessComponent<Props> = (props) => {
  const { children, ...restProps } = props;

  return (
    <Button {...restProps} >
      {children}
    </Button>
  );
};

export default ButtonControl;
```


``` tsx
// decorators/with-form-item.tsx
import * as React from 'react';
import { Form } from 'antd';
const FormItem = Form.Item;

type BaseProps = {
};

type HOCProps = FormItemProps & {
  error?: string;
};

type FormItemProps = {
  label?: typeof FormItem.prototype.props.label;
  labelCol?: typeof FormItem.prototype.props.labelCol;
  wrapperCol?: typeof FormItem.prototype.props.wrapperCol;
  required?: typeof FormItem.prototype.props.required;
  help?: typeof FormItem.prototype.props.help;
  validateStatus?: typeof FormItem.prototype.props.validateStatus;
  colon?: typeof FormItem.prototype.props.colon;
};

export function withFormItem<WrappedComponentProps extends BaseProps>(
  WrappedComponent:
    React.StatelessComponent<WrappedComponentProps> | React.ComponentClass<WrappedComponentProps>,
) {
  const HOC: React.StatelessComponent<HOCProps & WrappedComponentProps> =
    (props) => {
      const {
        label, labelCol, wrapperCol, required, help, validateStatus, colon,
        error, ...passThroughProps,
      } = props as HOCProps;

      // filtering out empty decorator props in functional style
      const formItemProps: FormItemProps = Object.entries({
        label, labelCol, wrapperCol, required, help, validateStatus, colon,
      }).reduce((definedProps: any, [key, value]) => {
        if (value !== undefined) { definedProps[key] = value; }
        return definedProps;
      }, {});
      
      // injecting additional props based on condition
      if (error) {
        formItemProps.help = error;
        formItemProps.validateStatus = 'error';
      }

      return (
        <FormItem {...formItemProps} hasFeedback={true} >
          <WrappedComponent {...passThroughProps as any} />
        </FormItem>
      );
    };

  return HOC;
}

```


```tsx

// components/consumer-component.tsx
...
import { Button, Input } from '../controls';
import { withFormItem, withFieldState } from '../decorators';

// 你可以使用装饰器来创建特殊的组件
const
  ButtonField = withFormItem(Button);
 
// 你可以利用函数组合来组合多个 装饰器
const
  InputFieldWithState = withFormItem(withFieldState(Input));

// 增强的组件将会继承自 Base 组件以及所有提供的 HOC 的 props
<ButtonField type="primary" htmlType="submit" wrapperCol={{ offset: 4, span: 12 }} autoFocus={true} >
  Next Step
</ButtonField>
...
<InputFieldWithState {...formFieldLayout} label="Type" required={true} autoFocus={true} 
  fieldState={configurationTypeFieldState} error={configurationTypeFieldState.error}
/>
...
 
// 你可以使用功能性包比如 ramda 或者 lodash 来获得更好的功能组合, 比如:
const
  InputFieldWithState = compose(withFormItem, withFieldStateInput)(Input);
// 注意: 考虑该组合函数是否需要特意声明, 否则你将失去类型推断

```

---

# Redux Connected Component

> 注意: 类型推断在 `connect` 函数的类型声明中并没有提供完整安全, 这将导致无法利用类型推导来自动计算出应有的 Props 接口, 比如上面的例子.

> 以下的解决方案,并不完美, 我仍在研究之中看能否将其改进, 请稍后回来看或者你可以提供更好的解决方案

- 这个解决方案使用类型推断来从 `mapStaeToProps`函数中获得 Props 的类型
- 最小化手动声明并维护 注入 `connect` 帮助函数后的 Props 类型
- 使用 `returntypeof` 帮助函数, 因为 Typescript 目前并不支持此种功能 (https://github.com/piotrwitek/react-redux-typescript#returntypeof-polyfill)[https://github.com/piotrwitek/react-redux-typescript#returntypeof-polyfill]
- 真实案例: (https://github.com/piotrwitek/react-redux-typescript-starter-kit/blob/ef2cf6b5a2e71c55e18ed1e250b8f7cadea8f965/src/containers/currency-converter-container/index.tsx)[https://github.com/piotrwitek/react-redux-typescript-starter-kit/blob/ef2cf6b5a2e71c55e18ed1e250b8f7cadea8f965/src/containers/currency-converter-container/index.tsx]


```tsx
import { returntypeof } from 'react-redux-typescript';

import { RootState } from '../../store/types';
import { increaseCounter, changeBaseCurrency } from '../../store/action-creators';
import { getCurrencies } from '../../store/state/currency-rates/selectors';

const mapStateToProps = (rootState: RootState) => ({
  counter: rootState.counter,
  baseCurrency: rootState.baseCurrency,
  currencies: getCurrencies(rootState),
});
const dispatchToProps = {
  increaseCounter: increaseCounter,
  changeBaseCurrency: changeBaseCurrency,
};

// Props types inferred from mapStateToProps & dispatchToProps
const stateProps = returntypeof(mapStateToProps);
type Props = typeof stateProps & typeof dispatchToProps;

class CurrencyConverterContainer extends React.Component<Props, {}> {
  handleInputBlur = (ev: React.FocusEvent<HTMLInputElement>) => {
    const intValue = parseInt(ev.currentTarget.value, 10);
    this.props.increaseCounter(intValue); // number
  }
  
  handleSelectChange = (ev: React.ChangeEvent<HTMLSelectElement>) => {
    this.props.changeBaseCurrency(ev.target.value); // string
  }
  
  render() {
    const { counter, baseCurrency, currencies } = this.props; // number, string, string[]
    
    return (
      <section>
        <input type="text" value={counter} onBlur={handleInputBlur} ... />
        <select value={baseCurrency} onChange={handleSelectChange} ... >
          {currencies.map(currency =>
            <option key={currency}>{currency}</option>,
          )}
        </select>
        ...
      </section>
    );
  }
}

export default connect(mapStateToProps, dispatchToProps)(CurrencyConverterContainer);

```


---

# Redux

---

## Actions

## - KISS Style

这一段我将聚集于 KISS 原则 , 并远离特有的抽象, 这一点可以在其他许多的 Typescript Redux 的指导文章中发现 , 将尽可能的于Javascript的使用方式相同但仍然可以从静态类型获益:

  - 经典的基于常量的类型
  - 非常接近JS的标准用法
  - 标准的模板代码
  - 可以输出 action 类型和 action creators类型, 以便在其他模块比如 `redux-saga`或`redux-observable`中复用

```tsx

// Action Types
export const INCREASE_COUNTER = 'INCREASE_COUNTER';
export const CHANGE_BASE_CURRENCY = 'CHANGE_BASE_CURRENCY';

// Action Creators
export const actionCreators = {
  increaseCounter: () => ({
    type: INCREASE_COUNTER as typeof INCREASE_COUNTER,
  }),
  changeBaseCurrency: (payload: string) => ({
    type: CHANGE_BASE_CURRENCY as typeof CHANGE_BASE_CURRENCY, payload,
  }),
}

// Examples
store.dispatch(actionCreators.increaseCounter(4)); // Error: Supplied parameters do not match any signature of call target. 
store.dispatch(actionCreators.increaseCounter()); // OK => { type: "INCREASE_COUNTER" }

store.dispatch(actionCreators.changeBaseCurrency()); // Error: Supplied parameters do not match any signature of call target. 
store.dispatch(actionCreators.changeBaseCurrency('USD')); // OK => { type: "CHANGE_BASE_CURRENCY", payload: 'USD' }
```

## - DRY Style

这是更遵循 DRY 原则的途径, 我将引入一个简单的帮助工厂函数来自动为 action creators 添加类型. 这里的好处在于我们可以减少重复的模板代码. 这更易于我们在其他模块中复用 action creators:

  - 使用帮助工厂函数来自动创建类型化的 action creators - (源码将会被关联)
  - 相较 KISS 风格, 具有更少的重复和模板代码
  - 易于在其他模块中 例如 `redux-saga` or `redux-observable` 使用

```tsx
import { createActionCreator } from 'react-redux-typescript';

// Action Creators
export const actionCreators = {
  increaseCounter: createActionCreator('INCREASE_COUNTER'), // { type: "INCREASE_COUNTER" }
  changeBaseCurrency: createActionCreator('CHANGE_BASE_CURRENCY', (payload: string) => payload), // { type: "CHANGE_BASE_CURRENCY", payload: string }
  showNotification: createActionCreator('SHOW_NOTIFICATION', (payload: string, meta?: { type: string }) => payload),
};

// Examples
store.dispatch(actionCreators.increaseCounter(4)); // Error: Supplied parameters do not match any signature of call target. 
store.dispatch(actionCreators.increaseCounter()); // OK: { type: "INCREASE_COUNTER" }
actionCreators.increaseCounter.type === "INCREASE_COUNTER" // true

store.dispatch(actionCreators.changeBaseCurrency()); // Error: Supplied parameters do not match any signature of call target. 
store.dispatch(actionCreators.changeBaseCurrency('USD')); // OK: { type: "CHANGE_BASE_CURRENCY", payload: 'USD' }
actionCreators.changeBaseCurrency.type === "CHANGE_BASE_CURRENCY" // true

store.dispatch(actionCreators.showNotification()); // Error: Supplied parameters do not match any signature of call target.
store.dispatch(actionCreators.showNotification('Hello!')); // OK: { type: "SHOW_NOTIFICATION", payload: 'Hello!' }
store.dispatch(actionCreators.showNotification('Hello!', { type: 'warning' })); // OK: { type: "SHOW_NOTIFICATION", payload: 'Hello!', meta: { type: 'warning' } }
actionCreators.showNotification.type === "SHOW_NOTIFICATION" // true
```


---

# Reducers

---

引用到的 Typescript 文档:

  - 
  -

## 声明 reducer `State` 类型来达到运行时的不可变性

```tsx
// 1a. use readonly modifier to mark state props as immutable and guard with compiler against any mutations
export type State = {
  readonly counter: number,
  readonly baseCurrency: string,
};

// 1b. if you prefer you can use `Readonly` mapped type as alternative convention
export type StateAlternative = Readonly<{
  counter: number,
  baseCurrency: string,
}>;

// 2. declare initialState using State -> Note: only initialization is allowed with readonly modifiers
export const initialState: State = {
  counter: 0,
  baseCurrency: 'EUR',
};

initialState.counter = 3; // Error: Cannot assign to 'counter' because it is a constant or a read-only property

```

## switch 风格的 reducer

  - 使用静态常量类型
  - 更有利于单 prop 更新或者简单的 state 对象

```tsx
import { Action } from '../../types';

export default function reducer(state: State = initialState, action: Action): State {
  switch (action.type) {
    case INCREASE_COUNTER:
      return {
        ...state, counter: state.counter + 1, // no payload
      };
    case CHANGE_BASE_CURRENCY:
      return {
        ...state, baseCurrency: action.payload, // payload: string
      };

    default: return state;
  }
}
```


## if 风格的 reducer
> 使用从帮助工厂函数中的可选的静态 `type` 属性 在 `actionCreator` 上

  - if 风格下的 "block scope" 会给你使用本地变量来应对复杂的state更新逻辑
  - 通过使用可选的静态`type`属性在 `actionCreator`上, 我们可以脱离 类型的定义常量, 使其更易于创建 actions, 同时易于复用它们来检查 action的类型在reducers上, sagas,epics等等

```tsx
import { Action } from '../../types';

// Reducer
export default function reducer(state: State = initialState, action: Action): State {
  switch (action.type) {
    case actionCreators.increaseCounter.type:
      return {
        ...state, counter: state.counter + 1, // no payload
      };
    case actionCreators.changeBaseCurrency.type:
      return {
        ...state, baseCurrency: action.payload, // payload: string
      };

    default: return state;
  }
}

```

## 通过正确的类型检查进行扩展操作,防止过量或不匹配的 props

通过使用 `partialState` 对象在扩展操作时,我们可以保全对象的合并 - 这将确保新的state对象不会有任何过量或者不匹配的属性, 这将是完全匹配我们的`State`接口

**注意**: 本解决办法目前的强制性的, 因为在扩展操作时 Typescript的编译器并不会防范多于或不匹配的 props

**PS**: 这是一个关于改进此行为的 (提案)[https://github.com/Microsoft/TypeScript/issues/12936]


```tsx
import { Action } from '../../types';

// BAD
export function badReducer(state: State = initialState, action: Action): State {
  if (action.type === INCREASE_COUNTER) {
    return {
      ...state,
      counterTypoError: state.counter + 1, // OK
    }; // it's a bug! but the compiler will not find it 
  }
}

// GOOD
export function goodReducer(state: State = initialState, action: Action): State {
  let partialState: Partial<State> | undefined;

  if (action.type === INCREASE_COUNTER) {
    partialState = {
      counterTypoError: state.counter + 1, // Error: Object literal may only specify known properties, and 'counterTypoError' does not exist in type 'Partial<State>'. 
    }; // now it's showing a typo error correctly 
  }
  if (action.type === CHANGE_BASE_CURRENCY) {
    partialState = { // Error: Types of property 'baseCurrency' are incompatible. Type 'number' is not assignable to type 'string'.
      baseCurrency: 5,
    }; // type errors also works fine 
  }

  return partialState != null ? { ...state, ...partialState } : state;
}
```


---

# Store 类型

---

 - ## Action - 静态化全局的 action 类型
 - 应是在处理redux的action的层级比如:reducers,redux-sagas,redux-observable中被引入

 ```tsx
import { returntypeof } from 'react-redux-typescript';
import * as actionCreators from './action-creators';
const actions = Object.values(actionCreators).map(returntypeof);

export type Action = typeof actions[number];

 ```
 - RootState - 静态类型的全局状态树
 - 应是在 连结组件时引入, 提供类型安全给 Redux 的`connect`函数

```tsx
import {
  reducer as currencyRatesReducer, State as CurrencyRatesState,
} from './state/currency-rates/reducer';
import {
  reducer as currencyConverterReducer, State as CurrencyConverterState,
} from './state/currency-converter/reducer';

export type RootState = {
  currencyRates: CurrencyRatesState;
  currencyConverter: CurrencyConverterState;
};
```



---

## Create Store

- 创建Store - use `RootState` (in `combineReducers` and when providing preloaded state object) to set-up **state object type guard** to leverage strongly typed Store instance
```ts
import { combineReducers, createStore } from 'redux';
import { RootState } from '../types';

const rootReducer = combineReducers<RootState>({
  currencyRates: currencyRatesReducer,
  currencyConverter: currencyConverterReducer,
});

// rehydrating state on app start: implement here...
const recoverState = (): RootState => ({} as RootState);

export const store = createStore(
  rootReducer,
  recoverState(),
);
```

- composing enhancers - example of setting up `redux-observable` middleware
```ts
declare var window: Window & { devToolsExtension: any, __REDUX_DEVTOOLS_EXTENSION_COMPOSE__: any };
import { createStore, compose, applyMiddleware } from 'redux';
import { combineEpics, createEpicMiddleware } from 'redux-observable';

import { epics as currencyConverterEpics } from './currency-converter/epics';

const rootEpic = combineEpics(
  currencyConverterEpics,
);
const epicMiddleware = createEpicMiddleware(rootEpic);
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;

// store singleton instance
export const store = createStore(
  rootReducer,
  recoverState(),
  composeEnhancers(applyMiddleware(epicMiddleware)),
);
```

---

# Ecosystem

---

## Async Flow with "redux-observable"

```ts
import 'rxjs/add/operator/map';
import { combineEpics, Epic } from 'redux-observable';

import { RootState, Action } from '../types'; // check store section
import { actionCreators } from '../reducer';
import { convertValueWithBaseRateToTargetRate } from './utils';
import * as currencyConverterSelectors from './selectors';
import * as currencyRatesSelectors from '../currency-rates/selectors';

const recalculateTargetValueOnCurrencyChange: Epic<Action, RootState> = (action$, store) =>
  action$.ofType(
    actionCreators.changeBaseCurrency.type,
    actionCreators.changeTargetCurrency.type,
  ).map((action: any) => {
    const value = convertValueWithBaseRateToTargetRate(
      currencyConverterSelectors.getBaseValue(store.getState()),
      currencyRatesSelectors.getBaseCurrencyRate(store.getState()),
      currencyRatesSelectors.getTargetCurrencyRate(store.getState()),
    );
    return actionCreators.recalculateTargetValue(value);
  });

const recalculateTargetValueOnBaseValueChange: Epic<Action, RootState> = (action$, store) =>
  action$.ofType(
    actionCreators.changeBaseValue.type,
  ).map((action: any) => {
    const value = convertValueWithBaseRateToTargetRate(
      action.payload,
      currencyRatesSelectors.getBaseCurrencyRate(store.getState()),
      currencyRatesSelectors.getTargetCurrencyRate(store.getState()),
    );
    return actionCreators.recalculateTargetValue(value);
  });

const recalculateBaseValueOnTargetValueChange: Epic<Action, RootState> = (action$, store) =>
  action$.ofType(
    actionCreators.changeTargetValue.type,
  ).map((action: any) => {
    const value = convertValueWithBaseRateToTargetRate(
      action.payload,
      currencyRatesSelectors.getTargetCurrencyRate(store.getState()),
      currencyRatesSelectors.getBaseCurrencyRate(store.getState()),
    );
    return actionCreators.recalculateBaseValue(value);
  });

export const epics = combineEpics(
  recalculateTargetValueOnCurrencyChange,
  recalculateTargetValueOnBaseValueChange,
  recalculateBaseValueOnTargetValueChange,
);
```

---

## Selectors with "reselect"

```ts
import { createSelector } from 'reselect';
import { RootState } from '../types';

const getCurrencyConverter = (state: RootState) => state.currencyConverter;
const getCurrencyRates = (state: RootState) => state.currencyRates;

export const getCurrencies = createSelector(
  getCurrencyRates,
  (currencyRates) => {
    return Object.keys(currencyRates.rates).concat(currencyRates.base);
  },
);

export const getBaseCurrencyRate = createSelector(
  getCurrencyConverter, getCurrencyRates,
  (currencyConverter, currencyRates) => {
    const selectedBase = currencyConverter.baseCurrency;
    return selectedBase === currencyRates.base
      ? 1 : currencyRates.rates[selectedBase];
  },
);

export const getTargetCurrencyRate = createSelector(
  getCurrencyConverter, getCurrencyRates,
  (currencyConverter, currencyRates) => {
    return currencyRates.rates[currencyConverter.targetCurrency];
  },
);
```

---

# Extras

### tsconfig.json
> Recommended setup for best benefits from type-checking, with support for JSX and ES2016 features.  
> Add [`tslib`](https://www.npmjs.com/package/tslib) to minimize bundle size: `npm i tslib` -  this will externalize helper functions generated by transpiler and otherwise inlined in your modules  
```js
{
  "compilerOptions": {
    "baseUrl": "src/", // enables relative imports to root
    "outDir": "out/", // target for compiled files
    "allowSyntheticDefaultImports": true, // no errors on commonjs default import
    "allowJs": true, // include js files
    "checkJs": true, // typecheck js files
    "declaration": false, // don't emit declarations
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "forceConsistentCasingInFileNames": true,
    "importHelpers": true, // importing helper functions from tslib
    "noEmitHelpers": true, // disable emitting inline helper functions
    "jsx": "react", // process JSX
    "lib": [
      "dom",
      "es2016",
      "es2017.object"
    ],
    "target": "es5",
    "module": "es2015",
    "moduleResolution": "node",
    "noEmitOnError": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "strictNullChecks": true,
    "pretty": true,
    "removeComments": true,
    "sourceMap": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

### tslint.json
> Recommended setup is to extend build-in preset `tslint:latest` (for all rules use `tslint:all`).  
> Add tslint react rules: `npm i -D tslint-react` https://github.com/palantir/tslint-react  
> Amended some extended defaults for more flexibility.  
```json
{
  "extends": ["tslint:latest", "tslint-react"],
  "rules": {
    "arrow-parens": false,
    "arrow-return-shorthand": [false],
    "comment-format": [true, "check-space"],
    "import-blacklist": [true, "rxjs"],
    "interface-over-type-literal": false,
    "member-access": false,
    "member-ordering": [true, {"order": "statics-first"}],
    "newline-before-return": false,
    "no-any": false,
    "no-inferrable-types": [true],
    "no-import-side-effect": [true, {"ignore-module": "^rxjs/"}],
    "no-invalid-this": [true, "check-function-in-method"],
    "no-null-keyword": false,
    "no-require-imports": false,
    "no-switch-case-fall-through": true,
    "no-trailing-whitespace": true,
    "no-unused-variable": [true, "react"],
    "object-literal-sort-keys": false,
    "only-arrow-functions": [true, "allow-declarations"],
    "ordered-imports": [false],
    "prefer-method-signature": false,
    "prefer-template": [true, "allow-single-concat"],
    "quotemark": [true, "single", "jsx-double"],
    "triple-equals": [true, "allow-null-check"],
    "typedef": [true,"parameter", "property-declaration", "member-variable-declaration"],
    "variable-name": [true, "ban-keywords", "check-format", "allow-pascal-case"]
  }
}
```

### Default and Named Module Exports
> Most flexible solution is to use module folder pattern, because you can leverage both named and default import when you see fit.  
Using this solution you'll achieve better encapsulation for internal structure/naming refactoring without breaking your consumer code:  
```ts
// 1. in `components/` folder create component file (`select.tsx`) with default export:

// components/select.tsx
const Select: React.StatelessComponent<Props> = (props) => {
...
export default Select;

// 2. in `components/` folder create `index.ts` file handling named imports:

// components/index.ts
export { default as Select } from './select';
...

// 3. now you can import your components in both ways like this:

// containers/container.tsx
import { Select } from '../components';
or
import Select from '../components/select';
...
```

### Vendor Types Augmentation
> Augmenting missing autoFocus Prop on `Input` and `Button` components in `antd` npm package (https://ant.design/).  
```ts
// using relative import resolution
declare module '../node_modules/antd/lib/input/Input' {
  export interface InputProps {
    autoFocus?: boolean;
  }
}

// using node module resolution
import { Operator } from 'rxjs/Operator';
import { Observable } from 'rxjs/Observable';

declare module 'rxjs/Subject' {
  interface Subject<T> {
    lift<R>(operator: Operator<T, R>): Observable<R>;
  }
}
```

---

# FAQ

### - how to install react & redux types?
```
// react
npm i -D @types/react @types/react-dom @types/react-redux

// redux has types included in it's npm package - don't install from @types
```

### - should I still use React.PropTypes in TS?
> No. In TypeScript it is unnecessary, when declaring Props and State types (refer to React components examples) you will get completely free intellisense and compile-time safety with static type checking, this way you'll be safe from runtime errors, not waste time on debugging and get an elegant way of describing component external API in your source code.  

### - how to best initialize class instance or static properties?
> Prefered modern style is to use class Property Initializers  
```tsx
class MyComponent extends React.Component<Props, State> {
  // default props using Property Initializers
  static defaultProps: Props = {
    className: 'default-class',
    initialCount: 0,
  };
  
  // initial state using Property Initializers
  state: State = {
    counter: this.props.initialCount,
  };
  ...
}
```

### - how to best declare component handler functions?
> Prefered modern style is to use Class Fields with arrow functions  
```tsx
class MyComponent extends React.Component<Props, State> {
// handlers using Class Fields with arrow functions
  increaseCounter = () => { this.setState({ counter: this.state.counter + 1}); };
  ...
}
```

### - differences between `interface` declarations and `type` aliases
> From practical point of view `interface` types will use it's identity when showing compiler errors, while `type` aliases will be always unwinded to show all the nested types it consists of. This can be too noisy when reading compiler errors and I like to leverage this distinction to hide some not important type details in reported type errors  
Related `ts-lint` rule: https://palantir.github.io/tslint/rules/interface-over-type-literal/  

---

# Project Examples

https://github.com/piotrwitek/react-redux-typescript-starter-kit  
https://github.com/piotrwitek/react-redux-typescript-webpack-starter  
