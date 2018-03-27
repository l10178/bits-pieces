 # For Vue.js
 命名法说明
 1. camel命名法，形如thisIsAnApple
 2. pascal命名法，形如ThisIsAnApple
 3. 下划线命名法，形如this_is_an_apple
 4. 中划线命名法，形如this-is-an-apple

 用vuejs的webpack模板生成的项目中，router/index.js里面有一句：
 ```js
 import Hello from '@/components/Hello'
 ```
 这里路径前面的“@”符号表示什么意思？
 这是webpack的路径别名，相关定义在这里：
 ```js
resolve: {
    // 自动补全的扩展名
    extensions: ['.js', '.vue', '.json'],
    // 默认路径代理
    // 例如 import Vue from 'vue'，会自动到 'vue/dist/vue.common.js'中寻找
    alias: {
        '@': resolve('src'),
        '@config': resolve('config'),
        'vue$': 'vue/dist/vue.common.js'
    }
}
```
