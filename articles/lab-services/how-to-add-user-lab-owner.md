---
title: Guide pratique pour ajouter des propriétaires à un labo dans Azure Lab Services
description: Cet article explique comment un administrateur peut ajouter un utilisateur à un labo comme propriétaire dans Azure Lab Services.
ms.topic: article
ms.date: 09/04/2020
ms.openlocfilehash: 4b46dd08a3b9527cdfc7bdf766ee4976e67656d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89482667"
---
# <a name="how-to-add-additional-owners-to-an-existing-lab-in-azure-lab-services"></a>Guide pratique pour ajouter des propriétaires à un labo existant dans Azure Lab Services
Cet article vous explique comment, en tant qu’administrateur, vous pouvez ajouter des propriétaires à un labo existant.

## <a name="add-user-to-the-reader-role-for-the-lab-account"></a>Ajouter un utilisateur au rôle de lecteur pour le compte Lab
Pour ajouter un utilisateur à un labo existant comme propriétaire supplémentaire, vous devez d’abord lui accorder les autorisations de **lecture** sur le compte lab.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu de gauche, sélectionnez **Tous les services**. Effectuez une recherche sur **Lab Services**, puis sélectionnez-le.
3. Sélectionnez votre **compte Lab** dans la liste. 
2. Dans la page **Compte Lab**, sélectionnez **Contrôle d’accès (IAM)** dans le menu de gauche. 
2. Dans la page **Contrôle d’accès (IAM)** , sélectionnez **Ajouter** dans la barre d’outils, puis **Ajouter une attribution de rôle**.

    ![Attribution de rôle pour le compte Lab ](./media/how-to-add-user-lab-owner/lab-account-access-control-page.png)
3. Dans la page **Ajouter une attribution de rôle**, effectuez les étapes suivantes : 
    1. Sélectionnez **Lecteur** pour le **rôle**. 
    2. Sélectionnez l’utilisateur. 
    3. Sélectionnez **Enregistrer**. 

        ![Ajouter un utilisateur au rôle de lecteur pour le compte Lab ](./media/how-to-add-user-lab-owner/reader-lab-account.png)

## <a name="add-user-to-the-owner-role-for-the-lab"></a>Ajouter un utilisateur au rôle de lecteur pour le labo

> [!NOTE]
> Si l’utilisateur dispose d’un accès en lecture seule sur un labo, ce dernier ne s’affiche pas dans labs.azure.com.

1. De retour dans la page **Compte Lab**, sélectionnez **Tous les labs** dans le menu de gauche.
2. Sélectionnez le **labo** auquel vous voulez ajouter l’utilisateur en tant que propriétaire. 
    
    ![Sélectionner le laboratoire ](./media/how-to-add-user-lab-owner/select-lab.png)    
3. Dans la page **Lab**, sélectionnez **Contrôle d’accès (IAM)** dans le menu de gauche.
4. Dans la page **Contrôle d’accès (IAM)**, sélectionnez **Ajouter** dans la barre d’outils, puis **Ajouter une attribution de rôle**.
5. Dans la page **Ajouter une attribution de rôle**, effectuez les étapes suivantes : 
    1. Sélectionnez **Propriétaire** pour le **rôle**. 
    2. Sélectionnez l’utilisateur. 
    3. Sélectionnez **Enregistrer**. 

## <a name="next-steps"></a>Étapes suivantes
Vérifiez que l’utilisateur voit le labo quand il se connecte au [portail Lab Services](https://labs.azure.com).
