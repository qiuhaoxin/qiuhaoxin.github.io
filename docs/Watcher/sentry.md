## Sentry 源码解读

### Sentry 简介

Sentry 是一个适用于多端的一个监控系统，我们这里只对前端相关的一些包来做解读。希望通过阅读其源码来为自己的监控系统做借鉴。

- @sentry/browser

- @sentry/vue

- @sentry/react

### 重要方法

先列一下几个重要的方法，以便讲解源码的时候可以重点对照：

```Javascript
    // 获取全局对象
    // @sentry/utils
    export function getGlobalObject<T>(): T & SentryGlobal {
    return (isNodeEnv()
        ? global
        : typeof window !== 'undefined' // eslint-disable-line no-restricted-globals
        ? window // eslint-disable-line no-restricted-globals
        : typeof self !== 'undefined'
        ? self
        : fallbackGlobalObject) as T & SentryGlobal;
    }
```

```Javascript
    /**
 * Returns the default hub instance. 返回默认的数据中心实例
 *
 * If a hub is already registered in the global carrier but this module
 * contains a more recent version, it replaces the registered version.
 * Otherwise, the currently registered hub will be returned.
 */
export function getCurrentHub(): Hub {
  // Get main carrier (global for every environment)
  const registry = getMainCarrier();

  // If there's no hub, or its an old API, assign a new one
  if (!hasHubOnCarrier(registry) || getHubFromCarrier(registry).isOlderThan(API_VERSION)) {
    setHubOnCarrier(registry, new Hub());
  }

  // Prefer domains over global if they are there (applicable only to Node environment)
  if (isNodeEnv()) {
    return getHubFromActiveDomain(registry);
  }
  // Return hub that lives on a global object
  return getHubFromCarrier(registry);
}
```

### @sentry/browser

在使用中，我们一般通过以下代码来配置入口：

```Javascript
    import * as Sentry from "@sentry/browser";
    import { Integrations } from "@sentry/tracing";
    Sentry.init({
        dsn: "https://examplePublicKey@o0.ingest.sentry.io/0",
        // Alternatively, use `process.env.npm_package_version` for a dynamic release version
        // if your build tool supports it.
        release: "my-project-name@2.3.12",
        integrations: [new Integrations.BrowserTracing()],

        // Set tracesSampleRate to 1.0 to capture 100%
        // of transactions for performance monitoring.
        // We recommend adjusting this value in production
        tracesSampleRate: 1.0,
    })
```
