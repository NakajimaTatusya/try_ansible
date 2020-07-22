# WinRM の設定

## WinRM

* Ansible が HTTP、HTTPS で、リモート Windows を操作するために使用する。
* Windows10 は規定で無効化されている。
* Windows Server 2016 は規定で有効になっている。
  * ただし、Firewall 認証方式（証明書）によって、そのままでは接続できない。
* Windows Server 2012 R2 または、Windows Server 2012
  * 必要なソフトウエア
    * .NET Framework 4.6
    * Windows Management Framwork 5.0
* Windows Server 2008 R2
  * 必要なソフトウエア
    * .NET Framework 4.5
    * Windows Management Framwork 4.0
* Windows Server 2008
  * 必要なソフトウエア
    * .NET Framework 4.0
    * Windows Management Framwork 3.0

## Fire wall ruleの確認

```PS

Get-NetFirewallRule -Name WINRM-HTTP-In-TCP

Get-NetFirewallRule -Name WINRM-HTTP-In-TCP | Set-NetFirwallRule -Profile Any -PassThru

```

## WinRM コンフィグ確認

```cmd
winrm get winrm/config
```

## BASIC 認証許可

```cmd
winrm set winrm/config/service/auth '@{Basic="true"}'
```

## 平文許可

```cmd
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```
