---
title: Détecter les problèmes d’accès conditionnel à l’aide de l’outil What If - Azure Active Directory
description: Où trouver les stratégies d’accès conditionnel qui ont été appliquées et pourquoi
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: troubleshooting
ms.date: 08/07/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0112ab53c501d639d3f8e0d09d82ef3a12cb93a8
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94837244"
---
# <a name="troubleshooting-conditional-access-using-the-what-if-tool"></a>Résolution des problèmes d’accès conditionnel à l’aide de l’outil What If

L’[outil What If](what-if-tool.md) dans l’accès conditionnel est puissant quand vous essayez de comprendre pourquoi une stratégie a été appliquée à un utilisateur dans une circonstance spécifique ou si une stratégie s’applique à un état connu.

L’outil What If se trouve dans le **Portail Azure** > **Azure Active Directory** > **Accès conditionnel** > **What if**.

![Outil What If de l’accès conditionnel à l’état par défaut](./media/troubleshoot-conditional-access-what-if/conditional-access-what-if-tool.png)

> [!NOTE]
> À l’heure actuelle, l’outil What If n’évalue pas les stratégies en mode rapport seul.

## <a name="gathering-information"></a>Collecte d’informations

L’outil What If requiert uniquement un **utilisateur** pour la prise en main. 

Les informations supplémentaires suivantes sont facultatives, mais elles permettent de limiter l’étendue de cas spécifiques.

* Applications ou actions cloud
* Adresse IP 
* Pays/région
* Plateforme d’appareil.
* Applications clientes (préversion)
* État de l’appareil (préversion) 
* Risque à la connexion

Ces informations peuvent être collectées à partir de l’utilisateur, de leur appareil ou du journal des connexions Azure AD.

## <a name="generating-results"></a>Génération des résultats

Entrez les critères collectés dans la section précédente et sélectionnez **What If** pour générer une liste de résultats. 

À tout moment, vous pouvez sélectionner **Réinitialiser** pour effacer toutes les entrées de critères et revenir à l’état par défaut.

## <a name="evaluating-results"></a>Évaluation des résultats

### <a name="policies-that-will-apply"></a>Stratégies qui vont s’appliquer

Cette liste indique les stratégies d’accès conditionnel qui s’appliquent aux conditions. La liste inclut à la fois les contrôles d’octroi et de session qui s’appliquent. Par exemple, l’authentification multifacteur peut être nécessaire pour accéder à une application spécifique.

### <a name="policies-that-will-not-apply"></a>Stratégies qui ne vont pas s’appliquer

Cette liste affiche les stratégies d’accès conditionnel qui ne s’appliquent pas si les conditions sont appliquées. La liste inclut toutes les stratégies et la raison pour laquelle elles ne s’appliquent pas. Les exemples incluent les utilisateurs et les groupes qui peuvent être exclus d’une stratégie.

## <a name="use-case"></a>Cas d’utilisation

De nombreuses organisations créent des stratégies en fonction des emplacements réseau, en autorisant les emplacements approuvés et les emplacements de blocage où l’accès ne doit pas être autorisé.

Pour vérifier qu’une configuration a été correctement effectuée, un administrateur peut utiliser l’outil What If pour imiter l’accès, à partir d’un emplacement qui doit être autorisé et à partir d’un emplacement qui doit être refusé.

[ ![Outil What If présentant les résultats avec « Bloquer l’accès »](./media/troubleshoot-conditional-access-what-if/conditional-access-what-if-results.png)](./media/troubleshoot-conditional-access-what-if/conditional-access-what-if-results.png#lightbox)

Dans ce cas, l’utilisateur ne peut pas accéder à une application cloud lors de son trajet vers la Corée du Nord, car Contoso a bloqué l’accès à partir de cet emplacement.

Ce test peut être développé pour incorporer d’autres points de données afin de réduire l’étendue.

## <a name="next-steps"></a>Étapes suivantes

* [Qu’est-ce que l’accès conditionnel ?](overview.md)
* [Qu’est-ce que Azure Active Directory Identity Protection ?](../identity-protection/overview-identity-protection.md)
* [Qu’est-ce qu’une identité d’appareil ?](../devices/overview.md)
* [Fonctionnement : Azure AD Multi-Factor Authentication](../authentication/concept-mfa-howitworks.md)