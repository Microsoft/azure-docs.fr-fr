---
title: Gestion des modèles de bande passante pour la gamme StorSimple 8000 | Microsoft Docs
description: Décrit comment gérer les modèles de bande passante StorSimple, ce qui vous permet de contrôler la consommation de bande passante.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/29/2017
ms.author: alkohli
ms.openlocfilehash: 56170ffbbfe14248bcfd1f94549a3565873f8646
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005892"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-storsimple-bandwidth-templates"></a>Utiliser le service StorSimple Device Manager pour gérer les modèles de bande passante StorSimple

## <a name="overview"></a>Vue d’ensemble

Les modèles de bande passante vous permettent de configurer l’utilisation de la bande passante réseau sur plusieurs planifications selon le moment de la journée des couches de données à partir de l’appareil StorSimple vers le cloud.

Avec les planifications de limitation de bande passante, vous pouvez :

* Spécifiez des planifications de bande passante en fonction de l’utilisation du réseau par la charge de travail.
* Centraliser la gestion et réutiliser les planifications sur plusieurs appareils de manière simple et transparente.

> [!NOTE]
> Cette fonctionnalité est disponible uniquement pour les appareils physiques StorSimple (modèles 8100 et 8600) et non pour les StorSimple Cloud Appliances (modèles 8010 et 8020).


## <a name="the-bandwidth-templates-blade"></a>Le panneau Modèles de bande passante

Le panneau **Modèles de bande passante** affiche tous les modèles de bande passante pour votre service dans un format tabulaire et contient les informations suivantes :

* **Nom** : nom unique affecté au modèle de bande passante lors de sa création.
* **Planification** : nombre de planifications contenues dans un modèle donné de bande passante.
* **Utilisé par** : nombre de volumes utilisant les modèles de bande passante.

Vous trouverez également des informations supplémentaires pour aider à configurer les modèles de bande passante dans :

* [Questions et réponses sur les modèles de bande passante](#questions-and-answers-about-bandwidth-templates)
* [Meilleures pratiques pour les modèles de bande passante](#best-practices-for-bandwidth-templates)

## <a name="add-a-bandwidth-template"></a>Ajouter un modèle de bande passante

Pour créer un modèle de bande passante, procédez comme suit.

#### <a name="to-add-a-bandwidth-template"></a>Pour ajouter un modèle de bande passante

1. Accédez à votre service StorSimple Device Manager, cliquez sur **Modèles de bande passante**, puis cliquez sur **+ Ajouter un modèle de bande passante**.

    ![Cliquer sur + Ajouter un modèle de bande passante](./media/storsimple-8000-manage-bandwidth-templates/addbwtemp1.png)

2. Dans le panneau **Ajouter un modèle de bande passante**, effectuez les étapes suivantes :
   
    1. Spécifiez un nom unique pour votre modèle de bande passante.
    2. Définissez une planification de bande passante. Pour créer une planification :
   
        1. Dans la liste déroulante, sélectionnez les **Jours** de la semaine pour lesquels la planification est configurée. Vous pouvez sélectionner plusieurs jours.        
        
        2. Entrez une **Heure de début** au format _hh:mm_. Cette heure correspond au début de la planification.

        3. Entrez une **Heure de fin** au format _hh:mm_. Cette heure correspond à la fin de la planification.
      
           > [!NOTE]
           > Les planifications qui se chevauchent ne sont pas autorisées. Si les heures de début et de fin entraînent une planification qui se chevauche, un message d'erreur apparaîtra.

        4. Spécifiez le **Débit de bande passante**. Il s’agit de la bande passante en mégabits par seconde (Mbits/s) utilisée par votre appareil StorSimple dans les opérations impliquant le cloud (à la fois pour les chargements et les téléchargements). Fournissez un nombre compris entre 1 et 1 000 pour ce champ.

            ![Définir une planification de bande passante](./media/storsimple-8000-manage-bandwidth-templates/addbwtemp2.png)
         
            Répétez les étapes ci-dessus pour définir plusieurs planifications pour votre modèle jusqu’à ce que vous ayez terminé.

        5. Cliquez sur **Ajouter** pour commencer à créer un modèle de bande passante. Le modèle créé est ajouté à la liste des modèles de bande passante.
      

## <a name="edit-a-bandwidth-template"></a>Modifier un modèle de bande passante

Pour modifier un modèle de bande passante, procédez comme suit.

### <a name="to-edit-a-bandwidth-template"></a>Pour modifier un modèle de bande passante

1. Accédez à votre service StorSimple Device Manager et cliquez sur **Modèles de bande passante**.
2. Dans la liste des modèles de bande passante, sélectionnez le modèle que vous souhaitez supprimer. Cliquez dessus avec le bouton droit et sélectionnez **Supprimer** dans le menu contextuel.
3. Cliquez sur **OK** lorsque vous êtes invité à confirmer l’opération. Le modèle de bande passante doit être supprimé. 
4. La liste des modèles de bande passante est mise à jour afin de refléter la suppression.

> [!NOTE]
> Vous ne pouvez pas enregistrer vos modifications si la planification modifiée chevauche une planification existante dans le modèle de bande passante que vous modifiez.

## <a name="delete-a-bandwidth-template"></a>Supprimer un modèle de bande passante

Pour supprimer un modèle de bande passante, procédez comme suit.

#### <a name="to-delete-a-bandwidth-template"></a>Pour supprimer un modèle de bande passante

1. Accédez à votre service StorSimple Device Manager et cliquez sur **Modèles de bande passante**.
2. Dans la liste des modèles de bande passante, sélectionnez le modèle que vous souhaitez supprimer. Cliquez dessus avec le bouton droit et sélectionnez Supprimer dans le menu contextuel.
3. Cliquez sur **OK** lorsque vous êtes invité à confirmer l’opération. Le modèle de bande passante doit être supprimé.
4. La liste des modèles de bande passante est mise à jour afin de refléter la suppression.

Si le modèle est en cours d’utilisation par des volumes, vous n’êtes pas autorisé à le supprimer. Vous verrez un message d’erreur indiquant que le modèle est en cours d’utilisation. Un message d’erreur s’affiche vous indiquant que toutes les références au modèle doivent être supprimées.

Vous pouvez supprimer toutes les références au modèle en accédant à la page **Conteneurs de volumes** et en modifiant les conteneurs de volume qui utilisent ce modèle afin qu’ils utilisent un autre modèle ou un paramètre de bande passante personnalisé ou illimité. Lorsque toutes les références ont été supprimées, vous pouvez supprimer le modèle.

## <a name="use-a-default-bandwidth-template"></a>Utiliser un modèle de bande passante par défaut

Un modèle de bande passante par défaut est fourni et utilisé par les conteneurs de volumes par défaut pour appliquer les contrôles de bande passante lors de l’accès au cloud. Le modèle par défaut sert également de référence prête pour les utilisateurs qui créent leurs propres modèles. Les détails du modèle par défaut sont les suivants :

* **Nom** : nombre illimité de nuit et de week-ends
* **Planification** : une seule planification du lundi au vendredi qui applique un taux de bande passante de 1 Mbits/s entre 8 h 00 et 17 h 00, heure de l’appareil. Pour le reste de la semaine, la bande passante est définie sur Unlimited.

Le modèle par défaut peut être modifié. L’utilisation de ce modèle (y compris les versions modifiées) est suivie.

## <a name="create-an-all-day-bandwidth-template-that-starts-at-a-specified-time"></a>Créer un modèle de bande passante sur une journée entière qui commence à une heure spécifiée

Suivez cette procédure pour créer une planification qui démarre à une heure spécifiée et s’exécute toute la journée. Dans l’exemple, la planification commence à 9 heures du matin et s’exécute jusqu’à 9 h le lendemain matin. Il est important de noter que les heures de début et de fin d’une planification donnée doivent être contenues dans la même planification de 24 heures et ne peuvent pas couvrir plusieurs jours. Si vous avez besoin configurer des modèles de bande passante qui s’étendent sur plusieurs jours, vous devrez utiliser plusieurs planifications (comme illustré dans l’exemple).

#### <a name="to-create-an-all-day-bandwidth-template"></a>Pour créer un modèle de bande passante sur une journée entière

1. Créez une planification qui commence à 9 heures du matin et s’exécute jusqu’à minuit.
2. Ajoutez une autre planification. Configurez la deuxième planification pour qu’elle s’exécute à partir de minuit jusqu’à 9 heures du matin.
3. Enregistrez le modèle de bande passante.

La planification composite démarre ensuite à une heure de votre choix et s’exécute toute la journée.

## <a name="questions-and-answers-about-bandwidth-templates"></a>Questions et réponses sur les modèles de bande passante

**Q**. Qu’advient-il des contrôles de bande passante lorsque vous êtes entre deux planifications ? (Une planification est terminée et une autre n’a pas encore démarré).

**R**. Dans ce cas, aucun contrôle de bande passante n’est utilisé. Cela signifie que le périphérique peut utiliser une bande passante illimitée lors de la hiérarchisation des données vers le cloud.

**Q**. Peut modifier les modèles de bande passante sur un périphérique hors connexion ?

**R**. Vous ne pouvez pas modifier les modèles de bande passante sur les conteneurs de volumes si l’appareil correspondant est hors connexion.

**Q**. Peut modifier un modèle de bande passante associé à un conteneur de volumes lorsque les volumes associés sont hors connexion ?

**R**. Vous pouvez modifier un modèle de bande passante associé à un conteneur de volumes dont les volumes associés sont hors connexion ? Notez que lorsque les volumes sont hors connexion, aucune donnée n’est hiérarchisée de l’appareil vers le cloud.

**Q**. Peut-on supprimer un modèle par défaut ?

**R**. Même si vous pouvez le faire, il est déconseillé de supprimer un modèle par défaut. L’utilisation d’un modèle par défaut, y compris les versions modifiées, est suivie. Les données de suivi sont analysées et, au fil du temps, sont utilisées pour améliorer le modèle par défaut.

**Q**. Comment déterminer que vos modèles de bande passante doivent être modifiés ?

**R**. Lorsque vous commencez à constater que le réseau ralentit ou se retrouve bloqué plusieurs fois par jour, il est temps que vous modifiiez les modèles de bande passante. Si tel est le cas, surveillez le stockage et l’utilisation du réseau en examinant les graphiques des performances d’E/S et de débit du réseau.

À partir des données de débit du réseau, identifiez l’heure et les conteneurs de volumes où le goulot d’étranglement du réseau s’est produit. Si cela se produit lorsque les données sont en cours de hiérarchisation vers le cloud (obtenez ces informations à partir des performances d’E/S de tous les conteneurs de volume pour l’appareil vers le cloud), vous devez modifier les modèles de la bande passante associés à vos conteneurs de volumes.

Une fois que les modèles modifiés sont en cours d’utilisation, vous devez surveiller à nouveau le réseau en cas de latences importantes. Si des latences subsistent, vous devez revoir vos modèles de bande passante.

**Q**. Que se passe-t-il si plusieurs conteneurs de volumes sur mon appareil ont des planifications qui se chevauchent, mais que différentes limites s’appliquent à chacune ?

**R**. Supposons que vous disposiez d’un appareil avec 3 conteneurs de volumes. Les planifications associées à ces conteneurs se chevauchent totalement. Pour chacun de ces conteneurs, les limites de bande passante utilisées sont respectivement 5, 10 et 15 Mbits/s. Lorsque les E/S se produisent sur l’ensemble de ces conteneurs en même temps, la valeur minimale des 3 limites de bande passante peut s’appliquer : dans ce cas, 5 Mbits/s, car ces demandes sortantes d’E/S partagent la même file d’attente.

## <a name="best-practices-for-bandwidth-templates"></a>Meilleures pratiques pour les modèles de bande passante

Suivez ces meilleures pratiques pour votre appareil StorSimple :

* Configurez les modèles de bande passante sur votre appareil pour activer la limitation variable du débit réseau par l’appareil à différentes heures de la journée. Ces modèles de bande passante lorsqu’ils sont utilisés avec les planifications de sauvegarde peuvent exploiter efficacement une bande passante réseau supplémentaire pour les opérations de cloud pendant les heures creuses.
* Calculez la bande passante réelle requise pour un déploiement spécifique en fonction de la taille du déploiement et de l’objectif de délai de récupération.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’[utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

