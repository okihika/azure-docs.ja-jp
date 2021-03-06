---
title: "コスト効率に優れた HD ベースの Standard Storage と Azure VM ディスク | Microsoft Docs"
description: "コスト効率に優れた Standard Storage および VM の非管理対象ディスクと管理ディスクについて説明します。"
services: storage
documentationcenter: 
author: yuemlu
manager: aungoo-msft
editor: tysonn
ms.assetid: e2a20625-6224-4187-8401-abadc8f1de91
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/13/2017
ms.author: yuemlu
ms.translationtype: HT
ms.sourcegitcommit: c3a2462b4ce4e1410a670624bcbcec26fd51b811
ms.openlocfilehash: b432426cb5cc5401fa2e8f7aaa6bc0955aff0931
ms.contentlocale: ja-jp
ms.lasthandoff: 09/25/2017

---
# <a name="cost-effective-standard-storage-and-unmanaged-and-managed-azure-vm-disks"></a>コスト効率に優れた Standard Storage および Azure VM の非管理対象ディスクと管理ディスク

Azure Standard Storage は、待機時間の影響を受けないワークロードを実行する VM 向けの信頼性の高い低コストのディスク サポートを提供します。 また、BLOB、テーブル、キュー、ファイルもサポートしています。 Standard Storage では、データはハード ディスク ドライブ (HDD) に格納されます。 VM を使用するときに、開発/テスト シナリオや重要度の低いワークロードには Standard Storage ディスクを使用し、ミッション クリティカルな運用アプリケーションには Premium Storage ディスクを使用できます。 Standard Storage はすべての Azure リージョンで利用できます。 

この記事では、VM ディスクでの Standard Storage の使用について説明します。 BLOB、テーブル、キュー、ファイルでのストレージの使用の詳細については、[Storage の概要](../storage-introduction.md)に関する記事をご覧ください。

## <a name="disk-types"></a>ディスクの種類

Azure VM の Standard ディスクは、次の 2 とおりの方法で作成できます。

**非管理対象ディスク**: これは、VM ディスクに対応する VHD ファイルの格納に使用するストレージ アカウントをユーザーが管理する本来の方法です。 VHD ファイルは、ストレージ アカウントにページ BLOB として格納されます。 非管理対象ディスクは、主に Premium Storage を使用する VM (DSv2 シリーズや GS シリーズなど) も含め、すべてのサイズの Azure VM に接続できます。 Azure VM には複数の Standard ディスクをアタッチできるので、VM あたり最大 256 TB のストレージを使用できます。

[**Azure Managed Disks**](../../virtual-machines/windows/managed-disks-overview.md): この機能は、VM ディスクに使用するストレージ アカウントを管理します。 必要なディスクの種類 (Premium または Standard) とサイズを指定すれば、ディスクの作成と管理は Azure によって行われます。 ストレージ アカウントのスケーラビリティの制限を超えないように、複数のストレージ アカウントにディスクを配置することを気に掛ける必要はありません。Azure がこれを管理します。

どちらの種類のディスクも利用できる場合でも、Managed Disks を使用してさまざまな機能を活用することをお勧めします。

Azure Standard Storage を使用するには、[無料試用版](https://azure.microsoft.com/pricing/free-trial/)のページをご覧ください。 

Managed Disks を使用する VM を作成する方法については、次のいずれかの記事をご覧ください。

* [Resource Manager と PowerShell を使用して VM を作成する](/azure/virtual-machines/windows/quick-create-powershell.md)
* [Azure CLI 2.0 を使用して Linux VM を作成する](../../virtual-machines/windows/quick-create-cli.md)

## <a name="standard-storage-features"></a>Standard Storage の機能 

ここでは、Standard Storage の機能をいくつか紹介します。 詳細については、[Azure Storage の概要](../storage-introduction.md)に関する記事をご覧ください。

**Standard Storage**: Azure Standard Storage では、Azure Disks、Azure BLOB、Azure Files、Azure Tables、Azure Queues をサポートしています。 Standard Storage サービスを使用するには、まず、[Azure ストレージ アカウントを作成](storage-create-storage-account.md#create-a-storage-account)します。

**Standard Storage ディスク**: Standard Storage ディスクは、Premium Storage で使用されるサイズ シリーズの VM (DSv2 シリーズや GS シリーズなど) も含め、すべての Azure VM に接続できます。 Standard Storage ディスクは、1 台の VM にのみ接続できます。 ただし、1 台の VM に Standard Storage ディスクを 1 つ以上接続できます。その VM サイズで定義されている最大ディスク数まで接続することが可能です。 Standard Storage のスケーラビリティとパフォーマンスのターゲットに関する次のセクションで、仕様についてさらに詳しく説明します。 

**Standard ページ BLOB**: Standard ページ BLOB は VM の永続ディスクを保持するために使用されます。他の種類の Azure BLOB と同様に、REST から Standard ページ BLOB に直接アクセスすることもできます。 [ページ BLOB](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs) は、ランダム読み取り/書き込み操作に最適化された 512 バイト ページのコレクションです。 

**ストレージ レプリケーション**: ほとんどのリージョンで、Standard Storage アカウント内のデータはローカルにレプリケートすることも、複数のデータセンター間で geo レプリケートすることもできます。 ローカル冗長ストレージ (LRS)、ゾーン冗長ストレージ (ZRS)、geo 冗長ストレージ (GRS)、読み取りアクセス geo 冗長ストレージ (RA-GRS) の 4 種類のレプリケーションを使用できます。 現在、Standard Storage の Managed Disks では、ローカル冗長ストレージ (LRS) のみをサポートしています。 詳細については、[Storage のレプリケーション](../storage-redundancy.md)に関する記事をご覧ください。

## <a name="scalability-and-performance-targets"></a>スケーラビリティとパフォーマンスのターゲット

このセクションでは、Standard Storage を使用する際に考慮する必要のあるスケーラビリティとパフォーマンスのターゲットについて説明します。

### <a name="account-limits--does-not-apply-to-managed-disks"></a>アカウントの制限 - 管理ディスクは適用外

| **リソース** | **既定の制限** |
|--------------|-------------------|
| ストレージ アカウントあたりの容量 (TB)  | 500 TB |
| ストレージ アカウントあたりの最大受信速度<sup>1</sup> (米国リージョン) | GRS/ZRS が有効な場合は 10 Gbps、LRS の場合は 20 Gbps |
| ストレージ アカウントあたりの最大送信速度<sup>1</sup> (米国リージョン) | RA-GRS/GRS/ZRS が有効な場合は 20 Gbps、LRS の場合は 30 Gbps |
| ストレージ アカウントあたりの最大受信速度<sup>1</sup> (ヨーロッパおよびアジア リージョン) | GRS/ZRS が有効な場合は 5 Gbps、LRS の場合は 10 Gbps |
| ストレージ アカウントあたりの最大送信速度<sup>1</sup> (ヨーロッパおよびアジア リージョン) | RA-GRS/GRS/ZRS が有効な場合は 10 Gbps、LRS の場合は 15 Gbps |
| ストレージ アカウントあたりの合計要求レート (オブジェクト サイズが 1 KB の場合) | 最大 20,000 の IOPS、エンティティ/秒、またはメッセージ/秒 |

<sup>1</sup>受信とは、ストレージ アカウントに送信されるすべてのデータ (要求) のことです。 送信とは、ストレージ アカウントから受信するすべてのデータ (応答) のことです。

詳細については、「 [Azure Storage のスケーラビリティおよびパフォーマンスのターゲット](../storage-scalability-targets.md)」をご覧ください。

アプリケーションのニーズが 1 つのストレージ アカウントのスケーラビリティ ターゲットを超えた場合は、複数のストレージ アカウントを使用するようにアプリケーションをビルドし、それらのストレージ アカウント間でデータをパーティション分割します。 また、Azure Managed Disks を使用することもできます。Azure Managed Disks では、データのパーティション分割と配置の管理は Azure が行います。

### <a name="standard-disks-limits"></a>Standard ディスクの制限

Premium ディスクとは異なり、Standard ディスクの IOPS (1 秒あたりの入出力操作) とスループット (帯域幅) はプロビジョニングされません。 Standard ディスクのパフォーマンスは、ディスクのサイズではなく、ディスクの接続先の VM サイズによって異なります。 次の表に示すパフォーマンスの上限に達する可能性があります。

**Standard ディスクの制限 (管理ディスクと非管理対象ディスク)**

| **VM のレベル**            | **Basic レベルの VM** | **Standard レベルの VM** |
|------------------------|-------------------|----------------------|
| 最大ディスク サイズ          | 4095 GB           | 4095 GB              |
| ディスクあたり最大 8 KB の IOPS | 最大 300         | 最大 500            |
| ディスクあたりの最大帯域幅 | 最大 60 MB/秒     | 最大 60 MB/秒        |

ワークロードで高パフォーマンス、低待機時間のディスク サポートが必要な場合は、Premium Storage を使用することを検討してください。 Premium Storage の利点の詳細については、[高パフォーマンスの Premium Storage と Azure VM ディスク](../storage-premium-storage.md)に関する記事をご覧ください。 

## <a name="snapshots-and-copy-blob"></a>スナップショットと Copy BLOB

Storage サービスでは、VHD ファイルはページ BLOB です。 ページ BLOB のスナップショットを作成し、別の場所 (別のストレージ アカウントなど) にコピーできます。

### <a name="unmanaged-disks"></a>非管理対象ディスク

Standard Storage でスナップショットを使用する場合と同様に、非管理対象 Standard ディスクの[増分スナップショット](../../virtual-machines/windows/incremental-snapshots.md)を作成できます。 ソース ディスクがローカル冗長ストレージ アカウント内にある場合は、スナップショットを作成し、それらのスナップショットを geo 冗長 Standard Storage アカウントにコピーすることをお勧めします。 詳細については、「 [Azure Storage 冗長オプション](../storage-redundancy.md)」をご覧ください。

ディスクが VM に接続されている場合、ディスクで特定の API 操作を実行できなくなります。 たとえば、ディスクが VM にアタッチされている間は、その BLOB に対して [Copy Blob](/rest/api/storageservices/Copy-Blob) 操作を実行できません。 代わりに、[Snapshot Blob](/rest/api/storageservices/Snapshot-Blob) REST API メソッドを使って BLOB のスナップショットを作成してから、スナップショットの [Copy Blob](/rest/api/storageservices/Copy-Blob) を実行してアタッチされたディスクをコピーします。 または、ディスクの接続を解除してから必要な操作を実行します。

スナップショットの geo 冗長コピーを維持するには、AzCopy または Copy BLOB を使用して、ローカル冗長ストレージ アカウントから geo 冗長 Standard Storage アカウントにスナップショットをコピーします。 詳しくは、「[AzCopy コマンド ライン ユーティリティを使用してデータを転送する](storage-use-azcopy.md)」および「[BLOB のコピー](/rest/api/storageservices/Copy-Blob)」をご覧ください。

Standard Storage アカウントのページ BLOB に対して REST 操作を実行する方法の詳細については、[Azure Storage Services REST API](/rest/api/storageservices/Azure-Storage-Services-REST-API-Reference) に関する記事をご覧ください。

### <a name="managed-disks"></a>管理ディスク

管理ディスクのスナップショットは管理ディスクの読み取り専用コピーであり、Standard 管理ディスクとして保存されます。 Managed Disks では増分スナップショットは現在サポートされていませんが、今後サポートされる予定です。

管理ディスクが VM に接続されている場合、ディスクで特定の API 操作を実行できなくなります。 たとえば、ディスクが VM に接続されているときには、Shared Access Signature (SAS) を生成してコピー操作を実行することはできません。 代わりに、ディスクのスナップショットを作成してから、スナップショットのコピーを実行します。 または、ディスクの接続を解除してから、Shared Access Signature (SAS) を生成してコピー操作を実行します。

## <a name="pricing-and-billing"></a>価格と課金

Standard Storage を使用するときには、課金に関する次の考慮事項が適用されます。

* Standard Storage の非管理対象ディスク/データ サイズ 
* Standard 管理ディスク
* Standard Storage のスナップショット
* 送信データ転送
* トランザクション

**非管理対象ストレージのデータとディスク サイズ**: 非管理対象ディスクとその他のデータ (BLOB、テーブル、キュー、ファイル) については、使用している領域分に対してのみ課金されます。 たとえば、ページ BLOB が 127 GB としてプロビジョニングされている VM があり、その VM で実際には 10 GB の領域しか使用していない場合、10 GB の領域分に対して課金されます。 最大 8,191 GB の Standard Storage および最大 4,095 GB の Standard 非管理対象ディスクがサポートされます。 

**管理ディスク**: 管理ディスクは、プロビジョニング済みサイズに対して課金されます。 ディスクが 10 GB ディスクとしてプロビジョニングされており、5 GB しか使用していなくても、10 GB のプロビジョニング サイズに対して課金されます。

**スナップショット**: Standard ディスクのスナップショットについては、スナップショットで使用された追加の容量に対して課金されます。 スナップショットの詳細については、「 [BLOB のスナップショットの作成](/rest/api/storageservices/Creating-a-Snapshot-of-a-Blob)」をご覧ください。

**送信データ転送**: [送信データ転送](https://azure.microsoft.com/pricing/details/data-transfers/) (Azure データ センターから送信されるデータ) では、帯域幅の使用量に対して課金されます。

**トランザクション**: Standard Storage では、100,000 トランザクションあたり 0.0036 ドルが課金されます。 トランザクションには、ストレージに対する読み取り操作と書き込み操作の両方が含まれます。

Standard Storage、Virtual Machines、Managed Disks の価格の詳細については、次のページをご覧ください。

* [Azure Storage の料金](https://azure.microsoft.com/pricing/details/storage/)
* [Virtual Machines の料金](https://azure.microsoft.com/pricing/details/virtual-machines/)
* [Managed Disks の価格](https://azure.microsoft.com/pricing/details/managed-disks)

## <a name="azure-backup-service-support"></a>Azure Backup サービスのサポート 

非管理対象ディスクを使用する仮想マシンは、Azure Backup を使用してバックアップできます。 [詳細についてはこちらをご覧ください](../../backup/backup-azure-vms-first-look-arm.md)。

また、Managed Disks で Azure Backup サービスを使用して、時間ベースのバックアップ、VM の簡易復元、バックアップ リテンション期間ポリシーを適用したバックアップ ジョブを作成することもできます。 詳細については、[Managed Disks を使用する VM での Azure Backup サービスの使用](../../backup/backup-introduction-to-azure-backup.md#using-managed-disk-vms-with-azure-backup)に関するセクションをご覧ください。

## <a name="next-steps"></a>次のステップ

* [Azure ストレージの概要](../storage-introduction.md)

* [ストレージ アカウントの作成](../storage-create-storage-account.md)

* [Managed Disks の概要](../../virtual-machines/windows/managed-disks-overview.md)

* [Resource Manager と PowerShell を使用して VM を作成する](/azure/virtual-machines/windows/quick-create-powershell.md)

* [Azure CLI 2.0 を使用して Linux VM を作成する](../../virtual-machines/windows/quick-create-cli.md)

