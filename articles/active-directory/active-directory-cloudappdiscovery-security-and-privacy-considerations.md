---
title: "Cloud App Discovery のセキュリティとプライバシーの考慮事項 | Microsoft Docs"
description: "このトピックでは、Cloud App Discovery に関連するセキュリティとプライバシーの考慮事項について説明します。"
services: active-directory
documentationcenter: 
author: MarkusVi
manager: femila
ms.assetid: 2fce5c82-d3de-4097-808f-40214768df9e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/25/2017
ms.author: markvi
ms.reviewer: nigu
ms.translationtype: HT
ms.sourcegitcommit: 44e9d992de3126bf989e69e39c343de50d592792
ms.openlocfilehash: cec3d2cb02dd34dd5ac631e572936cfd7c8de033
ms.contentlocale: ja-jp
ms.lasthandoff: 09/25/2017

---
# <a name="cloud-app-discovery-security-and-privacy-considerations"></a>Cloud App Discovery のセキュリティとプライバシーの考慮事項
このトピックでは、Azure Active Directory Cloud App Discovery 内でデータがどのように収集、処理、およびセキュリティ保護されるかについて説明します。 Microsoft では、お客様のプライバシーの保護とお客様のデータのセキュリティ保護に努めています。 Microsoft は、サービスを運用するためのセキュリティ保護されたソフトウェア開発ライフサイクル プラクティスに従います。 Microsoft においてデータの保護は最優先事項になります。

> [!TIP] 
> [Microsoft Cloud App Security との統合](https://portal.cloudappsecurity.com)によって拡張される、Azure Active Directory (Azure AD) の新しいエージェントレスの Cloud App Discovery を確認してください。 

## <a name="overview"></a>概要
Cloud App Discovery は Azure AD の機能であり、Microsoft Azure でホストされます。  
Cloud App Discovery Endpoint Agent は、IT で管理されているコンピューターからアプリケーション検出データを収集するために使用します。 収集したデータは、暗号化されたチャネル経由でセキュリティで保護されて Azure AD Cloud App Discovery サービスに送信されます。 その後、組織の Cloud App Discovery データは、Azure Portal に表示されます。 

![Cloud App Discovery のしくみ](./media/active-directory-cloudappdiscovery-security-and-privacy-considerations/cad01.png) 

以降のセクションは、組織から Cloud App Discovery サービスへ、さらに最終的には Cloud App Discovery ポータルへの情報のセキュリティ保護されたフローに従います。

## <a name="collecting-data-from-your-organization"></a>組織からのデータの収集
Azure Active Directory の Cloud App Discovery 機能を使用して組織内の従業員が使用するアプリケーションに関する洞察を得るには、まず Azure AD Cloud App Discovery Endpoint Agent を組織のコンピューターにデプロイする必要があります。

Azure Active Directory テナントの管理者 (またはその代理人) は、Azure Portal から、このエージェントのインストール パッケージをダウンロードできます。 エージェントは、手動でインストールすることも、SCCM またはグループ ポリシーを使用して組織内の複数のコンピューターにインストールすることもできます。

デプロイ オプションの詳細については、 [Cloud App Discovery Group Policy Deployment Guide (Cloud App Discovery のグループ ポリシーのデプロイ ガイド)](http://social.technet.microsoft.com/wiki/contents/articles/30965.cloud-app-discovery-group-policy-deployment-guide.aspx)をご覧ください。


### <a name="data-collected-by-the-agent"></a>エージェントによって収集されたデータ
Web アプリケーションへの接続が行われると、以下の一覧に概要が説明されている情報がエージェントによって収集されます。 情報は、管理者が検出用に構成したアプリケーションについてのみ収集されます。 エージェントが監視するクラウド アプリの一覧は、Microsoft [Azure Portal](https://portal.azure.com/) 内の Azure AD の Cloud App Discovery で **[設定]**->**[データ収集]**->**[アプリ コレクション]** ボックスの一覧を使用して編集できます。 

**情報カテゴリ**: ユーザー情報  
**説明**: ターゲットの Web アプリケーションに対する要求を作成したプロセスの Windows ユーザー名 (例: DOMAIN\username) とそのユーザーの Windows セキュリティ識別子 (SID)。

**情報カテゴリ**: プロセス情報  
**説明**: ターゲットの Web アプリケーションに対する要求を作成したプロセスの名前 (例: iexplore.exe)

**情報カテゴリ**: コンピューター情報  
**説明**: エージェントがインストールされているマシンの NetBIOS 名。

**情報カテゴリ**: アプリのトラフィック情報  
**説明**: 次の接続情報:

* ソース (ローカル コンピューター) と宛先の IP アドレスとポート番号
* 要求送信に使用する組織のパブリック IP アドレス
* 要求時刻
* 送受信されたトラフィックの量
* IP バージョン (4 または 6)
* TLS 接続のみ: Server Name Indication 拡張機能またはサーバー証明書のいずれかのターゲット ホスト名

HTTP 情報:

* メソッド (GET、POST など)
* プロトコル (HTTP/1.1 など)
* ユーザー エージェント文字列
* ホスト名
* ターゲット URI (クエリ文字列を除く)
* コンテンツ タイプ情報
* 参照元 URL 情報 (クエリ文字列を除く)

> [!NOTE]
> 上記の HTTP 情報は、暗号化されていないすべての接続について収集されます。
> TLS 接続では、ポータルで「詳細な検査」設定をオンにしているときにのみ、この情報がキャプチャされます。 設定は、既定では「オン」です。
> 詳細については、以下の説明と [Getting Started With Cloud App Discovery (Cloud App Discovery の概要)](http://social.technet.microsoft.com/wiki/contents/articles/30962.getting-started-with-cloud-app-discovery.aspx)
> 
> 

エージェントがネットワーク アクティビティに関して収集するデータに加えて、次のものに関する匿名データも収集します。
* ソフトウェアおよびハードウェア構成
* エラー レポート
* エージェントがどのように使用されているかに関するデータ


### <a name="how-the-agent-works"></a>エージェントの動作
エージェントのインストールには、次の 2 つのコンポーネントが含まれています。

* ユーザーモード コンポーネント
* カーネルモード ドライバー コンポーネント (Windows フィルタリング プラットフォーム ドライバー)

エージェントは、最初にインストールされたときに、コンピューター上にコンピューター固有の信頼された証明書を格納し、その後、これを使用して Cloud App Discovery サービスとセキュリティで保護された接続を確立します。 エージェントは、このセキュリティで保護された接続経由で、定期的に Cloud App Discovery サービスからポリシーの構成を取得します。 ポリシーには、監視するクラウド アプリケーション、自動更新の有効化などについての情報が含まれています。

コンピューター上で Chrome や Internet Explorer から Web トラフィックが送受信されると、Cloud App Discovery エージェントがトラフィックを分析し、関連するメタデータ を抽出します (前のセクション「 **エージェントによって収集されたデータ** 」を参照) 。  
エージェントは、1 分ごとに暗号化されたチャネル経由で、収集したメタデータを Cloud App Discovery サービスにアップロードします。

ドライバー コンポーネントでは、暗号化されたトラフィックをインターセプトし、暗号化されたストリームに挿入します。 詳細については、後のセクション「 **暗号化された接続からのデータのインターセプト (詳細な検査)** 」を参照してください。

### <a name="respecting-user-privacy"></a>ユーザーのプライバシーの尊重
私たちの目的は、管理者にアプリケーション使用状況の詳細な把握とユーザーのプライバシーとの間のバランスを組織に合わせて設定できるツールを提供することです。 そのために、ポータルの設定ページで次の項目を設定できるようにしました。

* **データ収集**: 管理者は検出データを取得するアプリケーションまたはアプリケーション カテゴリを指定できます。
* **詳細な検査**: 管理者はエージェントが SSL や TLS 接続の HTTP トラフィックを収集するかどうかを指定できます ("**詳細な検査**" といいます)。 詳細については、次のセクションで説明します。
* **同意オプション**: 管理者は Cloud App Discovery ポータルを使用して、エージェントによるデータの収集についてユーザーに通知するかどうか、およびエージェントがユーザー データの収集を開始する前にユーザーの同意を必要とするかどうかを選択できます。

Cloud App Discovery エンドポイントのエージェントが収集するのは、前のセクション「 **エージェントによって収集されたデータ** 」で説明された情報のみです。

### <a name="intercepting-data-from-encrypted-connections-deep-inspection"></a>暗号化された接続からのデータのインターセプト (詳細な検査)
既に説明したように、管理者は、エージェントを構成して、暗号化された接続のデータを監視することができます ("詳細な検査")。 TLS ([トランスポート層セキュリティ](https://msdn.microsoft.com/library/windows/desktop/aa380516%28v=vs.85%29.aspx)) は、インターネット上で現在最も一般的に使用されているプロトコルの 1 つです。 TLS で通信を暗号化することによって、クライアントが Web サーバーとのセキュリティで保護されたプライベート通信チャネルを確立できます。TLS は、認証の資格情報を渡すために不可欠な保護機能を提供するほか、機密情報の漏えいを防ぎます。

TLS が提供するエンド ツー エンドのセキュリティで保護された暗号化チャネルは重要なセキュリティとプライバシーの保護を実現しますが、このプロトコルは、悪意のある目的や不正な目的に利用されることがあります。 そのため、TLS は "ユニバーサル ファイアウォール バイパス プロトコル" と呼ばれることもあります。 問題の原因は、アプリケーション層のデータが SSL で暗号化されているためにほとんどのファイアウォールで TLS の通信を検査できないことにあります。 この点を突いて攻撃を行う攻撃者は、最もインテリジェントなアプリケーション層ファイアウォールでさえも TLS には盲目的であり、ホスト間で TLS の通信を単に中継することしかできないことを確信して、TLS を利用して悪意のあるペイロードをユーザーに送りつけます。 エンド ユーザーは、TLS を利用して、企業のファイアウォールとプロキシ サーバーによって適用されるアクセス制御をバイパスすることがあります。具体的には、TLS を利用してパブリック プロキシに接続し、TLS 以外のプロトコルをトンネリングしてファイアウォールを通過させるという方法ですが、これは通常であればポリシーによってブロックされる操作です。

詳細な検査では、Cloud App Discovery エージェントが信頼された仲介者として機能します。 保護された HTTPS リソースへのアクセスをクライアントが要求すると、Endpoint Agent ドライバーが接続をインターセプトし、移行先サーバーへの新しい接続を確立することで、クライアントに代わって SSL 証明書を取得します。 そのエージェントは証明書が信頼できることを確認し (証明書が失効していないことを確認し、その他の証明書チェックを実行する)、問題がなければ Endpoint Agent がサーバー証明書からの情報をコピーし、その情報を使用して、インターセプション証明書と呼ばれる独自のサーバー証明書を作成します。 インターセプション証明書は、エンドポイント エージェントがルート証明書を使用してすぐに署名できます。ルート証明書は、Windows の信頼された証明書ストアにインストールされています。 この自己署名ルート証明書はエクスポートできないとマークされ、管理者の ACL となります。 作成元のコンピューター以外の場所では使用されません。 エンドユーザーのクライアント アプリケーションがインターセプション証明書を受信すると、その証明書は信頼されます。それによって、ルート証明書までの証明書チェーンをすべて適切に検証できるからです。 このプロセスのほとんどはエンドユーザーには透過的ですが、以下に示す注意事項がいくつかあります。

詳細の検査を有効にすることにより、Cloud App Discovery Endpoint Agent は、TLS で暗号化された通信の暗号化を解除し、検査できます。これにより、サービスは、ノイズを軽減し、暗号化されたクラウド アプリの使用状況に関する情報を提供できます。

#### <a name="a-word-of-caution"></a>注意事項
詳細な検査を有効にする前に、法務部門と人事部門にその意図を伝え、同意を得ることを強くお勧めします。 エンド ユーザー個人の暗号化された通信を検査することは難しい問題です。 詳細の検査の実稼働環境へのロールアウトの前に、暗号化された通信が検査の対象となることが明示されるように、会社のセキュリティ ポリシーと利用ポリシーが更新されていることを確認してください。 ユーザーへの通知と機密性が高いと見なされるサイトの除外 (例: 銀行、医療サイト) も必要になる場合があります (これらを監視するように Cloud App Discovery を構成する場合)。 すでに述べたように、管理者は Cloud App Discovery ポータルを使用して、エージェントによるデータの収集についてユーザーに通知するかどうか、およびエージェントがユーザー データの収集を開始する前にユーザーの同意を必要とするかどうかを選択できます。

### <a name="known-issues-and-drawbacks"></a>既知の問題と欠点
TLS によるインターセプションが、エンド ユーザー エクスペリエンスに次のような影響を与える場合があります。

* Extended Validation (EV) 証明書が使用された場合、Web ブラウザーのアドレス バーが緑色で表示されます。これは、信頼済みの Web サイトを訪問していることを視覚的に示すためのものです。 TLS の検査では、クライアントに対して発行される証明書で EV を複製することはできないため、EV 証明書を使用する Web サイトが正常に動作する場合でも、アドレス バーは緑色で表示されません。  
* 公開キーのピン留め (証明書のピン留めとも呼ばれます) は、man-in-the-middle 攻撃や偽の証明書機関からユーザーを保護することを意図しています。 ピン留めされたサイトのルート証明書が既知の適切な CA のいずれかと一致しない場合、ブラウザーは接続を拒否してエラーを返します。 TLS のインターセプトは実際には仲介者 (man-in-the-middle) であるため、これらの接続は失敗します。
* ユーザーが、サイト情報を検査するためにブラウザー アドレス バーにあるロック アイコンをクリックした場合、Web サイトの証明書の署名に使用される証明機関で終了するチェーンは表示されませんが、代わりに、Windows の信頼された証明書ストアで終了する証明書チェーンが表示されます。

このような問題の発生回数を減らすには、拡張された検証または公開キーのピン留めを使用することがわかっているクラウド サービスおよびクライアント アプリケーションを追跡し、影響を受ける接続をインターセプトしなしように Endpoint Agent に指示します。 このような場合でも、これらのクラウド アプリケーションの使用に関してや転送されたデータ量のレポートを受信できます。しかし詳細な検査は実行されないため、アプリの使用方法に関する詳細情報は得られません。

## <a name="sending-data-to-cloud-app-discovery"></a>Cloud App Discovery へのデータの送信
メタデータは、エージェントによって収集された後、最大 1 分間、またはキャッシュ データのサイズが 5 MB に達するまで、コンピューターでキャッシュされます。 その後、圧縮され、セキュリティで保護された接続経由で Cloud App Discovery サービスに送信されます。

エージェントが何らかの理由で Cloud App Discovery サービスと通信できない場合は、収集されたメタデータはローカル ファイル キャッシュに格納されます。このキャッシュには、コンピューター上の特権を持つユーザー (Administrators グループなど) のみがアクセスできます。  
エージェントは、キャッシュしたメタデータが Cloud App Discovery サービスによって正常に受信されるまで、自動的に再送信を試みます。

## <a name="receiving-the-data-at-the-service-end"></a>サービス エンドでのデータの受信
エージェントは、Cloud App Discovery サービスに対して、上述のコンピューター固有のクライアント認証証明書を使用して、暗号化されたチャネル経由で転送されたデータを認証します。  
Cloud App Discovery サービスの分析パイプラインは、分析パイプラインのすべての段階でメタデータを論理的に分割することで、メタデータを顧客ごとに個別に処理します。
分析されたメタデータは、ポータルのさまざまなレポートで活用されます。

未処理のメタデータと分析されたメタデータは、最大 180 日間保存されます。 さらに、お客様は、自分で選択した Azure BLOB ストレージ アカウントに、分析されたメタデータをキャプチャすることができます。
これは、メタデータのオフライン分析や、データの保存期間の延長として役立ちます。

## <a name="accessing-the-data-using-the-azure-portal"></a>Azure Portal を使用するデータ アクセス
収集されたメタデータのセキュリティ保護を確保するために、既定では、テナントのグローバル管理者のみが Azure Portal で Cloud App Discovery 機能にアクセスできます。  
ただし管理者は、他のユーザーやグループにこのアクセスを委任できます。

> [!NOTE]
> 詳細については、 [Getting Started With Cloud App Discovery (Cloud App Discovery の概要)](http://social.technet.microsoft.com/wiki/contents/articles/30962.getting-started-with-cloud-app-discovery.aspx)
> 
> 


ポータルで、データにアクセスするすべてのユーザーは、Azure AD Premium ライセンスを持っている必要があります。

## <a name="additional-resources"></a>その他のリソース
* [自分の組織内で使用される承認されていないクラウド アプリを検出する方法](active-directory-cloudappdiscovery-whatis.md)
* [Article Index for Application Management in Azure Active Directory](active-directory-apps-index.md)


