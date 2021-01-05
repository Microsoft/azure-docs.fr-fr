---
title: Éditeur d’attributs d’approvisionnement cloud Azure AD Connect
description: Cet article explique comment utiliser l’éditeur d’attributs.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/22/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7fabad618233e8866c545e1c5ccbcc8cb7508ebf
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652095"
---
# <a name="azure-ad-connect-cloud-provisioning-attribute-mapping"></a>Mappage d’attributs d’approvisionnement cloud Azure AD Connect

L’approvisionnement cloud Azure AD Connect offre une nouvelle fonctionnalité qui vous permet de mapper facilement les attributs entre vos objets utilisateur/groupe locaux et les objets dans Azure AD.  Cette fonctionnalité a été ajoutée à la configuration de l’approvisionnement cloud.

Vous pouvez personnaliser les mappages d’attributs par défaut en fonction des besoins de votre organisation. Vous pouvez ainsi modifier ou supprimer des mappages d’attributs existants ou en créer de nouveaux.  Pour obtenir la liste des attributs qui sont synchronisés, consultez [Attributs synchronisés](../hybrid/reference-connect-sync-attributes-synchronized.md?context=azure%2factive-directory%2fcloud-provisioning%2fcontext%2fcp-context/hybrid/reference-connect-sync-attributes-synchronized.md).

## <a name="understanding-attribute-mapping-types"></a>Présentation des types de mappages d’attributs
Avec les mappages d’attributs, vous contrôlez la façon dont les attributs sont renseignés dans Azure AD.
Quatre différents types de mappages sont pris en charge :

- **Direct** : l’attribut cible est renseigné avec la valeur d’un attribut de l’objet lié dans AD.
- **Constant** : l’attribut cible est renseigné avec une chaîne spécifique que vous avez spécifiée.
- **Expression** : l’attribut cible est renseigné en fonction du résultat d’une expression semblable à un script.
  Pour plus d’informations, consultez l’article [Écriture d’expressions pour les mappages d’attributs](reference-expressions.md).
- **Aucun** : l’attribut cible reste inchangé. Toutefois, si l’attribut cible est vide, il est renseigné avec la valeur par défaut que vous spécifiez.

Avec ces quatre types de base, les mappages d’attributs personnalisés prennent en charge le concept d’affectation de valeur **par défaut** facultative. L’affectation de valeur par défaut garantit qu’un attribut cible est renseigné avec une valeur s’il n’existe aucune valeur dans Azure AD ou sur l’objet cible. La configuration la plus courante consiste à laisser ce champ vide.

## <a name="understanding-attribute-mapping-properties"></a>Présentation des propriétés de mappage d’attributs

La section précédente vous a présenté la propriété de type de mappage d’attributs.
Outre cette propriété, les mappages d’attributs prennent en charge les attributs suivants :

- **Attribut source** : attribut utilisateur du système source (par exemple, Active Directory).
- **Attribut cible** : attribut utilisateur dans le système cible (par exemple, Azure Active Directory).
- **Valeur par défaut si null (facultatif)**  : la valeur qui sera passée au système cible si l’attribut source est null. Cette valeur sera configurée uniquement lors de la création d’un utilisateur. La « valeur par défaut si null » ne sera pas configurée lors de la mise à jour d’un utilisateur existant.  
- **Appliquer ce mappage**
  - **Toujours** : appliquez ce mappage à la création de l’utilisateur et aux actions de mise à jour.
  - **Lors de la création uniquement** : appliquez ce mappage uniquement aux actions de création d’utilisateur.

> [!NOTE]
> Ce document explique comment utiliser le Portail Azure pour mapper des attributs.  Pour plus d’informations sur l’utilisation de Graph, consultez [Transformations](how-to-transformation.md)

## <a name="using-attribute-mapping"></a>Utilisation du mappage d’attributs

Pour utiliser la nouvelle fonctionnalité, suivez les étapes ci-dessous.

1.  Dans le portail Azure, sélectionnez **Azure Active Directory**.
2.  Sélectionnez ensuite **Azure AD Connect**.
3.  Sélectionnez **Gérer le provisionnement**.

    ![Gérer le provisionnement](media/how-to-configure/manage1.png)

4. Sous **Configuration**, sélectionnez votre configuration.
5. Sélectionnez **Cliquez pour modifier les mappages**.  L’écran de mappage d’attributs s’ouvre.

    ![Ajout d'attributs](media/how-to-attribute-mapping/mapping6.png)

6.  Cliquez sur **Ajouter un attribut**.

    ![Type de mappage](media/how-to-attribute-mapping/mapping1.png)

7. Sélectionnez le **Type de mappage**.  Dans cet exemple, nous utilisons Expression.
8.  Entrez l’expression dans la zone.  Pour cet exemple, nous utilisons : `Replace([mail], "@contoso.com", , ,"", ,).`
9.  Entrez l’attribut cible.  Dans cet exemple, nous utilisons ExtensionAttribute15.
10. Sélectionnez le moment de l’application, puis cliquez sur **Appliquer**

    ![Modifier les mappages](media/how-to-attribute-mapping/mapping2a.png)

11. De retour sur l’écran de mappage d’attributs, vous devriez voir votre nouveau mappage d’attribut.  
12. Cliquez sur **Enregistrer le schéma**.

    ![Enregistrer le schéma](media/how-to-attribute-mapping/mapping3.png)

## <a name="test-your-attribute-mapping"></a>Tester votre mappage d’attributs

Pour tester votre mappage d’attributs, vous pouvez utiliser [l’approvisionnement à la demande](how-to-on-demand-provision.md).  Dans la liste 

1. Dans le portail Azure, sélectionnez **Azure Active Directory**.
2. Sélectionnez ensuite **Azure AD Connect**.
3. Sélectionnez **Gérer le provisionnement**.
4. Sous **Configuration**, sélectionnez votre configuration.
5. Sous **Valider**, cliquez sur le bouton **Approvisionner un utilisateur**. 
6. Sur l’écran d’approvisionnement à la demande.  Entrez le **nom unique** d’un utilisateur ou d’un groupe, puis cliquez sur le bouton **Approvisionner**.  
7. Une fois l’opération terminée, vous devriez voir un écran de réussite et 4 cases à cocher vertes indiquant que l’approvisionnement a réussi.  

    ![Réussite de l’approvisionnement](media/how-to-attribute-mapping/mapping4.png)

8. Sous **Exécuter l’action** cliquez sur **Afficher les détails**.  À droite, vous devriez voir le nouvel attribut synchronisé et l’expression appliquée.

  ![Action à effectuer](media/how-to-attribute-mapping/mapping5.png)

## <a name="next-steps"></a>Étapes suivantes

- [Présentation du provisionnement cloud Azure AD Connect](what-is-cloud-provisioning.md)
- [Écriture d’expressions pour les mappages d’attributs](reference-expressions.md)
- [Attributs qui sont synchronisés](../hybrid/reference-connect-sync-attributes-synchronized.md?context=azure%2factive-directory%2fcloud-provisioning%2fcontext%2fcp-context/hybrid/reference-connect-sync-attributes-synchronized.md)