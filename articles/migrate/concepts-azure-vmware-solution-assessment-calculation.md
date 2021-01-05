---
title: Calculs d’évaluations AVS dans Azure Migrate | Microsoft Docs
description: Offre une vue d’ensemble des calculs d’évaluation AVS dans le service Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 06/25/2020
ms.openlocfilehash: 67d4137a21753b221e17a1effde35bc1b89600d3
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753805"
---
# <a name="server-assessment-overview-migrate-to-azure-vmware-solution"></a>Vue d’ensemble de l’évaluation du serveur (migrer vers Azure VMware Solution)

[Azure Migrate](migrate-services-overview.md) fournit un hub central pour effectuer le suivi de la découverte, de l’évaluation et de la migration de vos applications et charges de travail locales. Il effectue également le suivi de vos instances de cloud privées et publiques vers Azure. Le hub fournit des outils Azure Migrate pour l’évaluation et la migration ainsi que des offres de fournisseurs de logiciels indépendants (ISV) tiers.

Server Assessment est un outil d’Azure Migrate qui évalue les serveurs locaux pour la migration vers les machines virtuelles IaaS Azure et Azure VMware Solution (AVS). Cet article donne des informations sur le calcul des évaluations Azure VMware Solution (AVS).

> [!NOTE]
> L’évaluation Azure VMware Solution (AVS) est actuellement en version préliminaire et ne peut être créée que pour des machines virtuelles VMware.

## <a name="types-of-assessments"></a>Types d'évaluations

Les évaluations que vous créez à l'aide de l'outil Évaluation de serveur correspondent à un instantané ponctuel des données. Vous pouvez créer deux types d’évaluation à l’aide d’Azure Migrate : Server Assessment.

**Type d’évaluation** | **Détails**
--- | --- 
**Microsoft Azure** | Évaluations pour migrer vos serveurs locaux vers des machines virtuelles Azure. <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md), [machines virtuelles Hyper-V](how-to-set-up-appliance-hyper-v.md) et [serveurs physiques](how-to-set-up-appliance-physical.md) locaux pour la migration vers Azure à l’aide de ce type d’évaluation. [En savoir plus](concepts-assessment-calculation.md)
**Azure VMware Solution (AVS)** | Évaluations pour migrer vos serveurs locaux vers [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md) locales pour la migration vers Azure VMware Solution (AVS) avec ce type d’évaluation. [En savoir plus](concepts-azure-vmware-solution-assessment-calculation.md)

L’évaluation Azure VMware Solution (AVS) dans Server Assessment fournit deux options de critères de dimensionnement :

**Évaluation** | **Détails** | **Données**
--- | --- | ---
**Basée sur les performances** | Évaluations basées sur les données de performances collectées sur des machines virtuelles locales. | **Taille de nœud recommandée**: En fonction des données d’utilisation du processeur et de la mémoire, ainsi que du type de nœud, du type de stockage et du paramètre FTT que vous sélectionnez pour l’évaluation.
**Localement** | Évaluations basées sur le dimensionnement local. | **Taille de nœud recommandée**: En fonction de la taille des machines virtuelles locales, ainsi que du type de nœud, du type de stockage, et du paramètre FTT que vous sélectionnez pour l’évaluation.

## <a name="how-do-i-run-an-assessment"></a>Comment exécuter une évaluation ?

Une évaluation peut être réalisée de différentes façons.

- Évaluation des machines à l’aide des métadonnées du serveur collectées par une appliance Azure Migrate légère. L’appliance découvre des machines locales. Elle envoie ensuite les métadonnées et les données de performances des machines à Azure Migrate.
- Évaluation des machines à l’aide des métadonnées du serveur importées au format CSV.

## <a name="how-do-i-assess-with-the-appliance"></a>Comment procéder à une évaluation à l'aide de l'appliance ?

Si vous déployez une appliance Azure Migrate pour découvrir des serveurs locaux, procédez comme suit :

1. Configurez Azure et votre environnement local de manière à utiliser l’outil Évaluation de serveur.
2. Pour votre première évaluation, créez un projet Azure et ajoutez-y l’outil Évaluation de serveur.
3. Déployez une appliance Azure Migrate légère. Celle-ci découvre en continu les machines locales et envoie leurs métadonnées et leurs données de performances à Azure Migrate. Déployez l’appliance en tant que machine virtuelle. Vous n’avez rien à installer sur les machines que vous souhaitez évaluer.

Une fois que l’appliance a commencé la détection des machines, vous pouvez rassembler les machines que vous souhaitez évaluer au sein d’un groupe et exécuter une évaluation du groupe avec le type d’évaluation **Server Assessment**.

Créez votre première évaluation Azure VMware Solution (AVS) en suivant les étapes [ici](how-to-create-azure-vmware-solution-assessment.md).

## <a name="how-do-i-assess-with-imported-data"></a>Comment procéder à une évaluation à l'aide de données importées ?

Si vous évaluez des serveurs à l’aide d’un fichier CSV, aucune appliance n’est nécessaire. Au lieu de cela, procédez comme suit :

1. Configurez Azure de manière à utiliser l’outil Évaluation de serveur.
2. Pour votre première évaluation, créez un projet Azure et ajoutez-y l’outil Évaluation de serveur.
3. Téléchargez un modèle CSV et ajoutez-y les données du serveur.
4. Importez le modèle dans l’outil Évaluation de serveur.
5. Détectez les serveurs ajoutés avec l’importation, rassemblez-les dans un groupe et exécutez une évaluation pour le groupe avec le type d’évaluation **Azure VMware Solution (AVS)** .

## <a name="what-data-does-the-appliance-collect"></a>Quelles données l’appliance collecte-t-elle ?

Si vous utilisez l'appliance Azure Migrate à des fins d'évaluation, apprenez-en plus sur les métadonnées et les données de performance qui sont collectées pour [VMware](migrate-appliance.md#collected-data---vmware).

## <a name="how-does-the-appliance-calculate-performance-data"></a>Comment l'appliance calcule-t-elle les données de performances ?

Si vous utilisez l’appliance à des fins de découverte, elle collecte des données de performances pour les paramètres de calcul en procédant comme suit :

1. L’appliance collecte un point d’échantillonnage en temps réel.

    - **Machines virtuelles VMware** : Un point d’échantillonnage est collecté toutes les 20 secondes.

2. L’appliance combine les points d’échantillonnage pour créer un point de données unique toutes les 10 minutes. Pour créer le point de données, l’appliance sélectionne les valeurs maximales de tous les échantillons. Elle envoie ensuite le point de données à Azure.
3. Évaluation de serveur stocke tous les points d’échantillonnage de 10 minutes pour le dernier mois.
4. Lorsque vous créez une évaluation, Évaluation de serveur identifie le point de données approprié à utiliser pour le dimensionnement adéquat. L’identification est basée sur les valeurs de centile pour *Historique des performances* et *Utilisation en centile*.

    - Par exemple, si l’historique des performances est d’une semaine et que l’utilisation en centile est le 95e centile, Évaluation de serveur trie les points d’échantillonnage de 10 minutes pour la dernière semaine. Il les trie dans l’ordre croissant et sélectionne la valeur du 95e centile pour le dimensionnement adéquat.
    - Le 95e centile vous permet d’ignorer les valeurs hors norme qui peuvent être incluses quand vous choisissez le 99e centile.
    - Si vous souhaitez choisir l’utilisation maximale de la période et que vous ne souhaitez pas ignorer les valeurs hors norme, sélectionnez le 99e centile comme utilisation en centile.

5. Cette valeur est multipliée par le facteur de confort pour obtenir les données réelles d’utilisation de performances pour ces mesures collectées par l’appliance :

    - Utilisation du processeur
    - Utilisation de la RAM
    - IOPS de disque (lecture et écriture)
    - Débit de disque (lecture et écriture)
    - Débit réseau (entrant et sortant)

Les données de performances suivantes sont collectées, mais ne sont pas utilisées dans les suggestions de dimensionnement pour les évaluations AVS :
  - Les données IOPS et de débit du disque pour chaque disque attaché à la machine virtuelle.
  - Les E/S réseau pour gérer le dimensionnement basé sur les performances pour chaque carte réseau attachée à une machine virtuelle.

## <a name="how-are-avs-assessments-calculated"></a>Comment les évaluations AVS sont-elles calculées ?

Évaluation de serveur utilise les métadonnées et les données de performances des machines locales pour calculer les évaluations. Si vous déployez l’appliance Azure Migrate, l’évaluation utilise les données collectées par l’appliance. Toutefois, si vous exécutez une évaluation importée à l’aide d’un fichier CSV, vous devez fournir les métadonnées pour le calcul.

Les calculs se font dans les trois étapes suivantes :

1. **Calculer la préparation d’Azure VMware Solution (AVS)** : Si les machines virtuelles locales sont adaptées à la migration vers Azure VMware Solution (AVS).
2. **Calculer le nombre de nœuds AVS et l’utilisation sur les nœuds**: Estimation du nombre de nœuds AVS requis pour exécuter les machines virtuelles et prévision de l’utilisation du processeur, de la mémoire et du stockage sur tous les nœuds.
3. **Estimation des coûts mensuels** : Les coûts mensuels estimés pour tous les nœuds Azure VMware Solution (AVS) exécutant les machines virtuelles locales.

Les calculs sont dans l’ordre indiqué ci-dessus. Un serveur de machine ne passe à une étape ultérieure que s’il réussit la précédente. Par exemple, si un serveur échoue à l’étape relative à l’état de préparation pour AVS, il est marqué comme inapproprié pour Azure. Les calculs de dimensionnement et de coût ne sont pas effectués pour ce serveur

## <a name="whats-in-an-azure-vmware-solution-avs-assessment"></a>Qu’est-ce qu’une évaluation Azure VMware Solution (AVS) ?

Voici le contenu d’une évaluation AVS dans Server Assessment :


| **Propriété** | **Détails** 
| - | - 
| **Emplacement cible** | Spécifie l’emplacement du cloud privé AVS vers lequel vous souhaitez effectuer la migration.<br/><br/> L’évaluation AVS dans Server Assessment prend actuellement en charge les régions cibles suivantes : USA Est, Europe Ouest, USA Ouest. 
| **Type de stockage** | Spécifie le moteur de stockage à utiliser dans AVS.<br/><br/> Les évaluations AVS prennent uniquement en charge le vSAN comme type de stockage par défaut. 
**Instances réservées (RI)** | Cette propriété vous permet de spécifier des instances réservées dans AVS. Les instances réservées non prises en charge pour les nœuds AVS. 
**Type de nœud** | Spécifie le [type de nœud AVS](../azure-vmware/concepts-private-clouds-clusters.md) utilisé pour mapper les machines virtuelles locales. Le type de nœud par défaut est AV36. <br/><br/> Azure Migrate recommandera un nombre requis de nœuds pour les machines virtuelles à migrer vers AVS. 
**Paramètre FTT, niveau RAID** | Spécifie les combinaisons de Failure to Tolerate (Échec à tolérer) et RAID applicables. L’option FTT sélectionnée associée à la configuration de disque de machine virtuelle locale détermine le stockage vSAN total requis dans AVS. 
**Critère de dimensionnement** | Définit les critères à utiliser pour *dimensionner correctement* les machines virtuelles pour AVS. Vous pouvez effectuer un dimensionnement *en fonction des performances* ou dimensionner les machines virtuelles *comme des machines locales* sans tenir compte de l’historique des performances. 
**Historique des performances** | Définit la durée à prendre en compte pour évaluer les données de performances des machines. Cette propriété s’applique uniquement lorsque le critère de dimensionnement est *basé sur les performances*. 
**Utilisation en centile** | Spécifie la valeur de centile de l’échantillon de performance devant être pris en compte pour le dimensionnement adéquat. Cette propriété s’applique uniquement lorsque le dimensionnement est basé sur les performances.
**Facteur de confort** | Azure Migrate Server Assessment considère une mémoire tampon (facteur de confort) au cours de l’évaluation. Cette mémoire tampon est appliquée sur des données d’utilisation de l’ordinateur pour les machines virtuelles (processeur, mémoire, disque et réseau). Le facteur de confort prend en compte les problèmes, tels que l’utilisation saisonnière, l’historique des performances de courte durée et l’augmentation probable de l’utilisation future.<br/><br/> Par exemple, une machine virtuelle de 10 cœurs avec 20 % d’utilisation correspond normalement à une machine virtuelle à 2 cœurs. Toutefois, avec un facteur de confort de 2.0x, le résultat est une machine virtuelle de 4 cœurs. 
**Offer** | Affiche l'[offre Azure](https://azure.microsoft.com/support/legal/offer-details/) à laquelle vous êtes inscrit. Azure Migrate estime le coût en conséquence.
**Devise** | Affiche la devise de facturation de votre compte. 
**Remise (%)** | Répertorie les remises spécifiques à l’abonnement que vous recevez en plus de l’offre Azure. Le paramètre par défaut est 0 %. 
**Azure Hybrid Benefit** | Spécifie si vous disposez de Software Assurance et que vous êtes éligible à [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/). Bien qu’il n’ait aucun impact sur les prix des solutions VMware Azure en raison du prix basé sur les nœuds, les clients peuvent toujours appliquer les licences de système d’exploitation locales (basées sur Microsoft) dans AVS à l’aide d’Azure Hybrid Benefits. D’autres fournisseurs de systèmes d’exploitation logiciels devront fournir leurs propres termes de licence, tels que RHEL, par exemple. 
**Oversubscription pour les processeurs virtuels** | Spécifie le rapport entre le nombre de cœurs virtuels lié à un cœur physique dans le nœud AVS. La valeur par défaut dans les calculs est de 4 processeurs virtuels pour 1 cœur physique dans AVS. <br/><br/> Les utilisateurs de l’API peuvent définir cette valeur comme un entier. Notez que qu’un oversubscription des processeurs virtuels > 4:1 peut avoir un impact sur les charges de travail en fonction de leur utilisation du processeur. 


## <a name="azure-vmware-solution-avs-suitability-analysis"></a>Analyse de la pertinence d’VMware Solution (AVS)

Les évaluations AVS dans Server Assessment évaluent chaque machine virtuelle locale pour qu’elle soit adaptée à AVS en examinant les propriétés de la machine. Il attribue également chaque machine évaluée à l’une des catégories de pertinence suivantes :

- **Disponible pour Azure VMware Solution** : La machine peut être migrée telle quelle vers Azure (Azure VMware Solution) sans aucune modification. Elle démarrera dans AVS avec une prise en charge complète d’AVS.
- **Disponible sous conditions** : La machine virtuelle peut présenter des problèmes de compatibilité avec la version actuelle de vSphere, ainsi qu’exiger des outils VMware et ou d’autres paramètres avant que toutes les fonctionnalités de la machine virtuelle puissent être obtenues dans AVS.
- **Non disponible pour AVS** : La machine virtuelle ne démarrera pas dans AVS. Par exemple, si la machine virtuelle VMware locale est associée à un appareil externe tel qu’un CD-ROM, l’opération VMware VMotion échouera (si vous utilisez VMware VMotion).
- **État de la préparation inconnu** : Azure Migrate n’a pas pu déterminer l’état de préparation de la machine, car les métadonnées collectées dans l’environnement local sont insuffisantes.

Azure Migrate Server Assessment examine les propriétés de l’ordinateur pour déterminer la disponibilité Azure de la machine locale.

### <a name="machine-properties"></a>Propriétés de machine virtuelle

Server Assessment examine la propriété suivante de la machine virtuelle locale pour déterminer si elle peut s’exécuter sur Azure VMware Solution (AVS).


| **Propriété** | **Détails** | **État de préparation AVS** 
| - | - | - 
| **Protocole Internet** | AVS ne prend actuellement pas en charge l’adressage Internet IPv6.<br/><br/> Pour obtenir de l’aide sur une remédiation si votre machine est détectée avec IPv6, contactez votre équipe GBB AVS MSFT locale.| Prête pour le protocole Internet sous conditions


### <a name="guest-operating-system"></a>Système d’exploitation invité

Actuellement, les évaluations AVS n’examinent pas le système d’exploitation dans le cadre de l’analyse de la pertinence. Tous les systèmes d’exploitation exécutés sur des machines virtuelles locales sont susceptibles de s’exécuter sur la solution VMware Azure (AVS).

Outre les propriétés de machine virtuelle, Server Assessment examine le système d’exploitation invité des ordinateurs pour déterminer s’il peut s’exécuter sur Azure.


## <a name="sizing"></a>Dimensionnement

Une fois qu’une machine est marquée comme prête pour AVS, l’évaluation AVS dans Server Assessment fait des recommandations en matière de dimensionnement des nœuds, ce qui implique l’identification des exigences de machine virtuelle locales appropriées et la recherche du nombre total de nœuds AVS requis. Ces suggestions varient en fonction des propriétés d’évaluation spécifiées.

- Si le dimensionnement utilise le *dimensionnement basé sur les performances*, Azure Migrate prend en compte l’historique des performances de la machine pour effectuer les recommandations de dimensionnement appropriées pour AVS. Cette méthode est particulièrement utile si vous avez dépassé l’allocation de la machine virtuelle locale, mais que l’utilisation est faible et que vous souhaitez dimensionner correctement la machine virtuelle dans AVS pour réduire les coûts. Cette méthode vous permet d’optimiser les tailles lors de la migration.
- Si vous ne souhaitez pas prendre en compte les données de performances pour le dimensionnement des machines virtuelles et que vous souhaitez prendre les machines locales telles quelles dans AVS, vous pouvez définir les critères de dimensionnement de la même façon que *localement*. Ensuite, Server Assessment dimensionnera les machines virtuelles en fonction de la configuration locale sans tenir compte des données d’utilisation. 


### <a name="ftt-sizing-parameters"></a>Paramètres de dimensionnement FTT

Le moteur de stockage utilisé dans AVS est vSAN. Les politiques de stockage vSAN définissent les exigences de stockage pour vos machines virtuelles. Ces stratégies garantissent le niveau de service exigé pour vos machines virtuelles, car elles déterminent la façon dont le stockage est alloué à la machine virtuelle. Les combinaisons FTT-RAID disponibles sont : 

**Nombre de pannes tolérées (FTT, Failures to Tolerate)** | **Configuration RAID** | **Nombre minimal d’hôtes requis** | **Considérations sur la taille**
--- | --- | --- | --- 
1 | RAID-1 (Mise en miroir) | 3 | Une machine virtuelle de 100 Go consomme 200 Go.
1 | RAID-5 (Code d’effacement) | 4 | Une machine virtuelle de 100 Go consomme 133,33 Go
2 | RAID-1 (Mise en miroir) | 5 | Une machine virtuelle de 100 Go consomme 300 Go.
2 | RAID-6 (Code d’effacement) | 6 | Une machine virtuelle de 100 Go consomme 150 Go.
3 | RAID-1 (Mise en miroir) | 7 | Une machine virtuelle de 100 Go consomme 400 Go.

### <a name="performance-based-sizing"></a>Dimensionnement en fonction des performances

Pour le dimensionnement basé sur les performances, Server Assessment commence par les disques attachés à la machine virtuelle, suivis par les cartes réseau. Il mappe ensuite les spécifications de la machine virtuelle à un nombre approprié de nœuds pour AVS. L’appliance Azure Migrate profile l’environnement local pour collecter les données de performances du processeur, de la mémoire, des disques et de la carte réseau.

**Étapes de la collecte des données de performances :**

1. Pour les machines virtuelles VMware, l’appliance de Azure Migrate collecte un point d’échantillonnage en temps réel à chaque intervalle de 20 secondes. 
2. L’appliance cumule les points d’échantillonnage collectés toutes les 10 minutes et envoie la valeur maximale des 10 dernières minutes à Server Assessment.
3. Server Assessment stocke tous les points d’échantillonnage de 10 minutes pour le dernier mois. Ensuite, selon les propriétés d’évaluation spécifiées pour l’*historique des performances* et l’*utilisation du centile*, il identifie le point de données approprié à utiliser pour le dimensionnement adéquat. Par exemple, si l’historique des performances est défini sur 1 jour et que l’utilisation du centile est le 95e centile, Server Assessment utilise les points d’échantillonnage de 10 minutes pour le dernier jour, les trie dans l’ordre croissant et sélectionne la valeur du 95e centile pour le dimensionnement adéquat.
4. Cette valeur est multipliée par le facteur de confort pour obtenir les données d’utilisation des performances effectives pour chaque indicateur de performance (utilisation du processeur, utilisation de la mémoire, IOPS du disque (lecture et écriture), débit du disque (lecture et écriture) et débit réseau (entrée et sortie) collectées par l’appliance.

Une fois la valeur d’utilisation effective déterminée, le stockage, le réseau et le dimensionnement du calcul sont gérés comme suit.

**Dimensionnement du stockage** : Azure Migrate utilise l’espace disque de la machine virtuelle locale en tant que paramètre de calcul pour déterminer les exigences de stockage de vSAN AVS en plus du paramètre FTT sélectionné par le client. FTT - Les « échecs à tolérer » et l’exigence d’un nombre minimal de nœuds par option FTT déterminent le stockage vSAN total requis et la configuration requise pour les disques de machine virtuelle.

**Dimensionnement du réseau** : Actuellement, l’évaluation de serveur ne prend pas en considération les paramètres réseau pour les évaluations AVS.

**Dimensionnement du calcul** : Après avoir calculé les exigences de stockage, Server Assessment prend en compte les besoins des machines virtuelles en matière de processeurs et de mémoire pour déterminer le nombre de nœuds nécessaires pour AVS en fonction du type de nœud.

- En fonction des critères de dimensionnement, Server Assessment examine les données de la machine virtuelle basée sur les performances ou la configuration de la machine virtuelle locale. Le paramètre de facteur de confort permet de spécifier le facteur de croissance du cluster. Actuellement, l’hyperthreading est activé par défaut, ce qui signifie qu’un nœud à 36 cœurs aura 72 vCores. Une valeur de 4 vCores par cœur physique est utilisée pour déterminer les seuils de processeurs par cluster en utilisant le standard VMware de ne pas dépasser 80 % d’utilisation pour permettre la gestion de la maintenance ou des échecs sans compromettre la disponibilité du cluster. Il n’existe actuellement aucun remplacement disponible pour modifier les valeurs d’oversubscription et nous pourrions l’ajouter dans les futures versions.

### <a name="as-on-premises-sizing"></a>Dimensionnement « Localement »

Si vous utilisez *en tant que dimensionnement local*, Server Assessment ne prend pas en compte l’historique des performances des machines virtuelles et des disques. Au lieu de cela, il alloue des nœuds AVS en fonction de la taille allouée localement. Le type de stockage par défaut est vSAN dans AVS.

## <a name="confidence-ratings"></a>Niveaux de confiance

Chaque évaluation reposant sur les performances dans Azure Migrate est associée à un niveau de confiance, qui va de 1 étoile (le plus faible) à 5 étoiles (le plus élevé).

- Le niveau de confiance est assigné à une évaluation en fonction de la disponibilité des points de données nécessaires pour calculer l’évaluation.
- Le niveau de confiance d’une évaluation permet d’évaluer la fiabilité des recommandations de taille fournies par Azure Migrate.
- Les évaluations de confiance ne s’appliquent pas aux évaluations *locales*.
- Pour le dimensionnement basé sur les performances, les évaluations AVS dans Évaluation du Serveur nécessitent les données d’utilisation du processeur et de la mémoire de la machine virtuelle. Les données suivantes sont collectées, mais ne sont pas utilisées dans les recommandations de dimensionnement pour AVS :
  - Les données IOPS et de débit du disque pour chaque disque attaché à la machine virtuelle.
  - Les E/S réseau pour gérer le dimensionnement basé sur les performances pour chaque carte réseau attachée à une machine virtuelle.

  Si l’un de ces numéros d’utilisation n’est pas disponible dans vCenter Server, la suggestion de taille peut ne pas être fiable.

Selon le pourcentage de points de données disponibles, le niveau de confiance de l’évaluation se présente comme suit.


| **Disponibilité des points de données** | **Niveau de confiance** |
| - | - |
| 0-20 % | 1 étoile |
| 21-40 % | 2 étoiles |
| 41-60 % | 3 étoiles |
| 61-80 % | 4 étoiles |
| 81-100 % | 5 étoiles |

### <a name="low-confidence-ratings"></a>Niveaux de confiance faibles

Voici quelques raisons pour lesquelles une évaluation peut avoir un niveau de confiance faible :

- Vous n’avez pas profilé votre environnement pendant la durée pour laquelle vous créez l’évaluation. Par exemple, si vous créez l’évaluation avec une durée des performances définie sur 1 jour, vous devez attendre au moins un jour après le démarrage de la découverte pour que tous les points de données soient collectés.
- Certaines machines virtuelles ont été arrêtées pendant la période de calcul de l’évaluation. Si des machines virtuelles sont désactivées pendant une certaine durée, Server Assessment ne peut pas collecter les données de performances pour cette période.
- Certaines machines virtuelles ont été créées au cours de la période pour laquelle l’évaluation a été calculée. Par exemple, si vous avez créé une évaluation de l’historique des performances du mois précédent, mais que certaines machines virtuelles ont été créées dans l’environnement qu’il y a une semaine, l’historique des performances des nouvelles machines virtuelles n’existera pas pour la durée totale.

> [!NOTE]
> Si le niveau de confiance d’une évaluation est inférieur à cinq étoiles, nous vous recommandons d’attendre au moins un jour pour que l’appliance profile l’environnement, puis de recalculer l’évaluation. À défaut, il se peut que le dimensionnement basé sur les performances ne soit pas fiable. Dans ce cas, nous vous recommandons de faire passer l’évaluation au dimensionnement local.

## <a name="monthly-cost-estimation"></a>Estimation des coûts mensuels

Une fois les suggestions de dimensionnement terminées, Azure Migrate calcule le coût total d’exécution des charges de travail locales dans AVS en multipliant le nombre de nœuds AVS requis par le prix du nœud. Le coût par machine virtuelle est calculé en divisant le coût total par le nombre de machines virtuelles dans l’évaluation. 
- Le calcul prend en compte le nombre de nœuds requis, le type de nœud et l’emplacement.
- Il agrège le coût de tous les nœuds pour calculer le coût mensuel total.
- Les coûts sont affichés dans la devise spécifiée dans les paramètres d’évaluation.

Étant donné que la tarification pour Azure VMware Solution (AVS) se fait par nœud, le coût total ne comprend pas de coût de calcul ni de distribution des coûts de stockage. [En savoir plus](../azure-vmware/introduction.md)

Notez que comme Azure VMware Solution (AVS) est en préversion, les prix des nœuds dans l’évaluation sont les prix de la version préliminaire. Pour obtenir de l’aide, contactez votre équipe MSFT AVS GBB locale .

## <a name="migration-tool-guidance"></a>Guide de l’outil de migration

Dans le rapport de préparation à Azure pour l’évaluation Azure VMware Solution (AVS), vous pouvez voir les outils suggérés suivants : 
- **VMware HCX ou Enterprise** : pour les machines VMware, la solution VMware Hybrid Cloud Extension (HCX) est l’outil de migration recommandé pour migrer votre charge de travail locale vers votre cloud privé Azure VMware Solution (AVS). [En savoir plus](../azure-vmware/tutorial-deploy-vmware-hcx.md)
- **Inconnue** : Pour les machines importées via un fichier CSV, l’outil de migration par défaut est inconnu. Pour les machines VMware, il est cependant recommandé d’utiliser la solution VMWare Hybrid Cloud extension (HCX).

## <a name="next-steps"></a>Étapes suivantes

Créer une évaluation pour [des machines virtuelles AVS VMware](how-to-create-azure-vmware-solution-assessment.md).
