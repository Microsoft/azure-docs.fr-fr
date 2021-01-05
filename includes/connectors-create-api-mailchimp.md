---
ms.service: logic-apps
ms.topic: include
author: ecfan
ms.author: estfan
ms.date: 11/03/2016
ms.openlocfilehash: a7290e46a0c5e228689dd1f1a0dc7f9a1182ad60
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2020
ms.locfileid: "74789655"
---
## <a name="prerequisites"></a>Prérequis

* Un compte [MailChimp](https://www.MailChimp.com/) 

Pour pouvoir utiliser votre compte MailChimp dans une application logique, vous devez autoriser l’application à se connecter à votre compte. Heureusement, cette opération est très simple à effectuer dans votre application logique sur le portail Azure. 

Pour autoriser votre application logique à se connecter à votre compte MailChimp, procédez comme suit :

1. Pour créer une connexion à MailChimp, dans le Concepteur d’applications logiques, sélectionnez **Afficher les API gérées par Microsoft** dans la liste déroulante, puis entrez *MailChimp* dans la zone de recherche. Sélectionnez le déclencheur ou l’action que vous allez utiliser :  
   ![MailChimp étape 1](./media/connectors-create-api-mailchimp/mailchimp-1.png)
2. Si vous n’avez créé aucune connexion à MailChimp auparavant, vous êtes invité à indiquer vos informations d’identification MailChimp. Ces informations d’identification serviront à autoriser votre application logique à se connecter et à accéder aux données de votre compte MailChimp :  
   ![MailChimp étape 2](./media/connectors-create-api-mailchimp/mailchimp-2.png)
3. Indiquez votre nom d’utilisateur et votre mot de passe MailChimp pour autoriser votre application logique :  
   ![MailChimp étape 3](./media/connectors-create-api-mailchimp/mailchimp-3.png)   
4. Notez que la connexion a été créée et que vous pouvez maintenant poursuivre la procédure dans votre application logique :  
   ![MailChimp étape 4](./media/connectors-create-api-mailchimp/mailchimp-4.png)

