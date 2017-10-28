# Node.js `process.stdin.once('data', console.log)` 是怎么样使得进程不退出的?

我们都知道，把process.stdin接到一个可写流后可以使得node.js进程保持不退出。
```javascript
process.stdin.pipe(someWritableStream());
```

直接运行`node`或者`node --interactive`时, `REPL`也是通过`stdin`获取用户输入的信息。

从Node.js主循环我们可以得知，当任务队列仍然有"任务"时，循环会继续运行。

#### 背景知识

不管是`process.stdin.pipe`还是其他方法，调用后之所以进程得以保持运行。都是因为这些方法调用后，为主循环推入了一个"未结束的libuv的任务"。

需要注意的是Node.js中一些`Web API`如`setTimeout`是也是属于这类方法。在Node.js中调用`setTimeout`会为timmer任务队列推入一个回调计数，并在合适的时候执行回调。几乎所有常见的`Web API`都不是由JavaScript引擎提供的，而是由宿主环境外部定义的(例如[Microsoft/ChakraCore-Sample/OpenGLEngine/ChakraCoreHost.cpp](https://github.com/Microsoft/Chakra-Samples/blob/f35815c03d601a7bd7e2e306549124540d6d9038/ChakraCore%20Samples/OpenGL%20Engine/OpenGLEngine/ChakraCoreHost.cpp#L248))。

读者也可以参阅Node.js的主循环代码 [nodejs/node/src/node.cc#L4835](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/src/node.cc#L4835)

- [JavaScript 运行机制详解：再谈Event Loop(node.js环境的)](http://www.ruanyifeng.com/blog/2014/10/event-loop.html) 
- [[译] libuv 设计概述](https://segmentfault.com/a/1190000005873917)

#### 可是

所以当我把接到一个可写流后，可以使得进程保持运行，想必也是类似的原因。

```
process.stdin.pipe(someWritableStream());
```

侦听`data`事件也可以使得进程保持运行

```
process.stdin.on('data', console.log)
```

也尝试了`process.stdin`调用这些方法时，发现它们也能使得进程不会退出

```javascript
process.stdin.once('data', console.log)
```

```javascript
process.stdin.resume()
```

`process.stdin.on('data', console.log)` 还比较好解释，毕竟添加了一个事件侦听，每个输入都会被输出返回。

可是，后面两个是为什么呢? 说好的`once`呢? 

`process.stdin.resume()`从语义上来看也很奇怪，按理来说恢复了流，应该会默认结束才对吧？`process.stdin.pause()`却不会导致进程保持运行。

读者可以试试`node -e "process.stdin.resume()"`

#### 细节

所以里面的细节是怎么样的呢？

```javascript
process.stdin.resume();

setTimeout(function(){
  console.log("timeout")
}, 3000)
```

通过验证，我们得知循环还在执行，没有被阻塞。

为什么这个循环任务没有取消掉呢?

通过输出`process.stdin.on.toString()`和`process.stdin.resum.toString()`

```javascript
//process.stdin.on.toString()
function (ev, fn) {
  const res = Stream.prototype.on.call(this, ev, fn);

  if (ev === 'data') {
    // Start flowing on next tick if stream isn't explicitly paused
    if (this._readableState.flowing !== false)
      this.resume();
  } else if (ev === 'readable') {
    const state = this._readableState;
    if (!state.endEmitted && !state.readableListening) {
      state.readableListening = state.needReadable = true;
      state.emittedReadable = false;
      if (!state.reading) {
        process.nextTick(nReadingNextTick, this);
      } else if (state.length) {
        emitReadable(this);
      }
    }
  }

  return res;
}
```

```javascript
//process.stdin.resum.toString()
function () {
  var state = this._readableState;
  if (!state.flowing) {
    debug('resume');
    state.flowing = true;
    resume(this, state);
  }
  return this;
}
```

通过对比发现都把`process._readableState.flowing`设置成`true`, 并调用了一个内部的`resume(stream, state)`方法

顺藤摸瓜找到了
[nodejs/node/lib/_stream_readable.js#L802](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/lib/_stream_readable.js#L802)

```javascript
Readable.prototype.resume = function() {
  var state = this._readableState;
  if (!state.flowing) {
    debug('resume');
    state.flowing = true;
    // 让进程得以保持运行的根源
    resume(this, state); 
  }
  return this;
};

// 第一步
function resume(stream, state) {
  if (!state.resumeScheduled) {
    state.resumeScheduled = true;
    // 启动了一个nextTick任务执行_resume函数
    process.nextTick(resume_, stream, state);
  }
}

// 第二步
function resume_(stream, state) {
  if (!state.reading) {
    debug('resume read 0');
    stream.read(0);
  }

  state.resumeScheduled = false;
  state.awaitDrain = 0;
  stream.emit('resume');
  flow(stream);
  if (state.flowing && !state.reading)
    stream.read(0);
}

Readable.prototype.pause = function() {
  debug('call pause flowing=%j', this._readableState.flowing);
  if (false !== this._readableState.flowing) {
    debug('pause');
    this._readableState.flowing = false;
    this.emit('pause');
  }
  return this;
};

function flow(stream) {
  const state = stream._readableState;
  debug('flow', state.flowing);
  while (state.flowing && stream.read() !== null); 
  // 通过一个循环执行read方法
}
```

一开始我猜想可能`flow`方法是原因，但通过包装(monkeypactching)`process.stdin.read` 发现并没有反复执行。所以这里并不是因为造就了一个死循环才使得进程运行的。而且这样做也会消耗大量的CPU资源。

我们继续深入阅读read方法

```javascript
// you can override either this method, or the async _read(n) below.
Readable.prototype.read = function(n) {
  debug('read', n);
  n = parseInt(n, 10);
  var state = this._readableState;
  var nOrig = n;

  if (n !== 0)
    state.emittedReadable = false;

  if (n === 0 &&
      state.needReadable &&
      (state.length >= state.highWaterMark || state.ended)) {
    debug('read: emitReadable', state.length, state.ended);
    if (state.length === 0 && state.ended)
      endReadable(this);
    else
      emitReadable(this); //
    return null;
  }

  n = howMuchToRead(n, state);

  if (n === 0 && state.ended) {
    if (state.length === 0)
      endReadable(this);
    return null;
  }
 
  var doRead = state.needReadable;
  debug('need readable', doRead);

  if (state.length === 0 || state.length - n < state.highWaterMark) {
    doRead = true;
    debug('length less than watermark', doRead);
  }

  if (state.ended || state.reading) {
    doRead = false;
    debug('reading or ended', doRead);
  } else if (doRead) {
    debug('do read');
    state.reading = true;
    state.sync = true;
    if (state.length === 0)
      state.needReadable = true;
    this._read(state.highWaterMark);
    state.sync = false;
    if (!state.reading)
      n = howMuchToRead(nOrig, state);
  }

  var ret;
  if (n > 0)
    ret = fromList(n, state);
  else
    ret = null;

  if (ret === null) {
    state.needReadable = true;
    n = 0;
  } else {
    state.length -= n;
  }

  if (state.length === 0) {
    if (!state.ended)
      state.needReadable = true;

    if (nOrig !== n && state.ended)
      endReadable(this);
  }

  if (ret !== null)
    this.emit('data', ret);

  return ret;
};
```

通过对所有关键方法进行了调试输出
```javascript
console.log(process.stdin._readableState.flowing);
console.log(process.stdin._readableState.resumeScheduled);

monkeyPatching(process.stdin, "resume", function (fn, ...args) {
  console.log('process.stdin resume')
  return fn(...args)
});

monkeyPatching(process.stdin, "read", function (fn, ...args) {
  console.log('process.stdin read', args[0])
  return fn(...args)
});

monkeyPatching(process.stdin, "_read", function (fn, ...args) {
  console.log('process.stdin _read', args[0]);
  console.log(fn.raw.toString());
  return fn(...args)
});

monkeyPatching(process.stdin, "unshift", function (fn, ...args) {
  console.log('process.stdin unshift')
  return fn(...args)
});

monkeyPatching(process.stdin, "push", function (fn, ...args) {
  console.log('process.stdin push')
  return fn(...args)
});

process.stdin.on("data", function(data){
  console.log("on data...", data);
});
process.stdin.on("end", function (data) {
  console.log("on end...", data);
});
process.stdin.on("readable", function(){
  console.log("on readable...");
  console.log("this._readableState.reading", process.stdin._readableState.reading);
  console.log("this._readableState.resumeScheduled", process.stdin._readableState.resumeScheduled)
});
```

尝试一番后，发现原来是调用了`_read`方法。通过查找`_read`方法的内容，`stdin`的`_read`方法居然来自于`net`模块([lib/net.js#L472](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/lib/net.js#L472))

```javascript
Socket.prototype._read = function(n) {
  debug('_read');

  if (this.connecting || !this._handle) {
    debug('_read wait for connection');
    this.once('connect', () => this._read(n));
  } else if (!this._handle.reading) {
    // not already reading, start the flow
    debug('Socket._read readStart');
    this._handle.reading = true;
    var err = this._handle.readStart();
    if (err)
      this.destroy(errnoException(err, 'read'));
  }
};
```

从中找到了`stdin._handle.readStart`

```
node -e "process.stderr._handle.readStart()"
```

这是一个原生绑定的方法, 当我们调用`stdin`的方法侦听事件或者读入流时，最终会调用这个`stdin._handle.readStart`使得`libuv`循环推入了一个任务，接收用户从tty中输入的数据。

定义`stdin._handle.readStart`方法的位置在[src/stream_base-inl.h#L57](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/src/stream_base-inl.h#L57)

这个`readStart`方法是以个虚函数会被子类具体实现的，那么`stdin`的对应的是哪个`readStart`呢？

在`lib/net.js`, 可以看到每个`Stream`创建时都会有一个对应的`_handle`, 这个`_handle`可能是参数传入或者通过判断`fd`自动构造。

判断`fd`的方法是`TTYWrap.guessHandleType`
```javascript
function createHandle(fd) {
  var type = TTYWrap.guessHandleType(fd);
  if (type === 'PIPE') return new Pipe();
  if (type === 'TCP') return new TCP();
  throw new TypeError('Unsupported fd type: ' + type);
}
```

`TTYWrap`来自于C++的`tty_wrap`模块。

通过搜索`TTYWrap.guessHandleType`，以及对`guessHandleType`断点, 定位到`stdin`的`_handle`是在`lib/internal/process/stdio.js`中创建的。

> [lib/internal/process/stdio.js#L42](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/lib/internal/process/stdio.js#L42) 

```javascript
function getStdin() {
    if (stdin) return stdin;

    const tty_wrap = process.binding('tty_wrap');
    const fd = 0;

    switch (tty_wrap.guessHandleType(fd)) {
      case 'TTY':
        var tty = require('tty');
        stdin = new tty.ReadStream(fd, {
          highWaterMark: 0,
          readable: true,
          writable: false
        });
        break;
```

当`tty_wrap.guessHandleType(fd)`返回`TTY`时创建一个`tty`可读流

> [lib/tty.js](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/lib/tty.js#L37)

```javascript
const { TTY, isTTY } = process.binding('tty_wrap');

//...

function ReadStream(fd, options) {
  if (!(this instanceof ReadStream))
    return new ReadStream(fd, options);
  if (fd >> 0 !== fd || fd < 0)
    throw new errors.RangeError('ERR_INVALID_FD', fd);

  options = util._extend({
    highWaterMark: 0,
    readable: true,
    writable: false,
    handle: new TTY(fd, true) // <==
  }, options);

  net.Socket.call(this, options);

  this.isRaw = false;
  this.isTTY = true;
}
```

接下来查看了`tty_wrap`模块对应的代码

> [src/tty_wrap.h](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/src/tty_wrap.h#L33)

> [src/tty_wrap.cc](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/src/tty_wrap.cc#L48)

可以看到的是`tty_wrap`作为一个`handle`并没有直接实现`readStart`方法。`TTY`继承自`StreamWrap`。(在较新的源码中改名为了`LibuvStreamWrap`)

> [src/stream_wrap.cc#L155](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/src/stream_wrap.cc#L155)

`src/stream_wrap.cc#L155`里面的`stream()`是一个`uv_stream_t`, 是通过构造函数传进来`uv_handle_t`类型转换得来的，来自于 [src/tty_wrap.cc#L176](https://github.com/nodejs/node/blob/2146c88bc7383ce66839e08cc22da17a51f5cacf/src/tty_wrap.cc#L176)

```c++
uv_tty_init(env->event_loop(), &handle_, fd, readable)
```

`uv_stream_t`和`uv_tty_init`之前在uvbook里也没有介绍这两个类型。接下来就是进一步查文档了解了。

但可以推断调用`process.stdin.on`或者`process.stdin.pipe`等方法时，进程得以保持运行的原因，有了进一步的了解。如下

#### process.stdin的环境初始化

1) lib/internal/bootstrap_node.js
    ```javascript
    NativeModule.require('internal/process/stdio').setup()
    ```

2) lib/internal/process/stdio.js
    - `getStdin()`

3) lib/tty.js 
    - `new tty.ReadStream(fd`

4) src/stream_wrap.cc
    ```c++
    uv_tty_init(env->event_loop(), &handle_, fd, readable)
    ```

#### 用户调用的接口

1) `process.stdin.on` / `process.stdin.pipe` 等等
2) lib/_stream_readable.js
      
    - `process.stdin.resume`
    - `process.stdin.read`

3) lib/net.js
    - `process.stdin._read`
    - `process.stdin._handle.readStart`

4) src/stream_wrap.cc
    - `StreamWrap::readStart`
    - `uv_read_start`
