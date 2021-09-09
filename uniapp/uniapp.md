
### uniapp设置全局变量
#### 共用模块
定义一个专用的模块，用来组织和管理这些全局的变量，在需要的页面引入。

注意这种方式只支持多个vue页面或多个nvue页面之间公用，vue和nvue之间不公用。

示例如下：
在 uni-app 项目根目录下创建 common 目录，然后在 common 目录下新建 helper.js 用于定义公用的方法
```text
const websiteUrl = 'http://uniapp.dcloud.io';  
const now = Date.now || function () {  
    return new Date().getTime();  
};  
const isArray = Array.isArray || function (obj) {  
    return obj instanceof Array;  
};  

export default {  
    websiteUrl,  
    now,  
    isArray  
}
```
接下来在 pages/index/index.vue 中引用该模块
```text
<script>  
    import helper from '../../common/helper.js';  

    export default {  
        data() {  
            return {};  
        },  
        onLoad(){  
            console.log('now:' + helper.now());  
        },  
        methods: {  
        }  
    }  
</script>
```
这种方式维护起来比较方便，但是缺点就是每次都需要引入。

#### 挂载Vue.prototype

将一些使用频率较高的常量或者方法，直接扩展到 Vue.prototype 上，每个 Vue 对象都会“继承”下来。

注意这种方式只支持vue页面

示例如下：
在 main.js 中挂载属性/方法
```text
Vue.prototype.websiteUrl = 'http://uniapp.dcloud.io';  
Vue.prototype.now = Date.now || function () {  
    return new Date().getTime();  
};  
Vue.prototype.isArray = Array.isArray || function (obj) {  
    return obj instanceof Array;  
};
```
然后在 pages/index/index.vue 中调用
```text
<script>  
    export default {  
        data() {  
            return {};  
        },  
        onLoad(){  
            console.log('now:' + this.now());  
        },  
        methods: {  
        }  
    }  
</script>
```
这种方式，只需要在 main.js 中定义好即可在每个页面中直接调用。

Tips

* 每个页面中不要在出现重复的属性或方法名。
* 建议在 Vue.prototype 上挂载的属性或方法，可以加一个统一的前缀。比如 $url、global_url 这样，在阅读代码时也容易与当前页面的内容区分开


#### globalData

小程序中有个globalData概念，可以在 App 上声明全局变量。 Vue 之前是没有这类概念的，但 uni-app 引入了globalData概念，并且在包括H5、App等平台都实现了。
在 App.vue 可以定义 globalData ，也可以使用 API 读写这个值。

globalData支持vue和nvue共享数据。

globalData是一种比较简单的全局变量使用方式。

定义：App.vue
```text
<script>  
    export default {  
        globalData: {  
            text: 'text'  
        },  
        onLaunch: function() {  
            console.log('App Launch')  
        },  
        onShow: function() {  
            console.log('App Show')  
        },  
        onHide: function() {  
            console.log('App Hide')  
        }  
    }  
</script>  

<style>  
    /*每个页面公共css */  
</style>  

```

js中操作globalData的方式如下：

赋值：getApp().globalData.text = 'test'

取值：console.log(getApp().globalData.text) // 'test'

如果需要把globalData的数据绑定到页面上，可在页面的onshow声明周期里进行变量重赋值。HBuilderX 2.0.3起，nvue页面在uni-app编译模式下，也支持onshow。



#### Vuex

Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

HBuilderX 2.2.5+起，支持vue和nvue之间共享。参考

这里以登录后同步更新用户信息为例，简单说明下 Vuex 的用法，更多更详细的 Vuex 的内容，建议前往其官网 Vuex 学习下。

举例说明：

在 uni-app 项目根目录下新建 store 目录，在 store 目录下创建 index.js 定义状态值

```text
const store = new Vuex.Store({  
    state: {  
        login: false,  
        token: '',  
        avatarUrl: '',  
        userName: ''  
    },  
    mutations: {  
        login(state, provider) {  
            console.log(state)  
            console.log(provider)  
            state.login = true;  
            state.token = provider.token;  
            state.userName = provider.userName;  
            state.avatarUrl = provider.avatarUrl;  
        },  
        logout(state) {  
            state.login = false;  
            state.token = '';  
            state.userName = '';  
            state.avatarUrl = '';  
        }  
    }  
})
```


然后，需要在 main.js 挂载 Vuex

```text
import store from './store'  
Vue.prototype.$store = store
```

最后，在 pages/index/index.vue 使用

```text
<script>  
    import {  
        mapState,  
        mapMutations  
    } from 'vuex';  

    export default {  
        computed: {  
            ...mapState(['avatarUrl', 'login', 'userName'])  
        },  
        methods: {  
            ...mapMutations(['logout'])  
        }  
    }  
</script>
```
