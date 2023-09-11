# Chap. 10 テスティング
## 10.1 テストを記述する理由
### プロジェクトにおけるテスティングの必要性
1. 自信が増す・・・容易に変更できるコードとなる。
1. 後方互換性の確保・・・自動テストならば、常に最新版のライブラリでテストできる。
1. コストの削減・・・リリース後のバグ修正は開発中に修正するより10-25倍のコストがかかる。自動テストを開発しておけば、早期にバグが発見でき、コストが減る。
1. ユースケースのコード化・・・ユースケース用としてテストを活用できる。
1. 法規制コンプライアンスの確保・・・法規制の要件について常に適合できる。

## 10.2 APIテストの種類
### 一般的なソフトウェアテストの手法
1. ホワイトボックステスト・・・ソースコードについて理解したうえで開発したテスト。通常、プログラミング言語で記述される。
1. ブラックボックステスト・・・実装についての知識なしで、製品仕様に基づいて開発したテスト。テストはエンドユーザーアプリケーションを手動で使用して行われる。API使用にのみ基づいてAPIをテストするコードの記述にも使える。
1. グレーボックステスト・・・ホワイトボックステストとブラックボックステストを組み合わせたテスト。ブラックボックステストが、実装の詳細について理解したうえで行われる。

## 10.2.1 単体テスト
- 単体テストとは、個別のメソッドやクラスなど、最小単位のソースコードを検証するテスト。
- 目的は、テスト可能な最小単位にAPIを切り分け、その機能が単独で正しく行われることを検証する。
- 単体テストは、実装の知識をもとに開発者が記述するため、ホワイトボックステストに相当。

### 例
- 文字列を倍制度の浮動小数点数に変換する関数のテスト
    ```c++
    bool StringToDouble(const std::string &str, double &result);
    ```
- この関数は文字列パラメータを受け取って、変換が成功したかどうかを示すブール型を返す。成功した場合、倍精度浮動小数点数の値が`result`参照パラメータに書き込まれる。
    ```c++
    void Test_StringToDouble()
    {
        double value;

        Assert(StringToDouble("1", value), "+ve test");
        AssertEqual(value, 1.0, "'1' == 1.0");

        Assert(StringToDouble("-1", value), "-ve test");
        AssertEqual(value, -1.0, "'-1' == -1.0");

        Assert(StringToDouble("0.0", value), "zero");
        AssertEqual(value, 0.0, "'0.0' == 0.0");

        Assert(StringToDouble("-0", value), "minus zero");
        AssertEqual(value, -0.0, "'-0' == -0.0");
        AssertEqual(value, 0.0, "'-0' == 0.0");

        Assert(StringToDouble("3.14159265", value), "pi");
        AssertEqual(value, 3.14159265, "pi value");

        Assert(StringToDouble("2e10", value), "large scientific");
        AssertEqual(value, 2e10, "");

        Assert(StringToDouble("+4.3e-10", value), "small scientific");
        AssertEqual(value, 4.3e-10, "");

        AssertFalse(StringToDouble("",value), "empty");
        AssertFalse(StringToDouble(" ",value), "white space");
        AssertFalse(StringToDouble("+-1",value), "+");
        AssertFalse(StringToDouble("1.1.0",value), "multiple points");
        AssertFalse(StringToDouble("text",value), "not a number");

        std::cout << "SUCCESS!" << std::endl;

    }

    ```

### 単体テストの二つの手法
1. フィクスチャ設定・・・単体テストではテストを実行する前に、一貫した不変の環境に初期設定することは一般的。具体的には、各テストに関連した`setUp()`関数を実行し、テストの設定ステップを実際のテスト操作から切り離して行う。`tearDown()`関数は、テストが完了した時点でこの環境をクリーンナップするのに使われる。この手法を使うと、同じフィクスチャ設定で多くのテストで利用できる。
1. スタブ/モックオブジェクト・・・後述

## 10.2.2 統合テスト
- 統合テストとは、単体テストとは対照的に、コンポーネント同士のやり取りを検証する。理想を言えば、単体テストで検証済みのコンポーネントを使用することが望ましい。
- 目的は、APIのすべてのコンポーネントが一緒に機能し、一貫性があり、ユーザが必要とするタスクが行われることを確実にすること。
- 統合テストは、APIの仕様に基づいて開発されることが多く、内部実装の詳細についての知識は必要ない。つまり、クライアントの視点で記述するテストであるため、ブラックボックステストに相当する。

## 10.2.3 パフォーマンステスト
- API機能が最低速度やメモリ要件を満たすかどうかを検証する。

## 10.3 優れたテストの記述
### 10.3.1 優れたテストの特徴
1. スピード・・・失敗によるフィードバックを素早く得るため、高速に実行できる。
1. 安定性・・・繰り返しが可能で依存関係のない一貫性のあるテストにすること。
1. 移植可能性・・・様々なプラットフォームに実装するAPIの場合、これらのプラットフォームすべてでテストする必要がある。プラットフォームによって変更すべきテストコードに、有働小数点数の同値比較がある。丸め誤差、アーキテクチャの違い、コンパイラの違いによってプラットフォームによる計算結果に違いが生じる可能性がある。
1. 高いコーディング規約・・・APIと同じレベルのコーディング規約を守ること。
1. 再現性のある失敗・・・テストが失敗した場合、その失敗を簡単に再現できるようにすること。

### 10.3.2 何をテストすべきか
1. 条件テスト・・・コード内の可能性のあるすべてのパスをテストする。
1. 等価クラス・・・等価クラスとは、期待される振る舞いを持つテスト入力の集合。例えば、0-65535を受け付ける関数ならば、-10, 100, 100000は三つの等価クラス。
1. 境界条件・・・大半のエラーは期待値の境界付近で発生するもの。例えば、要素nの連結リストに要素を挿入するルーチンをテストする場合、0,1,n-1,nの位置で挿入テストをする。
1. パラメータテスト・・・すべてのパラメータで行う。
1. 返し値アサート・・・様々な入力パラメータに対して、正しい結果を返しているかどうかを検証するテスト
1. getter/setterペア・・・setterの前にgetterを呼び出して適切なデフォルト値が返るか、setterの後にgetterを呼び出して適切な値が返るか
    ```c++
    AssertEqual(obj.GetValue(), 0, "test default");
    obj.SetValue(42);
    AssertEqual(obj.GetValue(), 42, "test set then get");
    ```
1. 動作の順序・・・同じテストを実行するのに動作の順序を変えて行うと、実行順序における想定動作や相互独立性のないふるまいを発見できる可能性がある。
1. 回帰テスト・・・可能な限り、旧バージョンのAPIと後方互換性を保つことが重要であるため、後方互換性のテストは必要。
1. ネガティブテスト・・・エラー条件を生成または強制し、予期せぬ状況でコードがどのように反応するのかを検証するテスト。
1. バッファオーバーラン・・・バッファオーバーランまたはバッファオーバーフローは、割り当てられたバッファサイズを超えてメモリに書き込まれると発生する。
1. メモリオーナーシップ・・・C++のプログラムでは、クラッシュを引き起こす大きな原因にメモリエラーがある。動的にメモリを割り当てたメモリを返すAPI呼び出しの場合、APIがメモリを所有するのか、あるいはクライアントがメモリの解放に責任を持つのかをドキュメントで明確にする必要があり、仕様通りに動くかを検証する。
1. Null入力・・・ポインタパラメータを受け取れる関数はすべてテストして、Nullポインタを渡されても問題なくふるまうことを確認する。

## 10.3.3 テストの目標を定める
- 多くの場合、APIの可能なコードパスをすべてテストするのは現実的でないので、全体の機能からそのサブセットをテストするかを決断することになる。その際の指標として
1. APIのメインのユースケースまたはワークフローを検証するテストにフォーカスする
1. 複数の機能のカバーするテストまたは最大のコードカバレッジが可能なテストにフォーカスする
1. もっとも複雑でハイリスクなコードにフォーカスする
1. うまく定義されていない設計部分にフォーカスする
1. 最大のパフォーマンスを持つ機能またはセキュリティ問題のある機能にフォーカスする
1. クライアントに最悪のインパクトを与える可能性のあるテスティング問題にフォーカスする
1. 開発サイクルの初期段階で完了できる初期の機能テストにフォーカスする

## 10.3.4 QAチームとの協力
マイクロソフトはQA技術者を大きく二つに分けている
1. ソフトウェアテストエンジニア(STE)・・・プログラム経験は少なく、コンピュータサイエンスの知識が必要ない場合もある。STEは基本的に手動でブラックボックステストを行う。
1. テストにおけるソフトウェアデザインエンジニア(SDET)・・・コードを記述でき、ホワイトボックステスト、ツールの記述、自動テストの生成を行う技能を持つ技術者。

## 10.4 テスト可能なコードの記述
### 10.4.1 テスト駆動型開発(TDD)
- 小さなステップで徐々に変更していくことが重要で、短いテストを記述し、そのテストをパスさせるのに十分なコードのみを記述する。
- 例
    ```c++
    void TestNoRatings()
    {
        MovieRating *nemo = new MovieRating("Finding Nemo");
        AssertEqual(nemo->GetRatingCount(), 0, "no ratings");
    }
    ```
- テストをパスさせるための最もシンプルなコードを記述する。
    ```c++
    class MovieRating
    {
    public:
        MovieRating(const std::string &name){}
        int GetRatingCount() const {return 0;}
    };
    ```
- 次のテストコードを追加
    ```c++
    void TestAverageRating()
    {
        MovieRating *nemo = new MovieRating("Finding Nemo");
        nemo->AddRating(4.0f);
        nemo->AddRating(5.0f);
        AssertEqual(nemo->GetAverageRating(), 4.5f, "nemo avg rating");
    }
    ```
- 最小限のコードを書く。
    ```c++
    class MovieRating
    {
    public:
        MovieRating(const std::string &name){}
        int GetRatingCount() const {return 0;}
        void AddRating(float r){}
        float GetAverageRating() const {return 4.5f;}
    };
    ```
- もう一つテストを記述すると実装がもっと一般的にならざるを得ない。
    ```c++
    void TestAverageRatingAndCount()
    {
        MovieRating *cars = new MovieRating("Cars");
        cars->AddRating(3.0f);
        cars->AddRating(4.0f);
        cars->AddRating(5.0f);
        AssertEqual(cars->GetRatingCount(), 3, "three rating");
        AssertEqual(cars->GetAverageRating(), 4.0f, "cars avg rating");
    }
    ```
- 以下のように実装できる。
    ```c++
    class MovieRating
    {
    public:
        MovieRating(const std::string &name):
            mNumRatings(0),
            mRatingSum(0.0f)
        {}
        int GetRatingCount() const
        {
            return mNumRatings;
        }
        void AddRating(float r)
        {
            mRatingSum += r;
            mNumRatings++;
        }
        float GetAverageRating() const 
        {
            return mRatingSum / mNumRatings;
        }
    private:
        int mNumRatings;
        float mRatingSum;
    };
    ```
## 10.4.2 スタブオブジェクトとモックオブジェクト
- 単体テストの安定性や障害からの回復性を高めるテクニックとして、システム内のオブジェクトの代理を務めるテストオブジェクトを使う方法がある。テスト用として軽量で制御可能なオブジェクトを使って予測不可能なリソース(ファイルシステム、外部データベース、ネットワークなど)を置き換える。
### 代理オブジェクトの種類として
1. フェイクオブジェクト・・・機能的な動作を持つが、テスト用にシンプルな実装を使うオブジェクト。ローカルディスクとの通信をシミュレーションするメモリ内ファイルシステムなど。
1. スタブオブジェクト・・・あらかじめ準備したリソースを返すオブジェクト。
1. モックオブジェクト・・・事前に振る舞いをプログラムした測定機能をつけたオブジェクト。
### スタブ/モックオブジェクトの例
- 以下の三つのクラスを使ってゲームをも出リングすることを考える。
1. `Card`(トランプカード一枚)・・・1枚のカードを表す。別のカードの値に対して自分の値を比較する機能を持つ。
1. `Deck`(トランプのカード1組)・・・トランプ一組を格納する。カードを切ったり配ったりする。
1. `CardGame`(カードゲーム)・・・ゲームロジックを制御する。ゲームをプレイし、ゲームの勝者を返す。

- `Deck`オブジェクトはランダムなトランプのカードを返す。テスト用に、定義した順序でトランプのカードを返すスタブ`Deck`を作成できる。`StudDeck`は`Deck`クラスから継承するため、仮想にする必要がある。
    ```c++
    class Deck
    {
    public:
        Deck();
        virtual ~Deck();
        virtual void Shuffle();
        virtual int RemainingCards();
        virtual Card DealCard();
    };
    ```
- `StubDeck`クラスはこの`Deck`クラスから継承し、`Shuffle()`メソッドをオーバーライドして何もさせないようにできる。カードの順序を乱数にしたくないため。しかし、これでは、`StucDeck`クラスのカード順が固定されてしまう。
- `Deck`基本クラスにプロテクトした`AddCard()`メソッドを追加し、`StubDeck`クラスでパブリックとして開示する
    ```c++
    #include "cardgame.h"

    void TestCardGame()
    {
        StubDeck deck;
        deck.Addcard("9C");
        deck.AddCard("2H");
        ...
        CardGame game(deck);
        game.Play();

        AssetEqual(game.GetWinner(), CardGame::PLAYER_ONE);
    }
    ```
- モックオブジェクトとスタブオブジェクトの一番の違いは、モックが動作の検証を行うこと。
- モックオブジェクトとは、あるオブジェクトのすべての関数呼び出しを記録する判定機能を備え、関数が呼び出された回数、関数に渡されたパラメータ、複数の関数が呼び出された順序といった動作を検証する。
- モックテストフレームワークを使ってこの作業を自動化するとよい。
- 例 : Googleのモックフレームワーク
    ```c++
    #include "cardgame.h"
    #include <gmock/gmock.h>
    #include <gtest/gtest.h>

    using namespace testing;

    class MockDeck : public Deck
    {
    public:
        MOCK_METHOD0(Shuffle, void());
        MOCK_METHOD0(RemainingCards, int());
        MOCK_METHOD0(DealCard, Card());
    };
    ```
- テスト例
    ```c++
    TEST(CardGame, Test1)
    {
        MockDeck deck;

        EXPECT_CALL(deck, Shuffle())
            .Times(AtLeast(1));
        EXPECT_CALL(deck, DealCard())
            .Times(52)
            .WillOnce(Return(Card("JS")))
            .WillOnce(Return(Card("2H")))
            .WillOnce(Return(Card("9C")))
        ...
        ;
        CardGame game(deck);
        game.Play();

        ASSERT_EQ(game.GetWinner(), CardGame::PLAYER_ONE);
    }
    ```
- `Shuffle()`メソッドが最低一回呼び出され、2つ目では`DealCard()`メソッドが52回呼び出されて、最初の呼び出しでは`Card("JS")`、次の呼び出しでは`Card("2H")`を返し、、、と続く。
- `AddCard()`メソッドが不必要なことに注意。

