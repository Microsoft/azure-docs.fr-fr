---
title: Inscrire Azure Peering Service - Portail Azure
description: Découvrez comment inscrire Azure Peering Service à l’aide du portail Azure.
services: peering-service
author: derekolo
ms.service: peering-service
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: Infrastructure-services
ms.date: 05/18/2020
ms.author: derekol
ms.openlocfilehash: 62210105ba4ba84cff4b246c5b7cf8cc794fb5e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91534945"
---
# <a name="register-peering-service-by-using-the-azure-portal"></a>Inscrire Peering Service à l’aide du portail Azure

Azure Peering Service est un service réseau qui améliore la connectivité entre le client et les services cloud Microsoft, comme Microsoft 365, Dynamics 365, les services SaaS, Azure ou les services Microsoft accessibles via l’Internet public.

Cet article explique comment inscrire une connexion Peering Service à l’aide du portail Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) maintenant.

> 

## <a name="prerequisites"></a>Prérequis

Vous devez disposer des éléments suivants :

### <a name="azure-account"></a>Compte Azure

Vous devez avoir un compte Microsoft Azure actif et valide. Sinon, vous ne pourrez pas configurer la connexion Peering Service. Peering Service est une ressource fournie dans les abonnements Azure. 

### <a name="connectivity-provider"></a>Fournisseur de connectivité

Vous pouvez faire appel à un fournisseur de services Internet ou un partenaire d’échange Internet pour obtenir une connectivité Peering Service qui vous permettra de connecter votre réseau au réseau Microsoft.

Vérifiez que les [fournisseurs de connectivité](location-partners.md) sont des partenaires de Microsoft.



## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Dans un navigateur, accédez au portail Azure et connectez-vous avec votre compte Azure.

## <a name="register-a-peering-service-connection"></a>Inscrire une connexion Peering Service

1. Pour inscrire une connexion Peering Service, sélectionnez  **Créer une ressource** > **Peering Service**.

    ![Inscrire Peering Service](./media/peering-service-portal/peering-servicecreate.png)
1. Entrez les informations suivantes sous l’onglet **Basics** (Général) dans la page **Create a peering service connection** (Créer une connexion Peering Service).

 
1. Sélectionnez l’abonnement et le groupe de ressources associé à l’abonnement.

   ![Inscrire Peering Service, onglet Basics](./media/peering-service-portal/peering-servicebasics.png)

1. Entrez le **nom** sous lequel vous voulez inscrire l’instance Peering Service.
 
1. Sélectionnez le bouton **Next:Configuration** (Suivant : Configuration) en bas de la page. La page **Configuration** s’affiche.

## <a name="configure-the-peering-service-connection"></a>Configurer la connexion Peering Service

1. Dans la page **Configuration**, sélectionnez l’emplacement où Peering Service doit être activé, en sélectionnant l’option dans la liste déroulante **Peering service location** (Emplacement de Peering Service).

1. Sélectionnez le fournisseur de services auprès duquel obtenir Peering Service, en sélectionnant un nom de fournisseur dans la liste déroulante **Peering service provider** (Fournisseur Peering Service).
 
1. Sélectionnez **Create new prefix** (Créer un préfixe) en bas de la section **Prefixes** (Préfixes). Plusieurs zones de texte s’affichent. À présent, entrez le nom de la ressource de préfixe ainsi que les préfixes associés au fournisseur de services.

1. Sélectionnez **Prefix Key** (Clé de préfixe) et ajoutez la clé de préfixe attribuée par votre fournisseur (ISP ou IXP). Cette clé permet à Microsoft de valider le préfixe et le fournisseur qui a alloué votre préfixe IP.
   > ![La capture d’écran montre l’onglet Configuration de la page Créer une connexion au service de peering, où vous pouvez entrer la clé de préfixe.](./media/peering-service-portal/peering-serviceconfiguration.png)

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

> [!NOTE]
> Vous ne pouvez pas modifier un préfixe existant.
>

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la connexion Peering Service, consultez [Connexion Peering Service](connection.md).
- Pour en savoir plus sur la télémétrie des connexions Peering Service, consultez [Télémétrie des connexions Peering Service](connection-telemetry.md).
- Pour mesurer les données de télémétrie, consultez [Mesurer les données de télémétrie des connexions](measure-connection-telemetry.md).
- Pour inscrire la connexion par le biais d’Azure PowerShell, consultez [Inscrire une connexion Peering Service - Azure PowerShell](powershell.md).
- Pour inscrire la connexion à l’aide d’Azure CLI, consultez [Inscrire une connexion Peering Service - Azure CLI](cli.md).
