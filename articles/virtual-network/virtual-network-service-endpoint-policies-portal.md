---
title: Créer et associer des stratégies de point de terminaison de service - Portail Azure
titlesuffix: Azure Virtual Network
description: Dans cet article, découvrez comment configurer et associer des stratégies de point de terminaison de service à l’aide du portail Azure.
services: virtual-network
documentationcenter: virtual-network
author: RDhillon
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 02/21/2020
ms.author: rdhillon
ms.openlocfilehash: bdf0e87c92a55d0dbb5bbe34334a6de4580cb350
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004957"
---
# <a name="create-change-or-delete-service-endpoint-policy-using-the-azure-portal"></a>Créer, changer ou supprimer une stratégie de point de terminaison de service à l’aide du portail Azure

Les stratégies de point de terminaison de service vous permettent de filtrer le trafic de réseau virtuel sur des ressources Azure spécifiques sur des points de terminaison de service. Si vous n’êtes pas familiarisé avec les stratégies de point de terminaison de service, consultez la [vue d’ensemble des stratégies de point de terminaison de service](virtual-network-service-endpoint-policies-overview.md) pour en savoir plus.

 Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une stratégie de point de terminaison de service
> * Créer une définition de stratégie de point de terminaison de service
> * Créer un réseau virtuel avec un sous-réseau
> * Associer une stratégie de point de terminaison de service à un sous-réseau

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure 

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-service-endpoint-policy"></a>Créer une stratégie de point de terminaison de service

1. Sélectionnez **+ Créer une ressource** en haut à gauche du portail Azure.
2. Dans le volet de recherche, tapez « stratégie de point de terminaison de service », sélectionnez **Stratégie de point de terminaison de service**, puis **Créer**.

![Créer une stratégie de point de terminaison de service](./media/virtual-network-service-endpoint-policies-portal/create-sep-resource.png)

3. Entrez ou sélectionnez les informations suivantes dans **Informations de base** : 

   - Abonnement : sélectionnez votre abonnement pour la stratégie
   - Groupe de ressources : Sélectionnez **Créer** et entrez *myResourceGroup*.
   - Nom : myEndpointPolicy
   - Emplacement : USA Centre
 
   ![Informations de base sur la création d’une stratégie de point de terminaison de service](./media/virtual-network-service-endpoint-policies-portal/create-sep-basics.png)

4. Sélectionnez **+ Ajouter** sous **Ressources**, puis entrez ou sélectionnez les informations suivantes dans le volet **Ajouter une ressource**

   - Service : seul **Microsoft.Storage** est disponible avec les stratégies de points de terminaison de service
   - Étendue : vous avez le choix entre **Compte unique**, **Tous les comptes de l’abonnement** ou **Tous les comptes du groupe de ressources**
   - Abonnement : sélectionnez votre abonnement pour le compte de stockage. La stratégie et les comptes de stockage peuvent se trouver dans des abonnements différents.
   - Groupe de ressources : Sélectionnez votre groupe de ressources. Obligatoire si l’étendue est définie avec la valeur « Tous les comptes du groupe de ressources » ou « Compte unique ».  
   - Ressource : sélectionnez votre ressource de stockage Azure sous l’abonnement ou le groupe de ressources sélectionné
   - Cliquez sur le bouton **Ajouter** en bas pour terminer l’ajout de la ressource

   ![Définition de stratégie de point de terminaison de service - ressource](./media/virtual-network-service-endpoint-policies-portal/create-sep-add-resource.png)

   - Ajoutez des ressources supplémentaires en répétant les étapes ci-dessus en fonction des besoins

5. Facultatif : Entrez ou sélectionnez les informations suivantes dans **Étiquettes** :
   
   - Clé : sélectionnez votre clé pour la stratégie. Exemple : Département     
   - Valeur : entrez la valeur correspondant à la clé. Exemple : Finances

6. Sélectionnez **Vérifier + créer**. Validez les informations, puis cliquez sur **Créer**. Pour apporter d’autres modifications, cliquez sur **Précédent**. 

   ![Créer des validations finales de stratégie de point de terminaison de service](./media/virtual-network-service-endpoint-policies-portal/create-sep-review-create.png)
  
## <a name="view-endpoint-policies"></a>Afficher des stratégies de points de terminaison 

1. Dans la zone *Tous les services* du portail, commencez à taper *stratégies de point de terminaison de service*. Sélectionnez **Stratégies de points de terminaison de service**.
2. Sous **Abonnements**, sélectionnez votre abonnement et votre groupe de ressources, comme illustré dans l’image suivante.

   ![Afficher une stratégie](./media/virtual-network-service-endpoint-policies-portal/sep-view.png)
       
3. Sélectionnez la stratégie, puis cliquez sur **Définitions de stratégie** pour afficher ou ajouter des définitions de stratégie supplémentaires.

   ![Afficher des définitions de stratégie](./media/virtual-network-service-endpoint-policies-portal/sep-policy-definition.png)

4. Sélectionnez **Sous-réseaux associés** pour afficher les sous-réseaux auxquels la stratégie est associée. Si aucun sous-réseau n’est encore associé, suivez les instructions de l’étape suivante.

   ![Sous-réseaux associés](./media/virtual-network-service-endpoint-policies-portal/sep-associated-subnets.png)
 
5. Associer une stratégie à un sous-réseau

>[!WARNING] 
> Vérifiez que toutes les ressources ayant fait l’objet d’un accès à partir du sous-réseau sont ajoutées à la définition de stratégie avant d’associer la stratégie au sous-réseau indiqué. Une fois que la stratégie est associée, seul l’accès aux ressources de la *liste verte* sera autorisé sur les points de terminaison de service. 
>
> Vérifiez également qu’il n’existe aucun service Azure managé dans le sous-réseau associé à la stratégie de point de terminaison de service

- Avant de pouvoir associer une stratégie à un sous-réseau, vous devez créer un réseau virtuel et un sous-réseau. Pour obtenir de l’aide, reportez-vous à l’article [Créer un réseau virtuel](./quick-create-portal.md).

- Une fois que vous avez configuré le réseau virtuel et le sous-réseau, vous devez configurer des points de terminaison de service de réseau virtuel pour le stockage Azure. Dans le panneau Réseau virtuel, sélectionnez **Points de terminaison de service** puis, dans le volet suivant, sélectionnez **Microsoft.Storage** et le réseau virtuel ou le sous-réseau souhaité sous **Sous-réseaux**

- À présent, vous pouvez choisir de sélectionner la stratégie de point de terminaison de service dans la liste déroulante du volet ci-dessus si vous avez déjà créé des stratégies de points de terminaison de service avant de configurer le point de terminaison de service pour le sous-réseau, comme indiqué ci-dessous

    ![Associer un sous-réseau pendant la création du point de terminaison de service](./media/virtual-network-service-endpoint-policies-portal/vnet-config-service-endpoint-add-sep.png)

- OU, si vous associez des stratégies de points de terminaison de service après avoir déjà configuré des points de terminaison de service, vous pouvez choisir d’associer le sous-réseau à partir du panneau Stratégie de point de terminaison de service en accédant au volet **Sous-réseaux associés**, comme indiqué ci-dessous

    ![Associer un sous-réseau via SEP](./media/virtual-network-service-endpoint-policies-portal/sep-edit-subnet-association.png)

>[!WARNING] 
>L’accès aux ressources de stockage Azure dans toutes les régions est limité en fonction de la stratégie de point de terminaison de service de ce sous-réseau.

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez créé une stratégie de point de terminaison de service que vous avez associée à un sous-réseau. Pour en savoir plus sur les stratégies de point de terminaison de service, consultez la [vue d’ensemble des stratégies de point de terminaison de service](virtual-network-service-endpoint-policies-overview.md).
