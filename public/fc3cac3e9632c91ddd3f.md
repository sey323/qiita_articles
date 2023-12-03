---
title: Facescrubの顔画像データセットを準備する
tags:
  - Python
  - 顔認識
  - データセット
  - Face
  - 顔検出
private: false
updated_at: '2020-08-24T23:50:10+09:00'
id: fc3cac3e9632c91ddd3f
organization_url_name: null
slide: false
ignorePublish: false
---
# 初めに
GANやCNNの顔認識や顔画像生成の際に用いる顔画像データセットとしてFacescrubがある．早速利用しようとダウンロードしたところ使うときに結構コツが必要だったので，備忘録としてまとめた．


# Facescrub
Facescrubは俳優または女優の画像を集めたデータセットである．大規模顔画像処理の際に頻繁に用いられるデータセットで，カーネギー大学が公開している．URLは以下の通り．

> http://vintage.winklerbros.net/facescrub.html


## データセットのダウンロード
データセットのダウンロードには，Webページ内部の，`this form`にメールアドレスと名前，利用目的などを記述する必要がある．

> [Cognito Forms](https://www.cognitoforms.com/ADSC2/FaceScrubDatasetPasswordRequest)

![this_form.png](https://qiita-image-store.s3.amazonaws.com/0/163680/c341abcd-a6d7-644a-6fa5-ad4f576d8f8d.png)


ここに必要事項を記載すると，後日登録したメールアドレスに暗号化zipと解凍に必要なパスワードが送られてくる．それを解凍すると，以下の様なファイル構成になっている．

```sh:facescrub
facescrub
  |
  | - README.txt
  | - LICENSE.txt
  | - facescrub_actors.txt
   -- facescrub_actresses.txt
```

画像データがそのまま送られてくると思ったら，テキストファイルしかない...
実はこの中の`acescrub_actors.txt`と`facescrub_actresses.txt`に，俳優の名前と画像のURL，idが記述されており，画像データを利用するにはテキストに記述されているURLからダウンロードしてくる必要がある．


## 画像ダウンロード
画像のダウンロードには以下のリポジトリを用いる．

> [GitHub - lightalchemist/FaceScrub](https://github.com/lightalchemist/FaceScrub)

こちらのリポジトリの`<Python Version>_download_facescrub.py`に，先ほど取得した`facescrub_actors.txt`か`facescrub_actresses.txt`のテキストファイルを指定することで，自動的に全てのファイルをダウンロードしてくれる．
さらに引数を指定することで，顔領域だけ切り抜くやログの出力といった，細かい処理にも対応している．
以下のものだと`./actors_face`にURLの画像と画像から顔領域のみを抽出した画像をダウンロードし，`download.log`にログを書き出す．

```sh:
$ python python3_download_facescrub.py faceScrub/facescrub_actors.txt ./actors_face --crop_face --logfile=download.log --timeout=10 --max_retries=3
```
URLによってはダウンロードリンクのリンク先が無効になっているものもあるので，`--timeout`と`--max_retries`で実行に制限をかけた方が良い．
ダウンロード中は，こんな感じにログが大量に出力される．

```
2019-03-14 17:19:36
Processing line 6992: http://userserve-ak.last.fm/serve/_/353467/Billy%252BBob%252BThornton.jpg
2019-03-14 17:19:36
Line 6992: HTTPConnectionPool(host='userserve-ak.last.fm', port=80): Max retries exceeded with url: /serve/_/353467/Billy%252BBob%252BThornton.jpg (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x1050cb438>: Failed to establish a new connection: [Errno 8] nodename nor servname provided, or not known',)): http://userserve-ak.last.fm/serve/_/353467/Billy%252BBob%252BThornton.jpg
2019-03-14 17:19:36
Processing line 6993: http://www4.pictures.zimbio.com/gp/Billy%252BBob%252BThornton%252BConnie%252BAngland%252Bdating%252BZKHdAfnGiKGl.jpg
2019-03-14 17:19:36
Processing line 6994: http://images.amcnetworks.com/ifc.com/wp-content/uploads/2011/03/billy-bob-thornton-willie-nelson-ifc-sxsw.jpg
2019-03-14 17:19:36
Processing line 6995: http://images.sodahead.com/polls/000307749/polls_BillyBobThornton_1_300_3747_293487_poll_xlarge.jpeg
2019-03-14 17:19:36
Processing line 6996: http://2.bp.blogspot.com/-M52lwBAACXc/TzKaR2fCoJI/AAAAAAAABRI/R3V6XcIOFI8/s1600/a1-billy-bob-thornton-wallpaper-8-759729.jpg
2019-03-14 17:19:36
Processing line 6997: http://assets-s3.usmagazine.com/uploads/assets/articles/52525-billy-bob-thornton-i-didnt-think-i-was-good-enough-to-be-married-to-angelina-jol/1337180984_billy-bob-thornton-article.jpg
2019-03-14 17:19:36
Line 6991: 404 Client Error: Not Found for url: https://www.eonline.com/eol_images/Entire_Site/20080521/293.thornton.bb.052108.jpg: http://www.eonline.com/eol_images/Entire_Site/20080521/293.thornton.bb.052108.jpg
2019-03-14 17:19:36
Processing line 6998: http://static.squarespace.com/static/51b3dc8ee4b051b96ceb10de/t/51fd4c09e4b03005d2ef2bba/1375554571119/billy-bob-thornton.jpg
2019-03-14 17:19:36
Line 6981: Invalid content-type text/html: http://www.hdwpapers.com/download/billy_bob_thornton_closeup_wallpaper-1152x864.jpg
2019-03-14 17:19:36
Processing line 6999: http://www.femalefirst.co.uk/image-library/port/376/b/billy-bob-thornton-awi-1011.jpg
2019-03-14 17:19:36
Processing line 7000: http://i3.mirror.co.uk/incoming/article809643.ece/ALTERNATES/s615/Billy%252520Bob%252520Thornton%252520as%252520Bad%252520Santa-809643&
2019-03-14 17:19:37
Line 6995: 404 Client Error: Not Found for url: http://images.sodahead.com/polls/000307749/polls_BillyBobThornton_1_300_3747_293487_poll_xlarge.jpeg: http://images.sodahead.com/polls/000307749/polls_BillyBobThornton_1_300_3747_293487_poll_xlarge.jpeg
2019-03-14 17:19:37
Processing line 7001: http://www.eonline.com/eol_images/Entire_Site/20090223/425.thornton.jolie.022309.jpg
2019-03-14 17:19:37
```


ファイル構造は以下の通り．`images`と`faces`ディレクトリにそれぞれの俳優名のディレクトリが作られ，その中に画像ファイルがダウンロードされる．

```sh:
actors_face
  |
  | - faces/
  |   | - Aaron_Eckhart/
  |         | - Aaron_Eckhart_1_1.jpeg
  |         | - Aaron_Eckhart_3_3.jpeg
  |         | - Aaron_Eckhart_5_5.jpeg
  |                ・
  |                ・
  |   | - Adam_Brody/
  |   | - Adam_McKay/
  |       ・
  |       ・
  | -　images/
      | - Aaron_Eckhart/
      | - Adam_Brody/
      | - Adam_McKay/
          ・
          ・

```

人によるが1人当たり100枚前後の顔画像がダウンロードされる．
