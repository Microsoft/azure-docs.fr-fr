---
title: Créer une évaluation AVS avec Azure Migrate Server Assessment | Microsoft Docs
description: Décrit comment créer une évaluation Azure VMware Solution avec l’outil Azure Migrate Server Assessment
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: how-to
ms.date: 06/26/2020
ms.openlocfilehash: 93d17ec2a4fb5c191ce02c73a7a3532e9c854b00
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752071"
---
# <a name="create-an-azure-vmware-solution-avs-assessment"></a>Créer une évaluation Azure VMware Solution (AVS)

Cet article explique comment créer une évaluation Azure VMware Solution (AVS) pour les machines virtuelles VMware locales avec Azure Migrate : Server Assessment.

[Azure Migrate](migrate-services-overview.md) vous aide à migrer vers Azure. Azure Migrate offre un hub centralisé pour suivre la découverte, l’évaluation et la migration d’une infrastructure, d’applications et de données locales vers Azure. Le hub fournit des outils Azure pour l’évaluation et la migration, ainsi que des offres d’éditeurs de logiciels indépendant (ISV) tiers.

## <a name="before-you-start"></a>Avant de commencer

- Assurez-vous que vous avez [créé](./create-manage-projects.md) un projet Azure Migrate.
- Si vous avez déjà créé un projet, assurez-vous que vous avez [ajouté](how-to-assess.md) Azure Migrate : Server Assessment.
- Pour créer une évaluation, vous devez configurer une appliance Azure Migrate pour [VMware](how-to-set-up-appliance-vmware.md), qui découvre les machines locales et envoie des métadonnées et des données de performances à Azure Migrate : Server Assessment. [Plus d’informations](migrate-appliance.md)
- Vous pouvez également [importer des métadonnées de serveur](./tutorial-discover-import.md) au format CSV (valeurs séparées par des virgules).


## <a name="azure-vmware-solution-avs-assessment-overview"></a>Vue d’ensemble d’évaluation Azure VMware Solution (AVS)

Vous pouvez créer deux types d’évaluation à l’aide d’Azure Migrate : Server Assessment.

**Type d’évaluation** | **Détails**
--- | --- 
**Microsoft Azure** | Évaluations pour migrer vos serveurs locaux vers des machines virtuelles Azure. <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md), [machines virtuelles Hyper-V](how-to-set-up-appliance-hyper-v.md) et [serveurs physiques](how-to-set-up-appliance-physical.md) locaux pour la migration vers Azure à l’aide de ce type d’évaluation. [En savoir plus](concepts-assessment-calculation.md)
**Azure VMware Solution (AVS)** | Évaluations pour migrer vos serveurs locaux vers [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Vous pouvez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md) locales pour la migration vers Azure VMware Solution (AVS) avec ce type d’évaluation. [En savoir plus](concepts-azure-vmware-solution-assessment-calculation.md)

> [!NOTE]
> L’évaluation Azure VMware Solution (AVS) est actuellement en version préliminaire et ne peut être créée que pour des machines virtuelles VMware.


Il existe deux types de critères de dimensionnement que vous pouvez utiliser pour créer des évaluations Azure VMware Solution (AVS) :

**Évaluation** | **Détails** | **Données**
--- | --- | ---
**Basée sur les performances** | Évaluations basées sur les données de performances collectées sur des machines virtuelles locales. | **Taille de nœud recommandée**: En fonction des données d’utilisation du processeur et de la mémoire, ainsi que du type de nœud, du type de stockage et du paramètre FTT que vous sélectionnez pour l’évaluation.
**Localement** | Évaluations basées sur le dimensionnement local. | **Taille de nœud recommandée**: En fonction de la taille des machines virtuelles locales, ainsi que du type de nœud, du type de stockage, et du paramètre FTT que vous sélectionnez pour l’évaluation.


## <a name="run-an-azure-vmware-solution-avs-assessment"></a>Exécuter une évaluation Azure VMware Solution (AVS)

Procédez comme suit pour exécuter une évaluation Azure VMware Solution (AVS) :

1. Passez en revue les [meilleures pratiques](best-practices-assessment.md) liées à la création d’évaluations.

2. Sous l’onglet **Serveurs**, dans la vignette **Azure Migrate : Mosaïque**  d’évaluation de serveur, cliquez sur **Évaluer**.

    ![Capture d’écran montrant des serveurs Azure Migrate avec l’option Évaluer sélectionnée sous Outils d’évaluation.](./media/how-to-create-assessment/assess.png)

3. Dans **Évaluer les serveurs**, sélectionnez le type d’évaluation « Azure VMware Solution (AVS) », sélectionnez la source de détection et spécifiez le nom de l’évaluation.

    ![Notions de base d’évaluation](./media/how-to-create-avs-assessment/assess-servers-avs.png)

4. Cliquez sur **Tout afficher** pour passer en revue les propriétés de l’évaluation.

    ![Propriétés de l’évaluation AVS](./media/how-to-create-avs-assessment/avs-view-all.png)

5. Cliquez sur **Suivant** pour **sélectionner les machines à évaluer**. Dans **Sélectionner ou créer un groupe**, sélectionnez **Créer**, puis spécifiez un nom de groupe. Un groupe rassemble une ou plusieurs machines virtuelles à évaluer.

6. Dans **Ajouter des machines au groupe**, sélectionnez les machines virtuelles à ajouter au groupe.

7. Cliquez sur **Suivant** pour **vérifier + créer une évaluation** afin de passer en revue les détails de l’évaluation.

8. Cliquez sur **Créer une évaluation** pour créer le groupe, puis exécutez l’évaluation.

    ![Créer une évaluation AVS](./media/how-to-create-avs-assessment/avs-assessment-create.png)

9. Une fois l’évaluation créée, vous pouvez la voir dans **Serveurs** > **Azure Migrate : Server Assessment** > **Évaluations**.

10. Cliquez sur **Exporter l’évaluation** pour la télécharger sous la forme d’un fichier Excel.


## <a name="review-an-azure-vmware-solution-avs-assessment"></a>Vérifier une évaluation Azure VMware Solution (AVS)

Une évaluation Azure VMware Solution (AVS) indique :

- **Préparation pour Azure VMware Solution (AVS)** : Si les machines virtuelles locales sont adaptées à la migration vers Azure VMware Solution (AVS).
- **Nombre de nœuds AVS**: Estimation du nombre de nœuds AVS requis pour exécuter les machines virtuelles.
- **Utilisation des nœuds AVS**: Utilisation estimée du processeur, de la mémoire et du stockage sur tous les nœuds.
- **Estimation des coûts mensuels** : Les coûts mensuels estimés pour tous les nœuds Azure VMware Solution (AVS) exécutant les machines virtuelles locales.


### <a name="view-an-assessment"></a>Voir une évaluation

1. Dans **Objectifs de migration** >  **Serveurs**, cliquez sur **Évaluations** dans **Azure Migrate : Évaluation de serveur**.

2. Dans **Évaluations**, cliquez sur une évaluation pour l’ouvrir.

    ![Récapitulatif de l’évaluation AVS](./media/how-to-create-avs-assessment/avs-assessment-summary.png)

### <a name="review-azure-vmware-solution-avs-readiness"></a>Vérifier la préparation pour Azure VMware Solution (AVS)

1. Dans **Préparé pour Azure**, vérifiez si les machines virtuelles sont prêtes pour la migration vers AVS.

2. Passez en revue l’état des machines virtuelles :
    - **Disponible pour Azure VMware Solution** : la machine peut être migrée telle quelle vers Azure (Azure VMware Solution) sans aucune modification. Elle démarrera dans AVS avec une prise en charge complète d’AVS.
    - **Disponible sous conditions** : La machine virtuelle peut présenter des problèmes de compatibilité avec la version actuelle de vSphere, ainsi qu’exiger des outils VMware et ou d’autres paramètres avant que toutes les fonctionnalités de la machine virtuelle puissent être obtenues dans AVS.
    - **Non disponible pour AVS** : La machine virtuelle ne démarrera pas dans AVS. Par exemple, si la machine virtuelle VMware locale est associée à un appareil externe tel qu’un CD-ROM, l’opération VMotion échouera (si vous utilisez VMware VMotion).
    - **État de la préparation inconnu** : Azure Migrate n’a pas pu déterminer l’état de préparation de la machine, car les métadonnées collectées dans l’environnement local sont insuffisantes.

3. Vérifiez l’outil suggéré :
    - **VMware HCX ou Enterprise** : Pour les machines VMware, la solution VMWare Hybrid Cloud Extension (HCX) est l’outil de migration suggéré pour déplacer votre charge de travail locale vers votre cloud privé Azure VMware Solution (AVS). [En savoir plus](../azure-vmware/tutorial-deploy-vmware-hcx.md)
    - **Inconnue** : Pour les machines importées via un fichier CSV, l’outil de migration par défaut est inconnu. Pour les machines VMware, il est cependant suggéré d’utiliser la solution VMware Hybrid Cloud Extension (HCX). 

4. Cliquez sur un état **Préparé pour Azure VMware Solution**. Vous pouvez voir les informations relatives à l’état de préparation des machines virtuelles et explorer ces dernières en détail, notamment en ce qui concerne les valeurs pour le calcul, le stockage et le réseau.



### <a name="review-cost-details"></a>Passer en revue les détails des coûts

Cette vue montre une estimation des coûts liés à l’exécution des machines virtuelles dans Azure VMware Solution (AVS).

1. Vérifiez les coûts totaux mensuels. Les coûts sont agrégés pour toutes les machines virtuelles du groupe évalué. 

    - Les estimations de coût se basent sur le nombre total de nœuds AVS requis en tenant compte des besoins en ressources de toutes les machines virtuelles.
    - Étant donné que la tarification pour Azure VMware Solution (AVS) se fait par nœud, le coût total ne comprend pas de coût de calcul ni de distribution des coûts de stockage.
    - L’estimation des coûts porte sur l’exécution des machines virtuelles locales dans AVS. Azure Migrate Server Assessment ne prend pas en compte les coûts PaaS ou SaaS.
    
2. Vous pouvez passer en revue les estimations des coûts de stockage mensuels. Cette vue montre les coûts de stockage agrégés pour le groupe évalué, répartis selon différents types de disque de stockage.

3. Vous pouvez explorer les détails de machines virtuelles spécifiques.


### <a name="review-confidence-rating"></a>Examiner le niveau de confiance

Quand vous effectuez des évaluations basées sur les performances, un niveau de confiance est affecté à l’évaluation.

![Niveau de confiance](./media/how-to-create-assessment/confidence-rating.png)

- Un niveau allant de 1 étoile (le plus faible) à 5 étoiles (le plus élevé) est attribué.
- Le niveau de confiance vous aide à estimer la fiabilité des suggestions de taille fournies par l’évaluation.
- Le niveau de confiance est basé sur la disponibilité des points de données nécessaires pour calculer l’évaluation.
- Pour le dimensionnement basé sur les performances, les évaluations AVS dans Évaluation du Serveur nécessitent les données d’utilisation du processeur et de la mémoire de la machine virtuelle. Les données de performances suivantes sont collectées, mais ne sont pas utilisées dans les suggestions de dimensionnement pour les évaluations AVS :
  - Les données IOPS et de débit du disque pour chaque disque attaché à la machine virtuelle.
  - Les E/S réseau pour gérer le dimensionnement basé sur les performances pour chaque carte réseau attachée à une machine virtuelle.

Les niveaux de confiance d’une évaluation sont les suivants.

**Disponibilité des points de données** | **Niveau de confiance**
--- | ---
0 %-20 % | 1 étoile
21 %-40 % | 2 étoiles
41 %-60 % | 3 étoiles
61 %-80 % | 4 étoiles
81 %-100 % | 5 étoiles

[En savoir plus](concepts-azure-vmware-solution-assessment-calculation.md) sur les données performances 


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment utiliser le [mappage de dépendances](how-to-create-group-machine-dependencies.md) pour créer des groupes à haute fiabilité.
- [En savoir plus](concepts-azure-vmware-solution-assessment-calculation.md) sur le mode de calcul des évaluations AVS.