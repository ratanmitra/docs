リクエストとレスポンスオブジェクト
##################################

CakePHP2.0では新しくリクエストとレスポンスオブジェクトが追加されました。以前のバージョンではこれらのオブジェクトは配列で表現されており、
また関連するメソッドは :php:class:`RequestHandlerComponent`, :php:class:`Router`,
:php:class:`Dispatcher`, :php:class:`Controller` に分散していました。そのため、
リクエストにどのような情報が含まれているかを正確に表すオブジェクトは存在しませんでした。
バージョン2.0において :php:class:`CakeRequest` と :php:class:`CakeResponse` は上記の目的で使用されます。

.. index:: $this->request
.. _cake-request:

CakeRequest
###########

:php:class:`CakeRequest` はCakePHPで使われるデフォルトのリクエストオブジェクトです。
リクエストデータへの応答と対話が中心的な機能となります。リクエストごとに :php:class:`CakeRequest` は一つ作られ、
リクエストデータを使うアプリケーションの様々なレイヤーに参照が渡されます。
デフォルトの :php:class:`CakeRequest` は ``$this->request`` に設定され、コントローラ、ビュー、
ヘルパーの中で利用できます。またコントローラの参照を使うことでコンポーネントの中からもアクセスすることが出来ます。
:php:class:`CakeRequest` の役割は以下の通りです。:

* GET, POST, そしてFILES配列を慣れ親しんだデータ構造に変換する処理を行います。
* リクエストに関連する内省的環境を提供します。送信されたヘッダやクライアントのIPアドレス、サーバが実行されているサブドメイン/ドメインの情報などが含まれます。
* リクエストパラメータへのアクセス方法をインデックス付き配列とオブジェクトのプロパティの両方の形式で提供します。

リクエストパラメータへアクセスする
==================================

:php:class:`CakeRequest` はリクエストパラメータにアクセスするためにいくつかのインターフェイスを提供しています。
一つ目の方法は、添字付き配列です。二つ目の方法は ``$this->request->params`` を経由する方法です。
三つ目はオブジェクトのプロパティとしてアクセスする方法です。::

    $this->request->controller;
    $this->request['controller'];
    $this->request->params['controller'];

上記はすべて同じ値へアクセスすることになります。パラメータへアクセスする方法が複数あることで既存のアプリケーションの移植が楽になるかもしれません。
すべての :ref:`route-elements` はこのインターフェイスを通してアクセスされます。

:ref:`route-elements` に加えて :ref:`passed-arguments` や :ref:`named-parameters` へのアクセスがしばしば必要になります。
これらは両方ともリクエストオブジェクトと同様に利用可能です。::

    // 渡された引数
    $this->request->pass;
    $this->request['pass'];
    $this->request->params['pass'];

    // 名前付きパラメータ
    $this->request->named;
    $this->request['named'];
    $this->request->params['named'];

すべての渡された引数と名前付きパラメータにアクセスする方法が提供されています。
この中にはCakePHPの内部で使っている重要で役に立つパラメータが存在し、また、リクエストパラメータの中ですべて見つけられます。

* ``plugin`` リクエストをハンドリングしているプラグインで、プラグインが無い場合はnullです。
* ``controller`` 現在のリクエストをハンドリングしているコントローラです。
* ``action`` 現在のリクエストをハンドリングしているアクションです。
* ``prefix`` 現在のアクションのプレフィックスです。詳しくは、 :ref:`prefix-routing` 見て下さい。
* ``bare`` リクエストが :php:meth:`~Controller::requestAction()` から始まり bare オプションを含んでいたとき定義されます。生のリクエストは描画されたレイアウトを持ちません。
* ``requested`` アクションが :php:meth:`~Controller::requestAction()` から始まったとき定義され true が設定されます。

クエリ文字列パラメータにアクセスする
====================================

クエリ文字列パラメータは :php:attr:`CakeRequest::$query` を使って読み出すことができます。::

    // urlは /posts/index?page=1&sort=title
    $this->request->query['page'];

    // 配列を経由してアクセスできます
    // Note: 後方互換アクセッサです。将来のバージョンで非推奨になります。
    $this->request['url']['page'];

:php:attr:`~CakeRequest::$query` プロパティに直接アクセスするか、
エラーが発生しない方法でURLクエリ配列を読むために :php:meth:`CakeRequest::query()` を使うことができます。
キーが存在しない場合、 ``null`` が返ります。::

    $foo = $this->request->query('value_that_does_not_exist');
    // $foo === null

POSTデータにアクセスする
========================

すべてのPOSTデータは :php:attr:`CakeRequest::$data` を使ってアクセスされます。フォームデータが ``data``
接頭辞を含んでいる場合、接頭辞は取り除かれるでしょう。例えば::

    // name属性が'data[Post][title]'だった入力は次のようにアクセスします。
    $this->request->data['Post']['title'];

:php:attr:`~CakeRequest::$data` プロパティに直接アクセスするか、エラーが発生しない方法でdata配列を読むために
:php:meth:`CakeRequest::data()` を使うことができます。キーが存在しない場合、 ``null`` が返ります。::

    $foo = $this->request->data('Value.that.does.not.exist');
    // $foo == null

PUTまたはPOSTデータにアクセスする
=================================

.. versionadded:: 2.2

RESTサービスを構築しているとき ``PUT`` と ``DELETE`` リクエストのデータを受け付けることがよくあります。
2.2において ``application/x-www-form-urlencoded`` リクエストボディのデータは ``PUT``
と ``DELETE`` リクエストでは自動的に構文解析され ``$this->data`` に設定されます。
もしJSONやXMLデータを受け付けている場合、どうやってリクエストボディにアクセスすればいいのかについては以下の説明を見て下さい。

XMLまたはJSONデータにアクセスする
=================================

:doc:`/development/rest` を採用しているアプリケーションではURLエンコードされていないpost形式でデータを交換することがしばしばあります。
:php:meth:`CakeRequest::input()` を使っているどんな形式であっても入力データを読み込むことができます。
デコード関数が提供されることでデシリアライズされたコンテンツを受け取ることができます。::

    // PUT/POSTアクションで投稿されたデータをJSON形式にエンコードで取得する
    $data = $this->request->input('json_decode');

``json_decode`` の'as array'パラメータやXMLをDOMDocumentオブジェクトに変換したい時のように、
デシリアライズメソッドの中には呼び出し時に追加パラメータが必要なものがあるので :php:meth:`CakeRequest::input()`
は追加パラメータを渡せるようになっています。::

    // PUT/POSTアクションで投稿されたデータをXmlエンコードで取得する
    $data = $this->request->input('Xml::build', array('return' => 'domdocument'));

パス情報にアクセスする
======================

:php:class:`CakeRequest` はまたアプリケーションのパスについての役立つ情報を提供しています。 :php:attr:`CakeRequest::$base`
と :php:attr:`CakeRequest::$webroot` はURLの生成や、アプリケーションがサブディレクトリにいるのかどうかの決定に役立ちます。

.. _check-the-request:

リクエストを調べる
==================

さまざまなリクエストの状態を検出するために以前は :php:class:`RequestHandlerComponent` を使う必要がありました。
これらのメソッドは :php:class:`CakeRequest` に移動され後方互換を保ちつつ新しいインターフェイスが提供されています。
使い方は以下の通りです。::

    $this->request->is('post');
    $this->request->isPost(); // 非推奨

どちらのメソッド呼び出しも同じ値を返します。 :php:class:`RequestHandlerComponent` にてそのメソッドが利用できるようになったとき、
これらのメソッドは廃止され最終リリース前に削除されるかもしれません。また、新しい種類の検出器(detector)を作成するために
:php:meth:`CakeRequest::addDetector()` を使うことでリクエスト検出器を簡単に拡張することができます。4種類の異なる検出器を作成できます。:

* 環境変数の比較 - 環境変数の比較、 :php:func:`env()` から取得された値と既知の値を比較します。
  環境変数は提供された値に対して等価性をチェックされます。
* パターン値の比較 - パターン値の比較では :php:func:`env()` から取得された値と正規表現を比較します。
* オプションベースの比較 - オプションベースの比較では正規表現を作成するためにオプションのリストを使います。
  既に定義済みのオプション検出器を追加するための呼び出しはオプションをマージするでしょう。
* コールバック検出器 - コールバック検出器はチェックをハンドリングするために'callback'タイプを提供します。
  コールバックはパラメータとしてだけリクエストオブジェクトを受け取ります。

いくつかの例を示します。::

    // environment detectorを追加する
    $this->request->addDetector(
        'post',
        array('env' => 'REQUEST_METHOD', 'value' => 'POST')
    );

    // pattern value detectorを追加する
    $this->request->addDetector(
        'iphone',
        array('env' => 'HTTP_USER_AGENT', 'pattern' => '/iPhone/i')
    );

    // option detectorを追加する
    $this->request->addDetector('internalIp', array(
        'env' => 'CLIENT_IP',
        'options' => array('192.168.0.101', '192.168.0.100')
    ));

    // callback detectorを追加する。匿名関数か通常のコールバックが指定可能。
    $this->request->addDetector(
        'awesome',
        array('callback' => function ($request) {
            return isset($request->awesome);
        })
    );

:php:class:`CakeRequest` には :php:meth:`CakeRequest::domain()`, :php:meth:`CakeRequest::subdomains()` や
:php:meth:`CakeRequest::host()` といったサブドメインを扱うのに役立つメソッドがあるため、少し楽ができます。

利用可能な組み込みの検出器は以下の通りです。:

* ``is('get')`` 現在のリクエストがGETかどうかを調べます。
* ``is('put')`` 現在のリクエストがPUTかどうかを調べます。
* ``is('post')`` 現在のリクエストがPOSTかどうかを調べます。
* ``is('delete')`` 現在のリクエストがDELETEかどうかを調べます。
* ``is('head')`` 現在のリクエストがHEADかどうかを調べます。
* ``is('options')`` 現在のリクエストがOPTIONSかどうかを調べます。
* ``is('ajax')`` 現在のリクエストが X-Requested-With = XMLHttpRequestに由来するものかどうかを調べます。
* ``is('ssl')`` リクエストがSSL経由かどうかを調べます。
* ``is('flash')`` リクエストにFlashのUser-Agentがあるかどうかを調べます。
* ``is('mobile')`` リクエストがモバイルエージェントの共通リストに由来しているかどうかを調べます。

CakeRequest と RequestHandlerComponent
======================================

:php:class:`CakeRequest` が提供している多くの機能は以前、 :php:class:`RequestHandlerComponent` の中にあったので、
CakePHP2.0にどのように収まるのかを理解するために再考する必要がありました。2.0において :php:class:`RequestHandlerComponent`
はパトロン(sugar daddy)として振るいます。 :php:class:`CakeRequest` が提供するユーティリティの最上位に砂糖のレイヤーを提供しています。
レイアウトの切り替えやコンテンツタイプやajaxを基にしたビューといった砂糖は :php:class:`RequestHandlerComponent`
の領域です。ユーティリティと砂糖のクラスを分離することで欲しいもの、必要なものの取捨選択が簡単になるでしょう。

リクエストの他要素と対話する
============================

:php:class:`CakeRequest` はリクエストに関する様々なことを内省(introspect)するために使えます。
また、検出器によって様々なプロパティやメソッドからの他の情報を発見できます。

* ``$this->request->webroot`` はwebrootディレクトリを含みます。
* ``$this->request->base`` は基本パスを含みます。
* ``$this->request->here`` は現在のリクエストへの完全なアドレスを含みます。
* ``$this->request->query`` はクエリ文字列パラメータを含みます。

CakeRequest API
===============

.. php:class:: CakeRequest

    CakeRequestはリクエストパラメータのハンドリングをカプセル化し、内省化します。

.. php:method:: domain($tldLength = 1)

    アプリケーションが実行されているドメイン名を返します。

.. php:method:: subdomains($tldLength = 1)

    アプリケーションが実行されているサブドメインを配列で返します。

.. php:method:: host()

    アプリケーションのホスト名を返します。

.. php:method:: method()

    リクエストのHTTPメソッドを返します。

.. php:method:: onlyAllow($methods)

   許可されたHTTPメソッドを設定します。もしマッチしなかった場合、MethodNotAllowedException を投げます。
   405レスポンスには、通過できるメソッドを持つ ``Allow`` ヘッダが含まれます。

   .. versionadded:: 2.3

   .. deprecated:: 2.5
        代わりに :php:meth:`CakeRequest::allowMethod()` を使用してください。

.. php:method:: allowMethod($methods)

   許可されたHTTPメソッドを設定します。もしマッチしなかった場合、MethodNotAllowedException を投げます。
   405レスポンスには、通過できるメソッドを持つ ``Allow`` ヘッダが含まれます。

   .. versionadded:: 2.5

.. php:method:: referer($local = false)

    リクエストのリファラを返します。

.. php:method:: clientIp($safe = true)

    現在アクセスしているクライアントのIPアドレスを返します。

.. php:method:: header($name)

    リクエストで使われている ``HTTP_*`` ヘッダにアクセスできます。::

        $this->request->header('User-Agent');

    この例の場合、リクエストで使われているユーザエージェントが返るでしょう。

.. php:method:: input($callback, [$options])

    リクエストとデコード関数を通して渡されたinputデータを取得します。
    リクエストの本文を XML や JSON でやり取りするときに便利です。
    デコード関数の追加パラメータはinput()の引数として渡す事ができます。::

        $this->request->input('json_decode');

.. php:method:: data($key)

    リクエストデータへドット記法によるアクセスを提供します。リクエストデータの読み込みと変更が可能です。また次のように連鎖的に呼び出す事をできます。::

        // リクエストデータを修正し、フォームフィールドを生成できます。
        $this->request->data('Post.title', 'New post')
            ->data('Comment.1.author', 'Mark');

        // データの取得もできます。
        $value = $this->request->data('Post.title');

.. php:method:: is($check)

    リクエストがある基準に適合するかどうかを調べます。 :php:meth:`CakeRequest::addDetector()` で追加された追加のルールと同様に組み込みの検出ルールを使えます。

.. php:method:: addDetector($name, $callback)

    :php:meth:`CakeRequest::is()` と一緒に使われる検出器を追加します。詳しくは、 :ref:`check-the-request` を参照して下さい。

.. php:method:: accepts($type = null)

    クライアントがどのコンテンツタイプを受理するかを調べます。また、特定のコンテンツタイプが受理されるかどうかを調べます。

    すべてのタイプを取得::

        $this->request->accepts();

    あるタイプについて調べる::

        $this->request->accepts('application/json');

.. php:staticmethod:: acceptLanguage($language = null)

    クライアントによって受理されるすべての言語を取得します。また、特定の言語が受理されるかどうかを調べます。

    受理される言語のリストを取得::

        CakeRequest::acceptLanguage();

    特定の言語が受理されるかどうかを調べる::

        CakeRequest::acceptLanguage('es-es');

.. php:method:: param($name)

   ``$request->params`` の値を安全に読みます。パラメータの値を使う前に ``isset()`` や ``empty()`` を呼ぶ必要がなくなります。

   .. versionadded:: 2.4

.. php:attr:: data

    POSTデータの配列です。 :php:meth:`CakeRequest::data()` を使うとエラーが発生しないようにしつつプロパティを読み込むことができます。

.. php:attr:: query

    クエリ文字列パラメータの配列です。

.. php:attr:: params

    ルート要素とリクエストパラメータの配列です。

.. php:attr:: here

    現在のリクエストのuriを返します。

.. php:attr:: base

    アプリケーションへのベースパスです。アプリケーションがサブディレクトリに配置されていない限り、普通は ``/`` です。

.. php:attr:: webroot

    現在のwebrootてす。

.. index:: $this->response

CakeResponse
############

:php:class:`CakeResponse` はCakePHPのデフォルトのレスポンスクラスです。いくつかの機能とHTTPレスポンスの生成をカプセル化します。
また送信予定のヘッダを調べるためにモックやスタブとしてテストの手助けをします。:php:class:`CakeRequest` のように
:php:class:`CakeResponse` は :php:class:`Controller` や :php:class:`RequestHandlerComponent`
や :php:class:`Dispatcher` に以前からある多くのメソッドを強化します。古いメソッドは廃止され
:php:class:`CakeResponse` の使用が推奨されます。

:php:class:`CakeResponse` は次のような共通のレスポンスをラップするためのインターフェイスを提供します。:

* リダイレクトのためにヘッダを送ること。
* コンテンツタイプヘッダを送ること。
* ヘッダを送ること。
* レスポンスボディを送ること。

レスポンスクラスを変更する
==========================

CakePHPはデフォルトで  :php:class:`CakeResponse` を使います。 :php:class:`CakeResponse` は柔軟で透過的にクラスが使われます。
しかし、このクラスをアプリケーション固有のクラスに置き換える必要がある場合、 :php:class:`CakeResponse`
をオーバーライドして独自のクラスで置き換えることができます。
それは ``app/webroot/index.php`` 内の :php:class:`CakeResponse` を置き換えることで実現できます。

この置き換えによってすべてのコントローラが :php:class:`CakeResponse` の代わりに
``CustomResponse`` を使えるようになります。またコントローラの中で  ``$this->response``
と設定することでレスポンスインスタンスを置き換えることができます。レスポンスオブジェクトのオーバーライドは
:php:meth:`~CakeResponse::header()` とやりとりするメソッドをスタブ化しやすくするので、テストで使いやすいです。
詳しくは :ref:`cakeresponse-testing` を参照して下さい。

コンテンツタイプを扱う
======================

:php:meth:`CakeResponse::type()` を使うことでアプリケーションレスポンスのContent-Typeを制御することができます。
もし :php:class:`CakeResponse` に組み込まれていないコンテンツタイプを扱う必要がある場合、以下のように
:php:meth:`CakeResponse::type()` を使って設定することが出来ます。::

    // vCard タイプを追加する
    $this->response->type(array('vcf' => 'text/v-card'));

    // レスポンスのContent-Typeをcardに設定する
    $this->response->type('vcf');

大抵の場合、追加のコンテンツタイプはコントローラの :php:meth:`~Controller::beforeFilter()` コールバックの中で設定したいと思うので、
:php:class:`RequestHandlerComponent` が提供するビューの自動切り替え機能を活用できます。

.. _cake-response-file:

ファイルを送る
==============

リクエストに対するレスポンスとしてファイルを送りたいときがあります。
バージョン 2.3 より前は、 :php:class:`MediaView` を使うことができました。
2.3 以降から :php:class:`MediaView` は非推奨になり :php:meth:`CakeResponse::file()` を使って
ファイルを送信します。::

    public function sendFile($id) {
        $file = $this->Attachment->getFile($id);
        this->response->file($file['path']);
        // レスポンスオブジェクトを返すとコントローラがビューの描画を中止します
        return $this->response;
    }

上記の例のようにメソッドにファイルのパスを渡す必要があります。
CakePHP は、:php:attr:`CakeResponse::$_mimeTypes` に登録された、よく知られるファイルタイプであれば
正しいコンテンツタイプヘッダを送ります。 :php:meth:`CakeResponse::file()` を呼ぶ前に
:php:meth:`CakeResponse::type()` メソッドを使って、新しいタイプを追加できます。

もし、あなたが望むなら、 オプションを明記することによって、ブラウザ上に表示する代わりに
ファイルをダウンロードさせることができます。::

    $this->response->file(
        $file['path'],
        array('download' => true, 'name' => 'foo')
    );

文字列をファイルとして送る
==========================

動的に生成された pdf や ics のようにディスクに存在しないファイルを返すことができます。::

    public function sendIcs() {
        $icsString = $this->Calendar->generateIcs();
        $this->response->body($icsString);
        $this->response->type('ics');

        // 任意のダウンロードファイル名を指定できます
        $this->response->download('filename_for_download.ics');

        // レスポンスオブジェクトを返すとコントローラがビューの描画を中止します
        return $this->response;
    }

ヘッダを設定する
================

ヘッダの設定は :php:meth:`CakeResponse::header()` で行われます。このメソッドは少し違ったパラメータ設定と一緒に呼ばれます。::

    // ヘッダを一つ設定する
    $this->response->header('Location', 'http://example.com');

    // 複数ヘッダを設定する
    $this->response->header(array(
        'Location' => 'http://example.com',
        'X-Extra' => 'My header'
    ));
    $this->response->header(array(
        'WWW-Authenticate: Negotiate',
        'Content-type: application/pdf'
    ));

同じヘッダを複数回設定すると、普通のheader呼び出しと同じように、以前の値を上書きしていしまいます。
:php:meth:`CakeResponse::header()` が呼び出されなければヘッダは送られません。これらのヘッダはレスポンスが実際に送られるまでバッファリングされます。

.. versionadded:: 2.4

:php:meth:`CakeResponse::location()` を使うと直接 リダイレクトヘッダの設定や取得ができます。

ブラウザキャッシュと対話する
============================

時々、コントローラアクションの結果をキャッシュしないようにブラウザに強制する必要がでてきます。
:php:meth:`CakeResponse::disableCache()` はそういった目的で使われます。::

    public function index() {
        // do something.
        $this->response->disableCache();
    }

.. warning::

    Internet Explorerにファイルを送ろうとしている場合、SSLドメインからのダウンロードと一緒にdisableCache()を使うことをエラーにすることができます。

また、:php:meth:`CakeResponse::cache()` を使ってクライアントにレスポンスをキャッシュして欲しいことを伝えられます。::

    public function index() {
        //do something
        $this->response->cache('-1 minute', '+5 days');
    }

上記の例では、訪問者の体感スピード向上のため、クライアントにレスポンス結果を5日間キャッシュするように伝えています。
:php:meth:`CakeResponse::cache()` は、 第一引数に ``Last-Modified`` ヘッダの値を設定します。
第二引数に ``Expires`` ヘッダと ``max-age`` ディレクティブの値を設定します。
Cache-Control の ``public`` ディレクティブも設定されます。

.. _cake-response-caching:

HTTPキャッシュをチューニングする
================================

アプリケーションの速度を改善するための簡単で最善の方法の一つはHTTPキャッシュを使う事です。
このキャッシュモデルの元では、modified time, response entity tagなどいくつかのヘッダを設定することでレスポンスのキャッシュコピーを使うべきかどうかをクライアントが決定できるように助ける事が求められます。

キャッシュやデータが変更されたときに無効化(更新)するロジックのコードを持つのではなく、
HTTPは二つのモデル、expirationとvalidationを使います。これらは大抵の場合、自身でキャッシュを管理するよりかなり単純です。

:php:meth:`CakeResponse::cache()` と独立して、HTTPキャッシュヘッダをチューニングするための様々なメソッドが使えます。
この点に関して、ブラウザやリバースプロキシのキャッシュよりも有利だと言えます。

Cache Controlヘッダ
-------------------

.. versionadded:: 2.1

キャッシュ制御ヘッダはexpirationモデルの元で使われ、複数の指示を含んでいます。ブラウザやプロキシがどのようにキャッシュされたコンテンツを扱うのかをその指示で変更することができます。
Cache-Control ヘッダは以下の通りです。::

    Cache-Control: private, max-age=3600, must-revalidate

:php:class:`CakeResponse` のいくつかのユーティリティメソッドを用いることで、最終的に有効な
``Cache-Control`` ヘッダを生成します。 一つ目は、:php:meth:`CakeResponse::sharable()` メソッドです。
このメソッドは異なるユーザやクライアントの間で共有出来ることを考慮されたレスポンスかどうかを示します。
このメソッドは実際には、このヘッダが `public` または `private` のどちらなのかを制御しています。
privateにレスポンスを設定することは、レスポンスのすべてまたはその一部が特定のユーザ用であることを示しています。
共有キャッシュのメリットを活かすためにはコントロールディレクティブをpublicに設定する必要があります。

このメソッドの二番目のパラメータはキャッシュの `max-age` を指定するために使われます。
このパラメータはレスポンスが古いと見なされる秒数を表しています。::

    public function view() {
        ...
        // Cache-Control を3600秒の間、publicとして設定
        $this->response->sharable(true, 3600);
    }

    public function my_data() {
        ...
        // Cache-Control を3600秒の間、privateとして設定
        $this->response->sharable(false, 3600);
    }

:php:class:`CakeResponse` は ``Cache-Control`` ヘッダの中で各コンポーネントを設定するための分割されたメソッドを公開しています。

Expirationヘッダ
----------------

.. versionadded:: 2.1

``Expires`` ヘッダに、レスポンスが古いと見なされる日時を設定できます。
このヘッダは :php:meth:`CakeResponse::expires()` メソッドを使って設定されます。::

    public function view() {
        $this->response->expires('+5 days');
    }

またこのメソッドは、:php:class:`DateTime` インスタンスや :php:class:`DateTime` クラスによって構文解析可能な文字列を受け付けます。

Etagヘッダ
----------

.. versionadded:: 2.1

HTTPにおけるキャッシュの検証はコンテンツが定期的に変化するような場合によく使われ、
キャッシュが古いと見なせる場合にのみレスポンスコンテンツが生成されることをアプリケーションに求めます。
このモデルのもとでは、クライアントはページを直接使う代わりにキャッシュの中に保存し続け、アプリケーションに毎回リソースが変更されたかどうかを尋ねます。
これはイメージや他のアセットといった静的なリソースに対して使われる場合が多いです。

:php:meth:`~CakeResponse::etab()` メソッド(entity tagと呼ばれる)は要求されたリソースを識別するための一意な文字列です。大抵の場合はファイルのチェックサムのようなもので、
リソースが一致するかどうかを調べるためにキャッシュはチェックサムを比較するでしょう。

実際にこのヘッダを使うメリットを得るためには、手動で :php:meth:`CakeResponse::checkNotModified()`
メソッドを呼び出すかコントローラに :php:class:`RequestHandlerComponent` を読み込まなければなりません。::

    public function index() {
        $articles = $this->Article->find('all');
        $this->response->etag($this->Article->generateHash($articles));
        if ($this->response->checkNotModified($this->request)) {
            return $this->response;
        }
        ...
    }

Last Modifiedヘッダ
-------------------

.. versionadded:: 2.1

HTTPキャッシュのvalidationモデルのもとでは、リソースが最後に変更された日時を示すために ``Last-Modified`` ヘッダを設定することができます。
このヘッダを設定するとCakePHPがキャッシュしているクライアントにレスポンスが変更されたのかどうかを返答する手助けとなります。

実際にこのヘッダを使うメリットを得るためには、 :php:meth:`CakeResponse::checkNotModified()`
メソッドを呼び出すかコントローラに :php:class:`RequestHandlerComponent` を読み込まなければなりません。::

    public function view() {
        $article = $this->Article->find('first');
        $this->response->modified($article['Article']['modified']);
        if ($this->response->checkNotModified($this->request)) {
            return $this->response;
        }
        ...
    }

Varyヘッダ
----------

時には同じURLで異なるコンテンツを提供したいと思うかもしれません。これは多国語対応ページがある場合やブラウザごとに異なるHTMLを返すようなケースでしばしばおこります。
そのような状況では ``Vary`` ヘッダを使えます。::

        $this->response->vary('User-Agent');
        $this->response->vary('Accept-Encoding', 'User-Agent');
        $this->response->vary('Accept-Language');

.. _cakeresponse-testing:

CakeResponseとテスト
====================

コントローラとコンポーネントのテストが簡単に実施できた時、 :php:class:`CakeResponse` を使っていて良かったと思うかもしれません。
いくつものオブジェクトを横断して使われるメソッドの代わりに、コントローラとコンポーネントが :php:class:`CakeResponse`
に委譲しているのをまねる(mock)オブジェクトを準備するだけでよくなります。このことで'単体'テストを作りやすくなり、コントローラのテスト実施が簡単になります。::

    public function testSomething() {
        $this->controller->response = $this->getMock('CakeResponse');
        $this->controller->response->expects($this->once())->method('header');
        // ...
    }

さらに、CLIからヘッダ設定を試みた時に起こる'ヘッダ送信'エラーを避けるためにモックを使うことができるので、コマンドラインからより簡単にテストを実行できます。

CakeResponse API
================

.. php:class:: CakeResponse

    CakeResponseはクライアントへ送信するレスポンスと対話するために役立つメソッドをたくさん提供しています。

.. php:method:: header($header = null, $value = null)

    レスポンスと一緒に送られる一つまたは複数のヘッダを直接設定できます。

.. php:method:: location($url = null)

    レスポンスと一緒に送られるリダイレクトヘッダを直接設定できます。::

        // Set the redirect location
        $this->response->location('http://example.com');

        // Get the current redirect location header
        $location = $this->response->location();

    .. versionadded:: 2.4
 
.. php:method:: charset($charset = null)

    レスポンスの中で使われる文字コードの種類を設定します。

.. php:method:: type($contentType = null)

    レスポンスのコンテンツタイプを設定します。既知のコンテンツタイプの別名かコンテンツタイプの正式名称を使えます。

.. php:method:: cache($since, $time = '+1 day')

    レスポンスにキャッシュヘッダを設定することが出来ます。

.. php:method:: disableCache()

    レスポンスにクライアントのキャッシュを無効にするためのヘッダを設定します。

.. php:method:: sharable($public = null, $time = null)

    ``Cache-Control`` ヘッダに ``public`` か ``private`` を設定し、任意で、リソースの ``max-age`` ディレクティブを設定します。

    .. versionadded:: 2.1

.. php:method:: expires($time = null)

    ``Expires`` ヘッダに特定の日付を設定することができます。

    .. versionadded:: 2.1

.. php:method:: etag($tag = null, $weak = false)

    レスポンスリソースを一意に識別するために ``Etag`` ヘッダを設定します。

    .. versionadded:: 2.1

.. php:method:: modified($time = null)

    ``Last-Modified`` ヘッダに特定の日時を正しいフォーマットで設定します。

    .. versionadded:: 2.1

.. php:method:: checkNotModified(CakeRequest $request)

    リクエストオブジェクトとレスポンスのキャッシュヘッダを比較し、まだキャッシュが有効かどうかを決定します。
    もしまだ有効な場合、レスポンスのコンテンツは削除され ``304 Not Modified`` ヘッダが送られます。

    .. versionadded:: 2.1

.. php:method:: compress()

    レスポンスのgzip圧縮を使用開始します。

.. php:method:: download($filename)

    添付ファイルとしてレスポンスを送り、ファイル名を設定できます。

.. php:method:: statusCode($code = null)

    レスポンスのステータスコードを設定できます。

.. php:method:: body($content = null)

    レスポンスのコンテンツボディを設定します。

.. php:method:: send()

    レスポンスの作成が完了した後に、send()を呼び出すことでボディと同様に設定されているすべてのヘッダが送られます。
    各リクエストの最後に :php:class:`Dispatcher` によって自動的に行われます。

.. php:method:: file($path, $options = array())

    表示もしくはダウンロードするファイルの ``Content-Disposition`` ヘッダを設定できます。

    .. versionadded:: 2.3

.. meta::
    :title lang=ja: Request and Response objects
    :keywords lang=ja: request controller,request parameters,array indices,purpose index,response objects,domain information,request object,request data,interrogating,params,previous versions,introspection,dispatcher,rout,data structures,arrays,ip address,migration,indexes,cakephp
