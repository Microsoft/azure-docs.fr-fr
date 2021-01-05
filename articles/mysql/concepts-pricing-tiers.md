---
title: Niveaux tarifaires - Azure Database pour MySQL
description: Découvrez les différents niveaux tarifaires d’Azure Database pour MySQL, ce qui englobe les générations de calcul, les types de stockage, la taille de stockage, les vCores, la mémoire et les périodes de conservation des sauvegardes.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: c70e4a097a56b76089a26510bcf33b4c7c24c266
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018713"
---
# <a name="azure-database-for-mysql-pricing-tiers"></a>Niveaux tarifaires Azure Database pour MySQL

Vous pouvez créer un serveur Azure Database pour MySQL dans un des trois différents niveaux tarifaires : De base, Usage général et À mémoire optimisée. Les niveaux tarifaires diffèrent par la quantité de calcul dans vCores qui peut être configurée, la mémoire par vCore et la technologie de stockage utilisée pour stocker les données. Toutes les ressources sont approvisionnées au niveau du serveur MySQL. Un serveur peut avoir une ou plusieurs bases de données.

| Attribut   | **De base** | **Usage général** | **Mémoire optimisée** |
|:---|:----------|:--------------------|:---------------------|
| Génération de calcul | Gen 4, Gen 5 | Gen 4, Gen 5 | Gen 5 |
| vCores | 1, 2 | 2, 4, 8, 16, 32, 64 |2, 4, 8, 16, 32 |
| Mémoire par vCore | 2 Go | 5 Go | 10 Go |
| Taille de stockage | 5 Go à 1 To | 5 Go à 16 To | 5 Go à 16 To |
| Période de rétention de sauvegarde de bases de données | 7 à 35 jours | 7 à 35 jours | 7 à 35 jours |

Pour choisir un niveau tarifaire, utilisez le tableau suivant comme point de départ.

| Niveau tarifaire | Charges de travail cibles |
|:-------------|:-----------------|
| De base | Charges de travail qui nécessitent des performances légères en termes de calcul et d’E/S. Il s’agit, par exemple, de serveurs utilisés pour le développement ou le test, ou pour des applications à petite échelle rarement utilisées. |
| Usage général | La plupart des charges de travail professionnelles qui nécessitent une capacité de calcul et de mémoire équilibrée avec un débit d’E/S extensible. Il s’agit, par exemple, de serveurs destinés à l’hébergement d’applications web et mobiles, ainsi que d’autres applications d’entreprise.|
| Mémoire optimisée | Charges de travail de base de données haute performance qui nécessitent des performances en mémoire suffisantes pour un traitement plus rapide des transactions et une simultanéité plus élevée. Il s’agit, par exemple, de serveurs destinés au traitement de données en temps réel et à des applications transactionnelles ou analytiques haute performance.|

Après avoir créé un serveur, le nombre de vCores, la génération du matériel et le niveau tarifaire peuvent être augmentés ou diminués (excepté à vers et à partir de la version De base) en quelques secondes. Vous pouvez également augmenter ou diminuer de manière indépendante la quantité de stockage et la période de rétention des sauvegardes sans interruption de l’application. Vous ne pouvez pas modifier le type de stockage de sauvegarde après la création d’un serveur. Pour plus d’informations, consultez la section [Ressources de mise à l’échelle](#scale-resources).

## <a name="compute-generations-and-vcores"></a>Générations de calcul et vCores

Les ressources de calcul sont fournies en tant que vCores, représentant le processeur logique du matériel sous-jacent. Les régions Chine Est 1, Chine Nord 1, US DoD Centre et US DoD Est utilisent des processeurs logiques Gen 4 basés sur des processeurs Intel E5-2673 v3 (Haswell) à 2,4 GHz. Toutes les autres régions utilisent des processeurs logiques Gen 5 basés sur des processeurs Intel E5-2673 v4 (Broadwell) à 2,3 GHz.

## <a name="storage"></a>Stockage

Le stockage que vous approvisionnez est la quantité de stockage disponible pour votre serveur Azure Database pour MySQL. Le stockage est utilisé pour les fichiers de base de données, les fichiers temporaires, les journaux d’activité de transaction, et les journaux d’activité du serveur MySQL. La quantité totale de stockage que vous approvisionnez définit également la capacité d’E/S disponible sur votre serveur.

| Attribut de stockage   | De base | Usage général | Mémoire optimisée |
|:---|:----------|:--------------------|:---------------------|
| Type de stockage | Stockage de base | Stockage à usage général | Stockage à usage général |
| Taille de stockage | 5 Go à 1 To | 5 Go à 16 To | 5 Go à 16 To |
| Taille d’incrément de stockage | 1 Go | 1 Go | 1 Go |
| E/S par seconde | Variable |3 E/S par seconde/Go<br/>Min 100 E/S par seconde<br/>Max 20 000 IOPS | 3 E/S par seconde/Go<br/>Min 100 E/S par seconde<br/>Max 20 000 IOPS |

> [!NOTE]
> Un stockage d’une capacité maximale de 16 To et 20 000 IOPS sont pris en charge dans les régions suivantes : USA Est, USA Est 2, USA Centre, Brésil Sud, USA Ouest, USA Centre Nord, USA Centre Sud, Europe Nord, Europe Ouest, Royaume-Uni Sud, Royaume-Uni Ouest, Asie Sud-Est, Asie Est, Japon Est, Japon Ouest, Corée Centre, Corée Sud, Australie Est, Australie Sud-Est, USA Ouest 2, USA Centre-Ouest, Canada Est et Canada Centre.
>
> Toutes les autres régions prennent en charge une capacité de stockage maximale de 4 TO et jusqu’à 6 000 IOPS.
>

Vous pouvez ajouter de la capacité de stockage supplémentaire pendant et après la création du serveur et autoriser le système à faire évoluer le stockage automatiquement en fonction de la consommation de votre charge de travail. 

>[!NOTE]
> Le stockage peut seulement monter en puissance.

Le niveau De base n’offre pas de garantie d’E/S par seconde. Dans les niveaux tarifaires Usage général et À mémoire optimisée, les IOPS augmentent avec la taille de stockage approvisionnée selon un ratio de 3:1.

Vous pouvez surveiller votre consommation d’E/S dans le Portail Azure ou à l’aide des commandes Azure CLI. Les métriques pertinentes à surveiller sont [la limite de stockage, le pourcentage de stockage, le stockage utilisé et le pourcentage d’E/S](concepts-monitoring.md).

### <a name="reaching-the-storage-limit"></a>Atteindre la limite de stockage

Les serveurs avec moins de 100 Go de stockage approvisionnés sont marqués en lecture seule si l’espace de stockage libre est inférieur à 5 % de la taille de stockage approvisionnée. Les serveurs avec plus de 100 Go de stockage approvisionnés sont marqués en lecture seule lorsque l’espace de stockage libre est inférieur à 5 Go.

Par exemple, si vous avez approvisionné 110 Go de stockage et que l’utilisation réelle dépasse 105 Go, le serveur est marqué en lecture seule. Sinon, si vous avez provisionné 5 Go de stockage, le serveur est marqué en lecture seule quand le stockage disponible est inférieur à 256 Mo.

Pendant que le service tente de marquer le serveur en lecture seule, toutes les nouvelles demandes de transactions d’écriture sont bloquées et les transactions actives existantes continuent de s’exécuter. Une fois que le serveur est marqué en lecture seule, toutes les opérations d’écriture et validations de transaction ultérieures échouent. Les requêtes de lecture continueront de fonctionner sans interruption. Après avoir augmenté le stockage provisionné, le serveur sera prêt à accepter de nouvelles transactions d’écriture.

Nous vous recommandons d’activer la croissance automatique du stockage ou de configurer une alerte pour vous avertir lorsque votre serveur de stockage est proche du seuil, afin d’éviter la mise en lecture seule. Pour plus d’informations, consultez la documentation sur [comment configurer une alerte](howto-alert-on-metric.md).

### <a name="storage-auto-grow"></a>Croissance automatique du stockage

La croissance automatique du stockage permet à votre serveur de disposer en permanence d’un espace de stockage suffisant et de ne pas passer en lecture seule. Si la croissance automatique du stockage est activée, le stockage évolue automatiquement sans affecter la charge de travail. Pour les serveurs avec moins de 100 Go de stockage approvisionnés, la taille de stockage approvisionnée augmente de 5 Go lorsque l’espace de stockage libre est inférieur à 10 % de la taille de stockage approvisionnée. Pour les serveurs avec plus de 100 Go de stockage approvisionnés, la taille de stockage approvisionnée augmente de 5 % lorsque l’espace de stockage libre est inférieur à 10 Go de taille de stockage approvisionnée. Les limites de stockage maximales indiquées ci-dessus s’appliquent.

Par exemple, si vous avez approvisionné 1000 Go de stockage et que l’utilisation réelle dépasse 990 Go, la taille de stockage du serveur passe à 1050 Go. Sinon, si vous avez configuré 10 Go de stockage, la taille de stockage passe à 15 Go lorsque moins de 1 Go de stockage est libre.

N’oubliez pas que le stockage peut seulement monter en puissance.

## <a name="backup-storage"></a>Stockage de sauvegarde 

Azure Database pour MySQL fournit jusqu’à 100 % du stockage de votre serveur approvisionné en stockage de sauvegarde sans coût supplémentaire. Le stockage de sauvegarde que vous utilisez au-delà de cette quantité est facturé en Go par mois. Par exemple, si vous configurez un serveur avec 250 Go de stockage, vous disposez de 250 Go de stockage supplémentaire pour les sauvegardes de serveur sans frais. Le stockage pour les sauvegardes dépassant 250 Go est facturé conformément au [modèle de tarification](https://azure.microsoft.com/pricing/details/mysql/). Pour comprendre les facteurs influençant l’utilisation du stockage de sauvegarde, la surveillance et le contrôle du coût de stockage de sauvegarde, vous pouvez consulter la [documentation relative à la sauvegarde](concepts-backup.md).

## <a name="scale-resources"></a>Mettre les ressources à l’échelle

Après avoir créé votre serveur, vous pouvez modifier de manière indépendante les vCores, la génération du matériel, le niveau tarifaire (excepté à partir de la version De base), la quantité de stockage et la période de rétention de sauvegarde. Vous ne pouvez pas modifier le type de stockage de sauvegarde après la création d’un serveur. Le nombre de vCores peut être augmenté ou diminué. La période de rétention de sauvegarde peut être augmentée ou diminuée et va de 7 à 35 jours. La taille de stockage ne peut être qu’augmentée. La mise à l’échelle des ressources peut être effectuée par le biais du portail ou d’Azure CLI. Pour obtenir un exemple de mise à l’échelle à l’aide d’Azure CLI, consultez [Surveiller et mettre à l’échelle un serveur Azure Database pour MySQL à l’aide d’Azure CLI](scripts/sample-scale-server.md).

Lorsque vous modifiez le nombre de vCores, la génération du matériel ou le niveau tarifaire, une copie du serveur d’origine est créée avec la nouvelle allocation du calcul. Une fois que le nouveau serveur est opérationnel, les connexions sont basculées vers le nouveau serveur. Pendant le moment durant lequel le système bascule vers le nouveau serveur, aucune nouvelle connexion ne peut être établie, et toutes les transactions non validées sont restaurées. Cette fenêtre varie, mais dans la plupart des cas elle dure moins d’une minute.

La mise à l’échelle du stockage et la modification de la période de rétention de sauvegarde sont des opérations en ligne. Aucune interruption de service n’a lieu et votre application n’est pas affectée. Comme les E/S par seconde augmentent avec la taille du stockage approvisionné, vous pouvez augmenter le nombre de E/S par seconde disponibles pour votre serveur en mettant à l’échelle l’espace de stockage.

## <a name="pricing"></a>Tarifs

Pour obtenir les dernières informations sur la tarification, veuillez consulter le service [Page de tarification](https://azure.microsoft.com/pricing/details/mysql/). Pour voir le coût de la configuration souhaitée, le [Portail Azure](https://portal.azure.com/#create/Microsoft.MySQLServer) affiche le coût mensuel dans l’onglet **Niveau tarifaire** selon les options que vous avez sélectionnées. Si vous n’avez pas d’abonnement Azure, vous pouvez utiliser la calculatrice de prix Azure pour obtenir une estimation. Pour personnaliser les options, sur le site web [Calculatrice de prix d’Azure](https://azure.microsoft.com/pricing/calculator/), sélectionnez **Ajouter des éléments**, développez la catégorie **Bases de données**, puis choisissez **Azure Database pour MySQL**.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Créer un serveur MySQL dans le portail](howto-create-manage-server-portal.md).
- En savoir plus sur les [limites de service](concepts-limits.md).
- Apprendre à [effectuer un scale-out avec des réplicas en lecture](howto-read-replicas-portal.md).
