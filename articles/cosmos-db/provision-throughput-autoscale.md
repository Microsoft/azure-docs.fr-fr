---
title: Créer des conteneurs et des bases de données Azure Cosmos en mode de mise à l’échelle automatique
description: Découvrez les avantages et les cas d’usage, et apprenez à approvisionner des bases de données et des conteneurs Azure Cosmos en mode de mise à l’échelle automatique.
author: kirillg
ms.author: kirillg
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/11/2020
ms.custom: seo-nov-2020
ms.openlocfilehash: ba0dd347c4ee2cb41b34c2fc34f1848a7295dc3a
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368662"
---
# <a name="create-azure-cosmos-containers-and-databases-with-autoscale-throughput"></a>Créer des conteneurs et des bases de données Azure Cosmos en débit de mise à l’échelle automatique
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Dans Azure Cosmos DB, vous pouvez configurer un débit approvisionné standard (manuel) ou approvisionné en mode de mise à l’échelle automatique sur vos bases de données et conteneurs. Le débit approvisionné en mode de mise à l’échelle automatique dans Azure Cosmos DB vous permet de **mettre à l’échelle le débit (RU/s) de votre base de données ou conteneur automatiquement et instantanément**. Le débit est mis à l’échelle en fonction de l’utilisation, sans impact sur la disponibilité, la latence, le débit ou les performances de la charge de travail.

Le débit approvisionné en mode de mise à l’échelle automatique est adapté aux charges de travail stratégiques qui ont des modèles de trafic variables ou imprévisibles et qui nécessitent des contrats SLA sur des performances et une mise à l’échelle élevées. Cet article décrit les avantages et les cas d’usage du débit approvisionné en mode de mise à l’échelle automatique.

## <a name="benefits-of-autoscale"></a>Avantages de la mise à l’échelle automatique

Les bases de données et conteneurs Azure Cosmos configurés avec un débit approvisionné en mode de mise à l’échelle automatique présentent les avantages suivants :

* **Simple :** La mise à l’échelle automatique élimine la complexité liée à la gestion de l’unité de requête/s avec des scripts personnalisés ou une capacité de mise à l’échelle manuelle. 

* **Évolutifs :** Les bases de données et les conteneurs mettent automatiquement à l’échelle le débit approvisionné le cas échéant. Les connexions client et les applications ne sont pas interrompues et il n'y a aucun impact sur les contrats de niveau de service Azure Cosmos DB.

* **Économiques :** la mise à l’échelle automatique permet d’optimiser l’utilisation et le coût de l’utilisation de l’unité de requête/s en mettant à l’échelle la taille des unités quand elles ne sont pas utilisées. Vous payez uniquement les ressources dont vos charges de travail ont besoin sur une base horaire. Sur l’ensemble des heures du mois, si vous définissez le nombre maximal de RU/s(Tmax) avec mise à l’échelle automatique et que vous utilisez la totalité de Tmax pour 66 % des heures ou moins, la mise à l’échelle automatique vous permettra de faire des économies. Pour en savoir plus, consultez l’article qui explique [comment choisir entre le débit provisionné standard (manuel) et avec mise à l’échelle automatique](how-to-choose-offer.md).

* **Hautement disponibles :** les bases de données et les conteneurs qui utilisent la mise à l’échelle automatique utilisent le même backend Azure Cosmos DB distribué à l'échelle mondiale, à tolérance de panne et à haute disponibilité pour garantir la durabilité des données et la haute disponibilité.

## <a name="use-cases-of-autoscale"></a>Cas d’usage de la mise à l’échelle automatique

Les cas d’usage de la mise à l’échelle automatique incluent :

* **Charges de travail imprévisibles ou variables :** Lorsque vos charges de travail ont des pics variables ou imprévisibles dans le cadre de l’utilisation, la mise à l’échelle automatique vous aide à mettre à l’échelle automatiquement vers le bas ou vers le haut en fonction de l’utilisation. Les exemples incluent des sites web de vente au détail qui ont des modèles de trafic différents en fonction du caractère saisonnier ; des charges de travail IOT qui ont des pics à différents moments de la journée ; des applications métier qui voient l’utilisation maximale plusieurs fois par mois ou par an et bien plus encore. Avec la mise à l’échelle automatique, vous n’avez plus besoin de configurer manuellement la capacité au moment des pics ou moyenne. 

* **Nouvelles applications :** Si vous développez une nouvelle application et que vous n’êtes pas sûr du débit (RU/s) dont vous avez besoin, la mise à l’échelle automatique facilite la prise en main. Vous pouvez commencer par le point d’entrée de la mise à l’échelle automatique de 400 à 4 000 RU/s, analyser votre utilisation et déterminer l’unité de requête/s appropriée au fil du temps.

* **Applications peu utilisées :** Si vous avez une application qui est utilisée uniquement pendant quelques heures plusieurs fois par jour, semaine ou mois, par exemple un site d’application/web/blog à faible volume, la mise à l’échelle automatique ajuste la capacité pour gérer l’utilisation des pics et s’adapte à la mise à l’échelle. 

* **Charges de travail de développement et de test :** Si votre équipe ou vous-même utilisez des bases de données et des conteneurs Azure Cosmos pendant les heures de travail, mais que vous n’en avez pas besoin la nuit ou pendant le week-end, la mise à l’échelle automatique vous permet de réduire les coûts en réduisant au minimum en cas de non utilisation. 

* **Charges de travail/requêtes de production planifiées :** Si vous avez une série de demandes, d’opérations ou de requêtes planifiées que vous souhaitez exécuter pendant les périodes d’inactivité, vous pouvez le faire facilement avec la mise à l’échelle automatique. Lorsque vous devez exécuter la charge de travail, le débit est automatiquement mis à l’échelle jusqu’à ce qui est nécessaire et diminue après. 

La création d’une solution personnalisée pour ces problèmes nécessite non seulement un temps considérable, mais également une complexité dans la configuration ou le code de votre application. La mise à l’échelle automatique met en œuvre les scénarios prêts à l’emploi ci-dessus et élimine la nécessité d’une mise à l’échelle personnalisée ou manuelle de la capacité. 

## <a name="how-autoscale-provisioned-throughput-works"></a>Comment fonctionne le débit approvisionné en mode de mise à l’échelle automatique

Lors de la configuration de conteneurs et de bases de données avec mise à l’échelle automatique, vous devez spécifier le débit maximal `Tmax` requis. Azure Cosmos DB met à l’échelle le débit `T` tel que `0.1*Tmax <= T <= Tmax`. Par exemple, si vous définissez le débit maximal sur 20 000 RU/s, le débit sera mis à l’échelle entre 2 000 et 20 000 RU/s. Étant donné que la mise à l’échelle est automatique et instantanée, à tout moment, vous pouvez consommer jusqu’à `Tmax` approvisionné sans retard. 

Vous êtes facturé toutes les heures pour le débit le plus élevé `T` sur lequel le système est mis à l’échelle dans l’heure.

Le point d’entrée pour le débit maximal de mise à l’échelle automatique `Tmax` démarre à 4 000 RU/s, qui est mis à l’échelle entre 400 et 4 000 RU/s. Vous pouvez définir `Tmax` par incréments de 1 000 RU/s et modifier la valeur à tout moment.  

## <a name="enable-autoscale-on-existing-resources"></a>Activer la mise à l’échelle automatique sur les ressources existantes

Utilisez le [portail Azure](how-to-provision-autoscale-throughput.md#enable-autoscale-on-existing-database-or-container), le [modèle Azure Resource Manager](how-to-provision-autoscale-throughput.md#azure-resource-manager), [CLI](how-to-provision-autoscale-throughput.md#azure-cli) ou [PowerShell](how-to-provision-autoscale-throughput.md#azure-powershell) pour activer la mise à l’échelle automatique sur une base de données ou un conteneur existant. Vous pouvez basculer entre la mise à l’échelle automatique et le débit approvisionné standard (manuel) à tout moment. Pour plus d’informations, consultez [cette documentation](autoscale-faq.md#how-does-the-migration-between-autoscale-and-standard-manual-provisioned-throughput-work).

## <a name="throughput-and-storage-limits-for-autoscale"></a><a id="autoscale-limits"></a> Limites de débit et de stockage pour la mise à l’échelle automatique

Pour toute valeur de `Tmax`, la base de données ou le conteneur peut stocker un total de `0.01 * Tmax GB`. Une fois ce volume de stockage atteint, l’unité de requête/s maximale est automatiquement augmenté en fonction de la nouvelle valeur de stockage, sans aucun impact sur votre application. 

Par exemple, si vous démarrez avec une unité de requête/s maximale de 50 000 RU/s (avec une mise à l’échelle de 5 000 à 50 000 RU/s), vous pouvez stocker jusqu’à 500 Go de données. Si vous dépassez 500 Go, par exemple, si le stockage est à présent de 600 Go, la nouvelle unité de requête/s maximale sera de 60 000 RU/s (les mises à l’échelle entre 6 000 et 60 000 RU/s).

Lorsque vous utilisez le débit au niveau de la base de données avec la mise à l’échelle automatique, vous pouvez faire en sorte que les 25 premiers conteneurs partagent une unité de requête/s maximale de 4 000 (échelle entre 400 et 4 000 RU/s), à condition que vous ne dépassiez pas 40 Go de stockage. Pour plus d’informations, consultez [cette documentation](autoscale-faq.md#can-i-change-the-max-rus-on-the-database-or-container).

## <a name="comparison--containers-configured-with-manual-vs-autoscale-throughput"></a>Comparaison : conteneurs configurés en mode manuel contre débit de mise à l’échelle automatique
Pour plus d’informations, consultez cette [documentation](how-to-choose-offer.md) sur la façon de choisir entre le débit standard (manuel) et le débit de mise à l’échelle automatique.  

|| Conteneurs avec débit standard (manuel)  | Conteneurs avec débit de mise à l’échelle automatique |
|---------|---------|---------|
| **Débit approvisionné (RU/s)** | Approvisionné manuellement. | Mise à l’échelle automatique et instantanée en fonction des modèles d’utilisation de la charge de travail. |
| **Limitation du nombre de demandes/opérations (429)**  | Peut intervenir si la consommation dépasse la capacité approvisionnée. | Cela ne se produira pas si vous consommez des unités de requête/s dans la plage de débit de mise à l’échelle automatique que vous avez définie.    |
| **Planification de la capacité** |  Vous devez procéder à une planification de la capacité et approvisionner le débit exact dont vous avez besoin. |    Le système prend automatiquement en charge la planification et la gestion de la capacité. |
| **Tarification** | Vous payez pour l’unité de requête/s approvisionnée manuellement par heure, à l’aide de [l’unité de requête/s standard (manuelle) par taux horaire](https://azure.microsoft.com/pricing/details/cosmos-db/). | Vous payez par heure pour l’unité de requête/s la plus élevée pour laquelle le système effectue un scale-up dans l’heure. <br/><br/> Pour les comptes à une seule région d’écriture, vous payez pour l’unité de requête/s utilisée sur une base horaire, en utilisant [l’unité de requête/s de mise à l’échelle automatique par taux horaire](https://azure.microsoft.com/pricing/details/cosmos-db/). <br/><br/>Pour les comptes à plusieurs régions d’écriture, aucuns frais supplémentaires ne s’appliquent au mode de mise à l’échelle automatique. Vous payez le débit utilisé sur une base horaire en utilisant le même [taux horaire de RU/s pour les écritures multirégions](https://azure.microsoft.com/pricing/details/cosmos-db/). |
| **Idéal pour les types de charges de travail** |  Charges de travail prévisibles et stables|   Charges de travail imprévisibles et variables  |

## <a name="next-steps"></a>Étapes suivantes

* Examinez le [FAQ sur la mise à l’échelle automatique](autoscale-faq.md).
* Découvrez comment [choisir entre le débit manuel et de mise à l’échelle automatique](how-to-choose-offer.md).
* Découvrez comment [approvisionner le débit de mise à l’échelle automatique sur une base de données ou un conteneur Azure Cosmos](how-to-provision-autoscale-throughput.md).
* En savoir plus sur le [partitionnement](partitioning-overview.md) dans Azure Cosmos DB.


