---
title: Exiger l’authentification MFA à partir de réseaux non approuvés - Azure Active Directory
description: Découvrez comment configurer une stratégie d’accès conditionnel dans Azure Active Directory (Azure AD) pour les tentatives d’accès provenant de réseaux non approuvés.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 10/16/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: e4c020a9be7683bf045dbcc747dad3cb45058dd7
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93077674"
---
# <a name="how-to-require-mfa-for-access-from-untrusted-networks-with-conditional-access"></a>Procédure : Exiger une authentification multifacteur (MFA) pour l’accès à partir de réseaux non approuvés disposant d’un accès conditionnel   

Azure Active Directory (Azure AD) active l’authentification unique aux appareils, applications et des services depuis n’importe quel endroit. Vos utilisateurs peuvent accéder à vos applications cloud non seulement à partir du réseau de votre organisation, mais aussi à partir de n’importe quel emplacement Internet non approuvé. Une meilleure pratique courante pour l’accès à partir de réseaux non approuvés consiste à exiger une authentification multifacteur (MFA).

Cet article vous donne les informations dont vous avez besoin pour configurer une stratégie d’accès conditionnel qui requiert l’authentification MFA pour l’accès à partir de réseaux non approuvés. 

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous connaissez bien les [concepts de base](overview.md) de l’accès conditionnel. 

## <a name="scenario-description"></a>Description du scénario

Pour maîtriser l’équilibre entre sécurité et productivité, il peut suffire d’exiger un mot de passe pour les connexions depuis le réseau de votre organisation. Toutefois, en cas d’accès à partir d’un emplacement réseau non approuvé, le risque de connexion par des utilisateurs non légitimes est accentué. Pour résoudre ce problème, vous pouvez bloquer l’accès à partir de réseaux non approuvés. Il est également possible d’exiger l’authentification multifacteur (MFA) pour obtenir une garantie supplémentaire indiquant que la tentative a été effectuée par le propriétaire légitime du compte. 

L’accès conditionnel Azure AD permet de répondre à cette exigence avec une seule stratégie qui accorde l’accès : 

- à certaines applications cloud ;
- pour certains utilisateurs et groupes ;  
- en exigeant l’authentification multifacteur ; 
- Lorsque l’accès émane : 
   - d’un emplacement non approuvé.

## <a name="implementation"></a>Implémentation

Le défi de ce scénario consiste à traduire l’ *accès à partir d’un emplacement réseau non approuvé* en une condition d’accès conditionnel. Dans une stratégie d’accès conditionnel, vous pouvez configurer la [condition d’emplacements](location-condition.md) de façon à gérer les scénarios liés aux emplacements réseau. Cette condition permet de sélectionner des emplacements nommés, qui sont des regroupements logiques de plages d’adresses IP, de pays et de régions.  

En règle générale, une organisation possède au moins une plage d’adresses, par exemple, 199.30.16.0-199.30.16.15.
Pour configurer un emplacement nommé :

- Spécifiez cette plage (199.30.16.0/28) 
- Attribuez un nom descriptif, par exemple, **Réseau d’entreprise**. 

Au lieu de tenter de définir tous les emplacements non approuvés, vous pouvez :

- Inclure n’importe quel emplacement 

   :::image type="content" source="./media/untrusted-networks/02.png" alt-text="Capture d’écran du volet Emplacements Azure AD, avec l’option Configurer définie sur Oui, l’onglet Inclure visible et l’option N’importe quel emplacement sélectionnée et mise en évidence." border="false":::

- Exclure tous les emplacements approuvés 

   :::image type="content" source="./media/untrusted-networks/01.png" alt-text="Capture d’écran du volet Emplacements Azure AD, avec l’option Configurer définie sur Oui, l’onglet Exclure visible et l’option Tous les emplacements approuvés sélectionnée." border="false":::

## <a name="policy-deployment"></a>Déploiement de stratégie

Suivant l’approche décrite dans cet article, vous pouvez maintenant configurer une stratégie d’accès conditionnel pour les emplacements non approuvés. Pour vous assurer que votre stratégie fonctionne comme prévu, la meilleure pratique recommandée consiste à la tester avant de la déployer en production. Dans l’idéal, utilisez un locataire de test pour vérifier si votre nouvelle stratégie fonctionne comme prévu.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’accès conditionnel, voir [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](./overview.md)
