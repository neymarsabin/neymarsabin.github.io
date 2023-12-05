+++
layout = 'post'
title = 'Code Splitting and Dynamic Imports in React'
date = 2021-08-14T12:01:00+05:45
+++

We have realized at some point in developing react apps, as the app size grows the slower it gets because the bundled files are large and takes a lot of time to load/download by the browser. React recommends a special way to handle this with code splitting. This is essential to enhance the performance of your app if you are including large third party libraries. Code splitting is supported by bundlers like `webpack`, `rollup` etc that can create multiple bundle files that can be dynamically loaded at run time. This basically allows us to load just the things that are needed at the moment and this of course improves the performance of your app. We can do split in multiple ways like route based or component based. We can achieve this in React using `Dynamic Imports`. React has these functions `React.lazy` and `Suspense` which helps in dynamic import. 

Let's say we have a large `hello world` component. 
```js
import React from 'react';

const HelloWorld = () => { 
  return (
    <div className="helloworld-container">
      <p> Hello World </p>
    </div>
  )
}

export default HelloWorld
```

Normal import for this would look like this: 
```js
import HelloWorld from './HelloWorld';
```

But to dynamically import, we would do something like this: 
```js
import { lazy } from 'react';
const HelloWorld = lazy(() => import('./HelloWorld'));
```
This will automatically load the bundle containing the `HelloWorld` component when the above code runs. `lazy` takes a function that must call a dynamic import which returns a promise that resolves to a module with a default export containing a React component.

The `HelloWorld` component must be called inside a `Suspense` component(another React function) that allows us to show some fallback content while the `HelloWorld` component is being loaded. 
```js
import React, { lazy, Suspense } from 'react';
const HelloWorld = lazy(() => import('./HelloWorld'));

function App() { 
  return(
    <div className="app-container">
      <Suspense fallback={<div> loading.... </div>}>
        <HelloWorld />
      </Suspense>
    </div>
  )
}
```

Dynamic imports is an awesome way to optimize your react load time performance. Just consider this if you don't want your users to wait a lot of time at first downloading the large bundles.
