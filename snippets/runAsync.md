---
title: runAsync
tags: browser,function,promise,advanced
firstSeen: 2018-01-02T02:17:52+02:00
lastUpdated: 2022-02-10T16:37:58+08:00
---

Runs a function in a separate thread by using a [Web Worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers), allowing long running functions to not block the UI.

- Create a `Worker` using a `Blob` object URL, the contents of which should be the stringified version of the supplied function.
- Immediately post the return value of calling the function back.
- Return a `Promise`, listening for `onmessage` and `onerror` events and resolving the data posted back from the worker, or throwing an error.

```js
const runAsync = fn => {
  const objectUrl = URL.createObjectURL(new Blob([`postMessage((${fn})());`]), {
      type: 'application/javascript; charset=utf-8'
    });
  const worker = new Worker(objectUrl);
  return new Promise((res, rej) => {
    worker.onmessage = ({ data }) => {
      res(data), worker.terminate();
      URL.revokeObjectURL(objectUrl);
    };
    worker.onerror = err => {
      rej(err), worker.terminate();
      URL.revokeObjectURL(objectUrl);
    };
  });
};
```

```js
const longRunningFunction = () => {
  let result = 0;
  for (let i = 0; i < 1000; i++)
    for (let j = 0; j < 700; j++)
      for (let k = 0; k < 300; k++) result = result + i + j + k;

  return result;
};
/*
  NOTE: Since the function is running in a different context, closures are not supported.
  The function supplied to `runAsync` gets stringified, so everything becomes literal.
  All variables and functions must be defined inside.
*/
runAsync(longRunningFunction).then(console.log); // 209685000000
runAsync(() => 10 ** 3).then(console.log); // 1000
let outsideVariable = 50;
runAsync(() => typeof outsideVariable).then(console.log); // 'undefined'
```
