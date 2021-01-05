---
title: Guide de référence des opérations Azure Active Directory
description: Ce guide de référence des opérations décrit les vérifications et actions à entreprendre pour sécuriser et garantir les opérations, la gouvernance, l’authentification ainsi que la gestion des identités et des accès.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: c815256e60e5ca6582220d247d587a4488c4a55a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74534769"
---
# <a name="azure-active-directory-operations-reference-guide"></a>Guide de référence des opérations Azure Active Directory

Ce guide de référence des opérations décrit les vérifications et actions à entreprendre pour sécuriser et gérer les domaines suivants :

- **[Gestion des identités et des accès](active-directory-ops-guide-iam.md)** - Gestion du cycle de vie des identités et de leurs droits.
- **[Gestion de l’authentification](active-directory-ops-guide-auth.md)** - Gestion des informations d’identification, définition de l’expérience utilisateur d’authentification, délégation de l’affectation, mesure de l’utilisation et définition des stratégies d’accès en fonction de la posture de sécurité de l’entreprise.
- **[Gouvernance](active-directory-ops-guide-govern.md)** - Évaluation et certification de l’accès octroyé aux identités non privilégiées et privilégiées, audit et contrôle des changements apportés à l’environnement.
- **[Opérations](active-directory-ops-guide-ops.md)** - Optimisation des opérations Azure AD (Azure Active Directory).

Certaines suggestions présentées ici ne s’appliquent pas forcément à l’environnement de tous les clients. Par exemple, les bonnes pratiques relatives à AD FS ne s’appliquent peut-être pas si votre organisation utilise la synchronisation du code de hachage du mot de passe.

> [!NOTE]
> Ces suggestions sont valables à la date de publication mais peuvent évoluer. Les organisations doivent évaluer en permanence leurs pratiques de gestion des identités à mesure que les produits et services Microsoft évoluent. Les suggestions peuvent changer quand les organisations s’abonnent à une autre licence Azure AD Premium. Par exemple, Azure AD Premium P2 inclut davantage de suggestions de gouvernance.

## <a name="stakeholders"></a>Parties prenantes

Chaque section de ce guide de référence recommande l’affectation des parties prenantes pour permettre la réussite de la planification et de l’implémentation des tâches clés. Le tableau suivant présente la liste de toutes les parties prenantes de ce guide :

| Partie prenante | Description |
| :- | :- |
| Équipe des opérations IAM | Cette équipe gère les opérations quotidiennes du système de gestion des identités et des accès. |
| Équipe de productivité | Cette équipe détient et gère les applications de productivité telles que l’e-mail, le partage de fichiers et la collaboration ainsi que la messagerie instantanée et les conférences. |
| Propriétaire de l’application | Cette équipe est propriétaire de l’application d’un point de vue commercial et généralement technique au sein d’une organisation. |
| Équipe d’architecture InfoSec | Cette équipe planifie et conçoit les pratiques de sécurité des informations d’une organisation. |
| Équipe des opérations InfoSec | Cette équipe exécute et supervise les pratiques de sécurité des informations implémentées par l’équipe Architecture d’InfoSec. |

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment bien démarrer avec les [vérifications et actions de la gestion des identités et des accès](active-directory-ops-guide-iam.md).
