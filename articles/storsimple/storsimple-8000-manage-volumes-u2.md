---
title: Gérer des volumes StorSimple (Update 3)
description: Explique comment ajouter, modifier, analyser et supprimer des volumes StorSimple et comment les mettre hors connexion si nécessaire.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 12/08/2017
ms.author: alkohli
ms.openlocfilehash: a8fcadb8bdd4862dd95625228e3c190e5fe19a9a
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91961678"
---
# <a name="use-the-storsimple-device-manager-service-to-manage-volumes-update-3-or-later"></a>Utiliser le service StorSimple Device Manager pour gérer des volumes (Update 3 ou versions ultérieures)

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel explique comment utiliser le service StorSimple Device Manager pour créer et gérer des volumes sur les appareils de la gamme 8000 StorSimple exécutant Update 3 et versions ultérieures.

Le service StorSimple Device Manager est une extension dans le portail Azure qui vous permet de gérer votre solution StorSimple à partir d’une seule interface web. Utilisez le portail Azure pour gérer les volumes sur tous vos appareils. Vous pouvez également créer et gérer des services StorSimple, gérer des appareils, des stratégies de sauvegarde et un catalogue de sauvegarde, et afficher des alertes.

## <a name="volume-types"></a>Types de volume

Les volumes StorSimple peuvent être les suivants :

* **Volumes épinglés localement** : les données de ces volumes restent en permanence sur l’appareil StorSimple local.
* **Volumes hiérarchisés** : les données de ces volumes peuvent se disperser dans le cloud.

Un volume d’archivage est un type de volume hiérarchisé. La grande taille des blocs de déduplication utilisée pour les volumes d’archivage permet à l’appareil de transférer des segments de données plus importants vers le cloud.

Si nécessaire, vous pouvez modifier le type de volume local en volume hiérarchisé, et inversement. Pour plus d’informations, consultez [Modification du type de volume](#change-the-volume-type).

### <a name="locally-pinned-volumes"></a>Volumes épinglés localement

Les volumes épinglés localement sont des volumes entièrement configurés qui ne hiérarchisent pas les données dans le cloud, garantissant ainsi que les données principales restent en local, indépendantes de la connexion au cloud. Les données des volumes épinglés localement ne sont pas dédupliquées ni compressées, mais les instantanés des volumes épinglés localement sont dédupliqués. 

Les volumes épinglés localement sont totalement configurés. Vous devez donc disposer de suffisamment d’espace sur votre appareil lorsque vous les créez. Vous pouvez configurer des volumes épinglés localement jusqu’à une taille maximale de 8 To sur un appareil StorSimple 8100 et de 20 To sur un appareil 8600. StorSimple réserve l’espace restant local de l’appareil pour les instantanés, les métadonnées et le traitement des données. Vous pouvez augmenter la taille d’un volume épinglé localement pour qu’il occupe l’espace maximal disponible, mais vous ne pouvez pas réduire la taille d’un volume après sa création.

Si vous créez un volume épinglé localement, l’espace disponible pour la création de volumes hiérarchisés est réduit. L’inverse est également vrai : si vous disposez de volumes hiérarchisés, l’espace disponible pour la création des volumes épinglés localement sera inférieur, dans les limites maximales susmentionnées. Pour plus d’informations sur les volumes locaux, reportez-vous au [forum aux questions sur les volumes épinglés localement](storsimple-8000-local-volume-faq.md).

### <a name="tiered-volumes"></a>Volumes hiérarchisés

Les volumes hiérarchisés sont des volumes alloués dynamiquement dans lesquels les données fréquemment sollicitées restent locales sur l’appareil alors que les données moins fréquemment utilisées sont hiérarchisées automatiquement vers le cloud. L’allocation dynamique est une technologie de virtualisation dans laquelle le stockage disponible semble dépasser les ressources physiques. Plutôt que de réserver un espace de stockage suffisant à l’avance, StorSimple utilise l’allocation dynamique pour allouer juste assez d’espace pour répondre aux besoins actuels. La nature évolutive du stockage cloud simplifie cette approche, car StorSimple peut augmenter ou diminuer le stockage cloud pour répondre aux demandes changeantes.

Si vous utilisez le volume hiérarchisé pour les données d’archivage, l’activation de la case à cocher **Utiliser ce volume pour les données d’archivage auxquelles vous accédez moins souvent** définit la taille de bloc de déduplication pour votre volume sur 512 Ko. Si cette option n’est pas activée, le volume à plusieurs niveaux correspondant utilise une taille de segment de 64 Ko. Une grande taille de segment de déduplication permet à l’appareil d’accélérer le transfert des données d’archivage volumineuses vers le cloud.


### <a name="provisioned-capacity"></a>Capacité allouée

Reportez-vous au tableau suivant pour connaître la capacité maximale allouée pour chaque type d’appareil et de volume. (Notez que les volumes épinglés localement ne sont pas disponibles sur un appareil virtuel.)

| Type | Taille maximale de volume hiérarchisé | Taille maximale de volume épinglé localement |
| --- | --- | --- |
| **Appareils physiques** | | |
| 8100 |64 To |8 To |
| 8600 |64 To |20 To |
| **Appareils virtuels** | | |
| 8010 |30 To |N/A |
| 8020 |64 To |N/A |

## <a name="the-volumes-blade"></a>Panneau Volumes

Le panneau **Volumes** vous permet de gérer les volumes de stockage alloués sur l’appareil Microsoft Azure StorSimple pour vos initiateurs (serveurs). Il affiche la liste des volumes sur les appareils StorSimple connectés à votre service.

 ![Page volumes](./media/storsimple-8000-manage-volumes-u2/volumeslist.png)

Un volume est constitué d’une série d’attributs :

* **Nom du volume** : nom descriptif qui doit être unique et vous aide à identifier le volume. Ce nom est également utilisé dans les rapports d’analyse lorsque vous filtrez sur un volume particulier. Une fois que le volume est créé, il ne peut pas être renommé.
* **État** : peut être en ligne ou hors connexion. Si un volume est hors connexion, il n’est pas visible pour les initiateurs (serveurs) qui sont autorisés à l’utiliser.
* **Capacité** : spécifie le volume total de données qui peut être stocké par l’initiateur (serveur). Les volumes épinglés localement sont totalement configurés et résident sur l’appareil StorSimple. Les volumes hiérarchisés sont alloués dynamiquement et les données dédupliquées. Si les volumes sont alloués dynamiquement, votre appareil ne préalloue pas de capacité de stockage physique localement ou dans le cloud en fonction de la capacité de volume configurée. La capacité du volume est allouée et utilisée à la demande.
* **Type** : indique si le volume est **Hiérarchisé** (par défaut) ou **Épinglé localement**.

Suivez les instructions de ce didacticiel pour effectuer les tâches suivantes :

* Ajout d’un volume 
* Modification d’un volume 
* Modification du type de volume
* Suppression d’un volume 
* Mise hors connexion d’un volume 
* Analyse d’un volume 

## <a name="add-a-volume"></a>Ajout d’un volume

Vous [avez créé un volume](storsimple-8000-deployment-walkthrough-u2.md#step-6-create-a-volume) lors du déploiement de votre appareil de la gamme 8000 StorSimple. La procédure d’ajout d’un volume est similaire.

#### <a name="to-add-a-volume"></a>Pour ajouter un volume

1. Sélectionnez votre appareil dans la liste tabulaire des appareils du panneau **Appareils**. Cliquez sur **+ Ajouter un volume**.

    ![Ajouter un volume](./media/storsimple-8000-manage-volumes-u2/step5createvol1.png)

2. Dans le panneau **Ajouter un volume** :
   
    1. Le nom de votre appareil actuel est automatiquement ajouté dans le champ **Sélectionner un appareil**.

    2. Dans la liste déroulante, sélectionnez le conteneur de volumes dans lequel vous devez ajouter un volume.

    3.  Saisissez un **nom** pour le volume. Une fois le volume créé, vous ne pouvez pas le renommer.

    4. Dans la liste déroulante, sélectionnez le **type** de votre volume. Pour les charges de travail qui nécessitent des garanties locales, une faible latence et les meilleures performances possibles, sélectionnez un volume **épinglé localement** . Pour toutes les autres données, sélectionnez un volume **à plusieurs niveaux** . Si vous utilisez ce volume pour les données d’archivage, cochez la case **Utiliser ce volume pour des données d’archivage moins fréquemment sollicitées**.
      
       La configuration d’un volume à plusieurs niveaux est légère, et sa création peut être rapide. La sélection de **Utiliser ce volume pour les données d’archivage moins fréquemment sollicitées** pour le volume à plusieurs niveaux pour les données d’archivage définit la taille de segment de déduplication pour votre volume sur 512 Ko. Si cette option n’est pas activée, le volume à plusieurs niveaux correspondant utilise une taille de segment de 64 Ko. Une grande taille de segment de déduplication permet à l’appareil d’accélérer le transfert des données d’archivage volumineuses vers le cloud.
       
       La configuration d’un volume épinglé localement est complète, et garantit que les données principales sur le volume sont conservées en local sur l’appareil et ne débordent pas sur le cloud.  Si vous créez un volume épinglé localement, l’appareil recherche de l’espace disponible sur les couches locales pour configurer le volume de la taille demandée. L’opération de création d’un volume épinglé localement peut impliquer le débordement des données existantes de l’appareil vers le cloud, et le temps nécessaire pour créer le volume peut être long. La durée totale dépend de la taille du volume configuré, de la bande passante réseau disponible et des données sur votre appareil.

    5. Indiquez la **capacité allouée** pour votre volume. Prenez en note la capacité disponible en fonction du type de volume sélectionné. La taille de volume spécifiée ne doit pas dépasser la quantité d’espace disponible.
      
       Vous pouvez configurer des volumes épinglés localement d’une taille maximale de 8,5 To ou des volumes à plusieurs niveaux d’une taille maximale de 200 To sur l’appareil 8100. Sur l’appareil 8600, qui a une plus grande capacité, vous pouvez configurer des volumes épinglés localement d’une taille maximale de 22,5 To ou des volumes à plusieurs niveaux d’une taille maximale de 500 To. Un espace local sur l’appareil étant nécessaire pour héberger la plage de travail des volumes à plusieurs niveaux, la création de volumes épinglés localement a un impact sur l’espace disponible pour la configuration de volumes à plusieurs niveaux. Par conséquent, si vous créez un volume épinglé localement, l’espace disponible pour la création de volumes à plusieurs niveaux est réduit. De même, si vous créez un volume à plusieurs niveaux, l’espace disponible pour la création de volumes épinglés localement est réduit.
      
       Si vous configurez un volume épinglé localement de 8,5 To (taille maximale autorisée) sur votre appareil 8100, vous avez utilisé tout l’espace local disponible sur l’appareil. Vous ne pourrez donc pas créer de volume à plusieurs niveaux, puisque l’espace local de l’appareil sera insuffisant pour héberger la plage de travail du volume en question. Les volumes à plusieurs niveaux existants affectent également l’espace disponible. Par exemple, si vous avez un appareil 8100 qui possède déjà des volumes à plusieurs niveaux de 106 To, seuls 4 To d’espace sont disponibles pour les volumes épinglés localement.

    6. Cliquez sur la flèche du champ **Hôtes connectés**. Dans le panneau **Hôtes connectés**, sélectionnez un ACR existant ou ajoutez un nouvel ACR. Si vous choisissez un nouvel ACR, indiquez un **nom** pour votre ACR, ainsi que le **nom complet iSCSI** (IQN) de votre hôte Windows. Si vous ne possédez pas le nom qualifié, accédez à Obtenir le nom qualifié d’un hôte Windows Server. Cliquez sur **Créer**. Un volume est créé avec les paramètres spécifiés.

        ![Cliquez sur Create.](./media/storsimple-8000-manage-volumes-u2/step5createvol3.png)

Le nouveau volume est maintenant prêt à l’emploi.

> [!NOTE]
> Si vous créez un volume épinglé localement, puis un autre volume du même type immédiatement après, les tâches de création de volume s’exécutent de façon séquentielle. La première tâche de création de volume doit se terminer pour que la deuxième puisse commencer.

## <a name="modify-a-volume"></a>Modification d’un volume

Modifiez un volume lorsque vous avez besoin d’en augmenter la taille ou de modifier les hôtes qui peuvent y accéder.

> [!IMPORTANT]
> * Si vous modifiez la taille du volume sur l’appareil, vous devez la modifier sur l’hôte également.
> * Les étapes côté hôte décrites ici concernent Windows Server 2012 (2012R2). Les procédures pour Linux ou d’autres systèmes d’exploitation sont différentes. Consultez les instructions du système d’exploitation de l’hôte pour modifier le volume sur un hôte exécutant un autre système d’exploitation.

#### <a name="to-modify-a-volume"></a>Pour modifier un volume

1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans la liste tabulaire des appareils, sélectionnez l’appareil comportant le volume que vous souhaitez modifier. Cliquez sur **Paramètres > Volumes**.

    ![Accéder au panneau Volumes](./media/storsimple-8000-manage-volumes-u2/modifyvol2.png)

2. Dans la liste tabulaire de volumes, sélectionnez le volume et cliquez avec le bouton droit pour ouvrir le menu contextuel. Sélectionnez **Mettre hors connexion** pour mettre hors connexion le volume que vous allez modifier.

    ![Sélectionner le volume et le mettre hors connexion](./media/storsimple-8000-manage-volumes-u2/modifyvol4.png)

3. Dans le panneau **Mettre hors connexion**, passez en revue les répercussions de la mise hors connexion du volume et activez la case à cocher qui convient. Assurez-vous avant tout que le volume correspondant sur l’hôte est en mode hors connexion. Pour plus d’informations sur la mise hors connexion d’un volume sur un serveur hôte connecté à StorSimple, reportez-vous aux instructions spécifiques du système d’exploitation. Cliquez sur **Mettre hors connexion**.

    ![Passer en revue les répercussions de la mise hors connexion du volume](./media/storsimple-8000-manage-volumes-u2/modifyvol5.png)

4. Une fois que le volume est hors connexion (comme l’indique l’état du volume), sélectionnez le volume et cliquez avec le bouton droit pour ouvrir le menu contextuel. Sélectionnez **Modifier le volume**.

    ![Sélectionner Modifier le volume](./media/storsimple-8000-manage-volumes-u2/modifyvol9.png)


5. Dans le panneau **Modifier le volume**, vous pouvez apporter les modifications suivantes :
   
   1. Le **nom** du volume ne peut pas être modifié.
   2. Convertissez le **type** Épinglé localement en Hiérarchisé, ou inversement (pour plus d’informations, consultez [Modification du type de volume](#change-the-volume-type)).
   3. Augmenter la **capacité allouée**. La **capacité allouée** peut uniquement être augmentée. Vous ne pouvez pas réduire la taille d’un volume après sa création.
   4. Sous **Hôtes connectés**, vous pouvez modifier l’ACR. Pour modifier un ACR, le volume doit être en mode hors connexion.

       ![Passer en revue les répercussions de la mise hors connexion du volume 2](./media/storsimple-8000-manage-volumes-u2/modifyvol11.png)

5. Cliquez sur **Enregistrer** pour enregistrer vos modifications. Cliquez sur **Oui**lorsque vous êtes invité à confirmer l’opération. Le portail Azure affiche un message de mise à jour de volume. Il affiche un message de réussite quand le volume a été mis à jour avec succès.

    ![Passer en revue les répercussions de la mise hors connexion du volume 3](./media/storsimple-8000-manage-volumes-u2/modifyvol5.png)

7. Si vous développez un volume, procédez comme suit sur votre ordinateur hôte Windows :
   
   1. Accédez à **Gestion de l’ordinateur** ->**Gestion des disques**.
   2. Cliquez avec le bouton droit sur **Gestion des disques**, puis sélectionnez **Analyser les disques de nouveau**.
   3. Dans la liste des disques, sélectionnez le volume que vous avez mis à jour, cliquez avec le bouton droit, puis sélectionnez **Étendre le volume**. L’Assistant Étendre le volume démarre. Cliquez sur **Suivant**.
   4. Parcourez les étapes de l’Assistant en acceptant les valeurs par défaut. Une fois l’Assistant terminé, le volume doit afficher la taille augmentée.
      
      > [!NOTE]
      > Si vous développez un volume épinglé localement, puis un autre volume du même type immédiatement après, les tâches d’expansion de volume s’exécutent de façon séquentielle. La première tâche d’expansion de volume doit se terminer pour que la deuxième puisse commencer.
      

## <a name="change-the-volume-type"></a>Modification du type de volume

Vous pouvez modifier le type de volume hiérarchisé en épinglé localement, et inversement. Toutefois, cette conversion ne doit pas être effectuée fréquemment.

### <a name="tiered-to-local-volume-conversion-considerations"></a>Considérations en matière de conversion d’un volume hiérarchisé en un volume local

Raisons motivant la conversion d’un volume hiérarchisé en volume épinglé localement :

* Assurance de la disponibilité et des performances des données locales.
* Élimination des latences du cloud et des problèmes de connexion au cloud.

En général, il s’agit de petits volumes existants auxquels vous souhaitez accéder fréquemment. Un volume épinglé localement est entièrement configuré lors de sa création. 

Si vous convertissez un volume hiérarchisé en volume épinglé localement, StorSimple vérifie que vous disposez d’un espace suffisant sur l’appareil avant de démarrer la conversion. Si tel n’est pas le cas, vous recevez une erreur et l’opération est annulée. 

> [!NOTE]
> Avant de commencer la conversion d’un volume hiérarchisé en volume épinglé localement, vérifiez que vous tenez compte de l’espace requis par les autres charges de travail. 

Conversion d’un volume hiérarchisé en un volume épinglé localement peut affecter les performances du périphérique. En outre, les facteurs suivants peuvent augmenter le temps que nécessaire pour terminer la conversion :

* La bande passante est insuffisante.
* Il n’existe aucune sauvegarde actuelle.

Pour réduire les effets de ces facteurs :

* Passez en revue vos stratégies de limitation de bande passante et assurez-vous qu’une bande passante dédiée de 40 Mbits/s est disponible.
* Planifiez la conversion pendant les heures creuses.
* Effectuez un instantané cloud avant de commencer la conversion.

Si vous convertissez plusieurs volumes (prenant en charge différentes charges de travail), vous devez donner la priorité à la conversion de volume afin que les volumes de priorité élevée soient convertis en premier. Par exemple, vous devez convertir des volumes hébergeant des machines virtuelles ou des volumes avec des charges de travail SQL avant de convertir des volumes avec des charges de travail à partage de fichiers.

### <a name="local-to-tiered-volume-conversion-considerations"></a>Considérations en matière de conversion d’un volume local en un volume hiérarchisé

Vous souhaiterez peut-être modifier un volume épinglé localement en volume hiérarchisé si vous avez besoin d’espace supplémentaire pour approvisionner d’autres volumes. Lorsque vous convertissez le volume épinglé localement en volume hiérarchisé, la capacité disponible sur l’appareil augmente de la taille de la capacité libérée. Si des problèmes de connectivité empêchent la conversion d’un volume de type local en volume de type hiérarchisé, le volume local présente les propriétés d’un volume hiérarchisé tant que la conversion n’est pas terminée. Cela tient au fait que certaines données peuvent avoir été dispersées dans le cloud. Ces données dispersées continuent d’occuper l’espace local sur l’appareil qui ne peut pas être libéré tant que l’opération n’est pas redémarrée ni terminée.

> [!NOTE]
> La conversion d’un volume peut prendre un certain temps. Vous ne pouvez pas l’annuler une fois qu’elle a commencé. Le volume reste en ligne pendant la conversion, et vous pouvez effectuer des sauvegardes, mais vous ne pouvez pas développer ni restaurer le volume pendant l’exécution de la conversion.


#### <a name="to-change-the-volume-type"></a>Pour modifier le type de volume

1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans la liste tabulaire des appareils, sélectionnez l’appareil comportant le volume que vous souhaitez modifier. Cliquez sur **Paramètres > Volumes**.

    ![Accéder au panneau Volumes 2](./media/storsimple-8000-manage-volumes-u2/modifyvol2.png)

3. Dans la liste tabulaire de volumes, sélectionnez le volume et cliquez avec le bouton droit pour ouvrir le menu contextuel. Sélectionnez **Modifier**.

    ![Sélectionner Modifier dans le menu contextuel](./media/storsimple-8000-manage-volumes-u2/changevoltype2.png)

4. Dans le panneau **Modifier le volume**, modifiez le type de volume en sélectionnant le nouveau type dans la liste déroulante **Type**.
   
   * Si vous définissez le type sur **Épinglé localement**, StorSimple vérifie si sa capacité est suffisante.
   * Si vous définissez le type sur **Hiérarchisé** et que ce volume sera utilisé pour les données d’archivage, cochez la case **Utiliser ce volume pour des données d’archivage moins fréquemment sollicitées**.
   * Si vous configurez un volume épinglé localement en tant que volume hiérarchisé, ou _vice versa_, le message suivant s’affiche.
   
     ![Message de modification du type de volume](./media/storsimple-8000-manage-volumes-u2/changevoltype3.png)

7. Cliquez sur **Enregistrer** pour enregistrer les modifications. Lorsque vous êtes invité à confirmer la modification, cliquez sur **Oui** pour lancer le processus de conversion. 

    ![Enregistrer et confirmer](./media/storsimple-8000-manage-volumes-u2/modifyvol11.png)

8. Le portail Azure affiche une notification pour la création du travail de mise à jour du volume. Cliquez sur la notification pour surveiller l’état du travail de conversion du volume.

    ![Travail de conversion des volumes](./media/storsimple-8000-manage-volumes-u2/changevoltype5.png)

## <a name="take-a-volume-offline"></a>Mise hors connexion d’un volume

Vous devrez peut-être mettre un volume hors connexion si vous envisagez de le modifier ou de le supprimer. Lorsqu’un volume est hors connexion, il n’est pas disponible pour l’accès en lecture-écriture. Vous devez d’abord mettre le volume hors connexion sur l’hôte et sur l’appareil.

#### <a name="to-take-a-volume-offline"></a>Pour mettre un volume hors connexion

1. Assurez-vous que le volume en question n’est pas utilisé avant de le mettre hors connexion.
2. Mettez d’abord le volume hors connexion sur l’ordinateur hôte. Cela élimine tout risque d’endommagement des données sur le volume. Pour les instructions spécifiques, reportez-vous aux instructions du système d’exploitation de l’ordinateur hôte.
3. Une fois que l’ordinateur hôte est hors connexion, mettez le volume hors connexion sur l’appareil en procédant comme suit :
   
    1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans la liste tabulaire des appareils, sélectionnez l’appareil comportant le volume que vous souhaitez modifier. Cliquez sur **Paramètres > Volumes**.

        ![Accéder au panneau Volumes 3](./media/storsimple-8000-manage-volumes-u2/modifyvol2.png)

    2. Dans la liste tabulaire de volumes, sélectionnez le volume et cliquez avec le bouton droit pour ouvrir le menu contextuel. Sélectionnez **Mettre hors connexion** pour mettre hors connexion le volume que vous allez modifier.

        ![Sélectionner le volume et le mettre hors connexion](./media/storsimple-8000-manage-volumes-u2/modifyvol4.png)

3. Dans le panneau **Mettre hors connexion**, passez en revue les répercussions de la mise hors connexion du volume et activez la case à cocher qui convient. Cliquez sur **Mettre hors connexion**. 

    ![Passer en revue les répercussions de la mise hors connexion du volume 4](./media/storsimple-8000-manage-volumes-u2/modifyvol5.png)
      
      Vous êtes informé de la mise hors connexion du volume. L’état du volume est également mis à jour pour afficher la valeur Hors connexion.
      
4. Lorsqu’un volume est hors connexion, si vous sélectionnez le volume et cliquez sur le bouton droit, l’option **Mettre en ligne** est disponible dans le menu contextuel.

> [!NOTE]
> La commande **Mettre hors connexion** permet de demander à l’appareil de mettre le volume hors connexion. Si les ordinateurs hôtes utilisent toujours le volume, les connexions correspondantes sont rompues et la mise hors connexion du volume n’échoue pas.

## <a name="delete-a-volume"></a>Suppression d’un volume

> [!IMPORTANT]
> Vous pouvez supprimer un volume uniquement s’il est hors connexion.

Pour supprimer un volume, procédez comme indiqué ci-dessous.

#### <a name="to-delete-a-volume"></a>Pour supprimer un volume

1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans la liste tabulaire des appareils, sélectionnez l’appareil comportant le volume que vous souhaitez modifier. Cliquez sur **Paramètres > Volumes**.

    ![Accéder au panneau Volumes 4](./media/storsimple-8000-manage-volumes-u2/modifyvol2.png)

3. Vérifiez l’état du volume à supprimer. Si le volume que vous souhaitez supprimer n’est pas hors connexion, mettez-le hors connexion dans un premier temps. Suivez les étapes de la [Mise hors connexion d’un volume](#take-a-volume-offline).
4. Une fois le volume hors connexion, sélectionnez-le, cliquez avec le bouton droit pour ouvrir le menu contextuel, puis sélectionnez **Supprimer**.

    ![Sélectionner Supprimer dans le menu contextuel](./media/storsimple-8000-manage-volumes-u2/deletevol1.png)

5. Dans le panneau **Supprimer**, passez en revue les répercussions de la suppression d’un volume et cochez la case qui convient. Lorsque vous supprimez un volume, toutes les données qui se trouvent sur ce volume sont perdues. 

    ![Enregistrer et confirmer les modifications](./media/storsimple-8000-manage-volumes-u2/deletevol2.png)

6. Une fois le volume supprimé, la liste tabulaire de volumes est mise à jour et indique la suppression.

    ![Liste des volumes mise à jour](./media/storsimple-8000-manage-volumes-u2/deletevol3.png)
   
   > [!NOTE]
   > Si vous supprimez un volume épinglé localement, l'espace disponible pour les nouveaux volumes ne sera peut-être pas mis à jour immédiatement. Le service StorSimple Device Manager met à jour l’espace local disponible régulièrement. Nous vous recommandons de patienter quelques minutes avant d’essayer de créer le volume.
   >
   > En outre, les tâches de suppression de volume s’exécutent de façon séquentielle si vous supprimez un volume épinglé localement, puis un autre volume du même type immédiatement après. La première tâche de suppression de volume doit se terminer pour que la deuxième puisse commencer.

## <a name="monitor-a-volume"></a>Analyse d’un volume

L’analyse de volume vous permet de collecter des statistiques sur les E/S d’un volume. L’analyse est activée par défaut pour les 32 premiers volumes que vous créez. L’analyse des volumes supplémentaires est désactivée par défaut. 

> [!NOTE]
> L’analyse des volumes clonés est désactivée par défaut.


Suivez la procédure ci-dessous pour activer ou désactiver l’analyse d’un volume.

#### <a name="to-enable-or-disable-volume-monitoring"></a>Pour activer ou désactiver l’analyse de volume

1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**. Dans la liste tabulaire des appareils, sélectionnez l’appareil comportant le volume que vous souhaitez modifier. Cliquez sur **Paramètres > Volumes**.
2. Dans la liste tabulaire de volumes, sélectionnez le volume et cliquez avec le bouton droit pour ouvrir le menu contextuel. Sélectionnez **Modifier**.
3. Dans le panneau **Modifier le volume**, pour **Analyse**, sélectionnez **Activer** ou **Désactiver** afin d’activer ou de désactiver l’analyse.

    ![Désactiver l’analyse](./media/storsimple-8000-manage-volumes-u2/monitorvol1.png) 

4. Cliquez sur **Enregistrer** puis, lorsque vous êtes invité à confirmer l’opération, cliquez sur **Oui**. Le portail Azure affiche une notification de mise à jour du volume, puis un message de réussite, une fois que le volume est à jour.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [cloner un volume StorSimple](storsimple-8000-clone-volume-u2.md).
* Découvrez comment [utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

