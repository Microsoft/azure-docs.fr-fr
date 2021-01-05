---
title: Partager des données en dehors de votre organisation (portail Azure) - Guide de démarrage rapide Azure Data Share
description: Découvrez comment partager des données avec des clients et des partenaires via Azure Data Share dans ce guide de démarrage rapide.
author: joannapea
ms.author: joanpo
ms.service: data-share
ms.topic: quickstart
ms.date: 10/30/2020
ms.openlocfilehash: 1442720fdf48aaa7da76e181b168a04306ff3e33
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186454"
---
# <a name="quickstart-share-data-using-azure-data-share-in-the-azure-portal"></a>Démarrage rapide : Partager des données avec Azure Data Share dans le portail Azure

Dans ce guide de démarrage rapide, vous allez voir comment configurer une nouvelle instance d’Azure Data Share à l’aide du portail Azure afin de partager les données d’un compte de stockage.

## <a name="prerequisites"></a>Conditions préalables

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* L’adresse e-mail de connexion Azure de vos destinataires (l’utilisation de leur alias de courrier ne fonctionnera pas).
* Si le magasin de données Azure source se trouve dans un autre abonnement Azure que celui que vous utiliserez pour créer une ressource de partage de données, inscrivez le [fournisseur de ressources Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) dans l’abonnement où se trouve le magasin de données Azure. 

### <a name="share-from-a-storage-account"></a>Partager à partir d’un compte de stockage

* Compte Stockage Azure : Si vous n’en avez pas déjà, vous pouvez créer un [compte Stockage Azure](../storage/common/storage-account-create.md)
* Autorisation d’écrire dans le compte de stockage, qui est présent dans *Microsoft.Storage/storageAccounts/write*. Cette autorisation existe dans le rôle **Contributeur**.
* Autorisation d’ajouter l’attribution de rôle au compte de stockage, qui est présente dans *Microsoft.Authorization/role assignments/write*. Cette autorisation existe dans le rôle **Propriétaire**. 

## <a name="create-a-data-share-account"></a>Créer un compte Data Share

Créez une ressource Azure Data Share dans un groupe de ressources Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Cliquez sur le bouton **Créer une ressource** (+) dans le coin supérieur gauche du portail.

1. Recherchez *Data Share*.

1. Sélectionnez **Data Share**, puis sélectionnez **Créer**.

1. Indiquez les détails de base de votre ressource Azure Data Share à partir des informations suivantes. 

   **Paramètre** | **Valeur suggérée** | **Description du champ**
   |---|---|---|
   | Abonnement | Votre abonnement | Sélectionnez l’abonnement Azure à utiliser pour votre compte de partage de données.|
   | Groupe de ressources | *test-resource-group* | Utilisez un groupe de ressources existant ou créez-en un. |
   | Emplacement | *USA Est 2* | Sélectionnez une région pour votre compte de partage de données.
   | Nom | *datashareaccount* | Spécifiez un nom pour votre compte de partage de données. |

1. Sélectionnez **Vérifier + créer** puis **Créer** pour provisionner votre compte de partage de données. En règle générale, le provisionnement d’un nouveau compte de partage de données prend environ 2 minutes au maximum.

1. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.

## <a name="create-a-share"></a>Créer un partage

1. Accédez à la page de présentation de votre partage Data Share.

   ![Partager vos données](./media/share-receive-data.png "Partager vos données") 

1. Sélectionnez **Start sharing your data** (Commencer à partager vos données).

1. Sélectionnez **Create** (Créer).

1. Renseignez les détails de votre partage. Spécifiez un nom, un type de partage, une description du contenu du partage et des conditions d’utilisation (facultatif). 

   ![EnterShareDetails](./media/enter-share-details.png "Entrer les détails de partage") 

1. Sélectionnez **Continuer**.

1. Pour ajouter des jeux de données à votre partage, sélectionnez **Add Datasets** (Ajouter des jeux de données). 

   ![Ajouter des jeux de données à votre partage](./media/datasets.png "Groupes de données")

1. Sélectionnez le type de jeu de données à ajouter. Vous verrez une liste différente de types de jeux de données en fonction du type de partage (instantané ou sur place) que vous avez sélectionné à l’étape précédente. 

   ![AddDatasets](./media/add-datasets.png "Ajouter des jeux de données")    

1. Accédez à l’objet à partager, puis sélectionnez Add Datasets (Ajouter des jeux de données). 

   ![SelectDatasets](./media/select-datasets.png "Sélectionner des jeux de données")    

1. Sous l’onglet Destinataires, entrez les adresses e-mail de votre consommateur de données en sélectionnant + Add Recipient (Ajouter un destinataire).

   ![AddRecipients](./media/add-recipient.png "Ajouter des destinataires") 

1. Sélectionnez **Continuer**.

1. Si vous avez sélectionné le type de partage d’instantané, vous pouvez configurer la planification d’instantanés pour fournir des mises à jour de vos données à votre consommateur de données. 

   ![EnableSnapshots](./media/enable-snapshots.png "Activer les captures instantanées") 

1. Sélectionnez une heure de début et un intervalle de périodicité. 

1. Sélectionnez **Continuer**.

1. Sous l’onglet Review + Create (Revoir + Créer), passez en revue le contenu du package, les paramètres, les destinataires ainsi que les paramètres de synchronisation. Sélectionnez **Create** (Créer).

Votre partage Azure Data Share est désormais créé. Le destinataire de votre partage Data Share est maintenant prêt à accepter votre invitation.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque la ressource n’est plus nécessaire, accédez à la page **Vue d’ensemble du partage de données**, puis sélectionnez **Supprimer** pour la supprimer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez vu comment créer une instance Azure Data Share. Pour découvrir comment un consommateur de données peut accepter et recevoir un partage de données, passez au tutoriel [Accepter et recevoir des données](subscribe-to-data-share.md). 
