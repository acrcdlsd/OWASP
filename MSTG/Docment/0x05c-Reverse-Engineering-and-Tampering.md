## Androidでの改ざんとリバースエンジニアリング
Androidの開放性は, リバースエンジニアリングにとって好都合な環境である. 次の章で, プロセスとして, AndroidリバーシングとOS固有のツールのいくつかの特性について注目していく.

Androidは, "他の"モバイルOSでは利用可能ではない大きな利点をリバースエンジニアリングにもたらす. Androidはオープンソースであるため, Android Open Source Project(AOSP)でソースコードを調査し, 思い通りにOSや標準ツールを改ざんすることができる. 標準の小売りデバイスでさえ, 苦労することなく開発者モードを有効にしたり, アプリを再度ロードするようなことが可能である. SDKを搭載した強力なツールから幅広く利用可能なリバースエンジニアリングツールまで, 生活によりゆとりを持たせるために慎重に行うべきことがたくさん存在する.

しかしながら, Android固有の課題もいくつか存在する. 例えば, Javaのバイトコートやネイティブコード両方を処理する必要がある. Javaネイティブインタフェイス(JNI)は, リバースエンジニアリングを行う人を混乱させるために, 時々意図的に使用される(公平のために言うと, パフォーマンスの向上やレガシーコードのサポートなど, JNIの使用には合理的な理由が存在する). 開発者はデータや機能を"隠す"ためにネイティブレイヤを時々使用し, 実行が頻繁に二つのレイヤの間を飛び越えるアプリを構築するかもしれない.  

最低, JavaベースのAndroid環境とAndroidに基づくLinux OSやカーネル両方の実用的な知識が必要である. また, Java仮想マシンで実行するネイティブコードとバイトコード両方に対応するための適切なツールセットも必要である.

ここで留意すべきは, 以下のセクションで様々なリバースエンジニアリング技術を実演するための例として, [OWASP Mobile Testing Guide Crackmes](https://github.com/OWASP/owasp-mstg/blob/master/Crackmes/)を使用するため, 一部かつ完全なネタバレを期待しておくように. 我々は, 読み進める前にあなた自身の挑戦で試しにやってみることを推奨する.

### 必要なもの
以下のものがシステムにインストールされていることを確認する.
- 最新のSDKツールとSDK Platform-toolsパッケージ. これらのパッケージは, Android Debugging Bridge(ADB)クライアントと、Androidプラットフォームとインタフェイスをとる他のツールを搭載している.
- Android NDK. これは, 異なるアーキテクチャのネイティブコードをクロスコンパイルするためのあらかじめビルドされたツールチェインを含むNative Development Kitのことである.

SDKとNDKに加えて, Javaバイトコードをより人間が読み取れるようにするための何かが必要である. 幸いにもほとんどの場合, JavaデコンパイラはAndroidバイトコードをうまく処理する. 人気のあるフリーのデコンパイラには, [JD](http://jd.benow.ca), [JAD](http://www.javadecompilers.com/jad), [Proycon](http://proycon.com/en/), [CFR](http://www.benf.org/other/cfr/)がある. 便宜上, [apkx wrapper script](https://github.com/b-mueller/apkx)に先述したデコンパイラのうちのいくつかをパックした. このスクリプトは, リリースのAPKファイルからJavaコードを抽出するプロセスを完全に自動化し, 異なるバックエンドで試すことを容易にする(以下の例の一部でも使用するだろう).

他のツールは好みや予算に関する問題である. 異なる長所と短所を持つ大量のフリーもしくは商用の逆アセンブラ, デコンパイラ, フレームワークが存在し, この本ではそれらのいくつかをカバーしている.  

#### Android SDKのセットアップ
ローカルのAndroid SDKのインストールは, Android Studioを通して管理する. Android Studioで空のプロジェクトを生成し, SDK Manager GUIを開くために"Tools->Android->SDK Manager"を選択する. "SDK Platforms"タブでは, 多様なAPIレベルのSDKをインストールすることができる.  最近のAPIレベルを以下に示す.

- API 21: Android 5.0
- API 22: Android 5.1
- API 23: Android 6.0
- API 24: Android 7.0
- API 25: Android 7.1
- API 26: Android O Developer Preview

![sdk manager](Images/Chapters/0x05c/sdk_manager.jpg)

インストールされたSDKは以下のパスに存在する.

```
Windows:

C:\Users\<username>\AppData\Local\Android\sdk

MacOS:

/Users/<username>/Library/Android/sdk
```

注意: Linuxでは, SDKのディレクトリを選択する必要がある. 一般的には, `/opt`, `/srv`, `/usr/local`のいずれかが選択される.

#### Android NDKのセットアップ
Android NDKは, ネイティブコンパイラとツールチェインのあらかじめビルドされたバージョンを含んでいる. GCCとClang両方のコンパイラは, 伝統的にサポートされているが, GCCのアクティブサポートはNDKのバージョン14で終了した. デバイスアーキテクチャとホストOSは, 適切なバージョンを決定する. あらかじめビルドされたツールチェインは, NDKの`toolchains`ディレクトリにあり, そこは各アーキテクチャのサブディレクトリを1つ含んでいる.

| アーキテクチャ | ツールチェイン名 |
|-------------- | ---------------|
|ARM-based|arm-linux-androideabi-&lt;gcc-version&gt;|
|x86-based|x86-&lt;gcc-version&gt;|
|MIPS-based|mipsel-linux-android-&lt;gcc-version&gt;|
|ARM64-based|aarch64-linux-android-&lt;gcc-version&gt;|
|X86-64-based|x86_64-&lt;gcc-version&gt;|
|MIPS64-based|mips64el-linux-android-&lt;gcc-version&gt;|

正しいアーキテクチャを選択する傍ら, ターゲットとしたいネイティブAPIレベルに的確なsysrootを指定する必要がある. sysrootは, ターゲットに関するシステムヘッダとライブラリを含むディレクトリである. ネイティブAPIは, AndroidのAPIレベルによって変わってくる. 各APIレベルに対して考えられるsysrootは, `$NDK/platforms/`に存在する. 各APIレベルのディレクトリは, 様々なCPUやアーキテクチャのサブディレクトリを含んでいる.

ビルドシステムをセットアップする1つの可能性は, 環境変数としてコンパイラのパスと必要なフラグをエクスポートすることである. しかしながら, 状況をより簡単にするために, NDKによっていわゆるスタンドアロンのツールチェイン、すなわち必要とされる設定を組み込んだ「一時的な」ツールチェインを生成することができる.

スタンドアロンのツールチェインを設定するために, [NDKの最新の安定バージョン](https://developer.android.com/ndk/downloads/index.hml#stable-downloads)をダウンロードする必要がある. ZIPファイルを抽出し, NDKのルートディレクトリに遷移し, 以下のコマンドを実行する.

```bash
$ ./build/tools/make_standalone_toolchain.py --arch arm --api 24 --install-dir /tmp/android-7-toolchain
```

上記コマンドは, `/tmp/android-7-toolchain`ディレクトリにAndroid 7.0用のスタンドアロンのツールチェインを生成する. 便宜上, ツールチェインのディレクトリを指し示す環境変数をエクスポートすることができる(例ではこれを使用する). 以下のコマンドを実行するか, `.bash_profile`や他のスタートアップスクリプトに追加する.

```bash
$  export TOOLCHAIN=/tmp/android-7-toolchain
```

### 開発者モードの有効化
ADBデバッグインタフェイスを利用するために, デバイスでUSBデバッグを有効にする必要がある. Android 4.2以降, 設定アプリの「開発者オプション」のサブメニューはデフォルトで非表示となっている. これを表示するためには, "About phone(端末情報)"の"Build number(ビルド番号)"セクションを7回タップする. ビルド番号フィールドの位置はデバイスによって若干異なることに注意する必要がある(例えば, LGの端末では"About phone -> Software information"の下にある). この操作が完了すると, 設定メニューの下部に"Developer options(開発者オプション)"が表示される. 開発者オプションが有効になると, "USB debugging(USBデバッグ)"スイッチでデバッグを有効にすることができる.  

USBデバッグが有効になると, 以下のコマンドを用いて接続されたデバイスを表示することができる.

```bash
$ adb devices
List of devices attached
BAZ5ORFARKOZYDFA	device
```

### フリーのリバースエンジニアリング環境の構築
少ない努力で, リーズナブルなGUIベースのリバースエンジニアリング環境をフリーで構築することができる.

デコンパイルされたソースを検索するために, [IntelliJ](https://www.jetbrains.com/idea/)を推奨する. これは, コードのブラウジングに優れた比較的軽量なIDEであり, 基本的なコンパイルされたアプリのオンデバイスデバッグを可能にする. しかしながら, 魅力がなく, 動作が重く, 複雑なものを好むのであれば, Eclipseがぴったりだろう(著者の個人的偏見に基づいている).

Javaの代わりにSmaliを見ることが問題なければ, [Intellijのsmalideaプラグイン](https://github.com/JesusFreke/smali/wiki/smalidea)を使用することができる. Smalideaは, バイトコードと識別子の改名を通してシングルステップをサポートし, 命名されていないレジスタを監視するため, JDとIntelliJの設定の組み合わせよりもより一層強力である.

[APKTool](https://ibotpeaches.github.io/Apktool/)は, APKアーカイブから直接リソースを抽出し, 逆アセンブルすることができ, JavaバイトコードをSmali形式に逆アセンブルすることができる評判の良いフリーツールである(Smali/Baksmaliは, Dex形式のアセンブラ/逆アセンブラである. また, Smali/Baksmaliは"アセンブラ/逆アセンブラ"のアイスランド語でもある). APKToolを使用することで, パッケージを逆アセンブルすることができる. これはManifestにパッチを当て, 変更を適用するのに便利である.

 [Radare2](https://www.radare.org/)や[Angr](http://angr.io/)などのオープンソースのリバースエンジニアリングフレームワークを用いて, より複雑なタスク(プログラム解析や自動化された難読化の解除)を成し遂げることができる. 本ガイドを通して, これらのフリーツールやフレームワークの多くの使用例を見つけることができる.

#### 商用ツール
完全無料のセットアップが可能とはいえ, 商用ツールへの投資を考えるべきである. これらのツールの主な利点は, 利便性である. たとえば, 良いGUI, 多くの自動化, エンドユーザサポートを備えている. リバースエンジニアリングで生計を立てる場合, それらを用いることで多くの時間を節約できるだろう.

##### JEB
商用のデコンパイラである[JEB](https://www.pnfsoftware.com/)には, Androidアプリの静的解析および動的解析のために必要な機能すべてを一体化パッケージに含んでいる. JEBそこそこ信頼でき, かつ迅速なサポートが含まれている. また, 組み込みのデバッガを持っており, 効率の良い作業を可能にする. たとえば, デコンパイルされた(なおかつ注釈付きの)ソースに直接ブレークポイントを設定することは, 特にProGuardで難読化されたバイトコードの場合はすこぶる有益である. もちろん, このような利便性を得るのにお金がかからないわけがないし, 今やJEBはサブスクリプションベースのライセンスを通して提供されるため, 使用するためには毎月使用料を支払う必要がある.

##### IDA Pro
[IDA Pro](https://www.hex-rays.com/products/ida/)は, ARM, MIPS, Javaバイトコード, もちろんIntel ELFバイナリと互換性がある. また, Javaアプリケーションとネイティブプロセス両方のデバッガを搭載している. 強力なスクリプト, 逆アセンブリ, 拡張機能を備えたIDA Proは, ネイティブプログラムとライブラリの静的解析に最適である. しかしながら, Javaコードのために提供されている静的解析機能は, かなり基本的なものである. 例えば, 逆アセンブリされたSmaliを得ることはできるが, あまり多くはない. パッケージやクラスストラクチャを検索することはできず, 実行できないアクション(クラスの改名など)も存在する, そしてそれは, より複雑なJavaアプリを扱う作業を退屈にする可能性がある.

### リバースエンジニアリング
リバースエンジニアリングは, どのように動作しているのかを解明するためにアプリを分析するプロセスである. コンパイルされたアプリの検査(静的解析), 実行中のアプリの監視(動的解析), または両方を組み合わせることによって行うことができる.

#### Javaコードの静的解析
いくつかの厄介な, ツールを破壊する耐デコンパイル技術が適用されていない限り, Javaバイトコードは多くの問題なく, ソースコードに戻すことができる. 以下の例で, Androidレベル1のUnCrackableアプリを使用するので, ダウンロードしていない場合はダウンロードすること. まず初めに, デバイスやエミュレータにアプリをインストールし, crackmeが何であるかを確認するためにアプリを実行する.

```
$ wget https://github.com/OWASP/owasp-mstg/raw/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk
$ adb install UnCrackable-Level1.apk
```

<img src="Images/Chapters/0x05c/crackme-2.jpg" width="350px"/>

何らかのシークレットコードが見つけることを期待しているように思える.

我々は, アプリ内のどこかに格納された秘密の文字列を探しているので, 次のステップは中を見ることである. 初めに, APKファイルをunzipし, コンテンツを確認する.

```
$ unzip UnCrackable-Level1.apk -d UnCrackable-Level1
Archive:  UnCrackable-Level1.apk
  inflating: UnCrackable-Level1/AndroidManifest.xml  
  inflating: UnCrackable-Level1/res/layout/activity_main.xml  
  inflating: UnCrackable-Level1/res/menu/menu_main.xml  
 extracting: UnCrackable-Level1/res/mipmap-hdpi-v4/ic_launcher.png  
 extracting: UnCrackable-Level1/res/mipmap-mdpi-v4/ic_launcher.png  
 extracting: UnCrackable-Level1/res/mipmap-xhdpi-v4/ic_launcher.png  
 extracting: UnCrackable-Level1/res/mipmap-xxhdpi-v4/ic_launcher.png  
 extracting: UnCrackable-Level1/res/mipmap-xxxhdpi-v4/ic_launcher.png  
 extracting: UnCrackable-Level1/resources.arsc  
  inflating: UnCrackable-Level1/classes.dex  
  inflating: UnCrackable-Level1/META-INF/MANIFEST.MF  
  inflating: UnCrackable-Level1/META-INF/CERT.SF  
  inflating: UnCrackable-Level1/META-INF/CERT.RSA  

```

標準の設定では, Javaバイトコードとアプリデータのすべてはアプリルートディレクトリの`class.dex`ファイルにある. ファイルは, JavaプログラムをパックするAndroid特有の方法であるDalvik実行形式(DEX)に一致する. ほとんどのJavaデコンパイラは, 入力としてrplain classやJARを受け取るので, 初めにclass.dexファイルをJARに変換する必要がある. `dex2jar`や`enjarify`を使用することでこれを行うことができる.

JARファイルが手に入ると, 任意のフリーのデコンパイラを使用してJavaコードを生成することができる. この例では, CFRデコンパイラを使用している. CFRは活発に開発されており, 最新のリリースは著者のWebサイトで入手可能である. CFRはMITライセンスの下でリリースされたため, ソースコードは入手できないが, 自由に使用することができる.

CFRを実行するもっとも簡単な方法は, `dex2jar`をパッケージ化し, 抽出, 変換, デコンパイルを自動化している`apkx`を通すことである. インストールは以下を参照のこと.

```
$ git clone https://github.com/b-mueller/apkx
$ cd apkx
$ sudo ./install.sh
```

これは, `apkx`を`/usr/local/bin`にコピーしなければならない. `UnCrackable-Level1.apk`で実行する必要がある.

```bash
$ apkx UnCrackable-Level1.apk
Extracting UnCrackable-Level1.apk to UnCrackable-Level1
Converting: classes.dex -> classes.jar (dex2jar)
dex2jar UnCrackable-Level1/classes.dex -> UnCrackable-Level1/classes.jar
Decompiling to UnCrackable-Level1/src (cfr)
```

 今, `Uncrackable-Level1/src`ディレクトリでデコンパイルされたソースを発見できるだろう. ソースを見るために, シンプルなテキストエディタ(出来れば構文をハイライトしてくれるもの)が良いが, Java IDE内にコードをロードすることはナビゲーションをより簡単にする. デバイス上でのデバッグ機能を提供するIntelliJにコードをインポートしてみよう.

 IntelliJを開き, "New Project"ダイアログの左側のタブのプロジェクトタイプで"Android"を選択する. アプリケーション名に"Uncrackable1"を, 会社名に"vantagepoint.sg"を入力する. これは, もとのパッケージ名と一致する"sg.vantagepoint.uncrackable1"の結果となる. Intellijは正しいプロセスを識別するのにパッケージ名を使用するので, 後で実行中のアプリにデバッガを配置したい場合, 一致するパッケージ名を使用することは重要である.

 <img src="Images/Chapters/0x05c/intellij_new_project.jpg" width="650px" />

次のダイアログで, 任意のAPI番号を選択する. なお, 実際はそのプロジェクトをコンパイルしたくないので, 番号はどうでもよい. "next"をクリックし, "Add no Activity"を選択した後, "finish"をクリックする.

プロジェクトを作成すると, 左側にある"1: Project"を展開して, `app/src/main/java`フォルダに遷移する. IntelliJによって作成されたデフォルトのパッケージ"sg.vantagepoint.uncrackable1"を右クリックして削除する.

<img src="Images/Chapters/0x05c/delete_package.jpg" width="400px"/>

今, `Uncrackable-Level1/src`ディレクトリをファイルブラウザで開き, IntelliJプロジェクトビューの空の`Java`フォルダに`sg`ディレクトリをドラッグする(移動する代わりにフォルダをコピーするには, "alt"キーをホールドする).

<img src="Images/Chapters/0x05c/drag_code.jpg" width="700px" />

結果的に, アプリが構築された元のAndroid Studioプロジェクトに似た構造になるだろう,

<img src="Images/Chapters/0x05c/final_structure.jpg" width="400px"/>

IntelliJがコードをインデックスするとすぐに, まるで他のJavaプロジェクトを見ているかのようにそれを見ることができる. ここで留意すべきは, デコンパイルされたパッケージ, クラス, メソッドのほとんどは, 1文字の変わった名前が付けられていることである. これは, ビルド時にProGuardによってバイトコードが"縮小"されたためである. これは, バイトコードを少し読みにくくする難読化の基本的なタイプであるが, このような極めて単純なアプリでは, 大した頭痛の原因とはならない. しかしながら, より複雑なアプリを解析する場合は, 非常に迷惑となる可能性がある.

難読化されたコードを解析する場合, 付随するようにクラス名, メソッド名, その他の識別子に注釈をつけることが, グッドプラクティスである. `sg.vantagepoint.a`パッケージの`MainActivity`クラスを開く. "verify"ボタンをタップすると`verify`メソッドが呼び出される. このメソッドは, boolean(ブール値)を返す`a.a`と呼ばれる静的メソッドにユーザインプットを引き渡す. `a.a`がユーザインプットを検証することはもっともらしいので, これを反映するためにコードをリファクタリングする.

![User Input Check](Images/Chapters/0x05c/check_input.jpg)

クラス名(`a.a`の初めの`a`)を右クリックし, ドロップダウンメニューから(もしくはShift-F6を押して)Refactor->Renameを選択する. 今までのクラスに関して知っていることを与えられた, より意味のあるものにクラス名を変更する. たとえば, "Validator"と名付けることができる(いつでも後で名前を変更可能). 今, `a.a`は`Validator.a`になった. 静的メソッドを`a`から`check_input`に改名するために, 同じ手続きに従う.

![Refactored class and method names](Images/Chapters/0x05c/refactored.jpg)

Congratulations! たった今, 静的解析の基本を学習した. 解析されたプログラムに関する理論を理論化, 注釈付け, 少しずつ修正することは, 完全に理解するまで, もしくは少なくとも達成したいと思っていることは何でも十分に理解するまでである.

次に, `check_input`メソッドでCtrl+(Macの場合はCommand+)をクリックする. これはメソッド定義に連れていく. デコンパイルされたメソッドは以下のように見える.

```
hogehoge
```

従って, 16進エンコードされた暗号鍵(16 HEXバイト = 126bitで一般的な鍵長)のように見えるなにかと一緒に`sg.vantagepoint.a.a`(再び, すべて`a`と名付けられている)パッケージの`a`関数に引き渡されるBase64でエンコードされた文字列がある. 厳密に何がこの特定のすべきことをするのだろうか？調査するためにCtrlをクリックする.

```
hogehoge
```

今, 良い線を行っている. 単に標準のAES-ECBである. `check_input`の`arrby1`に格納されているBase64の文字列は暗号文であるように見える. それは128bitのAESで復号され, ユーザインプットと比較される. おまけのタスクとして, 抽出された暗号文を解読し, 秘密の値を見つけてみよう.

解読された文字列を入手するより早い方法は, 動的解析を加えることである. 方法を示すためにUnCrackable Level 1を後で再訪するので, まだプロジェクトを消さないように！

 #### ネイティブコードの静的解析
 DalvicとARTのどちらも, JavaコードがC/C++で書かれたネイティブコードとやり取りする方法を定義するJava Native Interface(JNI)をサポートしている. 他のLinuxベースのオペレーティングシステム同様に, ネイティブコードはELF動的ライブラリ("\*.so")の中にパッケージ化されている. これは, Androidアプリが`System.load`メソッドを通して実行時にロードする.

Android JNI機能は, Linux ELFライブラリにコンパイルされたネイティブコードで記述されている. それは標準的なLinuxフェアである. しかしながら, 幅広く使用されているCライブラリ(glibcなど)に頼る代わりに, Androidバイナリは, [Bionic](https://github.com/android/platform_bionic)と呼ばれるカスタムlibcに対してビルドされている. Bionicは, システムプロパティとロギングのような重要なAndroid特有のサービスのためのサポートを追加しており, それは完全にPOSIX互換というわけではない.

OWASP MSTGリポジトリからHelloWorld-JNI.apkをダウンロードすること. エミュレータやAndroidデバイスにインストールして実行するかは任意である.

```
$ wget HelloWorld-JNI.apk
$ adb install HelloWorld-JNI.apk
```

このアプリは必ずしも超大作というわけではない. "Hello from C++"というテキストのラベルを表示するだけである. これは, C/C++で新たなプロジェクトを作成したときにAndroidがデフォルトで生成するアプリである...が, JNI呼び出しの基本原理を示すには十分である.

<img src="Images/Chapters/0x05c/helloworld.jpg" width="300px"/>

`apkx`でAPKをデコンパイルする. これは, `HelloWorld/src`ディレクトリの中にソースコードを抽出する.

```
hogehoge
```

MainActivityは, `MainActivity.java`ファイルに含まれている. "Hello World"テキストビューは`onCreate()`メソッドに存在している.

```
hogehoge
```

上記例の一番下にある`public native String stringFromJNI`の宣言に注意すること. "native"というキーワードは, メソッドがネイティブ言語で実装されていることをJavaデコンパイラに示している. 対応する関数は実行中に解決されるが, 期待されているシグネチャでグローバルシンボルをエクスポートするネイティブライブラリがロードされた場合に限る(シグネチャは, パッケージ名, クラス名, メソッド名を含む). この例では, この要件は以下に示すC/C++の関数で満たされる.

```
hogehoge
```

ところで, この関数のネイティブ実装はどこにあるのだろうか？APKアーカイブの`lib`ディレクトリの中を確認すると, 異なるプロセッサアーキテクチャにちなんで名付けられた8つのサブディレクトリが目に入るだろう. これらのディレクトリのそれぞれは, 問題となっているプロセッサアーキテクチャのためにコンパイルされた`libnative-lib.so`と呼ばれるネイティブライブラリのバージョンを含んでいる. `System.loadLibrary`が呼び出されたとき, ローダーはアプリが実行しているデバイスに基づいて正確なバージョンを選択する.

<img src="Images/Chapters/0x05c/archs.jpg" width="300px" />

上記の命名規則に従い,
