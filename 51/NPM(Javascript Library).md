## NPM 实用包 / 轮子

### 日常数据处理

##### CLI
- [chalk](https://www.npmjs.com/package/chalk) (命令行颜色输出)
- [green](https://www.npmjs.com/package/green) (命令行参数解析)
- [shelljs](https://www.npmjs.com/package/shelljs)(调用系统命令)
- [yargs](https://www.npmjs.com/package/yargs)(命令行参数解析)

##### pipe(stream)工具包
- [streamifier](https://www.npmjs.com/package/streamifier) 字符串/buffer=>stream
- [through2](https://www.npmjs.com/package/through2) 
- [merge-stream](https://www.npmjs.com/package/merge-stream) 合并多个stream
- [stream-combiner](https://www.npmjs.com/package/stream-combiner) 首尾相连合并多个stream

##### 通用
- [is.js](https://github.com/pwnn/is.js) (类型校验)

##### 杂项
- [color](https://www.npmjs.com/package/color) (颜色)
- [dateformat](https://www.npmjs.com/package/dateformat)
- [extend](https://www.npmjs.com/package/extend)(普通合并)
- [deep-extend](https://www.npmjs.com/package/deep-extend)(文艺合并)
- [cheerio](https://www.npmjs.com/package/cheerio)(HTML解析与处理)
- [character-parser](https://www.npmjs.com/package/character-parser)(字符串解析与分析)
- [query-string](https://www.npmjs.com/package/query-string)(URL请求参数分析)
- [md5-hex](https://www.npmjs.com/package/md5-hex) (生成一段MD5,Hex)
- [time-stamp](https://www.npmjs.com/package/time-stamp) (时间格式)

##### 文本处理/代码处理
- [change-case](https://www.npmjs.com/package/change-case) (命名转换)
- [babel-core](https://www.npmjs.com/package/babel-core) (最强大的javascript语法解析处理引擎,基于Babylon(Acorn的fork))
- [detective](https://www.npmjs.com/package/detective) (基于Acorn,分析javascript代码文本,获得某函数及其参数的位置信息)
- [strman](https://github.com/dleitee/strman) 字符串处理

##### 类型判断
- [is-stream](https://www.npmjs.com/package/is-stream) 是否stream对象
- [is-promise](https://www.npmjs.com/package/is-promise) 是否Promise对象

##### 图片/素材处理
- [node-pngquant-native](https://github.com/xiangshouding/node-pngquant-native) (pngquant native binding for node.js)

### 路径
- [slash](https://www.npmjs.com/package/slash)(标准化路径)
- [multimatch](https://www.npmjs.com/package/multimatch)(匹配路径)
- [is-absolute-url](https://www.npmjs.com/package/is-absolute-url)(判断url是否绝对)

````javascript
  var regexIsUrlAbsolute = /(^(?:\w+:)\/\/)/ ;
  var regexIsUrlBase64 = /data:(\w+)\/(\w+);/
  function isUrlAbsolute (url) {
      if (typeof url !== 'string') {
          throw new TypeError('Expected a string');
      }
  
      if(regexIsUrlAbsolute.test(url)){
          return true;
      }

      return !!regexIsUrlBase64.test(url); 
  }
````

### 文件系统 file system
- [path-exists](https://www.npmjs.com/package/path-exists) (路径是否存在)
- [fs-extra](https://www.npmjs.com/package/fs-extra) (fs扩展)

### 异步流程
- [async](https://www.npmjs.com/package/async) (异步流程控制)
- [q](https://www.npmjs.com/package/q)(Promise)
- [bluebird](https://www.npmjs.com/package/bluebird)(Promise)

### 编辑

- [commonmark](https://www.npmjs.com/package/commonmark) (MarkDown解析与渲染)
- [marked](https://www.npmjs.com/package/marked) (MarkDown解析与渲染+1)

### 数学公式渲染

- [KaTeX and MathJax Comparison Demo](http://www.intmath.com/cg5/katex-mathjax-comparison.php)
