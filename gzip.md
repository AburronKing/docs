## 基于vue-cli3的gzip压缩配置

在目前打开网页时，会首先加载各种资源，js、css、图片等等，这时如果这些文件较大的话，打开网页就会比较慢，用户体验不好，有一种需要后端配合的最高级的方法是服务端渲染（SSR），今天我们只分享一下前端单方面优化加载速度的方法：gzip。

gzip压缩可以提高2-3倍的速度，非常棒，首先你要准备配置一下nginx。

    // nginx开启gzip服务
    gzip on;
    gzip_disable "msie6";

    gzip_vary on;
    gzip_static on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    // 需要开启gzip的格式
    gzip_types text/plain text/css application/json application/x-javascript text/xml 
    application/xml application/xml+rss text/javascript image/jpeg image/gif image/png image/jpg;
配置完成后，需要nginx -s reload 重启一下nginx即可。

接下来我们就要在前端vue项目中进行gzip相关的配置了，首先我们需要安装一个插件：
    $ npm i -D compression-webpack-plugin
安装完成后，在我们项目的 vue.config.js 中，引入该插件并配置一下：
    const CompressionPlugin = require("compression-webpack-plugin");
    module.export = {
      configureWebpack: () => {
        if (process.env.NODE_ENV === 'production') {
          return {
            plugins: [
              new CompressionPlugin({
                test: /\.js$|\.html$|\.css$|\.jpg$|\.jpeg$|\.png/, // 需要压缩的文件类型
                threshold: 10240, // 归档需要进行压缩的文件大小最小值，我这个是10K以上的进行压缩
                deleteOriginalAssets: false // 是否删除原文件
              })
            ]
          }
        }
      }
    }
配置完成后，对项目进行 npm run build 打包之后，你可以在dist文件夹下看到相应的.gz的文件，这就是进行压缩后生成的，这时我们在开发者工具中的network中查看我们的js或者其他文件的请求，可在response header中发现content-encoding:gzip，这表示支持gzip的请求。

> 看一下，网页请求的速度是不是飞快呢~