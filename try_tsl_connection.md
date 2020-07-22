# Set WinRM TSL Authenticatoin (author:TatsuyaNakajima)

* Windows10 ＋ WSL1（Ubuntu18.x）で疎通確認済み
* Windows Server とは検証していませぬ。（お願いします。
* **【注意】以下の手順は、はっきりと覚えていないため、冗長だったり、どこか間違っていたりする可能性があります。**

[Ansible Documentations WinRM](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html#id5)
[公的に署名された証明書なしで WinRM サービスまたは HTTPS を構成する](https://help.f-secure.com/product.html?business/radar/3.0/ja/task_8772A6A76D994406B4809EB264EB51EE-3.0-ja)

## Host Vars

* 設定が完了してから、以下の構文で接続できるはずです。

```yml
ansible_connection: winrm
# 公開鍵パス
ansible_winrm_cert_pem: /path/to/certificate/public/key.pem
# 秘密鍵パス
ansible_winrm_cert_key_pem: /path/to/certificate/private/key.pem
# 証明書
ansible_winrm_transport: certificate
```

```ini
ansible_connection=winrm
;公開鍵fullパス
ansible_winrm_cert_pem=/path/to/certificate/public/key.pem
;秘密鍵fullパス
ansible_winrm_cert_key_pem=/path/to/certificate/private/key.pem
;証明書
ansible_winrm_transport=certificate
```

## WinRM の設定

[reference](https://cloudpack.media/8239)

```ps
# WinRM 起動
winrm quickconfig
# OR
winrm qc

# Listener 確認
Get-Item WSMan:\localhost\Listener\*\Port
winrm e winrm/config/listener

# Listner 作成
# 上記のコマンド(winrm quickconfig)で作られているのでやらなくてもいい
winrm create winrm/config/listener?Address=*+Transport=HTTP
winrm create winrm/config/Listener?Address=*+Transport=HTTPS '@{Hostname="localhost";CertificateThumbprint="1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p7q8r9s0t"}'
#New-Item -Path WSMan:\LocalHost\Listener -Transport HTTPS -Address * `
#-CertificateThumbPrint "1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p7q8r9s0t" –Force

# 自己証明書発行
$Cert = New-SelfSignedCertificate -CertstoreLocation `
Cert:\LocalMachine\My -DnsName "localhost"
# 拇印の確認
$cert.Thumbprint

# リスナ作成
New-Item -Path WSMan:\LocalHost\Listener -Transport HTTPS -Address * `
-CertificateThumbPrint $Cert.Thumbprint –Force

# ポート5986解放
New-NetFirewallRule -DisplayName "Windows Remote Management (HTTPS-In)" `
-Description "WS-Management による Windows リモート管理のための受信規則です。[TCP 5986]" `
-Name "Windows Remote Management (HTTPS-In)" `
-Profile Any `
-LocalPort 5986 `
-Protocol TCP

# WinRM 設定の確認
winrm get winrm/config

# 証明書でログインを有効化
winrm set winrm/config/service/auth '@{Certificate="true"}'

# firewall confirm
Get-NetFirewallRule -Name WINRM*

# port 解放
# 5985 http
# 5986 https
netsh firewall add portopening TCP 80 "Windows Remote Management"

# このコマンドレットで一発でできるようだ。（上のコマンドはほとんどいらない）
# 1.WinRMを起動する
# 2.WinRMサービスのスタートアップの種類を自動に
# 3.どのIPアドレスからでも受け付けるリスナ作成
# 4.Windows FirewallにWS-Management traffic (httpのみ)の例外を作成
# 5.WinRMで接続した時に管理者権限で実行の設定
Enable-PSRemoting -Force

# 無効化
Disable-PSRemoting
```

## 証明書を生成する

### Linux OpenSSL を使用

```sh
#! /bin/bash

USERNAME="username"

cat > openssl.conf << EOL
distinguished_name = req_distinguished_name
[req_distinguished_name]
[v3_req_client]
extendedKeyUsage = clientAuth
subjectAltName = otherName:1.3.6.1.4.1.311.20.2.3;UTF8:$USERNAME@localhost
EOL

export OPENSSL_CONF=openssl.conf
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out cert.pem -outform PEM -keyout cert_key.pem -subj "/CN=$USERNAME" -extensions v3_req_client
rm openssl.conf
```

### windows server PowerShell を使用

```ps
# Windows で実行
# 管理者権限で実行
# 鍵がマップされるローカルユーザーの名前を設定する。
$username = "localusername"
$output_path = "C:\tmp"

# ファイルを生成する代わりに、対象ユーザーに証明書が設定されます。
# ローカルコンピュータフォルダの証明書ストア
$cert = New-SelfSignedCertificate -Type Custom `
    -Subject "CN=$username" `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2","2.5.29.17={text}upn=$username@localhost") `
    -KeyUsage DigitalSignature,KeyEncipherment `
    -KeyAlgorithm RSA `
    -KeyLength 2048

# Export the public key
$pem_output = @()
$pem_output += "-----BEGIN CERTIFICATE-----"
$pem_output += [System.Convert]::ToBase64String($cert.RawData) -replace ".{64}", "$&`n"
$pem_output += "-----END CERTIFICATE-----"
[System.IO.File]::WriteAllLines("$output_path\cert.pem", $pem_output)

# Export the private key in a PFX file
[System.IO.File]::WriteAllBytes("$output_path\cert.pfx", $cert.Export("Pfx"))
```

```ps
#$ip="192.168.137.169" # your ip might be different
#$c = New-SelfSignedCertificate -DnsName $ip `
#                               -CertStoreLocation cert:\LocalMachine\My

#winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"$ip`";CertificateThumbprint=`"$($c.ThumbPrint)`"}"

#netsh advfirewall firewall add rule name="WinRM-HTTPS" dir=in localport=5986 protocol=TCP action=allow
```

* PowerShellコマンドレットNew-SelfSignedCertificateを使用した認証用の証明書の生成は、Windows 10またはWindows Server 2012 R2以降のホストから生成された場合にのみ機能します。
* OpenSSLは、Ansibleを使用するためにPFX証明書からPEMファイルに秘密鍵を抽出するために必要です。

## Ansible の証明書をインポート

### Root

```ps
# Windows で実行
$cert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2
$cert.Import("C:\TMP\ansible\cert.pem")

$store_name = [System.Security.Cryptography.X509Certificates.StoreName]::Root
$store_location = [System.Security.Cryptography.X509Certificates.StoreLocation]::LocalMachine
$store = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Store -ArgumentList $store_name, $store_location
$store.Open("MaxAllowed")
$store.Add($cert)
$store.Close()
```

* ADCSを使用して証明書を生成する場合、発行証明書はすでにインポートされているため、この手順はスキップできます。

### Trusted Root

```ps
# Windows で実行

$cert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2
$cert.Import("C:\TMP\ansible\cert.pem")

$store_name = [System.Security.Cryptography.X509Certificates.StoreName]::TrustedPeople
$store_location = [System.Security.Cryptography.X509Certificates.StoreLocation]::LocalMachine
$store = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Store -ArgumentList $store_name, $store_location
$store.Open("MaxAllowed")
$store.Add($cert)
$store.Close()
```

* certlm.msc GUI で確認できます。

## ローカルユーザーアカウントに証明書を関連付ける

```ps
$username = "localusername"
$password = ConvertTo-SecureString -String "password" -AsPlainText -Force
$credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username, $password

$thumbprint = (Get-ChildItem -Path cert:\LocalMachine\root | Where-Object { $_.Subject -eq "CN=$username" }).Thumbprint

New-Item -Path WSMan:\localhost\ClientCertificate `
    -Subject "$username@localhost" `
    -URI * `
    -Issuer $thumbprint `
    -Credential $credential `
    -Force
```
