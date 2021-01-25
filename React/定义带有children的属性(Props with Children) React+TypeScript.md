# 定义带有children的属性(Props with Children) React+TypeScript

参考：[How to Define Props with Children in React + TypeScript App](https://www.newline.co/@bespoyasov/how-to-define-props-with-children-in-react-typescript-app--56bd18be)

<br>

### 使用ReactNode

```typescript
import {ReactNode} from 'react';

type ComponentProps = {
  foo: string;
  children?: ReactNode; 
}
```

<br>

### 使用PropsWithChildren

```typescript
// 两种方式任君选择
type PropsWithChildren<P> = P & { children?: ReactNode };
type ComponentProps = PropsWithChildren<{foo: string}>;
```

<br>

### 使用FunctionComponent

```typescript
type ComponentProps = {
  foo: string
}

const Component: React.FunctionComponent<ComponentProps> = ({foo}) => 
  <span>{foo}</span> 
// foo就是children?

// FC: shorthand for FunctionComponent
type ComponentProps = {
  foo: string
}

const Component: React.FC<ComponentProps> = ({foo}) =>
  <span>{foo}</span>
```



### 使用Class Components

```typescript
type AvatarProps = {
  src: string;
}

class Avatar extends React.Component<AvatarProps> {
  render() {
    const {src, children} = this.props;
    return <>
      <img src={src} alt="User photo" />
        {children}
    </>
  }
}
```

