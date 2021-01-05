---
title: Évaluations de machines virtuelles Azure dans Azure Migrate Server Assessment
description: Apprenez-en davantage sur les évaluations dans l'outil Azure Migrate Server Assessment
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 05/27/2020
ms.openlocfilehash: f8a4f29114f7e0a2ed7868f01e05e25c8a0d0ce1
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752224"
---
# <a name="server-assessment-overview-migrate-to-azure-vms"></a>Vue d’ensemble de l’évaluation du serveur (migrer vers des machines virtuelles Azure)

Cet article fournit une vue d’ensemble des évaluations dans l’outil [Azure Migrate : Évaluation de serveur](migrate-services-overview.md#azure-migrate-server-assessment-tool). L’outil permet d’évaluer des machines virtuelles VMware et Hyper-V ainsi que des serveurs physiques locaux en vue de leur migration vers Azure.

## <a name="whats-an-assessment"></a>Qu'est-ce qu'une évaluation ?

Une évaluation réalisée à l’aide de l’outil Évaluation de serveur permet de mesurer l’état de préparation et d’estimer l’effet de la migration des serveurs locaux vers Azure.

> [!NOTE]
> Dans Azure Government, examinez les emplacements d’évaluation des [cibles prises en charge](migrate-support-matrix.md#supported-geographies-azure-government). Notez que les recommandations relatives à la taille des machines virtuelles dans les évaluations utilisent la série de machines virtuelles spécifiquement pour les régions cloud Azure Government. [En savoir plus](https://azure.microsoft.com/global-infrastructure/services/?regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia&products=virtual-machines) sur les types de machines virtuelles.

## <a name="types-of-assessments"></a>Types d'évaluations

Vous pouvez créer deux types d’évaluation à l’aide d’Azure Migrate : Server Assessment.

**Type d’évaluation** | **Détails**
--- | --- 
**Microsoft Azure** | Évaluations pour migrer vos serveurs locaux vers des machines virtuelles Azure. <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md), [machines virtuelles Hyper-V](how-to-set-up-appliance-hyper-v.md) et [serveurs physiques](how-to-set-up-appliance-physical.md) locaux pour la migration vers Azure à l’aide de ce type d’évaluation.
**Azure VMware Solution (AVS)** | Évaluations pour migrer vos serveurs locaux vers [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md) locales pour la migration vers Azure VMware Solution (AVS) avec ce type d’évaluation. [En savoir plus](concepts-azure-vmware-solution-assessment-calculation.md)

Les évaluations que vous créez à l'aide de l'outil Évaluation de serveur correspondent à un instantané ponctuel des données. Une évaluation de machine virtuelle Azure dans Server Assessment fournit deux options de critères de dimensionnement :

**Type d’évaluation** | **Détails** | **Données**
--- | --- | ---
**Basée sur les performances** | Évaluations qui donnent des recommandations en fonction des données de performances collectées | La recommandation de taille de machine virtuelle est basée sur les données d’utilisation de l’UC et de la RAM.<br/><br/> La recommandation de type de disque est basée sur les opérations d’entrée/sortie par seconde (IOPS) et le débit des disques locaux. Les types de disques sont HDD Standard Azure, SSD Standard Azure et Azure Premium.
**Telle quelle locale** | Évaluations qui n’utilisent pas de données de performances pour formuler des recommandations | La recommandation de taille de machine virtuelle est basée sur la taille de la machines virtuelle locale.<br/><br> Le type de disque recommandé est basé sur le type de stockage sélectionné pour l’évaluation.

## <a name="how-do-i-run-an-assessment"></a>Comment exécuter une évaluation ?

Une évaluation peut être réalisée de différentes façons.

- Évaluation des machines à l’aide des métadonnées du serveur collectées par une appliance Azure Migrate légère. L’appliance découvre des machines locales. Elle envoie ensuite les métadonnées et les données de performances des machines à Azure Migrate.
- Évaluation des machines à l’aide des métadonnées du serveur importées au format CSV.

## <a name="how-do-i-assess-with-the-appliance"></a>Comment procéder à une évaluation à l'aide de l'appliance ?

Si vous déployez une appliance Azure Migrate pour découvrir des serveurs locaux, procédez comme suit :

1. Configurez Azure et votre environnement local de manière à utiliser l’outil Évaluation de serveur.
1. Pour votre première évaluation, créez un projet Azure et ajoutez-y l’outil Évaluation de serveur.
1. Déployez une appliance Azure Migrate légère. Celle-ci découvre en continu les machines locales et envoie leurs métadonnées et leurs données de performances à Azure Migrate. Déployez l’appliance en tant que machine virtuelle ou physique. Vous n’avez rien à installer sur les machines que vous souhaitez évaluer.

Une fois que l’appliance a commencé la découverte des machines, vous pouvez rassembler les machines que vous souhaitez évaluer au sein d’un groupe et exécuter une évaluation du groupe avec le type d’évaluation **Machine virtuelle Azure**.

Pour essayer ces étapes, suivez nos tutoriels consacrés à [VMware](./tutorial-discover-vmware.md), [Hyper-V](./tutorial-discover-hyper-v.md) ou aux [serveurs physiques](./tutorial-discover-physical.md).

## <a name="how-do-i-assess-with-imported-data"></a>Comment procéder à une évaluation à l'aide de données importées ?

Si vous évaluez des serveurs à l’aide d’un fichier CSV, aucune appliance n’est nécessaire. Au lieu de cela, procédez comme suit :

1. Configurez Azure de manière à utiliser l’outil Évaluation de serveur.
1. Pour votre première évaluation, créez un projet Azure et ajoutez-y l’outil Évaluation de serveur.
1. Téléchargez un modèle CSV et ajoutez-y les données du serveur.
1. Importez le modèle dans l’outil Évaluation de serveur.
1. Découvrez les serveurs ajoutés à l’aide de l’importation, rassemblez-les au sein d’un groupe et exécutez une évaluation du groupe avec le type d’évaluation **Machine virtuelle Azure**.

## <a name="what-data-does-the-appliance-collect"></a>Quelles données l’appliance collecte-t-elle ?

Si vous utilisez l'appliance Azure Migrate à des fins d'évaluation, apprenez-en plus sur les métadonnées et les données de performance qui sont collectées pour [VMware](migrate-appliance.md#collected-data---vmware) et [Hyper-V](migrate-appliance.md#collected-data---hyper-v).

## <a name="how-does-the-appliance-calculate-performance-data"></a>Comment l'appliance calcule-t-elle les données de performances ?

Si vous utilisez l’appliance à des fins de découverte, elle collecte des données de performances pour les paramètres de calcul en procédant comme suit :

1. L’appliance collecte un point d’échantillonnage en temps réel.

    - **Machines virtuelles VMware** : Un point d’échantillonnage est collecté toutes les 20 secondes.
    - **Machines virtuelles Hyper-V** : Un point d’échantillonnage est collecté toutes les 30 secondes.
    - **Serveurs physiques** : Un point d’échantillonnage est collecté toutes les cinq minutes.

1. L’appliance combine les points d’échantillonnage afin de créer un point de données unique toutes les 10 minutes pour les serveurs VMware et Hyper-V, et toutes les 5 minutes pour les serveurs physiques. Pour créer le point de données, l’appliance sélectionne les valeurs maximales de tous les échantillons. Elle envoie ensuite le point de données à Azure.
1. Évaluation de serveur stocke tous les points d’échantillonnage de 10 minutes pour le dernier mois.
1. Lorsque vous créez une évaluation, Évaluation de serveur identifie le point de données approprié à utiliser pour le dimensionnement adéquat. L’identification est basée sur les valeurs de centile pour *Historique des performances* et *Utilisation en centile*.

    - Par exemple, si l’historique des performances est d’une semaine et que l’utilisation en centile est le 95e centile, Évaluation de serveur trie les points d’échantillonnage de 10 minutes pour la dernière semaine. Il les trie dans l’ordre croissant et sélectionne la valeur du 95e centile pour le dimensionnement adéquat.
    - Le 95e centile vous permet d’ignorer les valeurs hors norme qui peuvent être incluses quand vous choisissez le 99e centile.
    - Si vous souhaitez choisir l’utilisation maximale de la période et que vous ne souhaitez pas ignorer les valeurs hors norme, sélectionnez le 99e centile comme utilisation en centile.

1. Cette valeur est multipliée par le facteur de confort pour obtenir les données réelles d’utilisation de performances pour ces mesures collectées par l’appliance :

    - Utilisation du processeur
    - Utilisation de la RAM
    - IOPS de disque (lecture et écriture)
    - Débit de disque (lecture et écriture)
    - Débit réseau (entrant et sortant)

## <a name="how-are-azure-vm-assessments-calculated"></a>Comment les évaluations de machines virtuelles Azure sont-elles calculées ?

Évaluation de serveur utilise les métadonnées et les données de performances des machines locales pour calculer les évaluations. Si vous déployez l’appliance Azure Migrate, l’évaluation utilise les données collectées par l’appliance. Toutefois, si vous exécutez une évaluation importée à l’aide d’un fichier CSV, vous devez fournir les métadonnées pour le calcul.

Les calculs se font dans les trois étapes suivantes :

1. **Calculer l’état de préparation pour Azure** : Évaluez si les machines sont adaptées à la migration vers Azure.
1. **Calculer les recommandations de dimensionnement** : Estimez le dimensionnement du calcul, du stockage et du réseau.
1. **Calculer les coûts mensuels** : Calculez les coûts mensuels de calcul et de stockage estimés pour l’exécution des machines dans Azure après la migration.

Les calculs sont dans l’ordre indiqué ci-dessus. Un serveur de machine ne passe à une étape ultérieure que s’il réussit la précédente. Par exemple, si un serveur échoue à l’étape relative à l’état de préparation pour Azure, il est marqué comme inapproprié pour Azure. Les calculs de dimensionnement et de coût ne sont pas effectués pour ce serveur.

## <a name="whats-in-an-azure-vm-assessment"></a>Qu’est-ce qu’une évaluation de machine virtuelle Azure ?

Le contenu d’une évaluation de machine virtuelle Azure dans Server Assessment est le suivant :

**Propriété** | **Détails**
--- | ---
**Emplacement cible** | Emplacement vers lequel vous souhaitez migrer. Évaluation de serveur prend actuellement en charge les régions Azure cibles suivantes :<br/><br/> Allemagne Centre, Allemagne Nord-Est, Asie Est, Asie Sud-Est, Australie Est, Australie Sud-Est, Brésil Sud, Canada Centre, Canada Est, Chine Est, Chine Nord, Corée Centre, Corée Sud, Europe Nord, Europe Ouest, Inde Centre, Inde Ouest, Inde Sud, Japon Est, Japon Ouest, Royaume-Uni Ouest, Royaume-Uni Sud, US Gov Arizona, US Gov Texas, US Gov Virginie, USA Centre, USA Centre Nord, USA Centre-Ouest, USA Centre Sud, USA Est, USA Est 2, USA Ouest et USA Ouest 2.
**Disque de stockage cible (dimensionnement tel quel)** | Type de disque à utiliser pour le stockage dans Azure. <br/><br/> Spécifie le disque de stockage cible en tant que disque managé Premium, disque managé SSD Standard ou disque managé HDD Standard.
**Disque de stockage cible (dimensionnement basé sur les performances)** | Spécifie le type du disque de stockage cible en tant que disque automatique, managé Premium, managé HDD Standard ou managé SSD Standard.<br/><br/> **Automatique** : La recommandation de disque est basée sur les données de performances des disques, c’est-à-dire les IOPS et le débit.<br/><br/>**Premium ou Standard** :  L’évaluation recommande une référence SKU de disque au sein du type de stockage sélectionné.<br/><br/> Si vous souhaitez un contrat de niveau de service (SLA) de machine virtuelle à une seule instance de 99,9 %, envisagez d’utiliser des disques managés Premium. Cette utilisation permet de s’assurer que tous les disques de l’évaluation sont recommandés comme disques managés Premium.<br/><br/> Azure Migrate prend uniquement en charge les disques managés pour l’évaluation de la migration.
**Azure Reserved VM Instances** | Spécifie des [instances réservées](https://azure.microsoft.com/pricing/reserved-vm-instances/) afin que les estimations de coût dans l’évaluation les prennent en compte.<br/><br/> Quand vous sélectionnez « Instances réservées », les propriétés « Remise (%) » et « Durée de fonctionnement de la machine virtuelle » ne sont pas applicables.<br/><br/> Azure Migrate prend actuellement en charge Azure Reserved VM Instances uniquement pour les offres de paiement à l’utilisation.
**Critère de dimensionnement** | Utilisé pour le dimensionnement adéquat de la machine virtuelle Azure.<br/><br/> Utilise le dimensionnement tel quel ou le dimensionnement basé sur les performances.
**Historique des performances** | Utilisé avec le dimensionnement basé sur les performances. L’historique des performances spécifie la durée utilisée lors de l’évaluation des données de performances.
**Utilisation en centile** | Utilisé avec le dimensionnement basé sur les performances. L’utilisation en centile spécifie la valeur de centile de l’échantillon de performances utilisé pour le dimensionnement adéquat.
**Séries de machine virtuelle** | Série de machines virtuelles Azure que vous souhaitez prendre en compte pour le dimensionnement adéquat. Par exemple, si vous ne disposez pas d’un environnement de production nécessitant des machines virtuelles de série A dans Azure, vous pouvez exclure la série A de la liste de séries.
**Facteur de confort** | Mémoire tampon utilisée pendant l’évaluation. Elle est appliquée aux données de l’UC, de la RAM, du disque et du réseau pour les machines virtuelles. Il prend en compte les problèmes, tels que l’utilisation saisonnière, l’historique des performances de courte durée et l’augmentation probable de l’utilisation future.<br/><br/> Par exemple, une machine virtuelle de 10 cœurs avec 20 % d’utilisation correspond normalement à une machine virtuelle à deux cœurs. Avec un facteur de confort de 2, le résultat est une machine virtuelle à quatre cœurs.
**Offer** | [Offre Azure](https://azure.microsoft.com/support/legal/offer-details/) dans laquelle vous êtes inscrit. Évaluation de serveur estime le coût de cette offre.
**Devise** | Devise de facturation de votre compte.
**Remise (%)** | Toute remise propre à un abonnement que vous recevez en plus de l’offre Azure. Le paramètre par défaut est 0 %.
**Durée de fonctionnement de la machine virtuelle** | Durée en jours par mois et heures par jour pour les machines virtuelles Azure qui ne s’exécutent pas en continu. Les estimations de coût sont basées sur cette durée.<br/><br/> Les valeurs par défaut sont de 31 jours par mois et de 24 heures par jour.
**Azure Hybrid Benefit** | Spécifie si vous disposez de Software Assurance et que vous êtes éligible à [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/). Si le paramètre a la valeur par défaut « Oui », les tarifs Azure pour les systèmes d’exploitation autres que Windows sont pris en compte pour les machines virtuelles Windows.
**Abonnement Contrat Entreprise** | Spécifie qu’un abonnement Contrat Entreprise (EA) est utilisé pour l’estimation des coûts. Prend en compte la remise applicable à l’abonnement. <br/><br/> Conserve les paramètres par défaut pour les instances réservées, la remise (en %) et les propriétés de durée de bon fonctionnement des machines virtuelles.


[Passez en revue les meilleures pratiques](best-practices-assessment.md) pour la création d’une évaluation avec Évaluation de serveur.

## <a name="calculate-readiness"></a>Calculer l’état de préparation

Toutes les machines ne conviennent pas pour s’exécuter dans Azure. Une évaluation de machine virtuelle Azure évalue toutes les machines locales et leur attribue une catégorie d’état de préparation.

- **Disponible pour Azure** : la machine peut être migrée telle quelle vers Azure sans aucune modification. Elle démarrera dans Azure avec une prise en charge complète d’Azure.
- **Préparé pour Azure sous condition**: La machine peut démarrer dans Azure, mais risque de ne pas être totalement prise en charge. Par exemple, Azure ne prend pas en charge un ordinateur qui exécute une ancienne version de Windows Server. Vous devez faire preuve de vigilance avant de migrer ces machines vers Azure. Pour résoudre les problèmes relatifs à l’état de préparation, suivez les instructions de correction suggérées par l’évaluation.
- **Non disponible pour Azure** : La machine ne démarrera pas dans Azure. Par exemple, si le disque d’une machine locale stocke plus de 64 To, Azure ne peut pas héberger la machine. Suivez les instructions de correction afin de résoudre le problème avant la migration.
- **État de la préparation inconnu** : Azure Migrate ne peut pas déterminer l’état de préparation d’une machine en raison de métadonnées insuffisantes.

Pour calculer l’état de préparation, Server Assessment examine les propriétés de la machine et les paramètres du système d’exploitation résumés dans les tableaux suivants.

### <a name="machine-properties"></a>Propriétés de machine virtuelle

Pour une évaluation de machine virtuelle Azure, Server Assessment examine les propriétés suivantes d’une machine virtuelle locale pour déterminer si elle peut s’exécuter sur Machines virtuelles Azure.

Propriété | Détails | État de préparation pour Azure
--- | --- | ---
**Type de démarrage** | Azure prend en charge les machines virtuelles avec un type de démarrage BIOS, et non UEFI. | Préparé pour Azure sous condition si le type de démarrage est UEFI
**Cœurs** | Chaque ordinateur ne doit pas comporter plus de 128 cœurs, ce qui correspond au nombre maximal pris en charge par une machine virtuelle Azure.<br/><br/> Si l’historique des performances est disponible, Azure Migrate prend en considération les cœurs utilisés pour la comparaison. Si les paramètres de l’évaluation spécifient le facteur de confort, le nombre de cœurs utilisés est multiplié par le facteur de confort.<br/><br/> En l’absence d’historique des performances, Azure Migrate utilise les cœurs alloués pour appliquer le facteur de confort. | Préparé pour Azure si le nombre de cœurs est dans la limite
**RAM** | Chaque ordinateur ne doit pas avoir plus de 3 892 Go de RAM, ce qui correspond à la taille maximale qu’une machine virtuelle Standard_M128m de série M Azure&nbsp;<sup>2</sup> prend en charge. [Plus d’informations](../virtual-machines/sizes.md)<br/><br/> Si l’historique des performances est disponible, Azure Migrate prend en considération la RAM utilisée pour la comparaison. Si un facteur de confort est spécifié, la RAM utilisée est multipliée par le facteur de confort.<br/><br/> En l’absence d’historique, la RAM allouée est utilisée pour appliquer un facteur de confort.<br/><br/> | Préparé pour Azure si la quantité de RAM est comprise dans la limite
**Disque de stockage** | La taille allouée d’un disque doit être inférieure ou égale à 32 To. Bien qu’Azure prenne en charge des disques de 64 To avec des disques SSD Ultra Azure, Azure Migrate : Évaluation de serveur vérifie actuellement 32 To comme limite de taille de disque, car il ne prend pas encore en charge les SSD Ultra. <br/><br/> Le nombre de disques attachés à l’ordinateur, y compris le disque du système d’exploitation, doit être inférieur ou égal à 65. | Préparé pour Azure si la taille et le nombre de disques sont dans les limites
**Mise en réseau** | Une machine ne doit pas avoir plus de 32 interfaces réseau (NIC) associées. | Préparé pour Azure si le nombre d’interfaces réseau est dans la limite

### <a name="guest-operating-system"></a>Système d’exploitation invité

Pour une évaluation de machine virtuelle Azure, outre la vérification des propriétés des machines virtuelles, Server Assessment examine le système d’exploitation invité d’une machine pour déterminer s’il peut s’exécuter sur Azure.

> [!NOTE]
> Afin de gérer l’analyse de l’invité pour les machines virtuelles VMware, Évaluation de serveur utilise le système d’exploitation spécifié pour la machine virtuelle dans vCenter Server. Toutefois, vCenter Server ne fournit pas la version du noyau pour les systèmes d’exploitation de machines virtuelles Linux. Pour découvrir la version, vous devez configurer la [découverte d’application](./how-to-discover-applications.md). L’appliance découvre ensuite les informations de version à l’aide des informations d’identification de l’invité que vous spécifiez lors de la configuration de la découverte d’application.


Évaluation de serveur utilise la logique suivante pour identifier l’état de préparation pour Azure en fonction du système d’exploitation :

**Système d’exploitation** | **Détails** | **État de préparation pour Azure**
--- | --- | ---
Windows Server 2016 avec tous les Service Packs | Azure assure un support complet. | Préparé pour Azure.
Windows Server 2012 R2 et tous les Service Packs | Azure assure un support complet. | Préparé pour Azure.
Windows Server 2012 et tous les Service Packs | Azure assure un support complet. | Préparé pour Azure.
Windows Server 2008 R2 avec tous les Service Packs | Azure assure un support complet.| Préparé pour Azure.
Windows Server 2008 (32 bits et 64 bits) | Azure assure un support complet. | Préparé pour Azure.
Windows Server 2003 et Windows Server 2003 R2 | Ces systèmes d’exploitation ont dépassé leur date de fin de support, et leur prise en charge dans Azure nécessite un [contrat de support personnalisé (CSA)](/troubleshoot/azure/virtual-machines/server-software-support). | Préparé pour Azure sous condition Envisagez de mettre à niveau le système d’exploitation avant de migrer vers Azure.
Windows 2000, Windows 98, Windows 95, Windows NT, Windows 3.1 et MS-DOS | La date de fin de support de ces systèmes d’exploitation est passée. La machine peut démarrer dans Azure, mais Azure ne fournit aucune prise en charge du système d’exploitation. | Préparé pour Azure sous condition Nous vous recommandons de mettre à niveau le système d’exploitation avant de migrer vers Azure.
Windows 7, Windows 8 et Windows 10 | Azure assure un support avec un [abonnement Visual Studio uniquement](../virtual-machines/windows/client-images.md). | Préparé pour Azure sous condition
Windows 10 Pro | Azure fournit la prise en charge avec les [droits d’hébergement multilocataire](../virtual-machines/windows/windows-desktop-multitenant-hosting-deployment.md). | Préparé pour Azure sous condition
Windows Vista et Windows XP Professionnel | La date de fin de support de ces systèmes d’exploitation est passée. La machine peut démarrer dans Azure, mais Azure ne fournit aucune prise en charge du système d’exploitation. | Préparé pour Azure sous condition Nous vous recommandons de mettre à niveau le système d’exploitation avant de migrer vers Azure.
Linux | Consultez les [systèmes d’exploitation Linux](../virtual-machines/linux/endorsed-distros.md) qu’Azure approuve. D’autres systèmes d’exploitation Linux peuvent démarrer dans Azure. Toutefois, nous vous recommandons de mettre à niveau le système d’exploitation vers une version approuvée avant d’effectuer la migration vers Azure. | Disponible pour Azure si la version est approuvée.<br/><br/>Préparé pour Azure sous condition si la version n’est pas approuvée.
Autres systèmes d’exploitation tels qu’Oracle Solaris, Apple macOS et FreeBSD | Azure n’approuve pas ces systèmes d’exploitation. La machine peut démarrer dans Azure, mais Azure ne fournit aucune prise en charge du système d’exploitation. | Préparé pour Azure sous condition Nous vous recommandons d’installer un système d’exploitation pris en charge avant de migrer vers Azure.  
Système d’exploitation spécifié comme **Autre** dans vCenter Server | Azure Migrate ne peut pas identifier le système d’exploitation dans ce cas. | État de la préparation inconnu. Assurez-vous qu’Azure prend en charge le système d’exploitation exécuté au sein de la machine virtuelle.
Systèmes d’exploitation 32 bits | La machine peut démarrer dans Azure, mais Azure peut ne pas fournir une prise en charge complète. | Préparé pour Azure sous condition Envisagez une mise à niveau vers un système d’exploitation 64 bits avant d’effectuer la migration vers Azure.

## <a name="calculating-sizing"></a>Calcul du dimensionnement

Une fois qu’une machine est marquée comme prête pour Azure, Server Assessment effectue des recommandations de dimensionnement dans le cadre de l’évaluation de machine virtuelle Azure. Ces recommandations identifient la référence SKU de la machine virtuelle et du disque Azure. Les calculs de dimensionnement varient selon que vous utilisez un dimensionnement local tel quel ou un dimensionnement basé sur les performances.

### <a name="calculate-sizing-as-is-on-premises"></a>Calculer le dimensionnement (local tel quel)

 Si vous utilisez le dimensionnement local tel quel, Server Assessment ne prend pas en compte l’historique des performances des machines virtuelles et des disques dans le cadre de l’évaluation de machine virtuelle Azure.

- **Dimensionnement du calcul** : Évaluation de serveur alloue une référence SKU de machine virtuelle Azure basée sur la taille allouée localement.
- **Dimensionnement du stockage et du disque** : Évaluation de serveur examine le type de stockage spécifié dans les propriétés d’évaluation et recommande le type de disque en conséquence. Les types de stockage possibles sont HDD Standard, SSD Standard et Premium. Le type de stockage par défaut est Premium.
- **Dimensionnement du réseau** : Server Assessment prend en compte la carte réseau sur la machine locale.

### <a name="calculate-sizing-performance-based"></a>Calculer le dimensionnement (basé sur les performances)

Si vous utilisez le dimensionnement basé sur les performances dans le cadre d’une évaluation de machine virtuelle Azure, Server Assessment effectue des recommandations de dimensionnement comme suit :

- Server Assessment prend en compte l’historique des performances de la machine afin d’identifier la taille de la machine virtuelle et le type de disque dans Azure.
- Si vous importez des serveurs à l’aide d’un fichier CSV, les valeurs que vous spécifiez sont utilisées. Cette méthode est particulièrement utile si vous avez suralloué la machine locale, que l’utilisation est faible et que vous voulez dimensionner correctement la machine virtuelle Azure afin de réduire les coûts.
- Si vous ne souhaitez pas utiliser les données de performances, réinitialisez les critères de dimensionnement sur un dimensionnement local tel quel, comme décrit dans la section précédente.

#### <a name="calculate-storage-sizing"></a>Calculer le dimensionnement du stockage

Pour le dimensionnement du stockage dans le cadre d’une évaluation de machine virtuelle Azure, Azure Migrate tente de mapper chaque disque attaché à la machine à un disque Azure. Le dimensionnement fonctionne comme suit :

1. Server Assessment ajoute les IOPS en lecture et en écriture d’un disque pour obtenir le nombre total d’IOPS requis. De même, il ajoute les valeurs de débit de lecture et d’écriture pour obtenir le débit total de chaque disque. Dans le cas des évaluations basées sur l’importation, vous avez la possibilité de fournir le nombre total d’IOPS, le débit total et le nombre total de disques dans le fichier importé sans spécifier de paramètres de disque individuels. Si vous procédez ainsi, le dimensionnement de disque individuel est ignoré et les données fournies sont utilisées directement pour calculer le dimensionnement puis sélectionner une référence SKU de machine virtuelle appropriée.

1. Si vous avez spécifié le type de stockage sur automatique, le type sélectionné est basé sur les valeurs réelles d’IOPS et de débit. Évaluation de serveur détermine si le disque doit être mappé à un disque HDD Standard, SSD Standard ou Premium dans Azure. Si le type de stockage est défini sur l’un de ces types de disques, Évaluation de serveur tente de trouver une référence SKU de disque au sein du type de stockage sélectionné.
1. Les disques sont sélectionnés comme suit :
    - Si Server Assessment ne parvient pas à trouver un disque avec les IOPS et le débit nécessaires, il marque la machine comme ne convenant pas pour Azure.
    - Si Server Assessment trouve un ensemble de disques adaptés, il sélectionne ceux qui prennent en charge l’emplacement spécifié dans les paramètres d’évaluation.
    - S’il existe plusieurs disques éligibles, Server Assessment sélectionne le disque dont le coût est le plus faible.
    - Si les données de performances d’un disque ne sont pas disponibles, la taille du disque de configuration est utilisée pour trouver un disque SSD Standard dans Azure.

#### <a name="calculate-network-sizing"></a>Calculer le dimensionnement du réseau

Pour une évaluation de machine virtuelle Azure, Server Assessment essaie de trouver une machine virtuelle Azure qui prend en charge le nombre de cartes réseau attachées à la machine locale et les performances requises pour ces cartes réseau.

- Pour obtenir les performances réseau réelles de la machine virtuelle locale, Évaluation de serveur agrège le débit de transmission de données à partir de la machine (réseau sortant) pour toutes les cartes réseau. Il applique ensuite le facteur de confort. Il utilise la valeur qui en résulte pour trouver une machine virtuelle Azure qui peut prendre en charge les performances réseau requises.
- Outre les performances du réseau, Évaluation de serveur détermine également si la machine virtuelle Azure peut prendre en charge le nombre requis de cartes réseau.
- Si aucune donnée de performances réseau n’est disponible, Évaluation de serveur considère uniquement le nombre de cartes réseau pour le dimensionnement de la machine virtuelle.

#### <a name="calculate-compute-sizing"></a>Calculer le dimensionnement du calcul

Après avoir calculé les exigences de stockage et de réseau, Évaluation de serveur prend en compte les besoins en UC et en RAM pour trouver une taille de machine virtuelle appropriée dans Azure.

- Azure Migrate examine les cœurs et la RAM réellement utilisés pour rechercher une taille de machine virtuelle Azure appropriée.
- Si aucune taille adaptée n’est trouvée, la machine est marquée comme inadaptée à Azure.
- Si une taille adaptée est trouvée, Azure Migrate applique les calculs de mise en réseau et de stockage. Il applique ensuite les paramètres d’emplacement et de niveau tarifaire de façon à obtenir la recommandation de taille de machine virtuelle finale.
- S’il existe plusieurs tailles de machine virtuelle Azure éligibles, celle dont le coût est le plus faible est recommandée.

## <a name="confidence-ratings-performance-based"></a>Niveaux de confiance (basés sur les performances)

Chaque évaluation de machine virtuelle Azure basée sur les performances dans Azure Migrate est associée à un niveau de confiance. Le niveau de confiance est compris entre une étoile (le plus bas) et cinq étoiles (le plus haut). Le niveau de confiance vous aide à estimer la fiabilité des recommandations de taille fournies par Azure Migrate.

- Le niveau de confiance est attribué à une évaluation. Le niveau de confiance est basé sur la disponibilité des points de données nécessaires au calcul de l’évaluation.
- Pour le dimensionnement basé sur les performances, Server Assessment a besoin :
    - Les données d’utilisation de l’UC et de la RAM de la machine virtuelle.
    - Les données IOPS et de débit du disque pour chaque disque attaché à la machine virtuelle.
    - Les E/S réseau pour gérer le dimensionnement basé sur les performances pour chaque carte réseau attachée à une machine virtuelle.

Si l’un de ces chiffres d’utilisation n’est pas disponible, les recommandations de taille peuvent ne pas être fiables.

> [!NOTE]
> Aucun niveau de confiance n’est attribué aux serveurs évalués à l’aide d’un fichier CSV importé. Aucun niveau de confiance ne s'applique non plus aux évaluations locales en l'état.

### <a name="ratings"></a>Évaluations

Ce tableau indique les niveaux de confiance des évaluations, qui dépendent du pourcentage de points de données disponibles :

   **Disponibilité des points de données** | **Niveau de confiance**
   --- | ---
   0-20 % | 1 étoile
   21-40 % | 2 étoiles
   41-60 % | 3 étoiles
   61-80 % | 4 étoiles
   81-100 % | 5 étoiles

### <a name="low-confidence-ratings"></a>Niveaux de confiance faibles

Voici quelques raisons pour lesquelles une évaluation peut avoir un niveau de confiance faible :

- Vous n’avez pas profilé votre environnement pendant la durée pour laquelle vous créez l’évaluation. Par exemple, si vous créez l’évaluation avec une durée des performances définie sur un jour, vous devez attendre au moins un jour après le démarrage de la découverte pour que tous les points de données soient collectés.
- Certaines machines virtuelles étaient arrêtées pendant la période de calcul de l’évaluation. Si des machines virtuelles sont désactivées pendant une certaine durée, Server Assessment ne peut pas collecter les données de performances pour cette période.
- Certaines machines virtuelles ont été créées pendant la période de calcul de l’évaluation. Par exemple, supposons que vous avez créé une évaluation de l’historique des performances du dernier mois, mais que certaines machines virtuelles ont été créées il y a seulement une semaine. L’historique des performances des nouvelles machines virtuelles n’existera pas pour toute la durée.

> [!NOTE]
> Si le niveau de confiance d’une évaluation est inférieur à cinq étoiles, nous vous recommandons d’attendre au moins un jour pour que l’appliance profile l’environnement, puis de recalculer l’évaluation. Dans le cas contraire, le dimensionnement basé sur les performances peut ne pas être fiable. Dans ce cas, nous vous recommandons de faire passer l’évaluation au dimensionnement local.

## <a name="calculate-monthly-costs"></a>Calculer les coûts mensuels

Une fois les recommandations de dimensionnement terminées, une évaluation de machine virtuelle Azure dans Azure Migrate calcule les coûts de calcul et de stockage après la migration.

- **Calcul de coût** : Azure Migrate utilise la taille de machine virtuelle Azure recommandée et l’API de facturation Azure pour calculer le coût mensuel de la machine virtuelle.

    Le calcul prend en compte les éléments suivants :
    - Système d’exploitation
    - Software Assurance
    - Instances réservées
    - Durée de fonctionnement de la machine virtuelle
    - Emplacement
    - Paramètres de devise

    Évaluation de serveur agrège le coût de toutes les machines pour calculer le coût de calcul mensuel total.

- **Coût de stockage** : Le coût de stockage mensuel d’une machine est calculé en additionnant le coût mensuel de tous les disques qui sont attachés à la machine.

    Server Assessment calcule les coûts totaux mensuels du stockage en additionnant les coûts de stockage de toutes les machines. Actuellement, le calcul ne prend pas en compte les offres spécifiées dans les paramètres d’évaluation.

Les coûts sont affichés dans la devise spécifiée dans les paramètres d’évaluation.

## <a name="next-steps"></a>Étapes suivantes

[Passez en revue](best-practices-assessment.md) les meilleures pratiques pour la création d’évaluations. 

- En savoir plus sur l’exécution d’évaluations pour les [machines virtuelles VMware](./tutorial-discover-vmware.md), les [machines virtuelles Hyper-V](./tutorial-discover-hyper-v.md) et les [serveurs physiques](./tutorial-discover-physical.md).
- En savoir plus sur l’évaluation de serveurs [importés avec un fichier CSV](./tutorial-discover-import.md).
- En savoir plus sur la configuration de la [visualisation des dépendances](concepts-dependency-visualization.md).