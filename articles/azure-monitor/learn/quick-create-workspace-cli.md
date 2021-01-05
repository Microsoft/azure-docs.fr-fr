---
title: Créer un espace de travail Log Analytics à l’aide d’Azure CLI | Microsoft Docs
description: Découvrez comment créer un espace de travail Log Analytics pour permettre la collecte de données et les solutions de gestion à partir de vos environnements locaux et cloud avec Azure CLI.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/26/2020
ms.openlocfilehash: 54d1d8a29c87f8d129c0ea5b29973c4fef0e6f7a
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888995"
---
# <a name="create-a-log-analytics-workspace-with-azure-cli-20"></a>Créer un espace de travail Log Analytics avec Azure CLI 2.0

Vous utilisez Azure CLI 2.0 pour créer et gérer des ressources Azure à partir de la ligne de commande ou dans les scripts. Ce démarrage rapide vous montre comment utiliser l’interface de ligne de commande Azure 2.0 pour déployer un espace de travail Log Analytics dans Azure Monitor. Un espace de travail Log Analytics est un environnement unique pour les données de journal d’activité Azure Monitor. Chaque espace de travail dispose d’un référentiel de données et d’une configuration propres. Les sources de données et les solutions sont configurées de façon à stocker leurs données dans un espace de travail particulier. Vous devez avoir un espace de travail Log Analytics si vous souhaitez collecter des données à partir des sources suivantes :

* Ressources Azure dans votre abonnement  
* Ordinateurs locaux surveillés par System Center Operations Manager  
* Regroupements d'appareils à partir de Configuration Manager  
* Données de diagnostics et de journaux du Stockage Azure  

Pour les autres sources, telles que les machines virtuelles Azure et les machines virtuelles Windows ou Linux dans votre environnement, consultez les rubriques suivantes :

* [Collect data from Azure virtual machines](./quick-collect-azurevm.md) (Collecter des données à partir de machines virtuelles Azure)
* [Collect data from hybrid Linux computer](./quick-collect-linux-computer.md) (Collecter des données à partir d’un ordinateur Linux hybride)
* [Collect data from hybrid Windows computer](quick-collect-windows-computer.md) (Collecter des données à partir d’un ordinateur Windows hybride)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.30 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-workspace"></a>Créer un espace de travail
Créez un espace de travail avec [az group deployment create](/cli/azure/group/deployment?view=azure-cli-latest#az-group-deployment-create). L’exemple suivant crée un espace de travail à l’emplacement *eastus* à l’aide d’un modèle Resource Manager à partir de votre ordinateur local. Le modèle JSON est configuré pour vous demander uniquement le nom de l’espace de travail et spécifie une valeur par défaut pour les autres paramètres susceptibles d’être utilisés comme configuration standard dans votre environnement. Vous pouvez aussi stocker le modèle dans un compte de stockage Azure pour mettre en place un accès partagé dans votre organisation. Pour plus d’informations sur le fonctionnement des modèles, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure CLI](../../azure-resource-manager/templates/deploy-cli.md).

Pour plus d’informations sur les régions prises en charge, consultez les [régions dans lesquelles Log Analytics est disponible](https://azure.microsoft.com/regions/services/) et recherchez Azure Monitor à partir du champ **Rechercher un produit**.

Les paramètres suivants sont définis comme une valeur par défaut :

* Emplacement : par défaut, USA Est
* Référence SKU : la valeur par défaut est le nouveau niveau de tarification par Go publié dans le modèle de tarification d’avril 2018.

>[!WARNING]
>Lors de la création ou de la configuration d’un espace de travail Log Analytics dans un abonnement pour lequel le nouveau modèle de tarification d’avril 2018 a été choisi, le seul niveau tarifaire Log Analytics valide est **PerGB2018**.
>

### <a name="create-and-deploy-template"></a>Créer et déployer un modèle

1. Copiez et collez la syntaxe JSON suivante dans votre fichier :

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "String",
            "metadata": {
              "description": "Specifies the name of the workspace."
            }
        },
        "location": {
            "type": "String",
            "allowedValues": [
              "eastus",
              "westus"
            ],
            "defaultValue": "eastus",
            "metadata": {
              "description": "Specifies the location in which to create the workspace."
            }
        },
        "sku": {
            "type": "String",
            "allowedValues": [
              "Standalone",
              "PerNode",
              "PerGB2018"
            ],
            "defaultValue": "PerGB2018",
            "metadata": {
            "description": "Specifies the service tier of the workspace: Standalone, PerNode, Per-GB"
        }
          }
    },
    "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('workspaceName')]",
            "apiVersion": "2015-11-01-preview",
            "location": "[parameters('location')]",
            "properties": {
                "sku": {
                    "Name": "[parameters('sku')]"
                },
                "features": {
                    "searchVersion": 1
                }
            }
          }
       ]
    }
    ```

2. Modifiez le modèle en fonction de vos besoins. Consultez la référence [Microsoft.OperationalInsights/workspaces modèle](/azure/templates/microsoft.operationalinsights/2015-11-01-preview/workspaces) pour découvrir les propriétés et les valeurs prises en charge.
3. Enregistrez ce fichier au format **deploylaworkspacetemplate.json** dans un dossier local.   
4. Vous êtes prêt à déployer ce modèle. Utilisez les commandes suivantes à partir du dossier qui contient le modèle. Lorsque vous êtes invité à entrer un nom d’espace de travail, fournissez un nom globalement unique dans tous les abonnements Azure.

    ```azurecli
    az group deployment create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deploylaworkspacetemplate.json
    ```

Le déploiement peut prendre plusieurs minutes. Lorsqu’il est terminé, vous voyez un message similaire au message suivant qui inclut le résultat :

![Exemple de résultat lorsque le déploiement est achevé](media/quick-create-workspace-cli/template-output-01.png)

## <a name="troubleshooting"></a>Dépannage
Lorsque vous créez un espace de travail qui a été supprimé au cours des 14 derniers jours et qui se trouve dans l’[état de suppression réversible](../platform/delete-workspace.md#soft-delete-behavior), l’opération peut avoir des résultats différents en fonction de la configuration de votre espace de travail :
1. Si vous fournissez les mêmes nom d’espace de travail, groupe de ressources, abonnement et région que dans l’espace de travail supprimé, votre espace de travail est récupéré avec ses données, sa configuration et ses agents connectés.
2. Si vous utilisez le même nom d’espace de travail, mais un groupe de ressources, un abonnement ou une région différents, vous recevez un message erreur indiquant que *le nom de l’espace de travail « nom-espace-de-travail » n’est pas unique* ou qu’il y a un *conflit*. Pour annuler la suppression réversible afin de supprimer définitivement votre espace de travail et d’en créer un nouveau sous le même nom, procédez comme suit pour récupérer l’espace de travail avant d’effectuer la suppression définitive :
   * [Récupérer](../platform/delete-workspace.md#recover-workspace) votre espace de travail
   * [Supprimer définitivement](../platform/delete-workspace.md#permanent-workspace-delete) votre espace de travail
   * Créer un espace de travail en reprenant le même nom d’espace de travail

## <a name="next-steps"></a>Étapes suivantes
Disposant à présent d’un espace de travail, vous pouvez configurer la collecte des données de télémétrie de surveillance, exécuter des recherches dans les journaux pour analyser ces données et ajouter une solution de gestion pour fournir des données et insights analytiques supplémentaires.  

* Pour activer la collecte de données à partir de ressources Azure avec Diagnostics Azure ou le stockage Azure, consultez [Collecte des journaux et des métriques des services Azure à utiliser dans Log Analytics](../platform/resource-logs.md#send-to-log-analytics-workspace).  
* Ajoutez [System Center Operations Manager comme source de données](../platform/om-agents.md) pour collecter des données à partir des agents rattachés à votre groupe d’administration Operations Manager et les stocker dans votre espace de travail Log Analytics.  
* Connectez [Configuration Manager](../platform/collect-sccm.md) pour importer des ordinateurs qui sont membres de collections dans la hiérarchie.  
* Examinez les [solutions de surveillance](../insights/solutions.md) disponibles et la façon d’ajouter ou de supprimer une solution dans votre espace de travail.

