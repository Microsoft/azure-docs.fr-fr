---
title: Nouvelle configuration de l’agent de provisionnement cloud Azure AD Connect
description: Cet article explique comment installer le provisionnement cloud.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/26/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: c8b18629a776dd98950f49b1f607cbc876abcd9c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91628885"
---
# <a name="create-a-new-configuration-for-azure-ad-connect-cloud-based-provisioning"></a>Créer une configuration pour le provisionnement cloud Azure AD Connect

Une fois l’agent installé, vous devez vous connecter au portail Azure et configurer le provisionnement cloud Azure Active Directory (Azure AD) Connect. Effectuez les étapes suivantes pour activer l’agent.

## <a name="configure-provisioning"></a>Configurer le provisionnement
Pour configurer le provisionnement, effectuez les étapes suivantes.

 1. Dans le portail Azure, sélectionnez **Azure Active Directory**.
 2. Sélectionnez ensuite **Azure AD Connect**.
 3. Sélectionnez **Gérer le provisionnement**.

 ![Gérer le provisionnement](media/how-to-configure/manage1.png)
 
 4. Sélectionnez **Nouvelle configuration**.
 5. Dans l’écran Configuration, sélectionnez votre domaine et indiquez si vous souhaitez activer ou non la synchronisation du hachage de mot de passe.  Cliquez sur **Créer**.  
 
 ![Créer une configuration](media/how-to-configure/configure1.png)


 6.  L’écran Modifier la configuration de l’approvisionnement s’ouvre.

   ![Modifier la configuration](media/how-to-configure/configure2.png)

 7. Entrez une adresse **E-mail de notification**. Une notification est envoyée à cette adresse e-mail si le provisionnement n’est pas sain.
 8. Déplacez le sélecteur sur Activer, puis sélectionnez Enregistrer.

## <a name="scope-provisioning-to-specific-users-and-groups"></a>Limiter l’étendue du provisionnement à certains utilisateurs ou groupes
Vous pouvez limiter l’étendue de l’agent afin de synchroniser des utilisateurs ou des groupes à l’aide d’unités d’organisation ou de groupes Active Directory locaux. Vous ne pouvez pas configurer des groupes et des unités d’organisation au sein d’une configuration. 

 1.  Dans le portail Azure, sélectionnez **Azure Active Directory**.
 2. Sélectionnez ensuite **Azure AD Connect**.
 3. Sélectionnez **Gérer le provisionnement (préversion)** .
 4. Sous **Configuration**, sélectionnez votre configuration.

 ![Section de configuration](media/how-to-configure/scope1.png)
 
 5. Sous **Configurer**, sélectionnez **Modifier les filtres d’étendue** pour modifier l’étendue de la règle de configuration.
 ![Edit scope](media/how-to-configure/scope3.png)
 7. Sur la droite, vous pouvez modifier l’étendue.  Cliquez sur **Terminé** et **Enregistrer** une fois que vous avez terminé.
 8. Une fois que vous avez changé l’étendue, vous devez [redémarrer le provisionnement](#restart-provisioning) pour lancer une synchronisation immédiate des changements.

## <a name="attribute-mapping"></a>Mappage d’attributs
L’approvisionnement cloud Azure AD Connect vous permet de mapper facilement les attributs entre vos objets utilisateur/groupe locaux et les objets dans Azure AD.  Vous pouvez personnaliser les mappages d’attributs par défaut en fonction des besoins de votre organisation. Vous pouvez ainsi modifier ou supprimer des mappages d’attributs existants ou en créer de nouveaux.  Pour plus d’informations, consultez [Mappage d’attributs](how-to-attribute-mapping.md).

## <a name="on-demand-provisioning"></a>Approvisionnement à la demande
L’approvisionnement cloud Azure AD Connect vous permet de tester les modifications de configuration, en appliquant ces modifications à un seul utilisateur ou groupe.  Vous pouvez l’utiliser pour valider et vérifier que les modifications apportées à la configuration ont été appliquées correctement et sont correctement synchronisées avec Azure AD.  Pour plus d’informations, consultez [Approvisionnement à la demande](how-to-on-demand-provision.md).

## <a name="restart-provisioning"></a>Redémarrer le provisionnement 
Si vous ne souhaitez pas attendre la prochaine exécution planifiée, déclenchez l’exécution du provisionnement à l’aide du bouton **Redémarrer le provisionnement**. 
 1.  Dans le portail Azure, sélectionnez **Azure Active Directory**.
 2. Sélectionnez ensuite **Azure AD Connect**.
 3.  Sélectionnez **Gérer le provisionnement (préversion)** .
 4. Sous **Configuration**, sélectionnez votre configuration.

   ![Sélection de la configuration pour redémarrer le provisionnement](media/how-to-configure/scope1.png)

 5. En haut de la page, sélectionnez **Redémarrer le provisionnement**.

## <a name="remove-a-configuration"></a>Supprimer une configuration
Pour supprimer une configuration, effectuez les étapes suivantes.

 1.  Dans le portail Azure, sélectionnez **Azure Active Directory**.
 2. Sélectionnez ensuite **Azure AD Connect**.
 3. Sélectionnez **Gérer le provisionnement (préversion)** .
 4. Sous **Configuration**, sélectionnez votre configuration.
   
   ![Sélection de la configuration pour supprimer la configuration](media/how-to-configure/scope1.png)

 5. En haut de l’écran de configuration, sélectionnez **Supprimer**.

>[!IMPORTANT]
>Lorsque vous supprimez une configuration, aucune demande de confirmation ne s’affiche. Vous devez donc être sûr de vous avant de sélectionner **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes 

- [Présentation du provisionnement](what-is-provisioning.md)
- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)
