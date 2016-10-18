#第二週レポート課題

##課題内容
'''  
Ruby のプロファイラで Cbench のボトルネックを解析しよう。

以下に挙げた Ruby のプロファイラのどれかを使い、Cbench や Trema のボトルネック部分を発見し遅い理由を解説してください。  
'''

##解答
以下に,実際にruby-profを用いてcbenchを解析した際に出力されたログを添付する.

'''  
 %self      total      self      wait     child     calls  name
  2.39     25.841     2.592     0.000    23.250   186136   BinData::Base#new
  2.22      5.733     2.408     0.000     3.325   192208   Kernel#clone
  2.15      7.140     2.323     0.000     4.817   361658  *BinData::BasePrimitive#_value
  2.12      2.361     2.295     0.000     0.065   596525   Kernel#respond_to?
  1.88     23.780     2.029     0.001    21.750   164850   BinData::Struct#instantiate_obj_at
  1.81      5.942     1.957     0.000     3.985   310104  *BinData::BasePrimitive#snapshot
  1.77      3.184     1.912     0.000     1.273   192208   Kernel#initialize_clone
  1.53      1.652     1.652     0.000     0.000   199088   BasicObject#!
  1.51      2.090     1.630     0.000     0.460   239613   Kernel#define_singleton_method
  1.46      1.576     1.576     0.000     0.000   196183   Symbol#to_s
  
'''

上記では,cbenchのプログラム内でボトルネックとなっている部分を上位から順に記載している.
上述の通り,BinDataクラスのメソッドとKernelモジュールのメソッドの実行に要する時間が長く,これらがボトルネックの主な要因となっていることがわかる.
cbenchプロセスが送信してくるPacket Inの中身が毎回同じものであるに関わらず, '''cbench.rb''' 内のPacketInハンドラ内の '''send_flow_mod_add''' において,Packet InごとにFlow Modメッセージを毎回初めから作成する処理を行っている.ここで,Flow modメッセージを作成するために用いられている,BinDataのクラスメソッドでありインスタンスであるオブジェクトを作成する '''new''' メソッドがボトルネックの最大の要因であると考えられる.