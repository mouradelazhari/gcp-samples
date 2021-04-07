# Google Cloud ハンズオン #1

## Google Cloud Platform（GCP）プロジェクトの選択

ハンズオンを行う GCP プロジェクトを作成し、 GCP プロジェクトを選択して **Start/開始** をクリックしてください。

**今回のハンズオンは Firestore Native mode を使って行うため、既存のプロジェクト（特にすでに使っているなど）だと不都合が生じる恐れがありますので新しいプロジェクトを作成してください。**


<walkthrough-project-setup>
</walkthrough-project-setup>


<!-- Step 1 -->
## はじめに

### **内容と目的**

本ハンズオンでは、Cloud Run を触ったことない方向けに、テスト アプリケーションを作成し、Cloud Firestore に接続してクエリをする方法などを学びます。
本ハンズオンを通じて、 Cloud Run を使ったアプリケーション開発のイメージを掴んでもらうことが目的です。


### **前提条件**

本ハンズオンははじめて Cloud Run に触れる方を想定しており、 事前知識がなくとも本ハンズオンの進行には影響ありません。
ハンズオン中で使用する GCP プロダクトについてより詳しく知りたい方は、Coursera の教材や公式ドキュメントを使い学んでいただくことをお勧めします。


### **対象プロダクト**

以下が今回学ぶ対象のプロタクトの一覧です。

- Cloud Run
- Cloud Firestore


<!-- Step 2 -->
## ハンズオンの内容

下記の内容をハンズオン形式で学習します。

### **学習ステップ**
- 環境準備：10 分
  - プロジェクト作成
  - gcloud コマンドラインツール設定
  - GCP 機能（API）有効化設定

- [Cloud Run](https://cloud.google.com/run) を用いたアプリケーション開発：40 分
  - サンプル アプリケーションのコンテナ化
  - コンテナの [Google Container Registry](https://cloud.google.com/container-registry/) への登録
  - Cloud Run のデプロイ
  - Cloud Firestore の利用

- クリーンアップ：10 分
  - プロジェクトごと削除
  - （オプション）個別リソースの削除
    - Cloud Run の削除
    - Cloud Firestore の削除
    - Container Registry に登録したコンテナイメージの削除
    - Owner 権限をつけた dev-key.json の削除
    - サービスアカウント dev-sa の削除


<!-- Step 3 -->
## 環境準備

<walkthrough-tutorial-duration duration=10></walkthrough-tutorial-duration>

最初に、ハンズオンを進めるための環境準備を行います。

下記の設定を進めていきます。

- gcloud コマンドラインツール設定
- GCP 機能（API）有効化設定
- サービスアカウント設定


<!-- Step 4 -->
## gcloud コマンドラインツール

Google Cloud は、CLI、GUI、Rest API から操作が可能です。ハンズオンでは主に CLI を使い作業を行いますが、GUI で確認する URL も合わせて掲載します。


### gcloud コマンドラインツールとは?

gcloud コマンドライン インターフェースは、GCP でメインとなる CLI ツールです。このツールを使用すると、コマンドラインから、またはスクリプトや他の自動化により、多くの一般的なプラットフォーム タスクを実行できます。

たとえば、gcloud CLI を使用して、以下のようなものを作成、管理できます。

- Google Compute Engine 仮想マシン
- Google Kubernetes Engine クラスタ
- Google Cloud SQL インスタンス

**ヒント**: gcloud コマンドラインツールについての詳細は[こちら](https://cloud.google.com/sdk/gcloud?hl=ja)をご参照ください。

<walkthrough-footnote>次に gcloud CLI をハンズオンで利用するための設定を行います。</walkthrough-footnote>


<!-- Step 5 -->
## gcloud コマンドラインツール設定

gcloud コマンドでは操作の対象とするプロジェクトの設定が必要です。

### GCP のプロジェクト ID を環境変数に設定

環境変数 `GOOGLE_CLOUD_PROJECT` に GCP プロジェクト ID を設定します。

```bash
export GOOGLE_CLOUD_PROJECT="{{project-id}}"
```

### CLI（gcloud コマンド） から利用する GCP のデフォルトプロジェクトを設定

操作対象のプロジェクトを設定します。

```bash
gcloud config set project $GOOGLE_CLOUD_PROJECT
```

### デフォルトリージョンを設定

リージョナルリソースを作成する際に指定するデフォルトのリージョンを設定します。

```bash
gcloud config set compute/region us-central1
```


<walkthrough-footnote>CLI（gcloud）を利用する準備が整いました。次にハンズオンで利用する機能を有効化します。</walkthrough-footnote>


<!-- Step 6 -->
## GCP 環境設定 Part1

GCP では利用したい機能ごとに、有効化を行う必要があります。
ここでは、以降のハンズオンで利用する機能を事前に有効化しておきます。


### ハンズオンで利用する GCP の API を有効化する

以下の機能を有効にします。

<walkthrough-enable-apis></walkthrough-enable-apis>

- Cloud Build API
- Google Container Registry API
- Google Cloud Firestore API

```bash
gcloud services enable \
  cloudbuild.googleapis.com \
  containerregistry.googleapis.com \
  run.googleapis.com \
  servicenetworking.googleapis.com
```

**GUI**: [APIライブラリ](https://console.cloud.google.com/apis/library?project={{project-id}})

<!-- Step 7 -->
## GCP 環境設定 Part2

### サービスアカウントの作成

ローカルの開発で使用するサービスアカウントを作成します。

```bash
gcloud iam service-accounts create dev-sa
```

作成したサービスアカウントに権限を付与します。 **今回のハンズオンはオーナー権限を付与していますが、実際の開発の現場では適切な権限を付与しましょう！**

```bash
gcloud projects add-iam-policy-binding {{project-id}} --member "serviceAccount:dev-sa@{{project-id}}.iam.gserviceaccount.com" --role "roles/owner"
```

キーファイルを生成します。

```bash
gcloud iam service-accounts keys create dev-key.json --iam-account dev-sa@{{project-id}}.iam.gserviceaccount.com
```

**GUI**: [サービスアカウント](https://console.cloud.google.com/iam-admin/serviceaccounts?project={{project-id}})

作成したキーを環境変数に設定します。

```bash
export GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/dev-key.json
```

<!-- Step 8 -->
## GCP 環境設定 Part3

### Firestore を有効にする

今回のハンズオンでは Firestore のネイティブモードを使用します。

GCP コンソールの [Datastore](https://console.cloud.google.com/datastore/entities/query/kind?project={{project-id}}) に移動し、 [SWITCH TO NATIVE MODE] をクリックしてください。ロケーション選択では `us-east1` を選択してください。

1. 切り替え画面

![switch1](https://storage.googleapis.com/egg-resources/egg1/public/firestore-switch-to-native1.png)
![switch2](https://storage.googleapis.com/egg-resources/egg1/public/firestore-switch-to-native2.png)

2. もしかしたらこちらの画面が表示されている場合もあります。同様にネイティブモードを選択していただければOKです。

![select-firestore-mode](https://storage.googleapis.com/egg-resources/egg1/public/select-mode.png)

3. ネイティブモードが有効になると、[Firestore コンソール](https://console.cloud.google.com/firestore/data/?project={{project-id}})でデータ管理の画面が有効になります。

**Datastore モードの場合でも、まだ一度もデータを登録していなければネイティブモードへの切り替えが可能です。**

<walkthrough-footnote>必要な機能が使えるようになりました。次に Cloud Run によるアプリケーションの開発に進みます。</walkthrough-footnote>


<!-- Step 9 -->
## Cloud Run を用いたアプリケーション開発

<walkthrough-tutorial-duration duration=40></walkthrough-tutorial-duration>

Cloud Run を利用したアプリケーション開発を体験します。

下記の手順で進めていきます。
  - サンプル アプリケーションのコンテナ化
  - コンテナの [Google Container Registry](https://cloud.google.com/container-registry/) への登録
  - Cloud Run のデプロイ
  - Cloud Firestore の利用


<!-- Step 10 -->
## Cloud Shell 復旧手順

もしハンズオン中に Cloud Shell を閉じてしまったり、リロードした場合、以下のコマンドを再実行してから作業を再開してください。(Step 1 から順番に進めている場合はこのページはスキップいただいて結構です)

**Step 10 以降からの再開**

- 環境変数 `GOOGLE_CLOUD_PROJECT` に GCP プロジェクト ID を設定

```bash
export GOOGLE_CLOUD_PROJECT="{{project-id}}"
```

- CLI（gcloud コマンド） から利用する GCP のデフォルトプロジェクトを設定

```bash
gcloud config set project $GOOGLE_CLOUD_PROJECT
```

- デフォルトリージョンを設定

```bash
gcloud config set compute/region us-central1
```

- 作業用のディレクトリへ移動

```bash
cd ~/cloudshell_open/gcp-getting-started-lab-jp/webapp/newgrad
```

- シークレットキーの参照先を設定

```bash
export GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/dev-key.json
```

**Step 16 以降からの再開**

上記に加えて、以下も実行してからお進みください。

- Cloud Run の URL の取得

```bash
URL=$(gcloud run services describe --format=json --region=us-central1 --platform=managed ca-app | jq .status.url -r)
echo ${URL}
```

**Cloud Shell が固まってしまう方へ**

Cloud Shell が遅い、固まってしまう、という場合はブーストモードを有効にすることで改善される可能性がありますのでお試しください。ブーストモードは、Cloud Shell VM のマシンスペックを 24 時間の間、一時的に向上させる機能です。

- Cloud Shell のブーストモードの有効化手順

ブーストモードを有効にするには、[その他] メニュー（Cloud Shell の右上にある 3 つの点のアイコン）の下の [ブーストモードを有効にする] オプションを使います。ブーストモードを有効にすると、Cloud Shell が再起動され、すぐにセッションが終了します。その後、新しい VM がプロビジョニングされますが、これには数分かかることがあります。ホーム ディレクトリのデータはそのまま残りますが、実行中のすべてのプロセスは失われます。


<!-- Step 11 -->
## アプリケーション コードの確認

**このステップのコードは answer/step11/main.go と同じです。**

ハンズオン用のサンプル Web アプリケーションとして　Go 言語で API サーバーを作成していきます。

まずはカレント ディレクトリにある main.go を確認してください。
単純な HTTP リクエストに対して `Welcome to CA!` を返す Go のコードになります。


```go:main.go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"
)

func main() {
	http.HandleFunc("/", indexHandler)

	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
		log.Printf("Defaulting to port %s", port)
	}

	log.Printf("Listening on port %s", port)
	if err := http.ListenAndServe(":"+port, nil); err != nil {
		log.Fatal(err)
	}
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "Welcome to CA!")
}
```

確認できたら、ローカルで動かしてみましょう。

```bash
go run main.go
```

ここまでは通常の Go アプリケーションと同じです。

**注意事項：今回のコードはあくまでサンプルの実装になりますのでご注意ください。**

<walkthrough-footnote>アプリケーションを作成し、起動することができました。次に実際にアプリケーションにアクセスしてみます。</walkthrough-footnote>


<!-- Step 12 -->
## Cloud Shell 上でアプリケーションを起動する

### CloudShell の機能を利用し、起動したアプリケーションにアクセスする

画面右上にあるアイコン <walkthrough-web-preview-icon></walkthrough-web-preview-icon> をクリックし、"プレビューのポート: 8080"を選択します。
これによりブラウザで新しいタブが開き、Cloud Shell 上で起動しているコンテナにアクセスできます。

正しくアプリケーションにアクセスできると、 **Welcome to CA!** と表示されます。

確認が終わったら、Cloud Shell 上で Ctrl+c を入力して実行中のアプリケーションを停止します。

<walkthrough-footnote>ローカル環境（Cloud Shell 内）で動いているアプリケーションにアクセスできました。次にアプリケーションのコンテナ化をします。</walkthrough-footnote>


<!-- Step 13 -->
## Cloud Shell 上でコンテナ化されたアプリケーションを起動する

### コンテナを作成する

Go 言語で作成されたサンプル Web アプリケーションをコンテナ化します。
ここで作成したコンテナは Cloud Shell インスタンスのローカルに保存されます。

```bash
docker build -t gcr.io/$GOOGLE_CLOUD_PROJECT/ca-app:v1 .
```

**ヒント**: `docker build` コマンドを叩くと、Dockerfile が読み込まれ、そこに記載されている手順通りにコンテナが作成されます。

### Cloud Shell 上でコンテナを起動する

上の手順で作成したコンテナを Cloud Shell 上で起動します。

```bash
docker run -p 8080:8080 \
--name ca-app \
-e GOOGLE_APPLICATION_CREDENTIALS=/tmp/keys/auth.json \
-v $PWD/auth.json:/tmp/keys/auth.json:ro \
gcr.io/$GOOGLE_CLOUD_PROJECT/ca-app:v1
```

**ヒント**: Cloud Shell 環境の 8080 ポートを、コンテナの 8080 ポートに紐付け、フォアグラウンドで起動しています。

<walkthrough-footnote>アプリケーションをコンテナ化し、起動することができました。次に実際にアプリケーションにアクセスしてみます。</walkthrough-footnote>


<!-- Step 14 -->
## 作成したコンテナの動作確認

### CloudShell の機能を利用し、起動したアプリケーションにアクセスする

画面右上にあるアイコン <walkthrough-web-preview-icon></walkthrough-web-preview-icon> をクリックし、"プレビューのポート: 8080"を選択します。
これによりブラウザで新しいタブが開き、Cloud Shell 上で起動しているコンテナにアクセスできます。

正しくアプリケーションにアクセスできると、先程と同じように `Welcome to CA!` と表示されます。

確認が終わったら、Cloud Shell 上で Ctrl+c を入力して実行中のコンテナを停止します。


<walkthrough-footnote>ローカル環境（Cloud Shell 内）で動いているコンテナにアクセスできました。次に Cloud Run にデプロイするための準備を進めます。</walkthrough-footnote>


<!-- Step 15 -->
## コンテナのレジストリへの登録

先程作成したコンテナはローカルに保存されているため、他の場所から参照ができません。
他の場所から利用できるようにするために、GCP 上のプライベートなコンテナ置き場（コンテナレジストリ）に登録します。

### 作成したコンテナをコンテナレジストリ（Google Container Registry）へ登録（プッシュ）する

```bash
docker push gcr.io/$GOOGLE_CLOUD_PROJECT/ca-app:v1
```

**GUI**: [コンテナレジストリ](https://console.cloud.google.com/gcr/images/{{project-id}}?project={{project-id}})

<walkthrough-footnote>次に Cloud Run にコンテナをデプロイをします。</walkthrough-footnote>


<!-- Step 16 -->
## Cloud Run にコンテナをデプロイする

### gcloud コマンドで Cloud Run の Service を作成し、コンテナをデプロイします

Cloud Run の名前は ca-app にしています。

```bash
gcloud run deploy --image=gcr.io/$GOOGLE_CLOUD_PROJECT/ca-app:v1 \
  --service-account="dev-sa@{{project-id}}.iam.gserviceaccount.com" \
  --platform=managed \
  --region=us-central1 \
  --allow-unauthenticated \
  ca-app
```

**参考**: デプロイが完了するまで、1〜2分程度かかります。

**GUI**: [Cloud Run](https://console.cloud.google.com/run?project={{project-id}})

### Cloud Run の Service の URLを取得します
```bash
URL=$(gcloud run services describe --format=json --region=us-central1 --platform=managed ca-app | jq .status.url -r)
echo ${URL}
```

ブラウザから取得した URL を開いてアプリケーションの動作を確認します。

**GUI**: [Cloud Run サービス情報](https://console.cloud.google.com/run/detail/us-central1/ca-app/general?project={{project-id}})


<!-- Step 17 -->
## Cloud Runのログを確認します

### コンテナのログを確認
**GUI**: [Cloud Run ログ](https://console.cloud.google.com/run/detail/us-central1/ca-app/logs?project={{project-id}})

アクセスログを確認します。

<walkthrough-footnote>Cloud Run にサンプル Web アプリケーションがデプロイされました。次は Cloud Build の設定に入ります。</walkthrough-footnote>



<!-- Step 18 -->
## Cloud Build によるビルド、デプロイの自動化

Cloud Build を利用し今まで手動で行っていたアプリケーションのビルド、コンテナ化、リポジトリへの登録、Cloud Run へのデプロイを自動化します。

### Cloud Build サービスアカウントへの権限追加

Cloud Build を実行する際に利用されるサービスアカウントを取得し、環境変数に格納します。

```bash
export CB_SA=$(gcloud projects get-iam-policy $GOOGLE_CLOUD_PROJECT | grep cloudbuild.gserviceaccount.com | uniq | cut -d ':' -f 2)
```

上で取得したサービスアカウントに Cloud Build から自動デプロイをさせるため Cloud Run 管理者の権限を与えます。

```bash
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT  --member serviceAccount:$CB_SA --role roles/run.admin
```

```bash
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT  --member serviceAccount:$CB_SA --role roles/iam.serviceAccountUser
```

<walkthrough-footnote>Cloud Build で利用するサービスアカウントに権限を付与し、Cloud Run に自動デプロイできるようにしました。</walkthrough-footnote>


<!-- Step 19 -->
## Cloud Build によるビルド、デプロイの自動化

### cloudbuild.yaml の確認

Cloud Build のジョブの中身は `newgrad` フォルダ下にある `cloudbuild.yaml` に定義されているので中身を確認してみましょう。

```
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/ca-app:$BUILD_ID', '.']

- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/ca-app:$BUILD_ID']

- name: 'gcr.io/cloud-builders/gcloud'
  args: [
    'run',
    'deploy',
    '--image=gcr.io/$PROJECT_ID/ca-app:$BUILD_ID',
    '--service-account=dev-sa@$PROJECT_ID.iam.gserviceaccount.com',
    '--platform=managed',
    '--region=us-central1',
    '--allow-unauthenticated',
    '--set-env-vars',
    'GOOGLE_CLOUD_PROJECT=$PROJECT_ID',
    'ca-app',
  ]
```

docker build コマンドでコンテナをビルドした際は、コンテナのタグを `gcr.io/{{project-id}}/ca-app:v1` としていましたが、Cloud Build では `gcr.io/{{project-id}}/ca-app:$BUILD_ID` としている事が分かります。$BUILD_ID には Cloud Build のジョブの ID が入ります。

<walkthrough-footnote>それでは　Cloud Build のジョブを実行してみましょう。</walkthrough-footnote>


<!-- Step 20 -->
## Cloud Build によるビルド、デプロイの自動化

### Cloud Build のジョブの実行

```bash
gcloud builds submit --config cloudbuild.yaml .
```

** ヒント **: コマンド末尾の `.` は、 `cloudbuild.yaml` がカレント ディレクトリに存在することを指しています。


### ジョブの確認

[Cloud Build の履歴](https://console.cloud.google.com/cloud-build/builds?project={{project-id}}) にアクセスし、ビルドが実行されていることを確認します。


### Cloud Run の確認

Cloud Run のコンテナの Image URL が Cloud Build で作成されたイメージになっていることを確認します。

**GUI**: [Cloud Run リビジョン](https://console.cloud.google.com/run/detail/us-central1/ca-app/revisions?project={{project-id}})


<walkthrough-footnote>Cloud Build による自動ビルド・デプロイの設定が完了しました。次は Firestore の実装に入ります。</walkthrough-footnote>


<!-- Step 21 -->
## Firestore の利用

サンプル Web アプリケーションが Firestore を利用するように編集していきます。今回は、基本的な CRUD 処理を実装します。

### 依存関係の追加

Firestore にアクセスするためにクライアントライブラリを追加します。
Go 言語の場合、 `go.mod` で Go パッケージの依存関係を設定できます。

今回のハンズオンで使う依存関係を全て書いた `go.mod` ファイルは既に `newgrad` フォルダに配置済みです。

```
module github.com/mouradelazhari/gcp-getting-started-lab-jp/webapp/newgrad

go 1.13

require (
	cloud.google.com/go/firestore v1.3.0
	github.com/gomodule/redigo v2.0.0+incompatible
	golang.org/x/net v0.0.0-20200904194848-62affa334b73 // indirect
	golang.org/x/oauth2 v0.0.0-20200902213428-5d25da1a8d43 // indirect
	golang.org/x/sys v0.0.0-20200909081042-eff7692f9009 // indirect
	golang.org/x/tools v0.0.0-20200913032122-97363e29fc9b // indirect
	google.golang.org/api v0.31.0
	google.golang.org/genproto v0.0.0-20200911024640-645f7a48b24f // indirect
	google.golang.org/grpc v1.32.0 // indirect
)
```

<walkthrough-footnote>Firestore を操作するコードを実装していきましょう。</walkthrough-footnote>



<!-- Step 22 -->
## Firestore の利用

### データの追加・取得機能

**このステップで作成したコードは answer/step22/main.go になります。**

`main.go` ファイルに以下のコードを追加します。
まずは import の中に以下を追記してください。

```go
"encoding/json"
"io"
"strconv"
"cloud.google.com/go/firestore"
"google.golang.org/api/iterator"
```

次に、main 関数にハンドラを追加します。

```go
	http.HandleFunc("/", indexHandler) // ここは既存の行
	http.HandleFunc("/firestore", firestoreHandler)
```

次に、Firestoreにリクエストのデータを追加するコードを追加します。一番下に以下のコードを追記してください。

```go
func firestoreHandler(w http.ResponseWriter, r *http.Request) {

	// Firestore クライアント作成
	pid := os.Getenv("GOOGLE_CLOUD_PROJECT")
	ctx := r.Context()
	client, err := firestore.NewClient(ctx, pid)
	if err != nil {
		log.Fatal(err)
	}
	defer client.Close()

	switch r.Method {
	// 追加処理
	case http.MethodPost:
		u, err := getUserBody(r)
		if err != nil {
			log.Fatal(err)
			w.WriteHeader(http.StatusInternalServerError)
			return
		}
		ref, _, err := client.Collection("users").Add(ctx, u)
		if err != nil {
			log.Fatalf("Failed adding data: %v", err)
			w.WriteHeader(http.StatusInternalServerError)
			return
		}
		log.Print("success: id is %v", ref.ID)
		fmt.Fprintf(w, "success: id is %v \n", ref.ID)

	// 取得処理
	case http.MethodGet:
		iter := client.Collection("users").Documents(ctx)
		var u []Users

		for {
			doc, err := iter.Next()
			if err == iterator.Done {
				break
			}
			if err != nil {
				log.Fatal(err)
			}
			var user Users
			err = doc.DataTo(&user)
			if err != nil {
				log.Fatal(err)
			}
			user.Id = doc.Ref.ID
			log.Print(user)
			u = append(u, user)
		}
		if len(u) == 0 {
			w.WriteHeader(http.StatusNoContent)
		} else {
			json, err := json.Marshal(u)
			if err != nil {
				w.WriteHeader(http.StatusInternalServerError)
				return
			}
			w.Write(json)
		}

	// それ以外のHTTPメソッド
	default:
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}
}

type Users struct {
	Id    string `firestore:id, json:id`
	Email string `firestore:email, json:email`
	Name  string `firestore:name, json:name`
}

func getUserBody(r *http.Request) (u Users, err error) {
	length, err := strconv.Atoi(r.Header.Get("Content-Length"))
	if err != nil {
		return u, err
	}

	body := make([]byte, length)
	length, err = r.Body.Read(body)
	if err != nil && err != io.EOF {
		return u, err
	}

	//parse json
	err = json.Unmarshal(body[:length], &u)
	if err != nil {
		return u, err
	}
	log.Print(u)
	return u, nil
}

```

こちらのコードは実際のプロジェクトの Firestore にデータを追加、または Firestore からデータを取得しようとしています。

<walkthrough-footnote>次のステップでコードをデプロイし、動作を確認してみましょう。</walkthrough-footnote>


<!-- Step 23 -->
## デプロイと確認 (Firestore 登録・取得機能の追加)

### Cloud Run へのデプロイ

Cloud Build を実行し、アプリケーションを Cloud Run にデプロイしてみましょう。

```bash
gcloud builds submit --config cloudbuild.yaml .
```

### URL の表示

以下のコマンドで URL を表示します。

```bash
echo $URL
```

### Firestore を利用した操作の実施

Cloud Shell から Cloud Run の Service の URL に対して、以下のような cURL コマンドをいくつか実行し、データの登録・取得の処理が行われることを確認しましょう。

**登録**

```
curl -X POST -d '{"email":"newgrad@example.com", "name":"New Grad"}' ${URL}/firestore
```

**取得（全件）**

```
curl ${URL}/firestore
```

<walkthrough-footnote>次は登録済みのデータを更新・削除する実装を行います。</walkthrough-footnote>


<!-- Step 24 -->
## Firestore の利用
### データの更新・削除処理

**このステップで作成したコードは answer/step24/main.go になります。**

先程のステップで実装したデータの登録処理では、各データに対して一意な ID が付与されていました。
ここでは、その ID を用いてデータを更新・削除する処理を追加します。

更新 API は Doc に ID の値をセットすることで一意なユーザーデータを対象にし、Set 関数で受け取ったリクエストの内容で更新します。
削除 API はパスパラメータで ID を指定する形式にしています。

`main.go` の import の中に以下を追記してください。

```go
"strings"
```

main 関数の HandleFunc に以下を追加します。

```go
	http.HandleFunc("/firestore/", firestoreHandler)
```

続いて `MethodGet` の case 句の後に以下の case 句を追記しましょう。

```go
	// 更新処理
	case http.MethodPut:
		u, err := getUserBody(r)
		if err != nil {
			log.Fatal(err)
			w.WriteHeader(http.StatusInternalServerError)
			return
		}

		_, err = client.Collection("users").Doc(u.Id).Set(ctx, u)
		if err != nil {
			w.WriteHeader(http.StatusInternalServerError)
			return
		}

		fmt.Fprintln(w, "success updating")

	// 削除処理
	case http.MethodDelete:
		id := strings.TrimPrefix(r.URL.Path, "/firestore/")
		_, err := client.Collection("users").Doc(id).Delete(ctx)
		if err != nil {
			w.WriteHeader(http.StatusInternalServerError)
			return
		}
		fmt.Fprintln(w, "success deleting")
```

<walkthrough-footnote>データ更新・削除の機能を実装したので Cloud Run にデプロイしましょう。</walkthrough-footnote>


<!-- Step 25 -->
## デプロイと確認 (Firestore 更新・削除機能の追加)

### Cloud Run へのデプロイ

Cloud Build を実行し、アプリケーションを Cloud Run にデプロイしてみましょう。

```bash
gcloud builds submit --config cloudbuild.yaml .
```

### URL の表示

以下のコマンドで URL を表示します。

```bash
echo $URL
```

### Firestore を利用した操作の実施

Cloud Shell から Cloud Run の Service の URL に対して、以下のような cURL コマンドをいくつか実行し、データの更新・削除の処理が行われることを確認しましょう。

**更新**

`<ID>` へはコンソールなどで確認した `id` の値をセットしてください。

![firestore-id](https://storage.googleapis.com/egg-resources/egg1/public/firestore-id.jpg)

```
curl -X PUT -d '{"id": "<ID>", "email":"cloud@example.com", "name":"Cloud Taro"}' ${URL}/firestore
```

**削除**

`<ID>` へは削除する `id` の値を指定してください。

```
curl -X DELETE ${URL}/firestore/<ID>
```

<walkthrough-footnote>Firestore についての実装は以上になります。</walkthrough-footnote>

## Congraturations!

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

これにて Cloud Run を使ったアプリケーション開発のハンズオンは完了です！！

デモで使った資材が不要な方は、次の手順でクリーンアップを行って下さい。

## クリーンアップ（プロジェクトを削除）

作成したリソースを個別に削除する場合は、こちらのページの手順を実施せずに次のページに進んで下さい。

### プロジェクトの削除

```bash
gcloud projects delete {{project-id}}
```

## クリーンアップ（個別リソースの削除）

### Cloud Run の削除

```bash
gcloud run services delete ca-app --platform managed --region=us-central1
```

### Firestore データの削除

Firestore コンソールから、ルートコレクションを削除してください。今回のハンズオンで作成したすべての user データが削除されます。

### Container Registry に登録したコンテナイメージの削除

Container Registry コンソールから、イメージを選択して削除してください。

### Cloud Source Repositories に作成したリポジトリの削除

[CSR の設定画面](https://source.cloud.google.com/admin/settings?projectId={{project-id}}&repository=ca-handson) にアクセスし、「このリポジトリを削除」を実行

### Owner 権限をつけた dev-key.json の削除

```bash
rm ~/cloudshell_open/gcp-getting-started-lab-jp/webapp/newgrad/dev-key.json
```

### サービスアカウントに付与したロールの取り消し

```bash
gcloud projects remove-iam-policy-binding {{project-id}} --member "serviceAccount:dev-sa@{{project-id}}.iam.gserviceaccount.com" --role "roles/owner"
```

### サービスアカウントの削除

```bash
gcloud iam service-accounts delete dev-sa@{{project-id}}.iam.gserviceaccount.com
```
