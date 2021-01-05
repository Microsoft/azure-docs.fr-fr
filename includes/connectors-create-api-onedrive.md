---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 07/21/2020
ms.openlocfilehash: 27df83552b450b7657c0595d1d419001df98148c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87040227"
---
## <a name="prerequisites"></a>Prérequis

* Un compte [OneDrive](https://www.microsoft.com/store/apps/onedrive/9wzdncrfj1p3) 

Pour pouvoir utiliser votre compte OneDrive avec Logic Apps, vous devez autoriser Logic Apps à se connecter à votre compte OneDrive dans le portail Azure.

Pour autoriser Logic Apps à se connecter à votre compte OneDrive, procédez comme suit :  

1. Connectez-vous au portail Azure. 

1. Sous **Services Azure**, sélectionnez **Logic Apps**. Ensuite, sélectionnez le nom de votre application logique dans la liste.

1. Dans le menu de l’application logique, sous **Outils de développement**, choisissez **Concepteur d’application logique**.

1. Dans le concepteur Logic Apps, sélectionnez **Afficher les API gérées par Microsoft** dans la liste déroulante, puis entrez *OneDrive* dans la zone de recherche. Sélectionnez le déclencheur ou l’action à utiliser :

   ![Capture d’écran du concepteur Logic Apps, montrant la liste des actions de l’API OneDrive à ajouter.](./media/connectors-create-api-onedrive/onedrive-1.png)

2. Si vous n’avez créé aucune connexion à OneDrive auparavant, connectez-vous à l’aide de vos informations d’identification OneDrive lorsque vous y êtes invité :  

   ![Capture d’écran du concepteur Logic Apps, montrant l’invite de connexion pour l’API OneDrive.](./media/connectors-create-api-onedrive/onedrive-2.png)

3. Sélectionnez **Connexion**, puis indiquez votre nom d’utilisateur et votre mot de passe. Sélectionnez **Connexion** : 

   ![Capture d’écran de la page de connexion d’un compte Microsoft, pour l’autorisation de l’API OneDrive.](./media/connectors-create-api-onedrive/onedrive-3.png)   

    Ces informations d’identification serviront à autoriser votre application logique à accéder aux données de votre compte OneDrive. 

4. Sélectionnez **Oui** pour autoriser l’application logique à utiliser votre compte OneDrive :  

   ![Capture d’écran de l’autorisation du compte Microsoft pour Logic Apps, avec les actions autorisées.](./media/connectors-create-api-onedrive/onedrive-4.png)   
   
5. La connexion est maintenant répertoriée dans l’étape. Sélectionnez Enregistrer, puis continuez à créer votre application logique. 

   ![Capture d’écran du concepteur Logic Apps, montrant l’éditeur d’action avec l’API OneDrive connectée.](./media/connectors-create-api-onedrive/onedrive-5.png)
