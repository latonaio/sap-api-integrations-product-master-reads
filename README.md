# sap-api-integrations-product-master-reads  
sap-api-integrations-product-master-reads は、外部システム(特にエッジコンピューティング環境)をSAPと統合することを目的に、SAP API で品目マスタデータを取得するマイクロサービスです。  
sap-api-integrations-product-master-reads には、サンプルのAPI Json フォーマットが含まれています。  
sap-api-integrations-product-master-reads は、オンプレミス版である（＝クラウド版ではない）SAPS4HANA API の利用を前提としています。クラウド版APIを利用する場合は、ご注意ください。  
https://api.sap.com/api/OP_API_PRODUCT_SRV_0001/overview  

## 動作環境  
sap-api-integrations-product-master-reads は、主にエッジコンピューティング環境における動作にフォーカスしています。  
使用する際は、事前に下記の通り エッジコンピューティングの動作環境（推奨/必須）を用意してください。  
・ エッジ Kubernetes （推奨）   
・ AION のリソース （推奨)   
・ OS: LinuxOS （必須）   
・ CPU: ARM/AMD/Intel（いずれか必須）  

## クラウド環境での利用
sap-api-integrations-product-master-reads は、外部システムがクラウド環境である場合にSAPと統合するときにおいても、利用可能なように設計されています。


## 本レポジトリ が 対応する API サービス
sap-api-integrations-product-master-reads が対応する APIサービス は、次のものです。

* APIサービス概要説明 URL: https://api.sap.com/api/OP_API_PRODUCT_SRV_0001/overview  
* APIサービス名(=baseURL): API_PRODUCT_SRV

## 本レポジトリ に 含まれる API名
sap-api-integrations-product-master-reads には、次の API をコールするためのリソースが含まれています。  

* A_Product（品目マスタ - 一般データ）
* A_ProductPlant（品目マスタ - プラントデータ）
* A_ProductPlantMRPArea（品目マスタ - MRPエリアデータ）
* A_ProductPlantProcurement（品目マスタ - 購買データ）
* A_ProductWorkScheduling（品目マスタ - 作業計画データ）
* A_ProductPlantSales（品目マスタ - 販売プラントデータ）
* A_ProductValuation（品目マスタ - 評価エリアデータ）
* A_ProductSalesDelivery（品目マスタ - 販売組織データ）
* A_ProductPlantQualityMgmt（品目マスタ - 品質管理データ）
* A_ProductSalesTax（品目マスタ - 販売税データ）
* A_ProductDescription（品目マスタ - テキストデータ）
* ToProductDesc（品目マスタ - テキストデータ ※To）

## API への 値入力条件 の 初期値
sap-api-integrations-product-master-reads において、API への値入力条件の初期値は、入力ファイルレイアウトの種別毎に、次の通りとなっています。  

### SDC レイアウト

* inoutSDC.Product.Product（品目）
* inoutSDC.Product.Plant.Plant（プラント）
* inoutSDC.Product.Plant.MRPArea.MRPArea（MRPエリア）
* inoutSDC.Product.Accounting.ValuationArea（評価エリア）
* inoutSDC.Product.SalesOrganization.ProductSalesOrg（販売組織）
* inoutSDC.Product.SalesOrganization.ProductDistributionChnl（流通チャネル）
* inoutSDC.Product.ProductDescription.Language（言語キー）
* inoutSDC.Product.ProductDescription.ProductDescription（品目テキスト）
* inoutSDC.Product.SalesTax.Country（国）
* inoutSDC.Product.SalesTax.TaxCategory（税カテゴリ）

## SAP API Bussiness Hub の API の選択的コール

Latona および AION の SAP 関連リソースでは、Inputs フォルダ下の sample.json の accepter に取得したいデータの種別（＝APIの種別）を入力し、指定することができます。  
なお、同 accepter にAll(もしくは空白)の値を入力することで、全データ（＝全APIの種別）をまとめて取得することができます。  

* sample.jsonの記載例(1)  

accepter において 下記の例のように、データの種別（＝APIの種別）を指定します。  
ここでは、"General", "Plant", "Accounting" が指定されています。    
  
```
  "api_schema": "sap.s4.beh.product.v1.Product.Created.v1",
  "accepter": ["General", "Plant", "Accounting"],
  "material_code": "21",
  "deleted": false
```
  
* 全データを取得する際のsample.jsonの記載例(2)  

全データを取得する場合、sample.json は以下のように記載します。  

```
  "api_schema": "sap.s4.beh.product.v1.Product.Created.v1",
  "accepter": ["All"],
  "material_code": "21",
  "deleted": false
```
## 指定されたデータ種別のコール

accepter における データ種別 の指定に基づいて SAP_API_Caller 内の caller.go で API がコールされます。  
caller.go の func() 毎 の 以下の箇所が、指定された API をコールするソースコードです。  

```
func (c *SAPAPICaller) AsyncGetProductMaster(product, plant, mrpArea, valuationArea, productSalesOrg, productDistributionChnl, language, productDescription, country, taxCategory string, accepter []string) {
	wg := &sync.WaitGroup{}
	wg.Add(len(accepter))
	for _, fn := range accepter {
		switch fn {
		case "General":
			func() {
				c.General(product)
				wg.Done()
			}()
		case "Plant":
			func() {
				c.Plant(product, plant)
				wg.Done()
			}()
		case "MRPArea":
			func() {
				c.MRPArea(product, plant, mrpArea)
				wg.Done()
			}()
		case "Procurement":
			func() {
				c.Procurement(product, plant)
				wg.Done()
			}()
		case "WorkScheduling":
			func() {
				c.WorkScheduling(product, plant)
				wg.Done()
			}()
		case "SalesPlant":
			func() {
				c.SalesPlant(product, plant)
				wg.Done()
			}()
		case "Accounting":
			func() {
				c.Accounting(product, valuationArea)
				wg.Done()
			}()
		case "SalesOrganization":
			func() {
				c.SalesOrganization(product, productSalesOrg, productDistributionChnl)
				wg.Done()
			}()
		case "ProductDescByProduct":
			func() {
				c.ProductDescByProduct(product, language)
				wg.Done()
			}()
		case "ProductDescByDesc":
			func() {
				c.ProductDescByDesc(language, productDescription)
				wg.Done()
			}()
		case "Quality":
			func() {
				c.Quality(product, plant)
				wg.Done()
			}()
		case "SalesTax":
			func() {
				c.SalesTax(product, country, taxCategory)
				wg.Done()
			}()
		default:
			wg.Done()
		}
	}

	wg.Wait()
}
```

## Output  
本マイクロサービスでは、[golang-logging-library-for-sap](https://github.com/latonaio/golang-logging-library-for-sap) により、以下のようなデータがJSON形式で出力されます。  
以下の sample.json の例は、SAP 品目マスタ の 一般データ が取得された結果の JSON の例です。  
以下の項目のうち、"Material" ～ "ProductStandardID" は、/SAP_API_Output_Formatter/type.go 内 の Type General {} による出力結果です。"cursor" ～ "time"は、golang-logging-library-for-sap による 定型フォーマットの出力結果です。  

```
{
	"cursor": "/home/ampamman/go/src/sap-api-integrations-product-master-reads/SAP_API_Caller/caller.go#L108",
	"function": "sap-api-integrations-product-master-reads/SAP_API_Caller.(*SAPAPICaller).General",
	"level": "INFO",
	"message": [
		{
			"Product": "A001",
			"IndustrySector": "M",
			"ProductType": "SERV",
			"BaseUnit": "AU",
			"ValidityStartDate": "",
			"ProductGroup": "A001",
			"Division": "00",
			"GrossWeight": "0.000",
			"WeightUnit": "KG",
			"SizeOrDimensionText": "",
			"ProductStandardID": "",
			"CreationDate": "2016-06-20T09:00:00+09:00",
			"LastChangeDate": "2017-04-11T09:00:00+09:00",
			"IsMarkedForDeletion": false,
			"NetWeight": "0.000",
			"ChangeNumber": "",
			"to_Description": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_PRODUCT_SRV/A_Product('A001')/to_Description"
		}
	],
	"time": "2022-01-26T14:51:52.138052513+09:00"
}
```
