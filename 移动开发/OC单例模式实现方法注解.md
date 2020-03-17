# OC单例模式实现方法注解

## 单例对象获取方法：
	//单例，显然这里要用类方法获取单例对象
	 + (instancetype) sharedInstance  
	 {  
	   static Singleton * _instance = nil;
	   static dispatch_once_t onceToken;
	
	   dispatch_once(&onceToken, ^{ 
	  	                               _instance = [[self alloc] init];
	                                });  
	   return _instance ;  
	 }  

## 注解：
- `static Singleton *_instance = nil`这里使用静态变量是为了保证对象被创建后在整个程序生命周期内不被释放。
- ` + (instancetype) sharedInstance;`显然只能使用类方法。
- `static dispatch_once_t onceToken;` `dispatch_once_t`是long类型。因为单例对象只能在整个程序结束后被释放，所以onceToken必须是static或global类型。
- `void dispatch_once( dispatch_once_t *predicate, dispatch_block_t block);`是同步函数，函数会等待block里的操作都执行完成才返回。这个函数有`predicate`和`block`两个形参。
	- `predicate`是`dispatch_once_t`类型的指针，用来测试传入的block是否执行完成。
	- `block`是`dispatch_block_t`类型的变量，`dispatch_block_t`是没有参数没有返回值的block类型。`block`指向传入的block对象，这个block在整个程序生命周期内只会被执行一次，block里执行的操作是创建单例对象。

