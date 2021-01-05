---
title: Charger les journaux sur Azure Monitor
description: Charger les journaux des services de données Azure Arc sur Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
zone_pivot_groups: client-operating-system-macos-and-linux-windows-powershell
ms.openlocfilehash: f3f29ae1ab868a96e6f70ed964f79c47bc591c4d
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92373420"
---
# <a name="upload-logs-to-azure-monitor"></a>Charger les journaux sur Azure Monitor

Régulièrement, vous pouvez exporter les journaux, puis les charger dans Azure. L’exportation et le chargement des journaux créent et mettent à jour le contrôleur de données, l’instance managée SQL et les ressources de groupe de serveurs PostgreSQL Hyperscale dans Azure.

> [!NOTE] 
> Il n’y a aucun coût pour les services de données compatibles avec Azure Arc pendant la période de la version préliminaire.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="before-you-begin"></a>Avant de commencer

Avant de charger les journaux, vous devez : 

1. [Créer un espace de travail Log Analytics](#create-a-log-analytics-workspace)
1. [Affecter un ID et une clé partagée à des variables d’environnement](#assign-id-and-shared-key-to-environment-variables)

## <a name="create-a-log-analytics-workspace"></a>Créer un espace de travail Log Analytics

Pour créer un espace de travail Log Analytics, exécutez ces commandes pour créer un espace de travail Log Analytics et définir les informations d’accès dans des variables d’environnement.

> [!NOTE]
> Si vous avez déjà un espace de travail, ignorez cette étape.

```azurecli
az monitor log-analytics workspace create --resource-group <resource group name> --workspace-name <some name you choose>
```

Exemple de sortie :

```output
{
  "customerId": "d6abb435-2626-4df1-b887-445fe44a4123",
  "eTag": null,
  "id": "/subscriptions/<Subscription ID>/resourcegroups/user-arc-demo/providers/microsoft.operationalinsights/workspaces/user-logworkspace",
  "location": "eastus",
  "name": "user-logworkspace",
  "portalUrl": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "user-arc-demo",
  "retentionInDays": 30,
  "sku": {
    "lastSkuUpdate": "Thu, 30 Jul 2020 22:37:53 GMT",
    "maxCapacityReservationLevel": 3000,
    "name": "pergb2018"
  },
  "source": "Azure",
  "tags": null,
  "type": "Microsoft.OperationalInsights/workspaces"
}
```

## <a name="assign-id-and-shared-key-to-environment-variables"></a>Affecter un ID et une clé partagée à des variables d’environnement

Enregistrez l’espace de travail Log Analytics `customerId` en tant que variable d’environnement à utiliser ultérieurement :

::: zone pivot="client-operating-system-windows-command"

```console
SET WORKSPACE_ID=<customerId>
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:WORKSPACE_ID='<customerId>'
```
::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
export WORKSPACE_ID='<customerId>'
```
::: zone-end

Cette commande renvoie les clés d’accès requises pour se connecter à votre espace de travail Log Analytics :

```azurecli
az monitor log-analytics workspace get-shared-keys --resource-group MyResourceGroup --workspace-name MyLogsWorkpace
```

Exemple de sortie :

```output
{
  "primarySharedKey": "JXzQp1RcGgjXFCDS3v0sXoxPvbgCoGaIv35lf11Km2WbdGFvLXqaydpaj1ByWGvKoCghL8hL4BRoypXxkLr123==",
  "secondarySharedKey": "p2XHSxLJ4o9IAqm2zINcEmx0UWU5Z5EZz8PQC0OHpFjdpuVaI0zsPbTv5VyPFgaCUlCZb2yEbkiR4eTuTSF123=="
}
```

Enregistrez la clé primaire dans une variable d’environnement à utiliser ultérieurement :

::: zone pivot="client-operating-system-windows-command"

```console
SET WORKSPACE_SHARED_KEY=<primarySharedKey>
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```console
$Env:WORKSPACE_SHARED_KEY='<primarySharedKey>'
```
```
::: zone-end


::: zone pivot="client-operating-system-macos-and-linux"

```console
export WORKSPACE_SHARED_KEY='<primarySharedKey>'
```

::: zone-end

## <a name="set-final-environment-variables-and-confirm"></a>Définissez les variables d’environnement finales et confirmez-les

Définissez l’URL de l’autorité du SPN dans une variable d’environnement :

::: zone pivot="client-operating-system-windows-command"

```console
SET SPN_AUTHORITY=https://login.microsoftonline.com
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```console
$Env:SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
export SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end


## <a name="verify-environment-variables"></a>Vérifier les variables d’environnement

Assurez-vous que toutes les variables d’environnement nécessaires sont définies si vous le souhaitez :


::: zone pivot="client-operating-system-windows-command"

```console
echo %WORKSPACE_ID%
echo %WORKSPACE_SHARED_KEY%
echo %SPN_TENANT_ID%
echo %SPN_CLIENT_ID%
echo %SPN_CLIENT_SECRET%
echo %SPN_AUTHORITY%
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:WORKSPACE_ID
$Env:WORKSPACE_SHARED_KEY
$Env:SPN_TENANT_ID
$Env:SPN_CLIENT_ID
$Env:SPN_CLIENT_SECRET
$Env:SPN_AUTHORITY
```


::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
echo $WORKSPACE_ID
echo $WORKSPACE_SHARED_KEY
echo $SPN_TENANT_ID
echo $SPN_CLIENT_ID
echo $SPN_CLIENT_SECRET
echo $SPN_AUTHORITY
```

::: zone-end

Une fois les variables d’environnement définies, vous pouvez charger les journaux dans l’espace de travail. 

## <a name="upload-logs-to-azure-monitor"></a>Charger les journaux sur Azure Monitor

 Pour charger des journaux pour vos instances managées SQL compatibles Azure Arc et groupes de serveurs Azure Arc PostgreSQL Hyperscale, exécutez les commandes CLI suivantes :

1. Connectez-vous au contrôleur de données Azure Arc avec [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)].

   ```console
   azdata login
   ```

   Suivez les invites pour définir l’espace de noms, le nom d’utilisateur de l’administrateur et le mot de passe. 

1. Exporter tous les journaux dans le fichier spécifié :

   ```console
   azdata arc dc export --type logs --path logs.json
   ```

2. Chargez les journaux sur un espace de travail Azure Monitor Log Analytics :

   ```console
   azdata arc dc upload --path logs.json
   ```

## <a name="view-your-logs-in-azure-portal"></a>Afficher vos journaux dans le portail Azure

Une fois vos journaux chargés, vous devez être en mesure de les interroger à l’aide de l’explorateur de requêtes de journal comme suit :

1. Ouvrez le portail Azure, recherchez le nom de votre espace de travail dans la barre de recherche en haut, puis sélectionnez-le.
2. Dans le panneau gauche, sélectionnez Journaux.
3. Sélectionnez Prise en main (ou sélectionnez les liens de la page Prise en main pour en savoir plus sur Log Analytics si vous débutez avec cette fonctionnalité).
4. Suivez le tutoriel pour en savoir plus sur Log Analytics si vous utilisez Log Analytics pour la première fois.
5. Développez Journaux personnalisés au bas de la liste des tables et vous verrez une table appelée « sql_instance_logs_CL ».
6. Sélectionnez l’icône représentant un œil en regard du nom de la table.
7. Sélectionnez le bouton « Afficher dans l’éditeur de requête ».
8. Vous disposez maintenant d’une requête dans l’éditeur de requête, qui affiche les 10 événements les plus récents dans le journal.
9. À partir de là, vous pouvez expérimenter l’interrogation des journaux à l’aide de l’éditeur de requête, définir des alertes, etc.

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

## <a name="next-steps"></a>Étapes suivantes

[Charger les métriques sur Azure Monitor](upload-metrics.md)

[Charger les données d’utilisation, les métriques et les journaux sur Azure Monitor](upload-usage-data.md)

[Charger des données de facturation dans Azure et les afficher dans le portail Azure](view-billing-data-in-azure.md)

[Afficher la ressource de contrôleur de données Azure Arc dans le portail Azure](view-data-controller-in-azure-portal.md)
