---
title: Activer la vérification de l’âge dans Azure Active Directory B2C | Microsoft Docs
description: Découvrez comment identifier les mineurs à l’aide de votre application.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/13/2018
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 6bd93f9062f8446ce20436a7a04e2054aaf5be71
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85386129"
---
# <a name="enable-age-gating-in-azure-active-directory-b2c"></a>Activer la vérification de l'âge dans Azure Active Directory B2C

>[!IMPORTANT]
>Cette fonctionnalité est en version préliminaire publique. N’utilisez pas cette fonctionnalité pour les applications de production.
>

La vérification de l’âge dans Azure Active Directory B2C (Azure AD B2C) vous permet d’identifier les mineurs qui souhaitent utiliser votre application. Vous pouvez choisir d’empêcher un mineur de se connecter à l’application. Les utilisateurs peuvent également revenir à l’application et indiquer leur tranche âge et l’état de leur consentement parental. Azure AD B2C peut bloquer les mineurs n’ayant pas de consentement parental. Vous pouvez également configurer Azure AD B2C pour que l’application gère elle-même les mineurs.

Après avoir activé la vérification de l’âge dans votre [flux utilisateur](user-flow-overview.md), les utilisateurs sont invités à entrer leur date de naissance et leur pays/région de résidence. Si un utilisateur qui se connecte n’a pas précédemment entré ces informations, il devra le faire à sa prochaine connexion. Les règles sont appliquées chaque fois qu’un utilisateur se connecte.

Azure AD B2C utilise les informations indiquées par l’utilisateur pour déterminer s’il est mineur. Le champ **ageGroup** est ensuite mis à jour dans son compte. La valeur peut être `null`, `Undefined`, `Minor`, `Adult` ou `NotAdult`.  Les champs **ageGroup** et **consentProvidedForMinor** sont ensuite utilisés pour calculer la valeur de **legalAgeGroupClassification**.

La vérification de l’âge implique deux valeurs d’âge : l’âge auquel un utilisateur n’est plus considéré comme mineur et l’âge auquel un mineur doit obtenir un accord parental. Le tableau suivant liste les règles d’âge utilisées pour déterminer si un utilisateur est mineur et s’il a besoin d’un accord parental.

| Pays/région | Nom du pays/de la région | Accord parental | Majorité |
| -------------- | ------------------- | ----------------- | --------- |
| Default | None | None | 18 |
| AE | Émirats Arabes Unis | None | 21 |
| AT | Autriche | 14 | 18 |
| BE | Belgique | 14 | 18 |
| BG | Bulgarie | 16 | 18 |
| BH | Bahreïn | None | 21 |
| CM | Cameroun | None | 21 |
| CY | Chypre | 16 | 18 |
| CZ | République tchèque | 16 | 18 |
| DE | Allemagne | 16 | 18 |
| DK | Danemark | 16 | 18 |
| EE | Estonie | 16 | 18 |
| EG | Égypte | None | 21 |
| ES | Espagne | 13 | 18 |
| FR | France | 16 | 18 |
| Go | Royaume-Uni | 13 | 18 |
| GR | Grèce | 16 | 18 |
| HR | Croatie | 16 | 18 |
| HU | Hongrie | 16 | 18 |
| IE | Irlande | 13 | 18 |
| IT | Italie | 16 | 18 |
| KR | Corée, République de | 14 | 18 |
| LT | Lituanie | 16 | 18 |
| LU | Luxembourg | 16 | 18 |
| LV | Lettonie | 16 | 18 |
| MT | Malte | 16 | 18 |
| N/D | Namibie | None | 21 |
| NL | Pays-Bas | 16 | 18 |
| PL | Pologne | 13 | 18 |
| PT | Portugal | 16 | 18 |
| RO | Roumanie | 16 | 18 |
| SE | Suède | 13 | 18 |
| SG | Singapour | None | 21 |
| SI | Slovénie | 16 | 18 |
| SK | Slovaquie | 16 | 18 |
| TD | Tchad | None | 21 |
| MJ | Thaïlande | None | 20 |
| TW | Taïwan | None | 20 |
| US | États-Unis | 13 | 18 |

## <a name="age-gating-options"></a>Options de vérification de l’âge

### <a name="allowing-minors-without-parental-consent"></a>Autoriser les mineurs sans consentement parental

Pour les flux utilisateur impliquant des opérations de création de compte et/ou de connexion, vous pouvez choisir d’autoriser les mineurs sans accord dans votre application. Les mineurs sans accord parental sont autorisés à se connecter ou à créer un compte normalement, et Azure AD B2C émet un jeton d’ID avec la revendication **legalAgeGroupClassification**. Cette revendication définit l’expérience des utilisateurs, notamment la collecte de l’accord parental et la mise à jour du champ **consentProvidedForMinor**.

### <a name="blocking-minors-without-parental-consent"></a>Bloquer les mineurs sans consentement parental

Pour les flux utilisateur impliquant des opérations de création de compte et/ou de connexion, vous pouvez choisir de bloquer les mineurs sans consentement de l’application. Les options suivantes vous permettent de gérer les utilisateurs bloqués dans Azure AD B2C :

- Renvoyer un JSON à l’application : cette option envoie à l’application une réponse indiquant que le mineur a été bloqué.
- Afficher une page d’erreur : l’utilisateur voit une page l’informant qu’il ne peut pas accéder à l’application.

## <a name="set-up-your-tenant-for-age-gating"></a>Configurer la vérification de l’âge sur un locataire

Pour utiliser la vérification de l’âge dans un flux utilisateur, vous devez configurer des propriétés supplémentaires sur votre locataire.

1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire et abonnement** dans le menu supérieur. Sélectionnez l’annuaire qui contient votre locataire.
2. Sélectionnez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
3. Sélectionnez **Propriétés** pour votre locataire dans le menu de gauche.
2. Sous la section **Vérification de l’âge**, cliquez sur **Configurer**.
3. Une fois l’opération terminée, la vérification de l’âge est configurée sur votre locataire.

## <a name="enable-age-gating-in-your-user-flow"></a>Activer la vérification de l’âge dans votre flux utilisateur

Une fois que votre locataire est configuré pour utiliser la vérification de l’âge, vous pouvez utiliser cette fonctionnalité dans les [flux utilisateur](user-flow-versions.md) dans lesquels elle est activée. Pour activer la vérification de l’âge, procédez comme suit :

1. Créez un flux utilisateur dans lequel la vérification de l’âge est activée.
2. Après avoir créé le flux utilisateur, sélectionnez **Propriétés** dans le menu.
3. Dans la section **Vérification de l’âge**, sélectionnez **Activée**.
4. Vous pouvez ensuite choisir la façon dont vous souhaitez gérer les utilisateurs qui s’identifient comme étant mineurs. Pour **Stratégies d’inscription ou de connexion**, sélectionnez `Allow minors to access your application` ou `Block minors from accessing your application`. Si le blocage de mineurs est sélectionné, sélectionnez `Send a JSON back to the application` ou `Show an error message`.




