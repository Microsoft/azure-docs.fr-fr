---
title: Diagnostiquer et résoudre les problèmes de disponibilité des Kits de développement logiciel (SDK) Azure Cosmos dans les environnements multirégionaux
description: Découvrez tout ce qu’il y a à savoir sur le comportement de disponibilité du Kit de développement logiciel (SDK) Azure Cosmos lors de son utilisation dans des environnements multirégionaux.
author: ealsur
ms.service: cosmos-db
ms.date: 10/20/2020
ms.author: maquaran
ms.subservice: cosmosdb-sql
ms.topic: troubleshooting
ms.reviewer: sngun
ms.openlocfilehash: b1c2377ba26b4ca64f5028fb1a51ca4e64f6a67c
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097887"
---
# <a name="diagnose-and-troubleshoot-the-availability-of-azure-cosmos-sdks-in-multiregional-environments"></a>Diagnostiquer et résoudre les problèmes de disponibilité des Kits de développement logiciel (SDK) Azure Cosmos dans les environnements multirégionaux
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article décrit le comportement de la dernière version des Kits de développement logiciel (SDK) Azure Cosmos lorsque vous constatez un problème de connectivité avec une région particulière ou lorsqu’un basculement de région se produit.

Tous les Kits de développement logiciel (SDK) Azure Cosmos vous donnent la possibilité de personnaliser les préférences régionales. Les propriétés suivantes sont utilisées dans différents Kits de développement logiciel (SDK) :

* La propriété [ConnectionPolicy.PreferredLocations](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.preferredlocations) dans le Kit de développement logiciel (SDK) .NET v2.
* Les propriétés [CosmosClientOptions.ApplicationRegion](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationregion) ou [CosmosClientOptions.ApplicationPreferredRegions](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions) dans le Kit de développement logiciel (SDK) .NET v3.
* La méthode [CosmosClientBuilder.preferredRegions](/java/api/com.azure.cosmos.cosmosclientbuilder.preferredregions) dans le Kit de développement logiciel (SDK) Java v4.
* Le paramètre [CosmosClient.preferred_locations](/python/api/azure-cosmos/azure.cosmos.cosmos_client.cosmosclient) dans le Kit de développement logiciel (SDK) Python.
* Le paramètre [CosmosClientOptions.ConnectionPolicy.preferredLocations](/javascript/api/@azure/cosmos/connectionpolicy#preferredlocations) dans le Kit de développement logiciel (SDK) JS.

Lorsque vous définissez la préférence régionale, le client se connecte à une région comme indiqué dans le tableau suivant :

|Type de compte |Lectures |Écritures |
|------------------------|--|--|
| Région d’écriture unique | Région recommandée | Région primaire  |
| Régions d’écriture multiples | Région recommandée | Région recommandée  |

Si vous **ne définissez pas de région préférée** , le client du Kit de développement logiciel (SDK) se connecte par défaut à la région primaire :

|Type de compte |Lectures |Écritures |
|------------------------|--|--|
| Région d’écriture unique | Région primaire | Région primaire |
| Régions d’écriture multiples | Région primaire  | Région primaire  |

> [!NOTE]
> La région primaire fait référence à la première région de la [liste des régions du compte Azure Cosmos](distribute-data-globally.md)

Dans des circonstances normales, le client du Kit de développement logiciel (SDK) se connecte à la région préférée (si une préférence régionale est définie) ou à la région primaire (si aucune préférence n’est définie) et les opérations sont limitées à cette région, sauf si l’un des scénarios ci-dessous se produit.

Dans les cas suivants, le client qui utilise le Kit de développement logiciel (SDK) Azure Cosmos expose les journaux et inclut les informations relatives aux nouvelles tentatives dans le cadre des **informations de diagnostic de l’opération** :

* La propriété *RequestDiagnosticsString* incluse dans les réponses dans le Kit de développement logiciel (SDK) .NET V2.
* La propriété *Diagnostics* incluse dans les réponses et les exceptions dans le Kit de développement logiciel (SDK) .NET V3.
* La méthode *getDiagnostics ()* incluse dans réponses et les exceptions dans le Kit de développement logiciel (SDK) Java V4.

Lors de la détermination de la région suivante par ordre de préférence, le client du Kit de développement logiciel (SDK) utilise la liste des régions du compte, en hiérarchisant les régions préférées (le cas échéant).

Pour obtenir des informations complètes sur les garanties de contrat SLA au cours de ces événements, consultez la section [Contrats SLA pour la disponibilité](high-availability.md#slas-for-availability).

## <a name="removing-a-region-from-the-account"></a><a id="remove-region"></a>Suppression d’une région du compte

Lorsque vous supprimez une région d’un compte Azure Cosmos, tout client SDK qui utilise activement le compte détectera la suppression de la région grâce à un code de réponse principale. Le client marque ensuite le point de terminaison régional comme non disponible. Le client réitère l’opération en cours et toutes les opérations ultérieures sont acheminées de manière permanente vers la région suivante par ordre de préférence.

## <a name="adding-a-region-to-an-account"></a>Ajout d’une région à un compte

Toutes les cinq minutes, le client SDK Azure Cosmos lit la configuration du compte et actualise les régions dont il a connaissance.

Si vous supprimez une région et la rajoutez ultérieurement au compte, si la région ajoutée a un ordre de préférences régionales plus élevé dans le Kit de développement logiciel (SDK) de configuration que la région sélectionnée, le Kit de développement logiciel (SDK) reviendra à l’utilisation de cette région de façon permanente. Une fois la région ajoutée détectée, toutes les requêtes ultérieures sont dirigées vers elle.

Si vous configurez le client pour qu’il se connecte de préférence à une région que le compte Azure Cosmos ne possède pas, la région par défaut est ignorée. Si vous ajoutez cette région ultérieurement, le client la détecte et bascule définitivement vers cette région.

## <a name="fail-over-the-write-region-in-a-single-write-region-account"></a><a id="manual-failover-single-region"></a>Basculement de la région d’écriture vers un compte à une seule région d’écriture

Si vous initiez un basculement de la région d’écriture actuelle, la demande d’écriture suivante échouera avec une réponse principale connue. Lorsque cette réponse est détectée, le client interroge le compte pour connaître la nouvelle région d’écriture, retente l’opération en cours et achemine définitivement toutes les futures opérations d’écriture vers la nouvelle région.

## <a name="regional-outage"></a>Panne régionale

Si le compte dispose d’une seule région d’écriture et que la panne régionale se produit pendant une opération d’écriture, le comportement est similaire à un [basculement manuel](#manual-failover-single-region). Pour les demandes de lecture ou les comptes à plusieurs régions d’écriture, le comportement est similaire à la [suppression d’une région](#remove-region).

## <a name="session-consistency-guarantees"></a>Garanties de cohérence de session

Lorsque vous utilisez la [cohérence de session](consistency-levels.md#guarantees-associated-with-consistency-levels), le client doit garantir qu’il peut lire ses propres écritures. Dans les comptes à une seule région d’écriture où la préférence pour la région de lecture est différente de la région d’écriture, il peut arriver que l’utilisateur génère une écriture et, lorsqu’il effectue une lecture à partir d’une région locale, que celle-ci n’ait pas encore reçu la réplication des données (contrainte de la vitesse de la lumière). Dans ce cas, le Kit de développement logiciel (SDK) détecte l’échec spécifique de l’opération de lecture et tente à nouveau la lecture sur la région primaire pour garantir la cohérence de la session.

## <a name="transient-connectivity-issues-on-tcp-protocol"></a>Problèmes de connectivité temporaires sur le protocole TCP

Dans les scénarios où le client SDK Azure Cosmos est configuré pour utiliser le protocole TCP, pour une requête donnée, il peut y avoir des situations où les conditions réseau empêchent temporairement la communication avec un point de terminaison particulier. Ces conditions réseau temporaires peuvent se manifester sous la forme d’expirations du délais TCP. Le client retentera la requête localement sur le même point de terminaison pendant quelques secondes.

Si l’utilisateur a configuré une liste de régions par défaut avec plusieurs régions, que le compte Azure Cosmos dispose de plusieurs régions d’écriture ou d’une seule région d’écriture et que l’opération est une demande de lecture, le client retente cette opération unique dans la région suivante de la liste de préférences.

## <a name="next-steps"></a>Étapes suivantes

* Examinez les [Contrats SLA pour la disponibilité](high-availability.md#slas-for-availability).
* Utiliser la version la plus récente du [Kit de développement logiciel (SDK) .NET](sql-api-sdk-dotnet-standard.md)
* Utiliser la version la plus récente du [Kit de développement logiciel (SDK) Java](sql-api-sdk-java-v4.md)
* Utiliser la version la plus récente du [Kit de développement logiciel (SDK) Python](sql-api-sdk-python.md)
* Utiliser la version la plus récente du [Kit de développement logiciel (SDK) Node](sql-api-sdk-node.md)
