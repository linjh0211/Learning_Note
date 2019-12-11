### 一、尽量不要用using namespace xxx

-  WHY？

因为它会把xxx命名空间的所有内容引进到当前命名空间，造成污染。

- HOW？

方法一：使用类似std::vector<int>的方式 

方法二：用作用域限制命名空间的可见性，eg :

```
class A {
	using namespace std;
}

void fun(){
	using namespace std;
}
```

