---
title: Qu’est-ce que le diagnostic de connexion dans Azure AD ? | Microsoft Docs
description: Présente une vue d’ensemble générale des diagnostics de connexion dans Azure AD.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: e2b3d8ce-708a-46e4-b474-123792f35526
ms.service: active-directory
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 12/15/2020
ms.author: markvi
ms.reviewer: tspring
ms.collection: M365-identity-device-management
ms.openlocfilehash: d6aedf41fbf1ed0d70467a2efe97431fdecaa4fa
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585908"
---
# <a name="what-is-sign-in-diagnostic-in-azure-ad"></a>Qu’est-ce que le diagnostic de connexion dans Azure AD ?

Azure AD fournit un modèle de sécurité flexible permettant de contrôler ce que les utilisateurs peuvent faire avec les ressources gérées. L’accès à ces ressources n’est pas uniquement contrôlé par votre **identité**, il l’est également par le **moyen** que vous utilisez pour accéder aux ressources. En général, la flexibilité s’accompagne d’un certain degré de complexité dû au nombre d’options de configuration dont vous disposez. De cette complexité peut découler une augmentation du risque d’erreurs.

En tant qu’administrateur informatique, vous avez besoin d’une solution qui vous offre le niveau d’insights adapté aux activités de votre système, pour que vous puissiez diagnostiquer et résoudre facilement les problèmes lorsqu’ils se produisent. Le diagnostic de connexion pour Azure AD est un exemple de ce type de solution. Utilisez le diagnostic pour analyser ce qui s’est produit lors d’une connexion, puis déterminer les actions que vous pouvez entreprendre pour résoudre les problèmes, sans devoir forcément faire intervenir le support Microsoft.

Cet article trace un panorama des possibilités de la solution et de la façon dont vous pouvez l’utiliser.


## <a name="requirements"></a>Spécifications

Le diagnostic de connexion est disponible dans toutes les éditions d’Azure AD.<br> Vous devez être administrateur global dans Azure AD pour l’utiliser.

## <a name="how-it-works"></a>Fonctionnement

Dans Azure AD, la réponse à une tentative de connexion est liée à votre **identité** et à la **façon** dont vous accédez à votre locataire. Par exemple, en tant qu’administrateur, vous pouvez habituellement configurer tous les aspects de votre locataire lorsque vous vous connectez à partir de votre réseau d’entreprise. En revanche, vous pouvez très bien être bloqué lorsque vous vous connectez avec le même compte depuis un réseau non approuvé.
 
En raison de la très grande flexibilité du système qui permet de répondre à une tentative de connexion, vous pouvez vous retrouver dans des scénarios où vous devrez résoudre des problèmes de connexion. Le diagnostic de connexion est une fonctionnalité qui :

- analyse les données à partir des connexions ; 

- affiche ce qui s’est passé et fournit des recommandations sur la façon de résoudre les problèmes. 

Le diagnostic de connexion pour Azure AD est conçu pour permettre le diagnostic automatique des erreurs de connexion. Pour suivre le processus de diagnostic, vous devez :

![Processus du diagnostic de connexion](./media/overview-sign-in-diagnostics/process.png)
 
1. **Définir** l’étendue des événements de connexion qui vous intéressent

2. **Sélectionner** la connexion que vous souhaitez examiner

3. **Examiner** le résultat du diagnostic

4. **Agir**

 
### <a name="define-scope"></a>Définir l’étendue

L’objectif de cette étape est de définir l’étendue des connexions que vous souhaitez examiner. Votre étendue est basée soit sur un utilisateur, soit sur un identificateur (correlationId, requestId) et un intervalle de temps. Pour préciser davantage l’étendue, vous pouvez également spécifier un nom d’application. Azure AD utilise les informations d’étendue pour localiser les événements appropriés qui vous intéressent.  

### <a name="select-sign-in"></a>Sélectionner la connexion  

En fonction de vos critères de recherche, Azure AD récupère toutes les connexions correspondantes et les présente dans un mode Liste de récapitulatifs de l’authentification. 

![Récapitulatif de l’authentification](./media/overview-sign-in-diagnostics/authentication-summary.png)
 
Vous pouvez personnaliser les colonnes affichées dans cette vue.

### <a name="review-diagnostic"></a>Examiner le diagnostic 

Pour l’événement de connexion sélectionné, Azure AD vous fournit un résultat de diagnostic. 

![Résultats de diagnostic](./media/overview-sign-in-diagnostics/diagnostics-results.png)

 
Le résultat commence par une évaluation. L’évaluation explique en quelques phrases ce qui s’est passé. L’explication vous aide à comprendre le comportement du système. 

À l’étape suivante, vous obtenez un résumé des stratégies d’accès conditionnel associées qui ont été appliquées à la connexion sélectionnée. Cette partie est effectuée par les étapes correctives recommandées afin de résoudre votre problème. La résolution d’un problème sans aide supplémentaire n’étant pas toujours possible, il peut s’avérer souhaitable d’ouvrir un ticket de support. 

### <a name="take-action"></a>Effectuer une action 
À ce stade, vous devez disposer des informations qui vous sont nécessaires pour résoudre votre problème.


## <a name="scenarios"></a>Scénarios

Cette section fournit une vue d’ensemble des scénarios de diagnostic couverts. Les scénarios suivants sont mis en œuvre : 
 
- Blocage par l’accès conditionnel

- Échec de l’accès conditionnel

- Authentification multifacteur à partir de l’accès conditionnel

- Authentification multifacteur à partir d’autres conditions exigées

- Validation par l’authentification multifacteur exigée

- Validation par l’authentification multifacteur exigée, mais la tentative de connexion de l’utilisateur ne provient pas d’un emplacement sécurisé

- Connexion réussie


### <a name="blocked-by-conditional-access"></a>Blocage par l’accès conditionnel

Ce scénario repose sur une connexion qui a été bloquée par une stratégie d’accès conditionnel.

![Bloquer l’accès](./media/overview-sign-in-diagnostics/block-access.png)

La section de diagnostic de ce scénario affiche des informations détaillées sur la connexion utilisateur et les stratégies appliquées.


### <a name="failed-conditional-access"></a>Échec de l’accès conditionnel

Ce scénario est le résultat classique de l’échec d’une connexion dû à une stratégie d’accès conditionnel dont les conditions n’ont pas été satisfaites. Voici des exemples courants :

![Contrôles exigés](./media/overview-sign-in-diagnostics/require-controls.png)

- Exiger un appareil joint à Azure AD hybride

- Demander une application cliente approuvée

- Exiger une stratégie de protection des applications   


La section de diagnostic de ce scénario affiche des informations détaillées sur la connexion utilisateur et les stratégies appliquées.


### <a name="mfa-from-conditional-access"></a>Authentification multifacteur à partir de l’accès conditionnel

Ce scénario s’appuie sur une stratégie d’accès conditionnel dont la condition exigeant de se connecter au moyen de l’authentification multifacteur est définie.

![Exiger une authentification multifacteur](./media/overview-sign-in-diagnostics/require-mfa.png)

La section de diagnostic de ce scénario affiche des informations détaillées sur la connexion utilisateur et les stratégies appliquées.



### <a name="mfa-from-other-requirements"></a>Authentification multifacteur à partir d’autres conditions exigées

Ce scénario est basé sur une exigence d’authentification multifacteur qui n’a pas été appliquée par une stratégie d’accès conditionnel. Par exemple, l’authentification multifacteur par utilisateur.


![Exiger une authentification multifacteur par utilisateur](./media/overview-sign-in-diagnostics/mfa-per-user.png)


Le but de ce scénario de diagnostic est de fournir plus de détails sur les éléments suivants :

- Source de l’interruption de l’authentification multifacteur 
- Résultat de l’interaction avec le client

Par ailleurs, cette section fournit également tous les détails de la tentative de connexion de l’utilisateur. 


### <a name="mfa-proof-up-required"></a>Validation par l’authentification multifacteur exigée

Ce scénario est basé sur les connexions qui ont été interrompues par des demandes de configuration de l’authentification multifacteur. Cette configuration est également appelée « validation ».

La validation par l’authentification multifacteur se produit lorsqu’un utilisateur est obligé d’utiliser l’authentification multifacteur, mais qu’il ne l’a pas encore configuré, ou qu’un administrateur a configuré l’utilisateur pour qu’il le configure.

Le but de ce scénario de diagnostic est de fournir des informations renseignant sur l’interruption de l’authentification multifacteur, afin de la configurer et de fournir la recommandation qui oblige l’utilisateur à procéder à la validation.

### <a name="mfa-proof-up-required-from-a-risky-sign-in"></a>Validation par l’authentification multifacteur exigée à partir d’une connexion risquée

Ce scénario est l’aboutissement de connexions qui ont été interrompues par une demande de configuration de l’authentification multifacteur émanant d’une connexion risquée. 

Le but de ce scénario de diagnostic est de fournir des informations renseignant sur l’interruption de l’authentification multifacteur, afin de la configurer et de fournir la recommandation qui oblige l’utilisateur à procéder à la validation, mais qui, pour cela, doit le faire depuis un emplacement réseau qui n’apparaît pas risqué. Par exemple, si un réseau d’entreprise est défini en tant qu’emplacement nommé, essayez plutôt d’effectuer la validation à partir du réseau d’entreprise.


### <a name="successful-sign-in"></a>Connexion réussie

Ce scénario est basé sur des connexions qui n’ont été interrompues ni par l’accès conditionnel ni par l’authentification multifacteur.

Le but de ce scénario de diagnostic est de fournir un aperçu de ce que l’utilisateur a produit lors de la connexion, au cas où il y aurait eu une stratégie d’accès conditionnel ou des stratégies devant s’appliquer, ou une authentification multifacteur configurée qui devait interrompre la connexion de l’utilisateur.



## <a name="next-steps"></a>Étapes suivantes

* [Présentation des rapports Azure Active Directory](overview-reports.md)
* [Présentation de la supervision d’Azure Active Directory](overview-monitoring.md)
