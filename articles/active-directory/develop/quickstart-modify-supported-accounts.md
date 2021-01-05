---
title: 'Procédure : Changer les types de comptes pris en charge par une application | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce guide de procédure, vous allez configurer une application inscrite auprès de la Plateforme d’identités Microsoft pour changer les utilisateurs ou les comptes qui peuvent accéder à l’application.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 11/15/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: marsma, aragra, lenalepa, sureshja
ms.openlocfilehash: 94a7f4d9ce1471aa1dd6aef3165562a2abc02816
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453262"
---
# <a name="how-to-modify-the-accounts-supported-by-an-application"></a>Guide pratique pour modifier les comptes pris en charge par une application

Lorsque vous avez inscrit votre application auprès de la plateforme d’identités Microsoft, vous avez spécifié qui (quels types de comptes) peut y accéder. Par exemple, vous avez peut-être spécifié des comptes uniquement dans votre organisation, qui est une application *monolocataire*. Ou vous avez peut-être spécifié des comptes dans une organisation quelconque (y compris la vôtre), qui est une application *multilocataire*.

Dans les sections suivantes, vous allez apprendre à modifier l’inscription de votre application dans le portail Azure pour changer les utilisateurs ou les types de comptes qui peuvent accéder à l’application.

## <a name="prerequisites"></a>Prérequis

* Une [application inscrite dans votre locataire Azure AD](quickstart-register-app.md)

## <a name="change-the-application-registration-to-support-different-accounts"></a>Modifier l’inscription d’application pour prendre en charge des comptes différents

Pour spécifier un paramètre différent pour les types de comptes pris en charge par une inscription d’application existante :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications**, puis sélectionnez votre application.
1. À présent, spécifiez qui peut utiliser l’application, à savoir l’*audience de connexion*.

    | Types de comptes pris en charge | Description |
    |-------------------------|-------------|
    | **Comptes dans cet annuaire organisationnel uniquement** | Sélectionnez cette option si vous générez une application destinée à être utilisée uniquement par des utilisateurs (ou des invités) dans *votre* locataire.<br><br>Souvent appelée application *métier* (LOB), il s’agit d’une application **monolocataire** dans la plateforme d’identités Microsoft. |
    | **Comptes dans un annuaire organisationnel** | Sélectionnez cette option si vous souhaitez que les utilisateurs de *n’importe quel* locataire Azure AD puisse utiliser votre application. Cette option est appropriée si, par exemple, vous générez une application Logiciel en tant que service (SaaS) que vous envisagez de fournir à plusieurs organisations.<br><br>C’est une application dite **multilocataire** dans la plateforme d’identités Microsoft. |
1. Sélectionnez **Enregistrer**.

### <a name="why-changing-to-multi-tenant-can-fail"></a>Pourquoi la transition en application multilocataire peut-elle échouer ?

Le basculement d’une inscription d’application monolocataire en application multilocataire peut parfois échouer en raison de collisions de noms d’URI d’ID d’application (URI d’ID d’application). Un exemple d’URI d’ID d’application est `https://contoso.onmicrosoft.com/myapp`.

L’URI ID d’application est l’une des méthodes d’identification d'une application dans les messages de protocole. Pour une application monolocataire, il suffit que l’URI d’ID d’application soit unique au sein de ce locataire. Pour une application multilocataire, il doit être globalement unique afin qu’Azure AD puisse trouver l’application sur tous les locataires. L’unicité globale est appliquée en exigeant que le nom d’hôte de l’URI d’ID d’application corresponde à l’un des [domaines d’éditeur vérifiés](howto-configure-publisher-domain.md) du locataire Azure AD.

Par exemple, si le nom du locataire est *contoso.onmicrosoft.com*, `https://contoso.onmicrosoft.com/myapp` est un URI d’ID d’application valide. Si le locataire a un domaine vérifié de *contoso.com*, un URI d’ID d’application valide serait également `https://contoso.com/myapp`. Si l’URI d’ID d’application ne suit pas le deuxième modèle, `https://contoso.com/myapp`, la conversion de l’inscription de l’application en application multilocataire échoue.

Pour plus d’informations sur la configuration d’un domaine d’éditeur vérifié, consultez [Configurer un domaine vérifié](howto-configure-publisher-domain.md).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les conditions d’une [conversion d’une application monolocataire en application multilocataire](howto-convert-app-to-be-multi-tenant.md).
