---
title: Fichier Include
description: Fichier Include
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 02/20/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: ace42278269ff6af31902dbecead81329815af12
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177299"
---
## <a name="create-a-topic-using-the-azure-portal"></a>Créer une rubrique à l’aide du Portail Azure
1. Dans la page **Espace de noms Service Bus**, sélectionnez **Rubriques** dans le menu de gauche.
2. Sélectionnez **+ Rubrique** dans la barre d’outils. 
4. Entrez un **nom** pour la rubrique. Conservez les valeurs par défaut des autres options.
5. Sélectionnez **Create** (Créer).

    ![Créer une rubrique](./media/service-bus-create-topics-subscriptions-portal/create-topic.png)

## <a name="create-subscriptions-to-the-topic"></a>Créer des abonnements à la rubrique
1. Sélectionnez la **rubrique** que vous avez créée dans la section précédente. 
    
    ![Sélectionner la rubrique](./media/service-bus-create-topics-subscriptions-portal/select-topic.png)
2. Dans la page **Rubrique Service Bus**, sélectionnez **Abonnements** dans le menu de gauche, puis **+ Abonnement** dans la barre d’outils. 
    
    ![Bouton Ajouter un abonnement](./media/service-bus-create-topics-subscriptions-portal/add-subscription-button.png)
3. Dans la page **Créer un abonnement**, entrez **S1** comme **nom** d’abonnement, puis sélectionnez **Créer**. 

    ![Page Créer un abonnement](./media/service-bus-create-topics-subscriptions-portal/create-subscription-page.png)
4. Répétez la dernière étape deux autre fois pour créer des abonnements nommés **S2** et **S3**.