---
title: Mettre à l’échelle automatiquement des groupes de machines virtuelles identiques dans le portail Azure
description: Guide pratique pour créer des règles de mise à l’échelle automatique pour des groupes de machines virtuelles identiques dans le portail Azure
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms:service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 05/29/2018
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: ac42fe3265163a5a967524fe11063803c9ca91d3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87080588"
---
# <a name="automatically-scale-a-virtual-machine-scale-set-in-the-azure-portal"></a>Mettre à l’échelle automatiquement un groupe de machines virtuelles identiques dans le portail Azure
Lorsque vous créez un groupe identique, vous définissez le nombre d’instances de machine virtuelle que vous souhaitez exécuter. À mesure que la demande de votre application change, vous pouvez augmenter ou diminuer automatiquement le nombre d’instances de machine virtuelle. La capacité de mise à l’échelle automatique vous permet de suivre la demande du client ou de répondre aux changements de performances de votre application tout au long de son cycle de vie.

Cet article explique comment créer avec le portail Azure des règles de mise à l’échelle automatique qui analysent les performances des instances de machine virtuelle dans votre groupe identique. Ces règles de mise à l’échelle augmentent ou réduisent le nombre d’instances de machine virtuelle en réponse à ces métriques de performances. Vous pouvez aussi effectuer ces étapes avec [Azure PowerShell](tutorial-autoscale-powershell.md) ou [Azure CLI](tutorial-autoscale-cli.md).


## <a name="prerequisites"></a>Prérequis
Pour créer des règles de mise à l’échelle, vous avez besoin d’un groupe de machines virtuelles identiques. Vous pouvez créer un groupe identique avec le [portail Azure](quick-create-portal.md), [Azure PowerShell](quick-create-powershell.md) ou [Azure CLI](quick-create-cli.md).


## <a name="create-a-rule-to-automatically-scale-out"></a>Créer une règle de scale-out automatique
Si la demande de votre application augmente, la charge sur les instances de machine virtuelle dans votre groupe identique augmente. Si cette augmentation de la charge est cohérente, au lieu d’une brève demande, vous pouvez configurer des règles de mise à l’échelle automatique pour augmenter le nombre d’instances de machine virtuelle dans le groupe identique. Lorsque ces instances de machine virtuelle sont créées et que vos applications sont déployées, le groupe identique commence à distribuer le trafic vers les instances via l’équilibreur de charge. Vous contrôlez les métriques à surveiller, telles que l’usage du processeur ou du disque, la durée pendant laquelle la charge de l’application doit respecter un seuil donné, et le nombre d’instances de machine virtuelle à ajouter au groupe identique.

1. Ouvrez le portail Azure et sélectionnez **Groupes de ressources** dans le menu à gauche du tableau de bord.
2. Sélectionnez le groupe de ressources qui contient votre groupe identique, puis choisissez ce groupe dans la liste des ressources.
3. Choisissez **Mise à l’échelle** dans le menu à gauche de la fenêtre des groupes identiques. Cliquez sur le bouton **Activer la mise à l’échelle automatique** :

    ![Activer la mise à l’échelle automatique dans le portail Azure](media/virtual-machine-scale-sets-autoscale-portal/enable-autoscale.png)

4. Entrez un nom pour vos paramètres, par exemple *Mise à l’échelle automatique*, puis sélectionnez l’option pour **ajouter une règle**.

5. Créons une règle qui augmente le nombre d’instances de machine virtuelle dans un groupe identique lorsque la charge moyenne du processeur est supérieure à 70 % pendant 10 minutes. Lorsque la règle déclenche l’opération, le nombre d’instances de machine virtuelle est augmenté de 20 %. Dans des groupes identiques comportant un petit nombre d’instances de machine virtuelle, vous pouvez définir le paramètre **Opération** sur *Augmenter le nombre de* puis spécifier la valeur *1* ou *2* pour le *nombre d’instances*. Dans des groupes identiques comportant un grand nombre d’instances de machine virtuelle, une augmentation de 10 ou 20 % d’instances de machine virtuelle peut être plus appropriée.

    Spécifiez les paramètres suivants pour votre règle :
    
    | Paramètre              | Explication                                                                                                         | Valeur          |
    |------------------------|---------------------------------------------------------------------------------------------------------------------|----------------|
    | *Agrégation de temps*     | Définit la manière dont les métriques collectées doivent être agrégées à des fins d’analyse.                                                | Average        |
    | *Nom de métrique*          | Métrique de performances à surveiller et sur laquelle appliquer des actions de groupe identique.                                                   | Percentage CPU |
    | *Statistiques de fragment de temps* | Définit la manière dont les métriques collectées dans chaque fragment de temps doivent être agrégées à des fins d’analyse.                             | Average        |
    | *Opérateur*             | Opérateur utilisé pour comparer les données de métrique au seuil.                                                     | Supérieur à   |
    | *Seuil*            | Pourcentage qui amène la règle de mise à l’échelle automatique à déclencher une action.                                                 | 70             |
    | *Durée*             | Temps de surveillance avant que les valeurs de métrique et de seuil soient comparées. N’inclut pas de période de refroidissement.                                   | 10 minutes     |
    | *opération*            | Définit si le groupe identique doit augmenter ou réduire l’échelle lorsque la règle s’applique et avec quel incrément.                        | Augmenter le pourcentage de |
    | *Nombre d’instances*       | Le pourcentage d’instances de machine virtuelle doit être modifié lorsque la règle se déclenche.                                            | 20             |
    | *Refroidissement (minutes)*  | Temps d’attente avant que la règle soit appliquée à nouveau afin que les actions de mise à l’échelle automatique aient le temps de porter effet. | 5 minutes      |

    Les exemples suivants montrent une règle créée dans le portail Azure et qui correspond à ces paramètres :

    ![Créer une règle de mise à l’échelle automatique pour augmenter le nombre d’instances de machine virtuelle](media/virtual-machine-scale-sets-autoscale-portal/rule-increase.png)

6. Pour créer la règle, sélectionnez **Ajouter**


## <a name="create-a-rule-to-automatically-scale-in"></a>Créer une règle pour effectuer automatiquement un scale-in
Au cours d’une soirée ou d’un week-end, la demande de votre application peut diminuer. Si cette charge réduite est constante pendant un certain temps, vous pouvez configurer des règles de mise à l’échelle automatique pour réduire le nombre d’instances de machine virtuelle dans le groupe identique. Cette action de diminution du nombre d’instances a pour effet de réduire le coût d’exécution de votre groupe identique, car vous seul exécutez le nombre d’instances requis pour répondre à la demande en cours.

1. Choisissez de nouveau l’option **Ajouter une règle**.
2. Créez une règle qui diminue le nombre d’instances de machine virtuelle dans un groupe identique lorsque la charge moyenne du processeur est inférieure à 30 % pendant 10 minutes. Lorsque la règle déclenche l’opération, le nombre d’instances de machine virtuelle est réduit de 20 %.

    Utilisez la même approche que la règle précédente. Ajustez les paramètres suivants pour votre règle :
    
    | Paramètre              | Explication                                                                                                          | Valeur          |
    |------------------------|----------------------------------------------------------------------------------------------------------------------|----------------|
    | *Opérateur*             | Opérateur utilisé pour comparer les données de métrique au seuil.                                                      | Inférieur à   |
    | *Seuil*            | Pourcentage qui amène la règle de mise à l’échelle automatique à déclencher une action.                                                 | 30             |
    | *opération*            | Définit si le groupe identique doit augmenter ou réduire l’échelle lorsque la règle s’applique et avec quel incrément                         | Diminuer le pourcentage de |
    | *Nombre d’instances*       | Le pourcentage d’instances de machine virtuelle doit être modifié lorsque la règle se déclenche.                                             | 20             |

3. Pour créer la règle, sélectionnez **Ajouter**


## <a name="define-autoscale-instance-limits"></a>Définir des limites pour la mise à l’échelle automatique des instances
Votre profil de mise à l’échelle automatique doit définir un nombre minimal, maximal et par défaut d’instances de machine virtuelle. Lorsque vos règles de mise à l’échelle automatique sont appliquées, assurez-vous que votre scale-out ne dépasse pas le nombre maximal d’instances ou que votre scale-in ne descend pas en dessous du nombre minimal d’instances.

1. Définissez les limites d’instance suivantes :

    | Minimum | Maximale | Default|
    |---------|---------|--------|
    | 2       | 10      | 2      |

2. Pour appliquer vos règles de mise à l’échelle automatique et vos limites d’instance, sélectionnez **Enregistrer**.


## <a name="monitor-number-of-instances-in-a-scale-set"></a>Surveiller le nombre d’instances dans un groupe identique
Pour voir le nombre et l’état des instances de machine virtuelle, sélectionnez **Instances** dans le menu à gauche de la fenêtre des groupes identiques. L’état indique si l’instance de la machine virtuelle crée (*Creating*) à mesure que le groupe identique augmente automatiquement le nombre d’instances, ou supprime (*Deleting*) à mesure que le groupe identique diminue automatiquement le nombre d’instances.

![Afficher une liste des instances de machine virtuelle du groupe identique](media/virtual-machine-scale-sets-autoscale-portal/view-instances.png)


## <a name="autoscale-based-on-a-schedule"></a>Mise à l’échelle automatique en fonction d’une planification
Les exemples précédents ont automatiquement mis à l’échelle un groupe identique en fonction de métriques de base de l’ordinateur hôte, telles que l’utilisation du processeur. Vous pouvez également créer des règles de mise à l’échelle automatique basées sur des planifications. Ces règles basées sur une planification vous permettent d’effectuer un scale-out automatiquement du nombre d’instances de machine virtuelle avant une augmentation anticipée de la demande d’une application, par exemple, aux heures de travail habituelles, et d’effectuer un scale-in automatiquement du nombre d’instances aux heures auxquelles vous anticipez moins de demandes, par exemple, durant le week-end.

1. Choisissez **Mise à l’échelle** dans le menu à gauche de la fenêtre des groupes identiques. Pour supprimer des règles de mise à l’échelle automatique existantes créées dans les exemples précédents, choisissez l’icône Corbeille.

    ![Supprimer les règles de la mise à l’échelle automatique existantes](media/virtual-machine-scale-sets-autoscale-portal/delete-rules.png)

2. Choisissez **Ajouter une condition de mise à l’échelle**. Sélectionnez l’icône crayon en regard du nom de la règle puis saisissez un nom comme *Scale-out à chaque journée de travail*.

    ![Renommer la règle de mise à l’échelle automatique par défaut](media/virtual-machine-scale-sets-autoscale-portal/rename-rule.png)

3. Sélectionnez la case d’option **Mettre à l'échelle à un nombre d'instances spécifique**.
4. Pour augmenter le nombre d’instances, entrez *10* comme nombre d’instances.
5. Choisissez **Répéter des jours spécifiques** pour le type **Planification**.
6. Sélectionnez tous les jours de travail, du lundi au vendredi.
7. Choisissez le fuseau horaire approprié, puis réglez **l’heure de début** sur *09:00*.
8. Choisissez de nouveau **Ajouter une condition de mise à l’échelle**. Répétez cette procédure pour créer une planification nommée *Effectuer un scale-in en soirée* qui réduit à *3* le nombre d’instances, se répète tous les jours de la semaine et commence à *18:00*.
9. Pour appliquer vos règles de mise à l’échelle automatique basées sur une planification, sélectionnez **Enregistrer**.

    ![Créer des règles de mise à l’échelle automatique basée sur une planification](media/virtual-machine-scale-sets-autoscale-portal/schedule-autoscale.PNG)

Pour voir comment vos règles de mise à l’échelle automatique sont appliquées, sélectionnez **Historique des exécutions** en haut de la fenêtre **Mise à l’échelle**. Le graphe et la liste des événements indiquent les déclenchements des règles de mise à l’échelle automatique ainsi que les augmentations ou diminutions du nombre d’instances de machines virtuelles dans votre groupe identique.


## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à utiliser des règles de mise à l’échelle automatique pour mettre à l’échelle horizontalement, et augmenter ou diminuer le *nombre* d’instances de machine virtuelle dans votre groupe identique. Vous pouvez également mettre à l’échelle verticalement pour augmenter ou diminuer la *taille* d’instance de machine virtuelle. Pour plus d’informations, voir [Mise à l’échelle verticale avec des groupes de machines virtuelles identiques](virtual-machine-scale-sets-vertical-scale-reprovision.md).

Pour plus d’informations sur la gestion de vos instances de machine virtuelle, voir [Gérer les groupes de machines virtuelles identiques avec Azure PowerShell](./virtual-machine-scale-sets-manage-powershell.md).

Pour découvrir comment générer des alertes lorsque vos règles de mise à l’échelle automatique déclenchent, voir [Utilisation d’actions de mise à l’échelle automatique pour envoyer des notifications d’alerte webhook et par courrier électronique dans Azure Monitor](../azure-monitor/platform/autoscale-webhook-email.md). Vous pouvez également [Utiliser des journaux d’audit pour envoyer des notifications d’alerte webhook et par courrier électronique dans Azure Monitor](../azure-monitor/platform/alerts-log-webhook.md).
