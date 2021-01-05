---
title: Bonnes pratiques d’évaluation dans Azure Migrate Server Assessment
description: Conseils pour créer des évaluations avec Azure Migrate Server Assessment.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 11/19/2019
ms.openlocfilehash: c1fff5b5b7f6450ad8d1977e55a1f6b255f3d668
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754315"
---
# <a name="best-practices-for-creating-assessments"></a>Meilleures pratiques pour la création d’évaluations

[Azure Migrate](./migrate-services-overview.md) fournit un hub d’outils qui vous permettent de découvrir, d’évaluer et de migrer des applications, une infrastructure et des charges de travail vers Microsoft Azure. Le hub comprend des outils Azure Migrate et des offres d’ISV (fournisseurs de logiciels indépendants) tiers.

Cet article résume les meilleures pratiques lors de la création d’évaluations à l’aide de l’outil Azure Migrate Server Assessment.

## <a name="about-assessments"></a>À propos des évaluations

Les évaluations que vous créez avec Azure Migrate Server Assessment sont une capture instantanée de données à un moment donné. Vous pouvez créer deux types d’évaluation à l’aide d’Azure Migrate : Évaluation de serveur :

**Type d’évaluation** | **Détails**
--- | --- 
**Microsoft Azure** | Évaluations pour migrer vos serveurs locaux vers des machines virtuelles Azure. <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md), [machines virtuelles Hyper-V](how-to-set-up-appliance-hyper-v.md) et [serveurs physiques](how-to-set-up-appliance-physical.md) locaux pour la migration vers Azure à l’aide de ce type d’évaluation. [En savoir plus](concepts-assessment-calculation.md)
**Azure VMware Solution (AVS)** | Évaluations pour migrer vos serveurs locaux vers [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md) locales pour la migration vers Azure VMware Solution (AVS) avec ce type d’évaluation. [En savoir plus](concepts-azure-vmware-solution-assessment-calculation.md)


### <a name="sizing-criteria"></a>Critères de dimensionnement
L’évaluation de serveur fournit deux options de critères de dimensionnement :

**Critère de dimensionnement** | **Détails** | **Données**
--- | --- | ---
**Basée sur les performances** | Évaluations qui donnent des recommandations en fonction des données de performances collectées | **Évaluation des machines virtuelles Azure** : Les recommandations concernant les machines virtuelles sont fonction des données d’utilisation du processeur et de la mémoire.<br/><br/> Les recommandations concernant le type de disque (HDD/SSD ou disque managé premium) sont fonction de l’IOPS et du débit des disques locaux.<br/><br/> **Évaluation d’Azure VMware Solution (AVS)**  : Les suggestions concernant les nœuds AVS sont basées sur les données d’utilisation du processeur et de la mémoire.
**Telle quelle locale** | Évaluations qui n’utilisent pas de données de performances pour formuler des recommandations. | **Évaluation des machines virtuelles Azure** : Les recommandations concernant la taille des machines virtuelles sont basées sur la taille de la machines virtuelle locale.<br/><br> Le type de disque recommandé est basé sur ce que vous sélectionnez dans le paramètre type de stockage pour l’évaluation.<br/><br/> **Évaluation d’Azure VMware Solution (AVS)**  : Les suggestions concernant les nœuds AVS sont basées sur la taille des machines virtuelles locales.

#### <a name="example"></a>Exemple
Par exemple, si vous avez une machine virtuelle locale avec quatre cœurs utilisée à 20 % et une mémoire de 8 Go utilisée à 10 %, les évaluations Azure VM seront les suivantes :

- **Évaluation basée sur les performances** :
    - Identifie les cœurs et la mémoire effectifs sur la base des cœurs (4 x 0,20 = 0,8) et de l’utilisation de la mémoire (8 Go x 0,10 = 0,8).
    - Applique le facteur de confort spécifié dans les propriétés d’évaluation (par exemple, 1.3x) pour obtenir les valeurs à utiliser pour le dimensionnement. 
    - Recommande la taille de machine virtuelle la plus proche dans Azure pouvant prendre en charge environ 1,04 cœur (0,8 x 1,3) et environ 1,04 Go (0,8 x 1,3) de mémoire.

- **Évaluation telle quelle (locale)**  :
    -  Recommande une machine virtuelle à quatre cœurs ; 8 Go de mémoire.


## <a name="best-practices-for-creating-assessments"></a>Meilleures pratiques pour la création d’évaluations

L’appliance Azure Migrate profile en permanence votre environnement local et envoie des métadonnées et des données de performances à Azure. Suivez ces meilleures pratiques pour évaluer les serveurs découverts à l’aide d’une appliance :

- **Créer des évaluations telles quelles** : Vous pouvez créer des évaluations telles quelles dès que vos machines s’affichent dans le portail Azure Migrate.
- **Créer une évaluation basée sur les performances** : Après la configuration de la détection, nous vous recommandons d’attendre au moins une journée avant d’exécuter une évaluation basée sur les performances :
    - La collecte des données de performances prend du temps. En attendant au moins une journée, vous pouvez être certain qu’il y a suffisamment de points de données de performances avant d’exécuter l’évaluation.
    - Lorsque vous exécutez des évaluations basées sur les performances, veillez à profiler votre environnement pour la durée de l’évaluation. Par exemple, si vous créez une évaluation avec une durée des performances définie sur une semaine, vous devez attendre au moins une semaine après le démarrage de la détection pour que tous les points de données soient collectés. Sinon, l’évaluation n’aura pas cinq étoiles.
- **Recalculer les évaluations** : Étant donné que les évaluations sont des instantanés à un moment donné, elles ne sont pas automatiquement mises à jour avec les données les plus récentes. Pour mettre à jour une évaluation avec les données les plus récentes, vous devez la recalculer.

Suivez ces meilleures pratiques pour évaluer les serveurs importés dans Azure Migrate via. le fichier .CSV :

- **Créer des évaluations telles quelles** : Vous pouvez créer des évaluations telles quelles dès que vos machines s’affichent dans le portail Azure Migrate.
- **Créer une évaluation basée sur les performances** : Cela permet d’obtenir une meilleure estimation des coûts, surtout si vous avez surapprovisionné la capacité du serveur localement. Toutefois, la précision de l’évaluation basée sur les performances dépend des données de performances que vous avez spécifiées pour les serveurs. 
- **Recalculer les évaluations** : Étant donné que les évaluations sont des instantanés à un moment donné, elles ne sont pas automatiquement mises à jour avec les données les plus récentes. Pour mettre à jour une évaluation avec les dernières données importées, vous devez la recalculer.
 
### <a name="ftt-sizing-parameters-for-avs-assessments"></a>Paramètres de dimensionnement FTT pour les évaluations AVS

Le moteur de stockage utilisé dans AVS est vSAN. Les stratégies de stockage vSAN définissent les exigences de stockage pour vos machines virtuelles. Ces stratégies garantissent le niveau de service exigé pour vos machines virtuelles, car elles déterminent la façon dont le stockage est alloué à la machine virtuelle. Voici les combinaisons FTT-RAID disponibles : 

**Nombre de pannes tolérées (FTT, Failures to Tolerate)** | **Configuration RAID** | **Nombre minimal d’hôtes requis** | **Considérations sur la taille**
--- | --- | --- | --- 
1 | RAID-1 (Mise en miroir) | 3 | Une machine virtuelle de 100 Go consomme 200 Go.
1 | RAID-5 (Code d’effacement) | 4 | Une machine virtuelle de 100 Go consomme 133,33 Go
2 | RAID-1 (Mise en miroir) | 5 | Une machine virtuelle de 100 Go consomme 300 Go.
2 | RAID-6 (Code d’effacement) | 6 | Une machine virtuelle de 100 Go consomme 150 Go.
3 | RAID-1 (Mise en miroir) | 7 | Une machine virtuelle de 100 Go consomme 400 Go.


## <a name="best-practices-for-confidence-ratings"></a>Meilleures pratiques pour les niveaux de confiance

Quand vous effectuez des évaluations basées sur les performances, une note de confiance des évaluations allant de 1 étoile (le plus faible) à 5 étoiles (le plus élevé) est attribuée. Pour utiliser efficacement les notes de confiance des évaluations :
- Azure Migrate Server Assessment a besoin des données d’utilisation du processeur/de la mémoire de la machine virtuelle.
- Pour chaque disque attaché à la machine virtuelle locale, les données IOPS/de débit en lecture/écriture sont nécessaires.
- Pour chaque carte réseau attachée à la machine virtuelle, des données en entrée/sortie du réseau sont nécessaires.

Selon le pourcentage de points de données disponibles pour la durée sélectionnée, la note de confiance d’une évaluation est fournie comme le résume le tableau ci-dessous.

   **Disponibilité des points de données** | **Niveau de confiance**
   --- | ---
   0 %-20 % | 1 étoile
   21 %-40 % | 2 étoiles
   41 %-60 % | 3 étoiles
   61 %-80 % | 4 étoiles
   81 %-100 % | 5 étoiles


## <a name="common-assessment-issues"></a>Problèmes d'évaluation courants

Voici comment résoudre certains problèmes d’environnement courants qui affectent les évaluations.

###  <a name="out-of-sync-assessments"></a>Évaluations non synchronisées

Si vous ajoutez ou supprimez des machines d’un groupe après avoir créé une évaluation, l’évaluation que vous avez créée sera marquée comme étant **non synchronisée**. Réexécutez l’évaluation (**Recalculer**) pour refléter les modifications du groupe.

### <a name="outdated-assessments"></a>Évaluations obsolètes

En cas de modification locale des machines virtuelles appartenant à un groupe qui a été évalué, l’évaluation est marquée comme **obsolète**. Une évaluation peut être marquée comme « Obsolète » en raison d’une ou de plusieurs modifications dans les propriétés ci-dessous :

- Nombre de cœurs du processeur
- Mémoire allouée
- Type de démarrage ou microprogramme
- Nom du système d'exploitation, version et architecture
- Nombre de disques
- Nombre de cartes réseau
- Modification de la taille du disque (Go alloués)
- Mise à jour de la carte d’interface réseau. Exemple : Modifications des adresses Mac, ajout d’adresses IP, etc.

Réexécutez l’évaluation (**Recalculer**) pour refléter les modifications.

### <a name="low-confidence-rating"></a>Faible niveau de confiance

Il peut arriver qu’une évaluation n’ait pas tous les points de données pour diverses raisons :

- Vous n’avez pas profilé votre environnement pour la durée pour laquelle vous créez l’évaluation. Par exemple, si vous créez une *évaluation basée sur les performances* avec une durée des performances définie sur une semaine, vous devez attendre au moins une semaine après le démarrage de la découverte pour que tous les points de données soient recueillis. Vous pouvez toujours cliquer sur **Recalculer** pour voir la note de confiance applicable la plus récente. Notez que la note de confiance est applicable uniquement quand vous créez une évaluation *basée sur les performances*.

- Plusieurs machines virtuelles ont été arrêtées pendant la période de calcul de l’évaluation. Si certaines machines virtuelles ont été mises hors tension pendant un certain temps, l’outil Évaluation de serveur ne peut pas recueillir les données de performances pour cette période.

- Peu de machines virtuelles ont été créées après que la découverte dans Évaluation de serveur a commencé. Par exemple, si vous créez une évaluation de l’historique des performances du mois dernier, mais si la création de quelques machines virtuelles dans l’environnement ne remonte qu’à une semaine. Dans ce cas, les données de performances pour les nouvelles machines virtuelles ne seront pas disponibles pour toute la durée et la note de confiance sera faible.

### <a name="migration-tool-guidance-for-avs-assessments"></a>Guide de l’outil de migration pour les évaluations AVS

Dans le rapport de préparation à Azure pour l’évaluation Azure VMware Solution (AVS), vous pouvez voir les outils suggérés suivants : 
- **VMware HCX ou Enterprise** : en ce qui concerne les machines VMware, la solution VMware Hybrid Cloud Extension (HCX) est l’outil de migration recommandé pour migrer votre charge de travail locale vers votre cloud privé Azure VMware Solution (AVS). [En savoir plus](../azure-vmware/tutorial-deploy-vmware-hcx.md)
- **Inconnue** : Pour les machines importées via un fichier CSV, l’outil de migration par défaut est inconnu. Pour les machines VMware, il est toutefois recommandé d’utiliser la solution VMware Hybrid Cloud Extension (HCX).


## <a name="next-steps"></a>Étapes suivantes

- [Découvrez](concepts-assessment-calculation.md) le mode de calcul des évaluations.
- [Découvrez](how-to-modify-assessment.md) comment personnaliser une évaluation.
