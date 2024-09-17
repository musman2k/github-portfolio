---
layout: post
title: ASPLOS 2020 の発表動画を視聴する
categories:
- blog
---

##### はじめに
年明けから世界中で猛威を振るっている COVID-19 の影響を受けて国際会議がオンラインで開催されるようになりました（Virtual Conference などと呼ばれています）。そのため三月にスイスのローザンヌで開催予定だった計算機アーキテクチャのトップ国際会議の一つ ASPLOS もオンラインでの開催となりました。会議は基本的に YouTube にアップロードされた発表動画と Slack を併用して進行されました。発表動画は [こちら](https://www.youtube.com/playlist?list=PLsLWHLZB96VeVp3IVzvSH58ttVz_Anr7H) にあります。会議にはアクティブに参加できませんでしたが、折角なので興味のある論文の発表動画を 10 数本見てみました。

面白いと思った論文を 4 つほど簡単に紹介します（順不同）。
###### Elastic Cuckoo Page Tables: Rethinking Virtual Memory Translation for Parallelism
ベストペーパー 4 本のうちの 1 本。UIUC（イリノイ大学アーバナシャンペーン校）の Josep Torrellas 教授のグループの論文です。動画は [こちら](https://www.youtube.com/watch?v=BIvpGx-znlk) 論文は [こちら](https://dl.acm.org/doi/10.1145/3373376.3378493)。

アドレス変換をつかさどる既存のページテーブルは複数段の木構造を用いる通称ラディックスページテーブルで構成されています（x86-64 アーキテクチャでは 4 段で、Intel Sunny Cove マイクロアーキテクチャからは 5 段）。この構成は逐次的にポインタチェイシング（ページウォーク）をする必要があるため性能オーバーヘッドが大きいという問題点があります。TLB、ヒュージテーブル、MMU キャッシュなどの改良を施してもなおメモリインテンシブな次世代アプリケーションでは 20--50% の実行時間をアドレス変換に費すことになるそうです。TLB を大きくするのはアクセス時間、面積、消費電力の観点から難しく、メインメモリサイズが今後さらに大きくなっていくためアドレス変換のオーバーヘッドはより深刻になると考えられます。

そこで以前からハッシュページテーブルというものが研究されています。簡単に言うと仮想アドレスのハッシュ値をエントリのインデックスにして物理ページを得る、というものです。とても素直なアイデアだと思いますが現実には色々と問題があります。空間的局所性が失なわれる、ハッシュタグを格納する必要があるので PTE（page table entry） のサイズが大きくなる、ハッシュ衝突に対処する必要がある、などです。近年の研究でこれらの問題は解決されつつありますがハッシュ衝突の問題は依然解決されていません。この論文ではアーキテクチャ研究ではお馴染みの Cuckoo Hashing（を改良したハッシュ）を用いてこの問題を解決しました。メモリレベル並列性を活用することで高性能化を達成していることも含めて優れたデザインだと思います。

###### Classifying Memory Access Patterns for Prefetching
スタンフォードの Christos Kozyrakis 教授と Google の Parthasarathy Ranganathan 博士という凄いタッグによる論文です（第一著者の学生は今は Google の社員）。動画は [こちら](https://www.youtube.com/watch?v=bpLl6PfUcmw)[^1] 論文は [こちら](https://dl.acm.org/doi/10.1145/3373376.3378498)。

[^1]:コロコロと場面転換が行われて正直気が散ってしまいました笑。

数 100 サイクルかかるメモリアクセスレイテンシはプロセッサの性能を律速する大きな要因となっています。プリフェッチはこのレイテンシを短縮する有望な技術ですが、汎用的に性能向上が期待できるプリフェッチャはこれまでに提案されていません（そんなものが存在するかは疑わしい）。特定の分野のアプリケーションに対して効果が大きいプリフェッチャに関する論文は多く発表されているものの、最新の商用マイクロプロセッサは単純な next-line やストライドプリフェッチャを搭載しているのが現状です。

この論文ではプログラムのメモリアクセスパターンを解析し、どの程度の割合のキャッシュミスがどのような特性を持っているか（どのようなプリフェッチャを用いれば対応できるのか）、理想のプリフェッチャではどの程度キャッシュミスを削減可能か、などに答えることのできる方法論を提案しています。あわせてこの方法論に基づく解析ツールを開発しました。解析の土台として、メモリアクセスのパターンを列挙し（ストライドアクセス、ポインタチェイシングなど）、メモリアクセスを分類するための「言語」を構築しています。解析ツールへの入力は、対象プログラムのバイナリ、またプログラムを実行して得られるトレースおよびキャッシュミスプロファイルです。出力はメモリアクセスの分類と、有効と思われるソフトウェアプリフェッチのヒントになります。もう少し具体的に言うと、このツールは実行時のプログラムカウンタおよびアクセスしたアドレスのトレースを静的なデータフロー解析と組み合わせることで、それぞれのメモリアクセスのアドレス計算に必要な「プリフェッチカーネル」を対象プログラム内から抽出します。このカーネルを上記の言語を用いて解析します[^2]。

[^2]:短い文章で説明することの限界を感じています。とりあえず雰囲気だけでも感じとっていただければ幸いです。

このツールを用いて Google のデータセンタで実行されているものを含むメモリインテンシブなプログラムの解析結果を提示しています。またこの解析に基づいたプリフェッチャを提案し、シミュレーションによる評価も行なっています。解析結果から、様々なアプリケーションに対応するためソフトウェアによって高度にカスタマイズ可能なプリフェッチャが必要になるのではないか、と結んでいます。

###### Thesaurus: Efficient Cache Compression via Dynamic Clustering
カナダの UBC（ブリティッシュコロンビア大学）のグループによる論文です。動画は [こちら](https://www.youtube.com/watch?v=w0e0Q-Vp01U) 論文は [こちら](https://dl.acm.org/doi/abs/10.1145/3373376.3378518)。素直な論文なので動画を見るだけでも大体の内容が分かると思います。

この論文はいわゆるキャッシュコンプレッション（キャッシュ圧縮）に関する研究です。キャッシュコンプレッションとは、簡単に言うとキャッシュ内に保存するデータを圧縮して実効的な容量を大きくすることで性能向上を狙うものです。既存の研究ではキャッシュライン内のデータの類似性を利用した圧縮技術に着目していました。この論文では、キャッシュ内のキャッシュライン間のデータの類似性を利用してキャッシュコンプレッションを実行する技術 Thesaurus を提案しています。

Thesaurus では局所性鋭敏型ハッシュ（locality sensitive hashing (LSH) ：類似したデータが高確率で同じハッシュ値となるようなハッシュ）を用いることで似たようなデータを持つキャッシュラインを検出します。LSH で同じハッシュ値を持つキャッシュラインを Base-Delta-Immediate (B∆I) という圧縮手法を用いて圧縮します[^3]。既存手法では LLC（last level cache：最下位キャッシュ）のワーキングセットを平均して 1.28x--1.48x 程度圧縮できていましたが、Thesaurus では 2.25x まで圧縮できるようになったとのことです。

[^3]: B∆I はキャッシュコンプレッションの state-of-the-art 技術で 2012 年の PACT という会議で発表されました。佐々木は発表の場にいたのですが、発表者の質疑が印象に残っています。こんなに参照される論文になるとは思いませんでした。

###### Accelerating Legacy String Kernels via Bounded Automata Learning
ミシガン大学アナーバー校のグループによる論文です。あまりきちんと理解できたわけではないため、ふわっとした説明になってしまいますが、個人的には一番ワクワクした論文かもしれません。動画は [こちら](https://www.youtube.com/watch?v=FB_2h5UyH1o) 論文は [こちら](https://dl.acm.org/doi/10.1145/3373376.3378503)。

この論文ではレガシーソフトウェアに含まれる特定のクラスのコード、具体的には Boolean string kernel functions（真または偽を返す関数）、を FPGA で高速化する手法を提案しています（本手法を実装した AutomataSynth は [こちら](https://github.com/kevinaangstadt/automata-synth)）。キーワードを列挙しますが、model learning (分野：learning theory)、software model checking (software engineering)、string decision procedures (PL/theory)、high-performance automata architectures (hardware) を駆使することで、対象となる関数と等価な振舞いをする高速なハードウェアを合成します。
FPGA（というかハードウェア）デザインは基本的にステートマシンとして記述されているため、有限オートマトンを中間表現として用います。このアプローチは HLS (high level synthesis) などよりもハードウェアに落としこむのに適していると著者らは主張しています。意図的にソフトウェア実装とハードウェア実装を切り離しているということですね。

まず対象関数と等価（または近似的）に振る舞う有限オートマトンを求めるために、Angluin の L\* アルゴリズム（を改良したもの）を用います。ここで近似的、とはある長さの入力までは等価であるが、その長さ以上に関しては等価であることを保証しない、という意味です。等価（または近似的）であるかの判断には有界モデル検証（BMC: bounded model checking）を用います。GitHub から対象となる 18 の関数（例えば <mycode>git</mycode> から <mycode>git_offset_1st_component()</mycode> や <mycode>Linux</mycode> から <mycode>{end,start}_line()</mycode> など）をマイニングして AutomataSynth を評価したところほぼ全てで等価な回路の出力に成功したそうです（3 つは近似的）。

##### まとめ
かなり駆け足感ありますが以上で紹介を終わります。どれか一つでも興味を持ってもらえたら嬉しいです。最後に、先日 ASPLOS に関していくつかツイートをしたので貼り付けておきます。動画はどれも一見の価値があると思うので、お時間のある方はぜひご覧ください。

---

<div style="float: left; width: 315px;">
<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">過去の ASPLOS で発表され大きなインパクトを与えた論文に与えられる ASPLOS Most Influential Paper Award。今年は二本。うち一本が 2002 年の ZebraNet。恥ずかしながら未読でしたが、動画からも分かるとおりワクワクする凄い仕事ですね。是非動画だけでも見てください！<a href="https://t.co/D4cnLl5i1t">https://t.co/D4cnLl5i1t</a></p>&mdash; 佐々木 広 (@hrshssk) <a href="https://twitter.com/hrshssk/status/1259421412768411649?ref_src=twsrc%5Etfw">2020年5月10日</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>
<div style="float: left; width: 315px;">
<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">ASPLOS の名物セッション WACI (wild and crazy ideas)。今はまだ現実味がないものの、いつか本当に研究されたりして？という突拍子もなく面白いアイデアを披露する場です。示唆に富んでますが気軽に見れるので息抜きにでもどうぞ。何か良いアイデアを閃くかもしれません！<a href="https://t.co/YqjavWbjmw">https://t.co/YqjavWbjmw</a></p>&mdash; 佐々木 広 (@hrshssk) <a href="https://twitter.com/hrshssk/status/1260786241957552128?ref_src=twsrc%5Etfw">2020年5月14日</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>
<div style="clear: left; float: left; width: 315px;">
<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">ASPLOS の発表動画を 10 数本見てみました。スライドを用いて説明する通常の研究発表のような形式の動画が多い中、Best Video Award を受賞したこの動画はもっと凝っていて個人的には飽きずに見られて楽しかったです！<a href="https://t.co/o7lNKiyeoW">https://t.co/o7lNKiyeoW</a></p>&mdash; 佐々木 広 (@hrshssk) <a href="https://twitter.com/hrshssk/status/1261982953887821824?ref_src=twsrc%5Etfw">2020年5月17日</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>
<div style="float: left; width: 315px;">
<blockquote class="twitter-tweet" data-conversation="none" data-lang="ja"><p lang="ja" dir="ltr">ASPLOS 2018 は投稿 303 採択 56、2019 は投稿 351（前年比 +16%）採択 74、2020 は投稿 478！（前年比 +36%）採択は 86。投稿数が多いカテゴリは HW/SW コデザイン、ヘテロアーキ・アクセラレータ、OS、セキュリティ、ディープラーニングシステム、メモリ、並列アーキ、コンパイラ、マイクロアーキ。</p>&mdash; 佐々木 広 (@hrshssk) <a href="https://twitter.com/hrshssk/status/1258953105506398208?ref_src=twsrc%5Etfw">2020年5月9日</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>
---
