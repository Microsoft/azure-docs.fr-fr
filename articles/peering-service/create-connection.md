---
title: 'Azure Peering Service : Créer '
description: Dans ce tutoriel, découvrez comment inscrire Azure Peering Service et un préfixe.
services: peering-service
author: derekolo
ms.service: peering-service
ms.topic: tutorial
ms.date: 05/2/2020
ms.author: derekol
Customer intent: With Azure Peering service enhancing the customer connectivity to Microsoft cloud services .
ms.openlocfilehash: f8f0ee71a8dd00498d868bb6dc11b3fa083df766
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021093"
---
# <a name="tutorial-create-a-peering-service-connection"></a>Tutoriel : Créer une connexion Peering Service

Ce tutoriel vous montre comment créer une ressource Peering Service et comment configurer une connexion Peering Service. 

1. Pour inscrire une connexion Peering Service, sélectionnez  **Créer une ressource** > **Peering Service**.

 
    ![Inscrire Peering Service](./media/peering-service-portal/peering-servicecreate.png)

2. Entrez les informations suivantes sous l’onglet **Basics** (Général) dans la page **Create a peering service connection** (Créer une connexion Peering Service).
 
3. Sélectionnez l’abonnement et le groupe de ressources associé à l’abonnement.

    ![Inscrire Peering Service, onglet Basics](./media/peering-service-portal/peering-servicebasics.png)

4. Entrez le **nom** sous lequel vous voulez inscrire l’instance Peering Service.

5. Sélectionnez le bouton **Next:Configuration** (Suivant : Configuration) en bas de la page. La page **Configuration** s’affiche.
## <a name="configure-the-peering-service-connection"></a>Configurer la connexion Peering Service

1. Dans la page **Configuration**, sélectionnez l’emplacement où Peering Service doit être activé, en sélectionnant l’option dans la liste déroulante **Peering service location** (Emplacement de Peering Service).

1. Sélectionnez le fournisseur de services auprès duquel obtenir Peering Service, en sélectionnant un nom de fournisseur dans la liste déroulante **Peering service provider** (Fournisseur Peering Service).
 
1. Sélectionnez **Create new prefix** (Créer un préfixe) en bas de la section **Prefixes** (Préfixes). Plusieurs zones de texte s’affichent. À présent, entrez le nom de la ressource de préfixe ainsi que les préfixes associés au fournisseur de services.

1. Sélectionnez **Prefix Key** (Clé de préfixe) et ajoutez la clé de préfixe attribuée par votre fournisseur (ISP ou IXP). Cette clé permet à Microsoft de valider le préfixe et le fournisseur qui a alloué votre préfixe IP.

    ![La capture d’écran montre l’onglet Configuration de la page Créer une connexion au service de peering, où vous pouvez entrer la clé de préfixe.](./media/peering-service-portal/peering-serviceconfiguration.png)

1. Sélectionnez le bouton **Vérifier + créer** en bas à gauche de la page. La page **Vérifier + créer** apparaît, et Azure valide votre configuration.

 1. Quand vous voyez le message **Validation réussie**, sélectionnez **Créer**.

> ![La capture d’écran montre l’onglet Vérifier + créer de la page Créer une connexion au service de peering.](./media/peering-service-portal/peering-service-prefix.png)

1. Une fois que vous avez inscrit une connexion Peering Service, une validation supplémentaire est effectuée sur les préfixes inclus. Vous pouvez vérifier l’état de la validation sous la section **Préfixes** du nom de la ressource. Si la validation échoue, l’un des messages d’erreur suivants s’affiche :

   - Préfixe Peering Service non valide ; le préfixe doit être dans un format valide, seul le préfixe Ipv4 est pris en charge.
   - Le préfixe n’a pas été reçu du fournisseur Peering Service.
   - L’annonce du préfixe n’a pas de communauté BGP valide. Contactez le fournisseur Peering Service.
   - Route de sauvegarde introuvable. Contactez le fournisseur Peering Service.
   - Le préfixe reçu a un chemin AS plus long. Contactez le fournisseur Peering Service.
   - Le préfixe reçu a un AS privé dans le chemin. Contactez le fournisseur Peering Service.

### <a name="add-or-remove-a-prefix"></a>Ajouter ou supprimer un préfixe

Sélectionnez **Ajouter des préfixes** dans la page **Préfixes** pour ajouter des préfixes.

Sélectionnez les points de suspension (...) à côté du préfixe indiqué, puis sélectionnez l’option **Supprimer**.

### <a name="delete-a-peering-service-connection"></a>Supprimer une connexion Peering Service

Dans la page **Toutes les ressources**, cochez la case de la connexion Peering Service et sélectionnez l’option **Supprimer** en haut de la page.
## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la connexion Peering Service, consultez [Connexion Peering Service](connection.md).
- Pour en savoir plus sur la télémétrie des connexions Peering Service, consultez [Télémétrie des connexions Peering Service](connection-telemetry.md).
- Pour mesurer les données de télémétrie, consultez [Mesurer les données de télémétrie des connexions](measure-connection-telemetry.md).
- Pour inscrire la connexion par le biais d’Azure PowerShell, consultez [Inscrire une connexion Peering Service - Azure PowerShell](powershell.md).
- Pour inscrire la connexion à l’aide d’Azure CLI, consultez [Inscrire une connexion Peering Service - Azure CLI](cli.md).