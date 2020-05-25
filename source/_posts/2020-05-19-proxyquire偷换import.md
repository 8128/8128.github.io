---
title: proxyquire偷换import
comments: true
toc: true
toc_number: false
copyright: true
mathjax: false
katex: false
hide: false
date: 2020-05-19 14:28:33
tags:
	- code
	- javascript
	- test
categories: javascript
description: 让你在test之中可以改变import内容
cover:
top_img:
keywords:
---

最近在写一个延迟handle burst event的东西，我要延迟30s访问，让docusign能把它的burst limit腾出来，防止它爆掉。但是我写了30s的延迟的话，我的test case就需要跑很长时间，因为test case也会跟着延迟30s。而且我写了这个test case到时候merge了相当于大家的test时间全都要+30s，这就很过分。这个时候就可以使用proxyquire来置换掉import信息，比如我原来有：

```js
export const BURST_LIMIT_EXCEED_ERR_DELAY = 30000;
```

这行在我的`docusignDefenition.ts`文件中，我在`docusignBurstHandler.ts`中引用了它来作为我的delay毫秒数，eg.引用如下:

```js
import { BURST_LIMIT_EXCEED_ERR_DELAY } from '../definitions/DocusignDefinitions';

export async function handleDocusignBurst(ctx: Ictx): Promise<void> {
  console.log(BURST_LIMIT_EXCEED_ERR_DELAY);
}

```

而到了`docusignBurstHandlerTests.ts`里，我们就可以用proxyquire来把这个import给替换掉：

```js
import * as sinon from 'sinon';
import * as proxy from 'proxyquire';
import { handleDocusignBurst } from '../../../lib/eSign/external/docusignBurstHandler';

const sandbox = sinon.sandbox.create();
const proxyquire = proxy.noCallThru();

describe('xxxx', function() {
  afterEach(function() {
    sandbox.restore();
  });
  
  it('xxxxx', async function() {
    const BURST_LIMIT_EXCEED_ERR_DELAY = 10;
      const burstTimeMock = sandbox.mock(BURST_LIMIT_EXCEED_ERR_DELAY);
      const handleDocusignBurstStab = proxyquire('../../../lib/eSign/external/docusignBurstHandler', {
        '../definitions/DocusignDefinitions': burstTimeMock,
      });
    	handleDocusignBurstStab.handleDocusignBurst();
  });
});
```

参考：https://ozmoroz.com/2017/09/mocking-es6-dependencies-1