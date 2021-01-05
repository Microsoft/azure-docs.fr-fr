---
title: Inscrire les applications Azure Active Directory pour l’API Azure pour FHIR
description: Ce tutoriel explique quelles sont les applications à inscrire pour l’API Azure pour FHIR et FHIR Server pour Azure.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 10/13/2019
ms.openlocfilehash: aa95dc5cc052fbff6c553de50f4f52dc5df850a5
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398111"
---
# <a name="register-the-azure-active-directory-apps-for-azure-api-for-fhir"></a>Inscrire les applications Azure Active Directory pour l’API Azure pour FHIR

Vous avez le choix entre plusieurs options quand vous configurez l’API Azure pour FHIR ou le projet FHIR Server pour Azure (OSS). Pour le projet open source, vous devez créer votre propre inscription d’application de ressource. Pour l’API Azure pour FHIR, cette application de ressource est créée automatiquement.

## <a name="application-registrations"></a>Inscriptions d’applications

Pour qu’une application interagisse avec Azure AD, elle doit être inscrite. Dans le contexte du serveur FHIR, il existe deux genres d’inscription d’application :

1. Inscriptions d’applications de ressources.
1. Inscriptions d’applications clientes.

Les **applications de ressources** sont des représentations dans Azure AD d’une API ou d’une ressource sécurisée avec Azure AD. C’est le cas notamment de l’API Azure pour FHIR. Une application de ressource liée à l’API Azure pour FHIR est créée automatiquement quand vous provisionnez le service. Toutefois, si vous utilisez le serveur open source, vous devez [inscrire une application de ressource](register-resource-azure-ad-client-app.md) dans Azure AD. Cette application de ressource a un URI d’identificateur. Il est recommandé que cet URI soit identique à l’URI du serveur FHIR. Cet URI doit être utilisé en tant que `Audience` pour le serveur FHIR. Une application cliente peut demander l’accès à ce serveur FHIR quand elle demande un jeton.

Les *applications clientes* sont des inscriptions des clients qui vont demander des jetons. Souvent, dans OAuth 2.0, nous distinguons au moins trois types d’application différents :

1. **Clients confidentiels** , également appelés applications web dans Azure AD. Les clients confidentiels sont des applications qui utilisent le [flux de code d’autorisation](../active-directory/azuread-dev/v1-protocols-oauth-code.md) afin d’obtenir un jeton pour le compte d’un utilisateur connecté présentant des informations d’identification valides. Ils sont appelés clients confidentiels, car ils peuvent détenir un secret et le présenter à Azure AD au moment de l’échange du code d’authentification contre un jeton. Dans la mesure où les clients confidentiels peuvent s’authentifier à l’aide du secret client, ils sont plus fiables que les clients publics, ils peuvent avoir des jetons plus longs, et ils peuvent se voir octroyer un jeton d’actualisation. Découvrez comment [inscrire un client confidentiel](register-confidential-azure-ad-client-app.md). Notez qu’il est important d’inscrire l’URL de réponse à laquelle le client reçoit le code d’autorisation.
1. **Clients publics**. Il s’agit de clients qui ne peuvent pas conserver un secret. En règle générale, il s’agit d’une application d’appareil mobile ou d’une application JavaScript monopage, où le secret client peut être découvert par un utilisateur. Les clients publics utilisent également le flux de code d’autorisation. Toutefois, ils ne sont pas autorisés à présenter un secret au moment de l’obtention d’un jeton. De plus, ils peuvent avoir des jetons dont la durée de vie est plus courte. Il est possible qu’ils ne disposent d’aucun jeton d’actualisation. Découvrez comment [inscrire un client public](register-public-azure-ad-client-app.md).
1. Clients de service. Ces clients obtiennent des jetons en leur nom (et non au nom d’un utilisateur) à l’aide du [flux d’informations d’identification du client](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md). Ils représentent généralement des applications qui accèdent au serveur FHIR de manière non interactive. Le processus d’ingestion en est l’illustration. Durant l’utilisation d’un client de service, il n’est pas nécessaire de démarrer le processus d’obtention d’un jeton par un appel au point de terminaison `/authorize`. Un client de service peut accéder directement au point de terminaison `/token` et présenter l’ID client ainsi que le secret client pour obtenir un jeton. Découvrez comment [inscrire un client de service](register-service-azure-ad-client-app.md)

## <a name="next-steps"></a>Étapes suivantes

Dans cette vue d’ensemble, vous avez passé en revue les types d’inscription d’application dont vous pouvez avoir besoin pour utiliser une API FHIR.

En fonction de la configuration, consultez les guides pratiques pour inscrire vos applications

* [Inscrire une application de ressource](register-resource-azure-ad-client-app.md)
* [Inscrire une application cliente confidentielle](register-confidential-azure-ad-client-app.md)
* [Inscrire une application cliente publique](register-public-azure-ad-client-app.md)
* [Inscrire une application de service](register-service-azure-ad-client-app.md)

Une fois que vous avez inscrit vos applications, vous pouvez déployer l’API Azure pour FHIR.

>[!div class="nextstepaction"]
>[Déployer l’API Azure pour FHIR](fhir-paas-powershell-quickstart.md)