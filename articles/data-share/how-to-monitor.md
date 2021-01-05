---
title: Comment surveiller Azure Data Share
description: Découvrez comment surveiller l’état d’une invitation, l’abonnement de partage et l’historique des instantanés dans Azure Data Share
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 07/30/2020
ms.openlocfilehash: cc400922115f348090677661cd1b30434be19b72
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92910472"
---
# <a name="monitor-azure-data-share"></a>Superviser Azure Data Share  

Cet article explique comment surveiller vos partages de données à l’aide d’Azure Data Share. En tant que fournisseur de données, vous êtes en mesure de surveiller divers aspects de vos relations de partage de données. Vous pouvez surveiller divers détails, par exemple pour savoir si vos consommateurs de données ont accepté votre invitation de partage de données, ainsi que s’ils ont créé un abonnement de partage et commencé à utiliser vos données. 

En tant que consommateur de données, vous pouvez surveiller les captures instantanées qui ont été déclenchées dans votre abonnement Azure. 

## <a name="monitor-invitation-status"></a>Surveiller l’état d’invitation

Affichez l’état de vos invitations de partage de données en accédant à Partages envoyés -> Invitations. 

![Statut de l’invitation](./media/invitation-status.png "Statut de l’invitation") 

Il existe trois états que votre invitation peut avoir :

* En attente - Le destinataire du partage de données n'a pas encore accepté l’invitation.
* Accepté - Le destinataire du partage de données a accepté l’invitation.
* Rejeté - Le destinataire du partage de données a rejeté l’invitation.

> [!IMPORTANT]
> Si vous supprimez une invitation après qu’elle a déjà été acceptée, cela ne revient pas à révoquer l’accès. Si vous souhaitez empêcher la copie des instantanés futurs dans votre compte de stockage de consommateurs de données, vous devez révoquer l’accès via l’onglet *Abonnements de partage*. 

## <a name="monitor-share-subscriptions"></a>Surveiller les abonnements de partage

Affichez l’état de vos abonnements de partage en accédant à Partages envoyés -> Abonnements de partage. Cela vous donnera plus d’informations sur les abonnements actifs créés par les consommateurs de données après avoir accepté votre invitation. Vous pouvez empêcher les futures mises à jour à votre consommateur de données en sélectionnant l’abonnement de partage, puis *Révoquer*. 

## <a name="snapshot-history"></a>Historique des instantanés 

Dans l’onglet **Historique** d’un partage, vous pouvez afficher le moment où les données sont copiées du fournisseur de données vers le magasin de données du consommateur de données. Vous êtes en mesure de surveiller la fréquence, la durée et l’état de chaque instantané. 

![Capture d’écran de Partages envoyés sur le Portail Azure](./media/sent-shares.png "Historique des instantanés") 

Vous pouvez afficher plus de détails sur chaque exécution d’instantané en cliquant sur la date de début de l’exécution. Cliquez ensuite sur l’état de chaque jeu de données pour afficher la quantité de données transférées, le nombre de fichiers/enregistrements copiés, la durée de l’instantané, le nombre de vCores utilisés et le message d’erreur, le cas échéant. 

L'historique des instantanés est de 30 jours. Si vous avez besoin d’enregistrer et de consulter plus de 30 jours d’historique, vous pouvez utiliser le paramètre de diagnostic.

## <a name="diagnostic-setting"></a>Paramètre de diagnostic

Vous pouvez configurer le paramètre de diagnostic pour enregistrer les données du journal ou les événements. Accédez à Supervision -> Paramètres de diagnostic et sélectionnez **Ajouter un paramètre de diagnostic**. Sélectionnez les données du journal ou les événements qui vous intéressent, ainsi que l’emplacement où vous souhaitez les stocker ou les envoyer. 

![Capture d’écran de la page Paramètres de diagnostic sur le Portail Azure](./media/diagnostic-settings.png "Paramètres de diagnostic") 

## <a name="next-steps"></a>Étapes suivantes 

En savoir plus sur la [terminologie propre à Azure Data Share](terminology.md)