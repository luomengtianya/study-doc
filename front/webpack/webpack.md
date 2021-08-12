[webpack中文文档](https://webpack.docschina.org/guides/asset-modules/)

#### 使用url-loader或者file-loader异常
```text
因为webpack5内置了资源处理模块，所以想用以前的url-loader或者file-loader模块，需要排除内置模块，才能正常使用
{
    test: /\.(png|jpe?g|gif)$/i,
+    dependency: { not: ['url'] },
    loader: 'url-loader'
}

或者直接使用内置的模块数据
 module: {
   rules: [
     {
       test: /\.png/,
       type: 'asset/resource'
     }
   ]
 }
```


#### 当执行webpack-dev-server命令的时候就报 `Cannot find module 'webpack-cli/bin/config-yargs'`
```text
webpack和webpack-dev-server版本冲突导致的, webpack5以上用
 "start": "webpack serve --open"
而不是
"start": "webpack-dev-serve --open"
```







