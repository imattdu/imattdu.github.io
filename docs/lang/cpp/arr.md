





### array



声明&初始化

```cpp
// 声明
int a1[10];
// 初始化
a1[10] = {1, 2, 3}
int a3[10][10] = {
        {
                1, 2, 3
        }
};
```

局部变量未初始化值是随机的

全局变量未初始化则是默认值





sizeof:获取字节大小



```cpp
// 如下方法需要引入 cstring
// 按照字节赋值
memset(a, -1, sizeof a);
// src数组赋值给dst
memcpy(dst, src, sizeof src);
```





### ctrl 控制语句



#### if

```cpp
#include <iostream>

using namespace std;

int main() {
    int i = 10;
    cin >> i;
    if (i < 0) {
        cout << -1;
    } else if (i > 0) {
        cout << 1;
    } else {
        cout << 0;
    }
    return 0;
}
```



#### while
