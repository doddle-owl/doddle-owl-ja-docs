==========================
ユーザマニュアル
==========================

.. contents:: コンテンツ 
   :depth: 3

.. |MR3| replace:: MR\ :sup:`3` \

実装アーキテクチャ
=============================
:numref:`implementation_architecture` にDODDLE-OWLの実装アーキテクチャを示す．DODDLE-OWLは，GUI コンポーネントとしてJava Swing を用いて，Java 言語で実装した．DODDLE-OWL は，オントロジー選択モジュール，入力モジュール，オントロジー構築・洗練モジュール，視覚化モジュール，変換モジュールから構成される．実装上は，オントロジー構築およびオントロジー洗練は，同一パネル上で操作できるようにしている．

Web 上の既存オントロジーを獲得するために，オントロジー選択モジュールではSwoogleWeb サービスを利用している．入力モジュール，オントロジー構築・洗練モジュールでは，WordNet を参照するために `extJWNL(Extended Java WordNet Library)  <http://extjwnl.sourceforge.net/Java>`_ を利用している．入力モジュールでは，日本語の形態素解析および品詞同定を行うために，日本語形態素解析器 `lucene-gosen <https://github.com/lucene-gosen/lucene-gosen>`_ を用いている．英語の品詞同定を行うために `The Stanford Parser <https://nlp.stanford.edu/software/lex-parser.shtml>`_ を用いている．英語および日本語の複合語を抽出するために専門用語自動抽出システム言選 [Nakagawa03]_ を用いている．日本語の複合語抽出には，言選以外に日本語係り受け解析器 `CaboCha <http://taku910.github.io/cabocha/>`_ を用いることもできる． `Apache POI <http://poi.apache.org>`_ と `Apache PDFBox <https://pdfbox.apache.org>`_ を利用することにより，テキスト文書のみでなく，PDF，Microsoft Word, Excel, PowerPoint など様々な形式のファイルからテキストを抽出することができる．視覚化モジュールには |MR3| <http://mrcube.org>を用いている．変換モジュールでは，OWL形式のオントロジーのインポートおよびエクスポートを支援するために， `Apache Jena <http://jena.apache.org>`_ を用いている． 


.. _implementation_architecture:
.. figure:: figures/implementation-architecture-of-doddle-owl.svg
   :scale: 100 %
   :alt: DODDLE-OWLの実装アーキテクチャ
   :align: center

   DODDLE-OWLの実装アーキテクチャ

オントロジー選択モジュール
======================================

Swoogle を用いた既存オントロジーの獲得
----------------------------------------------------
オントロジー検索エンジンSwoogle は19 種類のREST 形式のWeb サービス（Swoogle　Web サービス）を提供している．ユーザはURL を用いてクエリーを作成し，RDF/XML形式の検索結果を得ることができる．:numref:`swoogle-web-service-io` に領域オントロジー構築支援に利用可能なSwoogle Web サービスとその入出力を示す．:numref:`swoogle-web-service-io` のSWT (Semantic Web Term) はクラスまたはプロパティを表す．SWD (Semantic Web Document) はRDF/XML，N-Triple，N3 形式で記述されたRDF 文書を表す．SWO (Semantic Web Ontology) はクラスおよびプロパティの定義の割合が8 割以上のSWD を表す．

.. list-table:: 領域オントロジー構築支援に利用可能なSwoogle Webサービスとその入出力
   :name: swoogle-web-service-io

   * - タイプ
     - Swoogle Webサービス
     - 入力
     - 出力
   * - 1
     - Search ontology
     - 検索キーワード
     - 検索キーワードに関連するSWO のリスト
   * - 3
     - Search terms
     - 検索キーワード    
     - 検索キーワードに関連するSWT のリスト
   * - 4
     - Digest semantic web document
     - SWD
     - SWD のSwoogle メタデータ
   * - 13
     - List documents using term
     - SWT
     - SWT を定義，参照，populate しているSWD のリスト
   * - 16
     - ist domain classes of a property
     - プロパティ
     - 入力したプロパティの定義域のリスト
   * - 17
     - List properties of a domain class
     - クラス
     - 入力したクラスを定義域とするプロパティのリスト
   * - 18
     - List range classes of a property
     - プロパティ
     - 入力したプロパティの値域のリスト
   * - 19
     - List properties of a range class
     - クラス
     - 入力したクラスを値域とするプロパティのリスト

:numref:`swoogle-web-service-type-and-condition` は，:numref:`ontology_ranking` で示した既存オントロジー獲得の手順1 から4 の各手順で利用するSwoogle Web サービスのタイプおよび実行条件を示す．:numref:`swoogle-web-service-type-and-condition` の手順は，:numref:`ontology_ranking` の手順と一致している．:numref:`swoogle-web-service-type-and-condition` の各手順で利用するSwoogle Web サービスのタイプは，:numref:`swoogle-web-service-io` のタイプの番号と一致している．また，計算時間を削減するために，各手順において実行条件を設定している．
 
.. list-table:: 既存オントロジー獲得の各手順で利用するSwoogle Web サービスのタイプおよび実行条件
  :name: swoogle-web-service-type-and-condition

  * - 手順
    - 各手順で利用するSwoogle Web サービスのタイプ
    - 実行条件
  * - 1
    - 3
    - 各入力語について，獲得するクラスおよびプロパティ数は， TermRank によりランク付けされた上位5 個までとする．
  * - 2
    - 17, 19
    - 手順1 で獲得したクラスをrdfs:domain またはrdfs:range プロパティの値として持つプロパティの獲得数は，各クラスごとに上位100 個までとする．
  * - 3
    - 16, 18
    - 手順1 および2 で獲得したプロパティの定義域および値域の獲得数は，各プロパティごとに上位100 個までとする．
  * - 4
    - 1, 4, 13
    - 各入力語について獲得するオントロジー数は，OntoRank でランク付けされた上位10 個までとする．


SPARQL テンプレートを用いたオントロジー要素抽出
---------------------------------------------------------------------
:numref:`sparql-template1` から :numref:`sparql-template5` にRDFS，DAML，OWL語彙におけるオントロジーの要素を抽出するためのSPARQLで記述したテンプレートを示す．:numref:`sparql-template3` の見出しと説明抽出テンプレートを直接SPARQL のクエリーとした場合，OWLオントロジー中のすべてのrdfs:labelおよびrdfs:comment プロパティの値を抽出してしまう．オントロジー選択モジュールでは?concept 変数の部分を取得したい概念（クラスまたはプロパティ）のURIに置換することにより，特定の概念の見出しおよび説明のみを抽出できるようにしている．他のテンプレートも同様にテンプレートを直接SPARQLのクエリーとして用いるのではなく，変数部分をオントロジー選択モジュールが適切なURIに置換したものを最終的なSPARQLのクエリーとしている．?concept, ?subConcept, ?class, ?property, ?label, ?description,?domain, ?range 変数を用いてトリプルのパターンを各オントロジーの要素を抽出するテンプレートに記述し，テンプレートをOWLオントロジーに対応づけることで，様々なクラス，プロパティ，構造により表現されたオントロジーの要素を抽出することが可能となる．

.. code-block:: sparql
   :caption: RDFS，DAML，OWL基本語彙におけるクラス抽出テンプレート
   :name: sparql-template1

     PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
     PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
     PREFIX owl: <http://www.w3.org/2002/07/owl#>
     PREFIX daml03: <http://www.daml.org/2001/03/daml+oil#>
     PREFIX daml10: <http://www.w3.org/2001/10/daml+oil#>

     SELECT ?class WHERE {
          {?class rdf:type rdfs:Class} UNION {?class rdf:type owl:Class} UNION
          {?class rdf:type owl:Restriction} UNION {?class rdf:type owl:DataRange} UNION
          {?class rdf:type daml03:Class} UNION {?class rdf:type daml03:Datatype} UNION
          {?class rdf:type daml03:Restriction} UNION  {?class rdf:type daml10:Class} UNION
          {?class rdf:type daml10:Datatype} UNION {?class rdf:type daml10:Restriction}
     }

.. code-block:: sparql
   :caption: RDFS，DAML，OWL基本語彙におけるプロパティ抽出テンプレート
   :name: sparql-template2

     PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
     PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
     PREFIX owl:  <http://www.w3.org/2002/07/owl#>
     PREFIX daml03: <http://www.daml.org/2001/03/daml+oil#>
     PREFIX daml10: <http://www.w3.org/2001/10/daml+oil#>

     SELECT ?property WHERE {
         {?property rdf:type rdf:Property} UNION {?property rdf:type owl:ObjectProperty} UNION
         {?property rdf:type owl:DatatypeProperty} UNION {?property rdf:type owl:AnnotationProperty} UNION
         {?property rdf:type owl:FunctionalProperty} UNION {?property rdf:type owl:InverseFunctionalProperty} UNION
         {?property rdf:type owl:SymmetricProperty} UNION {?property rdf:type owl:OntologyProperty} UNION
         {?property rdf:type owl:TransitiveProperty} UNION {?property rdf:type daml03:Property} UNION
         {?property rdf:type daml03:ObjectProperty} UNION {?property rdf:type daml03:DatatypeProperty} UNION
         {?property rdf:type daml03:TransitiveProperty} UNION {?property rdf:type daml03:DatatypeProperty} UNION
         {?property rdf:type daml03:UniqueProperty}  UNION {?property rdf:type daml10:Property} UNION
         {?property rdf:type daml10:ObjectProperty} UNION {?property rdf:type daml10:DatatypeProperty} UNION
         {?property rdf:type daml10:TransitiveProperty} UNION {?property rdf:type daml10:DatatypeProperty} UNION
         {?property rdf:type daml10:UniqueProperty}
     }


.. code-block:: sparql
   :caption: RDFS，DAML，OWL基本語彙における見出しおよび説明抽出テンプレート
   :name: sparql-template3

     PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
     PREFIX daml03: <http://www.daml.org/2001/03/daml+oil#>
     PREFIX daml10: <http://www.w3.org/2001/10/daml+oil#>

     SELECT ?label ?description WHERE {
          {?concept rdfs:label ?label} UNION {?concept rdfs:comment ?description} UNION
          {?concept daml03:label ?label} UNION {?concept daml03:comment ?description} UNION
          {?concept daml10:label ?label} UNION  {?concept daml10:comment ?description}
     }
 
.. code-block:: sparql
   :caption: RDFS，DAML，OWL基本語彙における階層関係抽出テンプレート
   :name: sparql-template4

     PREFIX  rdfs: <http://www.w3.org/2000/01/rdf-schema#>
     PREFIX daml03: <http://www.daml.org/2001/03/daml+oil#>
     PREFIX daml10: <http://www.w3.org/2001/10/daml+oil#>

     SELECT ?subConcept WHERE {
         {?subConcept rdfs:subClassOf ?concept} UNION {?subConcept rdfs:subPropertyOf ?concept} UNION
         {?subConcept daml03:subClassOf ?concept} UNION {?subConcept daml03:subPropertyOf ?concept} UNION
         {?subConcept daml10:subClassOf ?concept} UNION {?subConcept daml10:subPropertyOf ?concept}
     }

.. code-block:: sparql
   :caption: RDFS，DAML，OWL基本語彙におけるその他の関係抽出テンプレート
   :name: sparql-template5

     PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
     PREFIX daml03: <http://www.daml.org/2001/03/daml+oil#>
     PREFIX daml10: <http://www.w3.org/2001/10/daml+oil#>

     SELECT ?property ?domain ?range WHERE {
         {?property rdfs:domain ?domain} UNION  {?property rdfs:range ?range} UNION
         {?property daml03:domain ?domain} UNION {?property daml03:range ?range} UNION
         {?property daml10:domain ?domain} UNION {?property daml10:range ?range}
     }

汎用オントロジー選択パネル
-------------------------------------------
:numref:`ontology-selection-panel` に汎用オントロジー選択パネルを示す．:numref:`ontology-selection-panel` (1) に示す，3 種類の汎用オントロジー（EDR 一般辞書，EDR 専門辞書，WordNet）の中から参照オントロジーを選択する．チェックボックスにチェックをつけた汎用オントロジーを用いて，その後領域オントロジーにおける概念階層を構築する．複数の汎用オントロジーが選択可能な利点としては，領域によっては，一つの汎用オントロジーだけでは語彙を網羅しきれない場合があるため，複数の汎用オントロジーを組み合わせて利用できるようにしている．:numref:`ontology-selection-panel` (2)の名前空間テーブルは，名前空間URI とその名前空間接頭辞の対応関係を管理している．:numref:`ontology-selection-panel` (3) に接頭辞と名前空間を入力し，:numref:`ontology-selection-panel` (3) 右側の「追加」ボタンで追加することができる．


.. _ontology-selection-panel:
.. figure:: figures/ontology-selection-panel.png
   :scale: 80 %
   :alt: 汎用オントロジー選択パネル
   :align: center

   汎用オントロジー選択パネル


OWLオントロジー選択パネル
------------------------------------------
:numref:`owl-ontology-selection-panel` にOWL オントロジー選択パネルを示す．:numref:`owl-ontology-selection-panel` (1) の「追加（ファイル）」または「追加(URI)」ボタンにより，参照オントロジーとする既存OWLオントロジーを選択する．:numref:`owl-ontology-selection-panel` (3) には，:numref:`owl-ontology-selection-panel` (1) のオントロジーリスト中で選択したオントロジーのOWLメタデータが表示される．また，:numref:`owl-ontology-selection-panel` (2) において，OWLオントロジー中から抽出する要素を決定するためのSPARQL テンプレートを指定する．SPARQL テンプレートの種類として，SPARQL テンプレートを用いたオントロジー要素抽出で述べた5 種類が利用できる．

.. _owl-ontology-selection-panel:
.. figure:: figures/owl-ontology-selection-panel.png
   :scale: 80 %
   :alt: OWLオントロジー選択パネル
   :align: center

   OWLオントロジー選択パネル


入力文書選択パネル
=================================
:numref:`input-document-selection-panel` に入力文書選択パネルを示す．入力文書選択パネルでは，領域に関連する英語または日本語で記述された文書を選択する．入力文書選択パネルでは，Apache POIとApache PDFBoxを用いて，様々な形式（Word, Excel, PowerPoint, PDF など）のファイルからテキストデータを抽出できる．単語を抽出する際には，抽出する単語の品詞を指定できるようにしている．名詞，動詞，その他の品詞，複合語のいずれかを抽出したり，1 文字だけの領域オントロジー構築に不要となる語を除去することができる．以下に :numref:`input-document-selection-panel` の各部分について説明する．

.. _input-document-selection-panel:
.. figure:: figures/input-document-selection-panel.png
   :scale: 80 %
   :alt: 入力文書選択パネル
   :align: center

   入力文書選択パネル

#. 入力文書のリストを表示する．
#. 入力文書の記述言語（日本語または英語）の選択と入力文書の追加および削除を行う．
#. 1 文の区切り文字を設定する．
#. (1) の入力文書リストの中から選択された文書の内容を表示する．
#. 抽出する語の品詞，複合語を抽出するかどうか，1 文字の語を抽出するかどうかを選択する．
#. (1) の入力文書リストで選択された文書中から(5) で指定した条件の語を抽出する．

入力語選択パネル
=================================
入力語選択モジュールの実装として，入力文書ビューア，入力語情報テーブル，削除語情報テーブルを実装した．

入力文書ビューア
--------------------------
入力文書ビューアでは，入力文書の内容を見ながらユーザは入力語の選択を行うことができる．:numref:`input-document-viewer` に入力文書ビューアのスクリーンショットを示す．以下では，入力文書ビューアの各部分について説明する．

.. _input-document-viewer:
.. figure:: figures/input-document-viewer.png
   :scale: 80 %
   :alt: 入力文書ビューア
   :align: center

   入力文書ビューア

#. 入力文書リストを表示する．
#. (1) で選択した入力文書の内容を(3) に表示する際に，何行目から何行目までを表示するかを選択する．
#. (1) で選択した入力文書の内容を表示する．表示される行範囲は(2) で選択される．入力文書中のハイパーリンクが張られている語をクリックすることで，入力語か不要語かを選択することができる．青色リンクは入力語を，灰色リンクは不要語を表している．
#. (3) のハイパーリンクにマウスカーソルを合わせた際に，ハイパーリンクが張られている語の用語名，品詞，TF，IDF，TF-IDF，上位概念が表示される．
#. (1) で選択した入力文書の内容を分割して(3) に表示する際の分割行数を設定する．
#. 自動用語抽出により，抽出できなかった用語を手動で追加することができる．(3) において用語を範囲選択し，マウスを右クリックすることでも，同様に手動で用語を追加することができる．追加された用語は，(3) において青色のハイパーリンクが張られる．
#. (3) に表示される入力文書の内容にハイパーリンクを張る用語の種類（複合語，名詞，動詞，その他の品詞）を選択する．

入力語情報テーブル
---------------------------------
入力語情報テーブルでは，入力文書から自動抽出された語から入力語を選択することができる．:numref:`input-term-table` に入力語情報テーブルのスクリーンショットを示す．以下では，入力語情報テーブルの各部分について説明する．

.. _input-term-table:
.. figure:: figures/input-term-table.png
   :scale: 80 %
   :alt: 入力語情報テーブル
   :align: center

   入力語情報テーブル

#. ユーザが入力した用語で(3) に表示する用語情報リストを絞り込む．
#. ユーザが入力した品詞で(3) に表示する用語情報リストを絞り込む．
#. 入力文書から自動抽出された用語情報を表示する．用語情報には，用語名，品詞，TF，IDF，TF-IDF，上位概念があり，それぞれの観点からリストをソートすることができる．抽出された語が，あらかじめユーザが用意した参照オントロジー中の概念の下位概念の見出しに含まれる場合，その概念の見出しを上位概念に表示する．概念階層中の上位概念を設定しておくことで，抽出された語を「もの」「場所」「時間」などに分類して表示することができ，入力語選択を支援することができる．
#. (3) の中で選択された用語情報の用語の入力文書中の出現箇所を表示する．
#. 最終的にユーザが決定した入力語のリスト．テキストエリアになっているため，入力文書に出現しなかった入力語の追加をユーザは行うことができる．
#. 「入力語リストに追加」ボタンを押すと，(3) の中で選択された行の用語を(5) の入力語リストに追加する．「削除」ボタンを押すと，(3) の中で選択された用語情報の用語を「削除語テーブル」に移す．
#. (5) に入力された入力語を設定し，入力概念選択パネルに移る．「入力語彙をセット」ボタンを押した場合は，新規に入力語リストを入力概念選択パネルに設定する．「入力語彙を追加」ボタンを押した場合は，設定済みの入力語リストに新たに入力語を追加する．

削除語情報テーブル
------------------------------------
削除語情報テーブルには，入力語情報テーブルから削除された用語情報のリストが表示される．:numref:`removed-term-table` に削除語情報テーブルのスクリーンショットを示す．削除語情報テーブルの各部分は，入力語情報テーブルと同様である．異なる点は，「戻す」ボタンと「完全削除」ボタンである．「戻す」ボタンにより，誤って削除語情報テーブルに移動させてしまった用語情報を入力語情報テーブルに戻すことができる．「完全削除」ボタンにより，用語情報をリストから完全に削除することができる．


.. _removed-term-table:
.. figure:: figures/removed-term-table.png
   :scale: 80 %
   :alt: 削除語情報テーブル
   :align: center

   削除語情報テーブル

入力概念選択パネル
============================
入力概念選択モジュールの実装として，入力概念選択パネルを実装した．:numref:`input-concept-selection-panel` に入力概念選択パネルを示す．入力概念選択パネルでは，入力語と参照オントロジー中の概念との対応付けを行う．語には多義性があり，ある入力語を見出しとして持つ概念が複数存在する可能性がある．入力概念選択パネルでは，対象領域にとって最も適切な入力語に対応する概念を選択する際の支援を行う．以下に入力概念選択パネルの各部分の説明を示す．


.. _input-concept-selection-panel:
.. figure:: figures/input-concept-selection-panel.png
   :scale: 80 %
   :alt: 入力概念選択パネル
   :align: center

   入力概念選択パネル

(1) 用語リスト
	入力語彙の中で参照オントロジー中の概念見出しと完全照合または部分照合した用語のリストを表示する．
(2) 概念リスト
	(1) で選択された語を見出しとしてもつ参照オントロジー中の概念のリストを表示する．
(3) 概念情報
	(2) で選択された概念の見出しおよび説明を言語ごとに分類して表示する．
(4) 未定義語リスト
	参照オントロジー中の概念の見出しと照合しなかった入力語（未定義語）を表示する．
(5) 概念階層
	(2) で選択された概念の参照オントロジー中の概念階層を表示する．
(6) 入力文書
	(1) で選択された語の入力文書中の出現箇所を表示する．
(7) 階層構築オプション
	階層構築における条件を設定する．

用語リスト
-----------------------
:numref:`input-concept-selection-panel-term-list` は :numref:`input-concept-selection-panel` (1) 用語リストを拡大した図である．以下では，入力概念選択パネルの用語リストの各部分について説明する．


.. _input-concept-selection-panel-term-list:
.. figure:: figures/input-concept-selection-panel-term-list.png
   :scale: 80 %
   :alt: 入力概念選択パネル: 用語リスト
   :align: center

   入力概念選択パネル: 用語リスト


#. テキストフィールドに検索キーワードを入力し，検索ボタンを押すと(2) および(3)の完全照合語リストおよび部分照合語リストに検索キーワードを含む入力語のみが表示される．
#. 完全照合語リストを表示する．1 番目の括弧内には，入力語を見出しとする参照オントロジー中の概念の数が表示される．システムが自動的に追加した入力語は，2番目の括弧内に「自動追加」と表示される．
#. 部分照合語リストを表示する．1 番目の括弧内には，部分照合語を形態素解析し，各形態素を「+」記号で結合した結果が表示される．2 番目の括弧内には，参照オントロジー中の概念の見出しと照合した部分照合語内の語が表示される．3 番目の括弧内には，2 番目の括弧内に表示された語を見出しとする参照オントロジー中の概念の数が表示される．
#. 完全照合語リストに関する設定を行うことができる． 
    #. 「意味数」チェックボックスは，完全照合語リスト中の各語を見出しとする参照オントロジー中の概念の数を表示するかどうかを設定するオプションである．
    #. 「システムが追加した入力語」チェックボックスは，システムが自動的に追加した語かどうかを完全照合語リスト中の語に提示するかどうかを設定するオプションである．部分照合語の中で参照オントロジー中の概念と照合した語を，ユーザが入力語として追加していなかった場合に，システムはその語を自動的に完全照合語として完全照合語リストに追加する．例えば，「資格取得日」をユーザが入力語として選択した場合，「資格取得日」自体は参照オントロジー中の概念の見出しに存在しないため，部分照合語となる．「資格取得日」の「日」に対して部分照合したとする．ここで，ユーザが「日」を入力語として選択している場合には問題ない．しかし，「日」をユーザが入力語として選択していなかった場合には，「日」が自動的に完全照合語リストに追加される．システムが自動的に追加した語には，「（自動追加）」と表示される．
    #. 「入力概念選択結果を対応する部分照合語リストに適用」チェックボックスは，完全照合語の入力概念選択結果を，その完全照合語に照合した部分照合語リストの入力概念選択に反映させるかどうかを設定するためのオプションである．例えば，完全照合語「日」に対して入力概念選択を行った結果を，部分照合語リスト中の「資格取得日」や「研究日」などにも反映させるかどうかを設定することができる．
#. 部分照合語リストに関する設定を行うことができる．
    #. 「意味数」チェックボックスは(4) の完全照合語リストのオプションにおける「意味数」と同様である． 
    #. 「形態素リスト」チェックボックスは，部分照合語を形態素解析器で形態素に分割したときの分割のされ方を表示するか否かを設定するためのオプションである．このオプションを有効にした場合，例えば，「資格取得日」に対して，「（資格+取得+日）」が表示される．「+」記号は形態素の区切りをあらわす． 
    #. 「照合結果」チェックボックスは，部分照合語の形態素リストの中で，参照オントロジー中の概念と照合した形態素リストを表示するか否かを設定するオプションである．このオプションを有効にした場合，例えば，「資格取得日」は，「日」で照合しているため，「（日）」と表示される． 
    #. 「選択中の完全照合語に対応する複合語のみ表示」チェックボックスは，完全照合語リストで選択した語を照合語とする部分照合語のみを表示するか否かを設定するためのオプションである．このオプションを有効にした場合，例えば，完全照合語リスト中の「日」を選択した場合，「資格取得日」や「研究日」など「日」と照合した部分照合語のみが部分照合語リストに表示される．
#. 入力語の追加および削除を行うことができる．

参考文献
=====================
.. [Nakagawa03] 中川裕志，森辰則，湯本紘彰，“出現頻度と連接頻度に基づく専門用語抽出，” 自然言語処理，vol.10，no.1，pp.29–35，2003，http://gensen.dl.itc.u-tokyo.ac.jp/.

