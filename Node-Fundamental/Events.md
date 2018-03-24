
### 1. 基本用法 Basic usage
基本上常见的两种用法，
1. 从EventEmitter继承,在新的constructor上调用EventEmitter.

web,桌面客户端，手机用户界面有一个共同点:都是事件驱动的。
事件Events是处理一些内在异步操作的很好的范例:比如用户输入。
下面用一个music player的例子讲解EventEmitter如何工作的。
继承1的示例代码。

```JS
const util = require('util');
const events = require('events');
const AudioDevice = {
        play: function(track) {   
          // Stub: Trigger playback through iTunes, mpg123, etc.
        },

        stop: function() {
        }
      };

function MusicPlayer() {
  this.playing = false;   //The class’s state can be configured, and then EventEmitter’s //constructor can be called as required.
  events.EventEmitter.call(this);
}

// The inherits method copies the methods from one prototype into another— this is the general pattern for creating classes based on EventEmitter.
util.inherits(MusicPlayer, events.EventEmitter);

var musicPlayer = new MusicPlayer();

musicPlayer.on('play', function(track) {
  this.playing = true;
  AudioDevice.play(track);
});

musicPlayer.on('stop', function() {
  this.playing = false;
  AudioDevice.stop();
});

musicPlayer.emit('play', 'The Roots - The Fire');

setTimeout(function() {
  musicPlayer.emit('stop');
}, 1000);
```
上面的例子可能看起来不是很明显，假设当play被触发的时候我们需要做一些其他的事-也许就是用户界面需要更新一下。
通过添加其他的listeners给这个play事件就可以解决。
```JS
var util = require('util');
var events = require('events');

function MusicPlayer() {
  this.playing = false;
  events.EventEmitter.call(this);
}
util.inherits(MusicPlayer, events.EventEmitter);

var musicPlayer = new MusicPlayer();

musicPlayer.on('play', function(track) {
  this.playing = true;
});

musicPlayer.on('stop', function() {
  this.playing = false;
});

musicPlayer.on('play', function(track) {    //New listeners can be added as needed.
  console.log('Track now playing:', track);
});

musicPlayer.emit('play', 'The Roots - The Fire');

setTimeout(function() {
  musicPlayer.emit('stop');
}, 1000);
```

移除监听器很简单
```JS
function play(track) {  //A reference to the listener is required to be able to remove it.
  this.playing = true;
}

musicPlayer.on('play', play);

musicPlayer.removeListener('play', play);

```
使用emitter.removeAllListeners移除所有的监听器。
utils.inherits工作的原理是ES5的Object.create方法。
在MusicPlayer构造器函数中用了events.EventEmitter.call(this);
是为了让示例附在domain上，如果domain被创建了。
版本8里domain已经被移除了。

2. 混合EventEmitter
在已经存在一个base class的情况下，就不能直接继承了，可行的方法是复制。将EventEmitter的方法用循环复制到这个基类中。
示例代码
```JS
const EventEmitter = require('events').EventEmitter;

function MusicPlayer(track) {
  this.track = track;
  this.playing = false;

  for (var methodName in EventEmitter.prototype) {        // This is the for-in loop that copies the relevant properties.
    this[methodName] = EventEmitter.prototype[methodName];
} }

MusicPlayer.prototype = {
  toString: function() {
    if (this.playing) {
      return 'Now playing: ' + this.track;
    } else {
      return 'Stopped';
    }
  }
};

const musicPlayer = new MusicPlayer('Girl Talk - Still Here');

musicPlayer.on('play', function() {
  this.playing = true;
  console.log(this.toString());
});

musicPlayer.emit('play');
```

### 2.处理错误Error handling
大多数事件都是同等对待，除了错误事件。
用EventEmitter处理错误有自己独特的规则。
先看官网中的处理error 的示例

下面的代码发现error会直接导致node崩溃。
```JS
const myEmitter = new MyEmitter();
myEmitter.emit('error', new Error('whoops!'));
// Throws and crashes Node.js
```
更优雅的处理方式。
```JS
const myEmitter = new MyEmitter();

myEmitter.on('error', (err) => {
  console.error('whoops! there was an error');
});

myEmitter.emit('error', new Error('whoops!'));
// Prints: whoops! there was an error
```
示例代码
```JS
var util = require('util');
var events = require('events');

function MusicPlayer() {
  events.EventEmitter.call(this);
}

util.inherits(MusicPlayer, events.EventEmitter);

var musicPlayer = new MusicPlayer();

musicPlayer.on('play', function(track) {
  this.emit('error', 'unable to play!');
});

musicPlayer.on('error', function(err) {       //Listening for an error event
  console.error('Error:', err);
});

setTimeout(function() {
  musicPlayer.emit('play', 'Little Comets - Jennifer');
}, 1000);

```
关于Node的newListener事件。
官方文档的说明
The EventEmitter instance will emit its own 'newListener' event before a listener is added to its internal array of listeners.
Listeners registered for the 'newListener' event will be passed the event name and a reference to the listener being added.

使用newListener事件来追踪监听器。
```JS
const util = require('util');
const events = require('events');

function EventTracker() {
  events.EventEmitter.call(this);
}

util.inherits(EventTracker, events.EventEmitter);

const eventTracker = new EventTracker();

eventTracker.on('newListener', function(name, listener) {   //Track whenever new listeners are added.
  console.log('Event name added:', name);
});

eventTracker.on('a listener', function() {
  // This will cause 'newListener' to fire
});
```
官网的一个示例
Any additional listeners registered to the same name within the 'newListener' callback will be inserted before the listener that is in the process of being added.
```JS
const myEmitter = new MyEmitter();
// Only do this once so we don't loop forever
myEmitter.once('newListener', (event, listener) => {
  if (event === 'event') {
    // Insert a new listener in front
    myEmitter.on('event', () => {
      console.log('B');
    });
  }
});
myEmitter.on('event', () => {
  console.log('A');
});
myEmitter.emit('event');
// Prints:
//   B
//   A
```
