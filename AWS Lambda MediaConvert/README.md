# Lambda-MediaConvert
* AWS S3に動画ファイルがアップロードされたことを検知してMedia ConverterでHLS形式にファイルを変換するlambdaを作成します。

# 🖥Environment
* AWS

## 🤔 1. 準備
Media ConverterのAPIエンドポイントを取得します。

##### 1. AWSへログインし、「Media Converter」画面へ遷移し、メニューから「アカウント」を選択する。
![1](https://user-images.githubusercontent.com/13768156/105949845-bdd07d00-60b0-11eb-82a1-7caab58001c1.png)

##### 2. 表示されたAPIエンドポイントをメモに控えておく。
![2](https://user-images.githubusercontent.com/13768156/105949848-bf01aa00-60b0-11eb-9560-308ae21eaa4c.png)

## 🔓 2. lambda実行用のIAMロールを作成する。

##### 1. 「IAM」画面へ遷移し、メニューから「ポリシー」を選択する。
![3](https://user-images.githubusercontent.com/13768156/105949853-c0cb6d80-60b0-11eb-98ad-91d71cab2968.png)

##### 2. 「ポリシーを作成」を選択する。
![4](https://user-images.githubusercontent.com/13768156/105949855-c1fc9a80-60b0-11eb-902b-1f6cfd9568a3.png)

##### 3. 「JSON」のタブを選択する。
![5](https://user-images.githubusercontent.com/13768156/105949858-c32dc780-60b0-11eb-9f62-cb19c91925a0.png)

##### 4. 以下のJSONを貼り付けて、「」を選択する。（media converterのjob作成）
![6](https://user-images.githubusercontent.com/13768156/105949862-c45ef480-60b0-11eb-979a-2ccbc03444ec.png)
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "mediaconvert:CreateJob",
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:PutLogEvents",
                "iam:PassRole"
            ],
            "Resource": "*"
        }
    ]
}
```

##### 5. ポリシーの「名前」と「説明」を選択して、「ポリシーの作成」を選択する。
![7](https://user-images.githubusercontent.com/13768156/105949867-c5902180-60b0-11eb-857c-6ec0bc113351.png)

##### 6. 「IAM」画面へ遷移し、メニューから「ロール」を選択する。
![8](https://user-images.githubusercontent.com/13768156/105949870-c6c14e80-60b0-11eb-8f85-556226b1a904.png)

##### 7. 「ロールの作成」を選択する。
![9](https://user-images.githubusercontent.com/13768156/105949874-c7f27b80-60b0-11eb-8355-6d116fe0ea06.png)

##### 8. 「Lambda」を選択し、「次のステップ：アクセス権限」を選択する。
![10](https://user-images.githubusercontent.com/13768156/105949878-c923a880-60b0-11eb-9ea6-58ffd3a74fe4.png)

##### 9. 先ほど作成したポリシーにチェックを入れて「次のステップ：タグ」を選択する。
![11](https://user-images.githubusercontent.com/13768156/105949890-ce80f300-60b0-11eb-9b17-9d906701249f.png)

##### 10. タグは特に設定しなくてもいいので、「次のステップ：確認」を選択する。
![12](https://user-images.githubusercontent.com/13768156/105949894-cfb22000-60b0-11eb-8b05-d52c3137b785.png)

##### 11. 「ロール」名の入力を行い、「ロールの作成」を選択する。
![13](https://user-images.githubusercontent.com/13768156/105949901-d0e34d00-60b0-11eb-9ec8-7b343d812d98.png)

## 🔓 3. lambda関数を作成する。
##### 1. 「lambda」画面へ遷移し、メニューから「関数の作成」を選択する。
![14](https://user-images.githubusercontent.com/13768156/105949905-d345a700-60b0-11eb-8905-1bc3f3e407bb.png)

##### 2. 関数作成画面で、「一から作成」「関数名」に関数名を入力、「ランタイム」にPython3.8を選択、「実行ロール」に「既存のロールを選択する」を選択して、ロールを先ほど先ほど選択したロールにして、「関数の作成」を選択する。
![15](https://user-images.githubusercontent.com/13768156/105949907-d476d400-60b0-11eb-8e11-5d9252d60c0c.png)
![16](https://user-images.githubusercontent.com/13768156/105949914-d5a80100-60b0-11eb-93a7-40d9ab497ccf.png)

##### 3. 関数が作成されたら、作成された関数の画面で「トリガーの追加」を選択する。
![17](https://user-images.githubusercontent.com/13768156/105949920-d8a2f180-60b0-11eb-8fdc-991189c0e4e1.png)

##### 4. トリガーの設定画面で関数のトリガーを設定する。
* 「S3」「対象のバケット」「サフィックスオプションに何かしらの動画の形式」を選択する。
![18](https://user-images.githubusercontent.com/13768156/105949932-de003c00-60b0-11eb-8fc8-0df1261f31b6.png)

##### 5. 関数の「コード」タブを選択し、lambda_function.pyの中身を以下のものに置き換える。
![19](https://user-images.githubusercontent.com/13768156/105949939-dfc9ff80-60b0-11eb-9a25-326fa2d8c5ae.png)
```
import json
import urllib.parse
import boto3
import os

s3 = boto3.client('s3')

def lambda_handler(event, context):
    account = context.invoked_function_arn.split(':')[4]
    region = event['Records'][0]['awsRegion']
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')

    template_name = os.environ['template_name']
    role_name = os.environ['role_name']
    
    inputFile = "s3://" + bucket + "/" + key
    outputKey = "s3://" + bucket + "/output/"
    
    mediaconvert = boto3.client(
        'mediaconvert', 
        region_name=region, 
        endpoint_url=os.environ['endpoint_url']
    )
    
    with open("job.json", "r") as jsonfile:
        job_object = json.load(jsonfile)
         
    job_object["OutputGroups"][0]["OutputGroupSettings"]["HlsGroupSettings"]["Destination"] = outputKey
    job_object["Inputs"][0]["FileInput"] = inputFile

    response = mediaconvert.create_job(
      JobTemplate='arn:aws:mediaconvert:%s:%s:jobTemplates/%s' % (region, account, template_name),
      Queue='arn:aws:mediaconvert:%s:%s:queues/Default' % (region, account),
      Role='arn:aws:iam::%s:role/%s' % (account, role_name),
      Settings=job_object
    )
```

##### 6. lambda_function.pyと同ディレクトリ階層に「job.json」を作成し、中身を以下とする。
```
{
    "Inputs": [
        {
            "FileInput": ""
        }
    ],
    "OutputGroups": [
        {
            "OutputGroupSettings": {
                "HlsGroupSettings": {
                    "Destination": ""
                }
            }
        }
    ]
}
```

##### 7. 環境変数に以下のものを設定する。
* endpoint_url = MediaConvert の API endpoint（1.2の参照）
* role_name = Media Converter用のRole名
* template_name = Media Converterの作成したJob [作り方はリンク先の動画ファイル（HLS）の作成をテンプレートの作成で実施する](https://github.com/tocca-systems/369-source/wiki/Laravel-CloudFront)
![22](https://user-images.githubusercontent.com/13768156/105949944-e193c300-60b0-11eb-8b25-348b5ffea772.png)

##### 8. 1～7までを終えたら保存して「デプロイ」を選択する。
![23](https://user-images.githubusercontent.com/13768156/105949965-eb1d2b00-60b0-11eb-8261-cfbaf9fa6af0.png)
![24](https://user-images.githubusercontent.com/13768156/105949967-ece6ee80-60b0-11eb-8eab-27737a2f5741.png)

