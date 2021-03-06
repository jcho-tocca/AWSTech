# Laravel-CloudFront
AWS CloudFrontを利用してLaravelアプリで、動画配信（オンデマンド）を行います。

# 🖥Environment
* Laravel 7.X

## 🔓 1. AWS S3バケットの作成。

##### 1. [Laravel-S3](https://github.com/tocca-systems/369-source/wiki/Laravel-S3)にLaraveからAWS S3にファイルをアップロードするための一連の説明が載ってます。

## 🎥 2. AWS CloudFrontの設定。

##### 1. AWSへログインし「CloudFront」画面へ遷移し、「Create Distribution」を選択する。
![1](https://user-images.githubusercontent.com/13768156/104421689-0e2ee180-55bf-11eb-9073-ead9febe1f41.png)

##### 2. 「Web」側の「Get Started」を選択する。
![3](https://user-images.githubusercontent.com/13768156/104421701-1129d200-55bf-11eb-887e-a918ac06c47c.png)

##### 3. 作成画面が表示されるので、各々設定していく。
![4](https://user-images.githubusercontent.com/13768156/104421703-125aff00-55bf-11eb-828d-e90dfc76ed4b.png)
![5](https://user-images.githubusercontent.com/13768156/104421705-138c2c00-55bf-11eb-8cea-ec6755fb5d8e.png)

* 「Origin Domain Name」： 作成してあるバケットを選択する。
* 「Restrict Bucket Access」： 「Yes」を選択する。
* 「Origin Access Identity」： 「Create a New Identity」を選択する。
* 「Grant Read Permissions on Bucket」： 「Yes, Update Bucket Policy」を選択する。
* 「Viewer Protocol Policy」： 「Redirect HTTP to HTTPS」を選択する。（これは場合による）
* そのほかの設定項目がデフォルトのままでもOK。

##### 4. 「Create Distribution」ボタンを選択する。
![6](https://user-images.githubusercontent.com/13768156/104421717-17b84980-55bf-11eb-9820-4a09bd5606c8.png)

##### 5. 作成完了画面が表示される。
![7](https://user-images.githubusercontent.com/13768156/104421723-18e97680-55bf-11eb-9056-5b4a1c34d97f.png)

##### 6. CloudFrontのTOP画面に戻り作成したDistributionを確認する。（Stateが「**Enabled**」になっていればOKです。）
![8](https://user-images.githubusercontent.com/13768156/104421729-1be46700-55bf-11eb-9c85-bc22e6f21151.png)


## 🎥 3. 動画ファイル（HLS）の作成。
AWS S3にアップロードした動画はHLS形式に変換します。[Appleが策定した規格です](https://developer.apple.com/streaming/)  
※ S3には動画がすでに配置されていると仮定して、以下の説明を行います。  
※ また、ローカルで例えば「mp4 ⇒ hl」sへの変換をかけた場合は、以下の手順は必要ありません。（そのままS3へアップロードすればよい）

##### 1. 「AWS Elemental MediaConvert」画面へ遷移し、「ジョブの作成」を選択する。
![9](https://user-images.githubusercontent.com/13768156/104421736-1dae2a80-55bf-11eb-8d4c-ad81ba84e443.png)

##### 2. 「入力1」タブの「入力ファイルURL」欄に変換を行うS3ファイルのURIを入力する。
![10](https://user-images.githubusercontent.com/13768156/104421741-1f77ee00-55bf-11eb-810b-79cd69382796.png)

##### 3. 「出力グループ」タブの「追加」ボタンを選択する。
![11](https://user-images.githubusercontent.com/13768156/104421750-20a91b00-55bf-11eb-8da9-fcc160886a88.png)

##### 4. 「Apple HLS」を選択して「選択」ボタンを押下する。
![12](https://user-images.githubusercontent.com/13768156/104421754-21da4800-55bf-11eb-9cf8-0d3f65021434.png)

##### 5. 「出力グループ」に「Apple HLS」が表示されるようになるので選択し、「送信先」欄にHLSファイルの出力先になるS3 URIファイルを入力する。
![13](https://user-images.githubusercontent.com/13768156/104421759-2272de80-55bf-11eb-9c73-7d910f8997de.png)

##### 6. 「出力グループ」に「H.264.AAC」が表示されているので選択し、「名前修飾子」欄に任意の名前を入力する。（出力されたファイルのお尻にくっつきます。）
![14](https://user-images.githubusercontent.com/13768156/104421764-23a40b80-55bf-11eb-9ef1-c47f0b749a74.png)

##### 7. 「出力グループ」に「H.264.AAC」が表示されているので選択し、「ビットレート」欄にビットレートを入力する。
![15](https://user-images.githubusercontent.com/13768156/104421765-243ca200-55bf-11eb-87d6-c8169e48dd44.png)

##### 8. 下記画像のような画面が表示されたら成功で、しばらく待つと出力先S3バケットにファイルが作成される。
![17](https://user-images.githubusercontent.com/13768156/104421770-256dcf00-55bf-11eb-8bf9-940f1bb39ac9.png)

## 🎥 4. S3バケットへのCORSの設定
現状のAWS S3の設定ではブラウザからの動画へのアクセスができないのでS3バケットにCORSの付与を行う。

##### 1. S3のバケット画面へ遷移し、「アクセス許可」を選択する。
![18](https://user-images.githubusercontent.com/13768156/104421776-27d02900-55bf-11eb-9b5a-460006f24e26.png)

##### 2. CORSの設定画面があるので、「編集する」ボタンを押下して、下記のような設定を記述する。（一例）
![19](https://user-images.githubusercontent.com/13768156/104421779-29015600-55bf-11eb-8618-ee6caba1c728.png)

```
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "PUT",
            "POST",
            "DELETE"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    },
    {
        "AllowedHeaders": [],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    }
]
```

## 🎥 5. リソースへのアクセス。
リソースへのアクセスは、CloudFront経由でのみ許可されている。（CloudFrontの「Restrict Bucket Access」が「Yes」のため）
アクセス例 ⇒ https://{クラウドフロントのドメイン}/{S3のディレクトリ}/{S3のファイル名}

![20](https://user-images.githubusercontent.com/13768156/104421789-2acb1980-55bf-11eb-9b7d-167c700797a5.png)
