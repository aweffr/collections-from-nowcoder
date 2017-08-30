# 堆 模板和使用

## 1.堆的实现
```cpp
/*
 * 小根堆的实现
 * 左儿子是自己的下标x2 + 1
 * 右儿子是自己的下标x2 + 2
 */

#include <iostream>
#include <memory.h>

#define MAX_N 10000
int heap[MAX_N], sz = 0;

using namespace std;

void push(int x) {
	// 自己节点的编号
	int i = sz++;

	while (i > 0) {
		// 父亲节点的编号
		int p = (i - 1) / 2;

		// 如果已经没有大小颠倒的就退出
		if (heap[p] <= x) break;

		// 把父亲节点数值放下来, 而把自己提上去
		heap[i] = heap[p];
		i = p;
	}
	heap[i] = x;
}

int pop() {
	// 最小值
	int ret = heap[0];

	// 要提到根的数值
	int x = heap[--sz];

	// 从根开始向下交换
	int i = 0;
	while (i * 2 + 1 < sz) {
		// 比较两个儿子的值
		int a = i * 2 + 1, b = i * 2 + 2;
		if (b < sz && heap[b] < heap[a]) a = b;

		// 如果没有大小颠倒则退出
		if (heap[a] >= x) break;

		// 把儿子的数值提上来
		heap[i] = heap[a];
		i = a;
	}

	heap[i] = x;

	return ret;
}

void print_array() {
	cout << heap[0];
	for (int i = 1; i < sz; i++) {
		cout << " " << heap[i];
	}
	cout << "\n";
}

// test demo
int main() {
	// 初始化堆
	memset(heap, 0, sizeof(heap));

	auto test = { 99, 7, 3, 4, 2, 15, 21, 6 };
	for (auto& i : test) {
		push(i);
		print_array();
	}
	while (sz > 0) {
		cout << pop() << "\n";
	}
}
```

## 2.标准库的堆
```cpp
#include <queue>
#include <cstdio>
using namespace std;

struct cmp {
	bool operator() (const int &lhs, const int &rhs) {
		return lhs < rhs;
	}
};

int main() {
	// 声明
	priority_queue<int, vector<int>, cmp> pque;
	
	auto arr = { 3, 5, 7, 2, 9, 16 };

	// 插入元素
	for (auto& i : arr) {
		pque.push(i);
	}

	// 不断循环直到空为止
	while (!pque.empty()) {
		// 获取并删除最大值
		printf("%d\n", pque.top());
		pque.pop();
	}

	return 0;
}
```
当cmp中是`lhs < rhs`时, 输出`16 9 7 5 3 2 `;
当cmp中是`lhs > rhs`时, 输出`2 3 5 7 9 16 `;


## 3. 例题 POJ 2431
```cpp
/*
Expedition(POJ 2431)
你需要驾驶一辆卡车行驶L单位距离. 最开始时, 卡车上有P单位的汽油. 卡车每开1单位距离就要消耗1单位的汽油.
如果在途中车上的汽油耗尽, 卡车就无法前行, 因而无法到达终点.
在途中一共有N个加油站, 第i个加油站在距离起点Ai单位距离的地方, 最多可以给卡车加Bi单位的汽油.
假设卡车的燃料箱是无限大的, 无论加多少油都没有问题.
那么请问卡车是否能够到达终点? 如果可以, 最少需要加多少次油?
如果可以到达终点, 输出最少的加油次数, 否则输出-1.

1 <= N <= 10000
1 <= L <= 1000000, 1 <= P <= 1000000
1 <= Ai <= L, 1 <= Bi <= 100
*/

/*
输入测例:
N = 4, L = 25, P = 10
A = {10, 14, 20, 21}
B = {10, 5, 2, 4}
输出:
2
*/

/*
思考方式: 在卡车开往终点的途中, 只有在加油站可以加油.
但是, 如果认为"在到达加油站i时, 就获得了一次在之后的任何时刻都可以加Bi单位汽油的权利", 在解决问题上也是一样的.
变为: 当油量到0时, 选能加油量最大的那个Bi来加油
*/


#include <cstdio>
#include <queue>
#define MAX_N 10000
using namespace std;

// 输入
int L, P, N;
int A[MAX_N + 1], B[MAX_N + 1];

void solve() {
	// 为了写起来方便, 把终点也认为是加油站
	A[N] = L;
	B[N] = 0;
	N++;

	// 维护加油站的优先队列
	priority_queue<int> que;

	// ans: 加油次数, pos: 现在所在位置, tank: 油箱中的汽油量
	int ans = 0, pos = 0, tank = P;

	for (int i = 0; i < N; i++) {
		// 接下来要前进的距离
		int d = A[i] - pos;

		// 不断加油直到油量足够行驶到下一个加油站
		while (tank - d < 0) {
			if (que.empty()) {
				printf("-1\n");
				return;
			}
			tank += que.top();
			que.pop();
			ans++;
		}

		// 行驶到下一个加油站, 更新状态
		tank -= d;
		pos = A[i];
		que.push(B[i]);
	}

	printf("%d\n", ans);
}
```