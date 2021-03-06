---
title: "Azure Site Recovery を使用したセカンダリ サイトへの Hyper-V レプリケーションのレプリケーション ポリシーをセットアップする | Microsoft Docs"
description: "Azure Site Recovery を使用したセカンダリ VMM サイトへの Hyper-V VM レプリケーションのポリシーをセットアップする方法について説明します。"
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: carmonm
editor: 
ms.assetid: 5d9b79cf-89f2-4af9-ac8e-3a32ad8c6c4d
ms.service: site-recovery
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/30/2017
ms.author: raynew
ms.translationtype: HT
ms.sourcegitcommit: fff84ee45818e4699df380e1536f71b2a4003c71
ms.openlocfilehash: a3ecc555e049914d7bffdddce7b0522f09362d24
ms.contentlocale: ja-jp
ms.lasthandoff: 08/01/2017

---
# <a name="step-8-set-up-a-replication-policy"></a>手順 8: レプリケーション ポリシーを設定する

[ネットワーク マッピング](vmm-to-vmm-walkthrough-network-mapping.md)を構成したら、この記事を参照し、[Azure Site Recovery](site-recovery-overview.md) を使用したセカンダリ サイトへの Hyper-V 仮想マシン (VM) のレプリケーションのレプリケーション ポリシーをセットアップします。

この記事に関するコメントは、この記事の末尾、または [Azure Recovery Services フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)で投稿してください。


## <a name="before-you-start"></a>開始する前に

- レプリケーション ポリシーを作成するときに、そのポリシーを使用するすべてのホストで同じオペレーティング システムが必要です。 VMM クラウドは異なるバージョンの Windows Server を実行する Hyper-V ホストを含むことができますが、その場合は、複数のレプリケーション ポリシーが必要です。
- 初期レプリケーションをオフラインで行うことができます。

## <a name="configure-replication-settings"></a>レプリケーションの設定を構成する

1. 新しいレプリケーション ポリシーを作成するには、**[インフラストラクチャの準備]** > **[レプリケーションの設定]** > **[+ 作成と関連付け]** の順にクリックします。

    ![ネットワーク](./media/vmm-to-vmm-walkthrough-replication/gs-replication.png)
2. **[ポリシーの作成と関連付け]**で、ポリシー名を指定します。 ソースとターゲットの種類は **Hyper-V**にする必要があります。
3. **[Hyper-V ホストのバージョン]** で、ホストで実行されているオペレーティング システムを選択します。
4. **[認証の種類]** と **[認証ポート]** で、プライマリおよび復旧用の Hyper-V ホスト サーバー間でトラフィックを認証する方法を指定します。 動作している Kerberos 環境が既にある場合を除き、**[証明書]** を選択してください。 Azure Site Recovery は、HTTPS 認証に使用する証明書を自動的に構成します 手動では何も行う必要はありません。 既定では、Hyper-V ホスト サーバーの Windows ファイアウォールで、ポート 8083 と 8084 (認証用) が開きます。 **Kerberos** を選択した場合は、ホスト サーバーの相互認証に Kerberos チケットが使用されます。 この設定は、Windows Server 2012 R2 で実行されている Hyper-V ホスト サーバーだけに関連することに注意してください。
5. **[コピーの頻度]**で、初期レプリケーションの後、差分データをレプリケートする頻度 (30 秒ごと、5 分ごと、または 15 分ごと) を指定します。
6. **[復旧ポイントの保持期間]**で、各復旧ポイントのリテンション期間の長さを時間単位で指定します。 保護されたマシンはこの期間内のどのポイントにも復旧できます。
7. **[アプリ整合性スナップショットの頻度]** で、アプリケーション整合性スナップショットを含む復旧ポイントの作成頻度 (1 - 12 時間) を指定します。 Hyper-V では 2 種類のバックアップを使用します。1 つは標準バックアップで、仮想マシン全体の増分バックアップを実行します。もう 1 つは、アプリケーション整合性スナップショットで、仮想マシン内部のアプリケーション データの特定の時点のスナップショットを作成します。 アプリケーション整合性スナップショットでは、ボリューム シャドウ コピー サービス (VSS) を使用して、スナップショットを作成するときにアプリケーションを一貫性のある状態に保ちます。 アプリケーション整合性スナップショットを有効にすると、ソースの仮想マシンで実行されるアプリケーションのパフォーマンスに影響があります。 設定する値は、追加で構成する復旧ポイントの数より少ない数にしてください。
8. **[データ転送の圧縮]**で、転送されるレプリケート済みデータを圧縮するかどうかを指定します。
9. ソース VM の保護を無効にする場合はレプリカ仮想マシンを削除するように指定するために、**[レプリカ VM の削除]** をオンにします。 この設定を有効にすると、ソース VM の保護を無効にしたときに、Site Recovery コンソールからのソース VM の削除、VMM コンソールからの VMM の Site Recovery 設定の削除、レプリカの削除が行われます。
10. ネットワーク経由でレプリケートする場合は、**[初期レプリケーションの方法]** で、初期レプリケーションを開始するかスケジュールを設定するかを指定します。 ネットワーク帯域幅を節約するために、トラフィックの多い時間帯を避けてスケジュールを設定することをお勧めします。 次に、 **[OK]**をクリックします

     ![Replication policy](./media/vmm-to-vmm-walkthrough-replication/gs-replication2.png)
11. 新しいポリシーを作成すると、自動的に VMM クラウドに関連付けられます。 **[レプリケーション ポリシー]** で **[OK]** をクリックします。 追加の VMM クラウド (およびその内部にある VM) をこのレプリケーション ポリシーに関連付けるには、**[レプリケーション]** > ポリシー名 > **[Associate VMM Cloud (VMM クラウドを関連付ける)]** の順に選択します。

     ![Replication policy](./media/vmm-to-vmm-walkthrough-replication/policy-associate.png)



## <a name="prepare-for-offline-initial-replication"></a>オフラインでの初期レプリケーションの準備

初期データ コピーのオフライン レプリケーションを行うことができます。 この準備として、次の手順を実行します。

* ソース サーバーで、データのエクスポートを実施するパスの場所を指定します。 そのエクスポート パスに対する NTFS のフル コントロール アクセス許可と共有アクセス許可を VMM サービスに付与します。 ターゲット サーバーで、データのインポートを実施するパスの場所を指定します。 このインポート パスに対して同じアクセス許可を割り当てます。
* インポート パスまたはエクスポート パスが共有されている場合は、その共有パスの配置先であるリモート コンピューター上の VMM サービス アカウントに対して、Administrator、Power User、Print Operator、または Server Operator グループのメンバーシップを割り当てます。
* いずれかの実行アカウントを使用してホストを追加している場合は、インポート パスおよびエクスポート パスに対する読み取りと書き込みのアクセス許可を VMM の実行アカウントに割り当てます。
* Hyper-V ではループバックの構成がサポートされていないため、インポート用の共有とエクスポート用の共有は、Hyper-V ホスト サーバーとして使用するどのコンピューターにも配置しないでください。
* Active Directory で、保護対象の仮想マシンを含む各 Hyper-V ホスト サーバーに対して、次のように制約付き委任を有効にして構成し、インポート パスとエクスポート パスの配置先であるリモート コンピューターを信頼します。
  1. ドメイン コントローラーで、 **[Active Directory ユーザーとコンピューター]**を開きます。
  2. コンソール ツリーで **[ドメイン名]** > **[コンピューター]** の順にクリックします。
  3. Hyper-V ホスト サーバー名を右クリックして、**[プロパティ]** をクリックします。
  4. **[委任]** タブで、**[指定されたサービスへの委任でのみこのコンピューターを信頼する]** をクリックします。
  5. **[任意の認証プロトコルを使う]**をクリックします。
  6. **[追加]** > **[ユーザーとコンピューター]**に投稿してください。
  7. エクスポート パスがホストされるコンピューターの名前を入力して **[OK]** をクリックします。 利用可能なサービスの一覧で、Ctrl キーを押したまま **[cifs]** > **[OK]** の順にクリックします。 インポート パスをホストするコンピューターの名前に対して、同じ手順を繰り返します。 必要に応じて、追加の Hyper-V ホスト サーバーに対して同じ手順を繰り返します。



## <a name="next-steps"></a>次のステップ

[手順 9: レプリケーションを有効にする](vmm-to-vmm-walkthrough-enable-replication.md)方法に関するページに進みます。

