---
title: itsme OpenID Connect avec Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer l’authentification Azure AD B2C à itsme OIDC à l’aide de la stratégie Flux d’utilisateurs clé secrète client. itsme est une application d’identificateur numérique. Elle permet de se connecter de façon sécurisée sans lecteurs de cartes, mots de passe, authentification à deux facteurs ni codes secrets multiples.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: ba7875caa6a1db7638bfeafcfea1efa7b2462152
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87489513"
---
# <a name="configure-itsme-openid-connect-oidc-with-azure-active-directory-b2c"></a>Configuration d’itsme OpenID Connect (OIDC) avec Azure Active Directory B2C

L’application d’identificateur numérique itsme permet de se connecter de façon sécurisée sans lecteurs de cartes, mots de passe, authentification à deux facteurs ni codes secrets multiples. L’application itsme assure une authentification client forte avec une identité vérifiée. Dans cet article, découvrez comment intégrer l’authentification Azure AD B2C à itsme OpenID Connect (OIDC) à l’aide d’une stratégie Flux d’utilisateurs clé secrète client.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un [locataire Azure AD B2C](tutorial-create-tenant.md) lié à votre abonnement Azure.
* Votre ID client, également appelé Code de partenaire, fourni par itsme.
* Votre Code de service fourni par itsme.
* La clé secrète client de votre compte itsme.

## <a name="scenario-description"></a>Description du scénario

![Diagramme de l’architecture itsme](media/partner-itsme/itsme-architecture-diagram.png)

<!--
Please clarify step 1 in the description below - we don't have steps in this tutorial for "adapting in the Azure AD B2C Custom Policy- User Journeys" - should this be added somewhere?
-->

| Étape | Description |
|------|------|
|1     | Sur votre site web ou votre application, incluez le bouton **Se connecter avec itsme** en l’adaptant au flux d’utilisateurs Azure AD B2C. Le flux d’interaction démarre lorsque l’utilisateur clique sur ce bouton.  |
|2     | Azure AD B2C démarre le flux OpenID Connect en envoyant une demande d’autorisation à l’API de clé secrète client itsme. Un point de terminaison de configuration bien connu/OpenID est disponible ; il contient des informations sur les points de terminaison.  |
|3     | L’environnement itsme redirige l’utilisateur vers la page itsme Identifiez-vous, qui permet à l’utilisateur de remplir son numéro de téléphone.  |
|4     | L’environnement itsme reçoit le numéro de téléphone de l’utilisateur et en valide l’exactitude.  |
|5     | Si le numéro de téléphone appartient à un utilisateur itsme actif, une action est créée pour l’application itsme.  |
|6     | L’utilisateur ouvre l’application itsme, vérifie la demande et confirme l’action.  |
|7     |  L’application informe l’environnement itsme que l’action a été confirmée. |
|8     |  L’environnement itsme envoie le code d’autorisation OAuth à Azure AD B2C. |
|9     |  À l’aide du code d’autorisation, Azure AD B2C effectue une demande de jeton. |
| 10 | L’environnement itsme vérifie la demande de jeton et, si elle est toujours valide, retourne le jeton d’accès OAuth et le jeton d’identification contenant les informations utilisateur demandées. |
| 11 | Enfin, l’utilisateur est redirigé vers l’URL de redirection en tant qu’utilisateur authentifié.  |
|   |   |

## <a name="onboard-with-itsme"></a>Intégration avec itsme

1. Pour créer un compte avec itsme, visitez itsme sur la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace).

2. Activez votre compte itsme en envoyant un e-mail à onboarding@itsme.be. Vous recevrez un **Code de partenaire** et un **Code de service** qui seront nécessaires pour votre configuration B2C.

3. Après l’activation de votre compte de partenaire itsme, vous recevrez un e-mail contenant un lien unique vers la **clé secrète client**.

4. Suivez les instructions indiquées sur [itsme](https://business.itsme.be/en) pour effectuer la configuration.

## <a name="integrate-with-azure-ad-b2c"></a>Intégrer avec Azure AD B2C

### <a name="set-up-a-new-identity-provider-in-azure-ad-b2c"></a>Configuration d’un nouveau fournisseur d’identité dans Azure AD B2C

> [!NOTE]
> Si vous n’en avez pas, [créez un locataire Azure AD B2C](tutorial-create-tenant.md) qui est lié à votre abonnement Azure.

1. Veillez à bien utiliser le répertoire qui contient le locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez l’annuaire qui contient votre locataire Azure AD B2C.

2. Sous **Services Azure**, sélectionnez **Azure AD B2C** (ou sélectionnez **Plus de services** et utilisez la zone de recherche **Tous les services** pour rechercher *Azure AD B2C*).

3. Sélectionnez **Fournisseurs d’identité**, puis **Nouveau fournisseur OpenID Connect**.

4. Remplissez le formulaire avec les informations suivantes :

   |Propriété | Valeur |
   |------------ |------- |
   | Nom | itsme |
   | URL de métadonnées | `https://oidc.<environment>.itsme.services/clientsecret-oidc/csapi/v0.1/.well-known/openid-configuration` <br>où `<environment>` est `e2e` (environnement de test) ou `prd` (production)  |
   | ClientID     | Votre **ID client**, également appelé **Code de partenaire**  |
   | Clé secrète client | Votre **clé secrète client** |
   | Étendue  | openid service:VOTRE_CODE_DE_SERVICE profile email [numéro_de_téléphone] [adresse]  |
   |Type de réponse | code |
   |Mode de réponse | query |
   |Indication du domaine | *Laissez ce champ vide* |
   |UserID | sub |
   |Nom d’affichage | name |
   |Prénom | given_name |
   |Surname | family_name |
   |E-mail | email|

5. Sélectionnez **Enregistrer**.

### <a name="configure-a-user-flow"></a>Configuration d’un flux d’utilisateurs

1. Dans votre locataire Azure AD B2C, sélectionnez **Flux d’utilisateurs** sous **Stratégies**.

2. Sélectionnez **Nouveau flux d’utilisateurs**.

3. Sélectionnez **Inscription et connexion**, sélectionnez une version, puis sélectionnez **Créer**.

4. Saisissez un **Nom**.

5. Dans la section **Fournisseurs d’identité**, sélectionnez **itsme**.

6. Sélectionnez **Create** (Créer).

7. Ouvrez le nouveau flux d’utilisateurs en sélectionnant son nom.

8. Sélectionnez **Propriétés** et ajustez les valeurs suivantes :

   * **Durée de vie du jeton d’accès et d’identification (en minutes)** : **5**.
   * **Durée de vie de la fenêtre glissante du jeton d’actualisation** : **N’expire jamais**.

### <a name="register-an-application"></a>Inscrire une application

1. Dans votre locataire B2C, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription** sous **Gérer**.

2. Donnez un **Nom** à l’application, puis entrez votre **URI de redirection**. Entrez `https://jwt.ms` à des fins de test.

3. L’authentification multifacteur doit être **Désactivée**.

4. Sélectionnez **Inscription**.

   a. À des fins de test, sélectionnez **Authentification**, puis cochez les cases **Jetons d’accès** et **Jetons d’identification** sous **Octroi implicite**.  

   b. Sélectionnez **Enregistrer**.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Dans votre locataire B2C, sélectionnez **Flux d’utilisateurs** sous **Stratégies**.

2. Sélectionnez le flux d’utilisateurs que vous avez créé.

3. Sélectionnez **Exécuter le flux utilisateur**.

   a. **Application** : *sélectionnez l’application enregistrée*

   b. **URL de réponse** : *sélectionnez l’URL de redirection*

4. La page itsme **Identifiez-vous** s’affiche.  

5. Entrez votre numéro de téléphone mobile, puis sélectionnez **Envoyer**.

6. Confirmez l’action dans l’application itsme.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

* [Stratégies personnalisées dans Azure AD B2C](custom-policy-overview.md)

* [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](custom-policy-get-started.md?tabs=applications)