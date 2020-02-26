---
layout: post
title: JestでcomponentDidMountにsetStateを含んだテストをする
date: 2020-02-26T23:15:18 UTC+0900
tags: react jest
---
ReactコンポーネントでcomponentDidMount内でsetStateが動くようなのをテストしようとしてハマりました。

例えば以下のような感じでテストしても動かないので悩んでいました。

`Test.tsx`
```typescript
import * as React from 'react';
import * as request from 'request-promise-native';
interface Props { };
interface State {
  time: string,
};
class Test extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      time: '',
    };
  }
  render() {
    return (
      <div>
        { this.state.time }
      </div>
    );
  }
  async componentDidMount() {
    const time = await request.get('localhost', {}, (err, res, body) => {
      return body;
      }
    );
    this.setState({time: time});
  }
}
export default Test;
```

`Test.test.tsx`
```typescript
import * as React from 'react';
import * as request from 'request-promise-native';
import { render } from '@testing-library/react';
import Test from './Test';
test('render test', () => {
  const mock_get = jest.spyOn(request, 'get');
  mock_get.mockResolvedValue(new Promise((resolve, reject) => {
    resolve('23:15:18');
  }));
  const c  = render(<Test />);
  setImmediate(() => {
    console.log(c.debug());
    expect(c.getByText('23:15:18')).toBeInTheDocument(); // この時点でstate.timeは''のまま
  }
  );
});
```

Enzymeがあればイケるのか？とか色々試したんですが、全然うまく動かなくて、ググってみれば[stack overflow](https://stackoverflow.com/questions/51351491/testing-async-componentdidmount-that-calls-setstate-with-jest-enzyme)に答えが。

最後のチェックの部分を以下のように `setImmediate` でラップすればいいみたい。

```typescript
  setImmediate(() => {
    expect(getByText('23:15:18')).toBeInTheDocument(); // この時点でstate.timeは''のまま
  });
```

結局のところ Promiseが非同期で動くんだけど、いつ動くかわからないのに手続き的に書いても動いてなくて、  
`setImmediate` で非同期処理が終わったあとのサイクルで `expect(...)` を動かすようにするということ...でいいんでしょうか。
