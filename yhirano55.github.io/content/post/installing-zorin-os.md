+++
title = "Zorin OS 12.1をインストール"
date = "2017-04-02T14:19:33+09:00"
description = "MBA2010 LateにZorin OSをインストールする手順"
tags = ["Linux", "ZORINOS"]

+++

MacBook Air 2010 Late（1.4GHz Intel Core 2 Duoプロセッサ, メモリ2G）の使い途がないので、USBメモリを使って、LINUXディストリビューションの1つである、[ZORIN OS 12.1  ULTIMATE](https://zorinos.com/)をインストールした。手順は[Ubuntuと同様](http://d.hatena.ne.jp/takeR/20140730/1406711628)だが、個人的な備忘録として作業手順を残しておく。

### ISOのダウンロード

[Download Zorin OS](https://zorinos.com/download/)より、ISOをダウンロードする。

### dmgフォーマットに変換

ターミナルで `.dmg` フォーマットに変換。

```sh
$ hdiutil convert -format UDRW -o ~/Downloads/Zorin-OS-12.1-Core-64.dmg ~/Downloads/Zorin-OS-12.1-Core-64.iso
```

### imgフォーマットにリネーム

続けて `.img` フォーマットにリネームする。

```sh
$ mv ~/Downloads/Zorin-OS-12.1-Core-64.dmg ~/Downloads/Zorin-OS-12.1-Core-64.img
```

### USBメモリをunmount

まず `diskutil list` で、内部・外部ディスクの一覧を表示。USBメモリに該当するメモリ（ここでは `/dev/disk2` とする）をunmountする。

```sh
$ diskutil unmountDisk /dev/disk2
```

### imgをUSBメモリに書き込む

下記コマンド実行後、しばらくすると、USBメモリとして認識されなくなり、取り出すようWarningが表示される。

```sh
$ sudo dd if=~/Downloads/Zorin-OS-12.1-Core-64.img of=/dev/rdisk2 bs=1m
```

取り出したら、インストール先のmacbookに差し込み、`option` キーを押しながら起動すると、USBのイメージを選択して起動できる。その後は、インストーラーのウィザードに従うのみ。
