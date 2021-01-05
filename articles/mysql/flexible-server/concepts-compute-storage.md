---
title: Options de calcul et de stockage - Azure Database pour MySQL - Serveur flexible
description: Cet article décrit les options de calcul et de stockage dans Azure Database pour MySQL - Serveur flexible.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 10/20/2020
ms.openlocfilehash: 36f31ee390a6a208b202698ec9bda59b644c9e30
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94534668"
---
# <a name="compute-and-storage-options-in-azure-database-for-mysql---flexible-server-preview"></a>Options de calcul et de stockage dans Azure Database pour MySQL - Serveur flexible (Préversion)

> [!IMPORTANT] 
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Vous pouvez créer un serveur flaxible Azure Database pour MySQL dans un des trois niveaux de calcul : Expansible, Usage général et À mémoire optimisée. Les niveaux de calcul sont différenciés par les références de machine virtuelle sous-jacentes utilisées : série B, série D et série E. Le choix du niveau de calcul et de la taille détermine la mémoire et les vCores disponibles sur le serveur. La même technologie de stockage est utilisée sur tous les niveaux de calcul. Toutes les ressources sont approvisionnées au niveau du serveur MySQL. Un serveur peut avoir une ou plusieurs bases de données.

| Ressource/Niveau | **Expansible** | **Usage général** | **Mémoire optimisée** |
|:---|:----------|:--------------------|:---------------------|
| Série de la machine virtuelle| Série B | Série Ddsv4 | Série Edsv4|
| vCores | 1, 2 | 2, 4, 8, 16, 32, 48, 64 | 2, 4, 8, 16, 32, 48, 64 |
| Mémoire par vCore | Variable | 4 Gio | 8 Gio * |
| Taille de stockage | 5 Gio à 16Tio | 5 Gio à 16Tio | 5 Gio à 16Tio |
| Période de rétention de sauvegarde de bases de données | 1 à 35 jours | 1 à 35 jours | 1 à 35 jours |

\* À l’exception de la référence E64ds_v4 (mémoire optimisée) qui dispose de 504 Go de mémoire

Pour choisir un niveau de calcul, utilisez le tableau suivant comme point de départ.

| Niveau de calcul | Charges de travail cibles |
|:-------------|:-----------------|
| Expansible | Idéal pour les charges de travail qui n’ont pas besoin de l’UC complète en permanence. |
| Usage général | La plupart des charges de travail professionnelles qui nécessitent une capacité de calcul et de mémoire équilibrée avec un débit d’E/S extensible. Il s’agit, par exemple, de serveurs destinés à l’hébergement d’applications web et mobiles, ainsi que d’autres applications d’entreprise.|
| Mémoire optimisée | Charges de travail de base de données haute performance qui nécessitent des performances en mémoire suffisantes pour un traitement plus rapide des transactions et une simultanéité plus élevée. Il s’agit, par exemple, de serveurs destinés au traitement de données en temps réel et à des applications transactionnelles ou analytiques haute performance.|

Une fois que vous avez créé un serveur, le niveau de calcul, la taille de calcul et la taille de stockage ont changé. La mise à l’échelle du calcul requiert un redémarrage et prend entre 60 et 120 secondes, alors que la mise à l’échelle du stockage ne nécessite pas de redémarrage. Vous pouvez également augmenter ou réduire la période de rétention de sauvegarde de manière indépendante. Pour plus d’informations, consultez la section [Ressources de mise à l’échelle](#scale-resources).

## <a name="compute-tiers-size-and-server-types"></a>Niveaux de calcul, tailles et types de serveurs

Les ressources de calcul peuvent être sélectionnées en fonction du niveau et de la taille. Cela détermine les vCores et la taille de la mémoire. Les vCores représentent le processeur logique du matériel sous-jacent.

Les spécifications détaillées des types de serveurs disponibles sont les suivantes :

| Taille de calcul         | vCores | Taille de la mémoire (Gio) | 
|----------------------|--------|-------------------|
| **Expansible**        |        |                   | 
| Standard_B1s         | 1      | 1                 |  
| Standard_B1ms        | 1      | 2                 | 
| Standard_B2s         | 2      | 4                 |  
| **Usage général**  |        |                   | 
| Standard_D2ds_v4     | 2      | 8                 |  
| Standard_D4ds_v4     | 4      | 16                | 
| Standard_D8ds_v4     | 8      | 32                | 
| Standard_D16ds_v4    | 16     | 64                | 
| Standard_D32ds_v4    | 32     | 128               |  
| Standard_D48ds_v4    | 48     | 192               |  
| Standard_D64ds_v4    | 64     | 256               | 
| **Mémoire optimisée** |        |                   |
| Standard_E2ds_v4     | 2      | 16                |
| Standard_E4ds_v4     | 4      | 32                |
| Standard_E8ds_v4     | 8      | 64                |
| Standard_E16ds_v4    | 16     | 128               |
| Standard_E32ds_v4    | 32     | 256               |
| Standard_E48ds_v4    | 48     | 384               |
| Standard_E64ds_v4    | 64     | 504               |

Pour plus d’informations sur la série de calcul disponible, reportez-vous à la documentation des machines virtuelles Azure pour [Expansible (série B)](../../virtual-machines/sizes-b-series-burstable.md), [Usage général (série Ddsv4)](../../virtual-machines/ddv4-ddsv4-series.md) et [À mémoire optimisée (série Edsv4)](../../virtual-machines/edv4-edsv4-series.md).

## <a name="storage"></a>Stockage

Le stockage que vous approvisionnez est la quantité de stockage disponible pour votre serveur flexible. Le stockage est utilisé pour les fichiers de base de données, les fichiers temporaires, les journaux d’activité de transaction, et les journaux d’activité du serveur MySQL. Dans tous les niveaux de calcul, le stockage minimal pris en charge est de 5 Gio et la valeur maximale est de 16 Tio. Le stockage est mis à l’échelle par incréments de 1 Gio et peut être mis à l’échelle après la création du serveur.

>[!NOTE]
> Le stockage peut seulement monter en puissance.

Vous pouvez surveiller votre consommation de stockage dans le portail Azure (avec Azure Monitor) à l’aide des métriques de limite de stockage, de pourcentage de stockage et de stockage utilisé. Reportez-vous à l’[article sur la surveillance](./concepts-monitoring.md) pour en savoir plus sur les métriques. 

### <a name="reaching-the-storage-limit"></a>Atteindre la limite de stockage

Lorsque le stockage consommé sur le serveur est proche de la limite configurée, le serveur est mis en mode lecture seule pour protéger les écritures perdues sur le serveur. Les serveurs avec moins de 100 Gio de stockage approvisionnés sont marqués en lecture seule si l’espace de stockage libre est inférieur à 5 % de la taille de stockage approvisionnée. Les serveurs avec plus de 100 Gio de stockage approvisionnés sont marqués en lecture seule lorsque l’espace de stockage libre est inférieur à 5 Gio.

Par exemple, si vous avez approvisionné 110 Gio de stockage et que l’utilisation réelle dépasse 105 Gio, le serveur est marqué en lecture seule. Sinon, si vous avez provisionné 5 Gio de stockage, le serveur est marqué en lecture seule quand le stockage disponible est inférieur à 256 Mo.

Pendant que le service tente de marquer le serveur en lecture seule, toutes les nouvelles demandes de transactions d’écriture sont bloquées et les transactions actives existantes continuent de s’exécuter. Une fois que le serveur est marqué en lecture seule, toutes les opérations d’écriture et validations de transaction ultérieures échouent. Les requêtes de lecture continueront de fonctionner sans interruption. 

Pour sortir le serveur du mode lecture seule, vous devez augmenter le stockage approvisionné sur le serveur. Pour ce faire, utilisez le portail Azure ou Azure CLI. Une fois augmenté, le serveur sera prêt à accepter de nouvelles transactions d’écriture.

Nous vous recommandons de configurer une alerte pour vous avertir quand votre serveur de stockage est proche du seuil afin d’éviter la mise en lecture seule. Reportez-vous à l’[article sur la surveillance](./concepts-monitoring.md) pour en savoir plus sur les métriques disponibles. 

Nous vous recommandons <!--turn on storage auto-grow or to--> de configurer une alerte pour vous avertir quand votre serveur de stockage est proche du seuil afin d’éviter la mise en lecture seule. Pour plus d’informations, consultez la documentation sur [comment configurer une alerte](how-to-alert-on-metric.md).

### <a name="storage-auto-grow"></a>Croissance automatique du stockage

La croissance automatique du stockage n’est pas encore disponible pour Azure Database pour MySQL - Serveur flexible.

## <a name="iops"></a>E/S par seconde
Le nombre minimal d’IOPS effectif est de 100 pour toutes les tailles de calcul et le nombre maximal d’IOPS effectif sont déterminés par les deux attributs suivants : 
- Calcul : le nombre maximal d’IOPS effectif peut être limité par le nombre maximal d’IOPS disponibles pour la taille de calcul sélectionnée.
- Stockage : dans tous les niveaux de calcul, les IOPS sont mises à l’échelle avec la taille de stockage approvisionnée selon un ratio de 3:1.

Vous pouvez mettre à l’échelle les IOPS efficaces en augmentant le stockage approvisionné ou en passant à une plus grande taille de calcul (si vos IOPS sont limitées par le calcul). Dans la préversion, le nombre maximal d’IOPS effectif pris en charge est de 20 000.

Pour en savoir plus sur le nombre maximal d’IOPS effectif par taille de calcul, l’utilisation de la combinaison du calcul et du stockage est illustrée ci-dessous : 

| Taille de calcul         | Nombre maximal d’IOPS effectif  | 
|----------------------|---------------------|
| **Expansible**        |                     |
| Standard_B1s         | 320                 |
| Standard_B1ms        | 640                 |
| Standard_B2s         | 1 280                | 
| **Usage général**  |                     |
| Standard_D2ds_v4     | 3200                |
| Standard_D4ds_v4     | 6 400                |
| Standard_D8ds_v4     | 12800               |
| Standard_D16ds_v4    | 20000               |
| Standard_D32ds_v4    | 20000               |
| Standard_D48ds_v4    | 20000               | 
| Standard_D64ds_v4    | 20000               | 
| **Mémoire optimisée** |                     | 
| Standard_E2ds_v4     | 3200                | 
| Standard_E4ds_v4     | 6 400                | 
| Standard_ E8ds_v4    | 12800               | 
| Standard_E16ds_v4   | 20000               | 
| Standard_E32ds_v4    | 20000               | 
| Standard_E48ds_v4    | 20000               | 
| Standard_E64ds_v4    | 20000               |  

Le nombre maximal d’IOPS effectif dépend du nombre maximal d’IOPS disponibles par taille de calcul. Consultez la formule ci-dessous et reportez-vous à la colonne *Débit du disque non mis en cache max. : IOPS/Mbits/s* dans la documentation [série B](../../virtual-machines/sizes-b-series-burstable.md), [série Ddsv4](../../virtual-machines/ddv4-ddsv4-series.md) et [série Edsv4](../../virtual-machines/edv4-edsv4-series.md).

**Nombre maximal d’IOPS effectif** = MINIMUM (*Débit du disque non mis en cache max. : IOPS/Mbits/s* de la taille de calcul et du stockage approvisionné en Gio x 3)

Vous pouvez surveiller votre consommation d’E/S dans le portail Azure (avec Azure Monitor) à l’aide de la métrique [IO percent](./concepts-monitoring.md). Si vous avez besoin de plus d’IOPS, vous devez savoir si vous êtes limité par la taille de calcul ou par le stockage approvisionné. Mettez à l’échelle le calcul ou le stockage de votre serveur en conséquence.

## <a name="backup"></a>Sauvegarde

Le service effectue automatiquement des sauvegardes de votre serveur. Vous pouvez sélectionner une période de conservation comprise entre 1 et 35 jours. Pour plus d’informations sur les sauvegardes, consultez cet [article sur les concepts de sauvegarde et de restauration](concepts-backup-restore.md).

## <a name="scale-resources"></a>Mettre les ressources à l’échelle

Après avoir créé votre serveur, vous pouvez modifier de manière indépendante le niveau de calcul, la taille de calcul (vCores et mémoire) et la quantité de stockage et la période de rétention de sauvegarde. La taille de calcul peut être augmentée ou réduite. La période de rétention de sauvegarde peut être augmentée ou diminuée et va de 1 à 35 jours. La taille de stockage ne peut être qu’augmentée. La mise à l’échelle des ressources peut être effectuée par le biais du portail ou d’Azure CLI.

> [!NOTE]
> La taille de stockage ne peut être qu’augmentée. Vous ne pouvez pas revenir à une taille de stockage inférieure après l’augmentation.

Lorsque vous modifiez le niveau de calcul ou la taille de calcul, le serveur redémarre pour que le nouveau type de serveur prenne effet. Pendant le moment durant lequel le système bascule vers le nouveau serveur, aucune nouvelle connexion ne peut être établie, et toutes les transactions non validées sont restaurées. Cette fenêtre varie, mais dans la plupart des cas elle dure entre 60 et 120 secondes. 

La mise à l’échelle du stockage et la modification de la période de rétention de sauvegarde sont des opérations en ligne et ne nécessitent pas de redémarrage du serveur.

## <a name="pricing"></a>Tarifs

Pour obtenir les dernières informations sur la tarification, veuillez consulter le service [Page de tarification](https://azure.microsoft.com/pricing/details/MySQL/). Pour voir le coût de la configuration souhaitée, le [Portail Azure](https://portal.azure.com/#create/Microsoft.MySQLServer/flexibleServers) affiche le coût mensuel dans l’onglet **Calcul et stockage** selon les options que vous avez sélectionnées. Si vous n’avez pas d’abonnement Azure, vous pouvez utiliser la calculatrice de prix Azure pour obtenir une estimation. Pour personnaliser les options, sur le site web [Calculatrice de prix d’Azure](https://azure.microsoft.com/pricing/calculator/), sélectionnez **Ajouter des éléments**, développez la catégorie **Bases de données**, sélectionnez **Azure Database pour MySQL**, puis **Serveur flexible** comme type de déploiement pour personnaliser les options.

Si vous souhaitez optimiser le coût du serveur, vous pouvez prendre en compte les conseils suivants :

- Réduisez le niveau de calcul ou la taille de calcul (vCores) si le calcul est sous-exploité.
- Envisagez de basculer vers un niveau de calcul par rafale si votre charge de travail n’a pas besoin de la capacité de calcul complète en continu à partir des niveaux Usage général et À mémoire optimisée.
- Arrêtez le serveur lorsqu’il n’est pas en cours d’utilisation.
- Réduisez la période de rétention des sauvegardes si une rétention de sauvegarde plus longue n’est pas nécessaire.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Créer un serveur MySQL dans le portail](quickstart-create-server-portal.md).
- En savoir plus sur les [limitations du service](concepts-limitations.md).