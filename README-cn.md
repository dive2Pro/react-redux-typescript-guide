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