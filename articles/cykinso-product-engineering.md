---
title: "【生成AI時代を生きる】今日からはじめる、プロダクトエンジニアリングの第一歩目【体験記】"
emoji: "🐑"
type: "idea"
topics: ["初心者", "ポエム", "学習"]
published: false
---

## この記事は？

とあるエンジニアが**プロダクトエンジニアリング**に興味を持ち、それを企業の中で実践した体験記をシェアするものです。

## あなたは誰？何をしている会社?

はじめまして、[株式会社サイキンソー](https://cykinso.co.jp/)にWEBエンジニアとして勤務している[岡本]()です。

https://cykinso.co.jp/

弊社は、**「細菌叢で人々を健康に」** をミッションとして掲げているヘルステックベンチャーで、主に**腸内検査事業**を展開しています。

事業領域は BtoC・BtoB 両方にまたがっており、例えば BtoC ビジネスとして**個人向けの腸内検査キットのオンライン販売**を行っています。

https://store.mykinso.com/shop/products/gut-v2

その中でも私は、BtoC領域におけるWebプロダクトの新規開発・運用保守をメインに行っています。例えば、上記の検査キットは**商品購入から検査結果閲覧までをすべてオンラインで完結できるサービス**となっており、マイページの登録や検査結果画面などのあらゆるWeb上のユーザー体験が業務スコープです。

（検査結果イメージ画像）

Webプロダクトの主な技術スタックとしては **Ruby on Rails** を採用しています。一部画面では、 **Next.js** のようなJSのフレームワークも採用しています。

## プロダクトエンジニアとは？

そんな **「BtoCビジネスを展開する自社開発企業のWebエンジニア」** である私が、最近興味関心を持っている言葉が、**プロダクトエンジニアリング**あるいは**プロダクトエンジニア**です。

昨今、業界各所でこの言葉を耳にする機会が増えたと感じていますが、プロダクトエンジニアという言葉の定義や生まれた背景については、以下の記事で体系的に整理されています。同記事では、プロダクトエンジニアを **『開発の中心にプロダクトの価値追求を置くエンジニア』** と定義しています。

https://note.com/niwa_takeru/n/n0ae4acf2964d

> みなさんは"プロダクトエンジニア”という職種を聞いたことはあるでしょうか。一般的なフロントエンドエンジニアやフルスタックエンジニアに比べると聞き馴染みはない一方で、開発の中心にプロダクトの価値追求を置くエンジニアを指すといえばしっくりくる方も多いと思います。プロダクト志向を持つエンジニアは体感的にも増加しており、採用の面でも国内外の企業でプロダクトエンジニアの職種を定義し始めています。

従来、エンジニアの区分けといえば、フロントエンド・バックエンド・インフラ、といった技術領域に基づいたものとなっていました。これは、各領域の専門性が高く、ひとりのエンジニアが複数領域を同時に修得することが難しい（フルスタックエンジニア、はそうそう生まれてこない）ことが理由でした。

しかし、生成AIに代表される新規テクノロジーにより、**技術の一般化**が徐々に進みつつあります。これにより、ひとりのエンジニアが複数領域を修得するハードルが相対的に下がり、エンジニアの種類を技術領域で区分けする必要性が薄れてきました。この流れに先に、プロダクトエンジニア、という新しい括りのポジションが生まれたとされています。

同記事にもとづいて、プロダクトエンジニアの特徴を私なりに整理すると、

- **プロダクトを通じたユーザーへの価値提供が目的**であると考える。一方、プロダクトを構築するための技術は、目的を達成するための手段だと考える。
- 自身の専門性や業務範囲を、**フロントエンド・バックエンド・インフラといった技術領域では区別しない**。プロダクトの価値向上につながるならば、なんでもやる。
- メイン領域はあくまでエンジニアリング（＝プロダクトの開発）ではあるが、従来のエンジニアがカバーしていなかった**プロダクトマネジメント**や**UXデザイン**にも、サブ領域として手を伸ばす。

といったイメージかと思います。詳しくは、ぜひ同記事を参照いただけたらと思います。

## AI時代に求められるプロダクトエンジニアリング思考

私が「プロダクトエンジニアリング」に興味関心を持っている理由は、**AI時代にエンジニアとしてバリューを発揮し続けるために求められるのが、まさにこのプロダクトエンジニアリング思考だと考えるから**です。

生成AIの登場は、私たちエンジニアの働き方やキャリアビジョンなど大きな影響を与えつつあります。

巷では、「コードはすべてAIが出力するから、エンジニアはいらなくなる」というエンジニア不要論が唱えられることもありますが、私個人の意見としては、「一足飛びでそんな世界になることは、さすがにないんじゃないかな」と思っています。

一方、「AIを使いこなせるエンジニアとそうではないエンジニアでは、**生産性**に大きな差が生まれる」という意見については、確かにそうかも、と思います。「AIが人の仕事を奪う」という世界が来るよりも先に、「AIを使いこなす人がそうではない人の仕事を奪う」という世界の方が先にやってくるのだろうな、と私は考えています。

しかし、先はさらっと書いた**生産性**というワードは、結構危険というか、取扱注意な言葉だと考えています。エンジニアは「生産性を高めること＝サイコーによいこと」と考えがちですが、**そもそも生産性とは、一体何を生産することを指しているのでしょうか？**

例えば、ある機能を実装したいとします。従来は10の時間がかかっていたところが、生成AIを活用することにより5の時間で実装できれば、「生成AIの活用により、生産性が2倍になった」と表現できるかもしれません。

**しかし、その機能が世の中の誰にも使われることがないものだったとしたら、どうでしょうか？**

ユーザーからすれば、特に価値を感じないもの、経営者からすれば、ビジネスに貢献しない（お金稼ぎにつながらない）もの。それを生み出す速度が高まったことを「生産性があがった」と表現するのは、直感的にはちょっとおかしいというか、生産の定義から間違えてしまっている感じがあります。

つまり何が言いたいのか。**生成AIにより技術の一般化が進む未来において、エンジニアとしてバリューを発揮し続けるためには、How（どう作るか）よりも、What（何を作るのか）やWhy（なぜ作るのか）に答える能力が強く求められるようになる** というのが、私の考えです。

「こんなプロダクトを作ってください」と生成AIに命じれば、ソースコードが返ってくるような時代が、部分的には到来しつつあります。しかし、「このプロダクトでユーザーの課題は解決できますか？」や「このプロダクトを作ったら、うちの会社は儲かりますか？」という質問には、生成AIはうまく答えられない（正確には、最も無難で面白みのない答えが返ってくるだけで、事業の意思決定に活用できるような答えは返ってこない）でしょう。

プロダクトは必ず、「こんなプロダクトを作りたい」という誰かが想いから始まります。それは、経営者だったり、プロダクトマネージャーだったり、クライアント企業だったりしますが、「言われた要件にそのまま忠実に開発をする」というエンジニアは、いずれ生成AIとの競合にさらされることになると思います。なぜなら、そのエンジニアは、How（どう作るか）という、生成AIが最も得意な領域でのみバリューを発揮しているからです。

そうではなく、

- **Why（なぜ作るのか）**
    - そもそもユーザーは、どんな課題を抱えているのか？
    - ユーザーは、その課題をお金を払ってでも解決したいのか？
    - 既存の競合プロダクトと比べて、ユーザーが自社プロダクトを選択する理由はあるか？
-  **What（何を作るのか）**
    - このプロダクトや機能は、本当にユーザーの課題解決につながるのか？
    - 自社で内製化せず、外部SaaSを組み合わせた方が安く作れるのではないか？
    - このドキュメントに書かれている要件は、本当にプロダクトマネージャーの頭の中の構想を正しく言語化されたもの？

といった問いに対しても主体的に参加し、技術的に実現可能なものとしてプロダクト完成までのロードマップをひき、実際に開発完了まで完遂できる。そういった能力をもったエンジニアは、生成AIに代替されない存在になれると思います。

そして、上記の思考をひとことでまとめたのが、**プロダクトエンジニアリング思考**、であると私は解釈しました。

## 明日からあなたもできる。プロダクトエンジニアの第一歩目は？

さて、プロダクトエンジニアに興味を持った私が、まず真っ先に実践できることが何かを考えました、

それは、**自身がそのプロダクトのユーザーとなり、現状のUXを徹底的に言語化すること**、でした。

UXデザインの書籍を読むと、ほぼ共通して「ユーザーの声を聞くことが一番大事」ということが書かれています。ユーザーインタビュー、アクセス解析など手法がさまざまありますが、私が感じたことは「**ユーザーに話を聞くよりも前に、そもそもエンジニアとして働いている自分は、自社のプロダクトが提供するUXのことをちゃんと理解できているのだろうか？**」という問いでした。

企業規模が大きくなるほど、ある一人の社員が自社プロダクトの全体像を把握することが難しくなります。エンジニアは、自身が担当した機能について誰よりも詳しいかもしれませんが、「その機能は本当に必要なのか？」という問いに正確に応えるためには、UXの全体像を知っておく必要があります。

UXの全体像を知る一番確実なやり方が、**自分がユーザーになってみる**、ことです。

UXの中で自分が何をどう感じたのかを、徹底的に言語化すること。これとセットで行うことが重要です。

エンジニアの目線だと、ついついUX≒UIと考えてしまいがちです。「ボタンが押しやすいのか」とか「画面設計は直感的か」とか。もちろん、それもUXの中の重要な要素なのですが、事業インパクトが大きいのは、もっと前提にあるそもそも論であることもしばしばあって、

- そもそも、自分が何をきっかけにこのプロダクトのことを知るだろうか？
- そもそも、このプロダクトに興味を持つとしたら、どんな流れがあるだろうか？
- そもそも、自分がこのプロダクトを、この値段を払ってでも欲しいだろうか？

最初は誰しもが、プロダクトの「未顧客」から始まります。

「未顧客」のデータは、なかなか定量的には見えてきません。なにせ未顧客ですから、計測のしようがありません。

## 体験記1 実践と言語化



## 体験記2 チーム内共有



## 体験記3 改善

## まとめ

##
