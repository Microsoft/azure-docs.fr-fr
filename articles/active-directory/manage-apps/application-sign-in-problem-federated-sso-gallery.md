---
title: Problèmes de connexion aux applications configurées pour l’authentification unique basée sur SAML
description: Instructions sur la manière de résoudre les problèmes rencontrés lors de la connexion à une application configurée pour l’authentification unique SAML fédérée avec Azure Active Directory
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 02/18/2019
ms.author: kenwith
ms.reviewer: luleon, asteen
ms.custom: contperf-fy21q2
ms.openlocfilehash: 596af29fe72a41b5a86a09e4e6d5072d63b6ac71
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586344"
---
# <a name="problems-signing-in-to-saml-based-single-sign-on-configured-apps"></a>Problèmes de connexion aux applications configurées pour l’authentification unique basée sur SAML
Pour résoudre les problèmes de connexion ci-dessous, nous vous recommandons d’effectuer les opérations suivantes afin de bénéficier du meilleur diagnostic et d’automatiser les étapes de résolution :

- Installez l’[extension de navigateur sécurisée Mes applications](./access-panel-deployment-plan.md) pour aider Azure Active Directory (Azure AD) à fournir un meilleur diagnostic et de meilleures résolutions lorsque vous utilisez l’expérience de test dans le portail Azure.
- Reproduisez l’erreur à l’aide de l’expérience de test sur la page de configuration d’application du portail Azure. En savoir plus sur le [débogage d’applications avec authentification unique SAML](./debug-saml-sso-issues.md)

Si vous utilisez l’[expérience de test](./debug-saml-sso-issues.md) dans le portail Azure avec l’extension de navigateur sécurisée Mes applications, vous n’avez pas besoin de suivre ces étapes manuellement pour ouvrir la page de configuration de l’authentification unique basée sur SAML.

Pour ouvrir la page de configuration de l’authentification unique basée sur SAML :
1.  Ouvrez le [**portail Azure**](https://portal.azure.com/) et connectez-vous comme **Administrateur général** ou **Coadministrateur**.
1.  Ouvrez l’**extension Azure Active Directory** en sélectionnant **Tous les services** en haut du menu de navigation principal de gauche.
1.  Entrez « **Azure Active Directory** » dans la zone de recherche de filtre et sélectionnez l’élément **Azure Active Directory**.
1.  Sélectionnez **Applications d’entreprise** dans le menu de navigation de gauche d’Azure Active Directory.
1.  Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications.

    Si l’application que vous recherchez ne figure pas dans la liste, utilisez la commande **Filtre** en haut de la **liste de toutes les applications** et définissez l’option **Afficher** sur **Toutes les applications**.

1.  Sélectionnez l’application pour laquelle vous souhaitez configurer l’authentification unique.
1. Une fois l’application chargée, sélectionnez **Authentification unique** dans le menu de navigation de gauche de l’application.
1. Sélectionner l’authentification basée sur SAML.

## <a name="application-not-found-in-directory"></a>Application introuvable dans le répertoire
`Error AADSTS70001: Application with Identifier 'https:\//contoso.com' was not found in the directory.`

**Cause possible**

L’attribut `Issuer` envoyé de l’application vers Azure AD dans la requête SAML ne correspond pas à la valeur de l’identificateur configurée pour l’application dans Azure AD.

**Résolution :**

Vérifiez que l’attribut `Issuer` de la requête SAML correspond à la valeur de l’identificateur configurée dans Azure AD.

Dans la page Configuration de l’authentification unique SAML, dans la section **Configuration SAML de base**, vérifiez que la valeur située dans la zone de texte de l’identificateur correspond à celle de l’identificateur mentionnée dans l’erreur.

## <a name="the-reply-address-does-not-match-the-reply-addresses-configured-for-the-application"></a>L’adresse de réponse ne correspond pas aux adresses de réponse configurées pour l’application
`Error AADSTS50011: The reply address 'https:\//contoso.com' does not match the reply addresses configured for the application.`

**Cause possible**

La valeur `AssertionConsumerServiceURL` dans la requête SAML ne correspond pas à la valeur de l’URL de réponse ou au modèle configuré dans Azure AD. La valeur `AssertionConsumerServiceURL` dans la requête SAML est l’URL que vous voyez dans l’erreur.

**Résolution :**

Vérifiez que la valeur `AssertionConsumerServiceURL` de la requête SAML correspond à la valeur de l’URL de réponse configurée dans Azure AD. 

Vérifiez ou mettez à jour la valeur figurant dans la zone de texte URL de réponse pour qu’elle corresponde à la valeur `AssertionConsumerServiceURL` dans la requête SAML.   

Une fois que vous avez mis à jour la valeur de l’URL de réponse dans Azure AD et qu’elle correspond à celle envoyée par l’application dans la requête SAML, vous devez être en mesure de vous connecter à l’application.

## <a name="user-not-assigned-a-role"></a>Utilisateur non affecté à un rôle
`Error AADSTS50105: The signed in user 'brian\@contoso.com' is not assigned to a role for the application.`

**Cause possible**

L’utilisateur ne dispose pas des autorisations nécessaires pour accéder à l’application dans Azure AD.

**Résolution :**

Pour affecter un ou plusieurs utilisateurs directement à une application, consultez [Démarrage rapide : Affecter des utilisateurs à une application](add-application-portal-assign-users.md).

## <a name="not-a-valid-saml-request"></a>Requête SAML non valide
`Error AADSTS75005: The request is not a valid Saml2 protocol message.`

**Cause possible**

Azure AD ne prend pas en charge les requêtes SAML envoyées par l’application pour l’authentification unique. Voici certains problèmes courants :
- Des champs obligatoires sont manquants dans la demande SAML
- La méthode de demande SAML encodée

**Résolution :**

1. Capturez la requête SAML. Pour savoir comment capturer la requête SAML, suivez le didacticiel [Comment déboguer une authentification unique SAML pour des applications dans Azure AD](./debug-saml-sso-issues.md).
1. Contactez le fournisseur de l’application et communiquez-lui les informations suivantes :
    - Demande SAML
    - [Spécifications du protocole SAML d’authentification unique Azure AD](../develop/single-sign-on-saml-protocol.md)

Le fournisseur d’application doit confirmer sa prise en charge de l’implémentation SAML Azure AD pour l’authentification unique.

## <a name="misconfigured-application"></a>Application mal configurée
`Error AADSTS650056: Misconfigured application. This could be due to one of the following: The client has not listed any permissions in the requested permissions in the client's application registration. Or, The admin has not consented in the tenant. Or, Check the application identifier in the request to ensure it matches the configured client application identifier. Please contact your admin to fix the configuration or consent on behalf of the tenant.`

**Cause possible**

L’attribut `Issuer` envoyé de l’application vers Azure AD dans la requête SAML ne correspond pas à la valeur de l’identificateur configurée pour l’application dans Azure AD.

**Résolution :**

Vérifiez que l’attribut `Issuer` de la requête SAML correspond à la valeur de l’identificateur configurée dans Azure AD. 

Vérifiez que la valeur située dans la zone de texte de l’identificateur correspond à celle de l’identificateur mentionnée dans l’erreur.

## <a name="certificate-or-key-not-configured"></a>Certificat ou clé non configuré(e)
`Error AADSTS50003: No signing key configured.`

**Cause possible**

L’objet d’application est endommagé et Azure AD ne reconnaît pas le certificat configuré pour l’application.

**Résolution :**

Pour supprimer et créer un nouveau certificat, effectuez les étapes suivantes :
1. Sur l’écran de configuration de l’authentification unique basée sur SAML, sélectionnez **Créer un certificat** sous **Certificat de signature SAML**.
1. Sélectionnez une date d’expiration, puis cliquez sur **Enregistrer**.
1. Cochez la case **Activer le nouveau certificat** pour substituer le certificat actif. Ensuite, cliquez sur **Enregistrer** en haut du volet, puis acceptez d’activer le certificat de substitution.
1. Dans la section **Certificat de signature SAML**, cliquez sur **Supprimer** pour supprimer le certificat **Inutilisé**.

## <a name="saml-request-not-present-in-the-request"></a>Requête SAML absente de la requête
`Error AADSTS750054: SAMLRequest or SAMLResponse must be present as query string parameters in HTTP request for SAML Redirect binding.`

**Cause possible**

Azure AD n'a pas pu identifier la requête SAML dans les paramètres URL de la requête HTTP. Cela peut se produire si l’application n’utilise pas la liaison de redirection HTTP pour l’envoi de la requête SAML vers Azure AD.

**Résolution :**

L’application doit envoyer la requête SAML encodée dans l’en-tête d’emplacement à l’aide de la liaison de redirection HTTP. Pour plus d'informations sur la mise en œuvre, lisez la section Liaison de redirection HTTP dans le [document de spécification du protocole SAML](https://docs.oasis-open.org/security/saml/v2.0/saml-bindings-2.0-os.pdf).

## <a name="azure-ad-is-sending-the-token-to-an-incorrect-endpoint"></a>Azure AD envoie le jeton vers un point de terminaison incorrect
**Cause possible**

Lors de l’authentification unique, si la requête de connexion ne contient pas une URL de réponse explicite (URL Assertion Consumer Service), Azure AD sélectionne l’une des URL de réponse configurées pour cette application. Même si l’application possède une URL de réponse explicite configurée, l’utilisateur peut être redirigé vers https://127.0.0.1:444. 

Lorsque l’application a été ajoutée comme ne figurant pas sur galerie, Azure Active Directory a créé cette URL de réponse en tant que valeur par défaut. Ce comportement a changé et Azure Active Directory n’ajoute plus cette URL par défaut. 

**Résolution :**

Supprimez les URL de réponse non utilisées qui sont configurées pour l’application.

Sur la page Configuration de l’authentification unique basée sur SAML, dans la section **URL de réponse (URL du service consommateur d’assertion)** , supprimez les URL de réponse non utilisées ou par défaut créées par le système. Par exemple : `https://127.0.0.1:444/applications/default.aspx`.


## <a name="authentication-method-by-which-the-user-authenticated-with-the-service-doesnt-match-requested-authentication-method"></a>La méthode d’authentification de l’utilisateur auprès du service ne correspond pas à la méthode d’authentification demandée.
`Error: AADSTS75011 Authentication method by which the user authenticated with the service doesn't match requested authentication method 'AuthnContextClassRef'. `

**Cause possible**

Le contexte `RequestedAuthnContext` se trouve dans la demande SAML. Cela signifie que l’application attend le contexte `AuthnContext` spécifié par la référence `AuthnContextClassRef`. Toutefois, l’utilisateur s’est déjà authentifié avant d’accéder à l’application, et le contexte `AuthnContext` (méthode d’authentification) utilisé pour cette authentification antérieure est différent de celui demandé. Par exemple, un utilisateur fédéré a accédé à myapps et WIA s’est produit. La référence `AuthnContextClassRef` sera `urn:federation:authentication:windows`. AAD n’effectuera pas de demande de nouvelle authentification, et utilisera le contexte d’authentification que lui a transmis le fournisseur d’identité (ADFS ou tout autre service de fédération dans ce cas). Par conséquent, il y aura une discordance si l’application demande autre chose que `urn:federation:authentication:windows`. Un autre scénario est quand une authentification multifacteur a été utilisée : `'X509, MultiFactor`.

**Résolution :**


`RequestedAuthnContext` est une valeur facultative. Ensuite, si possible, demandez à l’application si elle peut être supprimée.

Une autre option consiste à s’assurer que le contexte `RequestedAuthnContext` soit respecté. Pour ce faire, vous devez demander une nouvelle authentification. Ainsi, lorsque la demande SAML est traitée, une nouvelle authentification est effectuée, et le contexte `AuthnContext` est respecté. Pour demander une nouvelle authentification, la demande SAML contient la valeur `forceAuthn="true"`. 



## <a name="problem-when-customizing-the-saml-claims-sent-to-an-application"></a>Problème lors de la personnalisation des revendications SAML envoyées à une application
Pour savoir comment personnaliser les revendications d’attribut SAML envoyées à votre application, consultez [Mappage de revendications dans Azure Active Directory](../develop/active-directory-claims-mapping.md).

## <a name="errors-related-to-misconfigured-apps"></a>Erreurs liées à des applications mal configurées
Vérifiez que les deux configurations dans le portail correspondent à ce que vous avez dans votre application. Plus précisément, comparez l’ID de client/d’application, les URL de réponse, les clés secrètes client/clés et l’URI ID d’application.

Comparez la ressource à laquelle vous souhaitez accéder dans le code aux autorisations configurées sous l’onglet **Ressources nécessaires** pour vérifier que vous demandez uniquement les ressources que vous avez configurées.

## <a name="next-steps"></a>Étapes suivantes
- [Série de guides de démarrage rapide sur la gestion des applications](add-application-portal-assign-users.md)
- [Comment déboguer une authentification unique SAML pour des applications dans Azure AD](./debug-saml-sso-issues.md)
- [Spécifications du protocole SAML d’authentification unique Azure AD](../develop/single-sign-on-saml-protocol.md)