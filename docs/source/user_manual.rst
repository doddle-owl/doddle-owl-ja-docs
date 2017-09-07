==========================
ユーザマニュアル
==========================

.. contents:: コンテンツ 
   :depth: 3

.. |MR3| replace:: MR\ :sup:`3` \

実装の概要
==============
領域オントロジー構築支援環境の設計を基に，DODDLE-OWL(a Domain Ontology rapiD DeveLopment Environment – OWL extension) を実装した．:numref:`implementation_architecture` にDODDLE-OWLの実装アーキテクチャを示す．DODDLE-OWLは，GUI コンポーネントとしてJava Swing を用いて，Java 言語で実装した．DODDLE-OWL は，オントロジー選択モジュール，入力モジュール，オントロジー構築・洗練モジュール，視覚化モジュール，変換モジュールから構成される．実装上は，オントロジー構築およびオントロジー洗練は，同一パネル上で操作できるようにしている．

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

参考文献
=====================
.. [Nakagawa03] 中川裕志，森辰則，湯本紘彰，“出現頻度と連接頻度に基づく専門用語抽出，” 自然言語処理，vol.10，no.1，pp.29–35，2003，http://gensen.dl.itc.u-tokyo.ac.jp/.

