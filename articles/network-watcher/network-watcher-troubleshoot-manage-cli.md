---
title: Résoudre les problèmes liés à la passerelle de réseau virtuel et aux connexions Azure – Azure CLI
titleSuffix: Azure Network Watcher
description: Cette page explique comment utiliser Azure Network Watcher pour résoudre les problèmes liés à l’interface de ligne de commande Azure
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/19/2017
ms.author: damendo
ms.openlocfilehash: a35d6449d88620c2ac977ae45584c59f665378bf
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94960493"
---
# <a name="troubleshoot-virtual-network-gateway-and-connections-using-azure-network-watcher-azure-cli"></a>Résoudre les problèmes liés à la passerelle de réseau virtuel et aux connexions Azure par le biais de l’interface de ligne de commande Azure d’Azure Network Watcher

> [!div class="op_single_selector"]
> - [Portail](diagnose-communication-problem-between-networks.md)
> - [PowerShell](network-watcher-troubleshoot-manage-powershell.md)
> - [Azure CLI](network-watcher-troubleshoot-manage-cli.md)
> - [REST API](network-watcher-troubleshoot-manage-rest.md)

Le service Network Watcher offre de nombreuses fonctionnalités en lien avec la bonne compréhension de vos ressources réseau dans Azure. Il permet notamment de résoudre les problèmes liés aux ressources. Vous pouvez appeler la solution de résolution des problèmes de ressources par le biais du portail, de PowerShell, de l’interface de ligne de commande ou de l’API REST. Lorsque cette fonctionnalité est appelée, Network Watcher inspecte l’intégrité d’une passerelle de réseau virtuel ou d’une connexion et renvoie ses résultats.

Pour exécuter la procédure indiquée dans cet article, vous devez [installer l’interface de ligne de commande Azure pour Mac, Linux et Windows (CLI)](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Avant de commencer

Ce scénario suppose que vous ayez déjà suivi la procédure décrite dans [Create a Network Watcher (Créer une instance Network Watcher)](network-watcher-create.md) pour créer une instance Network Watcher.

Vous trouverez la liste des types de passerelles pris en charge sur la page [Types de passerelles pris en charge](network-watcher-troubleshoot-overview.md#supported-gateway-types).

## <a name="overview"></a>Vue d’ensemble

La résolution des problèmes liés aux ressources offre la possibilité de résoudre les problèmes qui surviennent avec les passerelles de réseau virtuel et les connexions. Lorsqu’une demande de résolution des problèmes liés aux ressources est faite, les journaux d’activité sont interrogés et inspectés. Lorsque l’inspection est terminée, les résultats sont renvoyés. Les demandes de résolution des problèmes liés aux ressources sont longues : vous devrez peut-être patienter plusieurs minutes avant d’obtenir un résultat. Les journaux d’activité de dépannage sont stockés dans un conteneur sur un compte de stockage spécifié.

## <a name="retrieve-a-virtual-network-gateway-connection"></a>Récupérer une connexion de passerelle de réseau virtuel

Dans cet exemple, la résolution des problèmes liés aux ressources s’effectue sur une connexion. Vous pouvez également la transférer à une passerelle de réseau virtuel. L’applet de commande suivante répertorie les connexions vpn dans un groupe de ressources.

```azurecli
az network vpn-connection list --resource-group resourceGroupName
```

Une fois que vous connaissez le nom de la connexion, vous pouvez exécuter cette commande pour obtenir son ID de ressource :

```azurecli
az network vpn-connection show --resource-group resourceGroupName --ids vpnConnectionIds
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

La résolution des problèmes liés aux ressources renvoie des données concernant l’intégrité de la ressource, et elle enregistre les journaux d’activité dans un compte de stockage qui fera l’objet d’une révision. À cette étape, nous créons un compte de stockage. Si vous disposez déjà d’un compte de stockage existant, vous pouvez l’utiliser.

1. Créer le compte de stockage

    ```azurecli
    az storage account create --name storageAccountName --location westcentralus --resource-group resourceGroupName --sku Standard_LRS
    ```

1. Obtenir les clés du compte de stockage

    ```azurecli
    az storage account keys list --resource-group resourcegroupName --account-name storageAccountName
    ```

1. Créer le conteneur

    ```azurecli
    az storage container create --account-name storageAccountName --account-key {storageAccountKey} --name logs
    ```

## <a name="run-network-watcher-resource-troubleshooting"></a>Procéder à la résolution des problèmes liés aux ressources Network Watcher

Résolvez les problèmes liés aux ressources avec l’applet de commande `az network watcher troubleshooting`. Nous transférons à l’applet de commande le groupe de ressources, le nom de Network Watcher, l’identifiant de la connexion, l’identifiant du compte de stockage, et le chemin d’accès à l’objet blob pour y stocker les résultats de résolution des problèmes.

```azurecli
az network watcher troubleshooting start --resource-group resourceGroupName --resource resourceName --resource-type {vnetGateway/vpnConnection} --storage-account storageAccountName  --storage-path https://{storageAccountName}.blob.core.windows.net/{containerName}
```

Une fois l’applet de commande exécutée, Network Watcher passe en revue la ressource pour vérifier son intégrité. Les résultats sont renvoyés à l’interpréteur de commandes, et les journaux d’activité des résultats sont stockés dans le compte de stockage spécifié.

## <a name="understanding-the-results"></a>Compréhension des résultats

Le texte d’action fournit des indications générales sur la façon de résoudre le problème. Si une action peut être entreprise pour résoudre le problème, un lien est fourni avec des indications supplémentaires. En l’absence d’indications supplémentaires, la réponse fournit l’URL permettant d’ouvrir un dossier de support.  Pour plus d’informations sur les propriétés de la réponse et sur ce qu’elle contient, consultez [Network Watcher Troubleshoot overview (Vue d’ensemble de la résolution des problèmes Network Watcher)](network-watcher-troubleshoot-overview.md)

Pour obtenir des instructions de téléchargement des fichiers à partir des comptes de stockage Azure, consultez [Prise en main du stockage d’objets blob Azure à l’aide de .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md). L’explorateur de stockage peut aussi être utilisé. Pour en savoir plus sur l’explorateur de stockage, cliquez sur le lien suivant : [Explorateur de stockage](https://storageexplorer.com/)

## <a name="next-steps"></a>Étapes suivantes

Si les paramètres ont été modifiés et arrêtent la connectivité VPN, consultez la page [Gérer les groupes de sécurité réseau à partir du portail](../virtual-network/manage-network-security-group.md) afin d’effectuer le suivi du groupe de sécurité réseau et des règles de sécurité pouvant être concernés.