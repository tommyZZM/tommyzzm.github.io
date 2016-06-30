本文简要描述babel编译es的流程

```javascript
let a = 123;
```

### 解析(语法) (sytanx)

语法解析模块开始以字符流的形式阅读源码
- 标记有意义的字符串序列(`token`) , 例如目标源码中的`token`包括`let`、`a`、`=`、`123`、`;` (词法分析)
- 利用语法分析模块组合`token`间的关系, 生成相应的节点(`Node`,`Identifier`...) (语法分析)
- 组合节点, 生成抽象语法树`AST`

结构大概如下,忽略Position、Scope等等其他信息;

```
- Program
  - body [{
  	- VariableDeclaration (kind:let)
    	- declarations [{
        	- VariableDeclarator 
            	- id (name:"a")
                - init (value:"123")
        }]
  }]
```

### 转换 (transform)

这里转换成es5实际上就是把`let`转换成`var`

```
Program.body[0].kind = "var"
```
```
- Program
  - body [{
  	- VariableDeclaration (kind:var)
    	- declarations [{
        	- VariableDeclarator 
            	- id (name:"a")
                - init (value:"123")
        }]
  }]
```

### 生成

生成器再将`AST`转换成代码

```
var a = 123;
```

DONE!

参考文献与相关网址
- https://segmentfault.com/a/1190000004607288
- https://github.com/thejameskyle/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-bindings
- https://github.com/babel/babylon/blob/master/ast/spec.md#identifier
- http://astexplorer.net/
