# CFS で Azure ML の Web サービス を呼び出す構成の一例

Connected Field Service for Dynamics 365 (CFS)  で Azure ML で用意されている Web サービスを呼び出す構成の一例として、クエリを中心に説明します。あくまでサンプルです。  

## 動作確認環境
* Dynamics 365 (CRM) Version 1612 (8.2.0.773)

## 事前作業
* CFS の初期セットアップをします。  
	* 以下のページを参考にセットアップをします。  
		Use Connected Field Service to remotely monitor and service customer equipment (field service)  
		https://www.microsoft.com/en-US/dynamics/crm-customer-center/use-connected-field-service-to-remotely-monitor-and-service-customer-equipment-field-service.aspx
	* デバイスのシミュレーターから異常値（70度以上）を発生させて、IoT 通知レコードが生成されることを確認ください。
* Azure ML のギャラリーから、以下を構成します。この例では、Step 1 of 3、Step 2A of 3、Step 3A of 3を実行して、Web サービスを構成しておきます。  
		Predictive Maintenance: Step 1 of 3, data preparation and feature engineering  
		https://gallery.cortanaintelligence.com/Experiment/Predictive-Maintenance-Step-1-of-3-data-preparation-and-feature-engineering-2
* Azure の CFS のリソースグループ内の Logic App 「IoT-To-CRM」をコピーして適宜名前を付けます。"label" という引数に、「故障するまで後何日残っているか」という日数が格納され、受け取ります。Dynamics 365 で作成されるレコードの「説明」フィールドの値などに適宜それを含めます。  
* Azure の CFS のリソースグループ内のストレージ アカウント内の BLOB に格納されているファイル devicerules.json をコピーしてオリジナルを保存します。「故障するまで後何日残っているか」の日数に対して、ある一定の閾値を超えたものを Dynamics 365 に取り込むための条件となる情報を devicerules.json に追加します。サンプルはこちら [devicerules.json](https://github.com/keijiinoue/CFS-AzureML/blob/master/devicerules.json "devicerules.json") に置いておきます。  

## 手順
1. 新規のキューの作成
	* Azure の CFS のリソースグループ内に既定で作成されている Service Bus の中に、新規でキューを作成します。  

1. Logic App とキューの関連付け
	* 事前作業で作成したLogic App を編集し、先ほど作成したキューでメッセージを受け取った場合に開始するようにします。  

1. 新規の Stream Analytics ジョブの作成
	* Azure の CFS のリソースグループ内に、新規に Stream Analytics ジョブを作成します。  
	* CFS で既定で作成される Stream Analytics ジョブ には2つありますが、出力が「AlertsQueue」に設定されているものと2つの同じ入力を設定します。  
	* 出力は、適宜名前を付け、先ほど作成したキューに設定します。
	* クエリは、こちらのサンプル [Query.txt
](https://github.com/keijiinoue/CFS-AzureML/blob/master/Query.txt "Query.txt") の通りに設定します。センサーデータの平均値(AVG)や標準偏差(STDEV)の値を Azure ML の Web サービスのパラメーターとして渡すための GROUP BY 句などが重要です。  
	* Stream Analytics ジョブを開始します。  
	* シミュレーターでデバイスからデータを送ると、Dynamics 365 で[IoT 通知」エンティティレコードが作成されることを確認します。  

