## Vue-Router 官方路由组件

### 基本知识点

- 1、路由守卫/钩子

```Javascript
    1、全局前置守卫
        beforeEach((to,from,next){

            return  //值可以为 false,
        })

    2、全局解析守卫
        beforeResolve()

    3、全局后置钩子
        afterEach()   // 你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 next 函数也不会改变导航本身
        //它们对于分析、更改页面标题、声明页面等辅助功能以及许多其他事情都很有用。
    4、路由独享的守卫
        /*
        * beforeEnter 守卫 只在进入路由时触发，不会在 params、query 或 hash 改变时触发。例如，从 /users/2 进入到 /users/3 或者从 /users/2#info 进入到 /users/2#projects。它们只有在 从一个不同的 路由导航时，才会被触发。
        */
        const routes = [
            {
                path: '/users/:id',
                component: UserDetails,
                beforeEnter: (to, from) => {
                    // reject the navigation
                    return false
                },
            },
        ]

    5、路由组件内的守卫(只能用在路由组件内哦)
        可以为路由组件内添加beforeRouteEnter、beforeRouteLeave、beforeRouteUpdate
        beforeRouteEnter(to,from,next){
            //注意，这是路由组件还么有创建，不能访问组件实例this,但可以提供一个回调函数给next，并把组件实例给next当参数
            next(vm=>{

            })
        }
        beforeRouteUpdate(to,from){
            //这里可以访问this了
            //一般用以路由切换但路由组件重用，获取相应参数并重新加载数据 比如/user/8   切换到/user/9
        }
        beforeRouteLeave(to,from){
            //可以访问this
            //一般用于页面数据变动了但没有保存而离开页面
        }

```

完整的导航解析流程：

- 、导航被触发
- 、在失活的组件里调用 beforeRouteLeave 守卫
- 、调用全局的 beforeEach 守卫
- 、在重用的组件里调用 beforeRouteUpdate 守卫(2.2+)。
- 、在路由配置里调用 beforeEnter
- 、解析异步路由组件
- 、在被激活的组件里调用 beforeRouteEnter
- 、调用全局的 beforeResolve 守卫(2.5+)。
- 、导航被确认
- 、调用全局的 afterEach 钩子。
- 、触发 DOM 更新。
- 、调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

- 2、路由元信息

- 3、数据获取
  进入某个路由，要从服务器获取数据，一般有两种方式，取决于用户体验

```Javascript
    1、导航完成之后获取
    先完成导航，然后再由组件的生命周期钩子获取数据
    2、导航完成之前获取
    导航完成前，由路由进入的守卫获取，在数据获取成功后执行导航
```

- 4、动态路由

````Javascript
    1、可以通过: 来设置动态路由
    //    /user/:userName   //匹配 /user/qiuhaoxin
    2、在参数中定义正则表达式
    //可重复参数
    const routes = [
        // 仅匹配数字
        // 匹配 /1, /1/2, 等
        { path: '/:chapters(\\d+)+' },
        // 匹配 /, /1, /1/2, 等
        { path: '/:chapters(\\d+)*' },
    ]
    //可选参数
    const routes = [
        // 匹配 /users 和 /users/posva
        { path: '/users/:userId?' },
        // 匹配 /users 和 /users/42
        { path: '/users/:userId(\\d+)?' },
    ]
```
- 5、嵌套路由

- 6、导航

```
    //声明式导航
    <router-link :to="" />   //内部其实调用的就是router.push
    //编程式导航
    this.$router.push({path:''}) //带path，params参数将会被忽略
    this.$router.push({name:'',params:{}})
    //
    this.$router.replace()   相当于this.$router.push({path:'',replace:true})

    this.$router.go(-1) // 相当于this.$router.back()
````

- 7、命名路由

- 8、命名视图
  这个如果想做同级视图的时候很有用

  ```
       <router-view class="view left-sidebar" name="LeftSidebar"></router-view>
       <router-view class="view main-content"></router-view>
       <router-view class="view right-sidebar" name="RightSidebar"></router-view>
  ```

  ```
    const router = createRouter({
    history: createWebHashHistory(),
    routes: [
        {
        path: '/',
        components: {
            default: Home,
            // LeftSidebar: LeftSidebar 的缩写
            LeftSidebar,
            // 它们与 `<router-view>` 上的 `name` 属性匹配
            RightSidebar,
        },
        },
    ],
    })
  ```

### 注意点

- 、如果是动态路由，如果只是参数不同，会复用路由组件

比如：/user/:userName 的路由，实际应用中如果 /user/qiuhaoxin 切换到 /user/haoxin_qiu 由于组件是复用的，组件的生命周期钩子不会被调用，
这个时候通过什么来获取参数的变化并做出相应的操作呢？

```Javascript
    1、可以在组件内箭头this.$route.params
        // vue 3.x
        this.$watch(
            ()=>this.$route.params,
            (toParams,fromParams)=>{

            }
        )
    2、如上面所说可以通过路由组件内的守卫
        beforeRouteUpdate(to,from){
            const {params:{}}=to;

        }
```

- 、hash 路由和 history 路由的区别
