---
title: Fichier Include
description: Fichier Include
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 10/16/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: bf107da82fb3f772a341e70ce472f08ea674a450
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75692570"
---
### <a name="create-a-storage-account-for-event-processor-host"></a>Créer un compte de stockage pour Event Processor Host
Event Processor Host est un agent intelligent qui simplifie la réception d’événements provenant d’Event Hubs en gérant les points de contrôle persistants et les destinataires parallèles. Pour les points de contrôle, Event Processor Host nécessite un compte de stockage. L’exemple suivant montre comment créer un compte de stockage et comment obtenir ses clés pour l’accès :

1. Dans le menu du Portail Azure, sélectionnez **Créer une ressource**.

    ![Élément de menu Créer une ressource, portail Microsoft Azure](./media/event-hubs-create-storage/create-resource.png)

2. Sélectionnez **Stockage** > **Compte de stockage**.
   
    ![Sélectionner Compte de stockage, portail Microsoft Azure](./media/event-hubs-create-storage/select-storage-account.png)

3. Sur la page **Créer un compte de stockage**, procédez comme suit : 

   1. Entrez le **nom du compte de stockage**.
   2. Choisissez un **abonnement** Azure qui contient le hub d’événements.
   3. Choisissez ou créez le **groupe de ressources** qui contient le hub d’événements.
   4. Choisissez l’**emplacement** dans lequel stocker la ressource. 
   5. Sélectionnez **Revoir + créer**.
   
        ![Vérifier + créer, Créer un compte de stockage, portail Microsoft Azure](./media/event-hubs-create-storage/review-create.png)

4. Sur la page **Vérifier + créer**, vérifiez les valeurs, puis sélectionnez **Créer**. 

    ![Vérifier les paramètres de compte de stockage et créer, portail Microsoft Azure](./media/event-hubs-create-storage/create-storage-account.png)
5. Une fois le message **Déploiements réussis** affiché dans vos notifications, sélectionnez **Accéder à la ressource** pour ouvrir la page Compte de stockage. Vous pouvez aussi développer **Détails du déploiement** et sélectionner ensuite votre nouvelle ressource dans la liste des ressources.  

    ![Accéder à la ressource, déploiement de compte de stockage, portail Microsoft Azure](./media/event-hubs-create-storage/go-to-resource.png) 
6. Sélectionnez **Conteneurs**.

    ![Sélectionner le service de conteneur Objets Blob, comptes de stockage, portail Microsoft Azure](./media/event-hubs-create-storage/select-blob-container-service.png)
7. Sélectionnez **+ Conteneur** en haut, entrez le **nom** du conteneur, puis sélectionnez **OK**. 

    ![Créer un conteneur d’objets blob, comptes de stockage, portail Microsoft Azure](./media/event-hubs-create-storage/create-new-blob-container.png)
8. Choisissez **Clés d’accès** dans le menu de la page **Compte de stockage**, puis copiez la valeur de **key1**.

    Enregistrez les valeurs suivantes dans le Bloc-notes ou un autre emplacement temporaire.
    - Nom du compte de stockage
    - Clé d’accès au compte de stockage
    - Nom du conteneur
