---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 2abfa29671bd804ee75194ef621fe07f06c015e9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177116"
---
#### <a name="to-create-a-volume"></a>Pour créer un volume
1. Sélectionnez votre appareil dans la liste tabulaire des appareils du panneau **Appareils**. Cliquez sur **+ Ajouter un volume**.

    ![Ajouter un volume](./media/storsimple-8000-create-volume-u2/step5createvol1.png)

2. Dans le panneau **Ajouter un volume** :
   
   1. Le nom de votre appareil actuel est automatiquement ajouté dans le champ **Sélectionner un appareil**.

   2. Dans la liste déroulante, sélectionnez le conteneur de volumes dans lequel vous devez ajouter un volume. 

   3. Saisissez un **nom** pour le volume. Vous ne pouvez pas renommer un volume une fois qu’il a été créé.

   4. Dans la liste déroulante, sélectionnez le **type** de votre volume. Pour les charges de travail qui nécessitent des garanties locales, une faible latence et les meilleures performances possibles, sélectionnez un volume **épinglé localement** . Pour toutes les autres données, sélectionnez un volume **à plusieurs niveaux** . Si vous utilisez ce volume pour les données d’archivage, cochez la case **Utiliser ce volume pour des données d’archivage moins fréquemment sollicitées**.
      
       La configuration d’un volume à plusieurs niveaux est légère, et sa création peut être rapide. La sélection de **Utiliser ce volume pour les données d’archivage moins fréquemment sollicitées** pour le volume à plusieurs niveaux pour les données d’archivage définit la taille de segment de déduplication pour votre volume sur 512 Ko. Si cette option n’est pas activée, le volume à plusieurs niveaux correspondant utilise une taille de segment de 64 Ko. Une grande taille de segment de déduplication permet à l’appareil d’accélérer le transfert des données d’archivage volumineuses vers le cloud.
       
       La configuration d’un volume épinglé localement est complète, et garantit que les données principales sur le volume sont conservées en local sur l’appareil et ne débordent pas sur le cloud.  Si vous créez un volume épinglé localement, l’appareil recherche de l’espace disponible sur les couches locales pour configurer le volume de la taille demandée. L’opération de création d’un volume épinglé localement peut impliquer le débordement des données existantes de l’appareil vers le cloud, et le temps nécessaire pour créer le volume peut être long. La durée totale dépend de la taille du volume configuré, de la bande passante réseau disponible et des données sur votre appareil.

   5. Indiquez la **capacité allouée** pour votre volume. Prenez en note la capacité disponible en fonction du type de volume sélectionné. La taille de volume spécifiée ne doit pas dépasser la quantité d’espace disponible.
      
       Vous pouvez configurer des volumes épinglés localement d’une taille maximale de 8,5 To ou des volumes à plusieurs niveaux d’une taille maximale de 200 To sur l’appareil 8100. Sur l’appareil 8600, qui a une plus grande capacité, vous pouvez configurer des volumes épinglés localement d’une taille maximale de 22,5 To ou des volumes à plusieurs niveaux d’une taille maximale de 500 To. Un espace local sur l’appareil étant nécessaire pour héberger la plage de travail des volumes à plusieurs niveaux, la création de volumes épinglés localement a un impact sur l’espace disponible pour la configuration de volumes à plusieurs niveaux. Par conséquent, si vous créez un volume épinglé localement, l’espace disponible pour la création de volumes à plusieurs niveaux est réduit. De même, si vous créez un volume à plusieurs niveaux, l’espace disponible pour la création de volumes épinglés localement est réduit.
      
       Si vous configurez un volume épinglé localement de 8,5 To (taille maximale autorisée) sur votre appareil 8100, vous avez utilisé tout l’espace local disponible sur l’appareil. Vous ne pourrez donc pas créer de volume à plusieurs niveaux, puisque l’espace local de l’appareil sera insuffisant pour héberger la plage de travail du volume en question. Les volumes à plusieurs niveaux existants affectent également l’espace disponible. Par exemple, si vous avez un appareil 8100 qui possède déjà des volumes à plusieurs niveaux de 106 To, seuls 4 To d’espace sont disponibles pour les volumes épinglés localement.

      1. Cliquez sur la flèche du champ **Hôtes connectés**. 

         ![Hôtes connectés](./media/storsimple-8000-create-volume-u2/step5createvol2.png)

      1. Dans le panneau **Hôtes connectés**, choisissez un ACR existant ou ajoutez-en un en procédant comme suit :

         1. Saisissez un **Nom** pour votre ACR.
         2. Sous **Nom de l’initiateur iSCSI**, indiquez le nom qualifié iSCSI  de votre hôte Windows. Si vous ne possédez pas le nom qualifié, accédez à [Obtenir le nom qualifié d’un hôte Windows Server](#get-the-iqn-of-a-windows-server-host).

      1. Cliquez sur **Créer**. Un volume est créé avec les paramètres spécifiés.

         ![Cliquez sur Create.](./media/storsimple-8000-create-volume-u2/step5createvol3.png)

         > [!NOTE]
         > N’oubliez pas que le volume que vous avez créé ici n’est pas protégé. Vous devez créer des stratégies de sauvegarde et les associer à ce volume pour effectuer des sauvegardes planifiées. 

