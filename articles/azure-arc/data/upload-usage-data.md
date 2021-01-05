---
title: Charger les données d’utilisation sur Azure Monitor
description: Télécharger l’utilisation des données des services de données Azure Arc sur Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
zone_pivot_groups: client-operating-system-macos-and-linux-windows-powershell
ms.openlocfilehash: 578f0d1ca742fe4445b8aeed6876d1a73fd3f79e
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92373417"
---
# <a name="upload-usage-data-to-azure-monitor"></a>Charger les données d’utilisation sur Azure Monitor

Vous pouvez exporter les informations d’utilisation périodiquement. L’exportation et le chargement de ces informations crée et met à jour le contrôleur de données, l’instance managée SQL et les ressources de groupe de serveurs PostgreSQL Hyperscale dans Azure.

> [!NOTE] 
> Il n’y a aucun coût pour les services de données compatibles avec Azure Arc pendant la période de la version préliminaire.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]


> [!NOTE]
> Attendez au moins 24 heures après avoir créé le contrôleur de données Azure Arc avant de charger des données d’utilisation.

## <a name="create-service-principal-and-assign-roles"></a>Créer un principal du service et attribuer des rôles

Avant de continuer, assurez-vous que vous avez créé le principal du service requis et que vous l’avez affecté à un rôle approprié. Pour plus d'informations, consultez :
* [Créez un principal du service](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal).
* [Affecter des rôles au principal du service](upload-metrics-and-logs-to-azure-monitor.md#assign-roles-to-the-service-principal)

## <a name="upload-usage-data"></a>Charger les données d’utilisation

Les informations d’utilisation telles que l’inventaire et l’utilisation des ressources peuvent être téléchargées sur Azure de la manière suivante :

1. Connectez-vous au contrôleur de données. Entrez les valeurs à l’invite. 

   ```console
   azdata login
   ```

1. Exportez les données d’utilisation à l’aide de la commande `azdata arc dc export`, comme suit :

   ```console
   azdata arc dc export --type usage --path usage.json
   ```
 
   Cette commande crée un fichier `usage.json` avec toutes les ressources de données Azure Arc activées, comme les instances managées SQL et les instances PostgreSQL Hyperscale, etc., qui sont créées sur le contrôleur de données.

2. Charger les données d’utilisation à l’aide de la commande ```azdata upload```

   ```console
   azdata arc dc upload --path usage.json
   ```

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

[Charger les journaux sur Azure Monitor](upload-logs.md)

[Charger des données de facturation dans Azure et les afficher dans le portail Azure](view-billing-data-in-azure.md)

[Afficher la ressource de contrôleur de données Azure Arc dans le portail Azure](view-data-controller-in-azure-portal.md)
