---
title: 'Conception d’identités hybrides : exigences en matière d’authentification multifacteur Azure | Microsoft Docs'
description: Avec le contrôle d’accès conditionnel, Azure AD vérifie les conditions spécifiques que vous choisissez lors de l’authentification de l’utilisateur et avant d’autoriser l’accès à l’application.
documentationcenter: ''
services: active-directory
author: billmath
manager: billmath
editor: ''
ms.assetid: 9c59fda9-47d0-4c7e-b3e7-3575c29beabe
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/18/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7d8ddf372e234bab242e4b28ba53dce7dd68cc89
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90976051"
---
# <a name="determine-multi-factor-authentication-requirements-for-your-hybrid-identity-solution"></a>Déterminer les besoins d'authentification multifacteur (Multi-Factor Authentication) pour votre solution d'identités hybrides
Dans un monde basé sur la mobilité, avec des utilisateurs qui accèdent à des données et à des applications dans le cloud et à partir de n'importe quel périphérique, la sécurisation de ces informations est devenue prééminente.  On entend parler chaque jour de nouvelles failles de sécurité.  Bien qu'il n'existe aucune garantie contre ces violations, l'authentification multifacteur (Multi-Factor Authentication), fournit une couche supplémentaire de sécurité afin de les empêcher.
Commencez par évaluer les besoins des organisations pour en matière d'authentification multifacteur. Il s'agit de comprendre ce que l'organisation veut sécuriser.  Cette version d'évaluation est importante pour définir les exigences techniques pour la configuration et l'actualisation des utilisateurs de l'organisation pour l'authentification multifacteur.

Veillez à répondre aux questions suivantes :

* Votre société tente-t-elle de sécuriser des applications Microsoft ? 
* Comment ces applications sont-elles publiées ?
* Votre société fournit-elle un accès à distance pour permettre aux employés d'accéder aux applications locales ?

Si c'est le cas, de quel type d'accès à distance s'agit-il ? Vous devez également évaluer où les utilisateurs qui accèdent à ces applications se trouvent. Cette version d'évaluation est une autre étape importante pour définir la stratégie d'authentification multifacteur appropriée. Veillez à répondre aux questions suivantes :

* Où les utilisateurs se trouveront-ils ?
* Peuvent-ils se trouver à un endroit quelconque ?
* Votre société souhaite-t-elle établir des restrictions en fonction de la localisation de l'utilisateur ?

Une fois que vous avez compris ces exigences, il est également important d'évaluer les besoins de l'utilisateur en matière d'authentification multifacteur. Cette version d'évaluation est importante, car elle définit les conditions requises pour le déploiement de l'authentification multifacteur. Veillez à répondre aux questions suivantes :

* Les utilisateurs sont-ils familiers avec l'authentification multifacteur ?
* Certains utilisateurs auront-ils besoin d'une authentification supplémentaire ?  
  * Si c'est le cas : tout le temps, lorsqu'ils proviennent de réseaux externes, ou accèdent à des applications spécifiques, ou sous d'autres conditions ?
* Les utilisateurs nécessiteront-ils une formation sur l'installation et la mise en œuvre de l'authentification multifacteur ?
* Quels sont les principaux scénarios pour lesquels votre société souhaite activer l'authentification multifacteur pour ses utilisateurs ?

Après avoir répondu à ces questions, vous serez en mesure de déterminer s'il une authentification multifacteur est déjà mise en œuvre localement. Cette version d'évaluation est importante pour définir les exigences techniques pour la configuration et l'actualisation des utilisateurs de l'organisation pour l'authentification multifacteur. Veillez à répondre aux questions suivantes :

* Votre société a-t-elle besoin de protéger des comptes privilégiés avec l'authentification multifacteur ?
* Votre société a-t-elle besoin d'activer l'authentification multifacteur pour certaines applications pour des raisons de conformité ?
* Votre société a-t-elle besoin d'activer l'authentification multifacteur pour tous les utilisateurs autorisés de ces applications ou uniquement pour les administrateurs ?
* L'authentification multifacteur doit-elle être toujours activée ou uniquement lorsque les utilisateurs se connectent en dehors votre réseau d'entreprise ?

## <a name="next-steps"></a>Étapes suivantes
[Définir une stratégie d'adoption des identités hybrides](plan-hybrid-identity-design-considerations-identity-adoption-strategy.md)

## <a name="see-also"></a>Voir aussi
[Présentation des considérations relatives à la conception](plan-hybrid-identity-design-considerations-overview.md)

