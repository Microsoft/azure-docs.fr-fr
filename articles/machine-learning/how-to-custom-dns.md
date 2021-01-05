---
title: Utiliser un serveur DNS personnalisé
titleSuffix: Azure Machine Learning
description: Comment configurer un serveur DNS personnalisé pour qu’il fonctionne avec un espace de travail Azure Machine Learning et un point de terminaison privé.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: jhirono
author: jhirono
ms.date: 11/20/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 07ff656c5eacbbcdc16c6c7cf098478ca6baf745
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509289"
---
# <a name="how-to-use-your-workspace-with-a-custom-dns-server"></a>Utilisation de votre espace de travail avec un serveur DNS personnalisé

Lorsque vous utilisez un espace de travail Azure Machine Learning avec un point de terminaison privé, il existe [plusieurs manières de gérer la résolution de noms DNS](../private-link/private-endpoint-dns.md). Par défaut, Azure gère automatiquement la résolution de noms pour votre espace de travail et votre point de terminaison privé. Si vous utilisez plutôt _votre propre serveur DNS personnalisé_ _, vous devez créer manuellement des entrées DNS pour l’espace de travail.

> [!IMPORTANT]
> Cet article ne traite que de la recherche du nom de domaine complet (FQDN) et des adresses IP pour ces entrées. il ne fournit pas d’informations sur la configuration des enregistrements DNS pour ces éléments. Pour savoir comment ajouter des enregistrements, consultez la documentation de votre logiciel DNS.

## <a name="prerequisites"></a>Prérequis

- Réseau virtuel Azure qui utilise [votre propre serveur DNS](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

- Un espace de travail Azure Machine Learning avec un point de terminaison privé. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

- Connaissances sur l’utilisation de l’[isolement réseau pendant la formation et l’inférence](./how-to-network-security-overview.md).

- Connaissances sur la [configuration de la zone DNS d’un point de terminaison privé Azure](../private-link/private-endpoint-dns.md)

- Éventuellement, [Azure CLI](/cli/azure/install-azure-cli) ou [Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="find-the-ip-addresses"></a>Rechercher les adresses IP

La liste suivante contient les noms de domaine complets (FQDN) utilisés par votre espace de travail et le point de terminaison privé :

* `<workspace-GUID>.workspace.<region>.cert.api.azureml.ms`
* `<workspace-GUID>.workspace.<region>.api.azureml.ms`
* `<workspace-GUID>.workspace.<region>.experiments.azureml.net`
* `<workspace-GUID>.workspace.<region>.modelmanagement.azureml.net`
* `<workspace-GUID>.workspace.<region>.aether.ms`
* `ml-<workspace-name>-<region>-<workspace-guid>.notebooks.azure.net`
* Si vous créez une instance de calcul, vous devez également ajouter une entrée pour `<instance-name>.<region>.instances.azureml.ms` avec l’adresse IP privée du point de terminaison privé de l’espace de travail.

    > [!NOTE]
    > Les instances de calcul sont accessibles uniquement à partir du réseau virtuel.

Pour rechercher les adresses IP internes des noms de domaine complets dans le réseau virtuel, utilisez l’une des méthodes suivantes :

> [!NOTE]
> Les noms de domaine complets et les adresses IP seront différents en fonction de votre configuration. Par exemple, la valeur GUID dans le nom de domaine sera spécifique à votre espace de travail.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az network private-endpoint show --endpoint-name <endpoint> --resource-group <resource-group> --query 'customDnsConfigs[*].{FQDN: fqdn, IPAddress: ipAddresses[0]}' --output table
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$workspaceDns=Get-AzPrivateEndpoint -Name <endpoint> -resourcegroupname <resource-group>
$workspaceDns.CustomDnsConfigs | format-table
```

# <a name="azure-portal"></a>[Azure portal](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez votre __espace de travail__ Azure Machine Learning.
1. Dans la section __Paramètres__, sélectionnez __Connexions des points de terminaison privés__.
1. Sélectionnez le lien dans la colonne __Point de terminaison privé__ qui s’affiche.
1. La liste des noms de domaine complets (FQDN) et les adresses IP du point de terminaison privé de l’espace de travail se trouvent au bas de la page.

:::image type="content" source="./media/how-to-custom-dns/private-endpoint-custom-dns.png" alt-text="Liste des noms de domaine complets dans le portail":::

---

Les informations retournées par toutes les méthodes sont les mêmes : une liste du nom de domaine complet et de l’adresse IP privée pour les ressources.

| FQDN | Adresse IP |
| ----- | ----- |
| `fb7e20a0-8891-458b-b969-55ddb3382f51.workspace.eastus.api.azureml.ms` | `10.1.0.5` |
| `ml-myworkspace-eastus-fb7e20a0-8891-458b-b969-55ddb3382f51.notebooks.azure.net` | `10.1.0.6` |

> [!IMPORTANT]
> Certains noms de domaine complets ne sont pas affichés dans la liste par le point de terminaison privé, mais ils sont requis par l’espace de travail. Ces noms de domaine complets sont répertoriés dans le tableau suivant et doivent également être ajoutés à votre serveur DNS :
>
> * `<workspace-GUID>.workspace.<region>.cert.api.azureml.ms`
> * `<workspace-GUID>.workspace.<region>.experiments.azureml.net`
> * `<workspace-GUID>.workspace.<region>.modelmanagement.azureml.net`
> * `<workspace-GUID>.workspace.<region>.aether.ms`
> * Si vous disposez d’une instance de calcul, utilisez `<instance-name>.<region>.instances.azureml.ms`, où `<instance-name>` est le nom de votre instance de calcul. Utilisez l’adresse IP privée du point de terminaison privé de l’espace de travail. Notez que l’instance de calcul est accessible uniquement à partir du réseau virtuel.
>
> Pour toutes ces adresses IP, utilisez la même adresse que les entrées `*.api.azureml.ms` retournées par les étapes précédentes.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation d’Azure Machine Learning avec un réseau virtuel, consultez [Présentation du réseau virtuel](how-to-network-security-overview.md).
