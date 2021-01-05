---
title: 'Azure AD Connect : msExchUserHoldPolicies et cloudMsExchUserHoldPolicies | Microsoft Docs'
description: Cette rubrique décrit le comportement des attributs msExchUserHoldPolicies et cloudMsExchUserHoldPolicies.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 09/15/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0fbda588d99de44c77118586519055a8fc474104
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861763"
---
# <a name="azure-ad-connect---msexchuserholdpolicies-and-cloudmsexchuserholdpolicies"></a>Azure AD Connect - msExchUserHoldPolicies et cloudMsExchUserHoldPolicies
Le document de référence suivant décrit ces attributs utilisés par Exchange et la manière appropriée de modifier les règles de synchronisation par défaut.

## <a name="what-are-msexchuserholdpolicies-and-cloudmsexchuserholdpolicies"></a>Que sont msExchUserHoldPolicies et cloudMsExchUserHoldPolicies ?
Il existe deux types de [conservations](/Exchange/policy-and-compliance/holds/holds) disponibles pour Exchange Server : conservation pour litige et conservation inaltérable. Quand vous activez la conservation pour litige, tous les éléments de boîte aux lettres sont mis en attente.  Une conservation inaltérable sert à conserver uniquement les éléments qui répondent aux critères d’une requête de recherche que vous avez définie à l’aide de l’outil eDiscovery inaltérable.

Les attributs MsExchUserHoldPolicies et cloudMsExchUserHoldPolicies autorisent les services AD et Azure AD locaux à déterminer quels utilisateurs font l’objet d’une conservation, selon qu’ils utilisent Exchange sur site ou Exchange en ligne.

## <a name="msexchuserholdpolicies-synchronization-flow"></a>Flux de synchronisation msExchUserHoldPolicies
Par défaut, MsExchUserHoldPolicies est synchronisé par Azure AD Connect directement avec l’attribut msExchUserHoldPolicies dans le métaverse, puis avec l’attribut msExchUserHoldPolices dans Azure AD.

Les tableaux suivants décrivent le flow :

Flux entrant depuis Active Directory local :

|Attribut Active Directory|Nom de l’attribut|Type de flux|Attribut du métaverse|Règles de synchronisation|
|-----|-----|-----|-----|-----|
|Active Directory local|msExchUserHoldPolicies|Direct|msExchUserHoldPolicies|Entrant depuis AD – Utilisateur Exchange|

Sortant vers Azure AD :

|Attribut du métaverse|Nom de l’attribut|Type de flux|Attribut Azure AD|Règles de synchronisation|
|-----|-----|-----|-----|-----|
|Azure Active Directory|msExchUserHoldPolicies|Direct|msExchUserHoldPolicies|Sortant vers AAD - UserExchangeOnline|

## <a name="cloudmsexchuserholdpolicies-synchronization-flow"></a>Flux de synchronisation cloudMsExchUserHoldPolicies
Par défaut, cloudMsExchUserHoldPolicies est synchronisé par Azure AD Connect directement avec l’attribut cloudMsExchUserHoldPolicies dans le métaverse. Ensuite, si msExchUserHoldPolicies n’a pas la valeur Null dans le métaverse, l’attribut est transmis à Active Directory.

Les tableaux suivants décrivent le flow :

Entrant depuis Azure AD :

|Attribut Active Directory|Nom de l’attribut|Type de flux|Attribut du métaverse|Règles de synchronisation|
|-----|-----|-----|-----|-----|
|Active Directory local|cloudMsExchUserHoldPolicies|Direct|cloudMsExchUserHoldPolicies|Entrant depuis AAD – Utilisateur Exchange|

Sortant vers Active Directory local :

|Attribut du métaverse|Nom de l’attribut|Type de flux|Attribut Azure AD|Règles de synchronisation|
|-----|-----|-----|-----|-----|
|Azure Active Directory|cloudMsExchUserHoldPolicies|IF(NOT NULL)|msExchUserHoldPolicies|Sortant vers AD - UserExchangeOnline|

## <a name="information-on-the-attribute-behavior"></a>Informations sur le comportement de l’attribut
L’attribut msExchangeUserHoldPolicies est un attribut d’autorité unique.  Un attribut d’autorité unique peut être défini sur un objet (dans ce cas, un objet utilisateur) dans l’annuaire local ou dans l’annuaire cloud.  Les règles de source de noms stipulent que si l’attribut est synchronisé à partir d’un emplacement local, alors Azure AD n’est pas autorisé à mettre à jour cet attribut.

Pour autoriser les utilisateurs à définir une stratégie de conservation sur un objet utilisateur dans le cloud, l’attribut cloudMSExchangeUserHoldPolicies est utilisé. Cet attribut est utilisé car Azure AD ne peut pas définir msExchangeUserHoldPolicies directement en fonction des règles expliquées ci-dessus.  Cet attribut est alors resynchronisé avec l’annuaire local si la valeur de msExchangeUserHoldPolicies n’est pas Null et remplace la valeur actuelle de msExchangeUserHoldPolicies.

Dans certaines circonstances, par exemple, si les deux sont modifiés localement et dans Azure en même temps, des problèmes peuvent survenir.  

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
