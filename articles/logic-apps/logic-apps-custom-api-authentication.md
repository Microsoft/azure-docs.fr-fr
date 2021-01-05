---
title: Ajouter l’authentification pour sécuriser les appels aux API personnalisées
description: Comment configurer l’authentification afin d’améliorer la sécurité des appels aux API personnalisées à partir d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 09/22/2017
ms.openlocfilehash: 5e0dcd478c6eb6696a0e07d35d4dccddac68ac1c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993183"
---
# <a name="increase-security-for-calls-to-custom-apis-from-azure-logic-apps"></a>Augmenter la sécurité des appels aux API personnalisées à partir d’Azure Logic Apps

Pour améliorer la sécurité des appels à vos API, vous pouvez configurer l’authentification Azure AD (Azure Active Directory) via le portail Azure afin d’éviter de devoir mettre à jour votre code. Vous pouvez également exiger et appliquer une authentification par le biais du code de votre API.

## <a name="authentication-options-for-your-api"></a>Options d’authentification de l’API

Vous pouvez améliorer la sécurité des appels à votre API personnalisée en appliquant les méthodes suivantes :

* [Aucune modification de code](#no-code) : protégez votre API avec [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md) par le biais du portail Azure, ce qui vous évite de mettre à jour votre code ou de redéployer votre API.

  > [!NOTE]
  > Par défaut, l’authentification Azure AD que vous activez dans le Portail Azure ne fournit pas une autorisation affinée. Par exemple, cette authentification verrouille votre API vis-à-vis d’un locataire spécifique et non d’un utilisateur ou d’une application spécifique. 

* [Mettre à jour le code de votre API](#update-code) : protégez votre API en appliquant [l’authentification par certificat](#certificate), [l’authentification de base](#basic) ou [l’authentification Azure AD](#azure-ad-code) par le biais du code.

<a name="no-code"></a>

### <a name="authenticate-calls-to-your-api-without-changing-code"></a>Authentifier les appels à votre API sans modifier le code

Voici les étapes générales de cette méthode :

1. Créez deux identités d’application Azure Active Directory (Azure AD) : une pour votre application logique et l’autre pour votre application web (ou application API).

2. Pour authentifier les appels à votre API, utilisez les informations d’identification (ID et clé secrète client) du principal de service associé à l’identité Azure AD de votre application logique.

3. Incluez les ID d’application dans votre définition d’application logique.

#### <a name="part-1-create-an-azure-ad-application-identity-for-your-logic-app"></a>Première partie : Créer une identité d’application Azure AD pour votre application logique

Votre application logique utilise cette identité d’application Azure AD pour s’authentifier auprès d’Azure AD. Vous n’avez besoin de configurer cette identité qu’une seule fois pour votre répertoire. Par exemple, vous pouvez choisir d’utiliser la même identité pour toutes vos applications logiques, même si vous pouvez créer des identités uniques pour chaque application logique. Vous pouvez configurer ces identités dans le portail Azure ou à l’aide de [PowerShell](#powershell).

**Créer l’identité d’application pour votre application logique dans le Portail Azure**

1. Dans le [Portail Azure](https://portal.azure.com "https://portal.azure.com"), choisissez **Azure Active Directory**. 

2. Vérifiez que vous vous trouvez dans le même répertoire que votre application web ou votre application API.

   > [!TIP]
   > Pour changer de répertoire, cliquez sur votre profil et sélectionnez un autre répertoire. Vous pouvez également sélectionner **Présentation** > **Changer de répertoire**.

3. Dans le menu du répertoire, sous **Gérer**, choisissez **Inscriptions des applications** > **Nouvelle inscription d’application**.

   > [!TIP]
   > Par défaut, la liste des inscriptions d’application affiche toutes les inscriptions d’application de votre répertoire. Pour afficher uniquement vos inscriptions d’application, sélectionnez **Mes applications** en regard de la zone de recherche. 

   ![Créer une inscription d’application](./media/logic-apps-custom-api-authentication/new-app-registration-azure-portal.png)

4. Donnez un nom à votre identité d’application, laissez le champ **Type d’application** défini sur **Application/API web**, indiquez une chaîne unique sous forme de domaine sous **URL de connexion**, puis cliquez sur **Créer**.

   ![Indication d’un nom et d’une URL de connexion pour l’identité de l’application](./media/logic-apps-custom-api-authentication/logic-app-identity-azure-portal.png)

   L’identité d’application que vous avez créée pour votre application logique s’affiche maintenant dans la liste des inscriptions d’application.

   ![Identité d’application pour votre application logique](./media/logic-apps-custom-api-authentication/logic-app-identity-created.png)

5. Dans la liste des inscriptions d’application, sélectionnez votre nouvelle identité d’application. Copiez et enregistrez **l’ID de l’application** à utiliser en tant qu’ID client pour votre application logique dans la partie 3.

   ![Copie et enregistrement de l’ID de l’application pour l’application logique](./media/logic-apps-custom-api-authentication/logic-app-application-id.png)

6. Si vos paramètres d’identité d’application ne sont pas visibles, choisissez **Paramètres** ou **Tous les paramètres**.

7. Sous **Accès d’API**, choisissez **Clés**. Sous **Description**, indiquez le nom de votre clé. Sous **Date d’expiration**, sélectionnez la durée de votre clé.

   La clé que vous créez joue le rôle de clé secrète ou de mot de passe de l’identité d’application pour votre application logique.

   ![Création d’une clé pour l’identité de l’application logique](./media/logic-apps-custom-api-authentication/create-logic-app-identity-key-secret-password.png)

8. Dans la barre d’outils, choisissez **Enregistrer**. Votre clé apparaît maintenant sous **Valeur**. 
**Veillez à copier et à enregistrer votre clé** en vue d’une utilisation ultérieure. En effet, elle est masquée dès lors que vous quittez la page **Clés**.

   Lorsque vous configurez votre application logique dans la partie 3, vous spécifiez cette clé comme clé secrète ou mot de passe.

   ![Copie et enregistrement de la clé pour une utilisation ultérieure](./media/logic-apps-custom-api-authentication/logic-app-copy-key-secret-password.png)

<a name="powershell"></a>

**Créer l’identité d’application pour votre application logique dans PowerShell**

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez effectuer cette tâche par le biais d’Azure Resource Manager avec PowerShell. Dans PowerShell, exécutez ces commandes :

1. `Add-AzAccount`

1. `$SecurePassword = Read-Host -AsSecureString`

1. Entrez un mot de passe et appuyez sur Entrée.

1. `New-AzADApplication -DisplayName "MyLogicAppID" -HomePage "http://mydomain.tld" -IdentifierUris "http://mydomain.tld" -Password $SecurePassword`

1. Veillez à copier **l’ID de locataire** (GUID pour votre locataire Azure AD), **l’ID d’application** et le mot de passe que vous avez utilisé.

Pour plus d’informations, consultez la page [Créer un principal du service pour accéder aux ressources à l’aide d’Azure PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md).

#### <a name="part-2-create-an-azure-ad-application-identity-for-your-web-app-or-api-app"></a>Deuxième partie : Créer une identité d’application Azure AD pour votre application web ou votre application API

Si votre application web ou votre application API est déjà déployée, vous pouvez activer l’authentification et créer l’identité de l’application dans le Portail Azure. Sinon, vous pouvez [activer l’authentification lorsque vous effectuez un déploiement avec un modèle Azure Resource Manager](#authen-deploy). 

**Créer l’identité d’application et activer l’authentification dans le Portail Azure pour les applications déployées**

1. Dans le [Portail Azure](https://portal.azure.com "https://portal.azure.com"), recherchez puis sélectionnez votre application web ou votre application API. 

2. Sous **Paramètres**, choisissez **Authentification/Autorisation**. Sous **Authentification App Service**, activez **l’authentification**. Sous **Fournisseurs d’authentification**, sélectionnez **Azure Active Directory**.

   ![Activer l’authentification](./media/logic-apps-custom-api-authentication/custom-web-api-app-authentication.png)

3. À présent, créez une identité d’application pour votre application web ou votre application API comme indiqué ici. Sur la page **Paramètres Azure Active Directory**, définissez le **Mode d’administration** sur **Express**. Choisissez **Créer une application AD**. Donnez un nom à votre identité d’application, puis cliquez sur **OK**. 

   ![Créer une identité d’application pour votre application web ou votre application API](./media/logic-apps-custom-api-authentication/custom-api-application-identity.png)

4. Sur la page **Authentification / Autorisation**, cliquez sur **Enregistrer**.

Vous devez à présent rechercher l’ID client et l’ID de locataire pour l’identité d’application associée à votre application web ou votre application API. Vous utiliserez ces ID dans la partie 3. Poursuivez cette procédure avec le portail Azure.

**Rechercher l’ID client et l’ID de locataire de l’identité de l’application pour votre application web ou votre application API dans le Portail Azure**

1. Sous **Fournisseurs d’authentification**, sélectionnez **Azure Active Directory**. 

   ![Sélectionner « Azure Active Directory »](./media/logic-apps-custom-api-authentication/custom-api-app-identity-client-id-tenant-id.png)

2. Sur la page **Paramètres Azure Active Directory**, définissez le **Mode d’administration** sur **Avancé**.

3. Copiez **l’ID client** et enregistrez ce GUID en vue de son utilisation dans la partie 3.

   > [!TIP] 
   > Si **l’ID client** et **l’URL de l’émetteur** ne s’affichent, essayez d’actualiser le Portail Azure et répétez l’étape 1.

4. Sous **URL de l’émetteur**, copiez et enregistrez simplement le GUID pour la partie 3. Vous pouvez également utiliser ce GUID dans le modèle de déploiement de votre application web ou de votre application API si nécessaire.

   Ce GUID est celui de votre locataire spécifique (« ID de locataire ») et doit apparaître dans cette URL :`https://sts.windows.net/{GUID}`

5. Sans enregistrer vos modifications, fermez la page **Paramètres Azure Active Directory**.

<a name="authen-deploy"></a>

**Activer l’authentification lorsque vous effectuez un déploiement avec un modèle Azure Resource Manager**

Vous devez toujours créer une identité d’application Azure AD pour votre application web ou votre application API, différente de l’identité d’application pour votre application logique. Pour créer l’identité d’application, suivez les étapes précédentes de la partie 2 avec le Portail Azure. 

Vous pouvez également suivre les étapes décrites dans la partie 1. Cependant, veillez à utiliser l’adresse `https://{URL}` réelle de votre application web ou de votre application API sous **URL de connexion** et **URI ID d’application**. À partir de ces étapes, vous devez enregistrer l’ID client et l’ID de locataire en vue de leur utilisation dans le modèle de déploiement de votre application. Vous les utiliserez également dans la partie 3.

> [!NOTE]
> Lorsque vous créez l’identité de l’application Azure AD pour votre application web ou votre application API, vous devez utiliser le portail Azure et pas PowerShell. L’applet de commande PowerShell ne configure pas les autorisations requises pour connecter les utilisateurs à un site web.

Une fois que vous disposez de l’ID client et de l’ID de locataire, incluez-les en tant que sous-ressource de votre application web ou de votre application API dans votre modèle de déploiement :

``` json
"resources": [ 
   {
      "apiVersion": "2015-08-01",
      "name": "web",
      "type": "config",
      "dependsOn": ["[concat('Microsoft.Web/sites/','parameters('webAppName'))]"],
      "properties": {
         "siteAuthEnabled": true,
         "siteAuthSettings": {
            "clientId": "<client-ID>",
            "issuer": "https://sts.windows.net/<tenant-ID>/"
         }
      }
   } 
]
```

Pour déployer automatiquement une application web et une application logique vides avec l’authentification Azure Active Directory, [consultez le modèle complet](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-custom-api/azuredeploy.json) ou cliquez sur **Déploiement sur Azure** ici :

[![Déployer sur Azure](media/logic-apps-custom-api-authentication/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-logic-app-custom-api%2Fazuredeploy.json)

#### <a name="part-3-populate-the-authorization-section-in-your-logic-app"></a>Troisième partie : Remplir la section Autorisation dans votre application logique

Cette section d’autorisation est déjà configurée dans le modèle précédent, mais si vous créez l’application logique directement, vous devez inclure la section d’autorisation complète.

Ouvrez votre définition d’application logique en mode code, accédez à la définition d’action **HTTP**, recherchez la section **Autorisation** et ajoutez ces propriétés :

```json
{
   "tenant": "<tenant-ID>",
   "audience": "<client-ID-from-Part-2-web-app-or-API app>", 
   "clientId": "<client-ID-from-Part-1-logic-app>",
   "secret": "<key-from-Part-1-logic-app>", 
   "type": "ActiveDirectoryOAuth"
}
```

| Propriété | Obligatoire | Description | 
| -------- | -------- | ----------- | 
| tenant | Oui | GUID du locataire Azure AD | 
| audience | Oui | GUID de la ressource cible à laquelle vous souhaitez accéder, c’est-à-dire l’ID client de l’identité de votre application web ou de votre application API | 
| clientId | Oui | GUID du client demandant l’accès, c’est-à-dire l’ID client de l’identité de votre application logique | 
| secret | Oui | Clé ou mot de passe de l’identité d’application pour le client qui demande le jeton d’accès | 
| type | Oui | Type d’authentification. Pour l’authentification ActiveDirectoryOAuth, la valeur est `ActiveDirectoryOAuth`. | 
|||| 

Par exemple :

``` json
{
   "actions": {
      "HTTP": {
         "inputs": {
            "method": "POST",
            "uri": "https://your-api-azurewebsites.net/api/your-method",
            "authentication": {
               "tenant": "tenant-ID",
               "audience": "client-ID-from-azure-ad-app-for-web-app-or-api-app",
               "clientId": "client-ID-from-azure-ad-app-for-logic-app",
               "secret": "key-from-azure-ad-app-for-logic-app",
               "type": "ActiveDirectoryOAuth"
            }
         }
      }
   }
}
```

<a name="update-code"></a>

### <a name="secure-api-calls-through-code"></a>Sécuriser les appels à l’API avec le code

<a name="certificate"></a>

#### <a name="certificate-authentication"></a>Authentification par certificat

Vous pouvez utiliser des certificats clients pour valider les demandes entrantes de votre application logique parvenant à votre application web ou votre application API. Pour configurer votre code, consultez [Configuration de l’authentification mutuelle TLS pour une application web](../app-service/app-service-web-configure-tls-mutual-auth.md).

Dans la section **Autorisation**, ajoutez ces propriétés :

```json
{
   "type": "ClientCertificate",
   "password": "<password>",
   "pfx": "<long-pfx-key>"
} 
```

| Propriété | Obligatoire | Description |
| -------- | -------- | ----------- |
| `type` | Oui | Type d’authentification. Pour les certificats client TLS/SSL, la valeur doit être `ClientCertificate`. |
| `password` | Non | Mot de passe pour l’accès au certificat client (fichier PFX) |
| `pfx` | Oui | Contenu codé base 64 du certificat client (fichier PFX) |
||||

<a name="basic"></a>

#### <a name="basic-authentication"></a>Authentification de base

Pour valider les demandes entrantes de votre application logique à l’attention de votre application web ou de votre application API, vous pouvez utiliser l’authentification de base, par exemple, un nom d’utilisateur et un mot de passe. L’authentification de base est une méthode courante que vous pouvez utiliser dans n’importe quel langage utilisé pour générer votre application web ou votre application API.

Dans la section **Autorisation**, ajoutez ces propriétés :

```json
{
   "type": "Basic",
   "username": "<username>",
   "password": "<password>"
}
```

| Propriété | Obligatoire | Description | 
| -------- | -------- | ----------- | 
| type | Oui | Le type d’authentification que vous souhaitez utiliser. Pour l’authentification de base, la valeur doit être `Basic`. | 
| username | Oui | Le nom d’utilisateur que vous souhaitez utiliser pour l’authentification. | 
| mot de passe | Oui | Le mot de passe que vous souhaitez utiliser pour l’authentification. | 
|||| 

<a name="azure-ad-code"></a>

#### <a name="azure-active-directory-authentication-through-code"></a>Authentification Azure Active Directory avec le code

Par défaut, l’authentification Azure AD que vous activez dans le Portail Azure ne fournit pas une autorisation affinée. Par exemple, cette authentification verrouille votre API vis-à-vis d’un locataire spécifique et non d’un utilisateur ou d’une application spécifique. 

Pour restreindre l’accès des API à votre application logique à l’aide du code, extrayez l’en-tête contenant le jeton JWT (JSON Web Token). Vérifiez l’identité de l’appelant et rejetez les demandes qui ne correspondent pas.

<!-- Going further, to implement this authentication entirely in your own code, 
and not use the Azure portal, learn how to 
[authenticate with on-premises Active Directory in your Azure app](../app-service/overview-authentication-authorization.md).

To create an application identity for your logic app and use that identity to call your API, 
you must follow the previous steps. -->

## <a name="next-steps"></a>Étapes suivantes

* [Déployer et appeler des API personnalisées à partir de workflows d’application logique](../logic-apps/logic-apps-custom-api-host-deploy-call.md)
