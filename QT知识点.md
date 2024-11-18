# QT基础

## 一、三大核心机制

### 信号槽

信号槽的五种连接方式（图略）  
connect(信号发出者，信号，信号接收者，槽，连接方式(隐藏默认自动连接))//五个参数

### 元对象系统

元对象系统分为三大类: QObject类、Q_OBJECT宏 和 元对象编译器 moc  
Qt的类包含Q_OBJECT宏 moc编译器会对该类编译成标准的C++代码

### 事件模型

事件的创建  
鼠标事件，键盘事件，窗口调整事件，模拟事件  
事件的交付  
Qt通过调用虚函数QObject::event()来交付事件。  
事件循环模型  
主事件循环通过调用QCoreApplication::exec()启动， 随着QCoreApplication::exit()结束，本地的事件循环可用利用QEventLoop构建。  
一般来说，事件是由触发当前的窗口系统产生的，但也可以通过使用 QCoreApplication::sendEvent()和
QCoreApplication::postEvent()来手工产生事件。需要说明的是QCoreApplication::sendEvent()会立即发送事件， QCoreApplication::postEvent()则会将事件放在事件队列中分发。  
自定义事件

## 二、信号和槽

### 五大连接方式

#### Qt::AutoConnection

一般信号槽不会写第五个参数，其实使用的默认值，使用这个值则连接类型会在信号发送时决定。

```C++
static QMetaObject::Connection connect(const QObject *sender, const QMetaMethod &signal,
                        const QObject *receiver, const QMetaMethod &method,
                        Qt::ConnectionType type = Qt::AutoConnection);
```

如果接收者和发送者在同一个线程，则自动使用Qt::DirectConnection类型。  
如果接收者和发送者不在一个线程，则自动使用Qt::QueuedConnection类型。

#### Qt::DirectConnection

槽函数会在信号发送的时候直接被调用，槽函数运行于信号发送者所在线程，有点类似于回调函数。  
效果看上去就像是直接在信号发送位置调用了槽函数。这个在多线程环境下比较危险，可能会造成奔溃。

#### Qt::QueuedConnection

槽函数在控制回到接收者所在线程的事件循环时被调用，槽函数运行于信号接收者所在线程。  
发送信号之后，槽函数不会立刻被调用，等到接收者的当前函数执行完，进入事件循环之后，槽函数才会被调用。多线程环境下一般用这个。

#### Qt::BlockingQueuedConnection

槽函数的调用时机与Qt::QueuedConnection一致，不过发送完信号后发送者所在线程会阻塞，直到槽函数运行完。  
使用该连接方式，接收者和发送者绝对不能在一个线程，否则程序会死锁！！！在多线程间需要同步的场合可能需要这个。

#### Qt::UniqueConnection

这个flag可以通过按位或（|）与以上四个结合在一起使用。当这个flag设置时，当某个信号和槽已经连接时，再进行重复的连接就会失败，也就是避免了重复连接。

### 原理

Qt信号槽的实现原理其实就是函数回调，不同的是直连直接回调、队列连接使用Qt的事件循环隔离了一次达到异步，最终还是使用函数回调

+ moc预编译帮助我们构建了信号槽回调的开头(信号函数体)和结尾(qt_static_metacall回调函数)，中间的回调过程Qt已经在QOjbect函数中实现
+ signals和slots就是为了方便moc解析我们的C++文件，从中解析出信号和槽
+ 信号槽总共有5种连接方式，前四种是互斥的，可以表示为异步和同步。第五种唯一连接时配合前4种方式使用的
+ 信号和槽本质上是一样的，但是对于使用者来说，信号只需要声明，moc帮你实现，槽函数声明和实现都需要自己写
+ connect方法就是把发送者、信号、接受者和槽存储到发送者的内存中，供后续执行信号时查找。存储方式是不同信号对应不同下标，用数组存储，快速查找，相同信号不同槽函数，用链表存储，快速删除和插入
+ 信号触发就是一系列函数回调

## 三、Qthread和QtConcurrent

+ QThread可以使用信号和槽机制，而QtConcurrent不支持。
+ QThread可以设置线程优先级，而QtConcurrent不支持。
+ QThread可以实现跨平台多线程，而QtConcurrent只能在支持C++11的平台上实现。
+ QThread可以实现更复杂的多线程任务，而QtConcurrent只能实现简单的多线程任务

## 四、数据流(QDataStream) 和文件流(QTextStream)

Qt中的数据流(QDataStream)和文件流(QTextStream)都是用于读写数据的流类，但它们有以下区别：

+ 数据类型：数据流(QDataStream)支持Qt中的所有基本数据类型和自定义类型，如QString、QByteArray等，
而文件流(QTextStream)只能读写文本数据，不支持二进制数据。
+ 数据格式：数据流(QDataStream)是二进制格式，可以直接读写二进制数据，而文件流(QTextStream)是文本格式，
只能读写文本数据，对于二进制数据需要进行编码和解码。
+ 数据编码：文件流(QTextStream)默认使用Unicode编码，可以通过设置不同的编码格式来读写不同的文本数据，
而数据流(QDataStream)不需要进行编码，它直接以二进制形式读写数据。
+ 应用场景：数据流(QDataStream)适用于读写二进制数据，例如读写文件、网络传输等场景，
而文件流(QTextStream)适用于读写文本数据，例如读写配置文件、日志文件等场景。

综上所述，数据流(QDataStream)和文件流(QTextStream)虽然都是Qt中的流类，但它们的数据类型、格式、编码和应用场景等方面有所不同，需要根据具体的需求选择合适的流类进行读写操作。

## 五、Qt中的指针

### QPointer

QPointer是Qt中的智能指针，它可以自动管理指针的生命周期，并且可以检测被管理的对象是否已经被销毁。
如果被管理的对象已经被销毁，QPointer会自动将指针置为nullptr。
QPointer通常用于管理QWidget对象等需要在程序运行期间动态创建和销毁的对象。

### QScopedPointer

QScopedPointer是Qt中的局部智能指针，它可以自动管理指针的生命周期，
并且可以在指针超出作用域时自动释放指针指向的内存。
QScopedPointer通常用于管理局部变量和堆内存对象等需要在作用域结束时自动释放的对象。

### QSharedPointer

QSharedPointer是Qt中的共享智能指针，它可以在多个指针之间共享同一个对象，
并且可以自动管理指针的生命周期。当最后一个指向对象的QSharedPointer被销毁时，
QSharedPointer会自动释放对象的内存。QSharedPointer通常用于管理需要在多个地方使用同一个对象的情况。

### QWeakPointer

QWeakPointer是Qt中的弱智能指针，它类似于QSharedPointer，但是不会增加对象的引用计数。
QWeakPointer通常用于在多个指针之间共享同一个对象时，避免产生循环引用问题。

### QSharedDataPointer

QSharedDataPointer是Qt中的共享数据指针，它可以在多个对象之间共享同一个数据，而不需要共享整个对象。
QSharedDataPointer通常用于实现copy-on-write（COW）技术，避免不必要的复制操作。

## 六、内存管理机制

所有继承自QOBJECT类的类，如果在new的时候指定了父亲，那么它的清理时在父亲被delete的时候delete的，所以如果一个程序中，所有的QOBJECT类都指定了父亲，那么他们是会一级级的在最上面的父亲清理时被清理，而不用自己清理；

程序通常最上层会有一个根的QOBJECT，就是放在setCentralWidget（）中的那个QOBJECT，这个QOBJECT在 new的时候不必指定它的父亲，因为这个语句将设定它的父亲为总的QAPPLICATION，当整个QAPPLICATION没有时它就自动清理，所以也无需清理。9这里QT4和QT3有不同，QT3中用的是setmainwidget函数，但是这个函数不作为里面QOBJECT的父亲，所以QT3中这个顶层的QOBJECT要自行销毁）。

这是有人可能会问那如果我自行delete掉这些QT接管负责销毁的指针了会出现什么情况呢，如果时这样的话，正常情况下QT的拥有这个对象的那个父亲会知道这件事情，它会直到它的儿子被你直接DELETE了，这样它会将这个儿子移出它的列表，并且重新构建显示内容，但是直接这样做时有风险的！也就是要说的下一条。

当一个QOBJECT正在接受事件队列时如果中途被你DELETE掉了，就是出现问题了，所以QT中建议大家不要直接DELETE掉一个 QOBJECT，如果一定要这样做，要使用QOBJECT的deleteLater()函数，它会让所有事件都发送完一切处理好后马上清除这片内存，而且就算调用多次的deletelater也不会有问题。

QT不建议在一个QOBJECT 的父亲的范围之外持有对这个QOBJECT的指针，因为如果这样外面的指针很可能不会察觉这个QOBJECT被释放，会出现错误，如果一定要这样，就要记住你在哪这样做了，然后抓住那个被你违规使用的QOBJECT的destroyed（）信号，当它没有时赶快置零你的外部指针。当然我认为这样做是及其麻烦也不符合高效率编程规范的，所以如果要这样在外部持有QOBJECT的指针，建议使用引用或者用智能指针，如QT就提供了智能指针针对这些情况，见一条。

QT中的智能指针封装为QPointer类，所有QOBJECT的子类都可以用这个智能指针来包装，很多用法与普通指针一样，可以详见QT assistant 通过调查这个QT的内存管理功能，发现了很多东西，现在觉得虽然这个QT弄的有点小复杂，但是使用起来还是很方便的，

要说的是某些内存泄露的检测工具会认为QT的程序因为这种方式存在内存泄露，发现时大可不必理会。

## 七、自定义控件

### 方法

+ 从外观设计上: QSS、继承绘制函数重绘、继承QStyle相关类重绘 、组合拼装等等
+ 从功能行为上:重写事件函数、添加或者修改信号和槽等等

## 八、QWidget和QML的区别

+ QWidget是一种基于C++的桌面应用程序开发技术，主要用于开发桌面应用程序，它是一种面向对象的技术，
可以使用C++语言来实现用户界面的设计和编程。
+ QML是一种基于JavaScript的应用程序开发技术，主要用于开发桌面应用程序和移动应用程序，
它是一种基于声明式的技术，可以使用JavaScript语言来实现用户界面的设计和编程。
+ 两者的本质有所不同，QWidget是基于C++的，QML是基于JavaScript的；使用上也有所不同，QWidget是面向对象的，QML是基于声明式的。

## 九、QObject的理解

+ `QObject` 类是Qt 所有类的基类。
+ `QObject`是Qt对象模型的核心。这个模型的中心要素就是一种强大的叫做信号与槽无缝对象沟通机制。
你可以用 `connect()` 函数来把一个信号连接到槽，也可以用`disconnect()` 函数来破坏这个连接。
为了避免永无止境的通知循环，你可以用`blockSignal()` 函数来暂时阻塞信号。
保护函数 `connectNotify()` 和 `disconnectNotify()` 可以用来跟踪连接。
+ 对象树都是通过`QObject` 组织起来的，当以一个对象作为父类创建一个新的对象时，
这个新对象会被自动加入到父类的 `children()` 队列中。这个父类有子类的所有权。
能够在父类的析构函数中自动删除子类。可以通过`findChild()`和`findChildren()` 函数来寻找子类。
+ 每个对象都一个对象名称`objectName()` ，而且它的类名也可以通过`metaObject()`函数。
你可以通过`inherits()` 函数来决定一个类是否继承其他的类。当一个对象被删除时，
它会发射`destory()` 信号，你可以抓住这个信号避免某些事情。
+ 对象可以通过`event()` 函数来接收事情以及过滤来自其他对象的事件。
就好比`installEventFiter()` 函数和`eventFilter()` 函数。`childEvent()` 函数能够重载实现子对象的事件。
+ `QObject`还提供了基本的时间支持，`QTimer` 类 提高了更高层次的时间支持。
+ 任何对象要实现信号与槽机制，`Q_OBJECT` 宏都是强制的。你也需要在源原件上运行元对象编译器。
不管是否真正用到信号与槽机制，最好在所有`QObject`子类使用 `Q_OBJECT` 宏，以避免出现一些不必要的错误。
+ 所有的 `Qt widgets` 都是基础`QObject`。如果一个对象是`widget`,那么 `isWidgetType()` 函数就能判断出。
