## 実践テスト駆動開発を読んでメモ
### 序文
- 英語タイトル「Growing Object-Oriented Software, Guided By Tests」の意味
  - なぜ`Growing`かというと、ソフトウェアをインクリメンタルに開発するもの感をだしたかったから
  - なぜ`Guided`かというと、テストを先に書くというプロセスがソフトウェアの設計を支援するものだから。
- 本の構成
  - 第１部「導入」：テストフレームワークの紹介など
  - 第２部「テスト駆動開発のプロセス」：プロセスの説明
  - 第３部「動くサンプル」：応用的なサンプル
  - 第４部：持続可能なテスト駆動開発」：システムをメンテ可能な状態に保つプラクティス
  - 第５部「高度なトピック」：テスト駆動開発の実践が困難な領域について
  - 付録：jMockとHamcrestの解説
### 第一部
#### 第一章「テスト駆動開発のポイントとは？」
- テスト駆動開発の核心サイクルは「失敗するユニットテストを書く」「通るようにする」「リファクタリングする」からなる
  - テストを書くことによる恩恵
    - 次の作業の受け入れ基準が明確
    - 疎結合が促される
    - コードが何をするかの説明になる
    - 網羅的なリグレッションテストスイートを厚くできる
  - テストを実行することによる恩恵
    - 早期のエラー検出ができる
    - 将来必要かも…と当面不要な柔軟性を持たせなくなる
- 初手ユニットテストを書く、のは誤り
  - 最初に書くのは「受け入れテスト」。新規追加機能を利用するテスト。
    - 可能な限りエンドツーエンドテストにするべき
  - これを通すための小さなサイクルがユニットレベルのテスト/実装/リファクタサイクル
  - 筆者はテストの種類を以下３種類に分類。受け入れ・インテグレーションのテクニックは紹介せず3章で例示するのみにとどめているらしい(∵採用技術や組織の文化で大きく違うから）
    - 受け入れテスト（何を構築するのかの理解・合意）
    - インテグレーションテスト（変更不可の領域(世間一般のフレームワーク・外部提供のライブラリ等）と合わせても動作するか）
    - ユニットテスト
- エンドツーエンドは外側の品質を保つものであるのに対し、ユニットテストは内側の品質（理解しやすさ、変更しやすさ）を保つためのもの
#### 第二章「オブジェクトをテスト駆動開発する」
- 値とオブジェクト
  - 値：状態不変のインスタンス。IDを持たず、状態が同じであれば同一とみなされる
  - オブジェクト：IDと状態を持つインスタンス。状態が時間によって変化する
- ドメインモデルはオブジェクト間のコミュニケーションに宿る
  - 静的なクラス表現は物事の一側面しか表現できない
- 命じよ、訊ねるな（デメテルの掟）
  - オブジェクトが意思決定を行う際、内部情報orトリガとなるメッセージ情報だけを元にせよ
  - 他のオブジェクトを操って何かを引き起こすことはさける
  - `Obj.getA().getB().setSavingFlag(true)`を一つの関数`allowSavingOfCustomisations()` にまとめることで、実装を隠蔽
- しかし、ときに訊ねることもある
  - コレクションから情報取り出し等は尋ねざるを得ない。でも、表現力は保つこと。
  - ``carriage.getSeats().getPercentReserved() < threshold`` は `carriage.hasSeatsAvailableWithin(threshold)` とすべし
- 内部状態は極力露呈させない・外部とのやり取りを中心にせよ…を守るオブジェクトをテストするには？
  - 隣接モジュールをモックする。テスト対象のオブジェクトだけをテストしたい＆隣接オブジェクトがどういうものかはわかっているため
#### 第三章「ツールの紹介」
- JUnit4
  - `@Test` のついたメソッドをテストとして扱う
  - `@Test(expected=例外) `で例外が出ることを期待するテストもかける
  - `@Before` でフィクスチャ（テスト開始時点で定められた状態）をセットアップできる。 `@After` がティアダウン
- Hamcrest
  - オブジェクトがマッチする条件を宣言的にかける
    - `Matcher<String> containBananas = new StringContains("banana"); assertFalse(containsBananas.matches(inputString)); ` みたいな
    - スタティックファクトリーを用いると `assertThat(inputString, not(containString("banana")))` とかかける
- jMock2の紹介
  - `@RunWith(JMock.class)`アノテーションをつけると使える。
  - `Mockery context = new JUniut4Mocker(); ActionEventListener listener=context.mock(ActionEventListener.class)` でActionEventListenerのモックが作れる
  - `context.checking(new Expectations(){{ oneOf(listener).auctionClosed()}});` でauctionCloseイベントが一回呼ばれたことを期待

### 第二部
#### 第四章「テスト駆動のサイクルに火を入れる」
- プロジェクト開始直後からデプロイとテストを行うべし。つまり、動くスケルトンをつかってビルド・デプロイ・テストが機能することを確かめる。
- 動くスケルトン＝実際の機能をできる限り薄くスライスしたものの実装。ビルド・デプロイ・エンドツーエンドのテストを自動化できなければならない。
- 疑似本番環境にデプロイし、その上でテストするところまでやる。
  - そもそもデプロイは自動化すべきで、本番の前にデプロイスクリプトも徹底的に試すべきである
  - デプロイのタイミングこそが運用を学ぶタイミングである。DBセットアップにワークフローが必要…みたいな話も事前に知れる。
- 動くスケルトンの形を定める
  - アプリケーションの抽象的な構造を決める。詳細化は不要。ホワイトボードに数分でかけるもの
  - 動くスケルトンのポイントは、最初のテストを書くことでプロジェクトのコンテキストを書き出すこと。
- はじめにデプロイ・テストまでの流れを自動化することで、プロジェクト初期段階で課題が明らかになって良い
#### 第五章「テスト駆動のサイクルを保つ」
- 失敗する受け入れテストを書く→（失敗するユニットテストを書く・テストを通るようにする）　というサイクル
- 受け入れテストを書く際、アプリケーションドメインに由来する用語しか使わない（DB、Webサーバなど由来の用語は使わない）。システムが何を行うものなのかを理解することが重要
- 最もシンプルな正常ケースを書き始めるのが良い
- メソッドをテストするのではない。こういうテストは何をしているかはわかるが、なんのためテストしているかが不明。テスト対象オブジェクトが提供するフィーチャに集中スべき。
- テストの声を聞く…テストが難しいフィーチャがあったら「なぜ難しいのか？」を問うべき。そこに設計改善のヒントがある
#### 第六章「オブジェクト指向スタイル」
- コードを理解し続け、保守し続けるためには、構造化をするしかない。２つの原則
  - 関心事を分離する
  - 抽象度を高くする
- 単一責任原則 (オブジェクトが何をしているかをand/orを使って説明するのはNG)
- オブジェクトとピア（オブジェクトがやり取りする相手）との関係性のパターン
  - 依存：依存サービスがないのにオブジェクト生成できてはいけない
  - 通知：オブジェクトの最新情報を取得し続ける。イベントリスナーみたいなもの
  - 調整：オブジェクトの振る舞いを調整するもの。
- 組み合わせてオブジェクトのAPIは、複数の構成要素が存在することを隠蔽するべきである
- 各オブジェクトは、それを実行するシステムに関する知識を組み込むべきではない（コンテキストからの独立）
#### 第七章「オブジェクト指向設計を実現する」
- TDDはオブジェクト指向の実現に役立つ
  - テストから書くことでやりたいことを明確にできる
  - テストの内容を読みやすくする行為は、コンポーネントを適切に分解するのに役立つ
  - テストのためオブジェクトを生成するには、依存ピアを渡す必要がある。つまり、依存先を知る必要がある
- jMock2の紹介の例のコードのように「あるメッセージがactionCloseイベントが一回呼びだされる」というコミュニケーションを記述する
- 値（オブジェクトではない。状態を持たない不変のもの）には型をつけたくなる
  - 分解: オブジェクトが関心事を複数実装している場合。ヘルパー型を導入することができる（ref:p.138)
  - 発芽：ドメインの新しい概念に印をつけたい場合。プレースホルダー型を導入できる
  - 包括：特定の値のグループが一緒に使われている＝何らかの構造体が必要
- オブジェクトの型のつける時の容量もおおよそ↑と同じ
  - 分解：巨大オブジェクトを複数オブジェクトに分ける
  - 発芽：オブジェクトが必要とする新しいサービスを定義し、そのサービスを提供するオブジェクトを追加
  - 包括：関係する複数オブジェクトは、それを包括するオブジェクトに隠蔽する
- オブジェクト間の関係はインターフェイスで定義する。インターフェイスの数が増えても良いので、インターフェイスを狭めることを筆者は好む。
