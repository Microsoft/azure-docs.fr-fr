---
title: Guide pratique pour sécuriser une application démon
titleSuffix: Azure Maps
description: Utilisez le portail Azure pour gérer l’authentification afin de configurer une application démon approuvée.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 2dd04f404330a6c86e2df09da610e16ba9b721f3
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895645"
---
# <a name="secure-a-daemon-application"></a>Sécuriser une application démon

Le guide suivant concerne les processus en arrière-plan, les minuteurs et les travaux qui sont hébergés dans un environnement approuvé et sécurisé, par exemple les travaux web Azure, les applications de fonction Azure, les services Windows et tout autre service en arrière-plan fiable.

> [!Tip]
> Microsoft recommande d’implémenter Azure Active Directory (Azure AD) et le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour les applications de production. Pour obtenir une vue d’ensemble des concepts, consultez [Authentification Azure Maps](./azure-maps-authentication.md).

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="scenario-shared-key-authentication"></a>Scénario : Authentification par clé partagée

Après la création du compte Azure Maps, les clés primaires et secondaires sont générées. Nous vous recommandons d’utiliser la clé primaire comme clé d’abonnement quand vous [utilisez l’authentification par clé partagée pour appeler Azure Maps](./azure-maps-authentication.md#shared-key-authentication). Vous pouvez utiliser une clé secondaire dans des scénarios comme des changements dans la permutation des clés. Pour plus d’informations, consultez [Authentification dans Azure Maps](./azure-maps-authentication.md).

### <a name="securely-store-shared-key"></a>Stocker la clé partagée en toute sécurité

La clé primaire et la clé secondaire autorisent l’accès à toutes les API pour le compte Maps. Les applications doivent stocker les clés dans un magasin sécurisé, par exemple Azure Key Vault. L’application doit récupérer la clé partagée en tant que secret Azure Key Vault pour éviter de la stocker en texte brut dans la configuration de l’application. Pour comprendre comment configurer un coffre de clés Azure, consultez le [Guide du développeur Azure Key Vault](../key-vault/general/developers-guide.md).

Les étapes suivantes décrivent ce processus :

1. Créez un coffre de clés Azure.
2. Créez un principal du service Azure AD en créant une inscription d’application ou une identité managée ; le principal créé est responsable de l’accès à Azure Key Vault.
3. Accordez au principal du service l’autorisation d’accès `Get` aux secrets de clé Azure.
4. Attribuez-vous temporairement (en tant que développeur) l’autorisation d’accès `Set` aux secrets.
5. Définissez la clé partagée dans les secrets Key Vault, référencez l’ID de secret en tant que configuration pour l’application démon, puis supprimez votre autorisation `Set` sur les secrets.
6. Implémentez l’authentification Azure AD dans l’application démon pour récupérer le secret de clé partagée à partir d’Azure Key Vault.
7. Créez une requête d’API REST Azure Maps avec clé partagée.

> [!Tip]
> Si l’application est hébergée dans un environnement Azure, vous devez implémenter une identité managée afin de réduire le coût et la complexité liés à la gestion d’un secret pour l’authentification auprès d’Azure Key Vault. Consultez le [tutoriel Azure Key Vault suivant pour vous connecter par le biais d’une identité managée](../key-vault/general/tutorial-net-create-vault-azure-web-app.md).

L’application démon est responsable de la récupération de la clé partagée à partir d’un stockage sécurisé. L’implémentation avec Azure Key Vault nécessite une authentification par le biais d’Azure AD pour accéder au secret. Au lieu de cela, nous privilégions l’authentification Azure AD directe auprès d’Azure Maps, en raison de la complexité et des exigences opérationnelles supplémentaires liées à l’utilisation de l’authentification par clé partagée.

> [!IMPORTANT]
> Pour simplifier la regénération des clés, nous recommandons de faire en sorte que les applications utilisent une seule clé à la fois. Elles peuvent alors regénérer la clé inutilisée et déployer la nouvelle clé regénérée dans un magasin de secrets sécurisé, tel qu’Azure Key Vault.

## <a name="scenario-azure-ad-role-based-access-control"></a>Scénario : Contrôle d’accès en fonction du rôle Azure AD

Une fois qu’un compte Azure Maps a été créé, la valeur `x-ms-client-id` Azure Maps est présente dans la page des détails d’authentification du portail Azure. Cette valeur représente le compte qui sera utilisé pour les requêtes d’API REST. Elle doit être stockée dans la configuration de l’application et récupérée avant d’effectuer des requêtes HTTP. L’objectif du scénario est de permettre à l’application démon de s’authentifier auprès d’Azure AD et d’appeler les API REST Azure Maps.

> [!Tip]
> Nous recommandons l’hébergement sur Machines virtuelles Microsoft Azure, Microsoft Azure Virtual Machine Scale Sets ou App Services pour bénéficier des avantages offerts par les composants Managed Identity.

### <a name="daemon-hosted-on-azure-resources"></a>Démon hébergé sur des ressources Azure

En cas d’exécution sur des ressources Azure, configurez des identités managées Azure pour bénéficier d’un effort de gestion des informations d’identification minimal et à moindre coût. 

Pour permettre à l’application d’accéder à une identité managée, Consultez [Vue d’ensemble des identités managées](../active-directory/managed-identities-azure-resources/overview.md).

Avantages des identités managées :

* Authentification de chiffrement à clé publique par certificat X509 managée par le système Azure
* Sécurité Azure AD avec certificats X509 au lieu de clés secrètes client
* Azure gère et renouvelle tous les certificats associés à la ressource Managed Identity
* Simplification de la gestion opérationnelle des informations d’identification, car aucun service de magasin de secrets sécurisé tel qu’Azure Key Vault n’est plus nécessaire 

### <a name="daemon-hosted-on-non-azure-resources"></a>Démon hébergé sur des ressources non-Azure

En cas d’exécution sur un environnement autre qu’Azure, les identités managées ne sont pas disponibles. Vous devez par conséquent configurer un principal du service par le biais d’une inscription d’application Azure AD pour l’application démon.

1. Dans le portail Azure, dans la liste des services Azure, sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription** .  

    > [!div class="mx-imgBorder"]
    > ![Inscription d’application](./media/how-to-manage-authentication/app-registration.png)

2. Si vous avez déjà inscrit votre application, passez à l’étape suivante. Si vous n’avez pas inscrit votre application, entrez un **Nom** , choisissez un **Type de compte de support** , puis sélectionnez **Inscrire** .  

    > [!div class="mx-imgBorder"]
    > ![Détails de l’inscription d’application](./media/how-to-manage-authentication/app-create.png)

3. Pour affecter des autorisations d’API déléguées à Azure Maps, accédez à l’application. Ensuite, sous **Inscriptions d’applications** , sélectionnez **Autorisations d’API** > **Ajouter une autorisation** . Sous **API utilisées par que mon organisation** , recherchez et sélectionnez **Azure Maps** .

    > [!div class="mx-imgBorder"]
    > ![Ajouter des autorisations d’API d’application](./media/how-to-manage-authentication/app-permissions.png)

4. Cochez la case en regard de **Accéder à Azure Maps** , puis sélectionnez **Ajouter des autorisations** .

    > [!div class="mx-imgBorder"]
    > ![Sélectionner des autorisations d’API d’application](./media/how-to-manage-authentication/select-app-permissions.png)

5. Effectuez les étapes suivantes pour créer une clé secrète client ou configurer un certificat.

    * Si votre application utilise l’authentification du serveur ou de l’application, dans la page d’inscription de votre application, accédez à **Certificats et secrets** . Ensuite, chargez un certificat de clé publique ou créez un mot de passe en sélectionnant **Nouveau secret client** .

        > [!div class="mx-imgBorder"]
        > ![Créer une clé secrète client](./media/how-to-manage-authentication/app-keys.png)

    * Après avoir sélectionné **Ajouter** , copiez le secret et stockez-le de manière sécurisée dans un service tel qu’Azure Key Vault. Pour stocker le certificat ou le secret de manière sécurisée, consultez le [Guide du développeur Azure Key Vault](../key-vault/general/developers-guide.md). Vous utiliserez ce secret pour obtenir des jetons auprès d’Azure AD.

        > [!div class="mx-imgBorder"]
        > ![Ajouter une clé secrète client](./media/how-to-manage-authentication/add-key.png)

### <a name="grant-role-based-access-for-the-daemon-application-to-azure-maps"></a>Accorder l’accès en fonction du rôle pour l’application démon à Azure Maps

Pour accorder le *contrôle d’accès en fonction du rôle Azure (Azure RBAC)* , vous devez attribuer l’identité managée créée ou le principal du service à une ou plusieurs définitions de rôle Azure Maps. Pour afficher les définitions de rôle Azure disponibles pour Azure Maps, accédez à **Contrôle d’accès (IAM)** . Sélectionnez **Rôles** , puis recherchez les rôles qui commencent par *Azure Maps* . Ces rôles Azure Maps sont les rôles auxquels vous pouvez accorder un accès.

> [!div class="mx-imgBorder"]
> ![Afficher les rôles disponibles](./media/how-to-manage-authentication/how-to-view-avail-roles.png)

1. Accédez à votre **compte Azure Maps** . Sélectionnez **Contrôle d’accès (IAM)**  > **Attributions de rôles** .

    > [!div class="mx-imgBorder"]
    > ![Accorder l’accès à l’aide d’Azure RBAC](./media/how-to-manage-authentication/how-to-grant-rbac.png)

2. Sous l’onglet **Attributions de rôles** , **ajoutez** une attribution de rôle. 

    > [!div class="mx-imgBorder"]
    > ![Screenshot shows the roll assignments with Add selected.](./media/how-to-manage-authentication/add-role-assignment.png)

3. Sélectionnez une définition de rôle Azure Maps intégrée, comme **Lecteur de données Azure Maps** ou **Contributeur aux données Azure Maps** . Sous **Attribuer l’accès à** , sélectionnez **Utilisateur, groupe ou principal de service Azure AD** ou Managed Identity avec **Identité managée affectée par l’utilisateur** / **Identité managée affectée par le système** . Sélectionnez le principal. Ensuite, sélectionnez **Enregistrer** .

    > [!div class="mx-imgBorder"]
    > ![How to add role assignment](./media/how-to-manage-authentication/how-to-add-role-assignment.png)

4. Vous pouvez vérifier que l’attribution de rôle a été appliquée sous l’onglet Attribution de rôle.

## <a name="request-token-with-managed-identity"></a>Demander un jeton avec Managed Identity

Une fois que vous avez configuré une identité managée pour la ressource d’hébergement, utilisez le SDK Azure ou l’API REST pour obtenir un jeton pour Azure Maps. Pour plus d’informations, consultez [Obtenir un jeton d’accès](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md). Si vous avez bien suivi les instructions, un jeton d’accès pouvant être utilisé sur les requêtes d’API REST devrait être retourné.

## <a name="request-token-with-application-registration"></a>Demander un jeton avec l’inscription d’application

Une fois que votre application est inscrite et associée à Azure Maps, vous pouvez demander des jetons d’accès.

* ID de ressource Azure AD `https://atlas.microsoft.com/`
* ID de l’application Azure AD
* ID de locataire Azure AD
* Clé secrète client d’inscription d’application Azure AD

Demande :

```http
POST /<Azure AD Tenant ID>/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=<Azure AD App ID>&resource=https://atlas.microsoft.com/&client_secret=<client secret>&grant_type=client_credentials
```

Réponse :

```json
{
    "token_type": "Bearer",
    "expires_in": "...",
    "ext_expires_in": "...",
    "expires_on": "...",
    "not_before": "...",
    "resource": "https://atlas.microsoft.com/",
    "access_token": "ey...gw"
}
```

Pour obtenir des exemples plus détaillés, consultez [Scénarios d’authentification pour Azure AD](../active-directory/develop/authentication-vs-authorization.md).

## <a name="next-steps"></a>Étapes suivantes

Recherchez les métriques d’utilisation de l’API pour votre compte Azure Maps :
> [!div class="nextstepaction"]
> [Afficher les métriques d’utilisation](how-to-view-api-usage.md)

Explorez des exemples qui montrent comment intégrer Azure AD à Azure Maps :
> [!div class="nextstepaction"]
> [Exemples Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples)