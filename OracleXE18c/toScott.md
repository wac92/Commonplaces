# OracleDatabase環境構築(とりあえず編)
## はじめに
-   OracleDatabaseを私用パソコンで学習しようと思ったものの
    情報が古い・難しすぎるなどで、ハマった部分があるので、備忘録として記しておきます。

## 対象
- スキーマとかアーキテクチャとか今はわからない。
- とりあえず、SELECT文とか書いて結果を得られれば満足。

## 目標
- Oracle Database 18c Express Editionのインストール
- サンプルスキーマ(SCOTT)を作成する。
- 作成したスキーマを使い、ハンズオンで学習する。

## 注意
- セットアップファイルは約2GBあります。
- インストールすると約4GBになります。
- 起動中はメモリを約300MB～500MB食います。

    - これらの条件が厳しいようであれば、OracleLiveSQLという、オンライン上で演習できる公式サイトもあります。
    [Live SQL](https://www.oracle.com/technetwork/jp/database/application-development/livesql/index.html)
    - メリット
        + ローカルに環境構築をする必要がない。
        + 他人が作成した練習用のスキーマが数多くある。

    - デメリット
        + Oracle社のアカウント登録が必要。
        + ログインとスキーマの復元を毎回行う必要があるため面倒。
        + 設定ファイル(リスナーとか)を直接見ることができない(?)。
        + 全て英語
    - 個人的には、ローカルで構築するほうが快適でのちのち学習も捗ると思います。

## 手順
1. セットアップファイルの準備
    1. 以下のサイトにアクセスします。
        + [Oracle Database Express Edition | Oracle 日本](https://www.oracle.com/jp/database/technologies/appdev/xe.html)
    2. 「Oracle Database XEをダウンロード」をクリックします。
    3. OSの選択画面が出ます。Windowsであれば、「Oracle Database 18c Express Edition for Windows x64」のリンクテキストをクリックします。
    4. 利用規約の確認ポップアップが出てくるので、チェックボックスにチェックして、ダウンロードを開始します。
    5. ダウンロードしたzipファイルを解凍します。

2. インストール
    1. 解凍して現れたフォルダに「setup.exe」という実行ファイルがあるので起動します。
    2. ウィザードに沿ってインストールをしていきます。利用規約のラジオボタンと、データベースパスワードの入力する以外はデフォルトのままで大丈夫です。
        - インストールには10分～20分かかります。    

3.  サンプルスキーマを作成する。
    1. utlsampl.sqlを編集する 
    - このファイルでサンプルスキーマの作成とデータの挿入がまるっとできます。テキストエディタで開いてください。
        + デフォルトのパス：「C:\app\(ユーザ名)\product\18.0.0\dbhomeXE\rdbms\admin\utlsampl.sql」
    - ただ、18cではそのまま動かそうとすると、うまく行かないようです。このファイルの以下の部分を編集してください。
        + 45行目

            ```
            CONNECT SCOTT/tiger
            ```
            こちらを
            ```
            CONNECT SCOTT/tiger@XEPDB1
            ```
            としてください。

        また、
        + 30行目あたり

            ``` 
            SET TERMOUT OFF 
            SET ECHO OFF
            ```

        + 最終行
            ```
            EXIT
            ```
        これらも削除しておくと、エラーが出ているとき特定しやすいです。


    2. tnsnames.oraを編集する
    - 上の```CONNECT SCOTT/tiger@XEPDB1```という文を成功させるために接続識別子~~とやら~~を解決する必要があります。こちらもテキストエディタで開いてください。
        
        デフォルトのパス「C:\app\\(ユーザ名)\product\18.0.0\dbhomeXE\network\admin\tnsnames.ora」
        
        - ファイルの中にこんなブロックがあると思います。
        ``` XE =
        (DESCRIPTION =
            (ADDRESS = (PROTOCOL = TCP)(HOST = *なんとか*)(PORT = 1521))
            (CONNECT_DATA =
            (SERVER = DEDICATED)
            (SERVICE_NAME = XE)
            )
        )
        ``` 
        - まるまるコピーして
        ``` XEPDB1 =
        (DESCRIPTION =
            (ADDRESS = (PROTOCOL = TCP)(HOST = *なんとか*)(PORT = 1521))
            (CONNECT_DATA =
            (SERVER = DEDICATED)
            (SERVICE_NAME = XEPDB1)
            )
        )
        ``` 

        ~~細けぇことはいいんだよ！~~
    3. SQL*Plus上でutlsampl.sqlを実行する
    - SQL*Plusとは、コマンドラインのSQL実行環境です。SQLDeveloperというGUI環境もありますがこちらは割愛します。
        1. ログイン
        - SQL*Plusを起動するとユーザ名の入力を求められます。
            ```
            / as sysdba
            ```
        これでとりあえず最強の管理者としてログインできます。

        2. SQLの実行
        - ログインが成功し、「SQL> 」という画面が出てきたら
            ```
            @?/rdbms/admin/utlsampl.sql
            ```
            と入力し、エンターを押します。
        - これにより、
            + SCOTTというユーザ
            + EMP表、DEPT表、SALGRADE表、BONUS表(空っぽ)
            が作成されるはずです。
4. あとは、色々いじってみる
- 例えばこんなこと
    とりあえずSELECTしてみる
    データを挿入してみる
    EMP表とDEPT表を結合してみる
    EMP表とSALGRADE表で非等価結合してみる
    BONUS表に列を追加してEMP表を参照する制約を追加してみる
    とか

## おわりに
- SCOTTスキーマは本当に単純なもので、Oracle公式ドキュメントでも
    > Oracleでは、ドキュメントおよびトレーニングの様々なサンプルで、単純なデータベース・スキーマSCOTTをその2つの重要な表EMPおよびDEPTとともに長い間使用していました。これらの表は、Oracle Databaseおよびその他のOracle製品の基本的な機能を表すには不十分です。
    [サンプル・スキーマの概要](https://docs.oracle.com/cd/E82638_01/comsc/introduction-to-sample-schemas.html)
    
    とあり、現在は「HRスキーマ」というものが、サンプルスキーマとして使われているようです。
- こちらに関しても時期を見て導入してみたいと思います。

## 参考サイト
- 
