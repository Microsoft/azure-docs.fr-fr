---
title: Sélectionner un type de disque pour les machines virtuelles Azure IaaS - disques managés
description: Découvrez les types de disques Azure disponibles pour les machines virtuelles, notamment les disques Ultra, les disques SSD Premium, les disques SSD Standard et les disques HDD Standard.
author: roygara
ms.author: rogarana
ms.date: 09/30/2020
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 1a7e8e71e26af241d16095a5fa1e6a02a7e3d4c2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500765"
---
# <a name="what-disk-types-are-available-in-azure"></a>Quels sont les types de disque disponibles dans Azure ?

Azure Managed Disks propose quatre types de disque, chaque type étant destiné à des scénarios clients spécifiques.

## <a name="disk-comparison"></a>Comparaison des disques

Le tableau suivant compare quatre types de disque managé : les disques Ultra, les disques SSD Premium, les disques SSD Standard et les disques HDD Standard pour vous aider dans votre choix.

| Detail | Disque Ultra | SSD Premium | SSD Standard | HDD Standard |
| ------ | ---------- | ----------- | ------------ | ------------ |
|Type de disque   |SSD   |SSD   |SSD   |HDD   |
|Scénario   |Charges de travail gourmandes en E/S, telles que le système [SAP HANA](workloads/sap/hana-vm-operations-storage.md), les bases de données de niveau supérieur (par exemple, SQL et Oracle), et autres charges de travail très lourdes en transactions.   |Charges de travail de production et sensibles aux performances   |Serveurs web, applications d’entreprise peu utilisées et Dev/Test   |Sauvegarde, non critique, accès peu fréquent   |
|Taille maximale du disque   |65 536 gibioctets (Gio)    |32 767 Gio    |32 767 Gio   |32 767 Gio   |
|Débit max.   |2 000 Mo/s    |900 Mo/s   |750 Mo/s   |500 Mo/s   |
|Nb max. d’E/S par seconde   |160 000    |20 000   |6 000 / 750   |2 000   |

## <a name="ultra-disk"></a>Disque Ultra

Les disques Ultra Azure fournissent un stockage sur disque présentant un débit élevé, un nombre d’IOPS élevé et une faible latence homogène pour les machines virtuelles Azure IaaS. Entre autres avantages, les disques Ultra permettent de changer dynamiquement le niveau de performance des disques en fonction de vos charges de travail sans avoir à redémarrer les machines virtuelles. Les disques Ultra sont adaptés aux charges de travail qui consomment beaucoup de données, par exemple SAP HANA, les bases de données de niveau supérieur et les charges de travail avec un grand nombre de transactions. Les disques Ultra peuvent uniquement être utilisés en tant que disques de données. Nous vous recommandons d’utiliser des disques SSD Premium comme disques de système d’exploitation.

### <a name="performance"></a>Performances

Lorsque vous approvisionnez un disque Ultra, vous pouvez configurer de manière indépendante la capacité et les performances du disque. Les disques Ultra se déclinent en plusieurs tailles fixes allant de 4 Gio à 64 Tio. Ils présentent un modèle de configuration de performances flexible qui vous permet de configurer les IOPS et le débit de manière indépendante.

Voici certaines fonctionnalités clés des disques Ultra :

- Capacité du disque : La capacité des disques Ultra va de 4 Gio à 64 Tio.
- IOPS du disque : Les disques Ultra prennent en charge des limites d’IOPS de 300 IOPS/Gio, avec au maximum 160 000 IOPS par disque. Pour atteindre le nombre d’IOPS provisionné, vérifiez que le nombre d’IOPS du disque sélectionné est inférieur à la limite d’IOPS de la machine virtuelle. Le nombre minimal d’IOPS garanti par disque est de 2 IOPS/Gio avec une base de référence globale minimale de 100 IOPS. Par exemple, si vous avez un disque Ultra de 4 Gio, vous avez au minimum 100 IOPS au lieu de huit IOPS.
- Débit du disque : Avec les disques Ultra, la limite de débit d’un seul disque est de 256 Kio/s pour chaque IOPS provisionnée, avec au maximum 2 000 Mbits/s par disque (où 1 Mbits/s = 10^6 octets par seconde). Le débit minimal garanti par disque est de 4 Kio/s pour chaque IOPS provisionnée avec une base de référence globale minimale de 1 Mbits/s.
- Avec les disques Ultra, vous pouvez changer les attributs de performances des disques (IOPS et débit) au moment de l’exécution, sans avoir à détacher ces derniers de la machine virtuelle. Une fois qu’une opération de redimensionnement de performances de disque est émise sur un disque, elle peut mettre jusqu’à une heure à être appliquée. Le nombre d’opérations de redimensionnement de performances pendant une période de 24 heures est limité à quatre. Une opération de redimensionnement de performances peut échouer si la capacité de la bande passante des performances est insuffisante.

### <a name="disk-size"></a>Taille du disque

|Taille de disque (Gio)  |Plafond d’IOPS  |Débit limite (Mbits/s)  |
|---------|---------|---------|
|4     |1,200         |300         |
|8     |2 400         |600         |
|16     |4 800         |1,200         |
|32     |9 600         |2 000         |
|64     |19 200         |2 000         |
|128     |38 400         |2 000         |
|256     |76 800         |2 000         |
|512     |153 600         |2 000         |
|1 024 à 65 536 (dans cette plage, les tailles augmentent par incréments de 1 Tio)     |160 000         |2 000         |

Les disques Ultra sont conçus pour fournir des latences inférieure à la milliseconde ainsi que l’IOPS et le débit cibles décrits dans le tableau précédent 99,99 % du temps.

### <a name="ga-scope-and-limitations"></a>Étendue et limitations de la version en disponibilité générale

[!INCLUDE [managed-disks-ultra-disks-GA-scope-and-limitations](../../includes/managed-disks-ultra-disks-GA-scope-and-limitations.md)]


Si vous souhaitez commencer à utiliser des disques Ultra, consultez notre article sur le sujet : [Utilisation de disques Ultra Azure](disks-enable-ultra-ssd.md).

## <a name="premium-ssd"></a>SSD Premium

Les disques SSD Premium Azure offrent une prise en charge très performante et une faible latence pour les machines virtuelles avec des charges de travail qui utilisent beaucoup d’entrée/sortie (E/S). Pour tirer parti de la vitesse et des performances des disques de stockage Premium, vous pouvez migrer les disques de machines virtuelles existantes vers des disques SSD Premium. Les disques SSD Premium sont adaptés aux applications de production critiques. Ces disques sont utilisables uniquement avec des séries de machines virtuelles compatibles avec le stockage Premium.

Pour en savoir plus sur les types et les tailles de machines virtuelles individuelles dans Azure pour Windows ou Linux, notamment les tailles compatibles avec le stockage Premium, consultez [Tailles des machines virtuelles dans Azure](sizes.md). À partir de cet article, vous devez vérifier chaque taille de machine virtuelle pour déterminer si elle est compatible avec le stockage Premium.

### <a name="disk-size"></a>Taille du disque
[!INCLUDE [disk-storage-premium-ssd-sizes](../../includes/disk-storage-premium-ssd-sizes.md)]

Lorsque vous configurez un disque de stockage Premium, contrairement au stockage standard, la capacité, les E/S par seconde et le débit de ce disque sont assurés. Par exemple, si vous créez un disque P50, Azure configure une capacité de stockage de 4 095 Go, 7 500 E/S par seconde et un débit de 250 Mo/s pour ce disque. Votre application peut utiliser tout ou partie de la capacité et des performances. Les disques SSD Premium sont conçus pour fournir des latences faibles de quelques millisecondes ainsi que l’IOPS et le débit cibles décrits dans le précédent tableau 99,9 % du temps.

## <a name="bursting"></a>Mode en rafales

Les tailles de disques SSD Premium inférieures à P30 offrent désormais un mode rafale et peuvent augmenter leur IOPS par disque jusqu’à 3 500 et leur bande passante jusqu’à 170 Mo/s. Ce mode en rafales est automatisé et fonctionne selon un système de crédits. Les crédits s’accumulent automatiquement dans un compartiment de rafales quand le trafic de disque est inférieur aux performances cibles provisionnées et les crédits sont automatiquement consommés quand le trafic dépasse la cible, jusqu’à la limite maximale de rafales. La limite maximale de rafales définit le plafond des IOPS de disque et de la bande passante même si vous disposez de crédits de rafales à consommer. Les rafales de disque offrent une meilleure tolérance aux changements imprévisibles des modèles d’E/S. Vous pouvez en tirer le meilleur parti pour le démarrage du disque du système d’exploitation et les applications dont le trafic est imprévisible.    

La prise en charge du mode rafale du disque sera activée sur les nouveaux déploiements des tailles de disque par défaut, sans nécessiter l’intervention de l’utilisateur. Pour les disques existants dont les tailles sont applicables, vous pouvez activer les rafales de l’une ou l’autre des manières suivantes : détacher et rattacher le disque, ou arrêter et redémarrer la machine virtuelle attachée. Toutes les tailles de disques applicables au mode en rafales vont commencer avec un compartiment disposant d’un crédit de rafales complet dès que le disque est attaché à une machine virtuelle qui prend en charge une durée maximale de pointe de rafales limitée à 30 minutes. Pour en savoir plus sur le fonctionnement du mode en rafales sur les disques Azure, consultez [Mode en rafales des disques SSD Premium](./disk-bursting.md). 

### <a name="transactions"></a>Transactions

Pour les disques SSD Premium, chaque opération d’E/S inférieure ou égale à 256 Kio de débit est considérée comme une seule opération d’E/S. Les opérations d’E/S dont le débit est supérieur à 256 Kio sont considérées comme des opérations d’E/S distinctes, de 256 Kio chacune.

## <a name="standard-ssd"></a>SSD Standard

Les disques SSD Azure Standard constituent une option de stockage économique optimisée pour les charges de travail nécessitant des performances cohérentes à des niveaux d’IOPS bas. Les disques SSD Standard offrent une bonne expérience de niveau d’entrée pour ceux qui souhaitent migrer vers le cloud, en particulier si vous rencontrez des problèmes de variation avec les charges de travail exécutées sur vos solutions HDD locales. Par rapport aux disques HDD Standard, les disques SSD Standard offrent une meilleure disponibilité, cohérence, fiabilité et latence. Les disques SSD Standard sont adaptés aux serveurs web, serveurs d’applications avec peu d’IOPS, applications d’entreprise peu utilisées et charges de travail Dev/Test. Comme les disques HDD Standard, les disques SSD Standard sont disponibles sur toutes les machines virtuelles Azure.

### <a name="disk-size"></a>Taille du disque
[!INCLUDE [disk-storage-standard-ssd-sizes](../../includes/disk-storage-standard-ssd-sizes.md)]

Les disques SSD Standard sont conçus pour fournir des latences de quelques millisecondes ainsi qu’un IOPS et un débit dans les limites décrites dans le tableau précédent 99 % du temps. L’IOPS et le débit réels peuvent varier selon les modèles de trafic. Les disques SSD Standard offrent des performances plus cohérentes que les disques HDD avec une latence plus faible.

### <a name="transactions"></a>Transactions

Pour les disques SSD Standard, chaque opération d’E/S inférieure ou égale à 256 Kio de débit est considérée comme une seule opération d’E/S. Les opérations d’E/S dont le débit est supérieur à 256 Kio sont considérées comme des opérations d’E/S distinctes, de 256 Kio chacune. Ces transactions ont un impact sur la facturation.

## <a name="standard-hdd"></a>HDD Standard

Les disques HDD standard Azure offrent une prise en charge fiable et économique des disques des machines virtuelles exécutant des charges de travail non sensibles aux latences. Dans le cadre d’un stockage Standard, les données sont stockées sur des disques durs. La latence, l’IOPS et le débit sur les disques HDD Standard peuvent varier beaucoup plus que sur les disques SSD. Les disques HDD Standard sont conçus pour offrir une latence inférieure à 10 ms en écriture et à 20 ms en lecture pour la plupart des opérations d’E/S. Toutefois, les performances réelles peuvent varier en fonction de la taille d’E/S et du modèle de charge de travail. Avec des machines virtuelles, vous pouvez utiliser des disques HDD Standard pour les scénarios Dev/Test et les charges de travail moins critiques. Les disques HDD Standard sont disponibles dans toutes les régions Azure et peuvent être utilisés avec toutes les machines virtuelles Azure.

### <a name="disk-size"></a>Taille du disque
[!INCLUDE [disk-storage-standard-hdd-sizes](../../includes/disk-storage-standard-hdd-sizes.md)]

### <a name="transactions"></a>Transactions

Pour les disques SSD Standard, chaque opération d’E/S est considérée comme une seule transaction, indépendamment de la taille des E/S. Ces transactions ont un impact sur la facturation.

## <a name="billing"></a>Facturation

Lorsque vous utilisez des disques managés, les considérations de facturation suivantes s’appliquent :

- Type de disque
- Taille des disques managés
- Instantanés
- Transferts de données sortantes
- Nombre de transactions

**Taille des disques managés** : les disques managés sont facturés selon la taille provisionnée. Azure fait correspondre la taille provisionnée (arrondie à la valeur supérieure) à la taille de disque offerte la plus proche. Pour plus d’informations sur les tailles de disque proposées, consultez les tableaux précédents. Chaque disque correspond à une offre de taille de disque configurée prise en charge, et est facturé en conséquence. Par exemple, si vous avez provisionné un disque SSD Standard de 200 Gio, celui-ci correspond à l’offre de taille de disque E15 (256 Gio). La facturation de n’importe quel disque configuré est calculée au prorata horaire sur la base du tarif mensuel de l’offre de stockage. Par exemple, si vous provisionnez un disque E10 et le supprimez au bout de 20 heures, l’offre E10 vous est facturée au prorata de 20 heures. Le montant facturé est indépendant de la quantité de données écrites sur le disque.

**Instantanés** : Les instantanés facturés en fonction de la taille utilisée. Par exemple, si vous créez une capture instantanée d’un disque managé avec une capacité approvisionnée de 64 Gio et une taille des données utilisées réelle de 10 Gio, l'instantané est facturé uniquement pour la taille des données utilisées de 10 Gio.

Pour plus d'informations sur les captures instantanées, consultez la section qui leur est consacrée dans la [Vue d'ensemble des disques managés](managed-disks-overview.md).

**Transferts de données sortantes**: les [transferts de données sortantes](https://azure.microsoft.com/pricing/details/bandwidth/) (données sortant des centres de données Azure) sont facturés en fonction de la bande passante utilisée.

**Transactions** : vous êtes facturé pour le nombre de transactions effectuées sur un disque managé standard. Pour les disques SSD Standard, chaque opération d’E/S inférieure ou égale à 256 Kio de débit est considérée comme une seule opération d’E/S. Les opérations d’E/S dont le débit est supérieur à 256 Kio sont considérées comme des opérations d’E/S distinctes, de 256 Kio chacune. Pour les disques SSD Standard, chaque opération d’E/S est considérée comme une seule transaction, indépendamment de la taille des E/S.

Pour plus d’informations sur la tarification d’Azure Disques managés (dont les coûts de transaction), consultez la page [Tarification de la fonctionnalité Disques managés](https://azure.microsoft.com/pricing/details/managed-disks).

### <a name="ultra-disk-vm-reservation-fee"></a>Frais de réservation de machine virtuelle avec disque Ultra

Les machines virtuelles Azure peuvent indiquer si elles sont compatibles avec les disques Ultra. Une machine virtuelle compatible avec les disques Ultra alloue une capacité de bande passante dédiée entre l'instance de machine virtuelle de calcul et l'unité d'échelle de stockage de blocs pour optimiser les performances et réduire la latence. L'ajout de cette fonctionnalité sur la machine virtuelle entraîne des frais de réservation qui ne sont imposés que si vous avez activé la fonctionnalité de disque Ultra sur la machine virtuelle sans y joindre de disque Ultra. Lorsqu'un disque Ultra est joint à la machine virtuelle compatible avec les disques Ultra, ces frais ne s'appliquent pas. Les frais sont calculés par processeur virtuel provisionné sur la machine virtuelle. 

> [!Note]
> Pour les [tailles de machine virtuelle avec des cœurs restreints](constrained-vcpu.md), les frais de réservation sont basés sur le nombre réel de processeurs virtuels et non sur les cœurs restreints. Pour Standard_E32-8s_v3, les frais de réservation sont basés sur 32 cœurs. 

Pour plus d’informations sur la tarification des disques Ultra, consultez la [page relative à la tarification des disques Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

### <a name="azure-disk-reservation"></a>Réservation de disque Azure

La réservation de disque est la possibilité d’acheter à l’avance un an de stockage sur disque avec une remise, ce qui réduit le coût total pour vous. Lors de l’achat d’une réservation de disque, vous sélectionnez une référence SKU de disque spécifique dans une région cible, par exemple 10 disques SSD P30 (1 Tio) Premium dans la région USA Est 2 pour une période d’un an. L’expérience de réservation est similaire aux instances de machines virtuelles réservées. Vous pouvez regrouper les réservations de machines virtuelles et de disques pour optimiser vos économies. Pour le moment, la réservation de disques Azure offre un plan d’engagement d’un an pour les références (SKU) de disques SSD Premium de P30 (1 Tio) à P80 (32 Tio) dans toutes les régions de production. Pour plus d’informations sur la tarification des disques réservés, consultez [Page de tarification des disques Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, consultez [Tarification des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/).