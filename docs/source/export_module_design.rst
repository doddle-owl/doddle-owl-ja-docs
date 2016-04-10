
変換モジュールの設計
====================

DODDLE-OWLによって構築される領域オントロジーは，階層関係とその他の関係から構成される．クラスのis-a階層は，OWLが提供するowl:Classクラスおよびrdfs:subClassOfプロパティにより定義する．クラスのhas-a階層は，owl:Classクラスおよびdoddle:partOfプロパティにより定義する．プロパティのis-a 階層は，owl:ObjectProperty クラスおよびrdfs:subPropertyOfプロパティにより定義する．プロパティのhas-a階層は，owl:ObjectPropertyクラスおよびdoddle:partOf プロパティにより定義する．その他の関係は，概念対の間の関係をOWLにおけるプロパティ，概念対をプロパティの定義域および値域としてとらえ，OWLが提供するowl:ObjectProperty クラス，rdfs:domain およびrdfs:range プロパティにより定義する．

:numref:`translation_module` の上部は，概念関係の定義の例として，「act」クラスの下位クラスとして「aim」と「behavior」クラスが定義された状態を，OWL形式に変換する方法を示している．:numref:`translation_module` の下部は，その他の関係の定義の例として，「time」と「offer」クラスの間に「attribute」プロパティという関係がある状態を，OWL形式に変換する方法を示している．

また，DODDLE-OWL では概念の見出しをrdfs:label プロパティ，概念の説明をrdfs:comment プロパティ，概念の表示見出しをskos:prefLabel プロパティを用いて定義している．概念の表示見出しは，概念に複数の見出しが定義されている場合に，概念階層を表示する際に優先的に表示する見出しのことである．


.. _translation_module:
.. figure:: figures/translation_module.png
   :scale: 80 %
   :alt: 変換モジュールによる領域オントロジーのOWL形式への変換例
   :align: center

   変換モジュールによる領域オントロジーのOWL形式への変換例
