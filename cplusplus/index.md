---
title: cplusplus
layout: page
comments: yes
---

### c plus plus

- **vector空间增长方式**

为了支持快速随机访问，vector将元素连续存储。当不得不获取新的内存空间时，vector和string的实现通常会分配比新的空间需求更大的内存空间。容器预留这些空间作为备用，从而减少容器空间重新分配次数。

容器大小管理操作有：

- c.shrink_to_fit()：将capacity()减少为与size()相同大小，退回不需要的内存空间。shrink_to_fit知识一个请求，标准库并不保证退还内存，依赖于具体实现
- c.capacity()：不重新分配内存空间的话，c可以保存多少元素
- c.reserve(n)：分配至少能容纳n个元素的内存空间
- c.size()：已经保存的元素的数目

只有当需求的内存空间超过当前容量时，reserve调用才会改变vector的容量。如果需求大小大于当前容量，reserve至少分配与需求一样大的内存空间（可能更大）；如果需求大小小于或者等于当前容量，reserve什么都不做，不会退回内存空间。类似的，resize函数只改变容器中元素的数目，而不是容器的容量，同样不能使用其减少容器预留的内存空间。

总之，通过在一个初始为空的vector上调用n次push_back来创建一个n个元素的vector，所花费的时间不能超过n的常数倍。

string与vector类似。

- **容器是否线程安全**

Answer from [http://stackoverflow.com/questions/1362110/is-the-c-stdset-thread-safe](http://stackoverflow.com/questions/1362110/is-the-c-stdset-thread-safe) says:

> A single object is thread safe for reading from multiple threads. For example, given an object A, it is safe to read A from thread 1 and from thread 2 simultaneously.

> If a single object is being written to by one thread, then all reads and writes to that object on the same or other threads must be protected. For example, given an object A, if thread 1 is writing to A, then thread 2 must be prevented from reading from or writing to A.

> It is safe to read and write to one instance of a type even if another thread is reading or writing to a different instance of the same type. For example, given objects A and B of the same type, it is safe if A is being written in thread 1 and B is being read in thread 2.

However, more precisely, concurrency aspect is left out as an implementation detail for different compilers. So the documentation that comes with your compiler is where one should look for answers related to concurrency. STL has three versions of implementation: SGI STL, STL PORT, VISUAL C++. Most of the STL implementation are not thread safe so much. But for concurrent reads of same object from multiple threads most implementations of STL are indeed safe. For SGI implementation:

> The SGI implementation of STL is thread-safe only in the sense that simultaneous accesses to distinct containers are safe, and simultaneous read accesses to to shared containers are safe. If multiple threads access a single container, and at least one thread may potentially write, then the user is responsible for ensuring mutual exclusion between the threads during the container accesses.

> And for G++: We currently use the SGI STL definition of thread safety.

> And for STLPort: Please refer to SGI site for detailed document on thread safety. Basic points are: simultaneous read access to the same container from within separate threads is safe; simultaneous access to distinct containers(not shared between threads) is safe; user must provide synchronization for all access if any thread may modify shared container.

线程安全：一个函数被称为thread-safe，当且仅当被多个并发进程反复调用时，它会一直产生正确的结果。所以对于vector的operator []函数来说，是线程安全的，不安全是指你自己代码级别的线程不安全。reference operate []虽然返回的是可以写操作的内容，但是其实反复调用这个函数是不会做写操作的。如何vector提供类似set(npos, value)函数，一定是不安全的。

> Note: For a vector<int> x with a size greater than one, x[1] = 5 and *x.begin() = 10 can be executed concurrently without a data race, but x[0] = 5 and *x.begin() = 10 executed concurrently may result in a data race. As an exception to the general rule, for a vector <bool> y, y[0] = true may race with y[1] = true.

多线程对vector<int>**不同元素**的读写操作是安全的，WHY？计算机取指令不是按byte取，而是按照机器字长（block）取数据，对于32位机器，一个机器字长为32位（4Byte），而x[0], x[1], x[2]..各元素为4字节，占满一个机器字长，所以取元素时候不会发生竞争。

vector<bool> x就不一样了，c++用char实现bool，char/bool长度为1字节，所以取x[0]时，会顺带取出x[1]，x[2]，x[3]。

另，32位机器int 4字节，long 8字节；64位机器int 8字节，long 8字节。int和long无区别。

- **map的时空复杂度**

Why is std::map implemented as a red-black tree? 

Probably the two most common self balancing tree algorithms are Red-Black Trees and AVL Trees. Both algorithms use the notion of rotations where the nodes of the tree are rotated to perform the re-balancing. While in both algorithms the insert/delete operations are O(log n), in the case of Red-Black Tree rebalancing rotation is an O(1) operation while with AVL is a O(log n) operation, the former is more efficient is this aspect of the re-balancing stage.

If your application does not have too many insertion and deletion operations, but weights heavily on searching, then AVL tree probably is a good choice. st::map uses Red-Black tree as it gets a reasonable trade-off between the complexity of node insertion/deletion and searching.

- **allocator原理**

[STL中的内存分配原理](http://blog.csdn.net/effective_coder/article/details/8873513)

- **父类和子类中构造函数以及析构函数的调用顺序**

[C++奇奇怪怪的题目之构造析构顺序](http://gaocegege.com/Blog/cpp/cppclass)

- **引用和指针的区别**

引用总是指向一个对象，没有null reference。所以，如果有可能指向一个对象也有可能不指向对象则必须使用指针。由于不存在null reference，所以使用前不需要进行有效性测试，而使用指针则需要测试其有效性。

[指针和引用的区别](http://blog.csdn.net/dujiangyan101/article/details/2844138)

- **右值引用的特点以及应用场景**

很多情况下都会发生对象拷贝，对象拷贝后就立即被销毁，若改成移动对象则会大幅度提升性能。为了支持移动操作，新标准引入一种新的引用类型：右值引用。通过&&获取右值引用，右值引用有个非常重要的性质：只能绑定到一个将要销毁的对象上。类似于任何引用（普通引用，也称为左值引用），一个右值引用也不过是某个对象的别名而已。

**左值持久；右值短暂**。左值有持久的状态，而右值要么是字面常量，要么是在表达式求值过程中创建的临时对象。由于右值引用只能绑定到临时对象，我们得知：右值引用的对象将要被销毁，该对象没有其他用户。这两个特性意味着使用右值引用的代码可以自由地接管所引用的对象的资源。

**标准库move函数**。虽然不能讲一个右值引用绑定到一个左值上，但我们可以显式地将一个左值转换为对应的右值引用类型。调用move新标准库函数来获得绑定到左值的右值引用。`int &&rr3 = std::move(rr1)`，调用move意味着除了对rr1赋值或销毁外，我们将不再使用它。我们可以销毁一个移后源对象，也可以赋予它新值，但不能使用一个移后源对象的值。

**移动构造函数**。为了让我们自己的类支持移动移动操作，需要为其定义移动构造函数和移动赋值函数。这两个成员类似对应的拷贝操作，但它们从给定对象“窃取”资源而不是拷贝资源。除了完成资源移动，移动构造函数还必须确保移后源对象必须处在这样一个状态：销毁它是无害的。特别是，一旦资源移动完成，源对象必须不再指向被移动的资源，这些资源所有权已经归属新创建的对象。

```
StrVec::StrVec(StrVec &&s): elements(s.elements), first_free(s.first_free), cap(s.cap) {
	// 令s进入这样的状态：对其运行析构函数是安全的
	s.elements = s.first_free = s.cap = nullptr;
}
```

**移动赋值运算符**。

```
StrVec& StrVec::operator=(StrVec&& rhs) noexcept {
	if (this != &rhs) {
		free(); // 释放已有元素
		elements = rhs.elements;
		first_free = rhs.first_free;
		cap = rhs.cap;
		rhs.elements = rhs.first_free = rhs.cap = nullptr;
	}
}
```

**合成的移动操作**

与处理拷贝构造函数和拷贝赋值函数已有，编译器也会合成移动构造函数和移动赋值运算符。如果一个类定义了自己的拷贝构造函数、拷贝赋值运算符或者析构函数，编译器不会为它合成移动构造函数和移动赋值运算符。如果一个类没有移动操作，通过正常的函数匹配，类会使用对应的拷贝操作代替移动操作。只有当一个类没有定义任何自己版本的拷贝控制成员，且类的每个非static数据成员都可以移动时，编译器才会为它合成移动构造函数或移动赋值运算符。编译器可以移动内置类型的成员，如果一个成员是类类型，且该类有对应的移动操作，编译器也能移动这个成员。

**移动右值，拷贝左值**。如果一个类既有移动构造函数，也有拷贝构造函数，编译器使用普通的函数匹配规则来确定使用哪个构造函数。

```
StrVec v1, v2;
v1 = v2; //v2是左值；使用拷贝赋值
StrVec getVec(istream &); //getVec返回一个右值
v2 = getVec(cin); //getVec(cin)是一个右值；使用移动赋值
```

**如果没有移动构造函数，右值也被拷贝**。如果一个类有一个拷贝构造函数但未定义移动构造函数，编译器不会合成移动构造函数。函数匹配规则保证该类型的对象会被拷贝，即使我们试图调用move来移动它们时也是如此。`Foo z(std::move(x))`，拷贝构造函数，因为未定义移动构造函数。我们将Foo&&转换为一个const Foo&。