=================================================
インタフェースに対して（許可・拒否）の設定
=================================================


:著者: RYOICHI
:版数: vRouter5600 version4.2R1S1

|  ここでは、ファイアウォールを通過するIP通信に対して許可または拒否する機能について紹介します。

**パケットフィルタリング設定のながれ**

| １．パケットフィルタリング設定の名前を設定する
| ２．送信元IPアドレスの指定とそれを許可・拒否のルールを設定する
| ３．２．で作ったルールに該当しないIPアドレスをもつ通信に関して許可・拒否するルールを設定する
| ４．１．で作ったパケットフィルタリング設定を適用させるインタフェースに対してフィルタのその方向について指定する


特定の送信元IPアドレスからの通信を拒否する設定
=================================================

| 特定のIPアドレスをもつホストからの通信をファイアウォールのインタフェースで遮断する設定
| を行います。
 

**サンプル設定のシナリオ**

* IPアドレス192.168.2.13を送信元IPアドレスとするホストからの通信をとめたい
* インタフェース（dp0s5）にインプットするトラフィックに対して有効化したい
* 192.168.2.13以外の送信元IPアドレスからの通信は、すべて許可する設定にしたい

| 構成図(編集するかどうか検討）

.. imageで指定するか、figureで設定するか？

.. figure:: /image/exfirewall_fig1.png
    :alt:     仮の画像
    :width:     35%

| 上記画像では、PacketFiltering設定で確認しているPPTの図を貼り付け


　**シナリオにおける設定のながれ**

| １．パケットフィルタリング設定の名前 **dp04_in**
| ２．192.168.2.13を送信元とするパケットを拒否するルールを10として設定
| ３．192.168.2.13以外を送信元とするパケットについて許可する設定
| ４．dp0s5 インタフェースにてインプット方向へ適用

**CLIにて入力するコマンド**

.. code-block:: none

 #edit security firewall name dp0s4_in 
 #set rule 10 source address 192.168.2.13 
 #set rule 10 action drop
 #set default-action accept
 #set interfaces dataplane dp0s5 firewall in dp0s4_in



| 正しく設定が完了したときのコンフィグレーションは次のとおりです。

.. code-block:: none

	#config in
	 interfaces {
		dataplane dp0s4 {
			address 192.168.1.50/24
		}
		dataplane dp0s5 {
			address 192.168.2.50/24
			firewall {
				in dp0s4_in
			}
		}
		dataplane dp0s6 {
			address 192.168.3.5/24
		}
		
	 security {
		firewall {
			name dp0s4_in {
				default-action accept
				rule 10 {
					action drop
					source {
						address 192.168.2.13
					}
				}
			}
		}
	 }


**コンフィグレーションなどにコメントをつけていくとしたら大変かも**

**コンフィグはログオプションがついているがこれをのせるかどうか**

**動作確認結果**

| 以下の検証結果ログから、検証構成図にあるサーバ(192.168.2.12)から
| 192.168.3.3あての通信（Ping)が成功しますが、フィルタリング設定を
| 実施したサーバ(192.168.2.13)から192.168.3.3あての通信（Ping)は失敗しており
| パケットフィルタリング機能が動作していることが確認できました。

.. code-block:: none

	#192.168.2.12から通信 -> OK

	test@ubu03:~$ ping 192.168.3.3
	PING 192.168.3.3 (192.168.3.3) 56(84) bytes of data.
	64 bytes from 192.168.3.3: icmp_seq=1 ttl=63 time=5.31 ms
	64 bytes from 192.168.3.3: icmp_seq=2 ttl=63 time=1.30 ms
	64 bytes from 192.168.3.3: icmp_seq=3 ttl=63 time=1.83 ms
	64 bytes from 192.168.3.3: icmp_seq=4 ttl=63 time=0.987 ms
	64 bytes from 192.168.3.3: icmp_seq=5 ttl=63 time=0.990 ms
	64 bytes from 192.168.3.3: icmp_seq=6 ttl=63 time=1.13 ms
	^C
	--- 192.168.3.3 ping statistics ---
	6 packets transmitted, 6 received, 0% packet loss, time 5007ms
	rtt min/avg/max/mdev = 0.987/1.928/5.319/1.544 ms


	#192.168.2.13から通信 -> NG

	test@ubu04:~$ ping 192.168.3.3
	PING 192.168.3.3 (192.168.3.3) 56(84) bytes of data.
	^C
	--- 192.168.3.3 ping statistics ---
	4 packets transmitted, 0 received, 100% packet loss, time 3023ms





特定の送信元IPアドレスからの通信だけ許可する設定
=================================================


| 特定のIPアドレスをもつホストからの通信のみ許可して、その他のIPアドレスを送信元とする
| 通信に関しては、ファイアウォールのインタフェースで遮断する設定を行います。
 

**サンプル設定のシナリオ**

* IPアドレス192.168.2.13を送信元IPアドレスとするホストからの通信だけ許可したい
* インタフェース（dp0s5）にインプットするトラフィックに対して有効化したい
* 192.168.2.13以外の送信元IPアドレスからの通信は、すべて拒否し遮断する設定にしたい

| 構成図(内容要チェック　　　編集するかどうか検討）

.. imageで指定するか、figureで設定するか？

.. figure:: /image/exfirewall_fig1.png
    :alt:     仮の画像
    :width:     35%

| 上記画像では、PacketFiltering設定で確認しているPPTの図を貼り付け


**シナリオにおける設定のながれ**

| １．パケットフィルタリング設定の名前 **dp04_in**
| ２．192.168.2.13を送信元とするパケットを許可するルールを10として設定
| ３．192.168.2.13以外を送信元とするパケットについて拒否する設定
| ４．dp0s5 インタフェースにてインプット方向へ適用

.. code-block:: none

    vRouterにおけるコンフィグレーション
	#config in
	 interfaces {
		dataplane dp0s4 {
			address 192.168.1.50/24
		}
		dataplane dp0s5 {
			address 192.168.2.50/24
			firewall {
				in dp0s4_in
			}
		}
		dataplane dp0s6 {
			address 192.168.3.5/24
		}
	 security {
		firewall {
			name dp0s4_in {
				default-action drop
				rule 10 {
					action accept
					log
					source {
						address 192.168.2.13
					}
				}
			}
		}
	 }



**動作確認結果**

| 以下の検証結果ログから、検証構成図にあるサーバ(192.168.2.13)から
| 192.168.3.3あての通信（Ping)が成功しますが、フィルタリング設定を
| 実施したサーバ(192.168.2.12)から192.168.3.3あての通信（Ping)は失敗しており
| パケットフィルタリング機能が動作していることが確認できました。

.. code-block:: none

	#192.168.2.13から通信 -> OK

	test@ubu04:~$ ping 192.168.3.3
	PING 192.168.3.3 (192.168.3.3) 56(84) bytes of data.
	64 bytes from 192.168.3.3: icmp_seq=1 ttl=63 time=3.27 ms
	64 bytes from 192.168.3.3: icmp_seq=2 ttl=63 time=4.03 ms
	64 bytes from 192.168.3.3: icmp_seq=3 ttl=63 time=1.76 ms
	64 bytes from 192.168.3.3: icmp_seq=4 ttl=63 time=2.03 ms
	64 bytes from 192.168.3.3: icmp_seq=5 ttl=63 time=2.36 ms
	^C
	--- 192.168.3.3 ping statistics ---
	5 packets transmitted, 5 received, 0% packet loss, time 4006ms
	rtt min/avg/max/mdev = 1.769/2.695/4.034/0.841 ms


	#192.168.2.12から通信 -> NG

	test@ubu03:~$ ping 192.168.3.3
	PING 192.168.3.3 (192.168.3.3) 56(84) bytes of data.
	^C
	--- 192.168.3.3 ping statistics ---
	5 packets transmitted, 0 received, 100% packet loss, time 4032ms
