## Setting up a Testing Environment for iOS Apps

前章で, iOS プラットフォームの概要を示し iOS アプリの構造を説明した.
ここでは, iOS アプリのセキュリティをテストする際に使える基本的なプロセスとテクニック, およびその流れを紹介する.
これらの基本的なプロセスは後述のより詳細なテストケースの概略に基づいている.

実際の Android デバイスのハードウェアから完全にエミュレートする Android エミュレータとは異なり, iOS SDK シミュレータは iOS デバイスの高度な***シミュレーション***を行う.
もっとも重要なこととしては, エミュレータバイナリが ARM コードの代わりに x86 コードにコンパイルされていることだ.
実環境に対してコンパイルされたアプリは実行されないため, ブラックボックス解析やリバースエンジニアリングは役に立たなくなる.

以下は最低限の iOS アプリのテスト環境である:
- admin 権限のラップトップ
- クライアント間で通信可能な Wi-Fi ネットワーク(もしくは USB 多重化)
- 最低 1 つの Jailbreak された iOS デバイス(iOS のバージョンは任意)
- Burpsuite や他の傍受可能なプロキシツール

Linux や Windows マシンをテストに使用する場合, それらの環境ではテストが難しい, あるいは不可能な事象に多くの遭遇するだろう.
加えて, Xcode 開発環境と iOS SDK は macOS でのみ使用できる.
つまり, Mac でソースコード解析やデバッグを行うことを勧める(ブラックボックステストも容易になる).

### Jailbreaking an iOS Device

テストを実施するにあたり Jailbreak 済みの iPhone もしくは iPad を用意するべきだ.
これらのデバイスであれば root アクセスやツールのインストールができ, セキュリティテストのプロセスを素直に進めることができる.
もし Jailbreak された端末がない場合, 以降の章で説明する軽減策を行ってほしい. ただし, 難しい手順の準備が求められる.

iOS の Jailbreak はよく Android の root 化と比較されるが, プロセスは全く異なる.
違いを説明するために, まず Android の "rooting" と "flashing" の概念を復習しよう.

- **Rooting**: これは大抵, 既存システムに `su` バイナリをインストールするか, システム全体を root 化済みのカスタム ROM に置き換える.
通常, ブートローダにアクセス可能な場合, root アクセスを手に入れるにはエクスプロイトは必要にならない.
- **Flashing custom ROMs** (すでに root 化されているもの): これは動作しているデバイスのブートローダをアンロックしたのち, OS を置き換えるものである(場合によってはエクスプロイトが必要になる).

iOS デバイスにおいては, iOS ブートローダで Apple が署名したイメージのみブートとフラッシュが許可されているため, カスタム ROM のフラッシュができない.
これは公式の iOS イメージでさえ, それらが Apple によって署名されていない場合インストールできない, このことから iOS のダウングレードもよくできなくなっている.

Jailbreak の目的は, 署名されていない任意のコードをデバイス上で実行するために iOS システムのプロテクション(一部のアップルによるコード署名機構)を無効にすることだ.
Jailbreak という単語は, プロセス無効化を自動化する all in one なツールを指した俗語である.

Cydia は Jay Freeman ("saurik") によって Jailbreak されたデバイス向けに開発された, 別の App Store である.
Cydia はグラフィカルユーザインターフェースと Advanced Packaging Tool (APT) を提供する.
Cydia で多くの (Apple に) "認可されていない" アプリを容易に使用できる.
ほとんどの Jailbreak ツールは Cydia を自動的にインストールする.

提供されている iOS 各バージョンの Jailbreak の開発は容易ではない.
セキュリティテスターとしては, 公開されている Jailbreak ツールを使用したいだろう.
それでも, iOS の色々なバージョンを Jailbreak するために使用されてきたテクニックを学ぶことをお勧めする. 多くの興味深いエクスプロイトに出会い, OS 内部についてもたくさんのことを学ぶことができるだろう.
例えば, iOS 9.x 系への Pangu9 は[最低5つの脆弱性でエクスプロイトする](https://www.theiphonewiki.com/wiki/Jailbreak_Exploits "Jailbreak Exploits"). これには use-after-free のカーネルバグ(CVE-2015-6794)と Photos アプリの任意のファイルシステムへのアクセス可能な脆弱性(CVE-2015-7037)が含まれている.

#### Benefits of Jailbreaking

エンドユーザはたいてい iOS システムの表示調整, 新しい機能の追加, 非公式の app store からサードパーティのアプリをインストールを行うために Jailbreak する.
しかしテスターにとっては, iOS デバイスを Jailbreak することは, それよりも利点がある.
その利点は以下のようなものだが, これらに限った話ではない:
- ファイルシステムへ root アクセス可能
- Apple に署名されていない(セキュリティツールが多く含まれる)アプリケーションを実行できる可能性がある.
- デバッギングや動的解析が制限されない
- Objective-C runtime にアクセス可能

#### Jailbreak Types

Jailbreak　には *tethered* や, *semi-tethered*, *semi-untethered*, *untethered* がある.

- Tethered jailbreaks では, 再起動をすると Jailbreak 状態は持続しない. そのため, Jailbreak するためにデバイスを再起動をするたびコンピュータにつなげる(tethered)必要がある.
  デバイスがコンピュータに繋がっていない場合, まったく再起動できない場合がある.
- Semi-tethered jailbreaks は再起動の間にデバイスをコンピュータに接続していないと再適用できない.
  デバイスは Jailbreak されていない状態にブートできる.
- Semi-untethered jailbreaks は自身でブート可能だが, コード署名無効化のカーネルパッチは自動的には適用されない.
  ユーザはアプリを開始するか, (Jailbreakのための)サイトに行き, 再度 Jailbreak をしなければならない.
- Untethered jailbreaks は適用するのが一度だけで良く, その後永続的に Jailbreak されたままになるため, エンドユーザにとって最も人気の方法である.

#### Caveats and Considerations

iOS デバイスの Jailbreak は Apple がシステムを堅牢にし, エクスプロイト可能な脆弱性を修正しているため, どんどん難しくなっている.
Jailbreak はとても時間的に制約のある工程になっている. Apple が短い期間で脆弱なバージョンへの署名を止めているためである(Hardware 依存の脆弱性は除く).
これは一度 Apple がファームウェアへの署名を止めた iOS の特定のバージョンにダウングレードできない, ということを意味する.

もしセキュリティテストのために使用する Jailbreak 済みのデバイスを持っているようであれば, 最新のバージョンへアップグレードした後にまた Jailbreak できることを 100% 確信できない場合はそのまま維持すること.
またスペアのデバイス(各メジャー iOS リリースへのアップデート用)を入手し, Jailbreak が公にリリースされることを待つことを考慮すること.
一度 Jailbreak の手段が公表されると, Apple は通常パッチを素早くリリースするため, Jailbreak 可能なバージョンにアップグレードし, Jailbreak を適用するには数日しかない.

iOS のアップグレードはチャレンジ-レスポンスで行われている.
デバイスはチャレンジのレスポンスが Apple に署名されている場合にのみ, OS インストールができる.
リサーチャーはこれを "signing window(署名中の好機**←ここ微妙**)" と呼んでいる. iTunes からダウンロードしデバイスにロードした OTA ファームウェアを, 任意のタイミングで適用できないためだ.

マイナーな iOS アップグレードの間では, 2 つとも Apple に署名される.
これが iOS デバイスをダウングレード可能な唯一のシチュエーションだ.
[IPSW Downloads website](https://ipsw.me "IPSW Downloads") から直近の signing window を確認でき, OTA ファームウェアをダウンロードできる.

#### Which Jailbreaking Tool to Use

iOS のバージョンが異なると, 異なる Jailbreak のテクニックが必要になる.
所有している iOS デバイスが Jailbreak で可能かどうかは[確認すること](https://canijailbreak.com/ "Can I Jailbreak").
その際には偽のツールやスパイウェアには注意すること. これらはしばしばドメイン名を Jailbreak を発表したグループや著者の名前に似せている.

Pangu 1.3.0 の Jailbreak は iOS 9.0 が動作している 64 bit デバイスで利用可能だ.
Jailbreak の方法がない iOS が動いているデバイスを持っている場合, (IPSW download や iTunes から) Jailbreak 可能なバージョンにダウングレードあるいはアップグレードすることで Jailbreak することができる.
しかし, Jailbreak に必要なバージョンが Apple に署名されていない場合, これは不可能かもしれない.

iOS Jailbreak の場は最新の説明を行うことが難しいほど, 急速に展開している.
しかし, 現時点で信頼性のあるいくつかのソースを示すことはできる.

- [The iPhone Wiki](https://www.theiphonewiki.com/ "The iPhone Wiki")
- [Redmond Pie](http://www.redmondpie.com/ "Redmone Pie")
- [Reddit Jailbreak](https://www.reddit.com/r/jailbreak/ "Reddit Jailbreak")

> もしあなたの iOS デバイスが動作しなくなったとしても, OWASP と MSTG は責任を負わない.

#### Dealing with Jailbreak Detection

いくつかのアプリは実行環境が Jailbreak されているか, 検出しようとする.
これは, Jailbreak が iOS デフォルトのセキュリティ機構をいくつか無効にするためである.
しかし, この検知を回避する方法はいくつかあり, "Reverse Engineering and Tampering on iOS" と "Testing Anti-Reversing Defenses on iOS" の章でそのテクニックのいくつかを紹介していく.

#### Jailbroken Device Setup

<img src="Images/Chapters/0x06b/cydia.png" width="500px"/>
- *Cydia Store*

Jailbreak して Cydia をインストールしたら(上のスクリーンショットで見られる), 以下を行っておく.

1. aptitude と openssh を Cydia からインストールする.
2. iDevice に SSH でログインする.
  * デフォルトユーザは `root` と `mobile`
  * デフォルトパスワードは `alpine`
3. `root` と `mobile` のパスワードを変更する
4. Cydia に次のレポジトリを追加する: `https://build.frida.re`
5. Cydia から Frida をインストールする

Cydia ではレポジトリを管理できる.
最も人気のあるレポジトリの一つは BigBoss だ.
もし Cydia のインストール時に初期設定でこのレポジトリがない場合, "Sources(ソース)" > "Edit(編集)" から左上の "Add(追加)" をクリックし, 次の URL を入力することで追加できる.

```
http://apt.thebigboss.org/repofiles/cydia/
```
また HackYouriPhone レポジトリも追加して AppSync パッケージを手に入れることもできる.

```
http://repo.hackyouriphone.org
```

以下は Cydia からインストール可能な, 便利なパッケージ群だ.

- BigBoss Recommended Tools: ハッカー向けツールのリストで, 多くのコマンドラインツールがインストールされる. wget や unrar, less, sqlite3 client などの標準的な Unix ユーティリティを含んでいる.
- adv-cmds: 高度なコマンドラインツール. finger や fingerd, last, lsvfs, md, ps を含む.
- [IPA Installer Console](http://cydia.saurik.com/package/com.autopear.installipa/ "IPA Installer Console"): コマンドラインから IPA アプリケーションパッケージをインストールするためのツール. パッケージ名は `com.autopear.installipa`.
- Class Dump: Mach-O ファイルに保存された Objective-C ランタイム情報を調べるためのコマンドラインツール.
- Substrate: サードパーティの iOS アドオン開発を容易に行えるようにしてくれるプラットフォーム.
- cycript: インライン展開や最適化を行う JavaScript コンパイラで, 動作しているプロセスにインジェクトされる immediate-mode のコンソール環境.
- AppList: 開発者がインストール済みアプリのリストを照会できるようにし, リストに基づいて設定パネルを提供する.
- PreferenceLoader: MobileSubstrate をベースにしたユーティリティで, 開発者は Settings(設定) アプリにエントリを追加できるようになる. AppStore が参照する Settings Bundles に似ている.
- AppSync Unified: 署名されていない iOS アプリケーションの同期およびインストールをできるようになる.

ワークステーションには最低でも以下をインストールすべきだ.

- SSH クライアント
- インターセプト可能なプロキシ. このガイドでは, [BURP Suite](https://portswigger.net/burp) を使用する.

このガイドでは他の便利なツールとして以下について触れる.

- [Introspy](https://github.com/iSECPartners/Introspy-iOS)
- [Frida](http://www.frida.re)
- [IDB](http://www.idbtool.com)
- [Needle](https://github.com/mwrlabs/needle)

### Static Analysis

iOS アプリに対する静的解析の方法は, オリジナルの Xcode プロジェクトファイルを使用することがもっとも望ましい.
理想的には, アプリをコンパイルおよびデバッグを行うことで, 潜在的にソースコードに存在する問題を素早く特定することができる.

オリジナルのソースコードを用いない iOS アプリのブラックボックス解析 はリバースエンジニアリングが必要となる.
例えば, iOS アプリでは利用可能なデコンパイラがないため, 詳細な検査のためにはアセンブリコードを読めなければならない.
この章では詳しいところまで触れないが, このトピックには "Reverse Engineering and Tampering on iOS(iOS でのリバースエンジニアリングと改ざん)" の章で再検討する.

次章の静的解析の説明はソースコードが手元にある前提で行なっている.

#### Automated Static Analysis Tools

いくつかの自動化ツールはたいていのものは商用ツールではあるものの, iOS アプリの解析に利用可能である.
無償でオープンソースのツールでは, [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF "Mobile Security Framework (MobSF)") と [Needle](https://github.com/mwrlabs/needle "Needle") が静的および動的な解析を行える機能を備えている.
他のプロダクトは付録の "Testing Tools" 内, "Static Source Code Analysis" に一覧化して記載している.

解析に自動化ツールを使用することは恥ずかしいことではない - それらのツールは, 容易に終えられる試験を助けてくれ, ビジネスロジックなどのより興味深い解析に集中できるようにしてくれる.
静的解析ツールでは false positives と false negatives が発生することを肝に命じておいて欲しい. 必ず結果を注意深く確認すること.

### Dynamic Analysis of Jailbroken Devices

Jailbreak 済みの端末があると解析が容易になる: アプリのサンドボックスへのアクセス権を取得することだけでなく, コード署名のチェックがないことで強力な動的解析テクニックを使用できる.
iOS では, たいていの動的解析ツールは Cydia Substrate 上でビルドされている. 後ほど触れるが, Cydia Substrate はランタイムパッチ開発のためのフレームワークである.
基本的な API モニタリングでは, Substrate の細かい動きをすべて把握していなくとも, Substrate 上でビルドされた既存の API モニタリングツールを使用することで, 行うことができる.

#### SSH Connection via USB

実際のブラックボックステストでは, 信頼性のある Wi-Fi 接続を使用できない場合がある.
この状況では, USB 上でデバイスの SSH サーバへ接続するのに [usbmuxd](https://github.com/libimobiledevice/usbmuxd "usbmuxd") を使用できる.

usbmuxd は USB iPhone 接続をモニタするためのソケットデーモンである.
usbmuxd を使用することで, モバイルデバイスのローカル Listen ソケットをホストマシンの TCP ポートにマッピングできる.
これにより実際のネットワーク設定を行わず, 対話的に SSH で iOS デバイスへ接続できる.
usbmuxd が通常モードで動作している iOS を検出した際, iPhone に接続し /var/run/usbmuxd で受け取ったリクエストを中継し始める.

iproxy をインストールおよび起動し, macOS から iOS デバイスに接続する.

```bash
$ brew install libimobiledevice
$ iproxy 2222 22
waiting for connection
```

上記のコマンドではローカルホストの 2222 バンポートを iOS デバイスの 22 番ポートにマッピングしている.
以下のコマンドで, デバイスに接続できる.

```shell
$ ssh -p 2222 root@localhost
root@localhost's password:
iPhone:~ root#
```

USB を使用した iPhone への接続は [Needle](https://labs.mwrinfosecurity.com/blog/needle-how-to/ "Needle") からも行うことができる

#### App Folder Structure

システムアプリケーションは "/Applications" ディレクトリに置かれている.
ユーザがインストールしたアプリのインストールフォルダ(iOS 9 から `/private/var/mobile/Containers/` 配下が使用されている)を識別するには [IPA Installer Console](http://cydia.saurik.com/package/com.autopear.installipa "IPA Installer Console") を使用すると良い.
以下のように, SSH でデバイスに接続し `ipainstaller`(同様のコマンドに `installipa`がある) を実行する.

```shell
iPhone:~ root# ipainstaller -l
...
sg.vp.UnCrackable1

iPhone:~ root# ipainstaller -i sg.vp.UnCrackable1
...
Bundle: /private/var/mobile/Containers/Bundle/Application/A8BD91A9-3C81-4674-A790-AF8CDCA8A2F1
Application: /private/var/mobile/Containers/Bundle/Application/A8BD91A9-3C81-4674-A790-AF8CDCA8A2F1/UnCrackable Level 1.app
Data: /private/var/mobile/Containers/Data/Application/A8AE15EE-DC8B-4F1C-91A5-1FED35258D87
```

見てわかる通り, ユーザインストールのアプリには 2 つの主なサブディレクトリがある(iOS 9 から `Shared` サブディレクトリが追加されている).
`Application` サブディレクトリは `Bundle` サブディレクトリ内にあり, アプリ名を反映している(`<name>.app` サブディレクトリに含まれている).

- `Bundle`
- `Data`

`Application` は `Bundle` のサブディレクトリである.
静的なインストーラファイルは `Application` ディレクトリにあり, 全ユーザデータは `Data` ディレクトリにある.

URI のランダムな文字列はアプリケーションの GUID である.
各アプリのインストールフォルダは固有の GUID を持っている.
同一のアプリでも `Bundle` の GUID と `Data` の GUID には関連はない.

#### Copying App Data Files

アプリのファイルはアプリ内の `Data` ディレクトリに保存される.
正確なパスを把握するため, デバイスに SSH でアクセスし, IPA Installer Console で(先ほど見たような)パッケージ情報を取得する.

```bash
iPhone:~ root# ipainstaller -l
...
sg.vp.UnCrackable1

iPhone:~ root# ipainstaller -i sg.vp.UnCrackable1
Identifier: sg.vp.UnCrackable1
Version: 1
Short Version: 1.0
Name: UnCrackable1
Display Name: UnCrackable Level 1
Bundle: /private/var/mobile/Containers/Bundle/Application/A8BD91A9-3C81-4674-A790-AF8CDCA8A2F1
Application: /private/var/mobile/Containers/Bundle/Application/A8BD91A9-3C81-4674-A790-AF8CDCA8A2F1/UnCrackable Level 1.app
Data: /private/var/mobile/Containers/Data/Application/A8AE15EE-DC8B-4F1C-91A5-1FED35258D87
```

`Data` ディレクトリを圧縮し, `scp` でデバイスからデータを容易に持ってくることができる.

```bash
iPhone:~ root# tar czvf /tmp/data.tgz /private/var/mobile/Containers/Data/Application/A8AE15EE-DC8B-4F1C-91A5-1FED35258D87
iPhone:~ root# exit
$ scp -P 2222 root@localhost:/tmp/data.tgz .
```

#### Dumping KeyChain Data

[Keychain-Dumper](https://github.com/ptoomey3/Keychain-Dumper/) を使用することで Jailbreak した端末の KeyChain コンテンツをダンプできる.
最も簡単にこのツールを手に入れる方法は GitHub レポジトリからバイナリをダウンロードすることだ.

``` bash
$ git clone https://github.com/ptoomey3/Keychain-Dumper
$ scp -P 2222 Keychain-Dumper/keychain_dumper root@localhost:/tmp/
$ ssh -p 2222 root@localhost
iPhone:~ root# chmod +x /tmp/keychain_dumper
iPhone:~ root# /tmp/keychain_dumper

(...)

Generic Password
----------------
Service: myApp
Account: key3
Entitlement Group: RUD9L355Y.sg.vantagepoint.example
Label: (null)
Generic Field: (null)
Keychain Data: SmJSWxEs

Generic Password
----------------
Service: myApp
Account: key7
Entitlement Group: RUD9L355Y.sg.vantagepoint.example
Label: (null)
Generic Field: (null)
Keychain Data: WOg1DfuH
```

このバイナリは"ワイルドカード"の entitlement を持つ自己署名証明書で署名されている.
この entitlement はキーチェーンの**全**アイテムへのアクセス権限を与えてくれる.
もし, あなたが心配性, あるいはテスト対象としているデバイスに非常にセンシティブなプライベートデータがある場合, ソースコードからビルドし, 自身で適切な entitlement の署名を行う必要がある. これらを行うための手順は Github レポジトリに記載がある.

#### Installing Frida

[Frida](https://www.frida.re "frida") はフレームワークで, JavaScript スニペットや自身のライブラリをネイティブな Android アプリと iOS アプリにインジェクトすることができる.
Android のセクションをすでに読んでいるなら, このツールはよく知っているだろう.
まだ読んでいなようであれば, ホストマシンに Frida Python パッケージをインストールする必要がある.

```shell
$ pip install frida
```

iOS アプリに Frida をアタッチするためには, Frida ランタイムをアプリにインジェクトする必要がある.
Jailbreak 済みのデバイスであれば容易で, Cydia から frida-server インストールするだけだ.
インストール後, frida-server は自動的に root 権限で実行され, 各プロセスへ容易にコードをインジェクト可能になる.

Cydia を起動し Frida レポジトリを追加する(Manage -> Sources -> Edit -> Add から `https://build.frida.re` を入力).
Frida パッケージを検索, インストールできるようになる.

USB でデバイスに接続し, `frida-ps` コマンドから Frida が動作しているかを確認する.
デバイスで動作しているプロセスのリストが返される.

```shell
$ frida-ps -U
PID  Name
---  ----------------
963  Mail
952  Safari
416  BTServer
422  BlueTool
791  CalendarWidget
451  CloudKeychainPro
239  CommCenter
764  ContactsCoreSpot
(...)
```

以下に Frida の使用方法をいくつか示すが, まず最初に Jailbreak されていないデバイスで動作させるために何をすべきかみてみよう.

### Dynamic Analysis on Non-Jailbroken Devices

もし Jailbreak したデバイスにさわれない場合, 起動時にロードされるダイナミックライブラリにパッチを当て, 対象のアプリを再パッケージ化する必要がある.
この方法で, アプリを計測でき, 動的解析に必要なことはほとんど行うことができる(もちろん, この方法ではサンドボックスから抜けることはできないが, たいてい抜ける必要はない).
しかし, このテクニックを使用できるのはアプリのバイナリが FairPlay-encrypted (言い換えれば App Store から入手したもの) でない場合に限る.


Thanks to Apple's confusing provisioning and code signing system, re-signing an app is more challenging than one would expect.
iOS won't run an app unless you get the provisioning profile and code signature header exactly.
This requires learning many concepts—certificate types, BundleIDs, application IDs, team identifiers, and how Apple's build tools connect them.
Suffice it to say, getting the OS to run a binary that hasn't been built via the default method (Xcode) can be a daunting process.

We're going to use `optool`, Apple's build tools, and some shell commands. Our method is inspired by [Vincent Tan's Swizzler project](https://github.com/vtky/Swizzler2/ "Swizzler"). [The NCC group](https://www.nccgroup.trust/au/about-us/newsroom-and-events/blogs/2016/october/ios-instrumentation-without-jailbreak/ "NCC blog - iOS instrumentation without jailbreak") has described an alternative repackaging method.

To reproduce the steps listed below, download [UnCrackable iOS App Level 1](https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/iOS/Level_01 "Crackmes - iOS Level 1") from the OWASP Mobile Testing Guide repo. Our goal is to make the UnCrackable app load FridaGadget.dylib during startup so we can instrument it with Frida.

> Please note that the following steps are applicable to macOS only. Xcode is available for macOS only.

#### Getting a Developer Provisioning Profile and Certificate

The *provisioning profile* is a plist file signed by Apple. It whitelists your code signing certificate on one or more devices. In other words, this represents Apple's explicitly allowing your app to run for certain reasons, such as debugging on selected devices (development profile). The provisioning profile also includes the *entitlements* granted to your app. The *certificate* contains the private key you'll use to sign.

Depending on whether you're registered as an iOS developer, you can obtain a certificate and provisioning profile in one of the following ways:

**With an iOS developer account:**

If you've developed and deployed iOS apps with Xcode before, you already have your own code signing certificate installed. Use the *security* tool to list your signing identities:

```shell
$ security find-identity -p codesigning -v
  1) 61FA3547E0AF42A11E233F6A2B255E6B6AF262CE "iPhone Distribution: Vantage Point Security Pte. Ltd."
  2) 8004380F331DCA22CC1B47FB1A805890AE41C938 "iPhone Developer: Bernhard Müller (RV852WND79)"
```

Log into the Apple Developer portal to issue a new App ID, then issue and download the profile. An App ID is a two-part string used consisting of a Team ID supplied by Apple and a bundle ID search string that you can set to an arbitrary value, such as `com.example.myapp`. Note that you can use a single App ID to re-sign multiple apps. Make sure you create a *development* profile and not a *distribution* profile so that you can debug the app.

In the examples below, I use my own signing identity, which is associated with my company's development team. I created the app-id "sg.vp.repackaged" and the provisioning profile  "AwesomeRepackaging" for these examples. I ended up with the file AwesomeRepackaging.mobileprovision—replace this with your own filename in the shell commands below.

**With a Regular iTunes Account:**

Apple will issue a free development provisioning profile even if you're not a paying developer. You can obtain the profile with Xcode and your regular Apple account: simply create an empty iOS project and extract embedded.mobileprovision from the app container, which is in the Xcode subdirectory of your home directory: `~/Library/Developer/Xcode/DerivedData/<ProjectName>/Build/Products/Debug-iphoneos/<ProjectName>.app/`. The [NCC blog post "iOS instrumentation without jailbreak"](https://www.nccgroup.trust/au/about-us/newsroom-and-events/blogs/2016/october/ios-instrumentation-without-jailbreak/ "iOS instrumentation without jailbreak") explains this process in great detail.

Once you've obtained the provisioning profile, you can check its contents with the *security* tool. Besides the allowed certificates and devices, you'll find the entitlements granted to the app in the profile. You'll need those for code signing, so extract them to a separate plist file as shown below. Have a look at the file contents to make sure everything is as expected.

```shell
$ security cms -D -i AwesomeRepackaging.mobileprovision > profile.plist
$ /usr/libexec/PlistBuddy -x -c 'Print :Entitlements' profile.plist > entitlements.plist
$ cat entitlements.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>application-identifier</key>
	<string>LRUD9L355Y.sg.vantagepoint.repackage</string>
	<key>com.apple.developer.team-identifier</key>
	<string>LRUD9L355Y</string>
	<key>get-task-allow</key>
	<true/>
	<key>keychain-access-groups</key>
	<array>
		<string>LRUD9L355Y.*</string>
	</array>
</dict>
</plist>
```

Note the application identifier, which is a combination of the Team ID (LRUD9L355Y) and Bundle ID (sg.vantagepoint.repackage). This provisioning profile is only valid for the app that has this app id. The "get-task-allow" key is also important—when set to "true," other processes, such as the debugging server, are allowed to attach to the app (consequently, this would be set to "false" in a distribution profile).

#### Other Preparations

To make our app load an additional library at startup, we need some way of inserting an additional load command into the main executable's Mach-O header. [Optool](https://github.com/alexzielenski/optool "Optool") can be used to automate this process:

```shell
$ git clone https://github.com/alexzielenski/optool.git
$ cd optool/
$ git submodule update --init --recursive
$ xcodebuild
$ ln -s <your-path-to-optool>/build/Release/optool /usr/local/bin/optool
```

We'll also use [ios-deploy](https://github.com/phonegap/ios-deploy "ios-deploy"), a tool that allows iOS apps to be deployed and debugged without Xcode:

```shell
$ git clone https://github.com/phonegap/ios-deploy.git
$ cd ios-deploy/
$ xcodebuild
$ cd build/Release
$ ./ios-deploy
$ ln -s <your-path-to-ios-deploy>/build/Release/ios-deploy /usr/local/bin/ios-deploy
```

The last line in optool and ios-deploy creates a symbolic link and makes the executable available system-wide.

Reload your shell to make the new commands available:

```shell
zsh: # . ~/.zshrc
bash: # . ~/.bashrc
```

To follow the examples below, you also need FridaGadget.dylib:

```shell
$ curl -O https://build.frida.re/frida/ios/lib/FridaGadget.dylib
```

Besides the tools listed above, we'll be using standard tools that come with macOS and Xcode. Make sure you have the [Xcode command line developer tools](http://railsapps.github.io/xcode-command-line-tools.html "Xcode Command Line Tools") installed.

#### Patching, Repackaging, and Re-Signing

Time to get serious! As you already know, IPA files are actually ZIP archives, so you can use any zip tool to unpack the archive. Copy FridaGadget.dylib into the app directory and use optool to add a load command to the "UnCrackable Level 1" binary.

```shell
$ unzip UnCrackable_Level1.ipa
$ cp FridaGadget.dylib Payload/UnCrackable\ Level\ 1.app/
$ optool install -c load -p "@executable_path/FridaGadget.dylib"  -t Payload/UnCrackable\ Level\ 1.app/UnCrackable\ Level\ 1
Found FAT Header
Found thin header...
Found thin header...
Inserting a LC_LOAD_DYLIB command for architecture: arm
Successfully inserted a LC_LOAD_DYLIB command for arm
Inserting a LC_LOAD_DYLIB command for architecture: arm64
Successfully inserted a LC_LOAD_DYLIB command for arm64
Writing executable to Payload/UnCrackable Level 1.app/UnCrackable Level 1...
```

Of course such blatant tampering invalidates the main executable's code signature, so this won't run on a non-jailbroken device. You'll need to replace the provisioning profile and sign both the main executable and FridaGadget.dylib with the certificate listed in the profile.

First, let's add our own provisioning profile to the package:

```shell
$ cp AwesomeRepackaging.mobileprovision Payload/UnCrackable\ Level\ 1.app/embedded.mobileprovision
```

Next, we need to make sure that the BundleID in Info.plist matches the one specified in the profile because the `codesign` tool will read the Bundle ID from Info.plist during signing; the wrong value will lead to an invalid signature.

```shell
$ /usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier sg.vantagepoint.repackage" Payload/UnCrackable\ Level\ 1.app/Info.plist
```

Finally, we use the codesign tool to re-sign both binaries. Instead of "8004380F331DCA22CC1B47FB1A805890AE41C938," you need to use your signing identity, which you can output by executing the command `security find-identity -p codesigning -v`.

```shell
$ rm -rf Payload/UnCrackable\ Level\ 1.app/_CodeSignature
$ /usr/bin/codesign --force --sign 8004380F331DCA22CC1B47FB1A805890AE41C938  Payload/UnCrackable\ Level\ 1.app/FridaGadget.dylib
Payload/UnCrackable Level 1.app/FridaGadget.dylib: replacing existing signature
```

entitlements.plist is the file you created earlier, for your empty iOS project.

```shell
$ /usr/bin/codesign --force --sign 8004380F331DCA22CC1B47FB1A805890AE41C938 --entitlements entitlements.plist Payload/UnCrackable\ Level\ 1.app/UnCrackable\ Level\ 1
Payload/UnCrackable Level 1.app/UnCrackable Level 1: replacing existing signature
```

#### Installing and Running an App

Now you should be ready to run the modified app. Deploy and run the app on the device as follows:

```shell
$ ios-deploy --debug --bundle Payload/UnCrackable\ Level\ 1.app/
```

If everything went well, the app should launch in debugging mode with lldb attached. Frida should now be able to attach to the app as well. You can verify this with the frida-ps command:

```shell
$ frida-ps -U
PID  Name
---  ------
499  Gadget
```

![Frida on non-JB device](Images/Chapters/0x06b/fridaStockiOS.png "Frida on non-JB device")

#### Troubleshooting

When something goes wrong (and it usually does), mismatches between the provisioning profile and code signing header are the most likely causes. Reading the [official documentation](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html "Maintaining Provisioning Profiles") helps you understand the code signing process. Apple's [entitlement troubleshooting page](https://developer.apple.com/library/content/technotes/tn2415/_index.html "Entitlements Troubleshooting ") is also a useful resource.

#### Automated Repackaging with Objection

[Objection](https://github.com/sensepost/objection "Objection") is a mobile runtime exploration toolkit based on [Frida](http://www.frida.re). One of the best things about Objection is that it works even with non-jailbroken devices. It does this by automating the process of app repackaging with `FridaGadget.dylib`.
We won’t cover Objection in detail in this guide, but you can find exhaustive documentation on the official [wiki pages](https://github.com/sensepost/objection/wiki "Objection - Documentation") and also [how to repackage an IPA](https://github.com/sensepost/objection/wiki/Patching-iOS-Applications "Patching iOS Apps").

### Method Tracing with Frida
Intercepting Objective-C methods is a useful iOS security testing technique. For example, you may be interested in data storage operations or network requests. In the following example, we'll write a simple tracer for logging HTTP(S) requests made via iOS standard HTTP APIs. We'll also show you how to inject the tracer into the Safari web browser.

In the following examples, we'll assume that you are working on a jailbroken device. If that's not the case, you need to first follow the steps outlined in the previous section to repackage the Safari app. 

Frida comes with `frida-trace`, a ready-made function tracing tool. `frida-trace` accepts Objective-C methods via the `-m` flag. You can pass it wildcards as well—given `-[NSURL *]`, for example, frida-trace will automatically install hooks on all `NSURL` class selectors. We'll use this to get a rough idea about which library functions Safari calls when the user opens a URL. 

Run Safari on the device and make sure the device is connected via USB. Then start `frida-trace` as follows:

```shell
$ frida-trace -U -m "-[NSURL *]" Safari
Instrumenting functions...                                              
-[NSURL isMusicStoreURL]: Loaded handler at "/Users/berndt/Desktop/__handlers__/__NSURL_isMusicStoreURL_.js"
-[NSURL isAppStoreURL]: Loaded handler at "/Users/berndt/Desktop/__handlers__/__NSURL_isAppStoreURL_.js"
(...)
Started tracing 248 functions. Press Ctrl+C to stop.     
```

Next, navigate to a new website in Safari. You should see traced function calls on the `frida-trace` console. Note that the `initWithURL:` method is called to initialize a new URL request object.

```shell
           /* TID 0xc07 */
  20313 ms  -[NSURLRequest _initWithCFURLRequest:0x1043bca30 ]
 20313 ms  -[NSURLRequest URL]
(...)
 21324 ms  -[NSURLRequest initWithURL:0x106388b00 ]
 21324 ms     | -[NSURLRequest initWithURL:0x106388b00 cachePolicy:0x0 timeoutInterval:0x106388b80 
```

We can look up the declaration of this method on the [Apple Developer Website](https://developer.apple.com/documentation/foundation/nsbundle/1409352-initwithurl?language=objc "Apple Developer Website - initWithURL Instance Method"):

```objective-c
- (instancetype)initWithURL:(NSURL *)url;
```

The method is called with a single argument of type `NSURL`. According to the [documentation](https://developer.apple.com/documentation/foundation/nsurl?language=objc "Apple Developer Website - NSURL class"), the `NSRURL` class has a property called `absoluteString` whose value should be the absolute URL represented by the `NSURL` object.

We now have all the information we need to write a Frida script that intercepts the `initWithURL:` method and prints the URL passed to the method. The full script is below. Make sure you read the code and inline comments to understand what's going on.


```python
import sys
import frida


// JavaScript to be injected
frida_code = """

	// Obtain a reference to the initWithURL: method of the NSURLRequest class
    var URL = ObjC.classes.NSURLRequest["- initWithURL:];
 
    // Intercept the method
    Interceptor.attach(URL.implementation, {
      onEnter: function(args) {

      	// We should always initialize an autorelease pool before interacting with Objective-C APIs

        var pool = ObjC.classes.NSAutoreleasePool.alloc().init();

        var NSString = ObjC.classes.NSString;

        // Obtain a reference to the NSLog function, and use it to print the URL value
        // args[2] refers to the first method argument (NSURL *url)

        var NSLog = new NativeFunction(Module.findExportByName('Foundation', 'NSLog'), 'void', ['pointer', '...']);

        NSLog(args[2].absoluteString_());

        pool.release();
      }
    });


"""

process = frida.get_usb_device().attach("Safari")
script = process.create_script(frida_code)
script.on('message', message_callback)
script.load()

sys.stdin.read()
```

Start Safari on the iOS device. Run the above Python script on your connected host and open the device log (we'll explain how to open them in the following section). Try opening a new URL in Safari; you should see Frida's output in the logs.

<img src="Images/Chapters/0x06b/frida-xcode-log.jpg" width="500px"/>

Of course, this example illustrates only one of the things you can do with Frida. To unlock the tool's full potential, you should learn to use its JavaScript API. The documentation section of the Frida website has a [tutorial](https://www.frida.re/docs/ios/) and [examples](https://www.frida.re/docs/examples/ios/) of Frida usage on iOS. 

[Frida JavaScript API reference](https://www.frida.re/docs/javascript-api/)


### Monitoring Console Logs

Many apps log informative (and potentially sensitive) messages to the console log. The log also contains crash reports and other useful information. You can collect console logs through the Xcode "Devices" window as follows:

1. Launch Xcode.
2. Connect your device to your host computer.
3. Choose Devices from the window menu.
4. Click on your connected iOS device in the left section of the Devices window.
5. Reproduce the problem.
6. Click the triangle-in-a-box toggle located in the lower left-hand corner of the Devices window's right section to view the console log's contents.

To save the console output to a text file, go to the bottom right and click the circular downward-pointing-arrow icon.

<img src="Images/Chapters/0x06b/device_console.jpg" width="500px"/>
- *Monitoring console logs through Xcode*

### Setting up a Web Proxy with Burp Suite

Burp Suite is an integrated platform for security testing mobile and web applications. Its tools work together seamlessly to support the entire testing process, from initial mapping and analysis of attack surfaces to finding and exploiting security vulnerabilities. Burp proxy operates as a web proxy server for Burp Suite, which is positioned as a man-in-the-middle between the browser and web server(s). Burp Suite allows you to intercept, inspect, and modify incoming and outgoing raw HTTP traffic.

Setting up Burp to proxy your traffic is pretty straightforward. We assume that you have an iOS device and workstation connected to a Wi-Fi network that permits client-to-client traffic. If client-to-client traffic is not permitted, you can use usbmuxd to connect to Burp via USB.

Portswigger provides a good [tutorial on setting up an iOS Device to work with Burp](https://support.portswigger.net/customer/portal/articles/1841108-configuring-an-ios-device-to-work-with-burp "Configuring an iOS Device to Work With Burp") and a [tutorial on installing Burp's CA certificate to an iOS device ](https://support.portswigger.net/customer/portal/articles/1841109-installing-burp-s-ca-certificate-in-an-ios-device "Installing Burp's CA Certificate in an iOS Device").

#### Bypassing Certificate Pinning

`[SSL Kill Switch 2](https://github.com/nabla-c0d3/ssl-kill-switch2 "SSL Kill Switch 2")` is one means of disabling certificate pinning. It can be installed via the Cydia store. It will hook on all high-level API calls and bypass certificate pinning.

The Burp Suite app "[Mobile Assistant](https://portswigger.net/burp/help/mobile_testing_using_mobile_assistant.html "Using Burp Suite Mobile Assistant")" can also be used to bypass certificate pinning.

In some cases, certificate pinning is tricky to bypass. Look for the following when you can access the source code and recompile the app:

- the API calls `NSURLSession`, `CFStream`, and `AFNetworking`
- methods/strings containing words like 'pinning', 'X509', 'Certificate', etc.

If you don't have access to the source, you can try binary patching or runtime manipulation:

- If OpenSSL certificate pinning is implemented, you can try [binary patching](https://www.nccgroup.trust/us/about-us/newsroom-and-events/blog/2015/january/bypassing-openssl-certificate-pinning-in-ios-apps/ "Bypassing OpenSSL Certificate Pinning in iOS Apps").
- Applications written with Apache Cordova or Adobe Phonegap use a lot of callbacks. Look for the callback function that's called on success and manually call it with Cycript.
- Sometimes, the certificate is a file in the application bundle. Replacing the certificate with Burp's certificate may be sufficient, but beware the certificate's SHA sum. If it's hardcoded into the binary, you must replace it too!

Certificate pinning is a good security practice and should be used for all applications that handle sensitive information. [EFF's Observatory](https://www.eff.org/pl/observatory) lists the root and intermediate CAs that major operating systems automatically trust. Please refer to the [map of the roughly 650 organizations that are Certificate Authorities Mozilla or Microsoft trust (directly or indirectly)](https://www.eff.org/files/colour_map_of_CAs.pdf "Map of the 650-odd organizations that function as Certificate Authorities trusted (directly or indirectly) by Mozilla or Microsoft"). Use certificate pinning if you don't trust at least one of these CAs.

If you want to get more details about white box testing and usual code patterns, refer to "iOS Application Security" by David Thiel. It contains descriptions and code snippets illustrating the most common certificate pinning techniques.

To get more information about testing transport security, please refer to the section "Testing Network Communication."

### Network Monitoring/Sniffing

You can remotely sniff all traffic in real-time on iOS by [creating a Remote Virtual Interface](https://stackoverflow.com/questions/9555403/capturing-mobile-phone-traffic-on-wireshark/33175819#33175819 "Wireshark + OSX + iOS") for your iOS device. First make sure you have Wireshark installed on your macOS machine.

1. Connect your iOS device to your macOS machine via USB.
2. Make sure that your iOS device and your macOS machine are connected to the same network.
3. Open "Terminal" on macOS and enter the following command: `$ rvictl -s x`, where x is the UDID of your iOS device. You can find the [UDID of your iOS device via iTunes](http://www.iclarified.com/52179/how-to-find-your-iphones-udid "How to Find Your iPhone's UDID").
4. Launch Wireshark and select "rvi0" as the capture interface.
5. Filter the traffic in Wireshark to display what you want to monitor (for example, all HTTP traffic sent/received via the IP address 192.168.1.1).

```shell 
ip.addr == 192.168.1.1 && http
```

