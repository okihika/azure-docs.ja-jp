---
title: "Azure CLI のサンプル スクリプト - NLB がインストールされた Windows Server 2016 の作成 | Microsoft Docs"
description: "Azure CLI のサンプル スクリプト - NLB がインストールされた Windows Server 2016 の作成"
services: virtual-machines-Windows
documentationcenter: virtual-machines
author: rickstercdn
manager: timlt
editor: tysonn
tags: 
ms.assetid: 
ms.service: virtual-machines-Windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
ms.translationtype: HT
ms.sourcegitcommit: 190ca4b228434a7d1b30348011c39a979c22edbd
ms.openlocfilehash: 9662e9a2ea128b609b76edfc4ec7b57016909761
ms.contentlocale: ja-jp
ms.lasthandoff: 09/09/2017

---

# <a name="load-balance-traffic-between-highly-available-virtual-machines"></a>高可用性仮想マシン間でトラフィックの負荷を分散する

このサンプル スクリプトでは、高可用性と負荷分散が設定された構成で、構成済みの複数の Ubuntu 仮想マシンを実行するために必要なものをすべて作成します。 このスクリプトを実行すると、Azure 可用性セットに接続され、Azure Load Balancer 経由でアクセス可能な仮想マシンが 3 つ作成されます。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>サンプル スクリプト

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-nlb/create-windows-vm-nlb.sh "VM の簡易作成")]

## <a name="clean-up-deployment"></a>デプロイのクリーンアップ 

次のコマンドを実行して、リソース グループ、VM、すべての関連リソースを削除します。

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>スクリプトの説明

このスクリプトでは、次のコマンドを使用して、リソース グループ、仮想マシン、可用性セット、ロード バランサー、およびすべての関連リソースを作成します。 表内の各コマンドは、それぞれのドキュメントにリンクされています。

| コマンド | メモ |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) | すべてのリソースを格納するリソース グループを作成します。 |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet#az_network_vnet_create) | Azure 仮想ネットワークとサブネットを作成します。 |
| [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip#az_network_public_ip_create) | 静的 IP アドレスおよび関連付けられる DNS 名を指定してパブリック IP アドレスを作成します。 |
| [az network lb create](https://docs.microsoft.com/cli/azure/network/lb#az_network_lb_create) | Azure ネットワーク ロード バランサー (NLB) を作成します。 |
| [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe#az_network_lb_probe_create) | NLB プローブを作成します。 NLB プローブは、NLB セット内の各 VM の監視に使用します。 アクセス不能になった VM には、トラフィックがルーティングされなくなります。 |
| [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule#az_network_lb_rule_create) | NLB 規則を作成します。 このサンプルでは、ポート 80 に対する規則を作成します。 HTTP トラフィックが NLB で受信されると、NLB セット内のいずれかの VM のポート 80 にルーティングされます。 |
| [az network lb inbound-nat-rule create](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-rule#az_network_lb_inbound_nat_rule_create) | NLB ネットワーク アドレス変換 (NAT) 規則を作成します。  NAT 規則により、NLB のポートを VM のポートにマッピングされます。 このサンプルでは、NLB セット内の各 VM への SSH トラフィックに対する NAT 規則を作成します。  |
| [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg#az_network_nsg_create) | インターネットと仮想マシン間のセキュリティ境界となるネットワーク セキュリティ グループ (NSG) を作成します。 |
| [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule#az_network_nsg_rule_create) | 受信トラフィックを許可する NSG 規則を作成します。 このサンプルでは、SSH トラフィック用にポート 22 を開きます。 |
| [az network nic create](https://docs.microsoft.com/cli/azure/network/nic#az_network_nic_create) | 仮想ネットワーク カードを作成し、作成したカードを仮想ネットワーク、サブネット、NSG に接続します。 |
| [az vm availability-set create](https://docs.microsoft.com/cli/azure/network/lb/rule#az_network_lb_rule_create) | 可用性セットを作成します。 可用性セットでは、障害が発生した場合でもセット全体に影響が及ばないように物理リソース全体に仮想マシンを分散させることで、アプリケーションの稼働時間を確保します。 |
| [az vm create](https://docs.microsoft.com/cli/azure/vm/availability-set#az_vm_availability_set_create) | 仮想マシンを作成し、作成したマシンをネットワーク カード、仮想ネットワーク、サブネット、およびNSG に接続します。 このコマンドでは、使用する仮想マシン イメージと管理者の資格情報も指定します。  |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az_vm_extension_set) | 入れ子になったリソースすべてを含むリソース グループを削除します。 |

## <a name="next-steps"></a>次のステップ

Azure CLI の詳細については、[Azure CLI のドキュメント](https://docs.microsoft.com/cli/azure/overview)のページをご覧ください。

その他の仮想マシン用の CLI サンプル スクリプトは、[Azure Windows VM のドキュメント](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)のページにあります。

