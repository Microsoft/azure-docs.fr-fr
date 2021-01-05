---
title: La page d’application ne fonctionne pas correctement pour l’application de proxy d’application | Microsoft Docs
description: Conseils quand la page ne s’affiche pas correctement dans une application de proxy d’application que vous avez intégrée à Azure AD
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/21/2018
ms.author: kenwith
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: 780847494da0ec5cd4eb8ab76f1e46125b8a7f8a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84764415"
---
# <a name="application-page-does-not-display-correctly-for-an-application-proxy-application"></a>La page d’application ne s’affiche pas correctement pour une application de proxy d’application

Cet article vous aide à résoudre les problèmes liés aux applications de proxy d’application Azure Active Directory quand vous accédez à la page, mais qu’un élément de la page ne semble pas correct.

## <a name="overview"></a>Vue d’ensemble
Quand vous publiez une application de proxy d’application, seules les pages sous la racine sont accessibles lors de l’accès à l’application. Si la page ne s’affiche pas correctement, l’URL interne racine utilisée pour l’application peut ne pas comporter certaines ressources de la page. Pour résoudre ce problème, vérifiez que vous avez publié *toutes* les ressources de la page dans le cadre de votre application.

Vous pouvez vérifier si les ressources manquantes sont le problème en ouvrant votre dispositif de suivi réseau (tel que Fiddler ou les outils F12 dans Internet Explorer/Microsoft Edge), en chargeant la page et en recherchant les erreurs 404. Cela indique que des pages sont introuvables et doivent encore être publiées.

Supposons, par exemple, que vous avez publié une application de dépenses via l’URL interne de `http://myapps/expenses`, mais que l’application utilise la feuille de style `http://myapps/style.css`. Dans ce cas, comme la feuille de style n’est pas publiée dans votre application, le chargement de l’application de dépenses génère une erreur 404 lors de la tentative de chargement de style.css. Dans cet exemple, le problème est résolu par la publication de l’application avec l’URL interne `http://myapp/`.

## <a name="problems-with-publishing-as-one-application"></a>Problèmes liés à la publication d’une seule application

S’il n’est pas possible de publier toutes les ressources dans la même application, vous devez publier plusieurs applications et activer des liens entre elles.

Pour ce faire, nous vous recommandons d’utiliser la solution des [domaines personnalisés](application-proxy-configure-custom-domain.md). Toutefois, cette solution exige que vous possédiez le certificat pour votre domaine et que vos applications utilisent des noms de domaines complets (FQDN). Pour connaître d’autres options, consultez la [documentation sur la résolution des liens rompus](application-proxy-page-links-broken-problem.md).

## <a name="next-steps"></a>Étapes suivantes
[Publier des applications avec le Proxy d’application Azure AD](application-proxy-add-on-premises-application.md)
