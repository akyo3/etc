# Grafanaアラート設定方法

本手順は、各ノードが停止した際に、以下のアプリにアラートの通知が届くように設定する手順です。  

通知先アプリ：LINE、Discord、Telegram、Slack
>Emailは特別な設定が必要なので非推奨とさせていただきます。

---

### Grafanaバージョン確認方法

Grafana Ver.8以降の手順となりますので実施前にGrafanaのバージョンを確認します。  

![Grafana-Alert-1](https://user-images.githubusercontent.com/80967103/167249824-ca90010e-a882-4f7c-9ca4-936bc6037439.png)
>Grafanaのバージョンは、左下の？マークにマウスカーソルを乗せることで確認できます。

<details>
<summary>Grafana Ver.8以前にアラート設定を作成されていた方はこちら</summary>

1. 左のメニューの鈴マークから、「Alert rules」を選択。
2. 既存のアラート一覧が表示されるので、「Edit alert」を選択。
3. アラート設定の一番下の「Delete」を選択して、左上の矢印(← Go Back)をクリックまたはEscを押下。
4. 右上の「Save Dashboard」を選択し、「Save」を選択。
5. Alert rulesのアラート設定が全て無くなるまで繰り返してください。
    - 終えたら、grafana-server.serviceを再起動します。

`Grafanaを搭載しているサーバ`  
<div>

```console
sudo systemctl restart grafana-server.service
```

</div>

その後、ブラウザを更新すると新バージョンとなっています。
> ブラウザを更新しても新バージョンとなっていない場合は一度Sign OutしてSign Inしてみてください。
</details>

---

## 1- ダッシュボードフォルダの新規作成
左メニューの＋マークから**Folder**を選択し、**Browse**配下の**Folder name**にお好きな名前を入力し、**Create**をクリックします。  
例）　`Cardano Alert`  

![Grafana-Alert-2](https://user-images.githubusercontent.com/80967103/167246590-2e0711bf-d858-4531-87bc-0ac62d9fef71.png)

___

## 2- アラートルール作成
左メニューの鈴マークから**Alert rules**を選択し、**New alert rule**をクリックします。

![Grafana-Alert-3](https://user-images.githubusercontent.com/80967103/167246875-c518133b-433e-4764-a888-675d315abe04.png)

___

## 3- BPのアラート設定

1. **Rule name**に以下を入力します。（任意変更可能）
```console
BP Alert
```

2. **Rule type**は `Grafana managed alert` を選択します。

3. **Folder**は、先ほど作成したフォルダを選択してください。  
> 例）　`Cardano Alert`

4. **Metrics browser**の欄に以下のメトリクスを入力します。
> <BPのIP> はご自身のBPのIPを入力してください。

```
cardano_node_metrics_slotInEpoch_int{instance="<BPのIP>:12798", job="prometheus", type="cardano-node"}
```

5. **Legend**の欄は以下を入力します。
```console
{{alias}}
```

6. **instant**を切り替えてオンにします。

7. **Conditions**の `IS ABOVE` を `IS BELOW`へ変更し、 `0` にします。

8. **Configure no data and error handling**を展開し、**Alert state if no data or all values are null**を`Alerting`に変更します。

9. 上記がすべて完了した後、右上の**Save and exit**をクリックします。

![Grafana-Alert-4](https://user-images.githubusercontent.com/80967103/167262207-9830a19a-4a13-4d7d-ba2e-47c42fc53740.png)

___

## 4- リレーのアラート設定
**New alert rule**を選択し、同じ要領でリレーのアラートを作成します。  
> 変更箇所は**Rule name**と**Metrics browser**です。  
> その他はBPのアラート作成と同じです。

![Grafana-Alert-5](https://user-images.githubusercontent.com/80967103/167258212-cd155970-bacc-4fa7-a9c5-91c8129745e2.png)


- **Rule name**に以下を入力します。（任意変更可能）
```console
Relay Alert
```

- **Metrics browser**の欄に以下のメトリクスを入力します。
```console
cardano_node_metrics_slotInEpoch_int{instance="localhost:12798", job="prometheus", type="cardano-node"}
```
___

## 5- 通知設定
**Contact points**を選択し**New contact point**をクリックします。  

![Grafana-Alert-6](https://user-images.githubusercontent.com/80967103/167258571-ac15c31b-554b-4feb-a0fb-9c4400420bb9.png)

___

## 6- アラート通知先設定
アラートをどのアプリで通知するかを設定します。
> 例）Discord

1. **Name**にはお好きな名前を入力してください。
2. **Contact point type**にて、アラートを通知するアプリを選択します。
3. **Webhook URL**を入力します。
> 通知先アプリのURLデータの入手方法は[**ブロック生成ステータス通知セットアップ**](https://docs.spojapanguild.net/setup/11-blocknotify-setup/#11-2)に纏められています。  
>
>アラート状態が回復したという通知が不要の方は**Notification setting**の**Disable resolved message**にチェックを入れてください。
4. 入力後、**Save contact point**をクリックします。

![Grafana-Alert-7](https://user-images.githubusercontent.com/80967103/167259161-070631c6-8c67-45f0-87a9-48aefefb0ceb.png)

___

## 7- 通知ポリシー設定
1. **Notification policies**に移動し、**New specific policy**をクリックします。  

![Grafana-Alert-8](https://user-images.githubusercontent.com/80967103/167262690-f043de35-5754-4bae-8dd5-aa15454dc031.png)

2. **Contact point**を先ほど作成したアラート設定に変更し、**Save policy**をクリックします。  

![Grafana-Alert-9](https://user-images.githubusercontent.com/80967103/167260395-f2cd515e-540d-416d-94cf-bd9c1bd8c17b.png)

___

## 通知確認  
ブロック生成予定の無いタイミングでノードを停止し、5分ほど待ってアラートがアプリに届くか確認してください。  

ノード停止コマンド
```console
sudo systemctl stop cardano-node
```

ノード起動コマンド
```console
sudo systemctl start cardano-node
```
