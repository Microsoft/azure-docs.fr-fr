---
title: Planifier la capacité pour la reprise d’activité d’Hyper-V avec Azure Site Recovery
description: Aidez-vous de cet article pour estimer la capacité à planifier quand vous configurez la reprise d’activité avec le service Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
services: site-recovery
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: a5764e44db31755110ac99a3e8e8e0984cdf9604
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87490572"
---
# <a name="plan-capacity-for-hyper-v-vm-disaster-recovery"></a>Planifier la capacité pour la reprise d’activité de machines virtuelles Hyper-V 

Le [Planificateur de déploiement Azure Site Recovery] (site-recovery-hyper-v-deployment-planner.md) pour le déploiement d’Hyper-V sur Azure fournit ce qui suit :

* Évaluation de l’éligibilité de la machine virtuelle en fonction du nombre de disques, de la taille du disque, des E/S par seconde, de l’activité et de quelques caractéristiques de machine virtuelle
* Besoin de bande passante réseau et évaluation d’objectif de point de récupération
* Exigences de l’infrastructure Azure
* Exigences de l’infrastructure locale
* Conseils de traitement par lot de réplication initiale
* Estimation du coût total de reprise d'activité après sinistre vers Azure


L’outil Azure Site Recovery Capacity Planner vous aide à déterminer vos besoins en capacité lors de la réplication de machines virtuelles Hyper-V avec Azure Site Recovery.

Utilisez Azure Site Recovery Capacity Planner pour analyser vos environnement source et charges de travail. Il vous aide à estimer les besoins en bande passante, les ressources de serveur dont vous avez besoin pour l’emplacement source, et les ressources (telles que machines virtuelles et stockage) dont vous avez besoin dans l’emplacement cible.

Vous pouvez exécuter l’outil en deux modes :

* **Planification rapide** : fournit des projections réseau et serveur sur la base du nombre moyen de machines virtuelles, des disques, du stockage, et du taux de modifications.
* **Planification détaillée** : fournit des détails sur chaque charge de travail au niveau de la machine virtuelle. Analysez la compatibilité de machine virtuelle et obtenez des projections réseau et serveur.

## <a name="before-you-start"></a>Avant de commencer

* Collecter des informations relatives à votre environnement, et notamment les machines virtuelles, le nombre de disques par machine virtuelle, le stockage par disque.
* Déterminer le taux de modification (l’évolution) quotidienne des données répliquées. Téléchargez l’[outil de planification de la capacité Hyper-V](https://www.microsoft.com/download/details.aspx?id=39057) pour obtenir le taux de modifications. [En savoir plus](./hyper-v-deployment-planner-overview.md) sur cet outil. Nous vous recommandons d’exécuter cet outil pendant une semaine pour capturer les moyennes.


## <a name="run-the-quick-planner"></a>Exécutez Quick Planner
1. Téléchargez et ouvrez [Site Azure Site Recovery Capacity Planner](https://aka.ms/asr-capacity-planner-excel). Vous devez exécuter les macros. Lorsque vous y êtes invité, opérez des sélections pour activer la modification et le contenu.

2. Dans la zone de liste **Sélectionner un type de planificateur**, sélectionnez **Quick Planner** (Planificateur rapide).

   ![Capture d’écran de l’option Select a planner type (Sélectionner un type de planificateur), avec l’option Quick Planner (Planificateur rapide) sélectionnée.](./media/site-recovery-capacity-planner/getting-started.png)

3. Dans la feuille de calcul **Capacity Planner** (Planificateur de capacité), saisissez les informations requises. Renseignez tous les champs entourés d’un cercle rouge dans la capture d’écran suivante :

   a. Dans **Sélectionner votre scénario**, choisissez **Hyper-V to Azure** (Hyper-V vers Azure) ou **VMware/Physical to Azure** (VMware/Physique vers Azure).

   b. Dans **Average daily data change rate (%)** (Taux de modification de données moyen par jour (%)), entrez les informations que vous recueillez à l’aide de l’[outil de planification de la capacité Hyper-V](./hyper-v-deployment-planner-overview.md) ou d’[Azure Site Recovery Deployment Planner](./site-recovery-deployment-planner.md).

   c. Le paramètre **Compression** n’est pas utilisé lors de la réplication de machines virtuelles Hyper-V vers Azure. Pour la compression, utilisez une appliance tierce telle que Riverbed.

   d. Dans **Retention in days** (Rétention en jours), spécifiez le nombre de jours de rétention des réplicas.

   e. Dans **Number of hours in which initial replication for the batch of virtual machines should complete** (Nombre d’heures prévu pour la réplication initiale du lot de machines virtuelles) et **Number of virtual machines per initial replication batch** (Nombre de machines virtuelles par lot de réplication initiale), entrez les paramètres de saisie utilisés pour calculer les exigences de réplication initiales. Lors du déploiement de Site Recovery, le jeu entier de données initiales est chargé.

   ![Capture d’écran de la feuille de calcul Capacity Planner (Outil de planification de la capacité), montrant les informations d’entrée requises.](./media/site-recovery-capacity-planner/inputs.png)

4. Une fois que vous avez entré les valeurs de l’environnement source, la sortie affichée inclut ce qui suit :

   * **Bande passante requise pour la réplication delta (en mégabits par seconde)**  : La bande passante réseau pour la réplication delta est calculée sur le taux de modification de données moyen par jour.
   * **Bande passante requise pour la réplication initiale (en mégabits par seconde)**  : la bande passante réseau pour la réplication initiale est calculée sur les valeurs de réplication initiale que vous entrez.
   * **Stockage requis (en Go)**  : stockage Azure total requis.
   * **Nombre total d’E/S par seconde sur le compte de stockage standard** : calculé sur la base d’une taille d’unité d’IOPS de 8K sur la totalité des comptes de stockage standard. Pour Quick Planner, le nombre est calculé en fonction de l’ensemble des disques de machine virtuelle sources et du taux de changement de données quotidien. Pour Detailed Planner, le nombre est calculé en fonction du nombre total de machines virtuelles mappées aux machines virtuelles Azure standard et du taux de modification de données sur ces dernières.
   * **Nombre de comptes de stockage standard requis** : Nombre total de comptes de stockage standard nécessaire pour protéger les machines virtuelles. Un compte de stockage standard peut prendre en charge jusqu’à 20 000 E/S par seconde sur toutes les machines virtuelles appartenant à un stockage standard. Le maximum d’E/S par seconde prises en charge par disque est de 500.
   * **Nombre de disques blob requis** : Nombre de disques qui seront créés sur le stockage Azure.
   * **Nombre de comptes Premium requis** : Nombre total de comptes de stockage premium nécessaire pour protéger les machines virtuelles. Une machine virtuelle source avec une valeur d’E/S par seconde élevée (supérieure à 20 000) a besoin d’un compte de stockage premium. Un compte de stockage premium peut prendre en charge jusqu’à 80 000 E/S par seconde.
   * **Total d’E/S par seconde sur un stockage Premium** : nombre calculé sur la base d’une taille d’unité d’IOPS de 256 K sur l’ensemble des comptes de stockage Premium. Pour Quick Planner, le nombre est calculé en fonction de l’ensemble des disques de machine virtuelle sources et du taux de changement de données quotidien. Concernant le Detailed Planner, le nombre est calculé en fonction du nombre total de machines virtuelles mappées sur des machines virtuelles Azure premium (séries DS et GS) et du taux de modification des données sur ces dernières.
   * **Nombre de serveurs de configuration requis** : indique le nombre de serveurs de configuration requis pour le déploiement.
   * **Nombre de serveurs de traitement supplémentaires requis** : Indique si des serveurs de processus supplémentaires sont nécessaires en plus du serveur de processus qui est exécuté sur le serveur de configuration par défaut.
   * **100 % de stockage supplémentaire sur la source** : Indique si un stockage supplémentaire est nécessaire dans l’emplacement source.

      ![Capture d’écran de la sortie affichée en fonction de l’entrée fournie.](./media/site-recovery-capacity-planner/output.png)

## <a name="run-the-detailed-planner"></a>Exécuter Detailed Planner

1. Téléchargez et ouvrez [Site Azure Site Recovery Capacity Planner](https://aka.ms/asr-capacity-planner-excel). Vous devez exécuter les macros. Lorsque vous y êtes invité, opérez des sélections pour activer la modification et le contenu.

2. Dans **Sélectionner un type de planificateur**, sélectionnez **Detailed Planner** dans la liste.

   ![Capture d’écran de l’option Select a planner type (Sélectionner un type de planificateur), avec l’option Detailed Planner (Planificateur détaillé) sélectionnée.](./media/site-recovery-capacity-planner/getting-started-2.png)

3. Dans la feuille de calcul **Workload Qualification**, saisissez les informations obligatoires. Vous devez renseigner tous les champs marqués.

   a. Dans **Processor Cores** (Cœurs du processeur), spécifiez le nombre total de cœurs sur un serveur source.

   b. Dans **Memory allocation (in MBs)** (Allocation de mémoire (en Moctets)), spécifiez la taille de la mémoire RAM d’un serveur source.

   c. Dans **Number of NICs** (nombre de cartes d’interface réseau), spécifiez le nombre de cartes réseau sur un serveur source.

   d. Dans **Total Storage (in GB)** (Stockage total (en Go)), spécifiez la taille totale du stockage de la machine virtuelle. Par exemple, si le serveur source comprend trois disques de 500 Go chacun, la taille de stockage totale est de 1 500 Go.

   e. Pour le **nombre de disques rattaché**, spécifiez le nombre total de disques d’un serveur source.

   f. Dans **Disk capacity utilization (%)** (Utilisation de la capacité du disque (%)), spécifiez l’utilisation moyenne.

   g. Dans **Daily data change rate (%)** (Taux de modification de données par jour (%)), spécifiez le taux de modification de données quotidien d’un serveur source.

   h. Dans **Mapping Azure VM size** (taille de machine virtuelle Azure pour le mappage), entrez la taille de machine virtuelle Azure que vous souhaitez mapper. Si vous ne souhaitez pas effectuer cette opération manuellement, sélectionnez **Compute IaaS VMs** (Machines virtuelles IaaS de calcul). Si vous entrez un paramètre manuel, puis sélectionnez **Compute IaaS VMs** (Machines virtuelles IaaS de calcul), il se peut que le paramètre manuel soit remplacé. Le processus de calcul identifie automatiquement la meilleure correspondance sur la taille de machine virtuelle Azure.

   ![Capture d’écran de la feuille de calcul Workload Qualification (Qualification de la charge de travail), montrant les informations d’entrée requises.](./media/site-recovery-capacity-planner/workload-qualification.png)

4. Si vous sélectionnez **Compute IaaS VMs** (Machines virtuelles IaaS de calcul), les opérations suivantes sont effectuées :

   * Validation des entrées obligatoires.
   * Calcule le nombre d’E/S par seconde et suggère la taille de machine virtuelle Azure la plus appropriée pour chaque machine virtuelle éligible pour une réplication vers Azure. Si aucune machine virtuelle de taille appropriée ne peut être détectée, une erreur s’affiche. Par exemple, si le nombre de disques attachés est 65, une erreur s’affiche parce que la taille de machine virtuelle Azure la plus élevée est 64.
   * Suggère un compte de stockage pouvant être utilisé pour une machine virtuelle Azure.
   * Calcule le nombre total de comptes de stockage standard et premium nécessaires pour la charge de travail. Faites défiler vers le bas pour afficher le type de stockage Azure et le compte de stockage qui peuvent être utilisés pour un serveur source.
   * Se termine et trie le reste de la table en fonction du type de stockage (standard ou premium) requis affecté à une machine virtuelle, et du nombre de disques attachés. Pour toutes les machines virtuelles qui répondent aux exigences de sauvegarde d’Azure, la colonne indiquant si **la machine virtuelle est qualifiée** affiche **Oui**. Si une machine virtuelle ne peut pas être sauvegardée sur Azure, une erreur s’affiche.

Les colonnes AA à AE qui sont générées fournissent des informations pour chaque machine virtuelle.

![Capture d’écran montrant les colonnes de sortie AA à AE.](./media/site-recovery-capacity-planner/workload-qualification-2.png)

### <a name="example"></a>Exemple
Exemple : pour les six machines virtuelles avec les valeurs indiquées dans le tableau, l’outil calcule et affecte la meilleure correspondance de machine virtuelle Azure, ainsi que les exigences de stockage Azure.

![Capture d’écran montrant les affectations de Workload Qualification (Qualification de la charge de travail).](./media/site-recovery-capacity-planner/workload-qualification-3.png)

* Dans l’exemple de résultat, notez les points suivants :

  * La première colonne est une colonne de validation pour les machines virtuelles, les disques et les taux de changement.
  * Pour cinq machines virtuelles, deux comptes de stockage standard et un compte de stockage premium sont nécessaires.
  * VM 3 ne se qualifie pas pour la protection, car un ou plusieurs disques ont un volume supérieur à 1 To.
  * VM1 et VM2 peuvent utiliser le premier compte de stockage standard
  * VM4 peut utiliser le deuxième compte de stockage standard
  * Les machines virtuelles VM5 et VM6 ont besoin d’un compte de stockage premium et peuvent utiliser un compte simple.

    > [!NOTE]
    > Le nombre d’E/S par seconde sur les comptes de stockage standard et premium est calculé au niveau de la machine virtuelle et non à celui du disque. Une machine virtuelle standard peut gérer jusqu’à 500 E/S par seconde par disque. Si le nombre d’E/S par seconde pour un disque est supérieur à 500, le stockage Premium est nécessaire. Si le nombre d’E/S par seconde pour un disque est supérieur à 500, mais que le nombre d’E/S par seconde pour l’ensemble des disques de machine virtuelle ne dépasse pas les limites établies pour les machines virtuelles Azure standard, le planificateur choisit une machine virtuelle standard et non une machine virtuelle des séries DS ou GS (les limites de machine virtuelle Azure sont la taille de machine virtuelle, le nombre de disques, le nombre de cartes, le processeur et la mémoire). Vous devez mettre à jour manuellement la taille de cellule Azure de mappage avec la machine virtuelle de série DS ou GS appropriée.


Une fois toutes les informations entrées, sélectionnez **Submit data to the planner tool** (Envoyer des données à l’outil planificateur) pour ouvrir Capacity Planner. Les charges de travail sont mises en surbrillance pour indiquer si elles sont éligibles pour la protection.

### <a name="submit-data-in-capacity-planner"></a>Envoyer des données dans Capacity Planner
1. Lorsque vous ouvrez la feuille de calcul **Capacity Planner**, celle-ci est remplie en fonction des paramètres que vous avez spécifiés. Le mot « Workload » (Charge de travail) apparaît dans la cellule **Infra inputs source** (Source des entrées Infra) pour indiquer les entrées de la feuille de calcul **Workload Qualification** (Qualification de la charge de travail).

2. Si vous souhaitez apporter des modifications, vous devez modifier la feuille de calcul **Workload Qualification** (Qualification de la charge de travail). Ensuite, sélectionnez de nouveau **Submit data to the planner tool** (Envoyer des données à l’outil planificateur).

   ![Capture d’écran montrant les entrées modifiées et les sorties résultantes dans la feuille de calcul Capacity Planner (Outil de planification de la capacité).](./media/site-recovery-capacity-planner/capacity-planner.png)

## <a name="next-steps"></a>Étapes suivantes
[Apprenez à exécuter](./hyper-v-deployment-planner-overview.md) l’outil de planification de la capacité.
