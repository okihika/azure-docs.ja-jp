---
title: "Azure Cosmos DB に接続する Azure 関数の作成 | Microsoft Docs"
description: "Azure CLI スクリプトのサンプル - Azure Cosmos DB に接続する Azure 関数の作成"
services: functions
documentationcenter: functions
author: rachelappel
manager: cfowler
editor: 
tags: functions
ms.assetid: 
ms.service: functions
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: 
ms.date: 04/20/2017
ms.author: rachelap
ms.custom: mvc
ms.translationtype: HT
ms.sourcegitcommit: 190ca4b228434a7d1b30348011c39a979c22edbd
ms.openlocfilehash: c2c3530df62a1f291be51739a7918f7b8ab08487
ms.contentlocale: ja-jp
ms.lasthandoff: 09/09/2017

---
# <a name="create-an-azure-function-that-connects-to-an-azure-cosmos-db"></a>Azure Cosmos DB に接続する Azure 関数の作成

このサンプル スクリプトでは、Azure Function App を作成し、Azure Cosmos DB データベースに接続します。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

CLI をローカルにインストールして使用する場合、このトピックでは、Azure CLI バージョン 2.0 以降を実行していることが要件です。 バージョンを確認するには、`az --version` を実行します。 インストールまたはアップグレードする必要がある場合は、「[Azure CLI 2.0 のインストール]( /cli/azure/install-azure-cli)」を参照してください。 

## <a name="sample-script"></a>サンプル スクリプト

このサンプルでは、Azure Function App を作成し、Cosmos DB エンドポイントとアクセス キーをアプリ設定に追加します。

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-connect-to-cosmos-db/create-function-app-connect-to-cosmos-db.sh "Azure Cosmos DB に接続する Azure 関数の作成")]

## <a name="clean-up-deployment"></a>デプロイのクリーンアップ

サンプル スクリプトの実行後、次のコマンドを使用すると、リソース グループとすべての関連リソースを削除できます。

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>スクリプトの説明

このスクリプトでは、次のコマンドを使用します。 表内の各コマンドは、それぞれのドキュメントにリンクされています。

| コマンド | メモ |
|---|---|
| [az login](https://docs.microsoft.com/cli/azure/#login) | Azure にログインします。 |
| [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) | 任意の場所にリソース グループを作成します |
| [az storage account create](https://docs.microsoft.com/cli/azure/storage/account) | ストレージ アカウントの作成 |
| [az functionapp create](https://docs.microsoft.com/cli/azure/functionapp#az_functionapp_create) | 新しい Function App を作成します |
| [az documentdb create](https://docs.microsoft.com/cli/azure/documentdb#az_documentdb_create) | DocumentDB データベースを作成します |
| [az group delete](https://docs.microsoft.com/cli/azure/group#az_group_delete) | クリーンアップ |

## <a name="next-steps"></a>次のステップ

Azure CLI の詳細については、[Azure CLI のドキュメント](https://docs.microsoft.com/cli/azure/overview)のページをご覧ください。

その他の Azure Functions CLI のサンプル スクリプトは、[Azure Functions のドキュメント](../functions-cli-samples.md)で確認できます。





