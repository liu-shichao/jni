1.为什么拷贝赋值函数和移动赋值函数都要返回自身的引用？

```
答：
1.返回自身的原因，可以实现连续赋值操作，a=b=c,等价为a=b返回a,a=c,最终的结果是a=c.
2.返回引用，是避免返回的时候构造临时的对象，而且临时对象不被接受的话会很快析构，也就是多了临时变量的一次构造和一次析构的而外开销。
```