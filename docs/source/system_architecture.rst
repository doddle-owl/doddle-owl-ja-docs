システム構成
==============

:numref:`system_flow` にDODDLE-OWLのシステム構成を示す．DODDLE-OWLは，オントロジー選択モジュール，入力モジュール，オントロジー構築モジュール，オントロジー洗練モジュール，視覚化モジュール，変換モジュールの主に六つのモジュールから構成される．

階層構築モジュールおよび階層洗練モジュールは階層関係構築支援のために，DODDLE-I [Yamaguchi99]_ で提案された．関係構築モジュールおよび関係洗練モジュールは，その他の関係構築支援のために，DODDLE-II [Kurematsu04]_ で提案された．オントロジー選択モジュール，入力モジュール，視覚化モジュール，変換モジュールがDODDLE-OWLで新たに提案するモジュールである．また，DODDLE-OWLでは，DODDLE-I およびDODDLE-II で提案された各モジュールを統合し，インタラクティブな領域オントロジー構築支援環境を実現している．

DODDLE-OWL では一つ以上の領域における専門文書があることを前提としている．また，ユーザは領域にとって重要な用語（入力語）を選択可能な知識をもっているものとする．


.. _system_flow:
.. figure:: figures/system_flow.png
   :scale: 80 %
   :alt: DODDLE-OWLのシステム構成
   :align: center

   DODDLE-OWLのシステム構成

参考文献
--------

.. [Yamaguchi99] 山口高平，槫松理樹，青木千鶴，関内律恵子，加賀山茂，吉野一，“計算機可読型辞書を利用した領域オントロジー構築支援環境，” 人工知能学会誌，vol.14，no.6，pp.1080–1087，1999.
.. [Kurematsu04] M. Kurematsu, T. Iwade, N. Nakaya, and T. Yamaguchi, “DODDLE II : A Domain Ontology Development Environment Using a MRD and Text Corpus,” IEICE transactions on information and systems, vol.87, no.4, pp.908-916, 2004.
