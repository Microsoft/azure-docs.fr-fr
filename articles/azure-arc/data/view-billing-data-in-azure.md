---
title: Télécharger des données de facturation dans Azure et les afficher dans le portail Azure
description: Télécharger des données de facturation dans Azure et les afficher dans le portail Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 9da725c433ad5d6233fd164d256692ca407714fc
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206450"
---
# <a name="upload-billing-data-to-azure-and-view-it-in-the-azure-portal"></a>Télécharger des données de facturation dans Azure et les afficher dans le portail Azure

> [!IMPORTANT] 
>  L’utilisation de services de données compatibles avec Azure Arc pendant la période de la préversion n’occasionne aucun coût. Bien que le système de facturation fonctionne de bout en bout, le compteur de facturation est défini sur $0.  Si vous suivez ce scénario, vous verrez des entrées dans votre facturation pour un service nommé **services de données hybrides** et pour des ressources d’un type appelé **microsoft.AzureData/`<resource type>`** . Vous pourrez voir un enregistrement pour chaque service de données Azure Arc que vous créez, mais chaque enregistrement sera facturé à $0.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="connectivity-modes---implications-for-billing-data"></a>Modes de connectivité – Implications pour les données de facturation

À l’avenir, il y aura deux modes dans lesquels vous pourrez exécuter vos services de données compatibles Azure Arc :

- **Connecté indirectement** – Il n’existe pas de connexion directe à Azure. Les données sont envoyées à Azure uniquement via un processus d’exportation/chargement. Tous les déploiements de services de données Azure Arc fonctionnent dans ce mode actuellement en préversion.
- **Connecté directement** – Dans ce mode, il y aura une dépendance du service Kubernetes activé par Azure Arc pour fournir une connexion directe entre Azure et le cluster Kubernetes sur lequel s’exécutent les services de données activés par Azure Arc. Cela permet d’obtenir davantage de fonctionnalités ainsi que d’utiliser le portail Azure et Azure CLI pour gérer vos services de données compatibles Azure Arc, tout comme vous gérez vos services de données dans Azure PaaS.  Ce mode de connectivité n’est pas encore disponible en préversion, mais le sera bientôt.

Si vous le souhaitez, vous pouvez en savoir plus sur la différence entre les [modes de connectivité](./connectivity.md).

Dans le mode connecté indirectement, les données de facturation sont régulièrement exportées du contrôleur de données Azure Arc vers un fichier sécurisé, puis chargées dans Azure et traitées.  Dans le mode connecté directement à venir, les données de facturation seront automatiquement envoyées à Azure environ 1 fois par heure pour fournir une vue en quasi-temps réel des coûts de vos services. Le processus d’exportation et de chargement des données dans le mode connecté indirectement peut également être automatisé à l’aide de scripts, ou nous pouvons créer un service qui le fera pour vous.

## <a name="upload-billing-data-to-azure"></a>Charger des données de facturation dans Azure

Pour charger des données de facturation dans Azure, les actions suivantes doivent avoir lieu au préalable :

1. Créez un service de données activé par Azure Arc si vous n’en avez pas encore. Par exemple, créez l’un des éléments suivants :
   - [Créer une instance gérée Azure SQL sur Azure Arc](create-sql-managed-instance.md)
   - [Créer un groupe de serveurs PostgreSQL Hyperscale compatibles avec Azure Arc](create-postgresql-hyperscale-server-group.md)
1. [Charger l’inventaire des ressources, les données d’utilisation, les métriques et les journaux sur Azure Monitor](upload-metrics-and-logs-to-azure-monitor.md) si vous ne l’avez pas encore fait.
1. Attendez au moins 2 heures à partir de la création du service de données afin que le processus de collecte de la télémétrie de facturation puisse collecter des données de facturation.

Exécutez la commande suivante pour exporter les données de facturation :

```console
azdata arc dc export -t usage -p usage.json
```

Actuellement, le fichier n’étant pas chiffré, vous pouvez en voir le contenu. N’hésitez pas à l’ouvrir dans un éditeur de texte pour voir à quoi ressemble le contenu.

Vous remarquerez qu’il existe deux ensembles de données : `resources` et `data`. L’ensemble de données `resources` est celui du contrôleur de données, des groupes de serveurs PostgreSQL Hyperscale et des instances managées SQL. Les enregistrements `resources` dans les données capturent les événements pertinents dans l’historique d’une ressource, lors de sa création, lors de sa mise à jour et lors de sa suppression. Les enregistrements `data` capturent le nombre de cœurs disponibles qu’une instance donnée peut utiliser à chaque heure.

Exemple d’entrée `resource` :

```console
    {
        "customObjectName": "<resource type>-2020-29-5-23-13-17-164711",
        "uid": "4bc3dc6b-9148-4c7a-b7dc-01afc1ef5373",
        "instanceName": "sqlInstance001",
        "instanceNamespace": "arc",
        "instanceType": "<resource>",
        "location": "eastus",
        "resourceGroupName": "production-resources",
        "subscriptionId": "482c901a-129a-4f5d-86e3-cc6b294590b2",
        "isDeleted": false,
        "externalEndpoint": "32.191.39.83:1433",
        "vCores": "2",
        "createTimestamp": "05/29/2020 23:13:17",
        "updateTimestamp": "05/29/2020 23:13:17"
    }
```

Exemple d’entrée `data` :

```console
        {
          "requestType": "usageUpload",
          "clusterId": "4b0917dd-e003-480e-ae74-1a8bb5e36b5d",
          "name": "DataControllerTestName",
          "subscriptionId": "482c901a-129a-4f5d-86e3-cc6b294590b2",
          "resourceGroup": "production-resources",
          "location": "eastus",
          "uploadRequest": {
            "exportType": "usages",
            "dataTimestamp": "2020-06-17T22:32:24Z",
            "data": "[{\"name\":\"sqlInstance001\",
                       \"namespace\":\"arc\",
                       \"type\":\"<resource type>\",
                       \"eventSequence\":1, 
                       \"eventId\":\"50DF90E8-FC2C-4BBF-B245-CB20DC97FF24\",
                       \"startTime\":\"2020-06-17T19:11:47.7533333\",
                       \"endTime\":\"2020-06-17T19:59:00\",
                       \"quantity\":1,
                       \"id\":\"4BC3DC6B-9148-4C7A-B7DC-01AFC1EF5373\"}]",
           "signature":"MIIE7gYJKoZIhvcNAQ...2xXqkK"
          }
        }
```

Exécutez la commande suivante pour charger le fichier usage.json dans Azure :

```console
azdata arc dc upload -p usage.json
```

## <a name="view-billing-data-in-azure-portal"></a>Afficher les données de facturation dans le portail Azure

Pour afficher les données de facturation dans le portail Azure, procédez comme suit :

1. Ouvrez le portail Azure en utilisant l’URL spéciale : [https://aka.ms/arcdata](https://aka.ms/arcdata).
1. Dans la zone de recherche en haut du type d’écran, tapez **Cost Management** , puis cliquez sur le service Cost Management.
1. Cliquez sur l’onglet **Analyse des coûts** à gauche.
1. Cliquez sur le bouton **Coût par ressource** en haut de la vue.
1. Assurez-vous que l’Étendue est définie sur l’abonnement dans lequel vos ressources de service de données ont été créées.
1. Sélectionnez **Coût par ressource** dans la liste déroulante Afficher en regard du sélecteur Étendue en haut de la vue.
1. Assurez-vous que le filtre de date est défini sur **Mois en cours** ou sur un autre intervalle de temps semblant logique compte tenu de la date de création des ressources de votre service de données.
1. Si vous souhaitez filtrer un seul type de service de données activé par Azure Arc, cliquez sur **Ajouter un filtre** afin d’ajouter un filtre par **Type de ressource** = `microsoft.azuredata/<data service type>`.
1. Vous voyez maintenant une liste de toutes les ressources qui ont été créées et chargées dans Azure. Étant donné que le compteur de facturation indique $0, le coût affiché sera toujours $0.

## <a name="download-billing-data"></a>Télécharger les données de facturation

Vous pouvez télécharger les données de synthèse de facturation directement à partir du portail Azure.

1. Dans la vue **Analyse des coûts -> Afficher par type de ressource** que vous avez atteinte en suivant les instructions ci-dessus, cliquez sur le bouton Télécharger en haut.
1. Choisissez le type de fichier à télécharger (Excel ou CSV), puis cliquez sur le bouton **Télécharger des données** .
1. Ouvrez le fichier dans un éditeur approprié en fonction du type de fichier sélectionné.

## <a name="export-billing-data"></a>Exporter les données de facturation

Vous pouvez également exporter automatiquement les données d’utilisation et de facturation **détaillées** à intervalles réguliers dans un conteneur Stockage Azure en créant un travail d’exportation pour la facturation. Cela est utile si vous souhaitez afficher les détails de votre facturation, par exemple le nombre d’heures facturées pour une instance donnée au cours de la période de facturation.

Pour configurer un travail d’exportation de facturation, procédez comme suit :

1. Cliquez **Exportations** à gauche.
1. Cliquez sur **Add** .
1. Entrez un nom et une fréquence d’exportation, puis cliquez sur Suivant.
1. Choisissez un compte de stockage existant ou créez-en un, puis remplissez le formulaire pour spécifier le compte de stockage, le conteneur et le chemin d’accès du répertoire dans lequel exporter les fichiers de données de facturation, puis cliquez sur Suivant.
1. Cliquez sur **Créer** .

Les fichiers d’exportation des données de facturation seront disponibles dans environ 4 heures, et seront exportés selon la planification que vous avez spécifiée lors de la création du travail d’exportation de la facturation.

Pour afficher les fichiers de données de facturation exportés, procédez comme suit :

Vous pouvez valider les fichiers de données de facturation dans le portail Azure. 

> [!IMPORTANT]
> Après avoir créé le travail d’exportation de facturation, patientez 4 heures avant de passer aux étapes suivantes.

1. Dans la zone de recherche en haut du portail, tapez **Comptes de stockage** , puis cliquez sur **Comptes de stockage** .
3. Cliquez sur le compte de stockage que vous avez spécifié lors de la création du travail d’exportation de facturation ci-dessus.
4. Cliquez sur Conteneurs à gauche.
5. Cliquez sur le conteneur que vous avez spécifié lors de la création du travail d’exportation de facturation ci-dessus.
6. Cliquez sur le dossier que vous avez spécifié lors de la création du travail d’exportation de facturation ci-dessus.
7. Explorez les dossiers et fichiers générés, puis cliquez sur l’un des fichiers. csv générés.
8. Cliquez sur le bouton **Télécharger** pour enregistrer le fichier dans votre dossier Téléchargements local.
9. Ouvrez le fichier à l’aide d’une visionneuse de fichiers. csv telle qu’Excel.
10. Filtrez les résultats pour afficher uniquement les lignes avec le **Type de ressource** = `Microsoft.AzureData/<data service resource type`.
11. Vous verrez le nombre d’heures d’utilisation de l’instance au cours de la période actuelle de 24 heures dans la colonne UsageQuantity.
