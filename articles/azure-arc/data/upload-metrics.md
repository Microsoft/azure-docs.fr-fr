---
title: Charger les métriques sur le Azure Monitor
description: Charger les métriques des services de données Azure Arc sur Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
zone_pivot_groups: client-operating-system-macos-and-linux-windows-powershell
ms.openlocfilehash: f319f912520a69a0c68f89a3d4178f63cc45ca1f
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356546"
---
# <a name="upload-metrics-to-azure-monitor"></a>Charger les métriques sur le Azure Monitor

Régulièrement, vous pouvez exporter les métriques de surveillance, puis les charger dans Azure. L’exportation et le chargement des données crée et met à jour le contrôleur de données, l’instance managée SQL et les ressources de groupe de serveurs PostgreSQL Hyperscale dans Azure.

> [!NOTE] 
> Il n’y a aucun coût pour les services de données compatibles avec Azure Arc pendant la période de la version préliminaire.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Prérequis

Avant de continuer, assurez-vous que vous avez créé le principal du service requis et que vous l’avez affecté à un rôle approprié. Pour plus d'informations, consultez :
* [Créez un principal du service](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal).
* [Affecter des rôles au principal du service](upload-metrics-and-logs-to-azure-monitor.md#assign-roles-to-the-service-principal)

## <a name="upload-metrics"></a>Charger les métriques

Avec les services de données Azure Arc, vous pouvez éventuellement charger vos métriques sur Azure Monitor afin de pouvoir agréger et analyser les métriques, déclencher des alertes, envoyer des notifications ou déclencher des actions automatisées. 

L’envoi de vos données à Azure Monitor vous permet également de stocker des données de métrique et de journalisation hors site et à grande échelle, ce qui permet un stockage à long terme des données pour l’analytique avancée.

Si vous disposez de plusieurs sites avec Azure Arc Data Services, vous pouvez utiliser Azure Monitor comme emplacement central pour collecter tous les journaux et toutes les métriques de vos sites.

## <a name="set-final-environment-variables-and-confirm"></a>Définissez les variables d’environnement finales et confirmez-les

Définissez l’URL de l’autorité du SPN dans une variable d’environnement :

::: zone pivot="client-operating-system-windows-command"

```console
SET SPN_AUTHORITY=https://login.microsoftonline.com
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
export SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end

Assurez-vous que toutes les variables d’environnement nécessaires sont définies si vous le souhaitez :


::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:SPN_TENANT_ID
$Env:SPN_CLIENT_ID
$Env:SPN_CLIENT_SECRET
$Env:SPN_AUTHORITY
```


::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
echo $SPN_TENANT_ID
echo $SPN_CLIENT_ID
echo $SPN_CLIENT_SECRET
echo $SPN_AUTHORITY
```

::: zone-end

::: zone pivot="client-operating-system-windows-command"

```console
echo %SPN_TENANT_ID%
echo %SPN_CLIENT_ID%
echo %SPN_CLIENT_SECRET%
echo %SPN_AUTHORITY%
```

::: zone-end

## <a name="upload-metrics-to-azure-monitor"></a>Charger les métriques sur le Azure Monitor

Pour charger les métriques de vos instances managées SQL compatibles Azure Arc et les groupes de serveurs PostgreSQL Hyperscale compatibles Azure Arc, exécutez les commandes CLI suivantes :

1. Connectez-vous au contrôleur de données avec `azdata`.
 
1. Exportez toutes les métriques vers le fichier spécifié :

   ```console
   azdata arc dc export --type metrics --path metrics.json
   ```

2. Charger les métriques sur le Azure Monitor :

   ```console
   azdata arc dc upload --path metrics.json
   ```

   >[!NOTE]
   >Attendez au moins 30 minutes après la création des instances de données compatibles avec Azure Arc pour le premier chargement.
   >
   >Veillez à `upload` les mesures immédiatement après `export`, car Azure Monitor accepte uniquement les métriques des 30 dernières minutes. [En savoir plus](../../azure-monitor/platform/metrics-store-custom-rest-api.md#troubleshooting)


Si vous voyez des erreurs indiquant « Impossible d’obtenir les métriques » pendant l’exportation, vérifiez si la collecte de données est définie sur `true` en exécutant la commande suivante :

```console
azdata arc dc config show
```

regardez sous la « section relative à la sécurité »

```output
 "security": {
      "allowDumps": true,
      "allowNodeMetricsCollection": true,
      "allowPodMetricsCollection": true,
      "allowRunAsRoot": false
    },
```

Vérifiez si les propriétés `allowNodeMetricsCollection` et `allowPodMetricsCollection` sont définies sur `true`.

## <a name="view-the-metrics-in-the-portal"></a>Afficher les métriques dans le portail

Une fois vos métriques chargées, vous pouvez les afficher à partir du Portail Azure.
> [!NOTE]
> Notez que le traitement des données chargées peut prendre quelques minutes avant que vous puissiez afficher les métriques dans le portail.


Pour voir vos métriques dans le portail, utilisez ce lien pour ouvrir le portail : <https://portal.azure.com> Ensuite, recherchez le nom de votre instance de base de données dans la barre de recherche :

Vous pouvez voir l’utilisation du processeur sur la page de vue d’ensemble ou, si vous souhaitez des métriques plus détaillées, vous pouvez cliquer sur les métriques dans le panneau de navigation de gauche.

Choisissez SQL Server comme espace de noms de métrique :

Sélectionnez la métrique que vous souhaitez voir (vous pouvez également en sélectionner plusieurs) :

Modifiez la fréquence pour la définir sur les 30 dernières minutes :

> [!NOTE]
> Vous ne pouvez charger les métriques que pour les 30 dernières minutes. Azure Monitor rejette les métriques datant de plus de 30 minutes.

## <a name="automating-uploads-optional"></a>Automatisation des chargements (facultatif)

Si vous souhaitez charger les métriques et les journaux sur une base planifiée, vous pouvez créer un script et l’exécuter sur la base d’un minuteur toutes les quelques minutes. Voici un exemple d’automatisation des chargements à l’aide d’un script de shell Linux.

Dans votre éditeur de texte ou de code, ajoutez ce qui suit au script dans le fichier, puis enregistrez-le en tant que fichier exécutable de script, tel que. sh (Linux/Mac) ou. cmd, .bat, .ps1.

```console
azdata arc dc export --type metrics --path metrics.json --force
azdata arc dc upload --path metrics.json
```

Rendre le fichier de script exécutable

```console
chmod +x myuploadscript.sh
```

Exécutez le script toutes les 20 minutes :

```console
watch -n 1200 ./myuploadscript.sh
```

Vous pouvez également utiliser un planificateur de travaux comme cron ou le Planificateur de tâches Windows ou encore un orchestrateur comme Ansible, Puppet ou Chef.

## <a name="general-guidance-on-exporting-and-uploading-usage-metrics"></a>Conseils généraux sur l’exportation et le chargement des métriques d’utilisation

Les opérations de création, lecture, mise à jour et suppression (CRUD) sur les services de données compatibles Azure Arc sont journalisées à des fins de facturation et de surveillance. Il existe des services en arrière-plan qui surveillent ces opérations CRUD et calculent la consommation de manière appropriée. Le calcul réel de l’utilisation ou de la consommation se produit de manière planifiée et est effectué en arrière-plan. 

Pendant la préversion, ce processus se produit la nuit. La recommandation générale est de charger les données d’utilisation une seule fois par jour. Lorsque les informations d’utilisation sont exportées et chargées plusieurs fois au cours de la même période de 24 heures, seul l’inventaire des ressources est mis à jour dans le Portail Azure mais pas dans l’utilisation des ressources.

Pour le chargement des mesures, Azure Monitor accepte uniquement les 30 dernières minutes de données ([En savoir plus](../../azure-monitor/platform/metrics-store-custom-rest-api.md#troubleshooting)). La recommandation pour le chargement des métriques est de charger les métriques immédiatement après la création du fichier d’exportation afin de pouvoir afficher l’ensemble du jeu de données dans le Portail Azure. Par exemple, supposons que vous avez exporté les métriques à 14 h et que vous avez exécuté la commande de chargement à 14 h 50. Étant donné qu’Azure Monitor n’accepte que les données des 30 dernières minutes, vous ne verrez aucune donnée dans le portail. 

## <a name="next-steps"></a>Étapes suivantes

[Charger les journaux sur Azure Monitor](upload-logs.md)

[Charger les données d’utilisation, les métriques et les journaux sur Azure Monitor](upload-usage-data.md)

[Charger des données de facturation dans Azure et les afficher dans le portail Azure](view-billing-data-in-azure.md)

[Afficher la ressource de contrôleur de données Azure Arc dans le portail Azure](view-data-controller-in-azure-portal.md)
