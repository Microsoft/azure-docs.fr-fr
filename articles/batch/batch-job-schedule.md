---
title: Planifier vos travaux
description: Utilisez la planification des travaux pour gérer vos tâches.
ms.topic: how-to
ms.date: 02/20/2020
ms.custom: seodec18
ms.openlocfilehash: 7da3c78e00f5d7e41a5396603cf4885a50cb6e5c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89146349"
---
# <a name="schedule-jobs-for-efficiency"></a>Planifier des travaux pour gagner en efficacité

La planification des travaux de traitement par lots vous permet de classer par ordre de priorité les travaux que vous souhaitez exécuter en premier tout en prenant en compte les tâches qui ont des dépendances avec d’autres tâches. En planifiant vos travaux, vous pouvez vous assurer d’utiliser le moins de ressources possible. Les nœuds peuvent être mis hors service lorsqu’ils ne sont pas nécessaires, et les tâches qui dépendent d’autres tâches sont lancées juste-à-temps pour optimiser les workflows. Un seul travail est exécuté à la fois. Un nouveau ne démarre pas tant que le précédent n’est pas terminé. Vous pouvez configurer votre travail de sorte qu’il se termine automatiquement. 

## <a name="benefit-of-job-scheduling"></a>Avantage de la planification des travaux

L’avantage de la planification des travaux est que vous pouvez spécifier une planification pour la création des travaux. Les tâches que vous planifiez à l’aide de la tâche du gestionnaire de travaux sont associées à un travail. La tâche du gestionnaire de travaux créera des tâches pour le travail. Pour ce faire, la tâche du gestionnaire de travaux doit s’authentifier auprès du compte Batch. Utilisez le jeton d’accès AZ_BATCH_AUTHENTICATION_TOKEN. Le jeton autorise l’accès au reste du travail. 

## <a name="use-the-portal-to-schedule-a-job"></a>Utiliser le portail pour planifier un travail

   1. Connectez-vous au [portail Azure](https://portal.azure.com/).

   2. Sélectionnez le compte Batch dans lequel vous souhaitez planifier des travaux.

   3. Sélectionnez **Ajouter** pour créer une planification de travail et renseignez le **formulaire de base**.



![Planifier un travail][1]

**ID de planification du travail** : Identificateur unique de cette planification de travail.

**Nom d'affichage** : Le nom d’affichage du travail n’a pas besoin d’être unique, mais sa longueur maximale est de 1024 caractères.

**Ne pas exécuter avant** : Spécifie l’heure à laquelle le travail est exécuté au plus tôt. Si vous ne la définissez pas, la planification est prête à exécuter les travaux immédiatement.

**Ne pas exécuter après** : Aucun travail n’est exécuté après l’heure définie ici. Si vous ne spécifiez pas d’heure, vous créez une planification de travail périodique qui reste active jusqu’à ce que vous la terminiez explicitement.

**Intervalle de périodicité** : Vous pouvez spécifier le laps de temps entre les travaux. Vous ne pouvez avoir qu’un seul travail planifié à la fois. Par conséquent, s’il est temps de créer un nouveau travail dans le cadre d’une planification de travail, mais que le travail précédent est encore en cours d’exécution, le service Batch ne crée pas le nouveau travail tant que le travail précédent n’est pas terminé.  

**Fenêtre de démarrage** : Ici, vous spécifiez l’intervalle de temps allant du moment où la planification indique qu’un travail doit être créé jusqu’à ce que travail soit terminé. Si le travail en cours ne se termine pas dans sa fenêtre, le travail suivant ne démarre pas.

En bas du formulaire de base, vous allez spécifier le pool sur lequel vous souhaitez que le travail s’exécute. Pour connaître l’ID de votre pool, sélectionnez **Mettre à jour**. 

![Spécifier le pool][2]


**ID de pool** : Identifiez le pool sur lequel vous allez exécuter le travail.

**Tâche de configuration du travail** : Sélectionnez **Mettre à jour** pour nommer la tâche du gestionnaire de travaux, ainsi que les tâches de préparation et de lancement du travail, si vous en utilisez.

**Priorité** : Attribuez une priorité au travail.

**Durée maximale de l’horloge** : Définissez la durée maximale pendant laquelle le travail peut s’exécuter. S’il ne se termine pas dans le délai d’exécution,Batch termine le travail. Si vous ne définissez pas cette valeur, il n’y a aucune limite de durée pour le travail.

**Nombre maximal de tentatives de tâche** : Précisez le nombre de fois qu’une tâche peut être retentée jusqu’à un maximum de quatre fois. Ce n’est pas le même que le nombre de nouvelles tentatives qu’un appel d’API peut avoir.

**Lorsque toutes les tâches sont terminées** : La valeur par défaut est aucune action.

**Lorsqu’une tâche échoue** : La valeur par défaut est aucune action. Une tâche échoue si le nombre de tentatives est épuisé ou si une erreur s’est produite lors du démarrage de la tâche. 

Après avoir sélectionné **Enregistrer**, si vous accédez à **Planifications de travaux** dans le volet de navigation gauche, vous pouvez suivre l’exécution du travail en sélectionnant **Informations d’exécution**.


## <a name="for-more-information"></a>Informations supplémentaires

Pour gérer un travail à l’aide d’Azure CLI, consultez [az batch job-schedule](/cli/azure/batch/job-schedule).

## <a name="next-steps"></a>Étapes suivantes

[Créer des dépendances de tâches pour exécuter des tâches qui dépendent d’autres tâches](batch-task-dependencies.md).





[1]: ./media/batch-job-schedule/add_job_schedule-02.png
[2]: ./media/batch-job-schedule/add_job_schedule-03.png


