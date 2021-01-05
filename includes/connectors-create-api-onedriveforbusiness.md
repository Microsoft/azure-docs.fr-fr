---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 07/21/2020
ms.openlocfilehash: 06a503e800309d8689735417931820d9f16a6e75
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87102641"
---
## <a name="prerequisites"></a>Prérequis

* Un compte [OneDrive Entreprise](https://OneDrive.com) 

Pour pouvoir utiliser votre compte OneDrive Entreprise avec Logic Apps, vous devez autoriser Logic Apps à se connecter à votre compte OneDrive Entreprise dans le portail Azure.

Pour autoriser Logic Apps à se connecter à votre compte OneDrive Entreprise, effectuez les étapes suivantes :  

1. Connectez-vous au portail Azure. 

1. Sous **Services Azure**, sélectionnez **Logic Apps**. Ensuite, sélectionnez le nom de votre application logique dans la liste.

1. Dans le menu de l’application logique, sous **Outils de développement**, choisissez **Concepteur d’application logique**.

1. Dans le concepteur Logic Apps, sélectionnez **Afficher les API gérées par Microsoft** dans la liste déroulante, puis entrez *OneDrive* dans la zone de recherche. Sélectionnez le déclencheur ou l’action à utiliser :  

   ![Capture d’écran du concepteur Logic Apps montrant le déclencheur de récurrence avec les actions de l’API OneDrive Entreprise.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-1.png)

2. Si vous n’avez créé aucune connexion à OneDrive Entreprise auparavant, suivez les invites pour indiquer vos informations d’identification OneDrive Entreprise. Ces informations d’identification servent à autoriser votre application logique à accéder aux données de votre compte OneDrive Entreprise :  

   ![Capture d’écran du concepteur Logic Apps, montrant l’invite de connexion pour OneDrive Entreprise.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-2.png)

3. Indiquez votre nom d’utilisateur et votre mot de passe OneDrive Entreprise pour autoriser votre application logique :  

   ![Capture d’écran de la page de connexion OneDrive Entreprise montrant l’invite de connexion.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-3.png)   

4. La connexion est maintenant listée dans l’étape. Sélectionnez Enregistrer, puis continuez à créer votre application logique. 

   ![Capture d’écran du concepteur Logic Apps montrant le déclencheur avec la connexion OneDrive Entreprise.](./media/connectors-create-api-onedriveforbusiness/onedriveforbusiness-4.png)   
