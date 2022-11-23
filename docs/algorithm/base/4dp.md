



## 1背包问题

### 01背包问题



有 N 件物品和一个容量是 V 的背包。每件物品只能使用一次。

第 i 件物品的体积是 vi，价值是 wi。

求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
输出最大价值。



#### 输入格式

第一行两个整数，N，V，用空格隔开，分别表示物品数量和背包容积。

接下来有 N 行，每行两个整数 vi,wi，用空格隔开，分别表示第 i 件物品的体积和价值。

#### 输出格式

输出一个整数，表示最大价值。

#### 数据范围

0<N,V≤1000
0<vi,wi≤1000

#### 输入样例

```
4 5
1 2
2 4
3 4
4 5
```

#### 输出样例：

```
8
```



#### code

##### 未优化



```cpp
#include <iostream>

using namespace std;
const int N = 1010;
int v[N], w[N], n, m, f[N][N];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            f[i][j] = max(f[i][j], f[i - 1][j]);
            if (j >= v[i]) {
                f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i]);
            }

        }
    }

    cout << f[n][m];
    return 0;
}
```



##### 优化

```cpp
#include <iostream>

using namespace std;
const int N = 1010;
int v[N], w[N], n, m, f[N];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];

    for (int i = 1; i <= n; i++) {
        for (int j = m; j >= v[i]; j--) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }

    cout << f[m];
    return 0;
}
```



### 完全背包



有 N 种物品和一个容量是 V 的背包，每种物品都有无限件可用。

第 ii 种物品的体积是 vi，价值是 wi。

求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
输出最大价值。

#### 输入格式

第一行两个整数，N，V，用空格隔开，分别表示物品种数和背包容积。

接下来有 N 行，每行两个整数 vi,wi，用空格隔开，分别表示第 i 种物品的体积和价值。

#### 输出格式

输出一个整数，表示最大价值。

#### 数据范围

0<N,V≤1000
0<vi,wi≤1000

#### 输入样例

```
4 5
1 2
2 4
3 4
4 5
```

#### 输出样例：

```
10
```



#### code



##### 未优化

```cpp
#include <iostream>

using namespace std;
const int N = 1010;
int v[N], w[N], n, m, f[N][N];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            for (int k = 0; k * v[i] <= j; k++) {
                f[i][j] = max(f[i][j], f[i - 1][j - k * v[i]] + k * w[i]);
            }
        }
    }

    cout << f[n][m];
    return 0;
}
```



##### 优化



```cpp
#include <iostream>

using namespace std;
const int N = 1010;
int v[N], w[N], n, m, f[N];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i];

    for (int i = 1; i <= n; i++) {
        for (int j = v[i]; j <= m; j++) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }

    cout << f[m];
    return 0;
}
```





```txt
f[i][j] = i-1j, i-1j-v + w, i-1j-2v + 2w, ...

fij-v = i-1j-v, i-1j-2v+w, ... 
  
ij = max(i-1j, ij-v + w)
  
```









![](https://raw.githubusercontent.com/imattdu/img/main/img/202205272312420.png)















### 多重背包1

有 N 种物品和一个容量是 V 的背包。

第 i 种物品最多有 si 件，每件体积是 vi，价值是 wi。

求解将哪些物品装入背包，可使物品体积总和不超过背包容量，且价值总和最大。
输出最大价值。

#### 输入格式

第一行两个整数，N，V，用空格隔开，分别表示物品种数和背包容积。

接下来有 N 行，每行三个整数 vi,wi,si，用空格隔开，分别表示第 i 种物品的体积、价值和数量。

#### 输出格式

输出一个整数，表示最大价值。

#### 数据范围

0<N,V≤100
0<vi,wi,si≤100

#### 输入样例

```
4 5
1 2 3
2 4 1
3 4 3
4 5 2
```

#### 输出样例：

```
10
```



```cpp
#include <iostream>

using namespace std;
const int N = 110;
int n, m, v[N], w[N], s[N];
int f[N][N];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> v[i] >> w[i] >> s[i];

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= m; j++) {
            for (int k = 0; k * v[i] <= j && k <= s[i]; k++) {
                f[i][j] = max(f[i][j], f[i - 1][j - k * v[i]] + k * w[i]);
            }
        }
    }

    cout << f[n][m];
    return 0;
}
```





### 多重背包II

有 NN 种物品和一个容量是 VV 的背包。

第 ii 种物品最多有 sisi 件，每件体积是 vivi，价值是 wiwi。

求解将哪些物品装入背包，可使物品体积总和不超过背包容量，且价值总和最大。
输出最大价值。

#### 输入格式

第一行两个整数，N，V，用空格隔开，分别表示物品种数和背包容积。

接下来有 NN 行，每行三个整数 vi,wi,sivi,wi,si，用空格隔开，分别表示第 ii 种物品的体积、价值和数量。

#### 输出格式

输出一个整数，表示最大价值。

#### 数据范围

0<N≤10000<N≤1000
0<V≤20000<V≤2000
0<vi,wi,si≤20000<vi,wi,si≤2000

##### 提示：

本题考查多重背包的二进制优化方法。

#### 输入样例

```
4 5
1 2 3
2 4 1
3 4 3
4 5 2
```

#### 输出样例：

```
10
```



0 1023



1 2 4 8 ... 512



0 - 512

可以凑出 0 1023任意数





```cpp
#include <iostream>

using namespace std;
const int N = 11 * 2000 + 10, M = 2010;
int n, m, v[N], w[N], f[N];

int main() {
    cin >> n >> m;
    int cnt = 0;
    for (int i = 1; i <= n; i++) {
        int a, b, c, k = 1;
        cin >> a >> b >> c;
        while (k <= c) {
            cnt++;
            v[cnt] = k * a;
            w[cnt] = k * b;
            c -= k;
            k *= 2;
        }
        if (c > 0) {
            cnt++, v[cnt] = c * a, w[cnt] = c * b;

        }
    }

    for (int i = 1; i <= cnt; i++) {
        for (int j = m; j >= v[i]; j--) {
            f[j] = max(f[j], f[j - v[i]] + w[i]);
        }
    }

    cout << f[m];
    return 0;
}
```











## 4.2





时间复杂度：

状态数量 * 转移计算量





```cpp
f[i][j] = max(f[i - 1][j], f[i - 1][j]) + a[i][j];
```

注意初始化每行多一个点













```cpp
#include <iostream>

using namespace std;
const int N = 510, INF = 0x3f3f3f3f;
int n, f[N][N], a[N][N];

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            cin >> a[i][j];
        }
    }
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= i + 1; j++) {
            f[i][j] = -INF;
        }
    }


    f[1][1] = a[1][1];
    for (int i = 2; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            f[i][j] = max(f[i -1][j - 1], f[i - 1][j]) + a[i][j];
        }
    }
    int res = -INF;
    for (int i = 1; i <= n; i++) {
        res = max(res, f[n][i]);
    }
    cout << res;
    return 0;
}
```

























### 最长公共子序列

给定两个长度分别为 N 和 M 的字符串 A 和 B，求既是 A 的子序列又是 B 的子序列的字符串长度最长是多少。

#### 输入格式

第一行包含两个整数 N 和 M。

第二行包含一个长度为 N 的字符串，表示字符串 A。

第三行包含一个长度为 M 的字符串，表示字符串 B。

字符串均由小写字母构成。

#### 输出格式

输出一个整数，表示最大长度。

#### 数据范围

1≤N,M≤1000

#### 输入样例：

```
4 5
acbd
abedc
```

#### 输出样例：

```
3
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202205302021741.png)





```cpp
#include <iostream>

using namespace std;
const int N = 1010;
int f[N][N], n, m;
string a, b;

int main() {
    scanf("%d %d", &n, &m);
    cin >> a >> b;
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            f[i][j] = max(f[i -1][j], f[i][j - 1]);
            if (a[i - 1] == b[j - 1]){
                f[i][j] = max(f[i][j], f[i - 1][j - 1] + 1);
            }
        }
    }
    
    cout << f[n][m];
    return 0;
}
```







### 石子合并



设有 NN 堆石子排成一排，其编号为 1，2，3，…，N。

每堆石子有一定的质量，可以用一个整数来描述，现在要将这 N 堆石子合并成为一堆。

每次只能合并相邻的两堆，合并的代价为这两堆石子的质量之和，合并后与这两堆石子相邻的石子将和新堆相邻，合并时由于选择的顺序不同，合并的总代价也不相同。

例如有 4 堆石子分别为 `1 3 5 2`， 我们可以先合并 1、2 堆，代价为 4，得到 `4 5 2`， 又合并 1，2堆，代价为 9，得到 `9 2` ，再合并得到 11，总代价为 4+9+11=24；

如果第二步是先合并 2，3堆，则代价为 7，得到 `4 7`，最后一次合并代价为 11，总代价为 4+7+11=22。

问题是：找出一种合理的方法，使总的代价最小，输出最小代价。

#### 输入格式

第一行一个数 N 表示石子的堆数 N。

第二行 N 个数，表示每堆石子的质量(均不超过 1000)。

#### 输出格式

输出一个整数，表示最小代价。

#### 数据范围

1≤N≤300

#### 输入样例：

```
4
1 3 5 2
```

#### 输出样例：

```
22
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202205300227884.png)





```cpp
min{f[i][k] + f[k+1][j] + s[j] - s[i - 1]}
k= i -> j-1
```







```cpp
#include <iostream>

using namespace std;
const int N = 310;
int s[N], f[N][N];

int main() {
    int n;
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%d", &s[i]);
        s[i] += s[i - 1];
    }
    
    for (int len = 2; len <= n; len++) {
        for (int i = 1; i + len - 1 <= n; i++) {
            int j = i + len - 1;
            f[i][j] = 1e8;
            for (int k = i; k < j; k++) {
                f[i][j] = min(f[i][j], f[i][k] + f[k + 1][j] + s[j] - s[i - 1]);
            }
        }
    }
    cout << f[1][n];
    return 0;
}
```













```txt
1 <= xxx1yyy <= abcdefg


1.xxx=000 ~ abc-1, yyy=000~999  abc * 1000

2.xxx = abc

2.1 d < 1 abc1yyy > abc0efg 0
2.2 d = 1 yyy = 000 ~ efg efg+1
2.3 d > 1 yyy = 000 ~ 999 1000


  
  
  
```





```txt



f[i,j] i列
j:行数二进制 10010


```











```
1 <= xxx1yyy <= abcdefg


1. xxx=000~abc yyy=(000~999) -> abc*1000
2. xxx=abc
	d<1 abc1yyy yyy=0
	d=1 yyy=(000~efg) -> efg+1
	d>1 yyy=(000~999) -> 1000
	
	
	
```











```
f[ij]

i列
j 行数二进制


1.j&k == 0
2.j|k 不存在连续奇数个0
```









```
f[ij] 从0走到j,走过所有的点是i的所有路径
i:二进制 1走过 0没有走过


```









