# GDBの超ざっくりな使い方

- コンパイル時に-g3オプションを付けます。 例：`gcc -g3 xxx.c`   
- gdbコマンドで実行します。 例：`gdb ./a.out`   
- (gdb) みたいに表示されたら、runと入力します。 ブレークポイントなどの設定もできます。気になったら調べてみてください。

# Segmentation fault（コアダンプ）
- 訳：セグメンテーション違反

以下はあくまで Segmentation fault の一例です。これ以外の理由でSegmentation fault が出ている場合は原因の特定が大変なので、gdb などのデバッガを使いましょう。


### エラーの生じるコード例（1）
#include <stdio.h>

int main()
{
    int i;
    scanf("%d", i);    // 本当は「&i」としなければいけない
                       // （アドレスを渡す必要があるため）
}


ターミナルの表示　コンパイル時
```bash
8_segmentation_fault.c: In function ‘main’:
8_segmentation_fault.c:6:13: warning: format ‘%d’ expects argument of type ‘int *’, but argument 2 has type ‘int’ [-Wformat=]
    6 |     scanf("%d", i);    # 本当は「&i」としなければいけない（アドレスを渡す必要があるため）
      |            ~^   ~
      |             |   |
      |             |   int
      |             int *
```

- 警告は出ますが、コンパイルは通ります。

ターミナルの表示　実行時
```
$ ./a.out
1
Segmentation fault (コアダンプ)
```
- 実行すると、Segmentation fault が出ます。

解説

- このエラーはメモリに不正なアスセスをした場合に発生します。 ポインタを習うまでは、このエラーが出た場合は大抵 scanf の引数の & の付け忘れです。この例では、 scanf("%d", i); ではなく、scanf("%d", &i);  とすることにより、正しいプログラムになります。
- それ以外でこのエラーが出た場合は、エラー箇所を探すのには苦労することがあります。デバッガ（例えば gdb など）を使えば、すぐに見つけることができるので、 を修得することをお薦めします。



### エラーの生じるコード例（2）：メモリ違反
```c
#include <stdio.h>

int main()
{
    int array[10];              // 配列の要素は10個しかない
    int i;
    for(i = 0; i < 20; ++i){    // 20個目までアクセスしようとしてる
        array[i] = i;
    }
    return 0;
}
```

実行結果
```bash
$ ./a.out
*** stack smashing detected ***: terminated
中止 (コアダンプ)
```


- CEDの場合、Segmentation Fault とはならず、上記のようなエラーになるようです。 
GDBの場合
```bash
(gdb) run
Starting program: /home0/y2024/s2430064/TA_JRE3/error_sample/SegmentationFault/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
*** stack smashing detected ***: terminated

Program received signal SIGABRT, Aborted.
__pthread_kill_implementation (no_tid=0, signo=6, threadid=140737351423808) at ./nptl/pthread_kill.c:44
44      ./nptl/pthread_kill.c: そのようなファイルやディレクトリはありません.
```
- （この場合はどこでエラーが起きているのか分かりづらい） 
- 多くの人は、上記のような明らかに配列の範囲外にアクセスすることよりも、以下のような事例でSegmentation fault が起こることの方が多いと思います。
```c
#include <stdio.h>

// フィボナッチ数列を求めたい
int main()
{
    int array[10];    // 配列のサイズは10とする

    array[0] = 0; array[1] = 1;
    for (int i = 0; i < 10; i++)    // 配列のサイズは10だから、「i < 10」の条件で良い？？？
    {
        array[i+2] = array[i+1] + array[i];    // 実はここでi+2番目の要素までアクセスしているので、配列長を超えてプログラムが止まってしまう
    }
    return 0;
}
```
- for文内でi+2番目までアクセスしているので、正しくは「for (int i = 0; i < 10-2; i++)」としなければいけません。 
- その他、うっかりループ内で配列の-1番目の要素にアクセスしてしまう、というのもあるあるです。 

### エラーの生じるコード例（3）：無限再帰（または再帰が長すぎる）
```c
#include <stdio.h>

void loop()
{
    loop();    // これではloopが終了条件なく無限に呼び出されてしまいます
}

int main()
{
    loop();
    return 0;
}
```

実行結果
```
$ ./a.out
Segmentation fault (コアダンプ)
```
- これだけだとどこで問題が起きているのか分かりません。 

GDBでデバッグした際の表示
```bash
(gdb) run
Starting program: /home0/y2024/s2430064/TA_JRE3/error_sample/SegmentationFault/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
0x0000555555555136 in loop () at 2_recursion_infinity.c:5
5           loop();    ## これではloopが終了条件なく無限に呼び出されてしまいます
```
- 5行目で問題が起こっているよ、と教えてくれます。 

### エラーの生じるコード例（4）：Null ポインタの参照
```c
#include <stdio.h>

int main()
{
    int *ptr = NULL;
    int a = *ptr;    // NULLポインタを参照しているためSegmentation Faultが発生
    printf("%d\n", a);
    return 0;
}
```

実行結果
```bash
$ ./a.out
Segmentation fault (コアダンプ)
```

GDBでデバッグした際の表示
```
(gdb) run
Starting program: /home0/y2024/s2430064/TA_JRE3/error_sample/SegmentationFault/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
0x0000555555555161 in main () at 3_nullptr.c:6
6           int a = *ptr;    // NULLポインタを参照しているためSegmentation Faultが発生
```
- 6行目で問題が起こっているよ、と教えてくれます。 


