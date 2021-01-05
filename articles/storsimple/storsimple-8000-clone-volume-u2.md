---
title: Clonage d’un volume sur la gamme StorSimple 8000 | Microsoft Docs
description: Décrit les différents types de clone et leur utilisation, et explique comment vous pouvez utiliser un jeu de sauvegarde pour cloner un volume individuel sur un appareil de la gamme StorSimple 8000.
services: storsimple
documentationcenter: NA
author: alkohli
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.date: 07/15/2020
ms.author: alkohli
ms.openlocfilehash: 5ebb7f1d9440bc7a8b75afe64403762532213c1d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91767566"
---
# <a name="use-the-storsimple-device-manager-service-in-azure-portal-to-clone-a-volume"></a>Utiliser le service StorSimple Device Manager dans le portail Azure pour cloner un volume

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel explique comment vous pouvez utiliser un jeu de sauvegarde pour cloner un volume individuel via le panneau **Catalogue de sauvegarde**. Il explique également la différence entre les clones *temporaires* et *permanents*. Les instructions de ce didacticiel s’applique à tous les appareils de la gamme StorSimple 8000 exécutant Update 3 ou une version ultérieure.

Le panneau **Catalogue de sauvegarde** du service StorSimple Device Manager affiche tous les jeux de sauvegarde créés lors de sauvegardes manuelles ou automatisées. Vous pouvez ensuite sélectionner un volume dans un jeu de sauvegarde à cloner.

 ![Liste des jeux de sauvegarde](./media/storsimple-8000-clone-volume-u2/bucatalog.png)

## <a name="considerations-for-cloning-a-volume"></a>Remarques relatives au clonage d’un volume

Tenez compte des informations suivantes lors du clonage d’un volume.

- Un clone se comporte de la même façon qu’un volume normal. Toute opération possible sur un volume est disponible pour le clone.

- La surveillance et la sauvegarde par défaut sont automatiquement désactivées sur un volume cloné. Vous devriez configurer un volume cloné pour toutes les sauvegardes.

- Un volume épinglé localement est cloné comme un volume à plusieurs niveaux. Si vous voulez que le volume cloné soit épinglé localement, vous pouvez convertir le clone en un volume localement épinglé une fois l'opération de clonage terminée. Pour plus d'informations sur la conversion d'un volume à plusieurs niveaux en volume épinglé localement, consultez la page [Changer de type de volume](storsimple-8000-manage-volumes-u2.md#change-the-volume-type).

- Si vous essayez de convertir un volume cloné de type plusieurs niveaux au type localement épinglé immédiatement après le clonage (lorsqu’il est toujours un clone temporaire), la conversion échoue avec le message d’erreur suivant :

    `Unable to modify the usage type for volume {0}. This can happen if the volume being modified is a transient clone and hasn’t been made permanent. Take a cloud snapshot of this volume and then retry the modify operation.`

    Cette erreur est signalée uniquement si vous effectuez un clonage sur un autre appareil. Vous pouvez convertir correctement le volume au type épinglé localement si vous convertissez tout d'abord le clone temporaire en clone permanent. Prenez un instantané cloud du clone temporaire pour le convertir en clone permanent.

## <a name="create-a-clone-of-a-volume"></a>Création du clone d’un volume

Vous pouvez créer un clone sur le même appareil, sur un autre ou même une appliance cloud à l’aide d’un instantané local ou cloud.

La procédure suivante décrit comment créer un clone à partir du catalogue de sauvegarde.  

Procédez comme suit pour créer un clone de votre volume à partir du catalogue de sauvegarde.

#### <a name="to-clone-a-volume"></a>Clonage d’un volume

1. Accédez à votre service StorSimple Device Manager, puis cliquez sur **Catalogue de sauvegarde**.

2. Sélectionnez un jeu de sauvegarde comme suit :
   
   1. Sélectionnez l’appareil approprié.
   2. Dans la liste déroulante, choisissez la stratégie de sauvegarde ou le volume pour la sauvegarde à sélectionner.
   3. Indiquez l’intervalle de temps.
   4. Cliquez sur **Appliquer** pour exécuter cette requête.

      Les sauvegardes associées au volume ou à la stratégie de sauvegarde sélectionné doivent figurer dans la liste des jeux de sauvegarde.
   
      ![Liste des jeux de sauvegarde 2](./media/storsimple-8000-clone-volume-u2/bucatalog.png)
     
3. Développez le jeu de sauvegarde pour afficher le volume associé et sélectionner un volume dans le jeu de sauvegarde. Cliquez dessus avec le bouton droit puis sélectionnez **Cloner** dans le menu contextuel.

   ![Liste des jeux de sauvegarde 3](./media/storsimple-8000-clone-volume-u2/clonevol3b.png) 

3. Dans le panneau **Cloner**, effectuez les étapes suivantes :
   
   1. Identifiez un appareil cible. Il s’agit de l’emplacement où sera créé le clone. Vous pouvez choisir le même appareil ou en spécifier un autre.

      > [!NOTE]
      > Assurez-vous que la capacité requise pour le clone est inférieure à la capacité disponible sur le périphérique cible.
       
   2. Indiquez un nom de volume unique pour votre clone. Le nom doit contenir entre 3 et 127 caractères.
      
       > [!NOTE]
       > Le champ **Cloner le volume comme** sera **Hiérarchisé** même si vous clonez un volume épinglé localement. Vous ne pouvez pas modifier ce paramètre. Toutefois, si vous voulez que le volume cloné soit aussi épinglé localement, une fois le clone créé, vous pouvez le convertir en volume épinglé localement. Pour plus d'informations sur la conversion d'un volume à plusieurs niveaux en volume épinglé localement, consultez la page [Changer de type de volume](storsimple-8000-manage-volumes-u2.md#change-the-volume-type).
          
   3. Sous **Hôtes connectés**, spécifiez un enregistrement de contrôle d’accès (ACR) pour le clone. Vous pouvez ajouter un nouvel ACR ou en choisir un dans la liste existante. L’ACR déterminera les hôtes qui peuvent accéder à ce clone.
      
       ![Liste des jeux de sauvegarde 4](./media/storsimple-8000-clone-volume-u2/clonevol3a.png) 

   4. Cliquez sur **Cloner** pour terminer l’opération.

4. Un travail de clonage est lancé et vous êtes informé lorsque le clone est créé avec succès. Cliquez sur la notification du travail ou accédez au panneau **Tâches** pour surveiller le travail de clonage.

    ![Liste des jeux de sauvegarde 5](./media/storsimple-8000-clone-volume-u2/clonevol5.png)

7. Lorsque le travail de clonage est terminé, accédez à votre appareil, puis cliquez sur **Volumes**. Dans la liste des volumes, vous devez voir le clone que vous venez de créer dans le même conteneur de volumes que le volume source.

    ![Liste des jeux de sauvegarde 6](./media/storsimple-8000-clone-volume-u2/clonevol6.png)

Un clone créé de cette manière est un clone temporaire. Pour plus d’informations sur les types de clone, consultez la page [Clones temporaires et permanents](#transient-vs-permanent-clones).


## <a name="transient-vs-permanent-clones"></a>Clones temporaires et permanents
Des clones temporaires sont créés uniquement lorsque vous clonez sur un autre appareil. Vous pouvez cloner un volume spécifique à partir d’un jeu de sauvegarde sur un autre appareil géré par StorSimple Device Manager. Le clone temporaire a des références aux données dans le volume d’origine et utilise ces données pour la lecture et l’écriture en local sur l’appareil cible.

Si vous prenez un instantané cloud d’un clone temporaire, le clone qui en résulte devient *permanent*. Pendant ce processus, une copie des données est créée sur le cloud et le temps nécessaire à la copie de ces données varie selon la taille des données et les latences Azure (il s’agit d’une copie interne à Azure). Ce processus peut prendre plusieurs jours voire semaines. Le clone temporaire devient un clone permanent et n’a pas de références aux données du volume d’origine à partir duquel il a été cloné.

## <a name="scenarios-for-transient-and-permanent-clones"></a>Scénarios pour les clones temporaires et permanents
Les sections suivantes décrivent des exemples de situations dans lesquelles les clones temporaires et permanents peuvent être utilisés.

### <a name="item-level-recovery-with-a-transient-clone"></a>Récupération au niveau de l’élément avec un clone temporaire
Vous devez récupérer un fichier de présentation Microsoft PowerPoint datant d’un an. Votre administrateur informatique identifie la sauvegarde spécifique dans cette période et filtre ensuite le volume. Ensuite, l’administrateur clone le volume, trouve le fichier que vous recherchez et vous le fournit. Dans ce scénario, un clone temporaire est utilisé.

### <a name="testing-in-the-production-environment-with-a-permanent-clone"></a>Test dans l’environnement de production avec un clone permanent
Vous devez vérifier un bogue de test dans l’environnement de production. Vous créez un clone du volume dans l’environnement de production et vous prenez ensuite un instantané cloud de ce clone pour créer un volume cloné indépendant. Dans ce scénario, un clone permanent est utilisé.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [restaurer un volume StorSimple à partir d’un jeu de sauvegarde](storsimple-8000-restore-from-backup-set-u2.md).
* Découvrez comment [utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

