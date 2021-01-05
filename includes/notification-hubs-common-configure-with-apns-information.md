---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 537c86512db4706077d0089a4f71ee945623c26b
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095303"
---
### <a name="configure-your-notification-hub-with-apns-information"></a>Configurer votre hub de notification avec des informations APNS

Sous **Services de notification**, sélectionnez **Apple**, puis suivez les étapes appropriées en fonction de l’approche que vous avez choisie dans la section [Création d’un certificat pour Notification Hubs](#creating-a-certificate-for-notification-hubs).  

> [!NOTE]
> Utilisez uniquement le mode **Production** ou **Application** si vous souhaitez envoyer des notifications Push aux utilisateurs ayant acheté votre application dans le Store.

### <a name="option-1-using-a-p12-push-certificate"></a>OPTION 1 : Utilisation d’un certificat Push .p12

1. Sélectionnez **Certificate**.

1. Sélectionnez l’icône du fichier.

1. Sélectionnez le fichier .p12 exporté précédemment, puis sélectionnez **Ouvrir**.

1. S’il y a lieu, entrez le mot de passe correct.

1. Sélectionnez le mode **Bac à sable**.

1. Sélectionnez **Enregistrer**.

### <a name="option-2-using-token-based-authentication"></a>OPTION 2 : Utilisation de l’authentification basée sur un jeton

1. Sélectionnez **Jeton**.
1. Entrez les valeurs suivantes déjà en votre possession :

    - **ID de clé**
    - **ID de bundle**
    - **ID de l’équipe**
    - **Jeton**

1. Choisissez **Bac à sable**.
1. Sélectionnez **Enregistrer**.
