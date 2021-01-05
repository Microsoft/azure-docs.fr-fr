---
title: Niveaux tarifaires - Azure Database for MariaDB
description: Découvrez les différents niveaux tarifaires d’Azure Database for MariaDB, notamment les générations de calcul, les types de stockage, la taille de stockage, les vCores, la mémoire et les périodes de conservation des sauvegardes.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: b5b5a506b2f932d20a617634ace7ebf02093fbfa
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94536283"
---
# <a name="azure-database-for-mariadb-pricing-tiers"></a>Niveaux tarifaires pour Azure Database for MariaDB

Vous pouvez créer un serveur Azure Database for MariaDB dans un des trois différents niveaux tarifaires : De base, Usage général et À mémoire optimisée. Les niveaux tarifaires diffèrent par la quantité de calcul dans vCores qui peut être configurée, la mémoire par vCore et la technologie de stockage utilisée pour stocker les données. Toutes les ressources sont provisionnées au niveau du serveur MariaDB. Un serveur peut avoir une ou plusieurs bases de données.

| Ressource | **De base** | **Usage général** | **Mémoire optimisée** |
|:---|:----------|:--------------------|:---------------------|
| Génération de calcul | Gen 5 |Gen 5 | Gen 5 |
| vCores | 1, 2 | 2, 4, 8, 16, 32, 64 |2, 4, 8, 16, 32 |
| Mémoire par vCore | 2 Go | 5 Go | 10 Go |
| Taille de stockage | 5 Go à 1 To | 5 Go à 4 To | 5 Go à 4 To |
| Période de rétention de sauvegarde de bases de données | 7 à 35 jours | 7 à 35 jours | 7 à 35 jours |

Pour choisir un niveau tarifaire, utilisez le tableau suivant comme point de départ.

| Niveau tarifaire | Charges de travail cibles |
|:-------------|:-----------------|
| De base | Charges de travail qui nécessitent des performances légères en termes de calcul et d’E/S. Il s’agit, par exemple, de serveurs utilisés pour le développement ou le test, ou pour des applications à petite échelle rarement utilisées. |
| Usage général | La plupart des charges de travail professionnelles qui nécessitent une capacité de calcul et de mémoire équilibrée avec un débit d’E/S extensible. Il s’agit, par exemple, de serveurs destinés à l’hébergement d’applications web et mobiles, ainsi que d’autres applications d’entreprise.|
| Mémoire optimisée | Charges de travail de base de données haute performance qui nécessitent des performances en mémoire suffisantes pour un traitement plus rapide des transactions et une simultanéité plus élevée. Il s’agit, par exemple, de serveurs destinés au traitement de données en temps réel et à des applications transactionnelles ou analytiques haute performance.|

Après avoir créé un serveur, le nombre de vCores et le niveau tarifaire (excepté vers et à partir du niveau De base) peuvent être augmentés ou diminués en quelques secondes. Vous pouvez également augmenter ou diminuer de manière indépendante la quantité de stockage et la période de rétention des sauvegardes sans interruption de l’application. Vous ne pouvez pas modifier le type de stockage de sauvegarde après la création d’un serveur. Pour plus d’informations, consultez la section [Ressources de mise à l’échelle](#scale-resources).

## <a name="compute-generations-and-vcores"></a>Générations de calcul et vCores

Les ressources de calcul sont fournies en tant que vCores, représentant le processeur logique du matériel sous-jacent. Les processeurs logiques Gen 5 sont basés sur des processeurs Intel E5-2673 v4 (Broadwell) 2.3 GHz.

## <a name="storage"></a>Stockage

Le stockage que vous provisionnez est la quantité de stockage disponible pour votre serveur Azure Database for MariaDB. Le stockage est utilisé pour les fichiers de base de données, les fichiers temporaires, les journaux d’activité des transactions et les journaux d’activité des serveurs MariaDB. La quantité totale de stockage que vous approvisionnez définit également la capacité d’E/S disponible sur votre serveur.

| Attributs de stockage   | De base | Usage général | Mémoire optimisée |
|:---|:----------|:--------------------|:---------------------|
| Type de stockage | Stockage de base | Stockage à usage général | Stockage à usage général |
| Taille de stockage | 5 Go à 1 To | 5 Go à 4 To | 5 Go à 4 To |
| Taille d’incrément de stockage | 1 Go | 1 Go | 1 Go |
| E/S par seconde | Variable |3 E/S par seconde/Go<br/>Min 100 E/S par seconde<br/>Max 6000 E/S par seconde | 3 E/S par seconde/Go<br/>Min 100 E/S par seconde<br/>Max 6000 E/S par seconde |

Vous pouvez ajouter de la capacité de stockage supplémentaire pendant et après la création du serveur et autoriser le système à faire évoluer le stockage automatiquement en fonction de la consommation de votre charge de travail.

>[!NOTE]
> Le stockage peut seulement monter en puissance.

Le niveau De base n’offre pas de garantie d’E/S par seconde. Dans les niveaux tarifaires Usage général et À mémoire optimisée, les IOPS augmentent avec la taille de stockage approvisionnée selon un ratio de 3:1.

Vous pouvez surveiller votre consommation d’E/S dans le Portail Azure ou à l’aide des commandes Azure CLI. Les métriques pertinentes à surveiller sont [la limite de stockage, le pourcentage de stockage, le stockage utilisé et le pourcentage d’E/S](concepts-monitoring.md).

### <a name="large-storage-preview"></a>Stockage volumineux (préversion)

Nous avons augmenté les limites de stockage dans nos niveaux de service Usage général et Mémoire optimisée. Les serveurs nouvellement créés qui ont opté pour la préversion peuvent configurer jusqu’à 16 To de stockage. Les IOPS évoluent à un taux de 3:1 jusqu’à 20 000. Comme pour le stockage à disponibilité générale actuel, vous pouvez ajouter de la capacité de stockage supplémentaire après la création du serveur et autoriser le système à faire évoluer le stockage automatiquement en fonction de la consommation de votre charge de travail.

| Attributs de stockage | Usage général | Mémoire optimisée |
|:-------------|:--------------------|:---------------------|
| Type de stockage | Stockage Premium Azure | Stockage Premium Azure |
| Taille de stockage | 32 Go à 16 To| 32 à 16 To |
| Taille d’incrément de stockage | 1 Go | 1 Go |
| E/S par seconde | 3 E/S par seconde/Go<br/>Min 100 E/S par seconde<br/>Max 20 000 IOPS| 3 E/S par seconde/Go<br/>Min 100 E/S par seconde<br/>Max 20 000 IOPS |

> [!IMPORTANT]
> Le stockage volumineux est actuellement en préversion publique dans les régions suivantes : USA Est, USA Est 2, Brésil Sud, USA Centre, USA Ouest, USA Centre Nord, USA Centre Sud, Europe Nord, Europe Ouest, Royaume-Uni Sud, Royaume-Uni Ouest, Asie Sud-Est, Asie Est, Japon Est, Japon Ouest, Corée Centre, Corée Sud, Australie Est, Australie Sud-Est, USA Ouest 2, USA Centre-Ouest, Canada Est et Canada Centre.
>
> Toutes les autres régions prennent en charge une capacité de stockage maximale de 4 TO et jusqu’à 6 000 IOPS.
>

### <a name="reaching-the-storage-limit"></a>Atteindre la limite de stockage

Les serveurs avec moins de 100 Go de stockage approvisionnés sont marqués en lecture seule si l’espace de stockage libre est inférieur à 5 % de la taille de stockage approvisionnée. Les serveurs avec plus de 100 Go de stockage approvisionnés sont marqués en lecture seule lorsque l’espace de stockage libre est inférieur à 5 Go.

Par exemple, si vous avez approvisionné 110 Go de stockage et que l’utilisation réelle dépasse 105 Go, le serveur est marqué en lecture seule. Sinon, si vous avez provisionné 5 Go de stockage, le serveur est marqué en lecture seule quand le stockage disponible est inférieur à 256 Mo.

Pendant que le service tente de marquer le serveur en lecture seule, toutes les nouvelles demandes de transactions d’écriture sont bloquées et les transactions actives existantes continuent de s’exécuter. Une fois que le serveur est marqué en lecture seule, toutes les opérations d’écriture et validations de transaction ultérieures échouent. Les requêtes de lecture continueront de fonctionner sans interruption. Après avoir augmenté le stockage provisionné, le serveur sera prêt à accepter de nouvelles transactions d’écriture.

Nous vous recommandons d’activer la croissance automatique du stockage ou de configurer une alerte pour vous avertir lorsque votre serveur de stockage est proche du seuil, afin d’éviter la mise en lecture seule. Pour plus d’informations, consultez la documentation sur [comment configurer une alerte](howto-alert-metric.md).

### <a name="storage-auto-grow"></a>Croissance automatique du stockage

La croissance automatique du stockage permet à votre serveur de disposer en permanence d’un espace de stockage suffisant et de ne pas passer en lecture seule. Si la croissance automatique du stockage est activée, le stockage évolue automatiquement sans affecter la charge de travail. Pour les serveurs avec moins de 100 Go de stockage approvisionnés, la taille de stockage approvisionnée augmente de 5 Go lorsque l’espace de stockage libre est inférieur à 10 % de la taille de stockage approvisionnée. Pour les serveurs avec plus de 100 Go de stockage approvisionnés, la taille de stockage approvisionnée augmente de 5 % lorsque l’espace de stockage libre est inférieur à 10 Go de taille de stockage approvisionnée. Les limites de stockage maximales indiquées ci-dessus s’appliquent.

Par exemple, si vous avez approvisionné 1000 Go de stockage et que l’utilisation réelle dépasse 990 Go, la taille de stockage du serveur passe à 1050 Go. Sinon, si vous avez configuré 10 Go de stockage, la taille de stockage passe à 15 Go lorsque moins de 1 Go de stockage est libre.

N’oubliez pas que le stockage peut seulement monter en puissance.

## <a name="backup"></a>Sauvegarde

Azure Database for MariaDB fournit jusqu’à 100 % du stockage de votre serveur provisionné en stockage de sauvegarde sans coût supplémentaire. Le stockage de sauvegarde que vous utilisez au-delà de cette quantité est facturé en Go par mois. Par exemple, si vous configurez un serveur avec 250 Go de stockage, vous disposez de 250 Go de stockage supplémentaire pour les sauvegardes de serveur sans frais. Le stockage pour les sauvegardes dépassant 250 Go est facturé conformément au [modèle de tarification](https://azure.microsoft.com/pricing/details/mariadb/). Pour comprendre les facteurs influençant l’utilisation du stockage de sauvegarde, la surveillance et le contrôle du coût de stockage de sauvegarde, vous pouvez consulter la [documentation relative à la sauvegarde](concepts-backup.md).

## <a name="scale-resources"></a>Mettre les ressources à l’échelle

Après avoir créé votre serveur, vous pouvez modifier de manière indépendante les vCores, le niveau tarifaire (excepté vers et à partir du niveau De base), la quantité de stockage et la période de rétention des sauvegardes. Vous ne pouvez pas modifier le type de stockage de sauvegarde après la création d’un serveur. Le nombre de vCores peut être augmenté ou diminué. La période de rétention de sauvegarde peut être augmentée ou diminuée et va de 7 à 35 jours. La taille de stockage ne peut être qu’augmentée. La mise à l’échelle des ressources peut être effectuée par le biais du portail ou d’Azure CLI. 

Quand vous changez le nombre de vCores ou le niveau tarifaire, une copie du serveur d’origine est créée avec la nouvelle allocation du calcul. Une fois que le nouveau serveur est opérationnel, les connexions sont basculées vers le nouveau serveur. Pendant le moment durant lequel le système bascule vers le nouveau serveur, aucune nouvelle connexion ne peut être établie, et toutes les transactions non validées sont restaurées. Cette fenêtre varie, mais dans la plupart des cas elle dure moins d’une minute.

La mise à l’échelle du stockage et la modification de la période de rétention de sauvegarde sont des opérations en ligne. Aucune interruption de service n’a lieu et votre application n’est pas affectée. Comme les E/S par seconde augmentent avec la taille du stockage approvisionné, vous pouvez augmenter le nombre de E/S par seconde disponibles pour votre serveur en mettant à l’échelle l’espace de stockage.

## <a name="pricing"></a>Tarifs

Pour obtenir les dernières informations sur la tarification, veuillez consulter le service [Page de tarification](https://azure.microsoft.com/pricing/details/mariadb/). Pour voir le coût de la configuration souhaitée, le [Portail Azure](https://portal.azure.com/#create/Microsoft.MariaDBServer) affiche le coût mensuel dans l’onglet **Niveau tarifaire** selon les options que vous avez sélectionnées. Si vous n’avez pas d’abonnement Azure, vous pouvez utiliser la calculatrice de prix Azure pour obtenir une estimation. Pour personnaliser les options, sur le site web [Calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/), sélectionnez **Ajouter des éléments**, développez la catégorie **Bases de données**, puis choisissez **Azure Database for MariaDB**.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [Limitations de service](concepts-limits.md).
- Découvrez comment [créer un serveur MariaDB dans le portail Azure](quickstart-create-mariadb-server-database-using-azure-portal.md).