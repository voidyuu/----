## 特性

- [[缓冲区]]是临界区，需要用 mutex 夹起来。

## 1. 生产者消费者问题

### 例题

桌子上有一个盘子，每次只能向其中放入一个水果。爸爸专向盘子中放苹果，妈妈专向盘子中放橘子，儿子专等吃盘子中的橘子，女儿专等吃盘子中的苹果。只有盘子为空时，爸爸或妈妈才可向盘子中放一个水果; 仅当盘子中有自己需要的水果时，儿子或女儿可以从盘子中取出。

```cpp
semaphore plate = 1;
semaphore apple = 0;
semaphore orange = 0;

dad(){
 while(1){
  prepare an apple;
  P(plate);
  put the apple to the plate;
  V(apple);
 }
}

mom(){
 while(1){
  prepare an orange;
  P(plate);
  put the orange to the plate;
  V(orange);
 }
}

son(){
 while(1){
  P(apple);
  take the apple from the plate;
  V(plate);
  eat the apple;
 }
}

daughter(){
 while(1){
  P(orange);
  take the orange from the plate;
  V(plate);
  eat the orange;
 }
}

```

### 2009 年真题

三个进程 P 1、P 2、P 3 互斥使用一个包含 N（N>0）个单元的缓冲区。
P 1 每次用 produce () 生成一个正整数并用 put () 送入缓冲区某一空单元中；
P 2 每次用 getodd () 从该缓冲区中取出一个奇数并用 countodd () 统计奇数个数；
P 3 每次用 geteven () 从该缓冲区中取出一个偶数并用 counteven () 统计偶数个数。
请用信号量机制实现这三个进程的[[同步与互斥]]活动，并说明所定义的信号量的含义。要求用伪代码描述。

```cpp
semaphore even = 0;
semaphore odd = 0;
semaphore empty = N;
semaphore mutex = 1;

p1(){
 while(1){
  x=produce();
  P(empty);
  P(mutex);
  if(x%2==0) V(even);
  else V(odd);
  put();
  V(mutex);
 }
}

p2(){
 while(1){
  P(odd);
  P(mutex);
  getodd();
  V(mutex);
  V(empty);
  countodd();
 }
}

p3(){
 while(1){
  P(even);
  P(mutex);
  geteven();
  V(mutex);
  V(empty);
  counteven();
 }
}
```

### 2014 年真题

系统中有一个能容纳 1000 件产品的环形缓冲区，初始为空。生产者在缓冲区未满时放入产品，否则等待；消费者在缓冲区未空时取出产品，且一个消费者必须连续取出 10 件产品后，其他消费者才能开始取产品。

```cpp
semaphore mutex1 = 1;//生产者的互斥信号量
semaphore mutex2 = 1;//消费者的互斥信号量
semaphore empty = 1000;
semaphore full = 0;

producer(){
 while(1){
  produce sth;
  P(empty);
  P(mutex1);
  put product to buffer area;
  V(mutex1);
  V(full);
 }
}

consumer(){
 while(1){
  P(mutex2);
  for(int i=0;i<10;i++){
   P(full);
   take a product from buffer area;
     V(empty);
  }
  V(mutex2);
  consume;
 }
}

```

## 2. 理发师问题

### 2011 年真题

某银行提供 1 个服务窗口和 10 个供顾客等待的座位。顾客到达银行时，若有空座位，则到取号机上领取一个号，等待叫号。取号机每次仅允许一位顾客使用。当营业员空闲时，通过叫号选取一位顾客，并为其服务。

``` cpp
semaphore empty = 10;
semaphore full = 0;
semaphore service = 0;
semaphore mutex = 1;

process customer(){
 P(empty);//等待空位
 P(mutex);
 从取号机获得一个号码;
 V(mutex);
 V(full);//通知营业员有新顾客
 P(service);//等待叫号
 V(empty);//叫到号后移动到服务窗口，空出当前位置
 获得服务;
}
 process 营业员
{
 while (TRUE)
 {
  P(full);
  V(service);//叫号
  为顾客服务;
 }
}
```

```cpp

```

## 3. 读者写者问题

## 4. 哲学家进餐问题

### 2019 年真题

有n（n≥3）位哲学家围坐在一张圆桌边，每位哲学家交替地就餐和思考。在圆桌中心有 m（m≥1）个碗，每两位哲学家之间有一根筷子。每位哲学家必须取到一个碗和两侧的筷子后，才能就餐，进餐完毕，将碗和筷子放回原位，并继续思考。为使尽可能多的哲学家同时就餐，且防止出现死锁现象。

```cpp
semaphore chopsticks[n];//下标为i的筷子为第i位哲学家左手边的筷子
semaphore bowl = m;
semaphore limit = min(m,n-1);//保证筷子和碗都充足

for(int i=0;i<n;i++){
 chopsticks[i]=1;
}

thinker_i(int i){
 P(limit);
 P(chopsticks[i]);
 P(chopsticks[(i+1)%5]);
 吃饭;
 V(chopsticks[(i+1)%5]);
 V(chopsticks[i]);
 V(limit);
 思考;
}
```

## 5. 单纯的同步问题

### 2013 统考真题

某博物馆最多可容纳 500 人同时参观，有一个出入口，该出入口一次仅允许一个人通过。

```cpp
semaphore mutex = 1;
semaphore room = 500;
visitor()  {
 P(room);
 P(mutex);
 进门;  
 V(mutex);
 参观;  
 P(mutex);
 出门; 
 V(mutex);
 V(room);
}  
```

### 2015 年真题

有A、B两人通过信箱进行辩论，每个人都从自口的信箱中取得对方的问题，将答案和向对方提出的新问题组成一个邮件放人对方的信箱中。假设A的信箱最多放M个邮件，B的信箱最多放 N个邮件。初始时A的信箱中有x个邮件，B 的信箱中有 y 个邮件。辩论者每取出一个邮件，邮件数减一。

```cpp
semaphore fulla = x;
semaphore fullb = y;
semaphore emptya = M-x;
semaphore emptyb = N-y;
semaphore mutexa = 1;
semaphore mutexb = 1;
A(){
 while(1){
  P(fulla);
  P(mutexa);
  从A的信箱取出一个邮件;
  V(mutexa);
  V(emptya);
  回答问题并提出一个新问题;
  P(emptyb);
  P(mutexa);
  将新邮件放入B的邮箱;
  V(mutexa);
  V(fullb);
 }
}

B(){
 while(1){
  P(fullb);
  P(mutexb);
  从A的信箱取出一个邮件;
  V(mutexb);
  V(emptyb);
  回答问题并提出一个新问题;
  P(emptya);
  P(mutexb);
  将新邮件放入B的邮箱;
  V(mutexb);
  V(fulla);
 }
}
```

### 2017 年真题

某进程中有 3 个并发执行的线程 thread 1, thread 2 和 thread 3。

```cpp
//复数的结构类型定义
typedef struct{
 float a;
 float b;
}cnum;

cnum x,y,z;//全局变量

//计算两个复数之和
cnum add(cnum p,cnum q){
 cnum s;
 s.a = p.a+q.a;
 s.b = p.b+q.b;
 return s;
}
```

```cpp
semaphore mutex_y1 = 1;//线程1和3对y的互斥
semaphore mutex_y2 = 1;//线程2和3对y的互斥
semaphore mutex_z = 1;//线程2和3对z的互斥

thread1(){
 cnum w;
 P(mutex_y1);
 w = add(x,y);
 V(mutex_y1);
 ...
}

thread2(){
 cnum w;
 P(mutex_y2);
 P(mutex_z);
 w = add(y,z);
 P(mutex_z);
 P(mutex_y2);
}

thread3(){
 cnum w;
 w.a = 1;
 w.b = 1;
 P(mutex_z);
 z = add(z,w);
 V(mutex_z);
 
 P(mutex_y1);
 P(mutex_y2);
 y = add(y,w);
 V(mutex_y2);
 V(mutex_y1);

}
```

### 2020 年真题

现有 5 个操作 A、B、C、D 和 E，操作 C 必须在 A 和 B 完成后执行，操作 E 必须在 C 和 D 完成后执行。

```cpp
semaphore a = 0,b = 0,c = 0,d = 0;
A(){
 ...;
 V(a);
}

B(){
 ...;
 V(b); 
}

C(){
 P(a);
 P(b);
 ...;
 V(c);
}

D(){
 ...;
 V(d);
}

E(){
 P(c);
 P(d);
 ...;
}
```

### 2022 年真题

某进程的两个线程 T 1 和 T 2 并发执行 A、B、C、D、E 和 F 共 6 个操作，其中 T 1 执行 A、E 和 F，T 2 执行 B、C 和 D。两个操作的执行顺序所满足的约束：C 在 A 和 B 完成后执行，D 和 E 在 C 完成后执行，F 在 E 完成后执行。

```cpp
semaphore a = b = c = d = e = f = 0;

T1(){
 A;
 Signal(a);
 Wait(c);
 E;
 F;
}

T2(){
 B;
 Wait(a);
 C;
 Signal(c);
 D;
}
```
