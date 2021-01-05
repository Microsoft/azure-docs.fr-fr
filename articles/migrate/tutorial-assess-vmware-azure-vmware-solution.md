---
title: Évaluer les machines virtuelles VMware en vue de leur migration vers Azure VMware Solution (AVS) avec Azure Migrate
description: Découvrez comment évaluer les machines virtuelles VMware en vue de leur migration vers AVS avec Azure Migrate Server Assessment.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: tutorial
ms.date: 09/14/2020
ms.custom: MVC
ms.openlocfilehash: f6d3c6f77b062939a88e7277cb7f0ab6ecff9fcb
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753074"
---
# <a name="tutorial-assess-vmware-vms-for-migration-to-avs"></a>Tutoriel : Évaluer les machines virtuelles VMware pour les migrer vers AVS

Dans le cadre de votre migration vers Azure, vous devez évaluer vos charges de travail locales afin de déterminer la préparation nécessaire, d’identifier les risques et d’estimer les coûts ainsi que le niveau de complexité de la tâche.

Cet article explique comment évaluer les machines virtuelles VMware découvertes en vue de leur migration vers Azure VMware Solution (AVS) à l’aide d’Azure Migrate : Server Assessment. AVS est un service géré qui vous permet d’exécuter la plateforme VMware dans Azure.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
- Exécuter une évaluation basée sur les métadonnées et les informations de configuration de l’ordinateur
- Exécuter une évaluation basée sur les données de performances

> [!NOTE]
> Les tutoriels indiquent le moyen le plus rapide de tester un scénario, et ils utilisent les options par défaut lorsque cela est possible. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.



## <a name="prerequisites"></a>Prérequis

Avant de suivre ce tutoriel pour évaluer vos machines en vue de leur migration vers AVS, vérifiez que vous avez découvert les machines que vous souhaitez évaluer :

- Pour découvrir des ordinateurs à l’aide de l’appliance Azure Migrate, [suivez ce tutoriel](tutorial-discover-vmware.md). 
- Pour découvrir des ordinateurs à l’aide d’un fichier CSV importé, [suivez ce tutoriel](tutorial-discover-import.md).


## <a name="decide-which-assessment-to-run"></a>Décider de l’évaluation à exécuter


Déterminez si vous souhaitez exécuter une évaluation à l’aide de critères de dimensionnement basés sur les données de configuration de l’ordinateur, sur les métadonnées collectées telles quelles localement ou sur les données de performances dynamiques.

**Évaluation** | **Détails** | **Recommandation**
--- | --- | ---
**Telle quelle locale** | Pour une évaluation basée sur les données de configuration ou les métadonnées de l’ordinateur.  | La taille de nœud recommandée dans AVS est basée sur la taille de la machine virtuelle locale, ainsi que sur les paramètres que vous spécifiez dans l’évaluation pour le type de nœud, le type de stockage et le paramètre Échecs à tolérer.
**Basée sur les performances** | Pour une évaluation basée sur les données de performances dynamiques collectées. | La taille de nœud recommandée dans AVS est basée sur les données d’utilisation du processeur et de la mémoire, ainsi que sur les paramètres que vous spécifiez dans l’évaluation pour le type de nœud, le type de stockage et le paramètre Échecs à tolérer.

## <a name="run-an-assessment"></a>Exécuter une évaluation

Exécutez une évaluation comme suit :

1. Dans la page **Serveurs** > **Serveurs Windows et Linux**, cliquez sur **Évaluer et migrer des serveurs**.

   ![Emplacement du bouton Évaluer et migrer des serveurs](./media/tutorial-assess-vmware-azure-vmware-solution/assess.png)

2. Dans **Azure Migrate : Server Assessment**, cliquez sur **Évaluer**.

3. Dans **Évaluer les serveurs** > **Type d’évaluation**, sélectionnez **Azure VMware Solution (AVS) (préversion)** .
4. Dans **Source de découverte** :

    - Si vous avez découvert des ordinateurs à l’aide de l’appliance, sélectionnez **Machines découvertes par l’appliance Azure Migrate**.
    - Si vous avez découvert des ordinateurs à l’aide d’un fichier CSV importé, sélectionnez **Machines importées**. 
    
5. Spécifiez un nom pour l’évaluation. 
6. Cliquez sur **Tout afficher** pour passer en revue les propriétés de l’évaluation.

    ![Page de sélection des paramètres d’évaluation](./media/tutorial-assess-vmware-azure-vmware-solution/assess-servers.png)


7. Dans **Propriétés d’évaluation** > **Propriétés de la cible** :

    - Dans **Emplacement cible**, sélectionnez la région Azure vers laquelle vous souhaitez effectuer la migration.
       - Les recommandations concernant la taille et le coût sont basées sur l’emplacement que vous spécifiez.
       - Vous pouvez effectuer une évaluation pour trois régions (USA Est, USA Ouest, Europe Ouest).
   - Dans **Type de stockage**, gardez **vSAN**. Il s’agit du type de stockage par défaut pour un cloud privé AVS.
   - Les **instances réservées** ne sont pas prises en charge pour les nœuds AVS.
8. Dans **Taille de la machine virtuelle** :
    - Dans **Type de nœud**, sélectionnez un type de nœud en fonction des charges de travail qui sont exécutées sur les machines virtuelles locales.
        - Azure Migrate recommande le type des nœuds nécessaires à la migration des machines virtuelles vers AVS.
        - Le type de nœud par défaut est AV36.
    - **Paramètre Échecs à tolérer - Niveau RAID** : sélectionnez une combinaison de valeurs Échecs à tolérer - RAID.  L’option Échecs à tolérer sélectionnée associée à la configuration de disque de machine virtuelle locale détermine le stockage vSAN total nécessaire dans AVS.
    - Dans **Surabonnement pour les processeurs** , spécifiez combien de cœurs virtuels doivent être associés à un même cœur physique dans le nœud AVS. Un surabonnement supérieur à 4:1 peut entraîner une dégradation des performances. Toutefois, il peut être utilisé pour les charges de travail de type serveur web.

9. Dans **Taille du nœud** : 
    - Dans **Critère de dimensionnement**, indiquez si vous souhaitez baser l’évaluation sur les métadonnées statiques ou sur les données de performances. Si vous utilisez des données de performances :
        - Dans **Historique des performances**, indiquez la période de données sur laquelle vous souhaitez baser l’évaluation.
        - Dans **Utilisation en centile**, spécifiez la valeur de centile que vous souhaitez utiliser pour l’échantillon de performances. 
    - Dans **Facteur de confort**, indiquez la mémoire tampon que vous souhaitez utiliser lors de l’évaluation. Celle-ci prend en compte les problèmes, tels que l’utilisation saisonnière, l’historique des performances de courte durée et l’augmentation probable de l’utilisation. Par exemple, si vous utilisez un facteur de confort de 2 :
    
        **Composant** | **Utilisation efficace** | **Ajouter un facteur de confort (2.0)**
        --- | --- | ---  
        Cœurs | 2 | 4
        Mémoire | 8 Go | 16 Go     

10. Dans **Tarifs** :
    - Dans **Offre**, l’[offre Azure](https://azure.microsoft.com/support/legal/offer-details/) à laquelle vous êtes abonné affiche les estimations de coûts Server Assessment.
    - Dans **Devise**, sélectionnez la devise de facturation de votre compte.
    - Dans **Remise (%)** , ajoutez les remises applicables à votre abonnement que vous recevez en plus de l’offre Azure. Le paramètre par défaut est 0 %.

11. Si vous avez apporté des modifications, cliquez sur **Enregistrer**.

    ![Propriétés de l’évaluation](./media/tutorial-assess-vmware-azure-vmware-solution/view-all.png)

12. Dans **Évaluer les serveurs**, cliquez sur **Suivant**.
13. Dans **Évaluer les serveurs** > **Sélectionner les machines à évaluer**, pour créer un groupe de serveurs à des fins d’évaluation, sélectionnez **Créer**, puis spécifiez un nom de groupe. 
14. Sélectionnez l’appliance, puis sélectionnez les machines virtuelles que vous souhaitez ajouter au groupe. Cliquez ensuite sur **Suivant**.
15. Dans **Vérifier + créer l’évaluation**, passez en revue les détails de l’évaluation, puis cliquez sur **Créer une évaluation** pour créer le groupe et lancer l’évaluation.

    > [!NOTE]
    > Pour les évaluations basées sur les performances, nous vous recommandons d’attendre au moins un jour après le démarrage de la découverte pour créer l’évaluation. Cela permet de collecter des données de performances avec un plus haut niveau de confiance. Dans l’idéal, une fois que vous avez démarré la découverte, vous devriez attendre la période de performances que vous avez spécifiée (jour/semaine/mois) afin d’obtenir un haut niveau de confiance.

## <a name="review-an-assessment"></a>Réviser une évaluation

Une évaluation AVS décrit les éléments suivants :

- Préparation AVS : Si les machines virtuelles locales sont adaptées à la migration vers Azure VMware Solution (AVS).
- Nombre de nœuds AVS: Estimation du nombre de nœuds AVS requis pour exécuter les machines virtuelles.
- Utilisation des nœuds AVS: Utilisation estimée du processeur, de la mémoire et du stockage sur tous les nœuds.
- Estimation des coûts mensuels : Les coûts mensuels estimés pour tous les nœuds Azure VMware Solution (AVS) exécutant les machines virtuelles locales.

## <a name="view-an-assessment"></a>Voir une évaluation

Pour voir une évaluation :

1. Dans **Serveurs** > **Azure Migrate : évaluation de serveur**, cliquez sur le nombre à côté de **Évaluations**.
2. Dans **Évaluations**, sélectionnez une évaluation pour l’ouvrir. 
3. Passez en revue le récapitulatif de l’évaluation. Vous pouvez également modifier les propriétés de l’évaluation ou recalculer celle-ci.
 

### <a name="review-readiness"></a>Vérifier la préparation nécessaire

1. Cliquez sur **Préparé pour Azure**.
2. Dans **Préparé pour Azure**, vérifiez l’état de la machine virtuelle.

    - **Disponible pour Azure VMware Solution** : La machine peut être déplacée vers Azure AVS telle quelle, sans aucune modification. La machine démarrera dans AVS avec une prise en charge complète d’AVS.
    - **Disponible sous conditions** : La machine peut présenter des problèmes de compatibilité avec la version actuelle de vSphere. Il peut s’avérer nécessaire d’installer les outils VMware ou de configurer d’autres paramètres, afin de disposer de toutes les fonctionnalités AVS.
    - **Non disponible pour AVS** : La machine virtuelle ne démarre pas dans AVS. Par exemple, si une machine virtuelle VMware locale est associée à un appareil externe (comme un CD-ROM) et que vous utilisez VMware VMotion, l’opération VMotion échouera.
 - **État de la préparation inconnu** : Azure Migrate n’a pas pu déterminer l’état de préparation de la machine, car les métadonnées collectées dans l’environnement local sont insuffisantes.

3. Examinez l’outil suggéré.

    - VMware HCX ou Enterprise : Pour les machines VMware, la solution VMWare Hybrid Cloud Extension (HCX) est l’outil de migration suggéré pour déplacer votre charge de travail locale vers votre cloud privé Azure VMware Solution (AVS). En savoir plus.
    - Inconnue : Pour les machines importées via un fichier CSV, l’outil de migration par défaut est inconnu. Pour les machines VMware, il est cependant suggéré d’utiliser la solution VMware Hybrid Cloud Extension (HCX).
4. Cliquez sur un état Préparé pour Azure VMware Solution. Vous pouvez voir les informations relatives à l’état de préparation des machines virtuelles et explorer ces dernières en détail, notamment en ce qui concerne les valeurs pour le calcul, le stockage et le réseau.

### <a name="review-cost-estimates"></a>Réviser les estimations de coûts

Ce résumé d’évaluation montre une estimation des coûts de calcul et de stockage liés à l’exécution des machines virtuelles dans Azure. 

1. Vérifiez les coûts totaux mensuels. Les coûts sont agrégés pour toutes les machines virtuelles du groupe évalué.

    - Les estimations de coût se basent sur le nombre total de nœuds AVS nécessaires en tenant compte des besoins en ressources de toutes les machines virtuelles.
    - Étant donné que les tarifs AVS sont donnés pour un nœud, le coût total n’indique pas la répartition entre les coûts de calcul et les coûts de stockage.
    - L’estimation des coûts porte sur l’exécution des machines virtuelles locales dans AVS. Azure Migrate Server Assessment ne prend pas en compte les coûts PaaS ou SaaS.

2. Passez en revue les estimations des coûts de stockage mensuels. Cette vue montre les coûts de stockage agrégés pour le groupe évalué, répartis selon les différents types de disque de stockage. 
3. Vous pouvez consulter le détail des coûts de machines virtuelles spécifiques.

### <a name="review-confidence-rating"></a>Examiner le niveau de confiance

Server Assessment attribue un niveau de confiance aux évaluations basées sur les performances. Ce niveau est exprimé en étoiles, 1 étoile étant le plus bas niveau et 5 étoiles le plus haut.

![Niveau de confiance](./media/tutorial-assess-vmware-azure-vmware-solution/confidence-rating.png)

Le niveau de confiance vous permet d’estimer la fiabilité des suggestions qui sont fournies par l’évaluation concernant la taille. Le niveau de confiance est basé sur la disponibilité des points de données nécessaires pour calculer l’évaluation.

> [!NOTE]
> Aucun niveau de confiance n’est attribué si vous créez une évaluation basée sur un fichier CSV.

Les niveaux de confiance sont les suivants.

**Disponibilité des points de données** | **Niveau de confiance**
--- | ---
0 %-20 % | 1 étoile
21 %-40 % | 2 étoiles
41 %-60 % | 3 étoiles
61 %-80 % | 4 étoiles
81 %-100 % | 5 étoiles

[En savoir plus](concepts-assessment-calculation.md#confidence-ratings-performance-based) sur les niveaux de confiance.

## <a name="next-steps"></a>Étapes suivantes

- Recherchez des dépendances de machine à l’aide du [mappage des dépendances](concepts-dependency-visualization.md).
- Configurez un mappage des dépendances [sans agent](how-to-create-group-machine-dependencies-agentless.md) ou [basé sur un agent](how-to-create-group-machine-dependencies.md).
