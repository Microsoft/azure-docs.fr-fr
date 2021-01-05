---
title: Analyse des coûts et budget
description: Découvrez comment obtenir une analyse des coûts et définir un budget pour les ressources de calcul et licences logicielles sous-jacentes utilisées dans le cadre de l’exécution de vos charges de travail Batch.
ms.topic: how-to
ms.date: 07/19/2019
ms.openlocfilehash: 50ca1ecfd0a973ff39dabfcb62175ce820d0a0d6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88654240"
---
# <a name="cost-analysis-and-budgets-for-azure-batch"></a>Analyse des coûts et budgets pour Azure Batch

Il n'y a aucuns frais pour utiliser Azure Batch, mais seulement pour les ressources de calcul sous-jacentes et les licences logicielles utilisées pour exécuter les charges de travail Batch. À un niveau élevé, les coûts sont induits par les machines virtuelles dans un pool, par le transfert de données depuis la machine virtuelle, ou par toute donnée d'entrée ou de sortie stockée dans le cloud. Examinons quelques composants clés de Batch pour comprendre d'où viennent ces coûts, comment établir un budget pour un pool ou un compte, et apprendre quelques techniques pour rendre vos charges de travail Batch plus rentables.

## <a name="batch-resources"></a>Ressources Batch

Les machines virtuelles représentent la ressource la plus importante utilisée pour le traitement Batch. Le coût d'utilisation des machines virtuelles pour Batch est calculé en fonction du type, de la quantité et de la durée d'utilisation. Les options de facturation des machines virtuelles incluent le [paiement à l'utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/) ou la [réservation](../cost-management-billing/reservations/save-compute-costs-reservations.md) (payer à l'avance). Les deux options de paiement offrent différents avantages selon votre charge de travail, et les deux modèles de paiement impacteront différemment votre facture.

Lorsque des applications sont déployées sur des nœuds Batch (machines virtuelles) à l’aide de [packages d’application](batch-application-packages.md), vous êtes facturé pour les ressources Stockage Azure que vos packages d’application consomment. Vous êtes également facturé pour le stockage de tout fichier d'entrée ou de sortie, notamment les fichiers de ressources et autres données de journal. En général, le coût des données de stockage associées à Batch est beaucoup moins élevé que celui des ressources de calcul. Chaque machine virtuelle d’un pool créé avec **VirtualMachineConfiguration** est associée à un disque OS qui utilise des disques managés par Azure. Les disques managés par Azure entraînent un coût supplémentaire, et d’autres niveaux de performance de disque ont également des coûts différents.

Les pools Batch utilisent des ressources de mise en réseau. En particulier, pour les pools **VirtualMachineConfiguration**, des équilibreurs de charge standard sont utilisés et nécessitent des adresses IP statiques. Les équilibreurs de charge utilisés par Batch sont visibles pour les comptes **Abonnement utilisateur**, mais pas pour les comptes **Service Batch**. Les équilibreurs de charge standard entraînent des frais pour toutes les données transférées vers et depuis les machines virtuelles d’un pool Batch ; sélectionnez les API Batch qui récupèrent les données des nœuds de pool (par exemple Get Task/Node File), les paquets d'application de tâches, les fichiers de ressources/sorties, et les images de conteneur.

### <a name="additional-services"></a>Services supplémentaires

Les services n'incluant pas les machines virtuelles et le stockage peuvent être intégrés au coût de votre compte Batch.

D'autres services couramment utilisés avec Batch peuvent inclure :

- Application Insights
- Data Factory
- Azure Monitor
- Réseau virtuel
- Machines virtuelles avec applications graphiques

Selon les services que vous utilisez avec votre solution Batch, des frais supplémentaires peuvent s’appliquer. Référez-vous à la [Calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour déterminer le coût de chaque service supplémentaire.

## <a name="cost-analysis-and-budget-for-a-pool"></a>Analyse des coûts et budgets d’un pool

Grâce au portail Azure, vous pouvez créer des budgets et des alertes de dépenses pour vos pools ou comptes Batch. Les budgets et les alertes sont utiles pour informer les parties prenantes de tout risque de dépassement de budget. Un retard dans la réception des alertes de dépenses et un léger dépassement de budget sont possibles. Dans cet exemple, nous allons examiner une analyse des coûts d'un pool Batch individuel.

1. Dans le portail Azure, sélectionnez **Gestion des coûts + facturation** dans le menu de navigation de gauche.
1. Sélectionner votre abonnement dans la section **Mes abonnements**
1. Accédez à **Analyse du coût** sous la section **Gestion des coûts** de la barre de navigation de gauche, qui affichera une vue comme celle-ci :
1. Sélectionnez **Ajouter un filtre**. Dans le premier menu déroulant, sélectionnez **Ressource** ![Sélectionner le filtre de ressources](./media/batch-budget/resource-filter.png)
1. Dans le deuxième menu déroulant, sélectionnez le pool Batch. Si le pool est sélectionné, l'analyse du coût ressemblera à l'analyse suivante.
    ![Analyse du coût d'un pool](./media/batch-budget/pool-cost-analysis.png)

L'analyse du coût qui en résulte montre le coût du pool ainsi que les ressources qui génèrent ce coût. Dans cet exemple, les machines virtuelles utilisées dans le pool sont la ressource la plus coûteuse.

Afin de créer un budget pour le pool, sélectionnez **Budget : aucun**, puis **Créer un nouveau budget >** . Utilisez maintenant la fenêtre pour définir un budget spécifique à votre pool.

Pour plus d’informations sur la définition d’un budget, consultez [Créer et gérer des budgets Azure](../cost-management-billing/costs/tutorial-acm-create-budgets.md).

> [!NOTE]
> Azure Batch est basé sur la technologie d’Azure Cloud Services et des machines virtuelles Azure. Si vous sélectionnez **Configuration des services cloud**, vous êtes facturé en fonction de la structure de tarification des services cloud. Si vous sélectionnez la **Configuration de la machine virtuelle**, vous êtes facturé en fonction de la structure de tarification des machines virtuelles. L'exemple sur cette page utilise la **configuration de machine virtuelle**.

## <a name="minimize-cost"></a>Réduire le coût

L'utilisation de plusieurs machines virtuelles et services Azure pour des périodes prolongées peut se révéler coûteuse. Heureusement, il existe des services pour vous aider à réduire vos dépenses, ainsi que des stratégies pour optimiser votre charge de travail.

### <a name="low-priority-virtual-machines"></a>Machines virtuelles de faible priorité

Les machines virtuelles de faible priorité réduisent le coût des charges de travail Batch en tirant parti de la capacité de calcul excédentaire d’Azure. Lorsque vous spécifiez des machines virtuelles de faible priorité dans vos pools, Batch utilise ce surplus pour exécuter votre charge de travail. L'utilisation de machines virtuelles de faible priorité à la place de machines virtuelles dédiées permet de réaliser de substantielles économies.

Pour en savoir plus sur la configuration des machines virtuelles de faible priorité pour votre charge de travail, consultez [Utiliser des machines virtuelles de faible priorité avec Batch](batch-low-pri-vms.md).

### <a name="virtual-machine-os-disk-type"></a>Type de disque du système d'exploitation de la machine virtuelle

Il existe plusieurs [types de disque du système d'exploitation de la machine virtuelle](../virtual-machines/disks-types.md). La plupart des machines virtuelles ont des tailles qui prennent en charge le stockage Premium et Standard. Lorsqu'une machine virtuelle de taille ‘s’ est sélectionnée pour un pool, Batch configure des disques SSD Premium pour le système d’exploitation. Lorsque la taille de machine virtuelle ‘non s’ est sélectionnée, un disque dur standard (moins cher) est utilisé. Par exemple, les disques de système d’exploitation SSD Premium sont utilisés pour `Standard_D2s_v3`, et les disques de système d’exploitation de type standard sont utilisés pour `Standard_D2_v3`.

Les disques de système d’exploitation SSD Premium sont plus chers, mais plus performants, et les machines virtuelles équipées de disques Premium peuvent démarrer un peu plus rapidement que celles équipées de disques de système d’exploitation de type standard. Avec Batch, le disque du système d’exploitation est généralement moins utilisé car les applications et les fichiers de tâches sont situés sur le disque SSD temporaire des machines virtuelles. Par conséquent, dans la plupart des cas, il n'est pas nécessaire de payer le surcoût d’un disque SSD provisionné lorsqu'une taille de machine virtuelle ‘s’ est spécifiée.

### <a name="reserved-virtual-machine-instances"></a>Instances de machines virtuelles réservées

Si vous avez l'intention d'utiliser Batch sur une longue période, vous pouvez réduire le coût des machines virtuelles en utilisant des [réservations Azure](../cost-management-billing/reservations/save-compute-costs-reservations.md) pour vos charges de travail. Le coût d’une réservation est nettement inférieur à celui d’un paiement à l'utilisation. Les instances de machines virtuelles utilisées sans réservation sont facturées à l’utilisation. Si vous achetez une réservation, la remise de réservation est appliquée et vous n'êtes plus facturé selon un paiement à l'utilisation.

### <a name="automatic-scaling"></a>Mise à l’échelle automatique

La [mise à l'échelle automatique](batch-automatic-scaling.md) ajuste dynamiquement le nombre de machines virtuelles dans votre pool Batch en fonction des exigences du travail en cours. En mettant à l'échelle le pool selon la durée de vie d'un travail, la mise à l'échelle automatique garantit que les machines virtuelles sont mises à l'échelle et utilisées uniquement lorsqu'il y a un travail à exécuter. Lorsque le travail est terminé ou qu'il n'y a pas de travail, les machines virtuelles sont automatiquement réduites pour économiser les ressources de calcul. La mise à l'échelle vous permet de réduire le coût global de votre solution Batch en n'utilisant que les ressources nécessaires.

Pour plus d’informations sur la mise à l’échelle automatique, consultez la section [Mettre automatiquement à l’échelle les nœuds de calcul dans un pool Azure Batch](batch-automatic-scaling.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [outils et API Batch](batch-apis-tools.md) disponibles pour créer et surveiller des solutions Batch.  

- En savoir plus sur [les machines virtuelles de faible priorité avec Batch](batch-low-pri-vms.md).
