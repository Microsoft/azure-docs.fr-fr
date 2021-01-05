---
title: Gérer les menaces pesant sur les ressources et les données
titleSuffix: Azure AD B2C
description: Découvrez les techniques de détection et de prévention d’attaques par déni de service et d’attaques de mot de passe dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/26/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: e5184698cab1874f327173fb30cf527feee48cad
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85384972"
---
# <a name="manage-threats-to-resources-and-data-in-azure-active-directory-b2c"></a>Gérer les menaces aux ressources et aux données dans Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) dispose de fonctionnalités intégrées qui peuvent vous aider à protéger vos ressources et vos données contre les menaces. Ces menaces incluent des attaques par déni de service et des attaques de mot de passe. Les attaques par déni de service peuvent rendre les ressources inaccessibles aux utilisateurs prévus. Une attaque de mot de passe peut mener à un accès non autorisé aux ressources.

## <a name="denial-of-service-attacks"></a>Attaques par déni de service

Azure AD B2C protège contre les attaques de flux SYN à l’aide d’un cookie SYN. Azure AD B2C protège également contre les attaques par déni de service en limitant les tarifs et les connexions.

## <a name="password-attacks"></a>Attaques de mot de passe

Les mots de passe définis par les utilisateurs doivent être d’une complexité raisonnable. Azure AD B2C dispose de techniques d’atténuation pour contrer les attaques de mot de passe. L’atténuation couvre à la fois la détection des attaques de mot de passe par force brute et celles basées sur un dictionnaire. À l’aide de différents signaux, Azure AD B2C analyse l’intégrité des demandes. Azure AD B2C est conçu pour différencier intelligemment les utilisateurs prévus des pirates et botnets.

Azure AD B2C applique une stratégie sophistiquée pour verrouiller les comptes. Le verrouillage des comptes est basé sur l’adresse IP de la requête et sur les mots de passe entrés. La durée du verrouillage augmente également en fonction de la probabilité de l’existence d’une attaque. Après 10 tentatives d’entrée d’un mot de passe sans succès (le seuil de tentative par défaut), le compte est verrouillé pendant une minute. La prochaine fois qu’une connexion échoue après le déverrouillage du compte (c’est-à-dire après que le compte a été automatiquement déverrouillé par le service à l’expiration de la période de verrouillage), un autre verrouillage d’une minute se produit et se poursuit pour chaque connexion non réussie. L’entrée du même mot de passe à plusieurs reprises n’est pas considérée comme plusieurs connexions ayant échoué.

Les 10 premiers verrouillages durent une minute. Les 10 verrouillages suivants durent un peu plus longtemps, et leur durée augmente tous les 10 verrouillages. Le compteur de verrouillage se remet à zéro après une connexion réussie lorsque le compte n’est pas verrouillé. Les périodes de verrouillage peuvent durer jusqu'à cinq heures.

## <a name="manage-password-protection-settings"></a>Gérer les paramètres de protection par mot de passe

Pour gérer les paramètres de protection par mot de passe, y compris le seuil de verrouillage :

1. Connectez-vous au [portail Azure](https://portal.azure.com)
1. Utilisez le filtre **Annuaire + abonnement** dans le menu du haut pour sélectionner l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sous **Sécurité**, sélectionnez **Méthodes d’authentification (préversion)** , puis **Protection par mot de passe**.
1. Entrez les paramètres de protection de mot par passe souhaités, puis sélectionnez **Enregistrer**.

    ![Page de protection par mot de passe du portail Azure dans les paramètres Azure AD](./media/threat-management/portal-02-password-protection.png)
    <br />*Réglage du seuil de verrouillage à 5 dans les paramètres de **Protection par mot de passe*** .

## <a name="view-locked-out-accounts"></a>Afficher les comptes verrouillés

Pour obtenir des informations sur les comptes verrouillés, vous pouvez consulter le [rapport d’activité de connexion](../active-directory/reports-monitoring/reference-sign-ins-error-codes.md) d’Active Directory. Sous **État**, sélectionnez **Échec**. Échec des tentatives de connexion avec un **code d’erreur de connexion** de `50053` indique un compte verrouillé :

![Section du rapport de connexion Azure AD montrant un compte verrouillé](./media/threat-management/portal-01-locked-account.png)

Pour en savoir plus sur l’affichage du rapport d’activité de connexion dans Azure Active Directory, consultez [Codes d’erreur du rapport d’activité de connexion](../active-directory/reports-monitoring/reference-sign-ins-error-codes.md).
