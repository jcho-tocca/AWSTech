# Laravel-AWS-S3
* Laravel上でAWS S3とのファイルのやり取りをおこなうための説明書きです。

# 🖥Environment
* Laravel 7.X

## 🔓 1. AWS IAMの設定
S3へアップロードしたファイルへのアクセスポリシーを設定するため、IAMの設定を行います。

##### 1. AWSへログインし、「IAM」画面へ遷移する。
![1](https://user-images.githubusercontent.com/13768156/103439117-a521a280-4c7d-11eb-9e6f-dfad0c753b13.png)

##### 2. 「ユーザ」」を選択する。
![2](https://user-images.githubusercontent.com/13768156/103439118-a783fc80-4c7d-11eb-8422-fcfb0ddad084.png)

##### 3. 「ユーザ作成」を選択します。
![3](https://user-images.githubusercontent.com/13768156/103439120-a94dc000-4c7d-11eb-8a03-ab62f4ae9ec1.png)

##### 4. IAMユーザ作成画面で「ユーザ名」を入力し、アクセスの種類を「プログラムによるアクセス」にチェックを入れて、次へ進みます。
![4](https://user-images.githubusercontent.com/13768156/103439123-ab178380-4c7d-11eb-84a3-302c3afb88b6.png)

##### 5. アクセス許可の設定を「既存のポリシーを直接アタッチ」にし、「AmazonS3FullAccess」にチェックを入れて、次に進みます。
![5](https://user-images.githubusercontent.com/13768156/103439125-b10d6480-4c7d-11eb-93e0-dbb75b3aad43.png)

##### 6. IAMタグは必要に応じて入力を行います。（今回は設定しません）「次のステップ：確認」ボタンを押下してください。
![6](https://user-images.githubusercontent.com/13768156/103439129-b66aaf00-4c7d-11eb-9b66-8e64da9b7160.png)

##### 7. 確認画面で3~6の入力内容を確認後に「ユーザの作成」ボタンを押下します。
![7](https://user-images.githubusercontent.com/13768156/103439135-bbc7f980-4c7d-11eb-8f40-e3eef9adcde5.png)

##### 8. ユーザの作成が完了したら、本画面で「.csvのダウンロード」を押下して認証情報をダウンロードします。（後で使用します。）
![8](https://user-images.githubusercontent.com/13768156/103439137-bec2ea00-4c7d-11eb-9d08-1ef5973b156b.png)

##### 9. IAMのダッシュボード画面に戻り、作成したユーザ名を押下します。
![9](https://user-images.githubusercontent.com/13768156/103439139-c1254400-4c7d-11eb-8a85-adcabe033f71.png)

##### 10. ユーザ画面で「認証情報」タブを開き、「管理」を押下したあとにポップする「コンソールアクセスの管理」画面で、「コンソールのアクセス」を「有効」に、「パスワードの設定」を「自動生成パスワード」にチェックをいれて「適用」ボタンを押下します。
![10](https://user-images.githubusercontent.com/13768156/103439140-c4203480-4c7d-11eb-9ce6-2d6ac16d5cda.png)
![11](https://user-images.githubusercontent.com/13768156/103439149-d26e5080-4c7d-11eb-9e21-a7763bda8d5a.png)

##### 11. ポップする「新しいパスワード」画面で「.csvのダウンロード」を押下して認証情報をダウンロードします。
![12](https://user-images.githubusercontent.com/13768156/103439151-d39f7d80-4c7d-11eb-9078-6a59d8fa44e2.png)

##### 12. IAMの設定は以上となります。

## 📦2. AWS S3バケットの設定
アップロードしたファイルの保存を実際に行う場所であるS3のバケットを作成します。

##### 1. AWSへログインし、「S3」画面へ遷移する。
![13](https://user-images.githubusercontent.com/13768156/103439152-d601d780-4c7d-11eb-9afe-7078adeb1669.png)

##### 2. S3トップ画面で「バケットを作成」ボタンを押下します。
![14](https://user-images.githubusercontent.com/13768156/103439153-d7330480-4c7d-11eb-9ded-a4e9fe1d6056.png)

##### 3. バケットを作成画面の一般的な設定項目で、「バケット名」に全世界で一意になる名称を入力し、「リージョン」（これはどれでもOK）を設定します。
![15](https://user-images.githubusercontent.com/13768156/103439154-d8643180-4c7d-11eb-9e74-ba72eaac480c.png)

##### 4. バケットを作成画面のブロックパブリックアクセスの設定項目で、「新しいパブリックバケットポリシーまたはアクセスポイントポリシーを介して付与されたバケットとオブジェクトへのパブリックアクセスをブロックする」と「任意のパブリックバケットポリシーまたはアクセスポイントポリシーを介したバケットとオブジェクトへのパブリックアクセスとクロスアカウントアクセスをブロックする」にチェックをいれます。
![16](https://user-images.githubusercontent.com/13768156/103439155-da2df500-4c7d-11eb-9381-b68fb0a80504.png)

##### 5. 「デフォルトの暗号化」の有効向こうは任意です。「バケットを作成」ボタンを押下して、バケットを作成します。
![17](https://user-images.githubusercontent.com/13768156/103439156-dbf7b880-4c7d-11eb-8be8-4bddebebdccb.png)

##### 6. S3トップ画面から作成したS3バケットを選択します。
![18](https://user-images.githubusercontent.com/13768156/103439158-dd28e580-4c7d-11eb-9ca8-36ca438e144f.png)

##### 7. 「アクセス許可」タブを選択し、「バケットポリシー」の「編集する」ボタンを押下します。
![19](https://user-images.githubusercontent.com/13768156/103439163-e44ff380-4c7d-11eb-845d-5133ecf589c9.png)20
![20](https://user-images.githubusercontent.com/13768156/103439164-e74ae400-4c7d-11eb-8aae-115b50c2563c.png)

##### 8. 「バケットポリシーを編集」画面へ遷移したら、「バケットポリシー」のポリシージェネレータボタンを押下します。
![21](https://user-images.githubusercontent.com/13768156/103439165-e914a780-4c7d-11eb-93d3-0f43e48bdc5e.png)

##### 9. ポリシーの設定は以下の画像通りになります。設定したら「Add Statement」ボタンを押下します。
* Pricipalには先ほど作成したAWS IAMのARNを、Amazon Resource Nameには作成したS3のARNを入力します。（プロパティタブから参照できます。）
![22](https://user-images.githubusercontent.com/13768156/103439168-ea45d480-4c7d-11eb-8eff-8daed727a169.png)

##### 10. 画面下部に設定したポリシーが表示されるので、「Generate Policy」ボタンを押下します。
![23](https://user-images.githubusercontent.com/13768156/103439170-eb770180-4c7d-11eb-9107-6560cdcb9f4e.png)

##### 11. 作成されたポリシーがJson形式で表示されるので、全選択コピーします。
![24](https://user-images.githubusercontent.com/13768156/103439171-eca82e80-4c7d-11eb-8847-3802ec34ce07.png)

##### 12. コピー内容をポリシーに貼り付けます。（上書きします）
![25](https://user-images.githubusercontent.com/13768156/103439172-ee71f200-4c7d-11eb-8a8d-6d7c4ff32154.png)

##### 13. S3の設定は以上となります。

## 💻 3. Laravelアプリの設定
手順1, 2で作成したIAMとS3の情報を元に、Laravelアプリ側の設定を行います。

##### 1. $「vagrant ssh」コマンドで仮想環境上にログインしたら、プロジェクトのホームディレクトリに遷移して、以下のコマンドを実行します。
```
$ COMPOSER_MEMORY_LIMIT=-1 composer require league/flysystem-sftp ~1.0 -vvv
$ COMPOSER_MEMORY_LIMIT=-1 composer require league/flysystem-aws-s3-v3 ~1.0 -vvv
$ COMPOSER_MEMORY_LIMIT=-1 composer require league/flysystem-cached-adapter ~1.0 -vvv
```

##### 2. Laravelのファイルシステム設定ファイルを開き（PJ_HOME/config/filesystems.php）、以下の画像の赤枠の内容を追記します。S3自体の設定は、.envファイルの中の記述を読み取る設定のようです。
![26](https://user-images.githubusercontent.com/13768156/103439272-bdde8800-4c7e-11eb-9b75-656b9d187fa7.png)
![27](https://user-images.githubusercontent.com/13768156/103439275-c0d97880-4c7e-11eb-9eb8-0a31b8a22cb5.png)

##### 3. .envファイルを開きます（PJ_HOME/.env）。
* AWS_ACCESS_KEY_IDには手順1-11で保存したCSVファイル「new_user_credentials.csv」ファイルの内容を転記。
* AWS_SECRET_ACCESS_KEYにも手順1-11で保存したCSVファイル「new_user_credentials.csv」ファイルの内容を転記。
* AWS_DEFAULT_REGIONにはS3バケットを作成したときのリージョンを転記。
* AWS_BUCKETには作成したバケットの名称を転記。

![28](https://user-images.githubusercontent.com/13768156/103439279-c8991d00-4c7e-11eb-986e-1bce01022b8b.png)

##### 3. アプリ側のS3バケットの使用設定は以上となります。

