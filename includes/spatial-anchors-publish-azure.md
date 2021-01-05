---
author: msftradford
ms.service: spatial-anchors
ms.topic: include
ms.date: 11/20/2020
ms.author: parkerra
ms.openlocfilehash: 80685dee7907b81832c94044d1feb8fcf2e41bde
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185377"
---
### <a name="open-the-publish-wizard"></a>Ouvrir l’Assistant de publication

Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **SharingService**, puis sélectionnez **Publier**.

L’Assistant de publication démarre. 

Sélectionnez **App Service** > **Publier** pour ouvrir le volet **Créer App Service**.

### <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure.

Dans le volet **Créer App Service**, sélectionnez **Ajouter un compte**, puis connectez-vous à votre abonnement Azure. Si vous êtes déjà connecté, sélectionnez le compte que vous souhaitez dans la liste déroulante.

   > [!NOTE]
   > Si vous êtes déjà connecté, ne sélectionnez pas encore **Créer**.
   >

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

[!INCLUDE [resource group intro text](resource-group.md)]

En regard de **Groupe de ressources**, sélectionnez **Nouveau**.

Nommez le groupe de ressources **myResourceGroup**, puis sélectionnez **OK**.

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

[!INCLUDE [app-service-plan](app-service-plan.md)]

En regard de **Plan d’hébergement**, sélectionnez **Nouveau**.

Dans le volet **Configurer un plan d’hébergement**, utilisez ces paramètres :

| Paramètre | Valeur suggérée | Description |
|-|-|-|
|Plan App Service| MySharingServicePlan | Nom du plan App Service |
| Emplacement | USA Ouest | Centre de données où l’application web est hébergée |
| Taille | Gratuit | [Niveau tarifaire](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) déterminant les fonctionnalités d’hébergement |

Sélectionnez **OK**.

### <a name="create-and-publish-the-web-app"></a>Créer et publier l’application web

Dans **Nom de l’application**, entrez un nom d’application unique. Les caractères valides sont a-z, 0-9 et les tirets (-). Sinon, acceptez le nom unique généré automatiquement. L’URL de l’application web est `https://<app_name>.azurewebsites.net`, où `<app_name>` est le nom de votre application.

Sélectionnez **Créer** pour commencer à créer les ressources Azure.

   À la fin de l’Assistant, il publie l’application web ASP.NET Core sur Azure, puis il ouvre l’application dans votre navigateur par défaut.

  ![Capture d’écran d’une application web ASP.NET publiée dans Azure.](./media/spatial-anchors-azure/web-app-running-live.png)

Le nom d’application que vous avez utilisé dans cette section est utilisé en tant que le préfixe d’URL au format `https://<app_name>.azurewebsites.net`. Copiez cette URL dans un éditeur de texte en vue de l’utiliser ultérieurement.
