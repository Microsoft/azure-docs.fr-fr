---
title: Rôles et exigences pour Azure Data Share
description: En savoir plus sur les autorisations requises pour partager et recevoir des données à l’aide d’Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: conceptual
ms.date: 10/15/2020
ms.openlocfilehash: ed4b4d9c1de1e9024e8ea86d4661b42d6c68b0ae
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460994"
---
# <a name="roles-and-requirements-for-azure-data-share"></a>Rôles et exigences pour Azure Data Share 

Cet article décrit les rôles et les autorisations nécessaires pour partager et recevoir des données à l’aide du service Azure Data Share. 

## <a name="roles-and-requirements"></a>Rôles et conditions requises

À l’aide du service Azure Data Share, vous pouvez partager des données sans échanger les informations d’identification entre le fournisseur de données et le consommateur de données. Le service Azure Data Share utilise des identités managées (précédemment appelées MSI) pour s’authentifier auprès du magasin de données Azure. 

L’identité managée de la ressource Azure Data Share doit être autorisée à accéder au magasin de données Azure. Le service Azure Data Share utilise ensuite cette identité managée pour lire et écrire des données pour le partage basé sur une capture instantanée et pour établir un lien symbolique pour le partage sur place. 

Pour partager ou recevoir des données à partir d’un magasin de données Azure, l’utilisateur doit au moins disposer des autorisations suivantes. Des autorisations supplémentaires sont requises pour le partage basé sur SQL.

* Autorisation d’écrire dans le magasin de données Azure. En règle générale, cette autorisation existe dans le rôle **Contributeur**.
* Autorisation de créer une attribution de rôle dans le magasin de données Azure. En règle générale, l’autorisation de créer des attributions de rôles existe dans le rôle **Propriétaire**, le rôle Administrateur des accès utilisateur ou un rôle personnalisé doté de l’autorisation Microsoft.Authorization/role assignments/write. Cette autorisation n’est pas requise si l’identité managée de la ressource de partage de données est déjà autorisée à accéder au magasin de données Azure. Consultez le tableau ci-dessous pour connaître le rôle requis.

Voici un résumé des rôles attribués à l’identité managée de la ressource Data Share :

|**Type de magasin de données**|**Magasin de données source de fournisseurs de données**|**Magasin de données cible de consommateurs de données**|
|---|---|---|
|Stockage Blob Azure| Lecteur des données blob du stockage | Contributeur aux données Blob du stockage
|Azure Data Lake Gen1 | Propriétaire | Non pris en charge
|Azure Data Lake Gen2 | Lecteur des données blob du stockage | Contributeur aux données Blob du stockage
|Cluster Azure Data Explorer | Contributeur | Contributeur
|

Pour le partage basé sur SQL, un utilisateur SQL doit être créé à partir d’un fournisseur externe dans la base de données SQL portant le même nom que la ressource Azure Data Share. Une autorisation d’administrateur Azure Active Directory est requise pour créer cet utilisateur. Voici un résumé de l’autorisation requise par l’utilisateur SQL.

|**Type de base de données SQL**|**Autorisation de l’utilisateur SQL fournisseur de données**|**Autorisation de l’utilisateur SQL consommateur de données**|
|---|---|---|
|Azure SQL Database | db_datareader | db_datareader, db_datawriter, db_ddladmin
|Azure Synapse Analytics | db_datareader | db_datareader, db_datawriter, db_ddladmin
|

### <a name="data-provider"></a>Fournisseur de données

Pour ajouter un jeu de données dans Azure Data Share, l’identité managée de la ressource de partage de données du fournisseur doit être autorisée à accéder au magasin de données Azure source. Par exemple, dans le cas d’un compte de stockage, l’identité managée de la ressource de partage de données se voit octroyer le rôle de lecteur des données Blob du stockage. 

Cette opération est effectuée automatiquement par le service Azure Data Share lorsque l’utilisateur ajoute un jeu de données via le Portail Azure et que l’utilisateur dispose de l’autorisation appropriée. Par exemple, l’utilisateur est propriétaire du magasin de données Azure ou est membre d’un rôle personnalisé qui dispose de l’autorisation Microsoft.Authorization/role assignments/write. 

L’utilisateur peut également demander au propriétaire du magasin de données Azure d’ajouter manuellement l’identité managée de la ressource de partage de données au magasin de données Azure. Cette action ne doit être effectuée qu’une seule fois par ressource de partage de données.

Pour créer une attribution de rôle pour l’identité managée de la ressource de partage de données manuellement, suivez les étapes ci-dessous.  

1. Accédez au magasin de données Azure.
1. Sélectionnez **Contrôle d’accès (IAM)** .
1. Sélectionnez **Ajouter une attribution de rôle**.
1. Sous *Rôle*, sélectionnez le rôle dans le tableau d’attribution de rôle ci-dessus (par exemple, pour un compte de stockage, sélectionnez *Lecteur des données Blob du stockage*).
1. Sous *Sélectionner*, saisissez le nom de votre ressource Azure Data Share.
1. Cliquez sur *Enregistrer*.

Pour en savoir plus sur l’attribution de rôle, reportez-vous à [Ajouter ou supprimer des attributions de rôle Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md#add-a-role-assignment). Si vous partagez des données à l’aide d’API REST, vous pouvez créer une attribution de rôle à l’aide d’une API en vous référant à [Ajouter ou supprimer des attributions de rôles Azure à l’aide de l’API REST](../role-based-access-control/role-assignments-rest.md). 

Pour les sources basées sur SQL, un utilisateur SQL doit être créé à partir d’un fournisseur externe dans la base de données SQL portant le même nom que la ressource Azure Data Share lors d’une connexion à la base de données SQL à l’aide de l’authentification Azure Active Directory. Cet utilisateur doit disposer de l’autorisation *db_datareader*. Vous trouverez un exemple de script avec d’autres prérequis pour le partage basé sur SQL dans le tutoriel [Partager à partir d’Azure SQL Database ou Azure Synapse Analytics](how-to-share-from-sql.md). 

### <a name="data-consumer"></a>Consommateur de données
Pour recevoir des données, l’identité managée de la ressource de partage de données du consommateur doit être autorisée à accéder au magasin de données Azure cible. Par exemple, dans le cas d’un compte de stockage, l’identité managée de la ressource de partage de données se voit octroyer le rôle de contributeur aux données Blob du stockage. 

Cette opération est effectuée automatiquement par le service Azure Data Share si l’utilisateur spécifie une banque de données cible via le Portail Azure et que l’utilisateur dispose de l’autorisation appropriée. Par exemple, l’utilisateur est propriétaire du magasin de données Azure ou est membre d’un rôle personnalisé qui dispose de l’autorisation Microsoft.Authorization/role assignments/write. 

L’utilisateur peut également demander au propriétaire du magasin de données Azure d’ajouter manuellement l’identité managée de la ressource de partage de données au magasin de données Azure. Cette action ne doit être effectuée qu’une seule fois par ressource de partage de données.

Pour créer une attribution de rôle pour l’identité managée de la ressource de partage de données manuellement, suivez les étapes ci-dessous. 

1. Accédez au magasin de données Azure.
1. Sélectionnez **Contrôle d’accès (IAM)** .
1. Sélectionnez **Ajouter une attribution de rôle**.
1. Sous *Rôle*, sélectionnez le rôle dans le tableau d’attribution de rôle ci-dessus (par exemple, pour un compte de stockage, sélectionnez *Lecteur des données Blob du stockage*).
1. Sous *Sélectionner*, saisissez le nom de votre ressource Azure Data Share.
1. Cliquez sur *Enregistrer*.

Pour en savoir plus sur l’attribution de rôle, reportez-vous à [Ajouter ou supprimer des attributions de rôle Azure à l’aide du Portail Azure](../role-based-access-control/role-assignments-portal.md#add-a-role-assignment). Si vous recevez des données à l’aide d’API REST, vous pouvez créer une attribution de rôle à l’aide d’une API en vous référant à [Ajouter ou supprimer des attributions de rôles Azure à l’aide de l’API REST](../role-based-access-control/role-assignments-rest.md). 

Pour les cibles basées sur SQL, un utilisateur SQL doit être créé à partir d’un fournisseur externe dans la base de données SQL portant le même nom que la ressource Azure Data Share lors d’une connexion à la base de données SQL à l’aide de l’authentification Azure Active Directory. Cet utilisateur doit disposer de l’autorisation *db_datareader, db_datawriter, db_ddladmin*. Vous trouverez un exemple de script avec d’autres prérequis pour le partage basé sur SQL dans le tutoriel [Partager à partir d’Azure SQL Database ou Azure Synapse Analytics](how-to-share-from-sql.md). 

## <a name="resource-provider-registration"></a>Inscription du fournisseur de ressources 

Vous devrez peut-être inscrire manuellement le fournisseur de ressources Microsoft.DataShare dans votre abonnement Azure dans les scénarios suivants : 

* Afficher l’invitation d’Azure Data Share pour la première fois dans votre locataire Azure
* Partager les données d’un magasin de données Azure dans un abonnement Azure différent de celui de votre ressource Azure Data Share
* Recevoir des données dans un magasin de données Azure dans un abonnement Azure différent de celui de votre ressource Azure Data Share

Suivez ces étapes pour inscrire le fournisseur de ressources Microsoft.DataShare dans votre abonnement Azure. Vous avez besoin d’un accès *Contributeur* à l’abonnement Azure pour inscrire le fournisseur de ressources.

1. Dans le portail Azure, accédez à **Abonnements**.
1. Sélectionnez l’abonnement que vous utilisez pour Azure Data Share.
1. Cliquez sur **Fournisseurs de ressources**.
1. Recherchez Microsoft.DataShare.
1. Cliquez sur **S'inscrire**.
 
Pour en savoir plus sur les fournisseurs de ressources, reportez-vous à [Fournisseurs et types de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les rôles dans Azure : [Comprendre les définitions de rôles](../role-based-access-control/role-definitions.md)