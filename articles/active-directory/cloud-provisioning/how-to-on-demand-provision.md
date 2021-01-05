---
title: Configuration de l’approvisionnement cloud à la demande Azure AD Connect
description: Cet article décrit la fonctionnalité d’approvisionnement à la demande.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/14/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 72623d69f9fb0cbc68758a362bb977e5e02d2c00
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91636900"
---
# <a name="azure-ad-connect-cloud-provisioning-on-demand-provisioning"></a>Configuration de l’approvisionnement cloud à la demande Azure AD Connect

L’approvisionnement cloud Azure AD Connect introduit une nouvelle fonctionnalité qui vous permet de tester les modifications de configuration, en appliquant ces modifications à un seul utilisateur.  Vous pouvez l’utiliser pour valider et vérifier que les modifications apportées à la configuration ont été appliquées correctement et sont correctement synchronisées avec Azure AD.  

> [!IMPORTANT] 
> Lorsque vous utilisez la configuration à la demande, les filtres d’étendue ne sont pas appliqués à l’utilisateur que vous avez sélectionné.  Cela signifie que vous pouvez utiliser l’approvisionnement à la demande sur des utilisateurs qui se trouvent en dehors des UO que vous avez spécifiées.


## <a name="using-on-demand-provisioning"></a>Utilisation de l’approvisionnement à la demande
Pour utiliser la nouvelle fonctionnalité, suivez les étapes ci-dessous.


1.  Dans le portail Azure, sélectionnez **Azure Active Directory**.
2.  Sélectionnez ensuite **Azure AD Connect**.
3.  Sélectionnez **Gérer le provisionnement**.

    ![Gérer le provisionnement](media/how-to-configure/manage1.png)
4. Sous **Configuration**, sélectionnez votre configuration.
5. Sous **Valider**, cliquez sur le bouton **Approvisionner un utilisateur**. 

 ![Approvisionner un utilisateur](media/how-to-on-demand-provision/on-demand2.png)

6. Sur l’écran d’approvisionnement à la demande.  Entrez le **nom unique** d’un utilisateur, puis cliquez sur le bouton **Approvisionner**.  
 
 ![Approvisionnement à la demande](media/how-to-on-demand-provision/on-demand3.png)
7. Une fois l’opération terminée, vous devriez voir un écran de réussite et 4 cases à cocher vertes indiquant que l’approvisionnement a réussi.  Les éventuelles erreurs s’affichent à gauche.

  ![Opération réussie](media/how-to-on-demand-provision/on-demand4.png)

Vous pouvez maintenant examiner l’utilisateur et déterminer si les modifications que vous avez apportées à la configuration ont été appliquées.  Le reste de ce document décrit les sections individuelles qui sont affichées dans les détails d’un utilisateur synchronisé avec succès.

## <a name="import-user-details"></a>Détails de l’utilisateur importé
Cette section fournit des informations sur l’utilisateur qui a été importé à partir d’Active Directory.  Voici à quoi ressemble l’utilisateur avant qu’il soit approvisionné dans Azure AD.  Cliquez sur le lien **Afficher les détails** pour afficher ces informations.

![Importer l'utilisateur](media/how-to-on-demand-provision/on-demand5.png)

À l’aide de ces informations, vous pouvez voir les différents attributs qui ont été importés et leurs valeurs.  Si vous avez créé un mappage d’attribut personnalisé, vous serez en mesure de voir la valeur ici.
![Détails de l’utilisateur importé](media/how-to-on-demand-provision/on-demand6.png)

## <a name="determine-if-user-is-in-scope-details"></a>Détails pour déterminer si l’utilisateur est dans l’étendue
Cette section fournit des informations sur l’étendue de l’utilisateur qui a été importé dans Azure AD.  Cliquez sur le lien **Afficher les détails** pour afficher ces informations.

![Étendue d’utilisateur](media/how-to-on-demand-provision/on-demand7.png)

À l’aide de ces informations, vous pouvez consulter des informations supplémentaires sur l’étendue de vos utilisateurs.

![Détails de l’étendue de l’utilisateur](media/how-to-on-demand-provision/on-demand10a.png)

## <a name="match-user-between-source-and-target-system-details"></a>Faire correspondre l'utilisateur entre le système source et les détails du système cible
Cette section indique si l’utilisateur existe déjà dans Azure AD ou non et si une jointure se produit au lieu d’approvisionner un nouvel utilisateur.  Cliquez sur le lien **Afficher les détails** pour afficher ces informations.
![Afficher les détails](media/how-to-on-demand-provision/on-demand8.png)

À l’aide de ces informations, vous pouvez voir si une correspondance a été trouvée ou si un nouvel utilisateur va être créé.

![Informations utilisateur](media/how-to-on-demand-provision/on-demand11.png)

Les détails de correspondance affichent un message avec l’une des trois opérations suivantes.  Il s'agit de :
- Créer : un utilisateur est créé dans Azure AD
- Mettre à jour : un utilisateur est mis à jour selon une modification apportée à la configuration
- Supprimer : un utilisateur est supprimé d’Azure AD.

Selon le type d’opération, le message varie.

## <a name="perform-action-details"></a>Détails de l’action effectuée
Cette section fournit des informations sur l’utilisateur qui a été approvisionné ou exporté dans Azure AD après l’application de la configuration.  Voici à quoi ressemble l’utilisateur une fois qu’il est approvisionné dans Azure AD.  Cliquez sur le lien **Afficher les détails** pour afficher ces informations.
![Détails de l’action effectuée](media/how-to-on-demand-provision/on-demand9.png)

À l’aide de ces informations, vous pouvez voir les valeurs des attributs après l’application de la configuration.  Sont-ils similaires à ce qui a été importé ou sont-ils différents ?  La configuration s’applique-t-elle correctement ?  

Cela vous permet de suivre la transformation des attributs à mesure qu’elle progresse dans le cloud et dans votre locataire Azure AD.

![Attribut trace](media/how-to-on-demand-provision/on-demand12.png)

## <a name="next-steps"></a>Étapes suivantes 

- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)
- [Installer le provisionnement cloud Azure AD Connect](how-to-install.md)
 