# 前端性能优化

## 网络传输—— Brotli 压缩算法

GZIP 压缩是实现此目的的一种方法，但 [Brotli 压缩](https://www.brotli.org/) 是另一种值得关注的新兴方法，**Brotli是专门为网络设计的**。

### Gzip

**Gzip 基于 DEFLATE 算法，它是 LZ77 和霍夫曼编码的组合**，最早用于 UNIX 系统的文件压缩。HTTP 协议上的 Gzip 编码是一种用来进 Web 应用程序性能的技术，Web [服务器](https://cloud.tencent.com/product/cvm?from=10680)和客户端（浏览器）必须共同支持 Gzip，当下主流的浏览器都是支持 Gzip 压缩，包括 IE6、IE7、IE8、IE9、FireFox、Google Chrome、Opera 等。

### Brotli

Google 认为互联网用户的时间是宝贵，尤其不应该浪费在无用的网页加载中。

2013年，他们发布了 Zotfli 压缩算法。该算法在默认设置下的输出比 zlib 的最大压缩比输出还要小 3-8%。PNG 优化器、Web 内容预处理等许多压缩方案中都集成了该算法。基于该算法的应用情况，于 2015 年 9 月推出了**无损压缩算法 Brotli，**最初用于用于网络字体的离线压缩。该算法由谷歌压缩团队的 Jyrki Alakuijala 和 Zoltan Szabadka 开发，其中 Jyrki 亦是 Zotfli 压缩算法的创建者。

2015年9月发布了包含通用无损数据压缩的 Brotli 增强版本，**特别侧重于HTTP压缩**。其中的编码器被部分改写以提高压缩比，编码器和解码器都提高了速度，流式API已被改进，增加更多压缩质量级别。新版本还展现了跨平台的性能改进，以及减少解码所需的内存。

**Brotli 通过变种的 LZ77 算法、Huffman 编码以及二阶文本建模等方式进行数据压缩，与其他压缩算法相比，它有着更高的压缩效率**。

与常见的通用压缩算法不同，Brotli 使用一个预定义的120千字节字典。该字典包含超过13000个常用单词、短语和其他子字符串，这些来自一个文本和HTML文档的大型语料库。预定义的算法可以提升较小文件的压缩密度。

使用 brotli 替换 deflate 来对文本文件压缩通常可以增加20%的压缩密度，而压缩与解压缩速度则大致不变。

Brotli 压缩算法具有多个特点，最典型的是以下 3 个：

- 针对常见的 Web 资源内容，Brotli 的性能相比 Gzip 提高了 17-25%；
- 当 Brotli 压缩级别为 1 时，压缩率比 Gzip 压缩等级为 9（最高）时还要高；
- 在处理不同 HTML 文档时，Brotli 依然能够提供非常高的压缩率。
- 比其他算法提供更快的解压与压缩算法

### 体积压缩结果

所有文件体积都为 KB

#### Gzip 

| 压缩等级 | HTML File 1   | HTML File 2  | CSS File 1   | CSS File 2   | JS File 1    | JS File 2    |
| -------- | ------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| 不压缩   | 1112          | 97.2         | 197.5        | 151.1        | 89.5         | 179.3        |
| 1        | 330.9 (70.2%) | 24.1 (75.2%) | 40 (79.7%)   | 23.7 (84.3%) | 36 (59.8%)   | 61.3 (65.8%) |
| 2        | 319.5 (71.3%) | 23.4 (75.9%) | 38.2 (80.7%) | 23.5 (84.4%) | 34.8 (61.1%) | 59.1 (67%)   |
| 3        | 309.9 (72.1%) | 22.8 (76.5%) | 36.7 (81.4%) | 23.1 (84.7%) | 33.9 (62.1%) | 57.4 (68%)   |
| 4        | 298.1 (73.2%) | 21.6 (77.8%) | 33.7 (82.9%) | 21.3 (85.9%) | 32 (64.2%)   | 54.2 (69.8%) |
| 5        | 288.5 (74.1%) | 20.7 (78.7%) | 31.8 (83.9%) | 21 (86.1%)   | 31 (65.4%)   | 52.4 (70.8%) |
| 6        | 282.9 (74.6%) | 20.6 (78.8%) | 31.1 (84.3%) | 20.8 (86.2%) | 30.8 (65.6%) | 51.9 (71.1%) |
| 7        | 281.8 (74.7%) | 20.5 (78.9%) | 30.9 (84.4%) | 20.8 (86.2%) | 30.8 (65.6%) | 51.7 (71.2%) |
| 8        | 281.3 (74.7%) | 20.5 (78.9%) | 30.8 (84.4%) | 20.7 (86.3%) | 30.8 (65.6%) | 51.6 (71.2%) |
| 9        | 281.3 (74.7%) | 20.5 (78.9%) | 30.8 (84.4%) | 20.7 (86.3%) | 30.8 (65.6%) | 51.6 (71.2%) |

#### Brotli 

| 压缩等级 | HTML File 1   | HTML File 2  | CSS File 1   | CSS File 2   | JS File 1    | JS File 2    |
| -------- | ------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| 不压缩   | 1112          | 97.2         | 197.5        | 151.1        | 89.5         | 179.3        |
| 0        | 345.6 (68.9%) | 23.3 (76%)   | 40.1 (79.7%) | 25.7 (83%)   | 37.2 (58.4%) | 62 (65.4%)   |
| 1        | 302.4 (72.8%) | 22.2 (77.2%) | 37.3 (81.1%) | 21.8 (85.6%) | 35.7 (60.1%) | 59.2 (67%)   |
| 2        | 279.9 (74.8%) | 20.9 (78.5%) | 34.3 (82.6%) | 20.6 (86.4%) | 33 (63.1%)   | 54.3 (69.7%) |
| 3        | 275.6 (75.2%) | 20.4 (79%)   | 33 (83.3%)   | 20.4 (86.5%) | 32.8 (63.4%) | 53.3 (70.3%) |
| 4        | 262.5 (76.4%) | 19.2 (80.2%) | 31.5 (84.1%) | 19.8 (86.9%) | 31.9 (64.4%) | 52 (71%)     |
| 5        | 251.3 (77.4%) | 18.2 (81.3%) | 28.9 (85.4%) | 19.1 (87.4%) | 30.3 (66.1%) | 49 (72.7%)   |
| 6        | 249.3 (77.6%) | 18.1 (81.4%) | 28.4 (85.6%) | 19.1 (87.4%) | 30.1 (66.4%) | 48.6 (72.9%) |
| 7        | 248.5 (77.7%) | 18.1 (81.4%) | 28 (85.8%)   | 19.1 (87.4%) | 30.1 (66.4%) | 48.3 (73.1%) |
| 8        | 248.1 (77.7%) | 18.1 (81.4%) | 27.8 (85.9%) | 19.1 (87.4%) | 30 (66.5%)   | 48.3 (73.1%) |
| 9        | 247.9 (77.7%) | 18 (81.5%)   | 27.6 (86%)   | 19 (87.4%)   | 30 (66.5%)   | 48.2 (73.1%) |
| 10       | 225.2 (79.7%) | 16.1 (83.4%) | 26.5 (86.6%) | 17.5 (88.4%) | 28.4 (68.3%) | 44.9 (75%)   |
| 11       | 223 (79.9%)   | 15.8 (83.7%) | 25.9 (86.9%) | 17.2 (88.6%) | 28 (68.7%)   | 44.2 (75.3%) |

![](./img/chart_html.png)

![](./img/chart_css.png)

![](./img/chart_js.png)

Brotli 采用与 Gzip 使用相同的技术，并使用现代方法对其进行增强：

- HTML 文件比 Gzip 小 21%
- CSS 文件比 Gzip 小 17%
- JavaScript 文件比 Gzip 小 14%

**Brotli 的压缩比更好，而 Gzip 在压缩速度方面领先。**

### 传输对比

| 压缩算法  | 请求数/s | 传输的字节数 (MB/s) | 最大的 RSS (MB) | 平均延迟 (ms) |
| --------- | -------- | ------------------- | --------------- | ------------- |
| br-stream | 203      | 0.25                | 3485.54         | 462.57        |
| lzma      | 233      | 0.37                | 330.29          | 407.71        |
| gzip      | 2276     | 3.44                | 204.29          | 41.86         |
| none      | 4061     | 14.06               | 125.1           | 23.45         |
| br-static | 4087     | 5.85                | 105.58          | 23.3          |



### Brotli 兼容性

主流浏览器基本上都支持了。

![](./img/brotli_caniuse.png)

## 服务器支持 Brotli 

支持Brotli压缩算法的浏览器使用的内容编码类型为  `br`

http请求头：`Accept-Encoding: gzip, deflate, sdch, br`

http返回头：`Content-Encoding: br`



### 在 Nginx 上启用Brotli

nginx目前并不支持Brotli算法，需要使用第三方模块，例如 [ngx_brotli](https://github.com/google/ngx_brotli) 进行实现。

下面是简单的安装步骤：

```shell
git clone https://github.com/google/ngx_brotli
cd ngx_brotli
git submodule update --init
cd /path/to/nginx_source/
./configure --add-module=/path/to/ngx_brotli
make && make install
```

或者使用现有镜像 [fholzer/nginx-brotli](https://hub.docker.com/r/fholzer/nginx-brotli) 制作项目镜像脚本：

```dockerfile
FROM fholzer/nginx-brotli:v1.21.6

RUN mkdir -p /webui
COPY ./dist  /webui

CMD ["-g", "daemon off;"]
```



### 前端支持 .br 

前端产物支持 br 产物为非必须项，出了更好，能够减少服务端动态压缩的工作。

webpack 配置：

```js
const CompressionPlugin = require('compression-webpack-plugin');
const zlib = require('zlib'); // node 自带库。只要版本不是太低（> 11 ?）就有

// plugins: [{...

new CompressionPlugin({
    filename: '[path][base].br',
    algorithm: 'brotliCompress',
    test: /\.(js|css|html|svg)$/,
    compressionOptions: {
        params: {
            [zlib.constants.BROTLI_PARAM_QUALITY]: 11,
        },
    },
    threshold: 10240,
    minRatio: 0.8,
})

// }],
```



### Nginx 配置文件的 http 块下增加以下指令

```shell
brotli               on;  
brotli_comp_level    6;  
brotli_buffers       16 8k;  
brotli_min_length    20;  
brotli_types         *;
```
