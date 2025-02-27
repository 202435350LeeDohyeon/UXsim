# UXsim: Python製のマクロ・メソ交通流シミュレータ

[![PyPi](https://img.shields.io/pypi/v/uxsim.svg)](https://pypi.python.org/pypi/uxsim)
[![arXiv](https://img.shields.io/badge/arXiv-2309.17114-b31b1b.svg)](http://dx.doi.org/10.48550/arXiv.2309.17114)
[(English readme is here)](https://github.com/toruseo/UXsim/blob/main/README.md)

*UXsim*はPython製のオープンソース・フリーなマクロ・メソ交通流シミュレータです．
これは，都市規模のような大局的な自動車交通とその渋滞を再現する交通シミュレーションであり，交通工学分野で標準的なモデルにより道路ネットワークの動的交通流を計算するものです．
UXsimは単純，軽量，柔軟であるため，研究・教育上の目的に適することを意図していますが，それ以外の目的にも自由に使用可能です．

- [Jupyter Notebookデモ](https://github.com/toruseo/UXsim/blob/main/demos_and_examples/demo_notebook_01.ipynb)
- [技術資料](https://toruseo.jp/UXsim/docs/index.html)
- [arXivプレプリント](https://arxiv.org/abs/2309.17114)
- [交通流理論・シミュレーションの専門書](https://www.coronasha.co.jp/np/isbn/9784339052794/)


## 主な機能・特徴

- 標準的なネットワーク交通流モデルの，単純かつ容易に使えるPython実装
- ネットワーク構造と時間帯別OD需要が与えられているときに，動的なネットワーク交通流を計算（動的交通量配分）．具体的な交通モデルは以下の通り：
	- Newellの単純追従モデル（Xモデル）
	- Lagrange版Incremental Node Model
	- Dynamic User Optimum型経路選択モデル（慣性付き）
- 信号交差点，流入制御，経路誘導，混雑課金などの交通マネジメントの組み込み
- 計算結果の各種分析（トリップ完了数，総旅行時間，遅れ時間など）と，そのpandas.DataFrameやCSVへのエクスポート
- 計算結果の可視化（時空間図，MFD，ネットワーク状況アニメーションなど）
- 純Pythonであることを活かした高度なカスタマイズ性
	- Python製の他のフレームワークとも直接連携可能．例：PyTorchを用いた深層強化学習による交通制御

## 主なファイル構成

- `uxsim`ディレクトリ: UXsimパッケージ
	- `uxsim/uxsim.py`: UXsim本体のコード
	- `uxsim/utils.py`: 関連コード
 	- `uxsim/utils`ディレクトリ: 関連ファイル
- `demos_and_examples`ディレクトリ: チュートリアルや使用例
- `dat`ディレクトリ: サンプルシナリオファイル

## インストール

### pipを使用

最も簡単な方法は、PyPIからpipを使用してインストールすることです。

```
pip install uxsim
```

<details>
<summary>他の方法（クリックして表示）</summary>
	
### カスタム設定でpipを使用

GitHubバージョンをインストールするには、`pip`も使用できます：

```
pip install -U -e git+https://github.com/toruseo/uxsim@main#egg=uxsim
```

または、このリポジトリの他の（開発中の）ブランチや自分のフォークでも：

```
pip install -U -e git+https://github.com/YOUR_FORK/uxsim@YOUR_BRANCH#egg=uxsim
```

	
### 手動インストール

このGithubリポジトリから`uxsim`ディレクトリをダウンロードするか、[最新リリース](https://github.com/toruseo/UXsim/releases/latest/download/uxsim.zip)からダウンロードして、以下のようにローカルディレクトリに配置します：
```
your_project_directory/
├── uxsim/ # uxsimディレクトリ
│ ├── utils/ # UXsimのユーティリティファイル
│ ├── uxsim.py # UXsimのメインコード。必要に応じてカスタマイズ可能
│ ├── utils.py # UXsimのユーティリティ関数
│ └── ... # その他元からあったファイル
├── your_simulation_code.py # 自作コード（必要なら）
├── your_simulation_notebook.ipynb # 自作Jupyterノートブック（必要なら）
├── ... # 必要に応じて他のファイルも置ける
```
この方法で、UXsimを自分の好みに合わせて柔軟にカスタマイズできます。

</details>

## 使用法

各自のPythonコード上で以下のようにインポート
```python
from uxsim import *
```
し，その後に自分のシミュレーションシナリオを定義します．


<details>
<summary>単純な例（クリックして表示）</summary>
Y字型の合流ネットワークのシミュレーションシナリオは以下の通り：
	
```python
from uxsim import *

# シミュレーション本体の定義
# 単位は全て秒とメートル
W = World(
    name="",    # シナリオ名
    deltan=5,   # 車両集計単位
    tmax=1200,  # シミュレーション時間
    print_mode=1, save_mode=0, show_mode=0,    # オプション
    random_seed=0    # ランダムシード
)

# Define the scenario
W.addNode("orig1", 0, 0) # ノードの作成
W.addNode("orig2", 0, 2)
W.addNode("merge", 1, 1)
W.addNode("dest", 2, 1)
W.addLink("link1", "orig1", "merge", length=1000, free_flow_speed=20, jam_density=0.2, merge_priority=0.5) # リンクの作成
W.addLink("link2", "orig2", "merge", length=1000, free_flow_speed=20, jam_density=0.2, merge_priority=2)
W.addLink("link3", "merge", "dest", length=1000, free_flow_speed=20, jam_density=0.2)
W.adddemand("orig1", "dest", 0, 1000, 0.4) # 交通需要の作成
W.adddemand("orig2", "dest", 500, 1000, 0.6)

# シミュレーション実行
W.exec_simulation()

# 結果表示
W.analyzer.print_simple_stats()

# ネットワーク交通状態のスナップショット可視化
W.analyzer.network(0, detailed=1, network_font_size=0)
W.analyzer.network(500, detailed=1, network_font_size=0)
W.analyzer.network(1000, detailed=1, network_font_size=0)
```

結果として以下のような文字列をターミナルに出力し，`out`ディレクトリにその下のような画像を出力します：
```
simulation setting:
 scenario name:
 simulation duration:    1200 s
 number of vehicles:     700 veh
 total road length:      3000 m
 time discret. width:    5 s
 platoon size:           5 veh
 number of timesteps:    240
 number of platoons:     140
 number of links:        3
 number of nodes:        4
 setup time:             0.00 s
simulating...
      time| # of vehicles| ave speed| computation time
       0 s|        0 vehs|   0.0 m/s|     0.00 s
     600 s|      100 vehs|  17.5 m/s|     0.03 s
    1195 s|       25 vehs|  20.0 m/s|     0.05 s
 simulation finished
results:
 average speed:  13.8 m/s
 number of completed trips:      675 / 700
 average travel time of trips:   142.7 s
 average delay of trips:         42.7 s
 delay ratio:                    0.299
```

<p float="left">
<img src="https://github.com/toruseo/UXsim/blob/images/simple_example_network1_1000.png" width="400"/>
</p>

</details>
 
[Jupyter Notebookデモ](https://github.com/toruseo/UXsim/blob/main/demos_and_examples/demo_notebook_01.ipynb)に基本的な使用法と機能をまとめています．
さらなる詳細は`demos_and_examples`ディレクトリ内の[使用例](https://github.com/toruseo/UXsim/tree/main/demos_and_examples)や，[UXsim技術資料](https://toruseo.jp/UXsim/docs/index.html)を確認してください．

## 計算例

### 大規模ネットワーク

10km x 10kmのグリッドネットワーク内を2時間で約6万台が通行する様子．計算時間は通常のデスクトップPCで約30秒．

リンク交通状況（リンクの線が太いと車両台数が多く，色が暗いと速度が遅い）と一部車両の軌跡：
<p float="left">
<img src="https://github.com/toruseo/UXsim/blob/images/gridnetwork_macro.gif" width="400"/>
<img src="https://github.com/toruseo/UXsim/blob/images/gridnetwork_fancy.gif" width="400"/>
</p>

上記ネットワークのある回廊上の車両軌跡図：

<img src="https://github.com/toruseo/UXsim/blob/images/tsd_traj_links_grid.png" width="600">

### 深層強化学習（AI）による信号制御

信号制御を[PyTorch](https://pytorch.org/)の深層強化学習によって効率化する例です．
下の左図は単純な固定時間の信号の場合で，交通需要がネットワーク容量を上回りグリッドロックが生じています．
右図は深層強化学習を適用した場合で，需要レベルが同じであるのに関わらず効率的な交通が実現しています．
このコードは[Jupyter Notebook](https://github.com/toruseo/UXsim/blob/main/demos_and_examples/demo_notebook_03en_pytorch.ipynb)にもまとめてあります.

<p float="left">
<img src="https://github.com/toruseo/UXsim/blob/images/anim_network1_0.22_nocontrol.gif" width="400"/>
<img src="https://github.com/toruseo/UXsim/blob/images/anim_network1_0.22_DQL.gif" width="400"/>
</p>

## 使用条件・ライセンス

本コードはMIT Licenseのもとで公開しています．
出典を明記すれば自由に使用できます．

使用された成果を発表される際は，参考文献として

- 瀬尾亨. マクロ交通流シミュレーション：数学的基礎理論とPythonによる実装. コロナ社, 2023
- Seo, T. UXsim: An open source macroscopic and mesoscopic traffic simulator in Python-a technical overview. arXiv preprint arXiv: 2309.17114, 2023

を引用してください．

## 謝辞

UXsimは交通流理論に関する様々な学術的成果に基づいています．この分野を進展させてきた皆様に感謝いたします．

## 関連リンク

- [瀬尾亨（作者）](https://toruseo.jp/)
- [瀬尾による関連シミュレータまとめサイト](https://toruseo.jp/uxsim/)
- 書籍『[マクロ交通流シミュレーション：数学的基礎理論とPythonによる実装](https://www.coronasha.co.jp/np/isbn/9784339052794/)』（著者：[瀬尾亨](https://toruseo.jp/)，出版社：[コロナ社](https://www.coronasha.co.jp/)）：UXsimは本書に含まれる交通流シミュレータ*UroborosX*を大幅に拡張したものです．理論と実装の詳細については本書を参照してください．
- [東京工業大学 瀬尾研究室](http://seo.cv.ens.titech.ac.jp/)
- [Webブラウザ上で動くインタラクティブ交通流シミュレータ](http://seo.cv.ens.titech.ac.jp/traffic-flow-demo/bottleneck_jp.html)：このシミュレータが用いているリンク交通流モデルと同じものをインタラクティブに動かして，交通流とそのシミュレーションの基礎を学べます．
