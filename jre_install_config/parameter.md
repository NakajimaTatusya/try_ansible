# 構成ファイルを使用したJDKおよびJREのインストール(Windows)

## 構成ファイルの使用

* installer /s INSTALLCFG=confiration_file_path /L logfile_path.log

## 構成ファイルのオプション

[参照先](https://docs.oracle.com/javase/jp/8/docs/technotes/guides/install/config.html)

| オプション | ＯＳ | 値 | 説明 | 規定値 | ランタイム構成ファイルに保存するか？ |
| :--- | :--- | :--- | :--- | :--- | :--- |
| AUTO_UPDATE | Windows, macOS | Enable,Disable | 自動更新機能を有効にします。 | Enable | Yes |
| DEPLOYMENT_RULE_SET | Windows、macOS、Linux、Solaris | パス | 組織のデプロイメント・ルール・セットのパスおよびファイル名を指定します。[Java Platform, Standard Editionデプロイメント・ガイドのデプロイメント・ルール・セット](https://docs.oracle.com/javase/jp/8/docs/technotes/guides/deploy/index.html)を参照してください。 | nothing | Yes |
| EULA | Windows | Enable,Disable | JavaアプレットまたはJava Web Startアプリケーションが起動される場合、ユーザーにエンド・ユーザー・ライセンス契約(EULA)の承諾を求めます。 | Disable | Yes |
| INSTALL_SILENT | Windows | Enable,Disable | サイレント(インタラクティブでない)インストールを実行します。 | Disable | Yes |
| INSTALLDIR | Windows | パス | ファイルのインストール先のフォルダ/ディレクトリを指定します。Windowsの場合、これはファミリの最初のインストール時のみ機能します。LinuxおよびSolarisでは、この目的にはオペレーティング・システム・ツールを使用するため、インストール・ディレクトリの再配置はオペレーティング・システム・ツールによって処理されます(例: rpm --prefix=path)。※環境変数の指定ができなかった。 | オペレーティング・システムのデフォルト・パス | No |
| INSTALLDIRPUBJRE | Windows | パス | JDKをインストールする際に、このオプションでは、パブリックJREがインストールされるフォルダを指定します。このオプションを指定する場合は、INSTALLDIRオプションを使用して、JDKがインストールされるフォルダも指定する必要があります。 | オペレーティング・システムのデフォルト・パス | No |
| NOSTARTMENU | Windows | Enable,Disable | インストーラがJavaの起動項目を設定せずにJDKまたはJREをインストールすることを指定します。 | Disable | No |
| REMOVEOUTOFDATEJRES | Windows | 0, 1 | 注意: オンラインおよびオフラインのインストーラ(ファイルおよびラッパー)のみに適用されます。JREのインストール時に既存の古いJREをアンインストールするようにします。REMOVEOUTOFDATEJRES=0を使用すると、すべての古いJavaバージョンがシステムに残ります。REMOVEOUTOFDATEJRES=1を使用すると、すべての古いJavaバージョンがシステムから削除されます。たとえば、jre1.8.0_60.exe /s REMOVEOUTOFDATEJRES=1を実行すると、セキュリティ・ベースラインを下回っているすべてのJREが削除されます。セキュリティ・ベースラインを上回っているJREはアンインストールされません。 | 0 | No |
| STATIC | Windows | Enable,Disable | 静的なインストールを実行します([静的なインストール](https://docs.oracle.com/javase/jp/8/docs/technotes/guides/install/windows_installer_options.html#static_installation)を参照してください)。 | Disable | No |
| USAGETRACKER_CFG | Windows、macOS、Linux、Solaris | パス | Java Usage Trackerのプロパティ・ファイルのパスおよびファイル名を指定します。[『Java Usage Trackerガイド』](https://docs.oracle.com/javacomponents/usage-tracker/overview/toc.htm)を参照してください。 | nothing | Yes |
| WEB_ANALYTICS | Windows | Enable,Disable | インストーラによるインストール関連の統計のOracleサーバーへの送信を有効または無効にします。 | Enable | Yes |
| WEB_JAVA_SECURITY_LEVEL | Windows、macOS、Linux、Solaris | H (高), VH (非常に高) | ブラウザまたはJava Web Startで実行中のJavaアプリケーションのインストールのセキュリティ・レベルを構成します。 | H | No |
| WEB_JAVA | Windows、macOS、Linux、Solaris | Enable,Disable | ダウンロード済のJavaアプリケーションがWebブラウザまたはJava Web Startでの実行を許可される/されないように、インストールを構成します。 | Enable | No |
