---
title: Gestion du calcul sur Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment gérer les paramètres de computing en périphérie (déclencheur, modules, affichage de la configuration du calcul, suppression de configuration) via le portail Azure sur votre ressource Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 05/20/2019
ms.author: alkohli
ms.openlocfilehash: 0d93edbefb61cb8ded09f6402ca1e1661429efe1
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91952146"
---
# <a name="manage-compute-on-your-azure-stack-edge-pro"></a>Gérer le calcul sur votre ressource Azure Stack Edge Pro

Cet article explique comment gérer le calcul sur votre appareil Azure Stack Edge Pro. Vous pouvez gérer le calcul via le portail Azure ou l’interface utilisateur web locale. Le portail Azure vous permet de gérer les modules, les déclencheurs, la configuration du calcul et l’interface utilisateur pour la définition des paramètres de calcul.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Gérer les déclencheurs
> * Gérer la configuration du calcul


## <a name="manage-triggers"></a>Gérer les déclencheurs

Les événements sont des faits qui se produisent au sein de votre environnement cloud ou sur votre appareil, sur lesquels vous pouvez agir. Par exemple, quand un fichier est créé dans un partage, il s’agit d’un événement. Les déclencheurs activent les événements. Dans votre ressource Azure Stack Edge Pro, des déclencheurs peuvent répondre à des événements de fichier ou intervenir en vertu d’une planification.

- **Fichier** : Ces déclencheurs répondent à des événements tels que la création ou la modification d’un fichier.
- **Scheduled** : Ces déclencheurs résultent d’une planification que vous pouvez définir avec une date de début, une heure de début et un intervalle de répétition.


### <a name="add-a-trigger"></a>Ajouter un déclencheur

Pour créer un déclencher, procédez comme suit sur le portail Azure.

1. Sur le portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Computing en périphérie > Déclencheur**. Sélectionnez **+ Ajouter un déclencheur** dans la barre de commandes.

    ![Sélectionner Ajouter un déclencheur](media/azure-stack-edge-manage-compute/add-trigger-1.png)

2. Dans le panneau **Ajouter un déclencheur**, entrez un nom unique pour votre déclencheur.
    
    <!--Trigger names can only contain numbers, lowercase letters, and hyphens. The share name must be between 3 and 63 characters long and begin with a letter or a number. Each hyphen must be preceded and followed by a non-hyphen character.-->

3. Sélectionnez un **Type** pour le déclencheur. Choisissez **Fichier** lorsque le déclencheur répond à un événement de fichier. Sélectionnez **Planifié** si vous souhaitez que le déclencheur démarre à une heure définie et se répète à un intervalle spécifié. L’ensemble d’options présenté varie en fonction de votre sélection.

    - **Déclencheur de fichier** : choisissez un partage monté dans la liste déroulante. Lors du déclenchement d’un événement de fichier dans ce partage, le déclencheur appelle une fonction Azure.

        ![Ajouter un partage SMB](media/azure-stack-edge-manage-compute/add-file-trigger.png)

    - **Déclencheur planifié** : spécifiez les date et heure de démarrage, ainsi que l’intervalle de répétition en heures, minutes ou secondes. Entrez le nom d’une rubrique. Une rubrique vous donne la possibilité de router le déclencheur vers un module déployé sur l’appareil.

        Voici un exemple de chaîne de routage : `"route3": "FROM /* WHERE topic = 'topicname' INTO BrokeredEndpoint("modules/modulename/inputs/input1")"`.

        ![Ajouter un partage NFS](media/azure-stack-edge-manage-compute/add-scheduled-trigger.png)

4. Sélectionnez **Ajouter** pour créer le déclencheur. Une notification indique que la création du déclencheur est en cours. Une fois le déclencheur créé, le panneau s’actualise pour afficher le nouveau déclencheur.
 
    ![Liste des déclencheurs mise à jour](media/azure-stack-edge-manage-compute/add-trigger-2.png)

### <a name="delete-a-trigger"></a>Supprimer un déclencheur

Pour supprimer un déclencheur, procédez comme suit sur le portail Azure.

1. Dans la liste des déclencheurs, sélectionnez le déclencheur à supprimer.

    ![Sélectionner le déclencheur](media/azure-stack-edge-manage-compute/add-trigger-1.png)

2. Cliquez avec le bouton droit, puis sélectionnez **Supprimer**.

    ![Sélectionner Supprimer](media/azure-stack-edge-manage-compute/add-trigger-1.png)

3. Cliquez sur **Oui**lorsque vous êtes invité à confirmer l’opération.

    ![Confirmation de suppression](media/azure-stack-edge-manage-compute/add-trigger-1.png)

La liste des déclencheurs est actualisée afin de refléter la suppression.

## <a name="manage-compute-configuration"></a>Gérer la configuration du calcul

Utilisez le portail Azure pour afficher la configuration de calcul, supprimer une configuration de calcul existante ou actualiser la configuration de calcul, afin de synchroniser les clés d’accès de l’appareil IoT et de l’appareil IoT Edge pour votre ressource Azure Stack Edge Pro.

### <a name="view-compute-configuration"></a>Afficher une configuration de calcul

Pour afficher la configuration de calcul de votre appareil, procédez comme suit sur le portail Azure.

1. Sur le portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Computing en périphérie > Modules**. Sélectionnez **Voir le computing** dans la barre de commandes.

    ![Sélectionner Voir le computing](media/azure-stack-edge-manage-compute/view-compute-1.png)

2. Prenez note de la configuration de calcul sur votre appareil. Lorsque vous avez configuré le calcul, vous avez créé une ressource IoT Hub. Sous cette ressource IoT Hub, un appareil IoT et un appareil IoT Edge sont configurés. Seuls les modules Linux sont pris en charge pour s’exécuter sur l’appareil IoT Edge.

    ![Afficher la configuration](media/azure-stack-edge-manage-compute/view-compute-2.png)


### <a name="remove-compute-configuration"></a>Supprimer la configuration du calcul

Pour supprimer la configuration de computing en périphérie de votre appareil, procédez comme suit sur le portail Azure.

1. Sur le portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Computing en périphérie > Prise en main**. Sélectionnez **Supprimer le computing** dans la barre de commandes.

    ![Sélectionner Supprimer le computing](media/azure-stack-edge-manage-compute/remove-compute-1.png)

2. Si vous supprimez la configuration de calcul, vous devrez reconfigurer votre appareil si vous avez besoin d’utiliser le calcul à nouveau. Quand vous êtes invité à confirmer l’opération, sélectionnez **Oui**.

    ![Sélectionner Supprimer le computing 2](media/azure-stack-edge-manage-compute/remove-compute-2.png)

### <a name="sync-up-iot-device-and-iot-edge-device-access-keys"></a>Synchroniser les clés d’accès de l’appareil IoT et de l’appareil IoT Edge

Lorsque vous configurez le calcul sur votre ressource Azure Stack Edge Pro, un appareil IoT et un appareil IoT Edge sont créés. Des clés d’accès symétriques sont affectées automatiquement à ces appareils. Conformément aux meilleures pratiques de sécurité, le service IoT Hub veille à une rotation régulière de ces clés.

Pour définir la rotation de ces clés, vous pouvez accéder au service IoT Hub que vous avez créé, puis sélectionnez l’appareil IoT ou l’appareil IoT Edge. Chaque appareil possède une clé d’accès primaire et une clé d’accès secondaire. Affectez la clé d’accès primaire à la clé d’accès secondaire, puis régénérez la clé d’accès primaire.

Si une rotation des clés de vos appareils IoT et IoT Edge a eu lieu, vous devez actualiser la configuration sur votre ressource Azure Stack Edge Pro pour obtenir les dernières clés d’accès. La synchronisation aide l’appareil à obtenir les dernières clés de vos appareils IoT et IoT Edge. Azure Stack Edge Pro utilise uniquement les clés d’accès primaires.

Pour synchroniser les clés d’accès pour votre appareil, procédez comme suit sur le portail Azure.

1. Sur le portail Azure, accédez à votre ressource Azure Stack Edge, puis à **Computing en périphérie > Prise en main**. Sélectionnez **Actualiser la configuration** dans la barre de commandes.

    ![Sélectionner Actualiser la configuration](media/azure-stack-edge-manage-compute/refresh-configuration-1.png)

2. Lorsque vous êtes invité à confirmer l’opération, sélectionnez **Oui**.

     ![Sélectionner Oui à l’invite](media/azure-stack-edge-manage-compute/refresh-configuration-2.png)

3. Fermez la boîte de dialogue une fois la synchronisation terminée.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Gérer le réseau du computing en périphérie via le portail Azure](azure-stack-edge-extend-compute-access-modules.md).
