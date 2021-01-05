---
title: Configuration d’emplacements nommés dans Azure Active Directory | Microsoft Docs
description: Découvrez comment configurer des emplacements nommés.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
ms.assetid: f56e042a-78d5-4ea3-be33-94004f2a0fc3
ms.service: active-directory
ms.workload: identity
ms.subservice: report-monitor
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 94f4d17596936dd9d0ebbdae3c351cac9ed2a570
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89299862"
---
# <a name="quickstart-configure-named-locations-in-azure-active-directory"></a>Démarrage rapide : Configurer des emplacements nommés dans Azure Active Directory

Les emplacements nommés vous permettent d’étiqueter des plages d’adresses IP approuvées dans votre entreprise. Azure AD utilise des emplacements nommés pour effectuer les actions suivantes :
- Détecter les faux positifs dans les [détections de risques](../identity-protection/overview-identity-protection.md). La connexion à partir d’un emplacement approuvé réduit le risque de connexion d’un utilisateur.   
- Configurer l’[accès conditionnel en fonction des emplacements](../conditional-access/location-condition.md).

Dans ce guide de démarrage rapide, vous apprenez à configurer des emplacements nommés dans votre environnement.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce démarrage rapide, les éléments suivants sont requis :

* Un locataire Azure AD. Inscrivez-vous pour obtenir un [essai gratuit](https://azure.microsoft.com/trial/get-started-active-directory/). 
* Un utilisateur, qui est administrateur général du locataire.
* Une plage IP qui est établie et fiable dans votre organisation. La plage d’adresses IP doit être au format **CIDR** (Classless Inter-Domain Routing).

## <a name="configure-named-locations"></a>Configurer des emplacements nommés

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le volet gauche, à partir de la section **Sécurité**, sélectionnez **Azure Active Directory**, puis **Accès conditionnel**.

    ![Onglet Accès conditionnel](./media/quickstart-configure-named-locations/entrypoint.png)

3. Dans la page **Accès conditionnel**, sélectionnez **Emplacements nommés**, puis **Nouvel emplacement**.

    ![Emplacements nommés](./media/quickstart-configure-named-locations/namedlocation.png)

6. Remplissez le formulaire dans la nouvelle page. 

   * Dans la zone de texte **Nom**, tapez un nom pour votre emplacement nommé.
   * Dans la zone **Plages d’adresses IP**, tapez la plage d’adresses IP au format CIDR.  
   * Cliquez sur **Créer**.
    
     ![Panneau Nouveau](./media/quickstart-configure-named-locations/61.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations, consultez les pages suivantes :

- [L’emplacement comme condition dans l’accès conditionnel](../conditional-access/concept-conditional-access-conditions.md#locations).
- [Rapport sur les connexions à risque](../identity-protection/overview-identity-protection.md)