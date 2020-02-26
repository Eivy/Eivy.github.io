---
layout: post
title: JestでcomponentDidMountにsetStateを含んだテストをする
date: 2020-02-26T23:15:18 UTC+0900
tags: react jest
---
ReactコンポーネントでcomponentDidMount内でsetStateが動くようなのをテストしようとしてハマりました。

例えば以下のような感じでテストしても動かないので悩んでいました。

```typescript
import jest from 'jest';
import * as React from 'react';
import { render } from '@testing-library/react';
import request from 'request-promise-native';
interface Props {};
interface State {
  time: string,
};
class Test extends React.Components {
  constructor(props: Prop) {
    super(prop);
    this.state = {
      time: '',
    };
  }
  render() {
    return (<div>{this.state.time}</div>);
  }
  componentDidMount() {
    request.get(api_server).then((err, res, body) => setState({time: body}));
  }
}
test('render test', () => {
  const mock_get = jest.spyOn(request, 'get');
  mock_get.mockResolveValue(new Promise((resolve, reject) => {
    resolve('23:15:18');
  }));
  const { getByText } = render(<Test />);
  expect(getByText('23:15:18')).toBeInTheDocument(); // この時点でstate.timeは''のまま
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
