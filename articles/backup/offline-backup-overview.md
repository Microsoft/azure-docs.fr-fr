---
title: Vue d’ensemble de la sauvegarde hors connexion
description: En savoir plus sur les composants de la sauvegarde hors connexion. Ils incluent la sauvegarde hors connexion basée sur Azure Data Box et la sauvegarde hors connexion basée sur le service Azure Import/Export.
ms.topic: conceptual
ms.date: 1/28/2020
ms.custom: references_regions
ms.openlocfilehash: c5e0f4e722e2dd15b7277a484af2a101844344e6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86503623"
---
# <a name="overview-of-offline-backup"></a>Vue d’ensemble de la sauvegarde hors connexion

Cet article donne une vue d’ensemble de la sauvegarde hors connexion.

Les sauvegardes complètes initiales sur Azure transfèrent généralement de grandes quantités de données en ligne et requièrent davantage de bande passante réseau, en comparaison avec les sauvegardes suivantes qui transfèrent uniquement les modifications incrémentielles. Dans certaines zones géographiques, les bureaux ou centres de donnés distants ne disposent pas toujours d’une bande passante réseau suffisante. Pour cette raison, ces sauvegardes initiales prennent plusieurs jours. Pendant ce temps, les sauvegardes utilisent en permanence le même réseau que celui approvisionné pour les applications qui s’exécutent dans le centre de données local.

Sauvegarde Azure prend en charge la sauvegarde hors connexion, laquelle transfère les données de sauvegarde initiales hors connexion, sans utiliser la bande passante réseau. Le service fournit un mécanisme permettant de copier des données de sauvegarde sur des appareils de stockage physiques. Les appareils sont ensuite envoyés à un centre de données Azure voisin et chargés dans un coffre Recovery Services. Ce processus garantit un transfert robuste des données de sauvegarde sans utiliser de bande passante réseau.

## <a name="offline-backup-options"></a>Options de sauvegarde hors connexion

La sauvegarde hors connexion est proposée dans deux modes, selon le propriétaire des appareils de stockage :

- Sauvegarde hors connexion basée sur Azure Data Box (préversion)
- Sauvegarde hors connexion basée sur le service Azure Import/Export

## <a name="offline-backup-based-on-azure-data-box-preview"></a>Sauvegarde hors connexion basée sur Azure Data Box (préversion)

Ce mode est actuellement pris en charge avec l’agent Microsoft Azure Recovery Services (MARS), en préversion. Cette option a recours à [Azure Data Box](https://azure.microsoft.com/services/databox/) pour l’envoi des appliances de transfert à votre centre de données ou bureau distant. Ces appliances appartiennent à Microsoft et elles sont sécurisées, inviolables et dotées de connecteurs USB. Les données de sauvegarde sont écrites directement sur ces appareils. Cette option réduit les efforts nécessaires pour trouver vos propres disques et connecteurs compatibles avec Azure ou pour approvisionner un stockage temporaire comme emplacement intermédiaire. Microsoft gère également la logistique du transfert de bout en bout, que vous pouvez suivre par le biais du Portail Azure.

Une architecture décrivant le déplacement des données de sauvegarde avec cette option est illustrée ici.

![Architecture Data Box de Sauvegarde Azure](./media/offline-backup-overview/azure-backup-databox-architecture.png)

Voici un résumé de l’architecture :

1. Sauvegarde Azure copie directement les données de sauvegarde sur ces appareils préconfigurés.
2. Vous pouvez ensuite renvoyer ces appareils à un centre de données Azure.
3. Azure Data Box copie les données sur un compte de stockage appartenant au client.
4. Sauvegarde Azure copie automatiquement les données de sauvegarde depuis le compte de stockage vers le coffre Recovery Services désigné. Des sauvegardes en ligne incrémentielles sont planifiées.

Pour utiliser la sauvegarde hors connexion basée sur Azure Data Box, consultez [Sauvegarde hors connexion avec Azure Data Box](offline-backup-azure-data-box.md).

## <a name="offline-backup-based-on-the-azure-importexport-service"></a>Sauvegarde hors connexion basée sur le service Azure Import/Export

Cette option est prise en charge par le serveur de sauvegarde Microsoft Azure (MABS), System Center Data Protection Manager (DPM) DPM-A et l’agent MARS. Elle utilise le [service Azure Import/Export](../storage/common/storage-import-export-service.md). Vous pouvez transférer des données de sauvegarde initiales vers Azure à l’aide de vos propres disques et connecteurs compatibles avec Azure. Cette approche nécessite l’approvisionnement d’un stockage temporaire appelé « emplacement intermédiaire » et l’emploi d’utilitaires prédéfinis pour formater et copier les données de sauvegarde sur les disques appartenant au client.

Une architecture décrivant le déplacement des données de sauvegarde avec cette option est illustrée ici.

![Architecture du service Import/Export de Sauvegarde Azure](./media/offline-backup-overview/azure-backup-import-export.png)

Voici un résumé de l’architecture :

1. Au lieu d’envoyer les données de sauvegarde sur le réseau, Sauvegarde Azure les écrit dans un emplacement intermédiaire.
2. Les données situées à l’emplacement intermédiaire sont écrites sur un ou plusieurs disques SATA à l’aide d’un utilitaire personnalisé.
3. Dans le cadre du travail préparatoire, l’utilitaire crée une tâche d’importation Azure. Les disques SATA sont envoyés au centre de données Azure le plus proche et référencent la tâche d’importation pour connecter les activités.
4. Dans le centre de données Azure, les données stockées sur les disques sont copiées vers un compte de stockage Azure.
5. Sauvegarde Azure copie les données de sauvegarde à partir du compte de stockage vers le coffre Recovery Services. Des sauvegardes incrémentielles sont planifiées.

Pour utiliser la sauvegarde hors connexion basée sur le service Azure Import/Export avec l’agent MARS, consultez [Workflow de la sauvegarde hors connexion dans Sauvegarde Azure](./backup-azure-backup-import-export.md).

Pour utiliser la même option avec MABS ou DPM-A, consultez [Workflow de la sauvegarde hors connexion pour DPM et le serveur de sauvegarde Azure](./backup-azure-backup-server-import-export.md).

## <a name="offline-backup-support-summary"></a>Résumé de la prise en charge de la sauvegarde hors connexion

Le tableau suivant compare les deux options disponibles pour vous aider à faire le bon choix selon votre scénario.

| **Considération**                                            | **Sauvegarde hors connexion basée sur Azure Data Box**                     | **Sauvegarde hors connexion basée sur le service Azure Import/Export**                |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Modèles de déploiement Sauvegarde Azure                              | Agent MARS (préversion)                                              | Agent MARS, MABS, DPM-A                                           |
| Nombre maximal de données de sauvegarde par serveur (MARS) ou par groupe de protection (MABS, DPM-A) | [Azure Data Box Disk](../databox/data-box-disk-overview.md) : 7,2 To <br> [Azure Data Box](../databox/data-box-overview.md) : 80 To       | 80 To (jusqu’à 10 disques de 8 To chacun)                          |
| Sécurité (données, appareil et service)                           | [Données](../databox/data-box-security.md#data-box-data-protection) : chiffrement AES 256 bits <br> [Appareil](../databox/data-box-security.md#data-box-device-protection) : boîtier robuste, interface propriétaire basées sur des informations d’identification pour copier les données <br> [Service](../databox/data-box-security.md#data-box-service-protection) : protégé par les fonctionnalités de sécurité Azure | Données : chiffrement BitLocker                                 |
| Approvisionnement d’un emplacement intermédiaire temporaire                     | Non requis                                                | Supérieur ou égal à la taille estimée des données de sauvegarde        |
| Régions prises en charge                                           | [Régions des disques Azure Data Box](../databox/data-box-disk-overview.md#region-availability) <br> [Régions Azure Data Box](../databox/data-box-disk-overview.md#region-availability) | [Régions du service Azure Import/Export](../storage/common/storage-import-export-service.md#region-availability) |
| Expédition internationale                                     | Non pris en charge  <br>    L’adresse source et le centre de données Azure de destination doivent se trouver dans le même pays/la même région.* | Prise en charge                                                    |
| Logistique du transfert (livraison, transport, enlèvement)           | Entièrement gérée par Microsoft                                     | Managée par le client                                            |
| Tarifs                                                      | [Tarifs Azure Data Box](https://azure.microsoft.com/pricing/details/databox/) <br> [Tarifs des disques Azure Data Box](https://azure.microsoft.com/pricing/details/databox/disk/) | [Tarifs du service Azure Import/Export](https://azure.microsoft.com/pricing/details/storage-import-export/) |

* Si aucun centre de données Azure n’existe dans votre pays/région, vous devez expédier vos disques à un centre de données Azure situé dans un autre pays/une autre région.

## <a name="next-steps"></a>Étapes suivantes

- [Sauvegarde hors connexion Sauvegarde Microsoft Azure avec Azure Data Box](offline-backup-azure-data-box.md#backup-data-size-and-supported-data-box-skus)
- [Workflow de la sauvegarde hors connexion dans Sauvegarde Azure](backup-azure-backup-import-export.md)
- [Workflow de la sauvegarde hors connexion pour DPM et le serveur de sauvegarde Azure](backup-azure-backup-server-import-export.md)
