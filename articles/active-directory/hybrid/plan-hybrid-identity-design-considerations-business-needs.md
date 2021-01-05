---
title: Identifier les exigences en matière de conception d’identités cloud hybrides Azure | Microsoft Docs
description: Identifier les besoins métier de l’entreprise vous conduira à définir les exigences pour la conception d’identités hybrides.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: de690978-84ef-41ad-9dfe-785722d343a1
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 074c203a0a5688855d4f7607a877b25ce6ee6ad7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89660561"
---
# <a name="determine-identity-requirements-for-your-hybrid-identity-solution"></a>Déterminer les besoins d’identité pour votre solution d’identités hybrides
La première étape dans la conception d’une solution d’identités hybrides consiste à déterminer les exigences de l’entreprise qui va exploiter cette solution.  L’identité hybride démarre comme un rôle de support (elle prend en charge toutes les autres solutions cloud en assurant l’authentification), et continue en proposant des fonctionnalités nouvelles et intéressantes qui révèlent de nouvelles charges de travail pour les utilisateurs.  Ces charges de travail ou les services que vous souhaitez adopter pour vos utilisateurs déterminent les exigences pour la conception d’identités hybrides.  Ces services et charges de travail doivent exploiter l’identité hybride à la fois en local et dans le cloud.  

Vous devez parcourir ces aspects essentiels de l’entreprise pour comprendre ce qui est désormais obligatoire et ce que l’entreprise prévoit pour l’avenir. Si vous ne disposez pas d’une visibilité de la stratégie à long terme pour la conception d’identités hybrides, votre solution ne suivra sans doute pas l’évolution et la mutation des besoins métier. Le schéma ci-dessous présente un exemple d’architecture d’identités hybrides et les charges de travail révélées pour les utilisateurs. Il s’agit simplement d’un exemple de toutes les nouvelles possibilités qui peuvent être révélées et proposées avec une stratégie solide d’identités hybrides. 

Certains composants faisant partie de l’architecture d’identités hybrides ![architecture d’identités hybrides ](./media/plan-hybrid-identity-design-considerations/hybrid-identity-architechture.png)

## <a name="determine-business-needs"></a>Déterminer les besoins métier
Les entreprises auront toutes des exigences différentes, même si elles appartiennent au même secteur ; les exigences concrètes peuvent varier. Vous pouvez toujours tirer parti des meilleures pratiques du secteur mais, en fin de compte, ce sont les besoins métier de l’entreprise qui vous permettront de définir les exigences pour la conception d’identités hybrides. 

Veillez à répondre aux questions suivantes pour identifier vos besoins métier :

* Votre entreprise cherche à réduire les coûts d’exploitation informatique ?
* Votre entreprise cherche à sécuriser ses ressources cloud (applications SaaS, infrastructure) ?
* Votre entreprise cherche à moderniser son infrastructure informatique ?
  * Vos utilisateurs sont plus mobiles et réclament au service informatique de créer des exceptions dans votre zone DMZ pour permettre à un trafic différent d’accéder aux différentes ressources ?
  * Votre entreprise possède des applications héritées qui devraient être publiées pour ces utilisateurs modernes, mais qui ne sont pas faciles à réécrire ?
  * Votre entreprise a besoin d’accomplir toutes ces tâches et d’en garder le contrôle simultanément ?
* Votre entreprise cherche à sécuriser les identités des utilisateurs et à réduire les risques en intégrant de nouveaux outils qui tirent parti de l’expertise de sécurité de Microsoft Azure en local ?
* Votre entreprise tente de se débarrasser des comptes « externes » redoutées en local, et de les déplacer vers le cloud où ils ne constituent plus une menace dormante à l’intérieur de votre environnement local ?

## <a name="analyze-on-premises-identity-infrastructure"></a>Analyser votre infrastructure d’identité locale
Maintenant que vous avez une idée des exigences de votre entreprise, vous devez évaluer votre infrastructure locale d’identités. Cette évaluation revêt une importance capitale à l’heure de définir les exigences techniques pour intégrer votre solution  actuelle d’identités au système de gestion des identités dans le cloud. Veillez à répondre aux questions suivantes :

* Quelle solution d’authentification et d’autorisation votre entreprise utilise-t-elle en local ? 
* Votre entreprise dispose-t-elle actuellement de services de synchronisation en local ?
* Votre entreprise utilise-t-elle des fournisseurs d’identité tiers ?

Vous devez également connaître les services cloud dont votre entreprise dispose. Il est très important d’effectuer une évaluation pour comprendre l’intégration actuelle avec les modèles SaaS, IaaS ou PaaS dans votre environnement. Veillez à répondre aux questions suivantes durant cette évaluation :

* Votre entreprise dispose-t-elle d’une intégration avec un fournisseur de services cloud ?
* Le cas échéant, quels sont les services utilisés ?
* Cette intégration est-elle actuellement en production ou s’agit-il d’un pilote ?

> [!NOTE]
> Cloud Discovery analyse vos journaux d’activité de trafic par rapport au catalogue d’applications cloud de Microsoft Cloud App Security, qui contient plus de 16 000 applications cloud classées et évaluées selon plus de 70 facteurs de risque, pour vous fournir une visibilité permanente sur l’utilisation du cloud, les clichés instantanés et le risque des clichés instantanés dans votre organisation. Pour commencer, consultez la [configuration de Cloud Discovery](/cloud-app-security/set-up-cloud-discovery).
> 
> 

## <a name="evaluate-identity-integration-requirements"></a>Évaluer les exigences d’intégration des identités
Ensuite, vous devez évaluer les exigences d’intégration des identités. Cette évaluation revêt une importance capitale à l’heure de définir les exigences techniques sur la manière dont les utilisateurs vont s’authentifier, sur l’aspect de la présence de l’organisation dans le cloud, sur la manière dont l’organisation va gérer les autorisations et sur le type d’expérience de l’utilisateur. Veillez à répondre aux questions suivantes :

* Votre organisation va-t-elle utiliser la fédération, l’authentification standard, ou les deux ?
* La fédération est-elle obligatoire ?  Pour l’une des raisons suivantes :
  * Authentification unique Kerberos
  * Votre entreprise possède des applications locales (générées en interne ou tierces) qui utilisent SAML ou des fonctionnalités de fédération similaires.
  * MFA via des cartes à puce RSA SecurID, etc.
  * Règles d’accès client répondant aux questions ci-dessous :
    1. Puis-je bloquer tous les accès externes à Microsoft 365 en fonction de l’adresse IP du client ?
    2. Puis-je bloquer tous les accès externes à Microsoft 365, à l’exception d’Exchange ActiveSync ?
    3. Puis-je bloquer tous les accès externes à Microsoft 365, à l’exception des applications basées sur le navigateur (Outlook Web App, SPO) ?
    4. Puis-je bloquer tous les accès externes à Microsoft 365 pour les membres de groupes AD désignés ?
* Préoccupations relatives à la sécurité et à l’audit
* Investissement existant déjà dans l’authentification fédérée
* Quel nom notre organisation va-t-elle utiliser pour notre domaine dans le cloud ?
* L’organisation dispose-t-elle d’un domaine personnalisé ?
  1. Ce domaine est-il public et facilement vérifiable via DNS ?
  2. Si ce n’est pas le cas, disposez-vous d’un domaine public qui peut être utilisé pour inscrire un autre UPN dans Active Directory ?
* Les identificateurs d’utilisateur sont-ils cohérents dans la représentation du cloud ? 
* L’organisation dispose d’applications qui requièrent l’intégration avec les services cloud ?
* L’organisation dispose-elle de plusieurs domaines et ceux-ci vont-ils utiliser l’authentification standard ou fédérée ?

## <a name="evaluate-applications-that-run-in-your-environment"></a>Évaluer des applications exécutées dans votre environnement
Maintenant que vous avez une idée de votre infrastructure locale et cloud, vous devez évaluer les applications exécutées dans ces environnements. Cette évaluation revêt une importance capitale à l’heure de définir les exigences techniques pour intégrer ces applications au système de gestion des identités dans le cloud. Veillez à répondre aux questions suivantes :

* Où nos applications vont-elles résider ?
* Les utilisateurs vont-ils accéder aux applications locales ?  Dans le cloud ? Ou les deux ?
* Existe-t-il des plans pour déplacer les charges de travail des applications existantes vers le cloud ?
* Existe-t-il des plans pour développer de nouvelles applications qui résideront en local ou dans le cloud et qui utiliseront l’authentification cloud ?

## <a name="evaluate-user-requirements"></a>Évaluer les exigences des utilisateurs
Vous devez également évaluer les exigences des utilisateurs. Cette évaluation revêt une importance capitale à l’heure de définir les étapes nécessaires pour intégrer et aider les utilisateurs tandis qu’ils passent au cloud. Veillez à répondre aux questions suivantes :

* Les utilisateurs vont-ils accéder aux applications en local ?
* Les utilisateurs vont-ils accéder aux applications dans le cloud ?
* En général, comment les utilisateurs vont-ils se connecter à leur environnement local ?
* Comment les utilisateurs vont-ils se connecter au cloud

> [!NOTE]
> Veillez à noter chaque réponse et à comprendre le raisonnement derrière la réponse. La section [Déterminer les exigences de réponse aux incidents](plan-hybrid-identity-design-considerations-incident-response-requirements.md) passe en revue les options disponibles et les pour et contre de chaque option.  En répondant à chacune de ces questions, vous sélectionnerez l’option correspondant le mieux à vos besoins métier.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
[Déterminer les exigences de synchronisation de répertoire](plan-hybrid-identity-design-considerations-directory-sync-requirements.md)

## <a name="see-also"></a>Voir aussi
[Présentation des considérations relatives à la conception](plan-hybrid-identity-design-considerations-overview.md)

