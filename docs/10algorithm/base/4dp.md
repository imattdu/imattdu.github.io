

### 01背包问题



有 NN 件物品和一个容量是 VV 的背包。每件物品只能使用一次。

第 ii 件物品的体积是 vivi，价值是 wiwi。

求解将哪些物品装入背包，可使这些物品的总体积不超过背包容量，且总价值最大。
输出最大价值。



#### 输入格式

第一行两个整数，N，VN，V，用空格隔开，分别表示物品数量和背包容积。

接下来有 NN 行，每行两个整数 vi,wivi,wi，用空格隔开，分别表示第 ii 件物品的体积和价值。

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













a b c d

求abc最大值







fij

fij-v





0 1023



1 2 4 8 ... 512



0 - 512

可以凑出 0 1023任意数













第i组物品选哪个

不选 选第一个 2 3 4 5 6







