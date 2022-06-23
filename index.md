---
layout: default
title: 軽率にGPUを使っていこう、OpenCL入門
---
# 軽率にGPUを使っていこう、OpenCL入門

はい、Cra2yPierr0tです。GPUを作る前に市販のGPUを触ってみるか、どうやらOpenCLってやつが標準化されてて良さそうだぞ。という動機でOpenCLを書き始めたら思いのほか辛かったし納得のいく入門記事が無かったので備忘録がてらOpenCL入門をここに置いときます。

目次
* mokuji
{:toc}

## OpenCLとは

並列プログラミング用のAPI、厳密な定義はwiki読んでね。

[https://ja.wikipedia.org/wiki/OpenCL](https://ja.wikipedia.org/wiki/OpenCL)

あとインストールとかは各自で頑張ってください。

## OpenCLの思想
OpenCLの思想としてループの関数(カーネル)への展開が挙げられる。例として以下のように1024x1024の画像処理を1個のカーネルで行うのではなく、1024x1024=1,048,576個のカーネルで行うことで処理を高速にする。

古典的な実装
```c
void mul(const int n, const float *a, const float *b, float *c) {
    for(int i = 0; i < n; i++){
        c[i] = a[i] + b[i];
    }
}
```
OpenCLでデータ並列性を活用する
```c
// これが同時に大量に実行される
__keren void add(__global const float *a, __global const float *b, 
                         __global float *c){
    int id = get_global_id(0);
    c[id] = a[id] + b[id];
}
```

## OpenCLの計算機システム
OpenCLにとって計算機システムは、単一の制御用の**ホスト**と一つ以上の**デバイス**によって構成されている。そしてデバイスは一つ以上の**Compute Unit(CU)** からなる。またCUも一つ以上の**Processing Element(PE)** から構成される。
メモリシステムも**ホストメモリ**と**デバイスメモリ**に大別される。ホストとデバイスを合わせて**プラットフォーム**とも呼ぶ。

![ホストとデバイス、プラットフォーム](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/platform.png?raw=true)

ホストとデバイスが何を指すかは覚えておいた方が良い。
## ホストとカーネル
OpenCLのプログラムは、GPU上で動く**カーネルプログラム**とCPU上で動く**ホストプログラム**に分かれている。

カーネルプログラムはGPUで動くプログラムであり、カーネルプログラムが大量に並列実行されることで高速な計算が可能となる。またホストプログラムはCPU上で動くプログラムであり、その主な役割はカーネルプログラムをGPUへ展開する事と、データを転送する事である。

![ホストがデバイスにカーネルを投げる様子](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/host_kernel.png?raw=true)

以上の通りOpenCLではホストとカーネルの二種類のプログラムを書く必要があり、特にホストプログラムはやることも多く複雑になりやすい。ホストとカーネルとは何かをきちんと覚えた上で読み進めてほしい。

## OpenCLの次元
解きたい問題には全て、直線状やキューブ状や平面状のようにある程度の**次元性**が存在している。
OpenCLでは最大3次元までを指定してカーネルを展開する。また各次元方向のサイズも指定する必要がある。この各次元のサイズをグローバルサイズと呼ぶ。そして各点は**ワークアイテム**によって処理される。

このワークアイテムを纏めたものを**ワークグループ**と呼ぶ。1つのワークグループに1つのCUが割り当てられ、ワークグループ内ではローカルメモリの共有と同期が可能である。

ワークグループ内にあるワークアイテムの数を指定することが可能であり、これを**ローカル(ワークグループ)サイズ**と呼ぶ。OpenCLランタイムにこのローカルサイズを自動に決定させることも可能だが、大抵の場合最適ではない。

アルゴリズムに最も適した次元を選ぶことが非常に重要である。

![2次元のワークアイテム](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/2dim_NDRange.png?raw=true)

![1次元のワークアイテム](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/1dim_NDRange.png?raw=true)

OpenCLの基礎知識は以上にとどめ、次は具体例を出しながらホストプログラムの書き方を解説する。

## OpenCLホストプログラミング入門
### 概要
ホストプログラムは基本的に5つのステップに分けられる。

1. デバイス, コンテキスト, コマンドキューを定義する
2. カーネルをビルドする
3. デバイスメモリを確保する
4. カーネルの引数を設定する
5. キューにメモリとカーネルを転送する

### 1. デバイス, コンテキスト, コマンドキューを定義する
コンテキスト、キューという概念が出てきた、コンテキストは利用するデバイスをまとめた実況環境(らしい)であるが、OpenCLを使うために必要なオブジェクトというフワッとした解釈で全然問題ない。

コマンドキューに関しては割と重要な概念である。これはコマンド(カーネル及びメモリ等の転送命令)が入るキューであり、デバイスでプログラムを実行するにはこのキューを必ず経由する必要がある。
一つのコマンドキューは一つのデバイスに向かっており、複数のコマンドキューが同じデバイスに向かう事も可能である。この場合、コマンドキュー同士で同期を取る必要は無い。

よってより正確なイメージは以下のようになる。

![より正確なホストプログラムのイメージ](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/host_kernel_queue.png?raw=true)

コマンドキューだが、キューに入れた順にデバイスに送るインオーダキューと、入れた順にならないアウト・オブ・オーダーキューが存在する。

主に使う関数はこちら

* 利用可能なプラットフォームを確保：
    * `clGetPlatformIDs()`
* デバイスを確保
    * `clGetDeviceIDs()`
* コンテキストを作成
    * `clCreateContext()`
* コマンドキューを作成
    * `clCreateCommandQueue()`

これらの関数を使って必要なものを定義するコードが以下になる。また先頭に`cl`が付いている型や関数はOpenCL APIで定義されているものである。
```c
// 環境構築関係
// プラットフォームIDを確保
cl_platform_id platform_id;
cl_uint platform_num;
clGetPlatformIDs(1, &platform_id, &platform_num);

// デバイスIDを確保
cl_device_id device_id;
clGetDeviceIDs(platform_id, CL_DEVICE_TYPE_DEFAULT, 1, &device_id, NULL);

// コンテキストを作成
cl_context context;
context = clCreateContext(0, 1, &device_id, NULL, NULL, NULL);

// コマンドキューを作成
cl_command_queue commands;
commands = clCreateCommandQueue(context, device_id, 0, NULL);
```

### 2. カーネルをビルドする
カーネルプログラムもプログラムであるので、コンパイルして実行形式にしてやる必要がある。OpenCLではカーネルプログラムから**プログラムオブジェクト**を作成し、そのプログラムオブジェクトをビルドした後**カーネルオブジェクト**を作成する。

コンパイル方式には、事前にカーネルプログラムをコンパイルするオフラインコンパイルと、ホストプログラムの実行時にコンパイルするオンラインコンパイルの二種類の方式がある。

主に使う関数がこちら

* プログラムオブジェクトを作成
    * `clCreateProgramWithSource()`
* プログラムオブジェクトをビルド
    * `clBuildProgram()`
* カーネルオブジェクトを作成
    * `clCreateKernel()`

これらの関数を使ってカーネルオブジェクトを作成するコードが以下になる。`clCreateProgramWithSource()`の引数であるkernelsourceはカーネルプログラムを保持している文字列である。
```c
// プログラムオブジェクトを作成
cl_program program;
program = clCreateProgramWithSource(context, 1, (const char **)&kernelsource, NULL, NULL);

// プログラムオブジェクトをビルド
clBuildProgram(program, 0, NULL, NULL, NULL, NULL);

// カーネルオブジェクトを作成
cl_kernel ko_vadd;
ko_vadd = clCreateKernel(program, "vadd", NULL);
```

### 3. デバイスメモリを確保する
データを格納するためにCPUでメモリを確保するのと同様に、GPUのメモリを確保してやる必要がある。

例として、ベクトル加算を行う場合、3つのメモリオブジェクトが必要になる。a, bを入力ベクタとして、cを出力ベクタとする。
先にホスト側でメモリを確保し、適当に初期化する。
```c
// ホストメモリを確保
float h_a[1024], h_b[1024], h_c[1024];
for(int i = 0; i < 1024; i++) {
    h_a[i] = rand() / (float)RAND_MAX;
    h_b[i] = rand() / (float)RAND_MAX;
}
```

次にデバイスメモリを確保し、メモリオブジェクトを得る。関数は`clCreateBuffer()`を使う。この関数を使ってメモリを確保するコードが以下になる。
```c
// デバイスメモリを確保
cl_mem d_a;
cl_mem d_b;
cl_mem d_c;

d_a = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                                     sizeof(float) * 1024, h_a, NULL);
d_b = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                                     sizeof(float) * 1024, h_b, NULL);
d_c = clCreateBuffer(context, CL_MEM_WRITE_ONLY,
                                     sizeof(float) * 1024, NULL, NULL);
```
`clCreateBuffer()`の引数を見てみると、入力ベクタには`CL_MEM_READ_ONLY`や`CL_MEM_COPY_HOST_PTR`、ホストメモリへのポインタが指定されていたり、出力ベクタには`CL_MEM_WRITE_ONLY`が指定されていたりと、なんとなく何を意図して引数を設定しているか分かるかもしれない。分からなければ後でリファレンスを読めばよい。

### 4. カーネルの引数を設定する
カーネルには関数のように引数を設定する事が可能である。引数を設定するには`clSetKernelArg()`を用いる。

例としてベクトル加算を行う場合、カーネルに渡す必要がある引数は配列a, b, cのアドレスである。第一引数を`float *a`、第二引数を`float *b`、第三引数を`float *c`として、カーネルに引数を設定するコードが以下になる。
```c
// 引数を設定
clSetKernelArg(ko_vadd, 0, sizeof(cl_mem), &d_a);
clSetKernelArg(ko_vadd, 1, sizeof(cl_mem), &d_b);
clSetKernelArg(ko_vadd, 2, sizeof(cl_mem), &d_c);
```

### 5. キューにメモリとカーネルを転送する
コマンドキューにメモリの書き込み指示、カーネルの展開・実行の指示、メモリの読み出し指示を行う。

主に使う関数がこちら

* デバイスメモリにデータを書き込む、今回はデバイスメモリ確保時に行っているので使わない(疑問点あり)
    * `clEnqueueWriteBuffer()`
*  カーネルをデバイスに展開し、実行する。後で詳しく扱う
    * `clEnqueueNDRangeKernel()`
* カーネルが完了まで待機する
    * `clFinish()`
* デバイスメモリからデータを読み出し
    * `clEnqueueReadBuffer()`

これらの関数を用いたコードが以下になる。
```c
// カーネルをエンキュー
size_t N = 1024;
clEnqueueNDRangeKernel(commands, ko_vadd, 1, NULL, &N, NULL, 0, NULL, NULL);

// 完了まで待つ
clFinish(commands);

// 結果を読み出し
clEnqueueReadBuffer(commands, d_c, CL_TRUE, sizeof(float) * 1024, h_c, 0, NULL, NULL);
```

この`cnEnqueueNDRangeKernel()`がカーネルをどんな形で展開するかを決める非常に重要な関数である。これに関してはカーネルプログラムの書き方を解説する際に詳しく扱う。

### ベクトル加算のホストプログラム
以上の1~5を組み合わせて完成させたホストプログラムが以下になる。一番下に計算結果の出力とクリーンアップを追加している。ライブラリと`kernelsource`、一番下以外は全て前述したコードと同じである。
OpenCLがインストールされている環境下で`$ gcc -lOpenCL vadd.c`でコンパイルが出来る。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <time.h>
#ifdef __APPLE__
#include <OpenCL/opencl.h>
#include <unistd.h>
#else
#include <CL/cl.h>
#endif

char *kernelsource = "__kernel void vadd (              \n" \
"   __global float *a,                                  \n" \
"   __global float *b,                                  \n" \
"   __global float *c) {                                \n" \
"   int i = get_global_id(0);                           \n" \
"   c[i] = a[i] + b[i];                                 \n" \
"}                                                      \n" \
"\n";

int main(int argc, char** argv) {
    // 環境構築関係
    // プラットフォームIDを確保
    cl_platform_id platform_id;
    cl_uint platform_num;
    clGetPlatformIDs(1, &platform_id, &platform_num);

    // デバイスIDを確保
    cl_device_id device_id;
    clGetDeviceIDs(platform_id, CL_DEVICE_TYPE_DEFAULT, 1, &device_id, NULL);

    // コンテキストを作成
    cl_context context;
    context = clCreateContext(0, 1, &device_id, NULL, NULL, NULL);

    // コマンドキューを作成
    cl_command_queue commands;
    commands = clCreateCommandQueue(context, device_id, 0, NULL);

    // プログラム関係
    // プログラムオブジェクトを作成
    cl_program program;
    program = clCreateProgramWithSource(context, 1, (const char **)&kernelsource, NULL, NULL);

    // プログラムオブジェクトをビルド
    clBuildProgram(program, 0, NULL, NULL, NULL, NULL);

    // カーネルオブジェクトを作成
    cl_kernel ko_vadd;
    ko_vadd = clCreateKernel(program, "vadd", NULL);

    // メモリ関係
    // ホストメモリを確保、適当に初期化
    float h_a[1024], h_b[1024], h_c[1024];
    for(int i = 0; i < 1024; i++) {
        h_a[i] = (float)(rand() % 10);
        h_b[i] = (float)(rand() % 10);
    }

    // デバイスメモリを確保
    cl_mem d_a;
    cl_mem d_b;
    cl_mem d_c;

    d_a = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                                         sizeof(float) * 1024, h_a, NULL);
    d_b = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                                         sizeof(float) * 1024, h_b, NULL);
    d_c = clCreateBuffer(context, CL_MEM_WRITE_ONLY,
                                         sizeof(float) * 1024, NULL, NULL);

    // 引数とか
    // 引数を設定
    clSetKernelArg(ko_vadd, 0, sizeof(cl_mem), &d_a);
    clSetKernelArg(ko_vadd, 1, sizeof(cl_mem), &d_b);
    clSetKernelArg(ko_vadd, 2, sizeof(cl_mem), &d_c);

    // 実行関係
    // カーネルをエンキュー
    size_t N = 1024;
    clEnqueueNDRangeKernel(commands, ko_vadd, 1, NULL, &N, NULL, 0, NULL, NULL);

    // 完了まで待つ
    clFinish(commands);

    // 結果を読み出し
    clEnqueueReadBuffer(commands, d_c, CL_TRUE, 0, sizeof(float) * 1024, h_c, 0, NULL, NULL);

    for(int i = 0; i < 1024; i++) {
        printf("h_c[%d] = %f\n", i, h_c[i]);
    }

    // クリーンアップ
    clReleaseMemObject(d_a);
    clReleaseMemObject(d_b);
    clReleaseMemObject(d_c);
    clReleaseProgram(program);
    clReleaseKernel(ko_vadd);
    clReleaseCommandQueue(commands);
    clReleaseContext(context);

    return 0;
}
```
なお、このプログラムはエラー処理を一切行っていないため、テンプレートとして使うことは全く推奨しない。

## OpenCLカーネルプログラミング入門
前章のホストプログラムにおけるカーネルプログラムは次のようなものだった。
```c
__kernel void vadd (
    __global float *a,
    __global float *b,
    __global float *c) {
    int i = get_global_id(0);
    c[i] = a[i] + b[i];
}
```

`get_global_id(0)`だとか`__global`だとか、いくつか見慣れない修飾子や関数があるだろう。これはOpenCLのカーネルプログラミングで用いる独自に拡張されたC言語の追加要素である。なお、この拡張されたC言語は文献によってはOpenCL Languageと呼ばれている。

ちなみに、配列のインデックスを取得している`get_global_id()`関数だが、これは直線に並んでいるワークアイテムの位置を取得する関数である。前項のホストプログラムは、ワークアイテムという仮想的なプロセッサを直線状に1024個生成し、それら全てにこのカーネルプログラムを実行させている。実行されているカーネルは同一だが、`get_global_id()`が返す値がワークアイテムの位置によって異なるため、1024要素のベクトル加算を実現することが出来る。This is SIMD.

以降では、OpenCLによるCの拡張要素とカーネルプログラムの書き方について解説する。

### OpenCLカーネルにおけるC言語

拡張されたC言語の概要は以下の通り。

* ISO C99からの派生である
    * いくつか制約が存在する：再帰、関数ポインタ、C99の標準ライブラリ関数等々
    * C99で定義されるプリプロセッサは利用可能
* 組み込みデータ型
    * スカラ型, ベクタ型, ポインタ
    * 型変換関数
    * 画像用データ型
* 組み込み関数 - 必須
    * work-item用関数(そういうのがある) ,math.h, 画像の読み書き
    * Relational関数(そういうのがある), Geometric関数, 同期関数
* 組み込み関数 - Optional
    * 倍精度, アトミック命令
    * 丸めモードの選択, image3d_t surfaceへの書き込み
* 関数修飾子
    * `__kernel`修飾子で関数をカーネルコードとして宣言する
    * カーネルは別のカーネル側の関数を呼べる
* アドレス空間修飾子
    * `__global`, `__local`, `__constant`, `__private`等が存在する
    * カーネル引数となるポインタは必ずアドレス空間修飾子と共に宣言する必要がある
* ワークアイテム関数
    * `get_work_dim()`, `get_global_id()`, `get_local_id()`, `get_group_id()`等が存在する
* 同期関数
    * **Barriers** : ワークグループ内の全てのワークアイテムは続行する前にバリア関数を実行しなければならない
    * **Memory fences** : メモリ操作の順序を提供する

### OpenCLカーネルのC言語における制約

拡張されたC言語において、いくつか機能が制約されている。

* 関数ポインタは使用できない
* ポインタのポインタはカーネル内では利用できるが、カーネルの引数としては利用できない
* ビットフィールドは利用できない
* 動的配列及び構造体は利用できない
* 再帰は利用できない(まだ)
* double型は予約語だがOpenCL v1.1においてはOptional

### OpenCLのメモリモデル
OpenCLのメモリにはいくつか種類が存在する。カーネルプログラミングとは少し離れるが、`__global`や`__private`の意味を理解するには必須の要素であり、また良い性能を発揮するには理解が必要不可欠の要素であるので、ここで解説する。

* プライベートメモリ
    * ワークアイテム毎に持ってるメモリ
    * 最も高速で最も小さい: O(10) words/WI(Work-Item)
* ローカルメモリ
    * ワークグループ内で共有されるメモリ
    * ワークグループ間では共有されない
    * O(1~10) Kbytes/Work-Group
* ローバル/定数メモリ
    * 全てのワークグループから見えるメモリ
    * グローバルメモリ：O(1~10) Gbytes
    * 定数メモリ：O(10~100) Kbytes
* ホストメモリ
    * CPU側のメモリ

![OpenCLのメモリ階層](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/memory.png?raw=true)

メモリ管理は明示的であり、プログラマはホストメモリ->グローバルメモリ->ローカルメモリの順にデータが移動される事に責任を持つ(逆方向も然り)。
ホストメモリ-グローバルメモリ間はO(1~10) Gbytes/s程度の帯域がある。

### 例：直列から並列へ
この項では行列乗算を例に、逐次実行のプログラムを並列実行のカーネルに書き換える流れを見せる。

#### 行列乗算：愚直な実装
以下はサイズNの行列A, Bの行列積を計算し、Cに格納するプログラムである。

```c
void mat_mul(int N, float *A, float *B, float *C) {
    int i, j, k;
    for(int i = 0; i < N; i++) {
        for(int j = 0; j < N; j++) {
            for(int k = 0; k < N; k++) {
                C[i*N + j] += A[i*N + k] * B[k*N + j];
            }
        }
    }
}
```
これをOpenCLカーネルにする。
#### 行列乗算：OpenCLカーネル(1/2)
まずは関数修飾子とアドレス空間修飾子を付ける
```c
__kernel void mat_mul (
    const int N,
    __global float *A,
    __global float *B,
    __global float *C
) {
    int i, j, k;
    for(int i = 0; i < N; i++) {
        for(int j = 0; j < N; j++) {
            for(int k = 0; k < N; k++) {
                C[i*N + j] += A[i*N + k] * B[k*N + j];
            }
        }
    }
}
```
#### 行列乗算：OpenCLカーネル(2/2)
そして外側のループを取り除いてワークアイテムの座標をセットする
```c
__kernel void mat_mul (
    const int N,
    __global float *A,
    __global float *B,
    __global float *C
) {
    int i, j, k;
    i = get_global_id(0);
    j = get_global_id(1);
    for(int k = 0; k < N; k++) {
        C[i*N + j] += A[i*N + k] + B[k*N + j];
    }
}
```

#### 行列乗算：改善されたOpenCLカーネル
Cの中間変数を置くとパフォーマンスが上がる
```c
__kernel void mat_mul (
    const int N,
    __global float *A,
    __global float *B,
    __global float *C
) {
    int i, j, k;
    i = get_global_id(0);
    j = get_global_id(1);
    float tmp = 0.0f;
    for(int k = 0; k < N; k++) {
        tmp += A[i*N + k] + B[k*N + j];
    }
    C[i*N + j] += tmp;
}
```

次は`get_global_id()`の正確な使い方や、ワークアイテムを思い通りの形に展開する方法を説明する。

###  ワークアイテム組み込み関数
上記のOpenCLカーネルでは、`get_global_id()`といった関数を使用してインデックスを取得していた。この関数はOpenCLにおいて**ワークアイテム組み込み関数**に分類され、カーネルが実行されているCUの位置や、全体のサイズ等を取得するための関数である。OpenCLではこれらの関数から得られた値を元にメモリへアクセスを行う。

OpenCL 1.0におけるワークアイテム組み込み関数は以下の通り

| 関数名 | 説明 |
| ----- | --- |
| get_work_dim | 次元の数を返す |
| get_global_size | 全体のワークアイテムの数を返す |
| get_global_id | グローバルワークアイテムIDを返す |
| get_local_size | ローカルのワークアイテムの数を返す |
| get_local_id | ローカルワークアイテムIDを返す |
| get_num_groups | ワークグループの数を返す |
| get_group_id | ワークグループIDを返す |

### カーネルとインデックスの展開
GPUへのカーネルの展開は`clEnqueueNDRangeKernel`によって行われる。
```c
cl_int clEnqueueNDRangeKernel (
    cl_command_queue command_queue,
    cl_kernel kernel,
    cl_uint work_dim,
    const size_t *global_work_offset,
    const size_t *global_work_size,
    const size_t *local_work_size,
    cl_uint num_events_in_wait_list,
    const cl_event *event_wait_list,
    cl_event *event)
```

`work_dim`でワークアイテムとワークグループの次元を設定し、`global_work_size`で全てのワークアイテムの数を設定する。これはsize_t型の配列を引数に取り、`{8, 8}`で64個、`{8}`で8個のワークアイテムが生成される。`local_work_size`ではワークグループ内におけるワークアイテムの数を設定し、設定方法は`global_work_size`と同一である。

`clEnqueuNDRangeKernel`とワークアイテム組み込み関数の対応を図式すると以下の通りになる。

![dim = 2の場合](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/2dim_clEnqueuNDRange.png?raw=true)

![dim = 1の場合](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/1dim_clEnqueuNDRange.png?raw=true)

性能を出すためには問題に最も適した`work_dim`と`global_size`、`local_size`を選ぶことが重要である。

### OpenCLにおけるデータ型
ホストプログラムでは`cl_uint`や`cl_mem`のような独自の型が使われていた。カーネルでも同様に、OpenCLによっていくつか型が追加されている。

#### スカラーデータ型

| カーネルにおける型 | ホストにおける型 | 説明 |
| ------------------ | ---------------- | ---- |
| bool | n/a | true又はfalseが格納される。trueは1として解釈され、falseは0として解釈される。 |
| char | cl_char | 符号付き8bit整数 |
| unsigned char, uchar | cl_uchar | 符号なし8bit整数 |
| short | cl_short | 符号付き16bit整数 |
| unsigned short, ushort | cl_ushort | 符号なし16bit整数 |
| int | cl_int | 符号付き32bit整数 |
| unsigned int, uint | cl_uint | 符号なし32bit整数 |
| long | cl_long | 符号付き64bit整数 |
| unsigned long, ulong | cl_ulong | 符号なし64bit整数 |
| float | cl_float | 単精度浮動小数点数 |
| half | cl_half | 半精度浮動小数点数 |
| size_t | n/a | sizeof演算子が返す符号なし整数。`clGetDeviceInfo`におけるCL_DEVICE_ADDRESS_BITSが32bitで定義されていれば32bitとなり、64bitで定義されていれば64bitとなる。 |
| ptrdiff_t | n/a | 2つのアドレスの減算結果が返す符号付き整数。`clGetDeviceInfo`におけるCL_DEVICE_ADDRESS_BITSが32bitで定義されていれば32bitとなり、64bitで定義されていれば64bitとなる。 |
| intptr_t | n/a | 符号付き整数。voidへのポインタをこの型に変換とその逆が可能であり、またその結果は元のポインタと等しくなる。 |
| uintptr_t | n/a | 符号なし整数。voidへのポインタをこの型に変換とその逆が可能であり、またその結果は元のポインタと等しくなる |
| void | void | |

#### ベクトルデータ型

ベクトルデータ型で値をまとめる事が出来る。[tex: n]に指定する値によってサイズが変わり、[tex: n]には2, 4, 8, 16のいずれかを指定出来る。

| カーネルにおける型 | ホストにおける型 | 説明 |
| -------- | -------- | -------- |
| char[tex: n] | cl_char[tex: n] | 符号付き8bit整数ベクトル |
| uchar[tex: n] | cl_uchar[tex: n] | 符号なし8bit整数ベクトル |
| short[tex: n] | cl_short[tex: n] | 符号付き16bit整数ベクトル |
| ushort[tex: n] | cl_ushort[tex: n] | 符号なし16bit整数ベクトル |
| int[tex: n] | cl_int[tex: n] | 符号付き32bit整数ベクトル |
| uint[tex: n] | cl_uint[tex: n] | 符号なし32bit整数ベクトル |
| long[tex: n] | cl_long[tex: n] | 符号付き64bit整数ベクトル |
| ulong[tex: n] | cl_ulong[tex: n] | 符号なし64bit整数ベクトル |
| float[tex: n] | cl_float[tex: n] | 単精度浮動小数点ベクトル |

### 一般行列ベクトル積を書いてみよう
習うより慣れろ、ということで線形代数演算APIであるBLAS(そういうのがある)のSGEMV、単精度行列-ベクトル積をOpenCLで実装してみよう。

#### SGEMV
単精度行列-ベクトル積SGEMVは以下の演算を行う。

<div align='center' class='scroll'>
[tex: \displaystyle
y = \alpha Ax + \beta y
]
</div>

| 変数名   | 説明 |
| -------- | ---- |
| [tex: \alpha] | スカラー |
| [tex: A] | 行列 |
| [tex: x] | ベクトル |
| [tex: \beta] | スカラー |
| [tex: y] | ベクトル |

なお、ベクトルは全て列ベクトルとして扱われる。また行列の行サイズを**N**, 列サイズを**M**とする。

#### とりあえずカーネルプログラムを書く
このSGEMVをどうにか並列に実行したい。パッと思いつくのはワークアイテムを一次元に展開して[tex: A]の行毎に実行しやる方法だろう。

![ワークアイテムが行毎に積和を行うイメージ](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/sgemv.png?raw=true)

性能が出るかは知らないがとりあえず実装してみる。

```c
__kernel void SGEMV (
    const int N,
    const float alpha,
    const float beta,
    __global float *A,
    __global float *x,
    __global float *y,
    __global float *y_out
) {
    int i = get_global_id(0);
    float tmp = 0.0f;

    for(int j = 0; j < N; j++) {
        tmp += alpha * A[i*N + j] * x[j];
    }
    tmp += beta * y[i];
    y_out[i] = tmp;
}
```

こんな感じだろうか、解説すると9行目でワークアイテムの位置を取得し、それを[tex: y]のインデックスとする。次にベクトルのサイズ分(この場合N)だけ乗算を回す。OpenCLはカーネルに2次元配列をそのまま渡すことが出来ないため、Aのインデックスは1次元配列を2次元配列として扱ったものを使っている。

#### ホストプログラムを書く
カーネルプログラムが書けたら次はそれらを展開するホストプログラムを書いてやる必要がある。忘れない内に`clEnqueueNDRangeKernel()`を真っ先に書く。
```c
size_t M = 1024;
clEnqueuNDRangeKernel(command_queue, ko_sgemv, 
                      1, NULL, &M, NULL, 0, NULL, NULL);
```
行列の列サイズを仮に1024としてる。また今回は、第6引数である`local_work_size`をNULLに設定し、OpenCLランタイムにワークグループが持つワークアイテムの数の設定を任せている。

他の部分はOpenCLホストプログラミング入門の流れに沿って書いていく。

##### 1. デバイス, コンテキスト, コマンドキューを定義する
OpenCLホストプログラミング入門と特に変更はない。
```c
// 環境構築関係
// プラットフォームIDを確保
cl_platform_id platform_id;
cl_uint platform_num;
clGetPlatformIDs(1, &platform_id, &platform_num);

// デバイスIDを確保
cl_device_id device_id;
clGetDeviceIDs(platform_id, CL_DEVICE_TYPE_DEFAULT, 1, &device_id, NULL);

// コンテキストを作成
cl_context context;
context = clCreateContext(0, 1, &device_id, NULL, NULL, NULL);

// コマンドキューを作成
cl_command_queue commands;
commands = clCreateCommandQueue(context, device_id, 0, NULL);
```

##### 2. カーネルをビルドする
カーネルオブジェクトの変数名と`clCreateKernel()`の引数をsgemvに変更した。
```c
// プログラム関係
// プログラムオブジェクトを作成
cl_program program;
program = clCreateProgramWithSource(context, 1, (const char **)&kernelsource, NULL, NULL);

// プログラムオブジェクトをビルド
clBuildProgram(program, 0, NULL, NULL, NULL, NULL);

// カーネルオブジェクトを作成
cl_kernel ko_sgemv;
ko_sgemv = clCreateKernel(program, "sgemv", NULL);

```

##### 3. デバイスメモリを確保する
ベクトル加算と異なり、行列ベクトル積であるのでAに関するメモリの確保の部分を追加する。

上記では行列の列サイズを1024としており、もう細かいことを考えるのが面倒なので行列サイズを1024x1024とする。

まずはホストメモリを確保する。
```c
float h_A[1024*1024];
float h_x[1024];
float h_y[1024];
float alpha;
float bata;
```
次に適当に初期化する。
```c
for(int i = 0; i < 1024) {
    for(int j = 0; j < 1024; j++) {
        h_A[1024*i + j] = (float)(rand() % 100) * 0.1;
    }
    h_x[i] = (float)(rand() % 100) * 0.1;
    h_y[i] = (float)(rand() % 100) * 0.1;
}
alpha = 1;
beta = 1;
```

デバイスメモリを確保する。`d_y_out`は計算結果を格納するためのメモリである。
```c
cl_mem d_A;
cl_mem d_x;
cl_mem d_y;
cl_mem d_y_out;

d_A = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                        sizeof(float) * 1024 * 1024, h_A, NULL);
d_x = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                        sizeof(float) * 1024, h_x, NULL);
d_y = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                        sizeof(float) * 1024, h_y, NULL);
d_y_out = clCreateBuffer(context, CL_MEM_WRITE_ONLY,
                             sizeof(float) * 1024, NULL, NULL);
```

##### 4. カーネルの引数を設定する
引数がN, [tex: A, x, y, \alpha, \beta], y_outと増えているので追加する。

```c
// 引数を設定
int N = 1024;
clSetKernelArg(ko_sgemv, 0, sizeof(int), &N);
clSetKernelArg(ko_sgemv, 1, sizeof(float), &alpha);
clSetKernelArg(ko_sgemv, 2, sizeof(float), &beta);
clSetKernelArg(ko_sgemv, 3, sizeof(cl_mem), &d_A);
clSetKernelArg(ko_sgemv, 4, sizeof(cl_mem), &d_x);
clSetKernelArg(ko_sgemv, 5, sizeof(cl_mem), &d_y);
clSetKernelArg(ko_sgemv, 6, sizeof(cl_mem), &d_y_out);
```

##### 5. キューにメモリとカーネルを転送する

`clEnqueueNDRangeKernel()`と結果の読み出し部分に変更を加える。

```c
// カーネルをエンキュー
size_t M = 1024;
clEnqueueNDRangeKernel(commands, ko_sgemv, 1, NULL, &M, NULL, 0, NULL, NULL);

// 完了まで待つ
clFinish(commands);

// 結果を読み出し
float result_y[1024];
clEnqueueReadBuffer(commands, d_y_out, CL_TRUE, 0, 
        sizeof(float) * 1024, result_y, 0, NULL, NULL);
```

##### SGEMVのホストプログラム
以上の1~5を組み合わせたホストプログラムが以下の通り、クリーンアップと実行時間、計算結果の計測部分を追加しているが、それ以外は全く変更を加えていない。

このプログラムを適当な名前で保存し、OpenCLランタイムがインストールされている環境下で`gcc -lOpenCL filename.c`とするとコンパイルできる。
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <time.h>
#ifdef __APPLE__
#include <OpenCL/opencl.h>
#include <unistd.h>
#else
#include <CL/cl.h>
#endif

char *kernelsource = "__kernel void sgemv (             \n" \
"   const int N,                                        \n" \
"   const float alpha,                                  \n" \
"   const float beta,                                   \n" \
"   __global float *A,                                  \n" \
"   __global float *x,                                  \n" \
"   __global float *y,                                  \n" \
"   __global float *y_out) {                            \n" \
"   int i = get_global_id(0);                           \n" \
"   float tmp = 0.0f;                                   \n" \
"                                                       \n" \
"   for(int j = 0; j < N; j++) {                        \n" \
"       tmp += alpha * A[i*N + j] * x[j];               \n" \
"   }                                                   \n" \
"   tmp += beta * y[i];                                 \n" \
"   y_out[i] = tmp;                                     \n" \
"}                                                      \n" \
"\n";

int main(int argc, char **argv) {

    // 環境構築関係
    // プラットフォームIDを確保
    cl_platform_id platform_id;
    cl_uint platform_num;
    clGetPlatformIDs(1, &platform_id, &platform_num);

    // デバイスIDを確保
    cl_device_id device_id;
    clGetDeviceIDs(platform_id, CL_DEVICE_TYPE_DEFAULT, 1, &device_id, NULL);

    // コンテキストを作成
    cl_context context;
    context = clCreateContext(0, 1, &device_id, NULL, NULL, NULL);

    // コマンドキューを作成
    cl_command_queue commands;
    commands = clCreateCommandQueue(context, device_id, 0, NULL);

    // プログラム関係
    // プログラムオブジェクトを作成
    cl_program program;
    program = clCreateProgramWithSource(context, 1, (const char **)&kernelsource, NULL, NULL);

    // プログラムオブジェクトをビルド
    clBuildProgram(program, 0, NULL, NULL, NULL, NULL);

    // カーネルオブジェクトを作成
    cl_kernel ko_sgemv;
    ko_sgemv = clCreateKernel(program, "sgemv", NULL);

    // メモリ関係
    // ホストメモリを確保
    float h_A[1024*1024];
    float h_x[1024];
    float h_y[1024];
    float alpha;
    float beta;

    // 適当に初期化
    for(int i = 0; i < 1024; i++) {
        for(int j = 0; j < 1024; j++) {
            h_A[1024*i + j] = (float)(rand() % 10) * 0.1;
        }
        h_x[i] = (float)(rand() % 10) * 0.1;
        h_y[i] = (float)(rand() % 10) * 0.1;
    }
    alpha = 1;
    beta = 1;

    // デバイスメモリを確保
    cl_mem d_A;
    cl_mem d_x;
    cl_mem d_y;
    cl_mem d_y_out;

    d_A = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                            sizeof(float) * 1024 * 1024, h_A, NULL);
    d_x = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                            sizeof(float) * 1024, h_x, NULL);
    d_y = clCreateBuffer(context, CL_MEM_READ_WRITE | CL_MEM_COPY_HOST_PTR,
                            sizeof(float) * 1024, h_y, NULL);
    d_y_out = clCreateBuffer(context, CL_MEM_WRITE_ONLY,
                             sizeof(float) * 1024, NULL, NULL);


    // 計測開始
    clock_t begin, end;
    double gpu_time, cpu_time;
    begin = clock();

    // 実行関係
    // 引数を設定
    int N = 1024;
    clSetKernelArg(ko_sgemv, 0, sizeof(int), &N);
    clSetKernelArg(ko_sgemv, 1, sizeof(float), &alpha);
    clSetKernelArg(ko_sgemv, 2, sizeof(float), &beta);
    clSetKernelArg(ko_sgemv, 3, sizeof(cl_mem), &d_A);
    clSetKernelArg(ko_sgemv, 4, sizeof(cl_mem), &d_x);
    clSetKernelArg(ko_sgemv, 5, sizeof(cl_mem), &d_y);
    clSetKernelArg(ko_sgemv, 6, sizeof(cl_mem), &d_y_out);

    // カーネルをエンキュー
    size_t M = 1024;
    clEnqueueNDRangeKernel(commands, ko_sgemv, 1, NULL, &M, NULL, 0, NULL, NULL);

    // 完了まで待つ
    clFinish(commands);

    // 結果を読み出し
    float result_y[1024];
    clEnqueueReadBuffer(commands, d_y_out, CL_TRUE, 0, 
            sizeof(float) * 1024, result_y, 0, NULL, NULL);

    // 計測終了
    end = clock();
    gpu_time = (double)(end - begin) / CLOCKS_PER_SEC;

    // 結果を検証
    float correct_y[1024];
    float tmp = 0;
    begin = clock();
    for(int i = 0; i < 1024; i++) {
        for(int j = 0; j < 1024; j++) {
            tmp += alpha * h_A[i*N + j] * h_x[j];
        }
        tmp += beta * h_y[i];
        correct_y[i] = tmp;
        tmp = 0;
    }
    end = clock();
    cpu_time = (double)(end - begin) / CLOCKS_PER_SEC;

    int correct_cnt = 0;
    for(int i = 0; i < 1024; i++) {
        printf("correct_y[%d] = %f ", i, correct_y[i]);
        printf("result_y[%d] = %f ", i, result_y[i]);
        if(correct_y[i] == result_y[i]) {
            printf("correct !!!\n");
            correct_cnt++;
        } else {
            printf("incorrect ...\n");
        }
    }
    printf("%d matched.\n", correct_cnt);
    printf("CPU time : %lf\n", cpu_time);
    printf("GPU time : %lf\n", gpu_time);

    // クリーンアップ
    clReleaseMemObject(d_A);
    clReleaseMemObject(d_x);
    clReleaseMemObject(d_y);
    clReleaseProgram(program);
    clReleaseKernel(ko_sgemv);
    clReleaseCommandQueue(commands);
    clReleaseContext(context);

    return 0;
}
```

ここまでの流れを正しく理解できていれば、簡単な処理に対してGPUという選択肢を考慮する事が可能になるだろう。

## カーネルの最適化
カーネルに工夫を加えるとよりGPUのポテンシャルを引き出し、性能の良いプログラムを書くことが出来る。

本章では[tex: C = A \times B]の行列乗算を例に、どうすればより良いカーネルを書けるか解説する。なお、ここの知識はほぼ[Hands on OpenCL](https://handsonopencl.github.io)の受け売りであるため、最適化に関してオススメの資料などがあれば是非教えてほしい。

### 最適化前のカーネル
以下は、CがサイズNxNの行列だとして、NxN個のワークアイテムを生成する事を前提にしたカーネルである。

```c
__kernel void mmul(
    const int N,
    __global float* A,
    __global float* B,
    __global float* C
) {
    int i = get_global_id(0);
    int j = get_global_id(1);
    
    float tmp = 0.0f;
    for(int k = 0; k < N; k++) {
        tmp += A[i*N + k] * B[k * N + j];
    }
    C[i*N + j] = tmp;
}
```

### ワークアイテムとワークグループ管理のオーバヘッド
ワークアイテムとワークグループの管理には大きなオーバーヘッドが存在する。つまり仮にN=1024だとして、NxN=1024x1024=1048576個のワークアイテムをGPUに管理させるのは得策ではない。

そこでワークアイテムを直線状に配置し、各ワークアイテムにはCの行ごとの計算をさせる。一度に計算する量が減るかもしれないが、ワークアイテムの数がNに削減できる。

![ワークアイテムにCの行毎に計算させる](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/overhead_workitem.png?raw=true)

ついでに今後の最適化を見越して、`clEnqueueNDRangeKernel()`をいじってワークグループが持つワークアイテムの数(`local_work_size`)を64に設定しておく。この場合、仮にN=1024だと16個のワークグループで実行できる。

以上の最適化を加えたカーネルが以下になる。
```c
__kernel void mmul(
    const N,
    __global float *A,
    __global float *B,
    __global float *C
) {
    int i = get_global_id(0);
    float tmp;
    for(int j = 0; j < N;j++) {
        tmp = 0.0f;
        for(int k = 0; k < N; k++) {
            tmp += A[i*N + k] * B[k*N + j];
        }
        C[i*N + j] = tmp;
    }
}
```

ただ筆者の環境ではこの最適化手法は効果が無かった。むしろ遅くなった。

![遅くなった、M2090だと早くなるらしい](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/osoi.png?raw=true)

### プライベートメモリの利用

前回のカーネルを引き続き最適化する。Aの行に注目してほしい、各ワークアイテムにおいてAの行を使いまわす事が出来るのは、行列乗算の計算方法からして当然だろう。このような何度も使いまわすデータはPEの近くに置いておくのが得策である。

![Aの行はワークアイテム内で再利用できる](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/row_workitem.png?raw=true)

だが、先程最適化を加えたカーネルを見てみると、引数のAに`__global`が付いていることからもわかる通り、Aの値をわざわざグローバルメモリから取りに行っている。これは非常に無駄である。

![OpenCLのメモリ階層](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/memory.png?raw=true)

そこでAの一列分だけグローバルメモリからプライベートメモリに移してやる。その修正を加えたカーネルが以下の通り。
```c
__kernel void mmul(
    const N,
    __global float *A,
    __global float *B,
    __global float *C
) {
    int i = get_global_id(0);
    float tmp;
    
    float Awrk[1024];
    for(int j = 0; j < N; j++) {
        Awrk[j] = A[i*N + j];
    }
    
    for(int j = 0; j < N; j++) {
        tmp = 0.0f;
        for(int k = 0; k < N; k++) {
            tmp += Awrk[k] * B[k*N + j];
        }
        C[i*N + j] = tmp;
    }
}
```

この最適化によって、最適化前の約3倍の計算速度を達成できた。

![約3倍の速度](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/hayai.png?raw=true)

### 更に速く！ローカルメモリの利用
Aの行をワークアイテム内で再利用できる事は話したが、先程最適化を加えたカーネルを見てみるとBの列を取得する際に、まだグローバルメモリにアクセスしている。

これもプライベートメモリに入れてしまいたいが、カーネルの最初から最後まで不変なAの行番号と異なり、計算に用いるBの列番号はワークアイテムの一番外側のfor文で変わってしまう。

だが、内側のfor文においてBの行番号は不変であるので、ある程度Bの列も使い回せる。そこでローカルメモリにBの列を置いておき、グローバルメモリより近く、そしてワークグループ内の全てのワークアイテムがアクセス出来るようにする。

![ワークグループ内でBの列は再利用できる](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/tsukaouOpenCL/reuse_col.png?raw=true)

ローカルメモリを使うようにしたカーネルが以下の通り。ローカルメモリにBwrkを確保し、ワークグループ内のワークアイテム達が並行してBwrkにBの一列を格納している。

```c
__kernel void mmul(
    const N,
    __global float *A,
    __global float *B,
    __global float *C,
    __local float* Bwrk
) {
    int i = get_global_id(0);
    int iloc = get_local_id(0);
    int nloc = get_local_size(0);
    float tmp;
    
    float Awrk[1024];
    for(int j = 0; j < N; j++) {
        Awrk[j] = A[i*N + j];
    }
    
    for(int j = 0; j < N; j++) {
        barrier(CLK_LOCAL_MEM_FENCE);
        for(int k = iloc; k < N; k += nloc) {
            Bwrk[k] = B[k*N + j];
        }
        barrier(CLK_LOCAL_MEM_FENCE);
        tmp = 0.0f;
        for(int k = 0; k < N; k++) {
            tmp += Awrk[k] * Bwrk[k];
        }
        C[i*N + j] = tmp;
        barrier(CLK_LOCAL_MEM_FENCE);
    }
}
```
合間合間に`barrier(CLK_LOCAL_MEM_FENCE)`が挿入されているが、これはバリア命令と呼び、ワークグループ内の全てのワークアイテムがこのバリア命令を実行するまで次の命令を実行させない働きがある。引数に`CLK_LOCAL_MEM_FENCE`を指定することでローカルメモリまでに対するアクセスが確実に完了させてから停止する。

なお大して速くならない。

### 真の速さを手に入れたくば
本章ではメモリ階層だけに注目して最適化を行ってきたが、実際には多くの行列乗算を高速に行うためのテクニックが存在している。

たとえば、ワークアイテムの個数は計算機のベクトル幅の倍数でなければならない。これはAMDではwavefront、NVIDIAではwarp、CPUではSIMDレーン数と呼ばれている。

データの再利用を最適化するためにはブロック化の技術が必要になる。行列をプライベートメモリにちょうど収まるようにタイルに分解したり、タイルをローカルメモリにコピーしたり、タイル間で乗算を行う技術である。

## おわりに
GPUはパワーです、軽率にGPUを使ってGPUの需要を増やしましょう。次はOpenCLデバイス自作をやりたい。あと感想、意見、コメントがあれば呟くなりなんなりしてくれると泣いて喜びます。

おすすめbook
[［増補改訂］GPUを支える技術 ――超並列ハードウェアの快進撃［技術基礎］ WEB+DB PRESS plus Hisa Ando著](https://www.amazon.co.jp/%E5%A2%97%E8%A3%9C%E6%94%B9%E8%A8%82-GPU%E3%82%92%E6%94%AF%E3%81%88%E3%82%8B%E6%8A%80%E8%A1%93-%E2%80%95%E2%80%95%E8%B6%85%E4%B8%A6%E5%88%97%E3%83%8F%E3%83%BC%E3%83%89%E3%82%A6%E3%82%A7%E3%82%A2%E3%81%AE%E5%BF%AB%E9%80%B2%E6%92%83-%E6%8A%80%E8%A1%93%E5%9F%BA%E7%A4%8E-PRESS/dp/4297119544/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=&sr=)
