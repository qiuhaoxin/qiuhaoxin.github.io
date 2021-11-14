### 前端中常用的判断

- 1、判断是否 node 环境

```Javascript
 export function isNodeEnv(): boolean {
  return Object.prototype.toString.call(typeof process !== 'undefined' ? process : 0) === '[object process]';
}
```

- 2、判断是否浏览器环境

```Javascript
  export function isBrowser():boolean{
    return typeof window !== 'undefined'
  }
```

- 3、
