# CEDのエラーメッセージの読み方

- CEDでC言語のコーディングする際に起きるエラーの説明資料です。
- プログラムが動かないときに、こちらの資料でエラーの原因を探してみてください。
- 誤り・不明瞭な記述・未記載のエラーがあれば 小泉 `koizumi.naoya@uec.ac.jp` まで連絡ください。


## 代表的なもの
### `i' undeclared (first use in this function)
- 訳：`i' は宣言されていません (この関数で初めて使用されます)

- エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    for (i = 0; i < 10; i++)
    {
        printf("%d\n", i);
    }
}
```

ターミナルの表示
```bash
1_undeclared.c: In function ‘main’:
1_undeclared.c:5:10: error: ‘i’ undeclared (first use in this function)
    5 |     for (i = 0; i < 10; i++)
      |          ^
1_undeclared.c:5:10: note: each undeclared identifier is reported only once for each function it appears in
```

解説
- これはプログラム中で使っている変数が宣言されていない場合に起こります。
- 関数の始めなどで、その変数を宣言してあげてください。
- また、長い名前の変数や綴りが難しい変数では、タイプミスをしている可能性も あるので、その点も充分にチェックしてください。


### expected declaration or statement at end of input

- 訳：入力の最後に宣言または文が必要です

エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    for (i = 0; i < 10; i++)
    {
        printf("%d\n", i);
    //　}　// }  が抜けている = 括弧で閉じてない。    
}
```

ターミナルの表示
```bash
2_parse_error_at_end_of_input.c: In function ‘main’:
2_parse_error_at_end_of_input.c:10:1: error: expected declaration or statement at end of input
   10 | }
      | ^
```

解説
- `}` がないのが 原因です。
- このエラーは大抵の場合、括弧の閉じ忘れか、括弧が2バイト文字 （俗に言う全角）の可能性があります。
- どこで括弧の対応がおかしくなっているかは、 エディタのインデント機能（emacs の M-x indent-regionなど） を使うと探しやすいです（括弧の対応がおかしい部分でインデントがずれる）。


### expected ‘;’ before ‘}’ token

- 訳：‘}’ トークンの前に ‘;’ が必要です

エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    for (i = 0; i < 10; i++)
    {
        printf("%d\n", i) // ;が抜けている
    }
}
```

ターミナルの表示
```bash
3_parse_error_before.c: In function ‘main’:
3_parse_error_before.c:8:26: error: expected ‘;’ before ‘}’ token
    8 |         printf("%d\n", i)
      |                          ^
      |                          ;
    9 |     }
      |     ~ 
```

解説

- expected ‘;’ before ‘}’ token 関連のエラーが出た場合は、エラーになった行の前の 行も確認しましょう。また、`}` 以外だけでなく、他の文字が入る こともあります。その文字の前に間違いがないか調べてみてください。
- この例では、最後の ; が抜けてい ます。また、その } に対応する { が抜けている場 合もあるので、確認しましょう。
- ネット上の資料では "parse error before"という表示が載っていますが、CEDでは`expected ‘;’ before …` と表示されるようです。

### stray ‘\343’ in program

- 訳：プログラム内の迷子の「\343」

エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    for (i = 0; i < 10; i++)　// ここに全角スペースがある
    {
        printf("%d\n", i);
    }
}
```

ターミナルの表示
```bash
4_parse_error_begfore_character.c:6:29: error: stray ‘\343’ in program
    6 |     for (i = 0; i < 10; i++)　// ここに全角スペースがある
      |                             ^~
```

解説
> 一見、どこにエラーがあるのか分かりにくいですが、このエラーが出た場合は 全角の文字が紛れ込んでいる可能性があります。この例では、6行目の最後に 全角スペースが入っています。この他、括弧などが全角になっている場合も、 このエラーが発生します。
- CEDだとカッコが全角になっている場合は`error: unknown type name ...`のようなエラーも出るようです。


### missing terminating " character

- 訳：終了文字「"」がありません
エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    for (i = 0; i < 10; i++)
    {
        printf("%d\n, i); // "が抜けてる
    }
    printf("\n");
}
```

ターミナルの表示
```bash
5_unterminated_string_or_character_constant.c: In function ‘main’:
5_unterminated_string_or_character_constant.c:8:16: warning: missing terminating " character
    8 |         printf("%d\n, i);
      |                ^
5_unterminated_string_or_character_constant.c:8:16: error: missing terminating " character
    8 |         printf("%d\n, i);
      |                ^~~~~~~~~~
5_unterminated_string_or_character_constant.c:9:5: error: expected expression before ‘}’ token
    9 |     }
      |     ^
5_unterminated_string_or_character_constant.c:8:16: error: expected ‘;’ before ‘}’ token
    8 |         printf("%d\n, i);
      |                ^
      |                ;
    9 |     }
      |     ~ 
```


- 前半2つの警告とエラーが、「”」の抜けを指摘している
- 後半二つは、それに伴って「;」や「}」が認識されずに起こっているもの。

解説
- これは " を閉じ忘れている場合に起きます。


### too few arguments to function ‘pow’

- 訳：関数「pow」の引数が少なすぎます
エラーの生じるコード例
```c
#include <stdio.h>
#include <math.h>

int main()
{
    printf("%f\\n", pow(10));
    return 0;
}
```

ターミナルの表示
```bash
tmp.c: In function ‘main’:
tmp.c:6:20: error: too few arguments to function ‘pow’
    6 |     printf("%f\\n", pow(10));
      |                    ^~~
In file included from /usr/include/features.h:486,
                 from /usr/include/x86_64-linux-gnu/bits/libc-header-start.h:33,
                 from /usr/include/stdio.h:27,
                 from tmp.c:1:
/usr/include/x86_64-linux-gnu/bits/mathcalls.h:140:1: note: declared here
  140 | __MATHCALL_VEC (pow,, (_Mdouble_ __x, _Mdouble_ __y));
      | ^~~~~~~~~~~~~~
```

解説
- pow() は引数が 2つなのに、1つしか与えていないので、エラーが出ています。 引数として何を与えるかを確認して、きちんと引数を与えましょう。


### too many arguments to function ‘pow’
- 訳：関数「pow」の引数が多すぎます
エラーの生じるコード例
```c
#include <stdio.h>
#include <math.h>

int main()
{
    printf("%f\n", pow(10, 2, 3));
    return 0;
}
```

ターミナルの表表
```bash
1_too_many_arguments_to_function.c: In function ‘main’:1_too_many_arguments_to_function.c:6:20: error: too many arguments to function ‘pow’
    6 |     printf("%f\\n", pow(10, 2, 3));
      |                    ^~~
In file included from /usr/include/features.h:486,
                 from /usr/include/x86_64-linux-gnu/bits/libc-header-start.h:33,
                 from /usr/include/stdio.h:27,
                 from 11_too_many_arguments_to_function.c:1:
/usr/include/x86_64-linux-gnu/bits/mathcalls.h:140:1: note: declared here
  140 | __MATHCALL_VEC (pow,, (_Mdouble_ __x, _Mdouble_ __y));
      | ^~~~~~~~~~~~~~
```

解説
- pow() は引数が 2つなのに、3つも与えているので、エラーが出ています。 引数として何を与えるかを確認して、きちんと引数を与えましょう。


### そのようなファイルやディレクトリはありません
エラーの生じるコード例
`````c
#include <stdio.h>
#include <stdlb.h>    // stdlib.h のタイプミス

int main()
{
    printf("%f\n", 2.0);
    return 0;
}
```

ターミナルの表示
```bash
12_no_such_file_or_directory.c:2:10: fatal error: stdlb.h: そのようなファイルやディレクトリはありません
    2 | #include <stdlb.h>    // stdlib.h のタイプミス
      |          ^~~~~~~~~
compilation terminated.
```
解説
- stdlb.h というファイルは存在しないのにincludeしようとしているため、エラーになります。 

### invalid preprocessing directive `命令'

- 訳：プリプロセスに、無効な`命令'が書かれています。

エラーの生じるコード例
```c
#include <stdio.h>
#includ <math.h>    // ここが間違っている

int main()
{
    printf("Hello\n");
}
```

ターミナルの表示
```bash
13_invalid_preprocessing_directive.c:2:2: error: invalid preprocessing directive #includ; did you mean #include?
    2 | #includ <math.h>
      |  ^~~~~~
      |  include
```

解説
- 綴りとかがあっているかを確認しましょう。 例では`include` の綴りを間違えています。


### conflicting types for ‘i’; have ‘double’
- 訳： 「i」の型が矛盾しています。「double」があります
エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    double i;    // iを再度定義している

    i = 2;
    printf("%f\n", i);

    return 0;
}
```

ターミナルの表示
```bash
14_redeclaration.c: In function ‘main’:
14_redeclaration.c:6:12: error: conflicting types for ‘i’; have ‘double’
    6 |     double i;
      |            ^
14_redeclaration.c:5:9: note: previous declaration of ‘i’ with type ‘int’
    5 |     int i;
      |         ^
```

解説
- 同じ名前の変数を 2回宣言しています。
- 片方の変数名を変更するか、消しましょう。



### redeclaration of ‘i’ with no linkage

- 訳：連結のない「i」の再宣言

エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    int i; // 同じ宣言

    i = 2;
    printf("%d\n", i);

    return 0;
}
```

ターミナルの表示
```bash
14-2_redeclaration.c: In function ‘main’:
14-2_redeclaration.c:6:9: error: redeclaration of ‘i’ with no linkage
    6 |     int i;
      |         ^
14-2_redeclaration.c:5:9: note: previous declaration of ‘i’ with type ‘int’
    5 |     int i;
      |         ^
```


上の例との違いは、同じint型でiを2回宣言していることです。 
片方の変数名を変更するか、消しましょう。 
元のサイトにはありませんでしたが、追加しました。 
その他、同じ変数名に関する注意点 スコープ内外で同じ変数名を定義すると、別物の変数として扱われます 
ただしスコープの内側の変数が優先されます。 
コンパイル時にオプションを付けないと、特に警告やエラーは出ません
```c
#include <stdio.h>

int main()
{
    int i = 1;
    {
        int i = 2;

        printf("内側のiは %d\\n", i);
    }
    printf("外側のiは %d\\n", i);

    return 0;
}
```

実行結果
```bash
$ ./a.out
内側のiは 2
外側のiは 1
```
 
通常、何の警告も出ずコンパイルできますが、-Wshadowオプションをつけてコンパイルすると警告を出してくれます。
```bash
$ gcc 14-3_redeclaration.c -Wshadow
14-3_redeclaration.c: In function ‘main’:
14-3_redeclaration.c:7:13: warning: declaration of ‘i’ shadows a previous local [-Wshadow]
    7 |         int i = 2;
      |             ^
14-3_redeclaration.c:5:9: note: shadowed declaration is here
    5 |     int i = 1;
      |         ^
```
  

### value required as left operand of assignment
- 訳：代入の左側の被演算子として左辺値が必要です

エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    if (i + 1 = 3) {} 
    return 0;
}
```
ターミナルの表示
```bash
tmp.c: In function ‘main’:
tmp.c:6:15: error: lvalue required as left operand of assignment
    6 |     if (i + 1 = 3) {}
      |               ^
```
解説
- 代入の右辺は様々な計算式を書けるが、左辺は代入先の変数でなければならない。
- 無効な左辺値に代入しようとすることは考えにくいので、 通常このエラーは、 誤って代入を書いたか、 左辺値の書き方を間違えたかのどちらかである。
- 前者の典型的な例は、比較演算子と代入演算子の書き間違いである。
- 上記の例の場合は、「==」としなければいけないところ「=」としてしまったため、「左辺i+1 に右辺 3 を代入しようとしている」と解釈されてエラーが出てしまっています。if 文の中では比較演算子「==」を使いましょう。

### assignment to expression with array type

- 訳：配列型の式への代入
- 
エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int a[10];
    a = 1;
    return 0;
}
```

ターミナルの表示
```bash
17_cast.c: In function ‘main’:
17_cast.c:6:7: error: assignment to expression with array type
    6 |     a = 1;
      |       ^
```
解説

- 代入の左辺と右辺の型が合っていない。
- 数値同士であれば暗黙の変換が行われるし、 ポインタ値と非ポインタ値の混在であれば「warning: assignment to ‘ポインタ型’ from ‘型’ makes pointer from integer without a cast」になる。 このエラーの典型的な例は、 配列変数として宣言したのに添え字をつけずに代入しようとした場合である。



今回のエラーは特に配列型に値を入れようとした場合に起こるようです。
エラー回避方法としては、通常通り配列の要素番目にアクセスする（a[i] = 1）もしくはポインタにする（*(a+i)=1）などが挙げられると思います。
解説文中にも書いてありますが、暗黙の型変換やポインタ・非ポインタのキャストにも注意してください。暗黙の型変換例えば、int型の変数にdoubleなどの小数の値を入れることはできますが（例：int a = 1.5;）、自動で小数点以下が切り捨てられます。コンパイル時に警告などは出ませんが、とても見つけにくいバグを生む可能性があるので気を付けましょう。
-Wconversionで警告を出すこともできるようです。
ポインタ・非ポインタのキャスト後述します

### variable or field ‘変数名’ declared void

- 訳：変数またはフィールド...がvoidと宣言
エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    void *p;    // エラーは起きない（任意の型のポインタを代入できるポインタになるらしいです）
    void a;     // 怒られる
    return 0;
}
```
ターミナルの表示
```bash
18_void.c: In function ‘main’:
18_void.c:6:10: error: variable or field ‘a’ declared void
    6 |     void a;     // 怒られる
      |          ^
```
解説
- 変数(あるいは構造体のメンバなど) var の型がvoidで宣言されている。
- voidは、関数の返値や引数がないことを示すものであり、 int や double のように、特定の変数の型にすることはできない。
<!-- 
 授業でvoidを関数名以外で使うことはほぼ無いと思うので、あまり出くわさないエラーだと思います。
 ただしポインタの型として「void*」が使われることはあるみたいです（5行目）。興味があれば調べてみてください。
配列も同様です。void array[10]のようなことはできません。（ただし同様にvoid *p[10]のようなvoidポインタの配列はできるようです。）
 -->

### ‘構造体名’ has no member named ‘メンバ名’

- 訳：構造体名 は メンバ名 という名前のメンバを持っていません

エラーの生じるコード例
```c
#include <stdio.h>

typedef struct {
    int x, y;    
} position;

int main()
{
    position p;
    p.z = 1; // zは存在しないメンバー
    return 0;
}
```
ターミナルの表示
```bash
19_member.c: In function ‘main’:
19_member.c:10:6: error: ‘position’ has no member named ‘z’
   10 |     p.z = 1;
      |      ^
```
解説

- 存在しない構造体メンバ mem に対する参照／代入が行われている。
- 型type は構造体型であるがmemという名前のメンバを持たない。 あるいはvar->mem という記述がある場合、 ポインタvar が指す型は構造体であるがmemという名前のメンバを持たない。
- 構造体「position」は見ての通り x, y のメンバしか持ちませんが、10行目で z という存在しないメンバにアクセスしようとしているためエラーが起こります。


### request for member ‘x’ in something not a structure or union
- 訳：構造体または共用体ではない何かのメンバ ‘mem’ の要求です

エラーの生じるコード例
```c
#include <stdio.h>

typedef struct {
    int x, y;
} position;

int main()
{
    int a;
    a.x = 1;    // aはint型なので、当然xというメンバは持たない

    position p, *pp;
    pp = &p;
    pp.x = 1;    // ppはポインタなので、xというメンバは持たない
    pp->x = 2;    // これはOK（ポインタがpを指し、そのpのメンバxを参照しているため）
    (*pp).x = 3;    // これもOK（上に同じ）

    return 0;
}
```

ターミナルの表示
```bash
20_struct_member.c: In function ‘main’:
20_struct_member.c:10:6: error: request for member ‘x’ in something not a structure or union
   10 |     a.x = 1;    // aはint型なので、当然xというメンバは持たない
      |      ^
20_struct_member.c:14:7: error: ‘pp’ is a pointer; did you mean to use ‘->’?
   14 |     pp.x = 1;    // ppはポインタなので、xというメンバは持たない
      |       ^
      |       ->
```
解説
- 構造体型でない変数に対し、メンバ参照を行った。
- `タイプミス` が `原因` なことが `多い` ようです。
    - 多分後半のエラーの方がよく見かけると思います。CEDのGCCでは「->ではありませんか？」と教えてくれます。環境によって（例えば自分のPC環境など）は違う表記になるかもしれません。

- int 型など構造型でない変数に対してメンバ参照を行いたい状況は考えにくいので、 ミスの内容として、以下のケースが考えられる。
    - ケース1: メンバ参照を行おうとする変数が誤っている
    - ケース2: メンバ参照を行おうとする変数の型が誤っている
    - ケース3: メンバ参照演算子が誤っている

### invalid operands to binary 二項演算子 (have ‘構造体1’ and ‘構造体2’)

- 訳：二項演算子 op への無効な被演算子です (‘type1’ と ‘type2’)

エラーの生じるコード例
```c
#include <stdio.h>

typedef struct {
    double x, y;
} vector;

int equal(vector v1, vector v2)
{
    if (v1.x == v2.x && v1.y == v2.y) { return 1; }
    else { return 0; }
}

int main()
{
    vector v1 = {1, 0}, v2 = {0, 1};
    if (v1 == v2)    // 自分で定義した構造体に対して二項演算子は使用できません。
    {
        // 何かする
    }

    if (equal(v1, v2))    // 自分で何かしらの関数を定義しましょう（こちらはOK）
    {
        // 何かする
    }
    return 0;
}
```

ターミナルの表示
```bash
21_operator.c: In function ‘main’:
21_operator.c:16:12: error: invalid operands to binary == (have ‘vector’ and ‘vector’)
   16 |     if (v1 == v2)    // 自分で定義した構造体に対して二項演算子は使用できません。
      |            ^~
```
解説
- 演算子opに対し、 その演算が適用できない式 (型type1とtype2) が与えられている。
- 自分で定義した構造体同士で二項演算子（+や-、==など）を使用することはできません。自分で比較用の関数などを作ると良いでしょう。
- Python などほかの言語だと、自分で定義した構造体などの+や-などを自分で定義することもできた気がしますが、Cにはそのような機能は無いようです。


# よくある警告

以下は-Wallオプションを付けてコンパイルしています。

### implicit declaration of function ‘prnitf’

- 訳：警告: 関数 'prnitf' の暗黙的な宣言
エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;
    for (i = 0; i < 10; i++)
    {
        prnitf("%d\n", i);
    }
}
```

ターミナルの表示
```bash
6_undefined_reference_to.c: In function ‘main’:
6_undefined_reference_to.c:8:9: warning: implicit declaration of function ‘prnitf’; did you mean ‘printf’? [-Wimplicit-function-declaration]
    8 |         prnitf("%d\n", i);
      |         ^~~~~~
      |         printf
/usr/bin/ld: /tmp/cc4YIa1z.o: in function `main':
```


- cedの場合、printff関数名タイプミスはエラーではなく警告になるようです（ただし実行ファイルは出力されない）

解説
- このエラーが出た場合は、大抵関数名の打ち間違いです。この例では、 printf としなければならないところを prnitf と書いてしまっています。



### control reaches end of non-void function
- 訳：制御が非 void 関数の終りに到達しました

警告の生じるコード例
```c
#include <stdio.h>

int func(int n)    // 返り値がint型の関数
{
    if (n == 0) { return 0; }
    // nが0ではなかった場合、返り値がint型の関数なのに何もreturnせず終わってしまう
    //return 1;    // 何かしら値を返すようにすれば警告は消える
}

int main()
{
    func(1);
    return 0;
}
```

ターミナルの表示
```bash
1_reaches_end_of_no-void.c: In function ‘func’:
1_reaches_end_of_no-void.c:8:1: warning: control reaches end of non-void function [-Wreturn-type]
    8 | }
      | ^
```

解説
- 返り値がある(void型ではない)関数の中で、 returnせずに関数末に到達している。
- 関数funcの返り値はint型なので、 必ず整数値を返さなければならない。 上記のコードでは、nの値が0のとき、 returnがないため返り値を返さずに関数を終了してしまう。
- 関数の定義内で、宣言した型の返り値をreturnする。

    - returnしないと、予想外の値が返されバグに繋がります。手元でfunc(1)の値をprintしたところ、「540655971」という訳の分からない数値が表示されました。関数定義の通りにreturnしましょう。
    - if文がネストされている時などに生じやすいミスなので、フォーマッタでコードを整理しましょう。
    - main関数もint型なので、return 0 しておきましょう。

### return type defaults to ‘int’

- 訳：戻り値の型をデフォルトの ‘int’ にします

警告の生じるコード例
```c
#include <stdio.h>

func(int n)    // 返り値が未定義
{
    if (n == 0) { return 0; }
    else { return 1; }
}

int main()
{
    printf("%d\n", func(1));
    return 0;
}
```

ターミナルの表示
```bash
2_return_type_default.c:3:1: warning: return type defaults to ‘int’ [-Wimplicit-int]
    3 | func(int n)    // 返り値がint型の関数
      | ^~~~
```
解説
- 関数を定義する際に、返り値の型を指定していない。
- 関数を定義する際に返り値の型を省略すると、 デフォルトで int 型として扱われ、この警告が出力される。 上の例のように、実際に int 型を変えす関数であれば実害はないが、 もし異なる型を返すコードを書いている場合は正常動作しない可能性がある。
- 関数の定義の際に、関数名の前に返り値の型を記述する。 とくに、返すものがない場合は、必ず関数をvoid型で宣言する。 未指定だとその関数は「int 型を返す」ことになってしまうからである。
`ちゃんと返り値を定義しましょう。返り値が無ければvoidにしましょう。`


### suggest parentheses around assignment used as truth value

- 訳：真理値として使用される代入を括弧で囲むことを提案します

警告の生じるコード例
```c
#include <stdio.h>

int main()
{
    int a = 1;
    if (a = 1)
    {
        // 何かの処理
    }
    return 0;
}
```
ターミナルの表示
```bash
3_used_as_truth_value.c: In function ‘main’:
3_used_as_truth_value.c:6:9: warning: suggest parentheses around assignment used as truth value [-Wparentheses]
    6 |     if (a = 1)
      |         ^
```
解説
- if, whileなどの条件式に、代入文が書かれている。
- 上級者が意図的にやっているのでない限り、 比較演算子「==」の代わりに誤って代入演算子「=」を書いている
- おそらく「aの値が1のとき」と書くつもりだったが、 「aに1を代入」している。 文法的に誤りではないので、警告を無視すればプログラムを実行できるが、 この行でaの値が1になり、条件文は常に成立する (代入した値1が条件の真偽値: 非0なので真)。 したがって、おそらく誤動作する。
- （if (a = b) という例について）この例では、「bをaに代入」したうえで、 「bの値が真(非0)なら条件が成立」する。 一般的には「a==b」つまり 「aとbの値が一致したら条件が成立」 の書き間違いの可能性が高いので警告が出る。 意図的にやりたい場合は「if ((a=b)){」のように 代入式を括弧で括れば意図的なものだとコンパイラが認識し、警告は出なくなる。 しかし、コードがそれほど短くなるわけではなく、 読みにくく／勘違いしやすくなるデメリットの方が大きいので、 以下のように書く方がよい。


<!-- 
if文の条件式は、a = 1、すなわちtrueになるので、if文の中が実行されます。しかし、コンパイラが「おそらくa==1の書き間違いだろう」と解釈して、警告を出してくれます。実際、書き間違いの場合が多いと思います。
意図的にやっているのであれば、if ((a = b))のようにカッコでもう一回覆ってしまえば警告は出ないそうです。でも、読みにくいので書き換えた方が良いです。おすすめしません。
ちなみに、「while ((c = fgetc(fp)) != EOF){...};」のような書き方を見かけたことがある人もいるかもしれませんが、このようなループ文では慣用句的に条件式内の代入を使うことが多いです。
 -->

### ignoring return value of 'fgets' declare with attribute 'warn_unused_result'



### unused variable ‘var’
- 訳：使用されない変数 ‘var’ です 

警告の生じるコード例
```c
#include <stdio.h>

int main()
{
    int i, j, a[10], b[10];    // jはどこにも使っていない

    for (i = 0; i < 10; i++)
    {
        a[i] = i;
        printf("%d ", a[i]);    // ここでaは使っている
        b[i] = 10 * i;    // bは書き込んではいるが使って（読み込んで）いない
    }
    
    return 0;
}
```
ターミナルの表示
```bash
4_unused_variable.c: In function ‘main’:
4_unused_variable.c:5:22: warning: variable ‘b’ set but not used [-Wunused-but-set-variable]
    5 |     int i, j, a[10], b[10];    // jはどこにも使っていない
      |                      ^
4_unused_variable.c:5:12: warning: unused variable ‘j’ [-Wunused-variable]
    5 |     int i, j, a[10], b[10];    // jはどこにも使っていない
      |            ^
```
解説
- 変数varの宣言があるが、一度も使っていない。
- 使うつもりで宣言したが、結局使っていない変数ならば、宣言を削除する。
- 使っていない変数があること自体はあまり実害がないので、 後で使うつもりで先に変数だけ宣言しておいた、 など心当たりがあるなら、 この警告は無視してもかまわない。

<!-- 

この例の場合、jは使っていないので当然警告が出る。
配列bは一見使っているので問題なさそうだが、書き込むだけ書き込んで後は放置しているだけの変数なので、警告が出る。
 -->




### warning: ‘i’ is used uninitialized

- 訳：警告: ‘i’ が初期化されていない状態で使用されています
エラーの生じるコード例
```c
#include <stdio.h>

int main()
{
    int i;    // iの値が初期化されていない
    printf("%d", i);
}
```

ターミナルの表示
```bash
$ gcc TA1_uninitialized_variable.c -Wall
TA1_uninitialized_variable.c: In function ‘main’:
TA1_uninitialized_variable.c:6:5: warning: ‘i’ is used uninitialized [-Wuninitialized]
    6 |     printf("%d", i);
      |     ^~~~~~~~~~~~~~~
```

- 注意：gccのみのコンパイルだと、警告は出ない。 -Wall オプションを付ける（or チェッカーでコンパイルする）とこの警告が出る。

### warning: implicit declaration of function ‘malloc’

- 訳：警告: 関数 'malloc' の暗黙的な宣言
エラーの生じるコード例
```c
#include <stdio.h>
// #include <stdlib.h>    // mallocを使うときは stdlib.h のインクルードが必要です

int main()
{
    int len = 100;
    int *p;
    p = (int*)malloc(sizeof(int) * len);    // malloc を使う

    for (int i = 0; i < len; i++)
    {
        p[i] = i;
    }

    free(p);    // 忘れないこと
    return 0;
}
```

ターミナルの表示
```bash
TA3_stdlib_malloc.c: In function ‘main’:
TA3_stdlib_malloc.c:8:15: warning: implicit declaration of function ‘malloc’ [-Wimplicit-function-declaration]
    8 |     p = (int*)malloc(sizeof(int) * len);    // malloc を使う
      |               ^~~~~~
TA3_stdlib_malloc.c:2:1: note: include ‘<stdlib.h>’ or provide a declaration of ‘malloc’
    1 | #include <stdio.h>
  +++ |+#include <stdlib.h>
    2 | //#include <stdlib.h>    // mallocを使うときは stdlib.h のインクルードが必要です
TA3_stdlib_malloc.c:8:15: warning: incompatible implicit declaration of built-in function ‘malloc’ [-Wbuiltin-declaration-mismatch]
    8 |     p = (int*)malloc(sizeof(int) * len);    // malloc を使う
      |               ^~~~~~
TA3_stdlib_malloc.c:8:15: note: include ‘<stdlib.h>’ or provide a declaration of ‘malloc’
TA3_stdlib_malloc.c:15:5: warning: implicit declaration of function ‘free’ [-Wimplicit-function-declaration]
   15 |     free(p);    // 忘れないこと
      |     ^~~~
TA3_stdlib_malloc.c:15:5: note: include ‘<stdlib.h>’ or provide a declaration of ‘free’
TA3_stdlib_malloc.c:15:5: warning: incompatible implicit declaration of built-in function ‘free’ [-Wbuiltin-declaration-mismatch]
TA3_stdlib_malloc.c:15:5: note: include ‘<stdlib.h>’ or provide a declaration of ‘free’
```

解説
- mallocを使うときは  #include <stdlib.h> をしましょう

### printf が正しく出力しない
- printf が変な文字や数値を出力するなどした場合は、フォーマット指定子を確認してください。

```c
#include <stdio.h>

int main()
{
    int i = 1;
    printf("%%dだと：「%d」\n", i);
    printf("%%cだと：「%c」\n", i);
}
```


実行結果は以下
```bash
%dだと：「1」
    %cだと：「」
```


文字列の読み込みに必要な配列長
```c
#include <stdio.h>

int main()
{
    char buf[5];     // 5文字まで読み込める？？？？？
    fgets(buf, sizeof(buf), stdin);
    printf("%s\n", buf);
    return 0;
}
```


「Hello」と入力すると…
```bash
$ ./a.out
    Hello
    Hell
```


「Hell」までしか出力されない…
基本的に、配列長は入力する文字列長+2要素必要だと考えた方が良いです。文字列の長さ + 改行コード「\n」 + 終端コード「\0」
先ほどの例だと、配列長以上の文字列を入力した場合に最後の要素に終端文字を入れるfgetsの仕様が働いたみたいです。（Copilot情報）メモ参考にした元のサイトの「undefined reference to `sin'」に関しては、cedだと-lmオプションをつけなくてもエラーは出ず問題なく動いたため省略。（本当はつけた方が良いかもしれないが。）
参考にした元のサイトの「/usr/include/stdio.h:261: parse error before `__va_list'」に関しては、元のサイトとエラー表示が異なるため省略


# Segmentation fault（コアダンプ）
- 訳：セグメンテーション違反

- このエラーが起きる原因はあまりにも多すぎるかつ複雑すぎるので、別項目にしました。
- 以下はあくまで Segmentation fault の一例です。これ以外の理由でSegmentation fault が出ている場合は原因の特定が大変なので、gdb などのデバッガを使いましょう。

GDBの超ざっくりな使い方

コンパイル時に-g3オプションを付けます。 例：gcc -g3 xxx.c   
gdbコマンドで実行します。 例：gdb ./a.out   
(gdb) みたいに表示されたら、runと入力します。 ブレークポイントなどの設定もできます。気になったら調べてみてください。 

エラーの生じるコード例（1）
```c
#include <stdio.h>

int main()
{
    int i;
    scanf("%d", i);    // 本当は「&i」としなければいけない
                       // （アドレスを渡す必要があるため）
}
```

ターミナルの表示　コンパイル時
```c
8_segmentation_fault.c: In function ‘main’:
8_segmentation_fault.c:6:13: warning: format ‘%d’ expects argument of type ‘int *’, but argument 2 has type ‘int’ [-Wformat=]
    6 |     scanf("%d", i);    # 本当は「&i」としなければいけない（アドレスを渡す必要があるため）
      |            ~^   ~
      |             |   |
      |             |   int
      |             int *
```


警告は出ますが、コンパイルは通ります。
ターミナルの表示　実行時
```bash
$ ./a.out
1
Segmentation fault (コアダンプ)
```


実行すると、Segmentation fault が出ます。
解説

このエラーはメモリに不正なアスセスをした場合に発生します。 ポインタを習うまでは、このエラーが出た場合は大抵 scanf の引数の & の付け忘れです。この例では、 scanf("%d", i); ではなく、scanf("%d", &i);  とすることにより、正しいプログラムになります。

それ以外でこのエラーが出た場合は、エラー箇所を探すのには苦労することがあ ります。デバッガ（例えば gdb など）を使えば、すぐに見つけることができる ので、 デバッガの使い方 を修得することをお薦めします。



エラーの生じるコード例（2）：メモリ違反
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


CEDの場合、Segmentation Fault とはならず、上記のようなエラーになるようです。 
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



（この場合はどこでエラーが起きているのか分かりづらい） 
多くの人は、上記のような明らかに配列の範囲外にアクセスすることよりも、以下のような事例でSegmentation fault が起こることの方が多いと思います。
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


for文内でi+2番目までアクセスしているので、正しくは「for (int i = 0; i < 10-2; i++)」としなければいけません。 
その他、うっかりループ内で配列の-1番目の要素にアクセスしてしまう、というのもあるあるです。 

エラーの生じるコード例（3）：無限再帰（または再帰が長すぎる）
プログラムの例
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
```bash
t$ ./a.out
Segmentation fault (コアダンプ)
```


これだけだとどこで問題が起きているのか分かりません。 
GDBでデバッグした際の表示
```bash
(gdb) run
Starting program: /home0/y2024/s2430064/TA_JRE3/error_sample/SegmentationFault/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
0x0000555555555136 in loop () at 2_recursion_infinity.c:5
5           loop();    // これではloopが終了条件なく無限に呼び出されてしまいます
```


5行目で問題が起こっているよ、と教えてくれます。 

エラーの生じるコード例（4）：Null ポインタの参照
プログラムの例
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
```bash
(gdb) run
Starting program: /home0/y2024/s2430064/TA_JRE3/error_sample/SegmentationFault/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
0x0000555555555161 in main () at 3_nullptr.c:6
6           int a = *ptr;    // NULLポインタを参照しているためSegmentation Faultが発生
```bash


6行目で問題が起こっているよ、と教えてくれます。 






# なぜこのページを作ったのか？
- 【背景】小泉が、この授業の参考図書として「コードが動かないので帰れません！ 新人プログラマーのためのエラーが怖くなくなる本」という本を読んでいたところ、「エラーを理解できるようになれば怖く無くなる」という話が載っていました。
- 【対応】そこで、CEDでよく出るエラーとその原因を記録することで、学生がすぐにエラーを見つけやすくし、自分自身でデバグできるようにしようと考えました。
- 【参考資料】図書館にある「コードが動かないので帰れません！ 新人プログラマーのためのエラーが怖くなくなる本」です。　https://uec.primo.exlibrisgroup.com/permalink/81UEC_INST/ntu2kh/alma991002502075307421

