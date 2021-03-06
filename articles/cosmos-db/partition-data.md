---
title: "Azure Cosmos DB でのパーティション分割と水平スケーリング | Microsoft Docs"
description: "Azure Cosmos DB でのパーティション分割のしくみ、パーティション分割とパーティション キーを構成する方法、アプリケーションに適したパーティション キーを選択する方法について説明します。"
services: cosmos-db
author: arramac
manager: jhubbard
editor: monicar
documentationcenter: 
ms.assetid: cac9a8cd-b5a3-4827-8505-d40bb61b2416
ms.service: cosmos-db
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/29/2017
ms.author: arramac
ms.custom: H1Hack27Feb2017
ms.translationtype: HT
ms.sourcegitcommit: f2ac16c2f514aaa7e3f90fdf0d0b6d2912ef8485
ms.openlocfilehash: 6f272136d535dddd9c8213293841ace203c042a1
ms.contentlocale: ja-jp
ms.lasthandoff: 09/08/2017

---

# <a name="partition-and-scale-in-azure-cosmos-db"></a>Azure Cosmos DB でのパーティション分割とスケーリング

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) は、世界規模で分散配置されるマルチモデルのデータベース サービスであり、高速で予測可能なパフォーマンスを実現するうえで役立つよう設計されています。 アプリケーションの規模の拡大に合わせてスケーリングがシームレスに行われます。 この記事では、Azure Cosmos DB におけるあらゆるデータ モデルのパーティション分割のしくみについて概説します。 また、Azure Cosmos DB コンテナーを構成してアプリケーションを効率的にスケーリングする方法について説明します。

パーティション分割とパーティション キーについては、Scott Hanselman と Azure Cosmos DB プリンシパル エンジニアリング マネージャー Shireesh Thota による次の Azure Friday 動画で説明されています。

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Azure-DocumentDB-Elastic-Scale-Partitioning/player]
> 

## <a name="partitioning-in-azure-cosmos-db"></a>Azure Cosmos DB でのパーティション分割
Azure Cosmos DB では、ミリ秒の応答時間順に任意の規模でスキーマのないデータを格納し、クエリを実行できます。 Azure Cosmos DB は、(ドキュメントの) "*コレクション*"、"*グラフ*"、または "*テーブル*" と呼ばれるデータを格納するためのコンテナーを提供します。 コンテナーは論理リソースであり、1 つ以上の物理パーティションまたはサーバーにまたがることができます。 パーティションの数は、ストレージのサイズとコンテナーのプロビジョニング済みスループットに基づいて Azure Cosmos DB によって決定されます。 Azure Cosmos DB のすべてのパーティションには SSD を使用した固定サイズのストレージが関連付けられ、またパーティションは可用性を高めるためにレプリケートされます。 パーティションの管理は Azure Cosmos DB で完全に行われるため、複雑なコードを記述してパーティションを管理する必要がありません。 Azure Cosmos DB のコンテナーは、ストレージとスループットに関して、事実上制限がありません。 

![リソースのパーティション分割](./media/introduction/azure-cosmos-db-partitioning.png) 

パーティション分割は、アプリケーションに対して透過的です。 Azure Cosmos DB では、単一のコンテナー リソースへのメソッド/API 呼び出しを介したきめ細かなアクセス制御、高速な読み取りと書き込み、クエリ、トランザクション ロジック、整合性レベルがサポートされます。 サービスでは、複数のパーティション間でのデータの分散、適切なパーティションへのクエリ要求のルーティングが処理されます。 

パーティション分割は、どのようなしくみになっているのでしょうか。 それぞれの項目には、パーティション キーと行キーが必要です。パーティション キーと行キーによって項目が一意に識別されます。 パーティション キーは、データの論理上のパーティションとして機能し、複数のパーティションに対してデータを分散配置する際の自然境界を Azure Cosmos DB に提供します。 以下、Azure Cosmos DB におけるパーティション分割のしくみを簡単に説明します。

* `T` 要求/秒スループットで Azure Cosmos DB コンテナーをプロビジョニングします。
* `T` 要求/秒のスループットを得るために必要なパーティションを背後で Azure Cosmos DB がプロビジョニングします。 パーティションあたりの最大スループット `t` を `T` が超えている場合、Azure Cosmos DB は、`N` ( = `T/t`) 個のパーティションをプロビジョニングします。
* Azure Cosmos DB が `N` 個のパーティションに対し、パーティション キー ハッシュのキー空間を均等に割り当てます。 したがって、それぞれのパーティション (物理パーティション) は 1 ～ N のパーティション キー値 (論理パーティション) をホストすることになります。
* 物理パーティション `p` がそのストレージの上限に達すると、Azure Cosmos DB は `p` を 2 つの新しいパーティション (`p1` と `p2`) にシームレスに分割し、 おおよそ半分のキーに対応する値を各パーティションに分散配置します。 この分割操作は、アプリケーションからは見えません。
* 同様に、`t*N` を超えるスループットをプロビジョニングすると、そのスループット引き上げに対応するために、既存のパーティションが Azure Cosmos DB によって分割されます。

パーティション キーのセマンティクスは、次の表に示したように、API ごとのセマンティクスに応じて若干異なります。

| API | パーティション キー | 行キー |
| --- | --- | --- |
| Azure Cosmos DB | カスタム パーティション キーのパス | `id` (固定) | 
| MongoDB | カスタム共有キー  | `_id` (固定) | 
| Graph | カスタム パーティション キーのプロパティ | `id` (固定) | 
| テーブル | `PartitionKey` (固定) | `RowKey` (固定) | 

Azure Cosmos DB には、ハッシュベースのパーティション分割が使用されています。 何か項目を書き込むと、Azure Cosmos DB がパーティション キー値をハッシュし、そのハッシュした結果を基に項目の格納先のパーティションを決定します。 同じパーティション キーを備えた項目はすべて同じ物理パーティションに格納されます。 パーティション キーの選択は、設計時に行う必要のある重要な決定事項の 1 つです。 値が多岐にわたっていて、なおかつ均等なアクセス パターンを持ったプロパティ名を選択する必要があります。

> [!NOTE]
> 多数の個別の値 (少なくとも数百から数千) を備えたパーティション キーにすることをお勧めします。
>

Azure Cosmos DB コンテナーは、"*固定*" または "*無制限*" として作成することができます。 固定サイズのコンテナーの上限は、容量が 10 GB で、スループットが毎秒 10,000 RU となります。 固定サイズのコンテナーに関して、一部の API ではパーティション キーを省略することができます。 コンテナーを無制限として作成する場合は、スループットに最低でも 2,500 RU/秒を指定する必要があります。

## <a name="partitioning-and-provisioned-throughput"></a>パーティション分割とプロビジョニング済みスループット
Azure Cosmos DB は、予測可能なパフォーマンスが得られるように設計されています。 コンテナーを作成するときに、*1 秒あたりの[要求ユニット](request-units.md) (RU)* でスループットを予約します。 各要求には、操作で使用されるシステム リソースの量 (CPU、メモリ、IO など) に比例する、RU の負担が割り当てられます。 セッション整合性での 1 KB のドキュメントの読み取りでは、1 RU を使用します。 読み取りは、格納された項目の数や、同時に実行する同時要求の数にかかわらず 1 RU になります。 サイズの大きい項目では、サイズに応じた、さらに多くの RU が必要になります。 アプリケーションにサポートする必要のあるエンティティのサイズと読み取りの数がわかっている場合は、アプリケーションの読み取りに必要な正確な量のスループットをプロビジョニングできます。 

> [!NOTE]
> コンテナーのフル スループットを達成するには、個々のパーティション キー値に対して要求を均等に分散できるパーティション キーを選択する必要があります。
> 
> 

<a name="designing-for-partitioning"></a>
## <a name="work-with-the-azure-cosmos-db-apis"></a>Azure Cosmos DB API の操作
コンテナーは、Azure Portal または Azure CLI を使って作成し、いつでもスケーリングすることができます。 このセクションでは、コンテナーを作成してスループットとパーティション キーの定義を指定する方法を、サポートされている API ごとに紹介します。

### <a name="azure-cosmos-db-api"></a>Azure Cosmos DB API
次のサンプルは、Azure Cosmos DB API を使ってコンテナー (コレクション) を作成する方法を示しています。 

```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey);
await client.CreateDatabaseAsync(new Database { Id = "db" });

DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "coll";
myCollection.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("db"),
    myCollection,
    new RequestOptions { OfferThroughput = 20000 });
```

項目 (ドキュメント) の読み取りは、REST API の `GET` メソッドを使うか、いずれかの SDK の `ReadDocumentAsync` を使って行うことができます。

```csharp
// Read document. Needs the partition key and the ID to be specified
DeviceReading document = await client.ReadDocumentAsync<DeviceReading>(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"), 
  new RequestOptions { PartitionKey = new PartitionKey("XMS-0001") });
```

### <a name="mongodb-api"></a>MongoDB API
MongoDB API を使用すると、お気に入りのツールやドライバー、SDK を通じてシャード コレクションを作成できます。 この例では、Mongo シェルを使用してコレクションを作成します。

Mongo シェルで次のコマンドを実行します。

```
db.runCommand( { shardCollection: "admin.people", key: { region: "hashed" } } )
```
    
結果:

```JSON
{
    "_t" : "ShardCollectionResponse",
    "ok" : 1,
    "collectionsharded" : "admin.people"
}
```

### <a name="table-api"></a>テーブル API

テーブル API では、アプリケーションの appSettings 構成でテーブルのスループットを指定します。

```xml
<configuration>
    <appSettings>
      <!--Table creation options -->
      <add key="TableThroughput" value="700"/>
    </appSettings>
</configuration>
```

そのうえで、テーブルは Azure Table Storage SDK を使って作成します。 パーティション キーは、`PartitionKey` 値として暗黙的に作成されます。 

```csharp
CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

CloudTable table = tableClient.GetTableReference("people");
table.CreateIfNotExists();
```

単一のエンティティを取得するには、次のスニペットを使用します。

```csharp
// Create a retrieve operation that takes a customer entity.
TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

// Execute the retrieve operation.
TableResult retrievedResult = table.Execute(retrieveOperation);
```
詳細については、[テーブル API を使用した開発](tutorial-develop-table-dotnet.md)に関するページを参照してください。

### <a name="graph-api"></a>Graph API

Graph API でコンテナーを作成するには、Azure Portal または Azure CLI を使用する必要があります。 また、Azure Cosmos DB はマルチモデルであるため、他のいずれかのモデルを使ってグラフ コンテナーを作成したりスケーリングしたりすることもできます。

Gremlin では、パーティション キーと ID を使って頂点または辺を読み取ることができます。 たとえばリージョン ("USA") をパーティション キー、"Seattle" を行キーとするグラフの場合、頂点は次の構文で見つけることができます。

```
g.V(['USA', 'Seattle'])
```

辺は、パーティション キーと行キーを使って参照することができます。

```
g.E(['USA', 'I5'])
```

詳細については、[Azure Cosmos DB での Gremlin のサポート](gremlin-support.md)に関するページを参照してください。


<a name="designing-for-partitioning"></a>
## <a name="design-for-partitioning"></a>パーティション分割のための設計
Azure Cosmos DB のスケーリングを効率よく行うには、コンテナーを作成する際に適切なパーティション キーを選ぶ必要があります。 パーティション キーを選ぶ際、主要な考慮事項が 2 つあります。

* **クエリとトランザクションの境界**。 パーティション キーを選ぶ際は、トランザクションを使用できるようにすると同時に、ソリューションのスケーラビリティ確保のためにエンティティを複数のパーティション キーに分散させることができるような、バランスの取れた条件にする必要があります。 極端な例を挙げると、すべての項目に同じパーティション キーを設定することは可能ですが、この場合はソリューションのスケーラビリティが制限される可能性があります。 その一方で、それぞれの項目に対して固有のパーティション キーを割り当てることもできます。 この場合、スケーラビリティは向上するものの、ストアド プロシージャとトリガーを介したクロス ドキュメント トランザクションを使用できなくなります。 理想的なパーティション キーは、効率的なクエリの使用を可能にし、かつソリューションのスケーラビリティを確保するために十分な基数を備えています。 
* **ストレージとパフォーマンスのボトルネックがない**。 ばらばらに存在する各種の値に対して書き込みを分散できるようなプロパティを選ぶことが重要です。 同じパーティション キーに対する要求は、単一のパーティションのスループットを超えることはできず、超えた場合は調整されます。 そのため、アプリケーション内の "ホット スポット" にならないパーティション キーを選択することが重要です。 1 つのパーティション キーのすべてのデータをパーティション内に格納する必要があるため、同じ値のデータが大量にあるパーティション キーは使用しないようにすることをお勧めします。 

実際に遭遇する可能性のあるいくつかのシナリオと、それぞれの適切なパーティション キーを見てみましょう。
* ユーザー プロファイルのバック エンドを実装する場合、ユーザー ID のパーティション キーを選択することをお勧めします。
* IoT データ (デバイス状態など) を格納する場合は、デバイス ID のパーティション キーを使用することをお勧めします。
* 時系列データのログ記録に Azure Cosmos DB を使用する場合は、ホスト名またはプロセス ID がパーティション キーとして適しています。
* マルチテナント アーキテクチャがある場合は、テナント ID のパーティション キーをお勧めします。

一部のユース ケース (IoT、ユーザー プロファイルなど) では、パーティション キーはお使いの ID (ドキュメント キー) と同じになる場合があります。 その他のケース (時系列データなど) では、パーティション キーは ID と異なる可能性があります。

### <a name="partitioning-and-loggingtime-series-data"></a>パーティション分割とログ記録/時系列データ
Azure Cosmos DB の一般的なユース ケースの 1 つは、ログ記録とテレメトリです。 膨大な量のデータの読み取り/書き込みが必要になる場合があるため、適切なパーティション キーを選択することが重要です。 選択するパーティション キーは、実行する予定のクエリの読み取り速度、書き込み速度、種類によって決まります。 適切なパーティション キーを選択する方法に関するヒントを次に示します。

* 長期間に少量の書き込みが累積し、タイム スタンプの範囲とその他のフィルターに基づくクエリを必要とするユース ケースでは、タイム スタンプのロールアップを使用します。 たとえば、日付をパーティション キーとして使用することをお勧めします。 この方法により、1 つのパーティションから 1 つの日付のすべてのデータを照会できます。 
* 書き込みが多いワークロードはよくあります。このような場合には、タイム スタンプに基づかないパーティション キーを使用してください。 この方法により、Azure Cosmos DB が複数のパーティションに対して均等に書き込みを分散することができます。 この場合は、ホスト名、プロセス ID、アクティビティ ID、または高基数の他のプロパティが適しています。 
* もう 1 つの方法は、複数コンテナーがあるハイブリッドです。この場合には、日または月ごとに 1 つのコンテナーがあり、パーティション キーは、ホスト名のように細分化されたプロパティです。 この方法の利点は、時間枠に基づいて異なるスループットを設定できることです。 たとえば、現在の月のコンテナーは、読み取りと書き込みに対応するため高スループットでプロビジョニングされますが、 過去の月の場合は、読み取りのみに対応するため、低いスループットでプロビジョニングされます。

### <a name="partitioning-and-multitenancy"></a>パーティション分割とマルチテナント方式
Azure Cosmos DB を使用したマルチテナント アプリケーションを実装する場合、一般に 2 つのパターンがあります。テナントごとに 1 つのパーティション キーを使用する方法と、テナントごとに 1 つのコンテナーを使用する方法です。 それぞれの長所と短所は次のとおりです。

* **テナントあたり 1 つのパーティション キー**。 このモデルでは、テナントは、単一のコンテナー内に併置されます。 ただし、1 つのテナント内の項目のクエリや挿入は、1 つのパーティションに対して実行されます。 また、1 つのテナント内のすべての項目全体にトランザクション ロジックを実装できます。 複数のテナントで 1 つのコンテナーが共有されるため、各テナント用に余分なヘッドルームをプロビジョニングするのではなく、単一のコンテナー内のテナントのリソースをプールして、ストレージとスループットのコストを削減できます。 欠点は、テナントごとにパフォーマンスを分離できないことです。 パフォーマンス/スループットの増加はコンテナー全体に適用されますが、テナントごとに適用されません。
* **テナントあたり 1 つのコンテナー**。 このモデルでは各テナントが独自のコンテナーを備え、テナントごとにパフォーマンスを予約できます。 Azure Cosmos DB の新しいプロビジョニング価格を使用することで、このモデルでは、少数のテナントを含んだマルチテナント アプリケーションでより高いコスト効果が得られます。

また、結合型/階層型の手法を使用して、小規模なテナントを併置したり、大規模なテナントを自身のコンテナーに移行したりすることもできます。

## <a name="next-steps"></a>次のステップ
この記事では、Azure Cosmos DB API を使ったパーティション分割の概念とベスト プラクティスについて概説しました。 

* [Azure Cosmos DB におけるスループットのプロビジョニング](request-units.md)について理解します。
* [Azure Cosmos DB の世界規模での分散](distribute-data-globally.md)について理解します。




