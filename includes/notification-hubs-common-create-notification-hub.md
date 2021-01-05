---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 7c3887367b6365ea3577bbff6cc19bf34d178ee6
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095309"
---
### <a name="create-a-notification-hub"></a>Création d’un hub de notifications 

Dans cette section, vous allez créer un hub de notifications et configurer l’authentification avec **APNS**. Vous pouvez utiliser un certificat Push p12 ou l’authentification basée sur un jeton. Si vous souhaitez utiliser un hub de notification que vous avez déjà créé, vous pouvez passer directement à l’étape 5.

1. Connectez-vous à [Azure](https://portal.azure.com).

1. Cliquez sur **Créer une ressource**, recherchez et choisissez **Hub de notification**, puis cliquez sur **Créer**.

1. Mettez à jour les champs suivants, puis cliquez sur **Créer** :

    **DÉTAILS DE BASE**  

    **Abonnement :** choisissez l’**abonnement** cible dans la liste déroulante.  
    **Groupe de ressources :** créez un **groupe de ressources** ou (sélectionnez un groupe existant).  

    **DÉTAILS DE L’ESPACE DE NOMS**  

    **Espace de noms du hub de notification** : entrez un nom global unique pour l’espace de noms du **hub de notification**.  

    > [!NOTE]
    > Vérifiez que l’option **Créer** est sélectionnée pour ce champ.

    **DÉTAILS DU HUB DE NOTIFICATIONS**  

    **Hub de notification :** entrez un nom pour le **hub de notifications**.  
    **Emplacement :** choisissez un emplacement approprié dans la liste déroulante.  
    **Niveau tarifaire :** conservez l’option par défaut **Gratuit**.  

    > [!NOTE]
    > Sauf si vous avez atteint le nombre maximal de hubs au niveau gratuit.

1. Une fois que le **hub de notifications** a été provisionné, accédez à cette ressource.
1. Accédez à votre nouveau **hub de notifications**.
1. Sélectionnez **Stratégies d’accès** dans la liste (sous **GÉRER**).
1. Notez les valeurs de **Nom de la stratégie** ainsi que les valeurs de **Chaîne de connexion** correspondantes.
