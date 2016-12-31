# Periscope
-------------
## Project code 

### 1. common/
* blocking_queue.h
    - `explict`关键字
```cpp
template <typename T>
class BlockingQueue {
 public:
  explicit BlockingQueue(size_t capacity) : capacity_(capacity) {}
  ...
 private:
   const size_t capacity_;

   std::weak_ptr<std::condition_variable> alt_data_avail_;
}
```
> 用*explicit*关键字修饰 constructor，防止参数类型的隐式转换，详见[此处](dump/Why You Should Always Use explicit Constructors · Simon's Graphics Blog.html)。

    - `std::mutex`，互斥量，被互斥量锁住的代码段叫临界区，A mutex is a *lockable object* that is designed to signal when critical sections(临界区) of code need exclusive access, preventing other threads with the same protection from executing concurrently and access the same memory locations.
```cpp
#include <iostream>   // std::cout
#include <mutex>      // std::mutex
#include <thread>     // std::thread

std::mutex mtx;       // mutex for critical section
void print_block(int n, char c) {
    mtx.lock(); 	  // critical section (exclusive access to std::cout signaled by locking mtx)
    for (int i=0; i<n; ++i) { std::cout << c; }
    std::cout << '\n';
    mtx.unlock();
}
int main() {
	std::thread th1 (print_block, 50, '*');
	std::thread th2 (print_block, 50, '$');
	th1.join();
	th2.join();
    return 0;
}
```
> `mtx.lock()` 和 `mtx.unlock()`夹住的区域形成了一个primitive单元，原子区、临界区。

    - `std::unique_lock`, A unique lock is an object that manages a mutex object with unique ownership in both states: locked and unlocked. On construction, the object requires a mutex object, for whose locking and unlocking operations becomes responsible. 
    The objects supports both states: locked and unlocked. The `unique_lock` class guarantees an unlocked status on destruction(even if not called explicitly).
    见下面两个例子，第一个例子就没有显式调用 unlock，第二个例子就显式调用了。
    ```cpp
    // unique_lock example
	#include <iostream>       // std::cout
	#include <thread>         // std::thread
	#include <mutex>          // std::mutex, std::unique_lock

	std::mutex mtx;           // mutex for critical section

	void print_block (int n, char c) {
	  // critical section (exclusive access to std::cout signaled by lifetime of lck):
	  std::unique_lock<std::mutex> lck (mtx);
	  for (int i=0; i<n; ++i) { std::cout << c; }
	  std::cout << '\n';
	}

	int main ()
	{
	  std::thread th1 (print_block,50,'*');
	  std::thread th2 (print_block,50,'$');

	  th1.join();
	  th2.join();

	  return 0;
	}
	```
	```cpp
	std::mutex mu_;
	std::unique_lock<std::mutex> lck(mu_);
	...blablabla...
	lck.unlock(); // 显式调用unlock,上个例子中就把这步省了

	```
	- cpp中的 *reference*即为别名。 

    - `std::list<T>`
      `list::font`, returns a reference to the first element in the list container.
      Unlike member `list::begin` which returns an iterator to this same element, this function returns a direct reference. Calling this function on an empty container causes undefined behavior. 
      > 所以下面代码中先检查了列表`data_`是否为空，然后才`data_.front()`,注意类的私有数据成员，一般在尾部添加个下划线`_`.

          `list::pop_font`, removes the first element in the list container, effectively reducing its size by one.
          > 所以当想pop(返回并删除)列表中元素时，经常是`list::front`和`list::pop_front`结合使用，如下代码所示。

	      ```cpp
		  private:
		    std::list<T> data_;

		  bool TryGet(T* out) {
		    std::unique_lock<std::mutex> lk(mu_);
		    if (data_.empty()) {
		      return false;
		    }
		    *out = data_.front(); // 将指针out指向的数据赋为data_的第一个数据 
		    data_.pop_front();
		    lk.unlock();
		    unused_space_.notify_one();
		    return true;
		  }
	      ```


    - `std::thread` constructor
    > `std::thread(run_function, arg1, arg2, ..)`, args are arguments for *run_function*
    > The constructor's arguments are the function the thread will execute, followed by the function's parameters.

    - `std::thread::join`, the function returns when the thread execution has completed. 
    This synchronizes the moment this function returns with the completion of all the operations in the thread: 
    > `t.join()`阻塞了调用`t.join()`的线程的执行，直到`t.join()`函数返回了，才能继续执行。
    > Joining means that the thread who invoked the new thread will wait for the new thread to finish execution, before it will continue its own execution.

        After a call to `t.join()`, the thread object `t` becomes non-joinable and can be destroyed safely.
    ```cpp
	// example for thread::join
	#include <iostream>       // std::cout
	#include <thread>         // std::thread, std::this_thread::sleep_for
	#include <chrono>         // std::chrono::seconds
	 
	void pause_thread(int n) 
	{
	  std::this_thread::sleep_for (std::chrono::seconds(n));
	  std::cout << "pause of " << n << " seconds ended\n";
	}
	```
	```	 
	int main() 
	{
	  std::cout << "Spawning 3 threads...\n";
	  std::thread t1 (pause_thread,3);
	  std::thread t2 (pause_thread,2);
	  std::thread t3 (pause_thread,1);
	  std::cout << "Done spawning threads. Now waiting for them to join:\n";
	  /*
	   * 如果将此三行注释掉，输出结果为：
	   * Spawning 3 threads...
	   * Done spawning threads. Now waiting for them to join:
	   * All threads joined!
	   *
	  t1.join();
	  t2.join();
	  t3.join();
	  */
	  std::cout << "All threads joined!\n";
	  return 0;
	}
	/* 不注释掉，主线程等待t3执行结束返回（3s）后结束，结果为：
	Spawning 3 threads...
	Done spawning threads. Now waiting for them to join:
	pause of 1 seconds ended
	pause of 2 seconds ended
	pause of 3 seconds ended
	All threads joined!
	*/
    ```
    ``` 
	int main() 
	{
	  std::cout << "Spawning 3 threads...\n";
	  /* (1)
	  std::thread t1 (pause_thread,1);
	  std::thread t2 (pause_thread,2);
	  std::thread t3 (pause_thread,3);
	  */
	  /* (2)
	  std::thread t1 (pause_thread,3);
	  std::thread t2 (pause_thread,2);
	  std::thread t3 (pause_thread,1);
	  */
	  std::cout << "Done spawning threads. Now waiting for them to join:\n";
	  t1.join();
	  std::cout << "t1 done" << std::endl;
	  t2.join();
	  std::cout << "t2 done" << std::endl;
	  t3.join();
	  std::cout << "t3 done" << std::endl;
	  std::cout << "All threads joined!\n";

	  return 0;
	}
	/* (1) 结果为：
	Spawning 3 threads...
	Done spawning threads. Now waiting for them to join:
	pause of 1 seconds ended // from t1
	t1 done
	pause of 2 seconds ended // from t2
	t2 done
	pause of 3 seconds ended // from t3
	t3 done
	All threads joined!
	* (2) 结果为：
	Spawning 3 threads...
	Done spawning threads. Now waiting for them to join:
	pause of 1 seconds ended  // from t3
	pause of 2 seconds ended  // from t2
	pause of 3 seconds ended  // from t1
	t1 done
	t2 done
	t3 done
	All threads joined!
	*/
	```

		> 可见，虽然有时我们无法控制 t1,t2,t3返回的先后，但可以保证调用 `tx.join()` 的线程（此处为主线程）内部的同步，比如主线程中出现在 `t1.join()`后面的`std::cout << "t1 done" << std::endl;`语句，一定是在 `t1`执行结束返回后才执行的。

	- `std::conditional_variable`. A condition variable is an object able to block the calling thread until notified to resume. 
	    *unconditional (1)* `void wait(unique_lock<mutex>& lck);`
		*predicate (2)* `void wait(unique_lock<mutex>& lck, Predicate pred);` If *pred* is specified, the function only blocks if *pred* returns *false*, and notifications can only unblock the thread when it becomes *true*.
		`conditional_variable（条件变量）` + `mutex（互斥量）`来实现 `信号量`

		```cpp
		private:
		  std::mutex mu_;
		  std::condition_variable unused_space_;
		  std::condition_variable data_avail_;

	    void SetAltDataAvail(std::shared_ptr<std::condition_variable> alt) {
	      std::unique_lock<std::mutex> lk(mu_);
	      alt_data_avail_ = alt;
	    }

		void Put(const T& elem) {
		    std::unique_lock<std::mutex> lk(mu_);
		    if (data_.size() >= capacity_) {
		      unused_space_.wait(lk, [this] { return data_.size() < capacity_; });
		    }
		    data_.push_back(elem);
		    lk.unlock();
		    data_avail_.notify_one();
		    if (auto alt = alt_data_avail_.lock()) {
		      alt->notify_one();
		    }
		}

	    T Get() {
	      std::unique_lock<std::mutex> lk(mu_);
	      if (data_.empty()) {
	        data_avail_.wait(lk, [this] { return !data_.empty(); });
	      }
	      T elem = data_.front();
	      data_.pop_front();
	      lk.unlock();
	      unused_space_.notify_one();
	      return elem;
	    }
		```
	注意观察 Put 与 Get方法中， `data_avail_` 与 `unused_space_` 的对称结构。`data_avail_` 使得 `data_` 非空时才可以Get,否则 `data_avail_.wait`. `unused_space_` 使得 `data_` 没有满时才可以Put,否则 `unused_space_.wait`. 
	并且当Get了一个数据时进行`unused_space_.notify_one()`,当Put了一个数据时进行`data_avail_.notify_one()`.
	但此处没看懂的是既然维护了 `data_avail_`,又为什么还要维护 `alt_data_avail_` 呢？

		`condition_variable::notify_one`, unblocks one of the threads currently waiting for this condition. If no threads are waiting, the function does nothing. If more than one, it is unspecified which of the threads is selected.
		`condition_variable::wait`, wait until notified. The execution of the current thread(which shall have locked lck's *mutex*) is blocked until notified. At the moment of blocking the thread, the function automatically calls *lck.unlock()*, allowing other locked threads to continue. Once *notified* (explicitly, by some other thread), the function unblocks and calls *lck.lock()*, leaving *lck* in the same state as when the function was called. Then the function returns (notice that this last mutex locking may block again the thread before returning). 因为临界区*mtx*此刻可能被别的thread占着(locked).



		```cpp
		#include <iostream>		
		#include <thread>
		#include <mutex>
		#include <condition_variable>

		std::mutex mtx;
		std::condition_variable produce, consume;

		int cargo=0;	// shared value by producers and consumers

		void consumer() {
			std::unique_lock<std::mutex> lck(mtx);
			while (cargo==0) {
				consume.wait(lck);
			}
			std::cout << cargo << '\n';
			cargo = 0
			produce.notify_one();
		}

		void producer(int id) {
			std::unique_lock<std::mutex> lck(mtx);
			while (cargo > 0) produce.wait(lck);
			cargo = id;
			consume.notify_one();
		}

		int main() {
			std::thread consumers[10], producers[10];
			
			for (int i=0; i<10; ++i) {  // spawn 10 consumers and 10 producers
				consumers[i] = std::thread(consumer);
				producers[i] = std::thread(producer, i+1); 
			}
			for (int i=0; i<10; ++i) {
				producers[i].join();
				consumers[i].join();
			}
			return 0;
		}
		```

	- PLACEHOLDER (Cpp Primer P400)



* rx_thread.h, rx_thread.cc
    * 信号量：可用于*互斥*及*同步*的目的。
    * 互斥量：是包含了优先级继承机制的二进制信号量。二进制信号量更适合于实现同步（在任务之间，或在任务和中断之间），而互斥量更适合于实现简单的互斥。
    * 队列：是任务间通信的主要形式。可使用队列在任务间，在中断与任务间传递消息，在多数时候它们作为线程安全的FIFO缓冲使用，新数据发送到队尾，有时也可发送到队头。生产者-消费者例子，一个消息队列，创建两个线程，一个线程作为生产者，向队列发送消息，另一个线程作为消费者，从队列接收消息。消费者优先级比生产者高，设置为读队列（`Get`）时阻塞。
	> 首先讲一下查到的`RxThread`和`TxThread`命名含义： `RxThread`（Receive）消费者，`TxThread`（To）生产者。线程`RxThread`阻塞于队列，等待数据，得到数据后进行处理，然后再次返回到阻塞于队列的状态。线程`TxThread`重复进入阻塞状态，`TxThread`有消息要发送时离开阻塞状态，通过队列向`RxThread`发送消息，使得`RxThread`收取消息离开阻塞。
	
    * 定义 `void RxThread(SSL* ssl, std::weak_ptr<BlockQueue>)`，创建消费者线程。
    ```cpp
    bool SSL_read_full(SSL* ssl, void* bufv, int bytes_to_read);
    int ReadMessageLength(SSL* ssl);

    template <typename T>
    void RxThread(SSL* ssl, std::weak_ptr<BlockingQueue<T*>> rx) {
      while (int len = ReadMessageLength(ssl)) {
        std::string data;
        data.resize(len);

        if (!SSL_read_full(ssl, &data[0], len)) {
          if (auto p = rx.lock()) { // rx指向的队列仍然存在
            p->Put(nullptr);  // 这步是为啥？为什么Put一个空指针
          }
          break;
        }

        auto msg = std::make_unique<T>();
        if (!msg->ParseFromString(data)) { // protobuf, parse message failed
          if (auto p = rx.lock()) {
            p->Put(nullptr);
          }
          break;
        }

        if (auto p = rx.lock()) {
          p->Put(msg.release());
        } else {
          break;
        }
      }
      SSL_free(ssl);
    }
    ```

    ```
    bool SSL_read_full(SSL* ssl, void* bufv, int bytes_to_read) {
      CHECK(bufv);
      CHECK_GT(bytes_to_read, 0);
      
    }
    ```
    	- `weak_ptr`伴随类，弱引用，不控制所指向对象生存期的智能指针，指向`shared_ptr`所管理的对象。
	`weak_ptr` points to a `shared_ptr` but does not increase its reference count. This means that the underying object can still be deleted even though there is a `weak_ptr` reference to it.
	由于对象可能不存在，我们不能使用`weak_ptr`直接访问对象，而必须要调用`lock()`.此函数检查`weak_ptr`指向的对象是否仍存在。如果存在，`lock`返回一个指向共享对象的`shared_ptr`，只要此 `shared_ptr` 存在，它所指向的底 层对象也就会一直存在。
	[`ParseFromString`][4], parses a message from the given string.

### 2. client/
* start.h
```cpp

#ifndef PERISCOPE_CLIENT_START_H_
#define PERISCOPE_CLIENT_START_H_

#include <string>
#include <memory>
#include "periscope/client/handler.h"

namespace periscope {
namespace client {

void Start(const std::string& name, const std::string& host, int port,
           std::unique_ptr<Handler> handler);

}  // namespace client
}  // namespace periscope

#endif
```

* start.cc
 
    * [gflags, `DEFINE`][2], defining flags in program.
        - Usage: `DEFINE_int32(flag_name, flag_value, flag_description);`
          flag_description: string type
```
DEFINE_int32(periscope_client_rx_queue_size, 100, "");
DEFINE_int32(periscope_client_tx_queue_size, 100, "");
```
 
    * gflags, access the flags
        - All defined flags are available to the program as just a normal variable, with the prefix FLAGS_ prepended.
    ```cpp
	auto rx = std::make_shared<RxQueue>(FLAGS_periscope_client_rx_queue_size);
	auto tx = std::make_shared<TxQueue>(FLAGS_periscope_client_tx_queue_size);
    ```
    > The type of `FLAGS_periscope_client_rx_queue_size` is `google::int32`.

    * `using = `, 别名声明，来定义类型的别名。
    ```cpp
    using SI = Sales_Item; // SI 是 Sales_Item 类型的同义词
    ```
    ```cpp
    // 需用不同的队列来管理不同类型的消息
    // 对于Client来说，`RxQueue`(Recieve Queue)应该是管理接收到的消息，即从服务器发送到客户端的消息（SeverToClient）
    using RxQueue = BlockingQueue<SeverToClientMessage*>;
    // 对于Client来说，`TxQueue`(To Queue)应该是管理发送出去的消息，即从客户端发送到服务器的消息（ClientToSever）
    using TxQueue = BlockingQueue<ClientToSeverMessage*>; 
    ```

    * `继承` `Service`类
    继承机制定义了父子关系，父类定义了所有子类共同的公有接口(*public interface*)和私有实现
	```cpp
	class ServiceImpl final : Service {
	  public:
        explicit ServiceImpl(std::shared_ptr<RxQueue> rx, std::shared_ptr<TxQueue> tx)
        : rx_(rx), tx_(tx) {}
	  private:
	   std::shared_ptr<RxQueue> rx_;
	   std::shared_ptr<TxQueue> tx_;
	   bool stop_received_ = false;
	   std::string stop_reason_;
	   bool rx_non_blocking_ = false;
	}
	```

	* handler.h

	* service.h



### 3. server/


### 4. tests/
* chat_test.cc
	- `std::thread::detach`, why we do `t.detach()` instead of just create the thread and let it run.
	In the destructor of `std::thread`, `std::terminate` is called if: 1) the thread was not joined; 2) and was not detached either
	Thus, you should always `join` or `detach` a thread before the flows of execution reaches the destructor.

	`std::terminate`, calls the current *terminate handler*. By default, the *terminate handler* calls abort. But this behavior can be redefined by calling `set_terminate`. 
	当程序抛出一个 exception 并且没有相应的 catch时就会自动调用`std::terminate`函数.
	通过`std::terminate()`可以显式调用*terminate handler*.
	```cpp
	// terminate example
	#include <iostream>       // std::cout, std::cerr
	#include <exception>      // std::exception, std::terminate

	int main (void) {
	  char* p;
	  std::cout << "Attempting to allocate 1 GiB...";
	  try {
	    p = new char [1024*1024*1024];
	  }
	  catch (std::exception& e) {
	    std::cerr << "ERROR: could not allocate storage\n";
	    std::terminate();      //  显式调用terminate handler(默认是abort)
	  }
	  std::cout << "Ok\n";     // 如果程序已经 abort 就不会输出 Ok
	  delete[] p;
	  return 0;
	}
	```

    * `gflags::ParseCommandLineFlags(&argc, &argv, false)`, a single function call.
    How to process the commandline flags, and set the *FLAG_xx* variables to the appropriate, non-default value based on what is seen on the commandline.跟 `getopt`库中的`getopt()`差不多，只是overhead小了很多。
    通常，`gflags::ParseCommandLineFlags(&argc, &argv, false)` 放在`main()`的开头，*argc*和*argv*被传入`main`函数参数列表中。稍后它们可能被修改，这就是为什么ParseCommandLineFlags中传入的是指向它们的指针。
    The last argument is called "remove_flags". If true, then ParseCommandLineFlags removes the flags and their arguments from *argv* and modifies *argc* appropriately. In this case, after the function call, *argv* will hold only commandline arguments, and not commandline flags. 

    After defining a flag, you may optionally register a *validator function* with the flag. If you do this, after the flag is parsed from the commandline, and whenever its value is changed via a call to *SetCommandLineOption()*, the validator function is called with the new value as an argument. The validator function should return 'true' if the flag value is valid, and false otherwise. If the function returns false for the new setting of the flag, the flag will retain its current value. If it returns false for the default value, ParseCommandLineFlags will die.
    Here is an example use of this functionality:
	```
	static bool ValidatePort(const char* flagname, int32 value) {
	   if (value > 0 && value < 32768)   // value is ok
	     return true;
	   printf("Invalid value for --%s: %d\n", flagname, (int)value);
	   return false;
	}
	DEFINE_int32(port, 0, "What port to listen on");
	// By doing the registration at global initialization time (right after DEFINE_int32)
	// we ensure that the registration happens before the commandline is parsed at the begining of main()
	DEFINE_validator(port, &ValidatePort);
	```

## OpenSSL
 [OpenSSL API][1]

## TCP Keep Alive
> Goal: Keep TCP alive. You will be able to check your connected socket (TCP sockets), and determine whether the connection is still up and running or if it has broken.

[1]: https://www.openssl.org/docs/manmaster/man3/
[2]: https://gflags.github.io/gflags/#define
[3]: 'dump/Why You Should Always Use explicit Constructors · Simon's Graphics Blog.html'
[4]: https://developers.google.com/protocol-buffers/docs/cpptutorial