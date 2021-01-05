---
title: Options de calcul et de stockage - Azure Database pour PostgreSQL - Serveur flexible
description: Cet article décrit les options de calcul et de stockage dans le serveur flexible Azure Database pour PostgreSQL.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: ca60c44d1e167367e2c138af1e7bfd4ba1a69417
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91710071"
---
# <a name="compute-and-storage-options-in-azure-database-for-postgresql---flexible-server"></a>Options de calcul et de stockage dans le serveur flexible Azure Database pour PostgreSQL

> [!IMPORTANT]
> Le serveur flexible Azure Database pour PostgreSQL est en préversion

Vous pouvez créer un serveur Azure Database pour PostgreSQL dans un des trois différents niveaux tarifaires : Expansible, Usage général et À mémoire optimisée. Les niveaux tarifaires diffèrent par la quantité de calcul dans vCores qui peut être configurée, la mémoire par vCore et la technologie de stockage utilisée pour stocker les données. Toutes les ressources sont approvisionnées au niveau du serveur PostgreSQL. Un serveur peut avoir une ou plusieurs bases de données.

| Ressource/Niveau | **Expansible** | **Usage général** | **Mémoire optimisée** |
|:---|:----------|:--------------------|:---------------------|
| vCores | 1, 2 | 2, 4, 8, 16, 32, 48, 64 | 2, 4, 8, 16, 32, 48, 64 |
| Mémoire par vCore | Variable | 4 Go | 6,75 à 8 Go |
| Taille de stockage | 32 Go à 16 To | 32 Go à 16 To | 32 Go à 16 To |
| Période de rétention de sauvegarde de bases de données | 7 à 35 jours | 7 à 35 jours | 7 à 35 jours |

Pour choisir un niveau tarifaire, utilisez le tableau suivant comme point de départ.

| Niveau tarifaire | Charges de travail cibles |
|:-------------|:-----------------|
| Expansible | Idéal pour les charges de travail qui n’ont pas besoin de l’UC complète en permanence. |
| Usage général | La plupart des charges de travail professionnelles qui nécessitent une capacité de calcul et de mémoire équilibrée avec un débit d’E/S extensible. Il s’agit, par exemple, de serveurs destinés à l’hébergement d’applications web et mobiles, ainsi que d’autres applications d’entreprise.|
| Mémoire optimisée | Charges de travail de base de données haute performance qui nécessitent des performances en mémoire suffisantes pour un traitement plus rapide des transactions et une simultanéité plus élevée. Il s’agit, par exemple, de serveurs destinés au traitement de données en temps réel et à des applications transactionnelles ou analytiques haute performance.|

Après avoir créé un serveur, le niveau de calcul, le nombre de vCores et la taille de stockage peuvent être augmentés ou diminués en quelques secondes. Vous pouvez également augmenter ou réduire la période de rétention de sauvegarde de manière indépendante. Pour plus d’informations, consultez la section [Ressources de mise à l’échelle](#scale-resources).

## <a name="compute-tiers-vcores-and-server-types"></a>Niveaux de calcul, vCores et types de serveurs

Les ressources de calcul peuvent être sélectionnées en fonction du niveau, des vCores et de la taille de la mémoire. Les vCores représentent le processeur logique du matériel sous-jacent.

Les spécifications détaillées des types de serveurs disponibles sont les suivantes :

| Nom de la référence SKU             | vCores | Taille de la mémoire | Nombre maximal d’E/S par seconde pris en charge | Bande passante d’E/S maximale prise en charge |
|----------------------|--------|-------------|--------------------|-----------------------------|
| **Expansible**        |        |             |                    |                             |
| B1ms                 | 1      | 2 Gio       | 640                | 15 Mio/s                  |
| B2s                  | 2      | 4 Gio       | 1 280               | 15 Mio/s                  |
| **Usage général**  |        |             |                    |                             |
| D2s_v3               | 2      | 8 Gio       | 3200               | 48 Mio/s                  |
| D4s_v3               | 4      | 16 Gio      | 6 400               | 96 Mio/s                  |
| D8s_v3               | 8      | 32 Gio      | 12800              | 192 Mio/s                 |
| D16s_v3              | 16     | 64 Gio      | 18000              | 384 Mio/s                 |
| D32s_v3              | 32     | 128 Go     | 18000              | 750 Mio/s                 |
| D48s_v3              | 48     | 192 Gio     | 18000              | 750 Mio/s                 |
| D64s_v3              | 64     | 256 Gio     | 18000              | 750 Mio/s                 |
| **Mémoire optimisée** |        |             |                    |                             |
| E2s_v3               | 2      | 16 Gio      | 3200               | 48 Mio/s                  |
| E4s_v3               | 4      | 32 Gio      | 6 400               | 96 Mio/s                  |
| E8s_v3               | 8      | 64 Gio      | 12800              | 192 Mio/s                 |
| E16s_v3              | 16     | 128 Go     | 18000              | 384 Mio/s                 |
| E32s_v3              | 32     | 256 Gio     | 18000              | 750 Mio/s                 |
| E48s_v3              | 48     | 384 Gio     | 18000              | 750 Mio/s                 |
| E64s_v3              | 64     | 432 Gio     | 18000              | 750 Mio/s                 |

## <a name="storage"></a>Stockage

Le stockage que vous approvisionnez est la quantité de stockage disponible pour votre serveur Azure Database pour PostgreSQL. Le stockage est utilisé pour les fichiers de base de données, les fichiers temporaires, les journaux d’activité de transaction, et les journaux d’activité du serveur PostgreSQL. La quantité totale de stockage que vous approvisionnez définit également la capacité d’E/S disponible sur votre serveur.

Le stockage est disponible dans les tailles fixes suivantes :

| Taille du disque | E/S par seconde |
|:---|:---|
| 32 Gio | Alloué 120, jusqu’à 3 500 |
| 64 Gio | Alloué 240, jusqu’à 3 500 |
| 128 Go | Alloué 500, jusqu’à 3 500 |
| 256 Gio | Alloué 1 100, jusqu’à 3 500 |
| 512 Go | Alloué 2 300, jusqu’à 3 500 |
| 1 Tio | 5 000 |
| 2 Tio | 7 500 |
| 4 Tio | 7 500 |
| 8 Tio | 16 000 |
| 16 Tio | 18 000 |

Notez que les E/S par seconde sont également contraintes par le type de machine virtuelle. Même si vous pouvez sélectionner n’importe quelle taille de stockage quel que soit le type de serveur, vous ne pourrez peut-être pas utiliser toutes les E/S par seconde que le stockage fournit, en particulier lorsque vous choisissez un serveur avec un petit nombre de vCores.

Vous pouvez ajouter une capacité de stockage supplémentaire pendant et après la création du serveur.

>[!NOTE]
> Le stockage peut seulement monter en puissance.

Vous pouvez surveiller votre consommation d’E/S dans le Portail Azure ou à l’aide des commandes Azure CLI. Les métriques pertinentes à surveiller sont [la limite de stockage, le pourcentage de stockage, le stockage utilisé et le pourcentage d’E/S](concepts-monitoring.md).

### <a name="maximum-iops-for-your-configuration"></a>Nombre maximal d’E/S par seconde pour votre configuration

|Nom de la référence SKU            |Taille de stockage en Gio                       |32 |64 |128 |256 |512  |1 024|2 048|4 096|8 192 |16 384|
|--------------------|------------------------------------------|---|---|----|----|-----|-----|-----|-----|------|------|
|                    |Nombre maximal d’E/S par seconde                              |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|**Expansible**       |                                          |   |   |    |    |     |     |     |     |      |      |
|B1ms                |640 E/S par seconde                                  |120|240|500 |640*|640* |640* |640* |640* |640*  |640*  |
|B2s                 |1280 E/S par seconde                                 |120|240|500 |1100|1280*|1280*|1280*|1280*|1280* |1280* |
|**Usage général** |                                          |   |   |    |    |     |     |     |     |      |      |
|D2s_v3              |3 200 E/S par seconde                                 |120|240|500 |1100|2300 |3200*|3200*|3200*|3200* |3200* |
|D4s_v3              |6 400 E/S par seconde                                |120|240|500 |1100|2300 |5 000 |6 400*|6 400*|6 400* |6 400* |
|D8s_v3              |12 800 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |12 800*|12 800*|
|D16s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|D32s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|D48s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|D64s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|**Mémoire optimisée**|                                          |   |   |    |    |     |     |     |     |      |      |
|E2s_v3              |3 200 E/S par seconde                                 |120|240|500 |1100|2300 |3200*|3200*|3200*|3200* |3200* |
|E4s_v3              |6 400 E/S par seconde                                |120|240|500 |1100|2300 |5 000 |6 400*|6 400*|6 400* |6 400* |
|E8s_v3              |12 800 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |12 800*|12 800*|
|E16s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|E32s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|E48s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |
|E64s_v3             |18 000 E/S par seconde                               |120|240|500 |1100|2300 |5 000 |7500 |7500 |16000 |18000 |

Lorsqu’elles sont marquées d’un \*, les E/S par seconde sont limitées par le type de machine virtuelle que vous avez sélectionné. Sinon, les E/S par seconde sont limitées par la taille de stockage sélectionnée.

### <a name="maximum-io-bandwidth-mibsec-for-your-configuration"></a>Bande passante d’E/S maximale (Mio/s) pour votre configuration

|Nom de la référence SKU            |Taille de stockage, Gio                             |32 |64 |128 |256 |512  |1 024|2 048|4 096|8 192 |16 384|
|--------------------|----------------------------------------------|---|---|----|----|-----|-----|-----|-----|------|------|
|                    |**Bande passante de stockage, Mio/s**                |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|**Expansible**       |                                              |   |   |    |    |     |     |     |     |      |      |
|B1ms                |10 Mio/s                                    |10*|10*|10* |10* |10*  |10*  |10*  |10*  |10*   |10*   |
|B2s                 |15 Mio/s                                    |15*|15*|15* |15* |15*  |15*  |15*  |15*  |15*   |15*   |
|**Usage général** |                                              |   |   |    |    |     |     |     |     |      |      |
|D2s_v3              |48 Mio/s                                    |25 |48*|48* |48* |48*  |48*  |48*  |48*  |48*   |48*   |
|D4s_v3              |96 Mio/s                                    |25 |50 |96* |96* |96*  |96*  |96*  |96*  |96*   |96*   |
|D8s_v3              |192 Mio/s                                   |25 |50 |100 |125 |150  |192* |192* |192* |192*  |192*  |
|D16s_v3             |384 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |384*  |384*  |
|D32s_v3             |750 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|D48s_v3             |750 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|D64s_v3             |750 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|**Mémoire optimisée**|                                              |   |   |    |    |     |     |     |     |      |      |
|E2s_v3              |48 Mio/s                                    |25 |48*|48* |48* |48*  |48*  |48*  |48*  |48*   |48*   |
|E4s_v3              |96 Mio/s                                    |25 |50 |96* |96* |96*  |96*  |96*  |96*  |96*   |96*   |
|E8s_v3              |192 Mio/s                                   |25 |50 |100 |125 |150  |192* |192* |192* |192*  |192*  |
|E16s_v3             |384 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |384*  |384*  |
|E32s_v3             |750 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|E48s_v3             |750 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |
|E64s_v3             |750 Mio/s                                   |25 |50 |100 |125 |150  |200  |250  |250  |500   |750   |

Lorsqu’elles sont marquées d’un \*, la bande passante d’E/S est limitée par le type de machine virtuelle que vous avez sélectionné. Sinon, la bande passante d’E/S est limitée par la taille de stockage sélectionnée.

### <a name="reaching-the-storage-limit"></a>Atteindre la limite de stockage

Lorsque vous atteignez la limite de stockage, le serveur commence à renvoyer des erreurs et empêchera toute autre modification. Cela peut également entraîner des problèmes avec d’autres activités opérationnelles, tels que les sauvegardes et l’archivage du journal WAL.

Nous vous recommandons de surveiller activement l’espace disque en cours d’utilisation et d’augmenter la taille du disque avant d’atteindre un stockage insuffisant. Vous pouvez configurer une alerte pour vous avertir lorsque le stockage de votre serveur devient insuffisant, ce qui vous permet d’éviter tout problème de manque d’espace disque. Pour plus d’informations, consultez la documentation sur [comment configurer une alerte](howto-alert-on-metrics.md).

### <a name="storage-auto-grow"></a>Croissance automatique du stockage

La croissance automatique du stockage n’est pas encore disponible pour le serveur flexible.

## <a name="backup"></a>Sauvegarde

Le service effectue automatiquement des sauvegardes de votre serveur. Vous pouvez sélectionner une période de conservation comprise entre 7 et 35 jours. En savoir plus sur les sauvegardes dans l’[article sur les concepts](concepts-backup-restore.md).

## <a name="scale-resources"></a>Mettre les ressources à l’échelle

Après avoir créé votre serveur, vous pouvez modifier de manière indépendante les vCores, le niveau de calcul, la quantité de stockage et la période de rétention de sauvegarde. Le nombre de vCores peut être augmenté ou diminué. La période de rétention de sauvegarde peut être augmentée ou diminuée et va de 7 à 35 jours. La taille de stockage ne peut être qu’augmentée. La mise à l’échelle des ressources peut être effectuée par le biais du portail ou d’Azure CLI.

> [!NOTE]
> La taille de stockage ne peut être qu’augmentée. Vous ne pouvez pas revenir à une taille de stockage inférieure après l’augmentation.

Lorsque vous modifiez le nombre de vCores ou le niveau de calcul, le serveur redémarre pour que le nouveau type de serveur prenne effet. Pendant le moment durant lequel le système bascule vers le nouveau serveur, aucune nouvelle connexion ne peut être établie, et toutes les transactions non validées sont restaurées. Cette fenêtre varie, mais dans la plupart des cas elle dure moins d’une minute. La mise à l’échelle du stockage fonctionne de la même manière et nécessite un redémarrage rapide.

La modification de la période de rétention des sauvegardes s’effectue en ligne.

## <a name="pricing"></a>Tarifs

Pour obtenir les dernières informations sur la tarification, veuillez consulter le service [Page de tarification](https://azure.microsoft.com/pricing/details/PostgreSQL/). Pour voir le coût de la configuration souhaitée, le [Portail Azure](https://portal.azure.com/#create/Microsoft.PostgreSQLServer) affiche le coût mensuel dans l’onglet **Niveau tarifaire** selon les options que vous avez sélectionnées. Si vous n’avez pas d’abonnement Azure, vous pouvez utiliser la calculatrice de prix Azure pour obtenir une estimation. Pour personnaliser les options, sur le site web [Calculatrice de prix d’Azure](https://azure.microsoft.com/pricing/calculator/), sélectionnez **Ajouter des éléments**, développez la catégorie **Bases de données**, puis choisissez **Azure Database pour PostgreSQL**.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Créer un serveur PostgreSQL dans le portail](how-to-manage-server-portal.md).
- En savoir plus sur les [limites de service](concepts-limits.md).
