---
title: Ajouter des approbations personnalisées à des flux d’inscription en libre-service - Azure AD
description: Ajouter des connecteurs d’API pour les flux de travail d’approbation personnalisés dans l’inscription en libre-service des identités externes - Azure Active Directory (Azure AD)
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: article
ms.date: 06/16/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 82f0408b7e46493f6c3ec62d48a992e87f196f78
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860607"
---
# <a name="add-a-custom-approval-workflow-to-self-service-sign-up"></a>Ajouter un flux de travail d’approbation personnalisé à l’inscription en libre-service

Avec les [connecteurs d’API](api-connectors-overview.md), vous pouvez intégrer vos propres flux de travail d’approbation personnalisés à l’inscription en libre-service afin de pouvoir gérer les comptes d’utilisateurs invités qui sont créés dans votre locataire.

Cet article fournit un exemple d’intégration à un système d’approbation. Dans cet exemple, le workflow utilisateur d’inscription en libre-service collecte les données utilisateur pendant le processus d’inscription et les transmet à votre système d’approbation. Le système d’approbation peut ensuite :

- Approuver automatiquement l’utilisateur et autoriser Azure AD à créer le compte d’utilisateur.
- Déclencher une révision manuelle. Si la requête est approuvée, le système d’approbation utilise Microsoft Graph pour approvisionner le compte d’utilisateur. Le système d’approbation peut également informer l’utilisateur que son compte a été créé.

## <a name="register-an-application-for-your-approval-system"></a>Inscrire une application pour votre système d’approbation

Vous devez inscrire votre système d’approbation en tant qu’application dans votre locataire Azure AD pour qu’il puisse s’authentifier auprès de Azure AD et avoir l’autorisation de créer des utilisateurs. En savoir plus sur les [notions de base de l’authentification et des autorisations pour Microsoft Graph](/graph/auth/auth-concepts).

1. Connectez-vous au [Portail Azure](https://portal.azure.com) en tant qu’administrateur Azure AD.
2. Sous **Services Azure**, sélectionnez **Azure Active Directory**.
3. Dans le menu de gauche, sélectionnez **Inscriptions d’applications**, puis sélectionnez **Nouvelle inscription**.
4. Entrez un **Nom** pour l’application, par exemple, _Approbations d’inscription_.

   <!-- ![Register an application for the approval system](./self-service-sign-up-add-approvals/approvals/register-an-approvals-application.png) -->

5. Sélectionnez **Inscription**. Vous pouvez conserver les valeurs par défaut des autres champs.

   ![Capture d’écran mettant en évidence le bouton Inscription.](media/self-service-sign-up-add-approvals/register-approvals-app.png)

6. Sous **Gérer** dans le menu de gauche, sélectionnez **Autorisations de l’API**, puis sélectionnez **Ajouter une autorisation**.
7. Dans la page **Demander des autorisations d’API**, sélectionnez **Microsoft Graph**, puis **Autorisations d’application**.
8. Sous **Sélectionner autorisations**, développez **Utilisateur**, puis activez la case à cocher **User.ReadWrite.All (Utilisateur.LireDroite.Tous)** . Cette autorisation permet au système d’approbation de créer l’utilisateur lors de l’approbation. Sélectionnez ensuite **Ajouter des autorisations**.

   ![Page Inscription d’une application](media/self-service-sign-up-add-approvals/request-api-permissions.png)

9. Dans la page **Autorisations de l’API**, sélectionnez **Accorder le consentement administrateur pour (nom de votre locataire)** , puis **Oui**.
10. Sous **Gérer** dans le menu de gauche, sélectionnez **Certificats et secrets**, puis **Nouvelle clé secrète client**.
11. Entrez une **Description** pour la clé secrète, par exemple _Approbations client secrètes_, puis sélectionnez le délai d’**Expiration** de la clé secrète client. Sélectionnez ensuite **Ajouter**.
12. Copiez la valeur de la clé secrète client.

    ![Copiez la clé secrète client pour l’utiliser dans le système d’approbation](media/self-service-sign-up-add-approvals/client-secret-value-copy.png)

13. Configurez votre système d’approbation pour utiliser l’**ID d’application** en tant qu’ID client et la **clé secrète client** que vous avez générée pour vous authentifier avec Azure AD.

## <a name="create-the-api-connectors"></a>Créer les connecteurs d’API

Ensuite, vous allez[créer les connecteurs d’API](self-service-sign-up-add-api-connector.md#create-an-api-connector) pour le workflow de votre utilisateur d’inscription en libre-service. Votre API de système d’approbation a besoin de deux connecteurs et de points de terminaison correspondants, comme les exemples ci-dessous. Ces connecteurs d’API effectuent les opérations suivantes :

- **Vérifiez l’état d’approbation**. Envoyer un appel au système d’approbation immédiatement après qu’un utilisateur se connecte avec un fournisseur d’identité pour vérifier si l’utilisateur dispose d’une requête d’approbation existante ou s’il a déjà été refusé. Si votre système d’approbation n’effectue que des décisions d’approbation automatique, ce connecteur d’API n’est peut-être pas nécessaire. Exemple de connecteur d’API « Vérifier le statut d’approbation ».

  ![Vérifier la configuration du connecteur de l’API d’état d’approbation](./media/self-service-sign-up-add-approvals/check-approval-status-api-connector-config-alt.png)

- **Demander une approbation** : envoyer un appel au système d’approbation une fois qu’un utilisateur a terminé la page de collection d’attributs, mais avant la création du compte d’utilisateur, pour demander l’approbation. La requête d’approbation peut être automatiquement accordée ou révisée manuellement. Exemple de connecteur d’API « Demander une approbation ». 

  ![Configuration du connecteur de l’API d’approbation de requête](./media/self-service-sign-up-add-approvals/create-approval-request-api-connector-config-alt.png)

Pour créer ces connecteurs, suivez les étapes dans [créer un connecteur d’API](self-service-sign-up-add-api-connector.md#create-an-api-connector).

## <a name="enable-the-api-connectors-in-a-user-flow"></a>Activer les connecteurs d’API dans un workflow utilisateur

À présent, vous allez ajouter les connecteurs d’API à un workflow utilisateur d’inscription en libre-service en procédant comme suit :

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) en tant qu’administrateur Azure AD.
2. Sous **Services Azure**, sélectionnez **Azure Active Directory**.
3. Dans le menu de gauche, sélectionnez **Identités externes**.
4. Sélectionnez **flux d’utilisateurs (préversion)** , puis sélectionnez le flux d’utilisateurs pour lequel vous souhaitez activer le connecteur d’API.
5. Sélectionnez **Connecteurs d’API**, puis sélectionnez les points de terminaison d’API que vous souhaitez appeler aux étapes suivantes dans le workflow d’utilisateur :

   - **Après vous être connecté avec un fournisseur d’identité**: Sélectionnez votre connecteur d’API d’état d’approbation, par exemple _Vérifier l’état d’approbation_.
   - **Avant de créer l’utilisateur** : Sélectionnez votre connecteur d’API de requête d’approbation, par exemple _Approbation de demande_.

   ![Ajouter des API au workflow de l’utilisateur](./media/self-service-sign-up-add-approvals/api-connectors-user-flow-api.png)

6. Sélectionnez **Enregistrer**.

## <a name="control-the-sign-up-flow-with-api-responses"></a>Contrôler le flux d’inscription avec les réponses d’API

Votre système d’approbation peut utiliser ses réponses lorsqu’il est appelé pour contrôler le flux d’inscription. 

### <a name="request-and-responses-for-the-check-approval-status-api-connector"></a>Requête et réponses pour le connecteur d’API « Vérifier l’état d’approbation »

Exemple de requête reçue par l’API du connecteur d’API « Vérifier le statut d’approbation » :

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ //Sent for Google and Facebook identity providers
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "lastName":"Smith",
 "ui_locales":"en-US"
}
```

Les revendications exactes envoyées à l’API dépendent des informations fournies par le fournisseur d’identité. « email » est toujours envoyé.

#### <a name="continuation-response-for-check-approval-status"></a>Réponse de continuation pour « Vérifier l’état d’approbation »

Le point de terminaison de l’API **Vérifier l’état d’approbation** doit renvoyer une réponse de continuation si :

- L’utilisateur n’a pas demandé d’approbation précédemment.

Exemple de réponse de continuation :

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue"
}
```

#### <a name="blocking-response-for-check-approval-status"></a>Réponse de blocage pour « Vérifier l’état d’approbation »

Le point de terminaison de l’API **Vérifier l’état d’approbation** doit renvoyer une réponse de blocage si :

- L’approbation de l’utilisateur est en attente.
- L’utilisateur a été refusé et ne doit pas être autorisé à demander de nouveau l’approbation.

Voici quelques exemples de réponses de blocage :

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your access request is already processing. You'll be notified when your request has been approved.",
    "code": "CONTOSO-APPROVAL-PENDING"
}
```

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your sign up request has been denied. Please contact an administrator if you believe this is an error",
    "code": "CONTOSO-APPROVAL-DENIED"
}
```

### <a name="request-and-responses-for-the-request-approval-api-connector"></a>Requête et réponses pour le connecteur d’API « Demande d’approbation »

Exemple de requête HTTP reçue par l’API du connecteur d’API « Demander une approbation » :

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ //Sent for Google and Facebook identity providers
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "surname":"Smith",
 "jobTitle":"Supplier",
 "streetAddress":"1000 Microsoft Way",
 "city":"Seattle",
 "postalCode": "12345",
 "state":"Washington",
 "country":"United States",
 "extension_<extensions-app-id>_CustomAttribute1": "custom attribute value",
 "extension_<extensions-app-id>_CustomAttribute2": "custom attribute value",
 "ui_locales":"en-US"
}
```

Les revendications exactes envoyées à l’API dépendent des informations collectées par l’utilisateur ou fournies par le fournisseur d’identité.

#### <a name="continuation-response-for-request-approval"></a>Réponse de continuation pour « Requête d’approbation »

Le point de terminaison de l’API **Requête d’approbation** doit renvoyer une réponse de continuation si :

- L’utilisateur peut être **_approuvé automatiquement_**.

Exemple de réponse de continuation :

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue"
}
```

> [!IMPORTANT]
> Si une réponse de continuation est reçue, Azure AD crée un compte d’utilisateur et dirige l’utilisateur vers l’application.

#### <a name="blocking-response-for-request-approval"></a>Réponse de blocage pour « Requête d’approbation »

Le point de terminaison de l’API **Requête d’approbation** doit retourner une réponse de blocage si :

- Une requête d’approbation de l’utilisateur a été créée et est maintenant en attente.
- Une requête d’approbation de l’utilisateur a été automatiquement refusée.

Voici quelques exemples de réponses de blocage :

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your account is now waiting for approval. You'll be notified when your request has been approved.",
    "code": "CONTOSO-APPROVAL-REQUESTED"
}
```

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your sign up request has been denied. Please contact an administrator if you believe this is an error",
    "code": "CONTOSO-APPROVAL-AUTO-DENIED"
}
```

La `userMessage` dans la réponse est présentée à l’utilisateur, par exemple :

![Exemple de page d’approbation en attente](./media/self-service-sign-up-add-approvals/approval-pending.png)

## <a name="user-account-creation-after-manual-approval"></a>Création d’un compte d’utilisateur après une approbation manuelle

Après avoir obtenu une approbation manuelle, le système d’approbation personnalisé crée un compte d’[utilisateur](/graph/azuread-users-concept-overview) à l’aide de [Microsoft Graph](/graph/use-the-api). La façon dont votre système d’approbation approvisionne le compte d’utilisateur dépend du fournisseur d’identité utilisé par l’utilisateur.

### <a name="for-a-federated-google-or-facebook-user"></a>Pour un utilisateur Google ou Facebook fédéré

> [!IMPORTANT]
> Le système d’approbation doit vérifier explicitement que `identities`, `identities[0]` et `identities[0].issuer` sont présents et que `identities[0].issuer` est égal à « facebook » ou « google » pour utiliser cette méthode.

Si votre utilisateur s’est connecté avec un compte Google ou Facebook, vous pouvez utiliser l’[API de création d’utilisateur](/graph/api/user-post-users?tabs=http).

1. L’utilisateur du système d’approbation reçoit la requête HTTP du workflow de l’utilisateur.

```http
POST <Approvals-API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@outlook.com",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value",
 "ui_locales":"en-US"
}
```

2. Le système d’approbation utilise Microsoft Graph pour créer un compte d’utilisateur.

```http
POST https://graph.microsoft.com/v1.0/users
Content-type: application/json

{
 "userPrincipalName": "johnsmith_outlook.com#EXT@contoso.onmicrosoft.com",
 "accountEnabled": true,
 "mail": "johnsmith@outlook.com",
 "userType": "Guest",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value"
}
```

| Paramètre                                           | Obligatoire | Description                                                                                                                                                            |
| --------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| userPrincipalName                                   | Oui      | Peut être généré en acceptant la `email`revendication envoyée à l’API, en remplaçant le caractère `@` par `_`et en l’ajoutant à `#EXT@<tenant-name>.onmicrosoft.com`. |
| accountEnabled                                      | Oui      | Cette propriété doit être définie sur `true`.                                                                                                                                                 |
| mail                                                | Oui      | Équivaut à la `email`revendication envoyée à l’API.                                                                                                               |
| userType                                            | Oui      | Doit être `Guest`. Désigne cet utilisateur en tant qu’utilisateur invité.                                                                                                                 |
| Identités                                          | Oui      | Informations d’identité fédérée.                                                                                                                                    |
| \<otherBuiltInAttribute>                            | Non       | D’autres attributs intégrés tels que `displayName`, `city`et d’autres. Les noms de paramètres sont les mêmes que les paramètres envoyés par le connecteur d’API.                            |
| \<extension\_\{extensions-app-id}\_CustomAttribute> | Non       | Attributs personnalisés relatifs à l’utilisateur. Les noms de paramètres sont les mêmes que les paramètres envoyés par le connecteur d’API.                                                            |

### <a name="for-a-federated-azure-active-directory-user"></a>Pour un utilisateur Azure Active Directory fédéré

Si un utilisateur se connecte avec un compte Azure Active Directory fédéré, vous devez utiliser l’[API d’invitation](/graph/api/invitation-post) pour créer l’utilisateur, puis éventuellement l’[API de mise à jour de l’utilisateur](/graph/api/user-update) pour affecter plus d’attributs à l’utilisateur.

1. Le système d’approbation reçoit la requête HTTP du workflow de l’utilisateur.

```http
POST <Approvals-API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value",
 "ui_locales":"en-US"
}
```

2. Le système d’approbation crée l’invitation à l’aide de la `email` fournie par le connecteur d’API.

```http
POST https://graph.microsoft.com/v1.0/invitations
Content-type: application/json

{
    "invitedUserEmailAddress":"johnsmith@fabrikam.onmicrosoft.com",
    "inviteRedirectUrl" : "https://myapp.com"
}
```

Exemple de réponse :

```http
HTTP/1.1 201 OK
Content-type: application/json

{
    ...
    "invitedUser": {
        "id": "<generated-user-guid>"
    }
}
```

3. Le système d’approbation utilise l’ID de l’utilisateur invité pour mettre à jour le compte de l’utilisateur avec les attributs utilisateur collectés (facultatif).

```http
PATCH https://graph.microsoft.com/v1.0/users/<generated-user-guid>
Content-type: application/json

{
    "displayName": "John Smith",
    "city": "Redmond",
    "extension_<extensions-app-id>_AttributeName": "custom attribute value"
}
```

## <a name="next-steps"></a>Étapes suivantes

- Prise en main de nos [exemples de démarrage rapide d’Azure Function](code-samples-self-service-sign-up.md#api-connector-azure-function-quickstarts).
- Vérifiez l’[inscription en libre-service pour les utilisateurs invités avec l’exemple d’approbation manuelle](code-samples-self-service-sign-up.md#custom-approval-workflows).
