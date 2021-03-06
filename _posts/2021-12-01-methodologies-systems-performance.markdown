---
layout: post
title:  "【書きかけ】詳解システムパフォーマンス 2: Methodologies"
date:   2021-12-01 11:50:00 +0900
categories:
  - Linux
  - Systems Performance: Enter and the Cloud
---
これの第2章，メソドロジーです．\\
<img src="https://www.oreilly.co.jp/books/images/picture_large978-4-87311-790-4.jpeg" width="60%">

```
パフォーマンスアナリストが複雑なシステムに立ち向かうときに，パフォーマンス問題を起こしている場所を特定し，問題を分析するためにどこから初め，どのような手順を踏んだらよいか方法を示してくれるのがメソドロジである
```

1. Terminology
======
重要な用語の一覧

| # | Term          |                                                                                                                                                                        |
|---|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | IOPS          | Input/output operations per seconds is a measure of the rate of data transfer operations                                                                               |
| 2 | Throughput    | The rate of twork performed.                                                                                                                                           |
| 3 | Response Time | the time for an operation to complete                                                                                                                                  |
| 4 | Latency       | A measure of the an operation spends waiting to be serviced                                                                                                            |
| 5 | utilization   | for resources that service requests, utilization is a measure of how busy a resource is, based on how much time in a given interval it was actively performing work.   |
| 6 | Saturation    | the degree to which a resource has queued work it cannot service                                                                                                       |
| 7 | bottleneck    | in systems performance, a bottleneck is a resource that limits the performance of the system.                                                                          |
| 8 | workload      | the input to the system or the load applied is the workload.                                                                                                           |
| 9 | cache         | a fast storage are that can dulicate or buffer a limited amount of data, to aviod communicating directly with a slower tier of storage, thereby improving performance. |

2. Models
======
システムパフォーマンスの基本原則

1. System Under Test(テスト対象システム)
---
入力(ワークロード)に対して，出力(パフォーマンス)の検証を行うために，ブラックボックス化されたアクターと理解した．\\
このアクターはテスト対象のアプリケーションが走る環境(サーバ，モバイルなどすべて)を指す．ブラックボックスになっているため\\
副次的影響を見落とす可能性がある．副次的影響を明らかにするために，構成要素を把握して，環境をマッピングする．分析を検討するために，キューイングシステムのネットワークとしてモデリングしても良い．

2. Queueing System
---
コンポーネントやリソースの中にはキューイングシステムとしてモデリング可能なものがある

3. Concepts
======
システムパフォーマンスの重要な概念

1. Latency
---
オペレーションが実行されるまでの待ち時間．

```
 Network |        Response Time                   | Completion
 Service |--------------------------------------->|
 Request |                                        |
         |------------------->|                   |
         | Connection Latency |------------------>|
                                 Data Transfer Time
```

この場合は，オペレーションはデータを転送せよという要求だ．システムはネットワーク接続が確立するまで待たなければならない．これがレイテンシである．

2. Time Scale
---
ここでは，例として各オペレーションにかかるレイテンシをまとめている．この表の内容は大まかなレイテンシのアテをするときに役に立ちそう

3.3GHz CPUのレジスタアクセスから順にしたレイテンシの例

| #  | operation                                 | latency    |
|----|-------------------------------------------|------------|
| 1  | 1 CPU cycle                               | 0.3ns      |
| 2  | L1 cache access                           | 0.9ns      |
| 3  | L2 cache access                           | 2.8ns      |
| 4  | L3 cache access                           | 12.9ns     |
| 5  | Main Memory Access                        | 120ns      |
| 6  | SSD I/O                                   | 50 ~ 120us |
| 7  | HDD I/O                                   | 1 ~ 10ms   |
| 8  | internet: san francisco to new york       | 40ms       |
| 9  | internet: san francisco to united kingdom | 81ms       |
| 10 | lightweight hardware virtualization boot  | 100ms      |
| 11 | internet: san francisco to australia      | 183ms      |
| 12 | OS virtualization syustem boot            | < 1s       |
| 13 | TCP timer-based retransmit                | 1 ~ 3s     |
| 14 | SCSI command timeout                      | 30s        |
| 15 | Hardware virtualization system boot       | 40s        |
| 16 | Physical system reboot                    | 5m         |

3. Trade-Offs
---
チューニングでよく見られるトレードオフは，CPUかメモリかを選ぶ．例えば，キャッシュ結果を格納するためにメモリを使えばCPUの利用を減らせる一方メモリサイズを節約するためにデータを圧縮するなどすればCPUを使う．システムに変更を加えるときにはのこのようなトレードオフがないか，またはトレードオフを考える．

4. Tuning Efforts
---
パフォーマンスのチューニングは，仕事が実行される場所からもっとも近いところで行ったときに最も効果的になる．つまり，ワークロードはアプリケーションによって処理されるので，アプリケーション自体のチューニングが効果的である．\\
アプリケーションレベルのチューニングはデータベースクエリの削減ないし省略できる場合があり，その場合は大きな係数でパフォーマンスが上がる．一方，デバイスレベルにまで下がってチューニングすると，アプリケーションのパフォーマンスはパーセントレベルでしか上がらない．

例：

| # | Layer       | Example tuning targets                                             |
|---|-------------|--------------------------------------------------------------------|
| 1 | application | application logic, request queue sizes, database queries performed |
| 2 | database    | database table layout, indexes, bufferring                         |
| 3 | syscall     | MMI/O, sync/async I/O flags                                        |
| 4 | filesystem  | record size, cache size, filesystem tunables, journaling           |
| 5 | storage     | RAID level, number and type of disks, storage tunables             |

アプリケーションはチューニングでは最も効果的なレベルになることが多いが，観察の場所としては最も効果的だとは必ずしも言えず，\\
例えば，遅いクエリはCPUやfilesystemでつかった時間やクエリを実行したディスクIOから最もよくわかることが多い．

OSのパフォーマンス分析をすれば，OSレベルの問題だけではなく，アプリケーションレベルの問題も見つけられる．\\
場合によっては，アプリケーションレベルで分析するよりも，OSレベルの分析を取り入れたほうが簡単に問題を見つけられる


5. Level of Appropriateness
---
どこまでやるかは，投資収益率や費用対効果に依存する．当然

6. When to Stop Analysis
---
パフォーマンス分析を行うときの課題は，いつやめるかである．

分析をやめるタイミングを検討できる３つのシナリオ:
1. パフォーマンス問題の大部分を説明したとき
2. 潜在的な投資収益率が分析のコストよりも小さい場合(割に合わないと判断できたとき)
3. 他に優先すべき投資対象がある場合

7. Point-In-Time Recommendations
---
パフォーマンスの推奨値，チューニング可能なパラメータの値が有効なのは，特定の基準時だけであり，ソフトウエアやハードウエアがアップグレードされたりあしたタイミングから無効になることがある\\
また，インターネット上に転がっているソリューションが手っ取り早く解決できることもあるが，システムやワークロードに適していない場合，問題が悪化することさえある．どのようなチューニング可能なパラメータがあって，過去に変更が必要になったかを見られるように，そのような推奨値を一覧で見られるようにすると便利である．また，最近ではPuppet, Salt, Chef, Ansibleなど構成管理ツールを用いて調整可能なパラメータを変更して，詳細な履歴を含むバージョン管理システムに保存しておくのは便利である．

8. Load vs. Architecture
---
例えば，他のCPUはアイドル状態で，特定のCPUがビジー状態になっているようであれば，この場合，アプリケーションのシングルスレッドというアーキテクチャがパフォーマンスの限界を生み出している\\
一方，すべてのCPUがビジー状態になって要求がキューイングされている場合は負荷の問題である．この例ではパフォーマンスの限界を生み出しているのは，使えるCPUの能力である．CPUが処理できる以上の負荷がかかっている．

9. Scalability
---
典型的なスループットプロファイルの中では，しばらくの間は線形スケーラビリティが観察されるが，飽和点に達すると，競合の増加とコヒーレンスのオーバヘッドにより，処理量が減り，スループットが落ちてしまう．\\
マルチスレッド環境では，さらにコンテキストスイッチが増え，CPUリソースを消費し，処理される実際の仕事が減って，スループットは下がり始める．CPUの負荷の場合はゆっくりとした低下プロファイルとなるが，メインメモリの場合はシステムがページングを起こし始めたときに急激な低下プロファイルを示すときがある

10. Metrics
---
パフォーマンスメトリックは，対象のアクティビティを測定するシステム，アプリケーションまたは追加のツールによって生成された統計です．コマンドラインで吸ううち的に，またはグラフィカルにパフォーマンス分析と監視のために調査する．

システムパフォーマンスメトリックの一般的なタイプは次の通りです．
- スループット: １秒あたりの操作またはデータ量のいずれか
- IOPS: 1秒あたりのIO操作
- 使用率: リソースのビジー状態
- レイテンシ: 平均またはパーセンタイルとしての操作時間

スループットの使用方法はコンテキストによって異なる．データベースの場合は１秒あたりのクエリまたは操作の尺度となり，ネットワークであれば，１秒あたりのビットまたはバイトの尺度となる．IOPSはIO操作のスループット測定である．

1. オーバーヘッド
観察者効果を避けなければ，メトリクスそのものが測定対象のパフォーマンスに影響を与えうる

2. 問題
ソフトウエアベンダが提供するメトリックは完全ではない，複雑で信頼性が低く，不正確であり，さらには間違っている可能性さえある

11. Utilization
---
使用率は時間かキャパシティをベースとして定義可能である

#### 1. Time-based
時間ベースの使用率は，待ち行列理論で正式に定義されている

```
  U = B / T
  where U = Utilization, B = Total time the system was busy during T, T = The observation period
```

時間ベースの特徴としては，コンポーネントが複数のオペレーションを実行できるものの場合(例えばディスク)，\\
100%になっていてもアイドル状態のディスクがあるため，より多くの要求を受け付けられることがある．例えば，iostatの%b`%util?`がこの時間ベースの指標となる．

```
Linux 5.13.0-21-generic (shun159.localhost)     2021年12月02日  _x86_64_        (4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.04    0.05    0.69    0.11    0.00   97.13

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
nvme0n1          0.39      8.63     0.32  45.09    0.28    22.21    1.54     24.98     1.07  40.94    3.30    16.18    0.20    252.83     0.00   0.00    0.45  1296.48    0.17    1.17    0.01   0.24
```

#### 2. Capacity-based
時間ではなく，容量の観点から使用率を定義する．つまり，使用率が100%のディスクはそれ以上の作業を受け入れられないことを意味する．\\
ただ，残念ながら必ずしも両方の情報が提供されるとは限らない．

12. Saturation
---
要求がリソースに対してどの程度であるかを表す．Capacityベースの使用率が100%を超えたとき，キューイングが始まると発生する

13. Profiling
---
プロファイリングは，調査及び理解できるターゲットの図を作成する．プロファイリングは通常，システムの状態を一定間隔でサンプリングし，それらのサンプリングのセットを調査することで実行する

14. Caching
---
パフォーマンスの向上させるためによく使われている．キャッシュのパフォーマンスを知るための指標として各キャッシュのヒット率というものがある．これはキャッシュ内に必要とされるデータが見つかる仮数と見つからない回数の比率である．

```
    hit ratio = hits / (hits + misses)
```
比率が高いほど，より高速なメディアから正常にアクセスされたデータが多く反映されるため，高いほどよい．\\
また，例えばキャッシュヒット時とキャッシュミス時の速度差のために，非線形のプロファイルになる．ヒット率差が大きければ大きいほど傾斜は急になる．

キャッシュのパフォーマンスを理解するための指標としては，１秒あたりのキャッシュミスを表すキャッシュミス率がある．\\
これはここのキャッシュミスによるパフォーマンス低下に比例しており，解釈しやすい．

### 1. アルゴリズム
キャッシュの限られたスペースに何を格納するかは，キャッシュ管理アルゴリズムにとポリシーによって決まる．
    - MRU(Most Recently Used)
    - LRU(Least Recently Used)
    - MFU(Most Frequently Used)
    - LFU(Least Frequently Used)
    - NFU(Not Frequently Used)

### 2. Hot, Cold and Warm Cache
キャッシュの状態の説明する際に使用する単語
    - Cold: から，または不要なデータが格納されたキャッシュである．ヒット率は０
    - Hot: よく要求されるデータがセットされていて，例えば99%以上というような高いヒット率を示す
    - Warm: 役に立つデータも格納しているが，ホットというほどヒット率は高くない
    - Warmth: キャッシュのウォーム度をあらわす．ウォーム度を上げるアクティビティといえば，キャッシュのヒット率を上げるためのアクティビティを指す．
    
4. Perspectives
===
パフォーマンス分析には，オーディエンス，指標，アプローチが異なる２つの視点がある．ワークロード分析とリソース分析だ．

```                                                                  
                          workload                                   
                             |                                       
                             v                                       
               +--------------------------+                          
               |       application        | ------  Workload analysis
               +--------------------------+         -----------------
                    |             |                   ||             
                    |             v                   ||      AA     
          ---       |     +-----------------+         ||      ||     
           |        |     |  system library |         ||      ||     
           |        |     +-----------------+         ||      ||     
operating  |        |             |                   ||      ||     
system     |        v             v                   ||      ||     
software   |   +--------------------------+           ||      ||     
stack      |   |       application        |           ||      ||     
           |   +--------------------------+           ||      ||     
           |                |                         ||      ||     
           |                v                         ||      ||     
           |   +--------------------------+           ||      ||     
           |   |         kernel           |           ||      ||     
           |   +--------------------------+           VV      ||     
          ---               |                                 ||     
                            v                                 ||     
               +--------------------------+                   ||     
               |         device           | ------ Resource analysis 
               +--------------------------+        ----------------- 
                                                                     
```                                                                  

1. Resource Analysis
---
実施するのは主として物理環境のリソースに対して責任を追うシステム管理者である．作業内容は以下の通り，
    - パフォーマンス問題の調査: 特定のタイプのリソースに問題があるかどうか判定する
    - キャパシティプランニング: 新システムの規模を決めるために役立つ情報を得る．また，既存のシステムリソースを使い切ったかどうかを確認する
    
リソース分析に適した指標としては，以下がある
    - IOPS
    - スループット
    - 使用率
    - 飽和
    
2. Workload Analysis
---
アプリケーションのパフォーマンスを解析する．ワークロード分析は主として，アプリケーションデベロッパやサポートスタッフなど，アプリケーションソフトウエアとその構成に責任を持つ人々によって使われる

```
             workload          +-------------+
 1. input -------------------->|             |
                 A             |             |
     2.latency   |             | Application |
                 v             |             |
 3.completion<-----------------|             |
                               +-------------+
 1. input: workload applied
 2. latency: the response time of the application
 3. completion: Looking for errors
```
アプリケーションのパフォーマンスを表現する最も重要な指標は，レイテンシである.\\
MySQLならばクエリレイテンシであり，ApacheであればHTTP要求レイテンシである．\\
ワークロード分析は，問題を明らかにして確認し，レイテンシの原因を見つけ，フィックス後にレイテンシが改善されたことを確認する．出発点がアプリケーションである．

レイテンシの調査は通常，アプリケーションからライブラリ，OSへと下へ向かって掘り下げていくことになる．

ワークロード分析に的にした指標としては，次が挙げられる.
    - スループット(Transactions/sec)
    - レイテンシ

5. Methodologies
===
システムパフォーマンス分析とチューニングのための様々なメソドロジーがある．最初の課題はどこからはじめ，どのように進めるか，である．\\
パフォーマンスの問題はソフトウエア，ハードウェア及びデータパスに沿ったコンポーネントを含むどこからでもする可能性がある．\\
メソドロジーは分析を開始する場所を示し，従うべき効果的な手順を提案することにより，これらの複雑なシステムにアプローチするのに役立つ．

数が多いのでアンチメソッドなどはざっと読んで飛ばす…
1,2のアンチメソッドは初心者がよくやりそうではある．3は笑い話のようでジョークになっていない…

1. Streetlight anti-method
---
2. Random change anti-method
---
3. Blame-someone-else anti-method
---
4. Ad hoc checklist method
---
5. Problem statement
---
非常に良いアプローチと感じた．ここから始めても良いかもしれない．

1. パフォーマンスに問題があると思ったのはなぜか
2. このシステムは，良好なパフォーマンスで動いていたことがあったか
3. 最近の変更はなにか．ソフトウエアかハードウエアか，負荷か
4. その問題はレイテンシか実行時間で表現できるか
5. この問題は他の人やアプリケーションに影響を及ぼしているか
6. 環境はどうなっているのか．どのソフトウエア，ハードウエアを使っているのか．バージョン・構成はどうか

これを尋ねる．自問しても良いだろう

6. Scientific method
---
仮説を立て，それを検証することで未知の事象について検討する

1. Question: パフォーマンス問題の記述(クエリが遅い原因はなにか)
2. Hypothesis: 低パフォーマンスの原因の仮説(他のテナントがディスクIOを実行していて，DB程度とディスクIOを奪い合っている)
3. Prediction: 観察または実験による検証の方法を組み立てる(クエリの過程でファイルシステムにIOレイテンシが計測されるならば，クエリの遅さはファイルシステムに原因がある)
4. Test: 検証を行い，データを集める(クエリレイテンシの比率という形でデータベースファイルシステムレイテンシをトレーシングする)
5. Analysis: 分析を行い，予測を検証する(ファイルシステム待ちでは5%未満の時間しか使っていないことがわかった)

問題がまだ解決されていないくても，環境の中のコンポーネントを除外できる．ステップ２へ戻り，新たな仮説をたてる．

7. Diagnosis Cycle
---
6と似た手法を用いる．`仮説→計測→データ→仮説`

科学的メソッドと同様に，このメソドロジーもデータのコレクションを使って仮設を意図的に検証する．診断サイクルはデータがすぐに新しい仮説を導いてくることを強調し，その仮説はすぐに検証され磨かれていく．どちらのアプローチも理論とデータのバランスが良い

8. Tools method
---
9. USE method
---
USE(Utilization, Saturation, Errors)メソッドはシステム的なボトルネックを見つけるために，パフォーマンス調査の初期の段階で使うべきメソドロジーである\\
すべてのリソースについて，使用率，飽和，エラーをチェックする

これらの用語は次のように定義されている：

- リソース: 物理サーバのすべての機能的なコンポーネント
- 使用率: 決められた間隔の中で，リソースが要求を処理するためにビジー状態だった時間の割合
- 飽和: 処理できない要求(キューで待機していることが多い)
- エラー: エラーイベントの回数

#### 1. 手順
使用率と飽和をチェックする前に，まずエラーをチェックする．このメソドロジーはシステムのボトルネックになっっていそうな問題を見つける．\\
しかしシステムは複数のパフォーマンス問題を抱えている場合がある．そのため，最初に見つけたものはある問題であっても決定的な問題ではないかもしれない．\\
見つかった問題は，更に他のメソドロジーで調査してもよい．

![USEmethod_flow](https://www.brendangregg.com/USEmethod/usemethod_flow.png)

#### 2. 指標の表現:
- 使用率: インターバルを通じての割合(1つのCPUが90%の使用率で実行されている)
- 飽和: 待機キューの長さ(CPUは，平均して長さが4のランキューを持っている)
- エラー: 報告されたエラーの数(このネットワークインターフェイスは，50回のレイトコリジョン起こしている)

長いインターバルで見ると全体的な使用率は低いように見えても，使用率が短時間に極端に高くなると，飽和やパフォーマンス問題を引き起こすことがある．一部のモニタリングツールは5分の間隔で使用率を報告してくる．例えば，CPUの使用率は毎秒毎秒去痰に大きく変化するので，5分の平均では，短時間に使用率が100%になり飽和を起こしていてもそれが見えなくなってしまう．

#### 3. リソースリスト
USEメソッドの最初のステップは，リソースのリストを作ることである．できる限り完全なリストを作る．

例:
- CPU: ソケット，コア，ハードウエアスレッド
- Main Memory: DRAM
- NIC: イーサネットポート
- Storage Device: ディスク
- Controller: ストレージ，ネットワーク
- Inter-Connect: CPU, メモリ, I/O

ここのコンポーネントは一般に単一のリソースタイプとして機能する．例えば，メインメモリは容量リソース，ネットワークインターフェイスはIOリソースである．しかし，複数のリソースタイプとして機能するコンポーネントもある．ストレージデバイスはIOリソースであると同時に容量リソースである．


ハードウエアキャッシュなどの一部の物理コンポーネントは，チェックリストに載せなくてもよい．USEメソッドは使用率，飽和が高くなるとボトルネックになりがちでパフォーマンスを低下させるようなリソースで最も効果的なメソドロジーだが，キャッシュは使用率は高い方がパフォーマンスが上がる．

#### 4. 機能ブロックダイヤグラム
リソースの反復的チェックはシステムの機能ブロックダイアグラムを探したり，新たに書いたりして行う方法もある．このようなダイアグラムはコンポーネントの関係も示すので，データフローのなかのボトルネックを探すときに特に役に立つ

![v480](https://www.brendangregg.com/USEmethod/v480.png)

#### 5. 指標のサンプル
リソースのリストが完成したら，指標のタイプを検討してみる．使用率，飽和，エラーである.

| # | Resource          | Type        | Metric                                                             |
|---|-------------------|-------------|--------------------------------------------------------------------|
| 1 | CPU               | Utilization | CPU Utilization(either per CPU or a system-wide average)           |
| 2 | CPU               | Saturation  | Run-queue length, scheduler latency, CPU pressure                  |
| 3 | Memory            | Utilization | Available free memory                                              |
| 4 | Memory            | Saturation  | Swappign, page scanning, OoM events, memory pressure               |
| 5 | Network Interface | Utilization | Receive throughput/max bandwidth, transmit thoughput/max bandwidth |
| 6 | Storage device IO | Utilization | Device busy percent                                                |
| 7 | Storage device IO | Saturation  | Wait queue length, IO pressure(Linux PSI)                          |
| 8 | Storage device IO | Errors      | Device Errors (soft, hard)                                         |

すべての組み合わせで繰り返し，各指標を取り出すための命令を入れておく，また，現在取得できないメトリックのメモを撮っておく，これはKnown-Unknownsである．

#### 6. Software resources
ソフトウエアリソースの一部も同じようにして検証できる．通常は，アプリケーション全体ではなく，小さなコンポーネントを対象とする．例えば…

- Mutex Locks
: 使用率は，ロックされていた時間，飽和はロックを待ってキューイングされていたスレッドとして定義する
- Thread Pools
: 使用率は，スレッド要求を処理してビジー状態になったとき，飽和はスレッドプールからのサービスまちになっていた要求の数として定義する
- Process/Thread Capacity
: システムは，プロセスやスレッドの数を限定している場合がある．使用率はそれらが使われているときである．飽和は，それらの割当を待っているときである．エラーは割当が失敗したときである．
- File descriptor capacity
: プロセス/スレッドの容量と同様だが，対象がファイル記述子となっている．指標が使える場合はそれを使う．そうでなければ，レイテンシ分析などの他のメソドロジーを利用する

#### 7. Suggested Interpretations
メトリックタイプの一般的な解釈方法の提案

- 使用率
: 使用率100%は，一般にボトルネックを起こしている兆候である．一方で，60%以下の使用率も問題を示している場合がある．\\
間隔次第では短時間100%使用率が隠されている可能性があるからだ．
- 飽和
:飽和が少しでも起きていれば，問題が起きている可能性がある．飽和は待機キューの長さかキュー内での待ち時間によって計測される
- エラー
:エラーカウンタが0以外なら精査すべきである．特に，パフォーマンスか低いときにエラーが増える場合は精査したほうがよい．

10. RED method
---
11. Workload characterization
---
12. Drill-down analysis
---
13. Latency analysis
---
14. Method R
---
15. Event tracing
---
16. Baseline statistics
---
17. Static performance tuning
---
18. Cache tuning
---
19. Micro-benchmarking
---
20. Performance mantras
---
21. Queueing Theory
---
22. Capacity Planing
---
23. Quantifying Performance gains
---
24. Performance Monitoring
---
