---
layout: mypost
title:  Learning RxSwfit Grammar.
---

# RxSwift语法内容

读```【RxSwift - Reactive Programming With Swift】```之余所记录下来的笔记，练手项目地址 [RxSwiftProject](https://github.com/aberfield/WorkSpace/tree/master/RxSwiftProject)

### Observables

### 什么是 Observable
+ Observable应该是可观察实例，实质上它其实是一个Sequence(序列)，这点可以看看Observable的源码。Observable其实就是数据流，所有的操作都会通过Observable传输。

### 怎么创建一个Observable

+  Just 只能发出一种特定事件的可观察实例

```
     let observable = Observable<Int>.just(one)
```

+ of 创建一个sequence能发出很多种事件信号

```
     let observable2 = Observable.of(one,two,three)
```

+ from  从集合中创建Sequence 例如数组、字典、集合

```
   let observable3 = Observable.from([one,two,three]) 
```

+  empty  一个没有事件的订阅，直接调用complete方法

```
        /// empty  一个没有事件的订阅，直接调用complete方法
        example("empty") {
            let observable = Observable<Void>.empty()
            observable.subscribe(onNext: { element in
                print(element)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).dispose()
        }
    
        --- Example of: empty ---
    completed
    disposed
```


+ nerver 永远都不会执行事件的订阅

```
        /// never 不会执行的一个订阅
        example("nerver") {
            let observable = Observable<Any>.never()
            observable.subscribe(onNext: { element in
                print(element)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).dispose()
        }
```

+ range 创建一个Sequence，它可以发出一个范围内的的所有事件

```
        /// range 创建一个Sequence，他会发出从开始到结束范围的事件
        example("range") {
            Observable<Int>.range(start: 1, count: 10).subscribe(onNext: { (i) in
                let n = Double(i)
                let fibonacci = Int(((pow(1.61803, n) - pow(0.61803, n)) / 2.23606).rounded())
                print(fibonacci)
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("complete")
            }, onDisposed: {
                print("disposed")
            }).dispose()
        }

        --- Example of: range ---
0
1
2
3
5
8
13
21
34
55
complete
disposed
```

+ create 创建一个自定义的Sequence,可以在定义里面的OnNext，onComplete，OnError

```
        /// 自定义可观察的sequence事件
        example("create") {
            
            
            let disposeBag = DisposeBag()
            
            let create =  Observable<String>.create({ (observer) -> Disposable in
                observer.onNext("1")
                
                observer.onError(MyError.anError)
                
                observer.onCompleted()
                observer.onNext("?")
                
                
                return Disposables.create()
            })
            
            create.subscribe(onNext: { (a) in
                print(a)
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
        }
```

+ deferred 为每个订阅者创建一个新的可观察序列,在下面代码中可以看到，每一次的subscribe都有创建DeferredSequence,这样Sequence通道不会同时处理几个Observable。

```
        example("deferred") {
            let disposeBag = DisposeBag.init()
            
            var flip = false
            
            let factory:Observable<Int> = Observable.deferred({ () -> Observable<Int> in
                flip = !flip
                if flip {
                    return Observable.of(1,2,3)
                }else {
                    return Observable.of(4,5,6)
                }
            })
            
            for _ in 0...3 {
                factory.subscribe(onNext: {
                    print($0,terminator:"\n")
                }, onCompleted: {
                    print("completed")
                }, onDisposed: {
                    print("disposed")
                }).disposed(by: disposeBag)
            }
            
        }

  --- Example of: deferred ---
1
2
3
completed
disposed
4
5
6
completed
disposed
1
2
3
completed
disposed
4
5
6
completed
disposed      
```


+ Single 只会发送一次事件，超过一次发送则会报错。可用用于串行报错处理，如下，用于读取文件报错

```
        example("Single") {
            let disposeBag = DisposeBag()
            <!-- 错误枚举 -->
            enum FileReadError:Error {
                case fileNotFound, unreadable, encodingFailed
            }
            
            <!-- 读取文件内容 -->
            func loadText(_ name:String) -> Single<String> {
                return Single.create(subscribe: { (single) -> Disposable in
                    let disposable = Disposables.create()
                    
                    guard let path = Bundle.main.path(forResource: name, ofType: "txt") else {
                        single(.error(FileReadError.fileNotFound))
                        return disposable
                    }
                    
                    guard let data = FileManager.default.contents(atPath: path) else {
                        single(.error(FileReadError.unreadable))
                        return disposable
                    }
                    
                    guard let contents = String.init(data: data, encoding: .utf8) else {
                        single(.error(FileReadError.encodingFailed))
                        return disposable
                    }
                    
                    single(.success(contents))
                    return disposable
                    
                })
            }
            
            loadText("Copyright").subscribe({ single in
                switch single {
                case .success(let string):
                    print(string)
                case .error(let error):
                    print(error)
                }
            }).disposed(by: disposeBag)
        }

 print:

--- Example of: Single ---
fileNotFound
```

+ PublishSubject PublishSubject只能接受在订阅之后的事件，例子如下，需要注意的事.completed、.error 这种结束事件不符合此规则。

```
        example("PublishSubject") {
            let subject = PublishSubject<String>()
            
            subject.onNext("Is anyone listening")
            
            
            let subscriptionOne = subject.subscribe(onNext: { (string) in
                print("subscriptionOne \(string)")
            }, onError: { (error) in
                print("error")
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            })
            
            subject.on(.next("1"))
            subject.on(.next("2"))
            subject.onNext("6")
            /// 结束订阅的事件，subscriptionTwo也能接受到
            //  subject.onCompleted()
            
            let subscriptionTwo = subject.subscribe(onNext: { (string) in
                print("subscriptionTwo \(string)")
            }, onError: { (error) in
                print("error")
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            })
            
            
            subject.on(.next("3"))
            subject.on(.next("4"))
            subject.onNext("7")
            
            subscriptionTwo.dispose()
            subscriptionOne.dispose()
            
            subject.onNext("9")
        }
        
print: 
--- Example of: PublishSubject ---
subscriptionOne 1
subscriptionOne 2
subscriptionOne 6
subscriptionOne 3
subscriptionTwo 3
subscriptionOne 4
subscriptionTwo 4
subscriptionOne 7
subscriptionTwo 7
disposed
disposed

``` 

+ BehaviorSubject 当订阅了BehaviorSubject，可以接受订阅之前的最后一个事件，例子如下

```
        example("BehaviorSubject") {
            let subject = BehaviorSubject.init(value: "initial Value")
            let disposeBag = DisposeBag()
            
            subject.subscribe(onNext: { (event) in
                print("1,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            /// 这个事件 2订阅可以收到
            subject.onNext("X")
            
            
            subject.subscribe(onNext: { (event) in
                print("2,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            /// 这个事件 3订阅可以收到
            subject.onNext("Y")
            
            
            subject.subscribe(onNext: { (event) in
                print("3,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            subject.onNext("Z")
        }

print:
--- Example of: BehaviorSubject ---
1,initial Value
1,X
2,X
1,Y
2,Y
3,Y
1,Z
2,Z
3,Z
disposed
disposed
disposed

```

+  ReplaySubject 根据bufferSize 来接受订阅前的第几个事件。

```
        // ReplaySubject 根据bufferSize 来接受订阅前的第几个事件。
        example("ReplaySubject") {
            /// 其实bufferSize 为1 时，其实事件处理就相当于BehaviorSubject。
            let subject = ReplaySubject<String>.create(bufferSize: 2)
            
            let disposeBag = DisposeBag()
            
            subject.onNext("1")
            
            subject.onNext("2")
            
            subject.onNext("3")
            
            
            subject.subscribe(onNext: { (event) in
                print("subject:1,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            subject.onNext("4")
            
            // 订阅2 可以接受到 额外接受到 事件3、事件4
            subject.subscribe(onNext: { (event) in
                print("subject:2,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            subject.onNext("5")
            
            /// 在发送error 事件后，replay机制还是在处理
            //            subject.onError(MyError.anError)
            
            /// dispose之后 不会触发replay
            //            subject.dispose()
            
            
            subject.subscribe(onNext: { (event) in
                print("subject:3,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            // 订阅3可以收到 事件4、事件5
            subject.onNext("6")
            
        }
```

+ Variable  是BehaviorSubject一个包装箱，就像是一个箱子一样，使用的时候需要调用asObservable()拆箱，里面的value是一个BehaviorSubject，他不会发出error事件，但是会自动发出completed事件。在接下来的几个版本可能会被移除，建议使用BehaviorRelay，下面是BehaviorRelay的用法

```
        // 和Variable效果一致
        example("BehaviorRelay") {
            let behaviorRelay = BehaviorRelay<String>(value: "initial value")
            
            let disposeBag = DisposeBag()
            
            behaviorRelay.accept("new initial value")
            
            behaviorRelay.asObservable().subscribe(onNext: { (event) in
                print("subject:1,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            behaviorRelay.accept("1")
            
            
            behaviorRelay.asObservable().subscribe(onNext: { (event) in
                print("subject:2,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
            behaviorRelay.accept("2")
            
            behaviorRelay.asObservable().subscribe(onNext: { (event) in
                print("subject:3,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
        }

print 
--- Example of: BehaviorRelay ---
subject:1,new initial value
subject:1,1
subject:2,1
subject:1,2
subject:2,2
subject:3,2
disposed
disposed
disposed
```

+ Maybe 可能发出一个元素，要么发出一个complete，要么发出一个error事件。适合那种可能发出一个元素，有可能不发出直接发出complete或者error事件。可以通过asMaybe将一个事件流转化成Maybe事件
```
<!-- 订阅MaybeObserver 之后， 只能收到一个.success事件。 -->
          let maybeObserver = Maybe<String>.create(subscribe: { (maybe) -> Disposable in
                maybe(.success("maybe success"))
                maybe(.success("other success"))

                maybe(.completed)
                
                
                return Disposables.create()
            })
            
```

+ Completeable 要么只能产生一个 completed 事件，要么产生一个 error 事件，不会发出任何元素。
```
<!-- 如下，当发出一个.completed事件之后，整个流程就相当于走完了 -->
          let completed = Completable.create(subscribe: { (complteable) -> Disposable in
                let success: Bool = (arc4random() % 4 == 0)
                if success {
                    complteable(.completed)
                }else {
                    complteable(.error(MyError.anError))
                }
                return Disposables.create()
            })
```


### 订阅事件处理方法，筛选过滤，指定执行事件。

+ ignoreElement，忽略掉所有元素事件，只发出error 和 completed事件。
```
<!-- 使用ignoreElements之后，订阅只有complete和error两种模式 -->
            let strikes = PublishSubject<String>.init()
            strikes.init().ignoreElements().subscribe(onCompleted: {
                print("成功")
            }, onError: { (error) in
                print("失败")
            }).disposed(by: disposeBag)
            
            
            strikes.onNext("X")

            strikes.onCompleted()

```
+ ElementAt 只执行指定位置的事件，下标是0开始，就是选定某个事件订阅。

+ Filter 根据条件过滤掉一些事件，只有符合条件的内容才能被订阅到。
```
        // 根据条件过滤掉一些事件
        example("filter") {
            let subject = Observable<Int>.of(1,2,3,4,5,6,7,8)
            let disposeBag = DisposeBag()
            subject.filter{
                 return $0 % 2 == 0
                }.subscribe(onNext: { (event) in
                print("subject:3,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
        }
```

+ skip 可以过滤掉指定n前的事件
```
<!-- 过滤掉前面四个事件 -->
            let subject = Observable<Int>.of(1,2,3,4,5,6,7,8)
            let disposeBag = DisposeBag()
            subject.skip(4).subscribe(onNext: { (event) in
                    print("subject:3,\(event)")
                }, onError: { (error) in
                    print(error)
                }, onCompleted: {
                    print("completed")
                }, onDisposed: {
                    print("disposed")
                }).disposed(by: disposeBag)
```

+ skipWhile，跳过满足条件的事件，但是只要出现过不满足条件事件，则skipWhile则会失效
```
            let subject = Observable<Int>.of(1,2,3,4,5,6,7,8)
            let disposeBag = DisposeBag()
            subject.skipWhile{
                return $0 < 4
                }.subscribe(onNext: { (event) in
                print("subject:3,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
```

+ skipuntil 一直跳过当前观察者的事件直到接收到了另外一个观察者事件。
```
<!-- 在这里，1事件会被直接过滤掉，当时接受到9事件之后，subject才会生效，所以会输出4 -->
            let disposeBag = DisposeBag()
            let subject = PublishSubject<Int>()
            let trigger = PublishSubject<Int>()
            subject.skipUntil(trigger).subscribe(onNext: { (i) in
                print(i)
            }, onError: { (error) in
                print("error")
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            subject.onNext(1)
            trigger.onNext(9)
            subject.onNext(4)
```

+ take 和 skip相反，skip是跳过指定多少的事件，take只会处理标志事件之前的内容，满足条件会自动.completed
```
            let subject = Observable<Int>.of(1,2,3,4,5,6)
            
            let disposeBag = DisposeBag()
            
            subject.take(3).subscribe(onNext: { (event) in
                print("subject:,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
```

+ takeWhile 依次判断 Observable 序列的每一个值是否满足给定的条件。 当第一个不满足条件的值出现时，它便自动完成。
```
            let subject = Observable<Int>.of(2,2,4,4,6,6)
            let disposeBag = DisposeBag()
            subject.enumerated().takeWhile{
                $0 < 3 && $1 % 2 == 0
                }.subscribe(onNext: { (event) in
                print("subject:,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
```

+   takeUnit 收到 Target事件消息后会调用.Completed方法。这个方法可以用来处理可能存在对象被释放后还去操作的问题
```
<!--  在这种情况下 如果self.rx已经被释放， self.rx.deallocated已经调用了 那么 subscribe中是不会再去执行。 -->
         someObservable
         .takeUntil(self.rx.deallocated)
         .subscribe(onNext: {
         print($0)
        })
```

+ distinctUntilChanged: 用来过滤掉连续重复的事件
```
          let disposeBag = DisposeBag()
            Observable<String>.of("A","A","B","B","A").distinctUntilChanged().subscribe(onNext: { (event) in
                print("subject:3,\(event)")
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
```

+  toArray:组装成数组
```
        example("toArray") {
            let bag = DisposeBag.init()
            
            Observable.of(1,2,3,4).toArray().subscribe(onNext: { (i) in
                print(i)
            }).disposed(by: bag)
        }
```

+ map :变换事件,函数可以对原有序列里面的事件元素进行改造，返回的还是原来的序列。
```
            let bag = DisposeBag()
            let formatter = NumberFormatter.init()
            formatter.numberStyle = .spellOut
            Observable<NSNumber>.of(1,3,555)
                .map{ formatter.string(from: $0) ?? ""}
                .subscribe(onNext: { (a) in
                    print(a)
                })
                .disposed(by: bag)
```

+ flatMap：对原有序列中的元素进行改造和处理，每一个元素返回一个新的sequence，然后把每一个元素对应的sequence合并为一个新的sequence序列。map函数 + 合并函数 = flatMap函数


+ materialize：可以将error completed 等结束事件 打包成普通事件。

+ dematerialize：对应Materialize，可以将打包成普通事件的 结束事件转化为结束事件。

### 怎么销毁Observable
+ dispose 通过dispose，在执行完OnComplete或者OnError之后立马消化

```
        /// 销毁方法，将这个订阅事件销毁
        example("dispose") {
            Observable.of("A","B","C").subscribe({ (event) in
                if let element = event.element {
                    print(element)
                }
            }).dispose()
        }
```

+  disposeBag 通过创建一个释放池，在某个时机将池子中所有Observable全部释放销毁，一般业务中可以在ViewDidAppear 或者 deinit销毁bag。

```
        /// 销毁事件集合，类似于Autorelease
        example("disposeBag") {
            let disposeBag = DisposeBag()
            
            Observable.of("A","B","C").subscribe{
                print($0.element ?? "1")
                }.disposed(by: disposeBag)
        }
```


+ create 创建一个自定义的Sequence,可以在定义里面的OnNext，onComplete，OnError


```
        /// 自定义可观察的sequence事件
        example("create") {
            
            
            let disposeBag = DisposeBag()
            
            let create =  Observable<String>.create({ (observer) -> Disposable in
                observer.onNext("1")
                
                observer.onError(MyError.anError)
                
                observer.onCompleted()
                observer.onNext("?")
                
                
                return Disposables.create()
            })
            
            create.subscribe(onNext: { (a) in
                print(a)
            }, onError: { (error) in
                print(error)
            }, onCompleted: {
                print("completed")
            }, onDisposed: {
                print("disposed")
            }).disposed(by: disposeBag)
            
        }
        
```

> Practice makes permanent. By completing these challenges, you’ll practice what you’ve learned in this chapter > and pick up a few more tidbits of knowledge about working with observables. A starter playground workspace as well as the finished version of it are provided for each challenge. Enjoy!


### RxSwift操作符决策树

> [如何选择操作符](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/content/decision_tree.html)


### 结束语

1. Bjarne Stroustrup（C++之父）说：
* 逻辑应该是清晰的，bug难以隐藏。
* 依赖最少，易于维护。
* 错误处理完全根据一个明确的策略。
* 性能接近最佳，避免代码混乱和无原则的优化。
* 整洁的代码只做一件事。
2. Michael Feathers（《修改代码的艺术》作者）说：
* 整洁的代码看起来总是像很在乎代码质量的人写的。
* 没有明显的需要改善的地方。
* 代码的作者似乎考虑到了所有的事情。
* 可以感受到，对好的代码的理解有很多共通的地方：
* 代码简单，代码意图明确，其他人才容易与你协作。
* 可读性和可维护性要高。
* 以最合适的方式解决问题。
