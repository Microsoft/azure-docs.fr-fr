---
title: Préparation à la modification du format dans les journaux de ressources Azure Monitor
description: Journaux de ressources Azure déplacés pour utiliser des objets blob d’ajout le 1er novembre 2018.
author: johnkemnetz
services: monitoring
ms.topic: conceptual
ms.date: 07/06/2018
ms.author: johnkem
ms.subservice: logs
ms.openlocfilehash: 45b18352d88877a5d611f203d87da83fd0d58c6b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87077125"
---
# <a name="prepare-for-format-change-to-azure-monitor-platform-logs-archived-to-a-storage-account"></a>Préparation à la modification du format dans les journaux de plateforme Azure Monitor archivés dans un compte de stockage

> [!WARNING]
> Si vous envoyez des [journaux ou des métriques d’activité des ressources Azure à un compte de stockage via les paramètres de ressources](./resource-logs.md#send-to-azure-storage) ou [des journaux d’activité à un compte de stockage via les profils de journaux d’activité](./resource-logs.md#send-to-azure-storage), le format des données dans le compte de stockage a été modifié en lignes JSON le 1er novembre 2018. Les instructions ci-dessous détaillent les conséquences liées à ce changement et comment mettre à jour vos outils pour qu’ils gèrent ce nouveau format.
>

## <a name="what-changed"></a>Ce qui a changé

Azure Monitor offre une fonctionnalité qui vous permet d’envoyer des journaux d’activité des ressources et des journaux d’activité vers un compte de stockage Azure, un espace de noms Event Hubs ou un espace de travail Log Analytics dans Azure Monitor. Afin de résoudre un problème de performances du système, le **1er novembre 2018 à 00:00 UTC**, le format des données de journaux envoyées au stockage d’objets blob a changé. Si vous disposez des outils qui lisent les données dans le stockage d’objets blob, vous devez les mettre à jour pour qu’ils comprennent le nouveau format de données.

* Le jeudi 1er novembre 2018 à 00:00 UTC, le format des objets blob a été modifié en [lignes JSON](http://jsonlines.org/). Cela signifie que chaque enregistrement sera délimité par une nouvelle ligne, sans tableau d’enregistrements extérieurs ni virgule entre les enregistrements JSON.
* Le format des objets blob a été modifié pour tous les paramètres de diagnostics sur tous les abonnements en même temps. Le premier fichier PT1H.json émis le 1er novembre a utilisé ce nouveau format. Les noms des objets blob et du conteneur ne changent pas.
* La définition d’un paramètre de diagnostic avant le 1er novembre a continué d’émettre des données dans le format actuel jusqu’au 1er novembre.
* Cette modification s’est effectuée immédiatement sur toutes les régions cloud publiques. La modification n’aura pas lieu dans Microsoft Azure géré par 21Vianet, Azure Allemagne ou Azure Government pour le moment.
* Elle affectera les types de données suivants :
  * [Journaux d’activité des ressources Azure](./resource-logs.md#send-to-azure-storage) ([voir la liste des ressources ici](./resource-logs-schema.md))
  * [Métriques de ressources Azure en cours d’exportation par les paramètres de diagnostic](diagnostic-settings.md)
  * [Données des journaux d’activité Azure en cours d’exportation par les profils de journaux](./activity-log.md)
* Cette modification n’affecte pas :
  * Journaux de flux de réseau
  * Les journaux du service Azure ne sont pas encore disponibles via Azure Monitor (par exemple, les journaux de ressources Azure App Service ou les journaux d’analytique de stockage)
  * Routage des journaux de ressources et d’activité Azure vers d’autres destinations (Event Hubs, Log Analytics)

### <a name="how-to-see-if-you-are-impacted"></a>Comme savoir si vous êtes affecté

Vous n’êtes affecté par cette modification que si vous :
1. envoyez des données de journaux dans un compte de stockage Azure à l’aide d’un paramètre de diagnostic, et que vous
2. disposez d’outils qui dépendent de la structure JSON de ces journaux d’activité dans le stockage.
 
Pour identifier si vous disposez de paramètres de diagnostic qui envoient des données à un compte de stockage Azure, vous pouvez accéder à la section **Moniteur** du portail, puis cliquez sur **Paramètres de diagnostic** et identifiez toutes les ressources dont le **Statut de diagnostic** est **Activé** :

![Panneau des paramètres de diagnostics Azure Monitor](media/diagnostic-logs-append-blobs/portal-diag-settings.png)

Si le Statut de diagnostic est activé, vous disposez d’un paramètre de diagnostic actif pour cette ressource. Cliquez sur la ressource pour voir si des paramètres de diagnostic envoient des données à un compte de stockage :

![Compte de stockage actif](media/diagnostic-logs-append-blobs/portal-storage-enabled.png)

Si vous disposez de ressources qui envoient des données à un compte de stockage à l’aide de ces paramètres de diagnostic de ressources, le format des données de ce compte de stockage sera affecté par cette modification. À moins que vous ne disposiez d’outils personnalisés qui agissent hors de ces comptes de stockage, la modification du format ne vous affectera pas.

### <a name="details-of-the-format-change"></a>Détails de la modification du format

Le format actuel du fichier PT1H.json dans le stockage d’objets blob Azure utilise un tableau JSON d’enregistrements. Voici un exemple de fichier journal KeyVault :

```json
{
    "records": [
        {
            "time": "2016-01-05T01:32:01.2691226Z",
            "resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT",
            "operationName": "VaultGet",
            "operationVersion": "2015-06-01",
            "category": "AuditEvent",
            "resultType": "Success",
            "resultSignature": "OK",
            "resultDescription": "",
            "durationMs": "78",
            "callerIpAddress": "104.40.82.76",
            "correlationId": "",
            "identity": {
                "claim": {
                    "http://schemas.microsoft.com/identity/claims/objectidentifier": "d9da5048-2737-4770-bd64-XXXXXXXXXXXX",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "live.com#username@outlook.com",
                    "appid": "1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"
                }
            },
            "properties": {
                "clientInfo": "azure-resource-manager/2.0",
                "requestUri": "https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01",
                "id": "https://contosokeyvault.vault.azure.net/",
                "httpStatusCode": 200
            }
        },
        {
            "time": "2016-01-05T01:33:56.5264523Z",
            "resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT",
            "operationName": "VaultGet",
            "operationVersion": "2015-06-01",
            "category": "AuditEvent",
            "resultType": "Success",
            "resultSignature": "OK",
            "resultDescription": "",
            "durationMs": "83",
            "callerIpAddress": "104.40.82.76",
            "correlationId": "",
            "identity": {
                "claim": {
                    "http://schemas.microsoft.com/identity/claims/objectidentifier": "d9da5048-2737-4770-bd64-XXXXXXXXXXXX",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "live.com#username@outlook.com",
                    "appid": "1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"
                }
            },
            "properties": {
                "clientInfo": "azure-resource-manager/2.0",
                "requestUri": "https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01",
                "id": "https://contosokeyvault.vault.azure.net/",
                "httpStatusCode": 200
            }
        }
    ]
}
```

Le nouveau format utilise des [lignes JSON](http://jsonlines.org/), où chaque événement correspond à une ligne et le caractère de la nouvelle ligne indique un nouvel événement. Voici à quoi ressemble l’exemple ci-dessus dans le fichier PT1H.json après modification :

```json
{"time": "2016-01-05T01:32:01.2691226Z","resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT","operationName": "VaultGet","operationVersion": "2015-06-01","category": "AuditEvent","resultType": "Success","resultSignature": "OK","resultDescription": "","durationMs": "78","callerIpAddress": "104.40.82.76","correlationId": "","identity": {"claim": {"http://schemas.microsoft.com/identity/claims/objectidentifier": "d9da5048-2737-4770-bd64-XXXXXXXXXXXX","http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "live.com#username@outlook.com","appid": "1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"}},"properties": {"clientInfo": "azure-resource-manager/2.0","requestUri": "https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01","id": "https://contosokeyvault.vault.azure.net/","httpStatusCode": 200}}
{"time": "2016-01-05T01:33:56.5264523Z","resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT","operationName": "VaultGet","operationVersion": "2015-06-01","category": "AuditEvent","resultType": "Success","resultSignature": "OK","resultDescription": "","durationMs": "83","callerIpAddress": "104.40.82.76","correlationId": "","identity": {"claim": {"http://schemas.microsoft.com/identity/claims/objectidentifier": "d9da5048-2737-4770-bd64-XXXXXXXXXXXX","http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "live.com#username@outlook.com","appid": "1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"}},"properties": {"clientInfo": "azure-resource-manager/2.0","requestUri": "https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01","id": "https://contosokeyvault.vault.azure.net/","httpStatusCode": 200}}
```

Ce nouveau format permet à Azure Monitor d’envoyer (push) des fichiers journaux via des [objets blob d’ajout](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-append-blobs), plus efficaces pour des ajouts en continu de nouvelles données d’événement.

## <a name="how-to-update"></a>Comment effectuer la mise à jour

Vous n’avez qu’à effectuer les mises à jour si vous disposez d’outils qui ingèrent ces fichiers journaux pour un traitement ultérieur. Si vous utilisez un outil externe Log Analytics ou SIEM, nous vous recommandons d’[utiliser Event Hubs pour ingérer ces données à la place](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/). L’intégration Event Hubs est plus facile en termes de traitement de journaux d’activité de plusieurs services et d’ancrage d’emplacement dans un journal particulier.

Les outils personnalisés doivent être mis à jour pour gérer le format actuel et le format en lignes JSON décrit ci-dessus. Votre outil est ainsi garanti de ne pas s’arrêter lorsque les données commencent à s’afficher dans le nouveau format.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [l’archivage des journaux de ressources relatifs aux ressources dans un compte de stockage](./resource-logs.md#send-to-azure-storage)
* En savoir plus sur [l’archivage des données de journal d’activité dans un compte de stockage](./activity-log.md#legacy-collection-methods)
