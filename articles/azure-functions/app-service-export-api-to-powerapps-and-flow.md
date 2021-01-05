---
title: Exportation d’une API hébergée sur Azure vers PowerApps et Microsoft Flow
description: Présentation de l’exposition d’une API hébergée dans App Service vers PowerApps et Microsoft Flow
ms.topic: conceptual
ms.date: 04/28/2020
ms.reviewer: sunayv
ms.openlocfilehash: 359f19598d5f1dff6cda4b303d33ed721ed72d97
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92054795"
---
# <a name="exporting-an-azure-hosted-api-to-powerapps-and-microsoft-flow"></a>Exportation d’une API hébergée sur Azure vers PowerApps et Microsoft Flow

[PowerApps](https://powerapps.microsoft.com/guided-learning/learning-introducing-powerapps/) est un service destiné à la création et à l’utilisation d’applications métier personnalisées qui se connectent à vos données et fonctionnent sur plusieurs plateformes. [Power Automate](/learn/modules/get-started-with-flow/index) facilite l’automatisation des workflows et des processus métier entre vos services et applications préférés. Divers connecteurs intégrés aux sources de données tels que Office 365, Dynamics 365, Salesforce, etc. sont fournis avec PowerApps et Microsoft Flow. Dans certains cas, les générateurs d’applications et de flux souhaitent également se connecter à des sources de données et à des API générées par leur organisation.

De même, les développeurs qui souhaitent exposer leurs API plus largement au sein d’une organisation peuvent rendre leurs API disponibles pour les générateurs d’applications et de flux. Cet article vous montre comment exporter une API générée avec [Azure Functions](../azure-functions/functions-overview.md) ou [Azure App Service](../app-service/overview.md). L’API exportée devient un *connecteur personnalisé*, qui est utilisé dans PowerApps et Microsoft Flow comme un connecteur intégré.

> [!IMPORTANT]
> La fonctionnalité de définition d’API présentée dans cet article est uniquement prise en charge pour la [version 1.x du runtime Azure Functions](functions-versions.md#creating-1x-apps) et les applications App Services. La version 2.x de Functions s’intègre à Gestion des API pour créer et tenir à jour des définitions OpenAPI. Pour en savoir plus, consultez [Créer une définition OpenAPI pour une fonction avec Gestion des API Azure](functions-openapi-definition.md). 

## <a name="create-and-export-an-api-definition"></a>Créer et exporter une définition d’API
Avant d’exporter une API, vous devez décrire l’API en utilisant une définition OpenAPI (anciennement appelée un fichier [Swagger](https://swagger.io/)). Cette définition contient des informations sur les opérations qui sont disponibles dans une API et sur la façon dont les données de demande et de réponse de l’API doivent être structurées. PowerApps et Microsoft Flow peuvent créer des connecteurs personnalisés pour toute définition OpenAPI 2.0. Azure Functions et Azure App Service prennent en charge la création, l’hébergement et la gestion des définitions OpenAPI. Pour plus d’informations, consultez [Héberger une API RESTful avec CORS dans Azure App Service](../app-service/app-service-web-tutorial-rest-api.md).

> [!NOTE]
> Vous pouvez également créer des connecteurs personnalisés dans l’interface utilisateur de PowerApps et de Microsoft Flow, sans utiliser de définition OpenAPI. Pour plus d’informations, consultez [S’inscrire et utiliser un connecteur personnalisé (PowerApps)](https://powerapps.microsoft.com/tutorials/register-custom-api/) et [S’inscrire et utiliser un connecteur personnalisé (Microsoft Flow)](/power-automate/developer/register-custom-api).

Pour exporter la définition de l’API, effectuez les opérations suivantes :

1. Dans le [Portail Azure](https://portal.azure.com), accédez à votre application de fonction ou à une application App Service.

    Dans le menu de gauche, sous **API**, sélectionnez **Définition de l’API**.

    :::image type="content" source="media/app-service-export-api-to-powerapps-and-flow/api-definition-function.png" alt-text="Définition de l’API Azure Functions":::

2. Le bouton **Exporter vers PowerApps + Microsoft Flow** doit être disponible (si ce n’est pas le cas, vous devez d’abord créer une définition OpenAPI). Sélectionnez ce bouton pour lancer le processus d’exportation.

    ![Bouton Exporter vers PowerApps + Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-apps-flow.png)

3. Sélectionnez le **Mode d’exportation** :

    **Express** : vous pouvez créer le connecteur personnalisé à partir du Portail Azure. Vous êtes censé être connecté à PowerApps ou Microsoft Flow et être autorisé à créer des connecteurs dans l’environnement cible. Cette approche est recommandée si ces deux exigences peuvent être satisfaites. Si vous utilisez ce mode, suivez les instructions de la section [Utiliser le mode d’exportation express](#express) plus bas.

    **Manuel** : vous pouvez exporter la définition de l’API, puis l’importer à l’aide du portail PowerApps ou Microsoft Flow. Cette approche est recommandée si l’utilisateur Azure et celui autorisé à créer des connecteurs sont des personnes différentes ou si le connecteur doit être créé dans un autre locataire Azure. Si vous utilisez ce mode, suivez les instructions de la section [Utiliser le mode d’exportation manuel](#manual) plus bas.

    ![Mode d’exportation](media/app-service-export-api-to-powerapps-and-flow/export-mode.png)

> [!NOTE]
> Comme le connecteur personnalisé utilise une *copie* de la définition d’API, PowerApps et Microsoft Flow ne savent pas immédiatement si vous apportez des modifications à l’application et à sa définition d’API. Si vous apportez des modifications, répétez les étapes d’exportation pour la nouvelle version.

<a name="express"></a>
## <a name="use-express-export"></a>Utiliser le mode d’exportation express

Pour effectuer l’exportation en mode **Express**, suivez les étapes ci-après :

1. Vérifiez que vous êtes connecté au locataire PowerApps ou Microsoft Flow vers lequel vous souhaitez effectuer l’opération d’exportation. 

2. Utilisez les paramètres spécifiés dans le tableau.

    |Paramètre|Description|
    |--------|------------|
    |**Environment**|Sélectionnez l’environnement dans lequel le connecteur personnalisé doit être enregistré. Pour plus d’informations, consultez [Présentation des environnements](https://powerapps.microsoft.com/tutorials/environments-overview/).|
    |**Nom de l’API personnalisée**|Entrez un nom, que les générateurs d’applications PowerApps et de flux Microsoft Flow verront dans leur liste de connecteurs.|
    |**Préparer la configuration de la sécurité**|Si nécessaire, fournissez les informations de configuration de sécurité nécessaires pour accorder aux utilisateurs l’accès à votre API. Cet exemple montre une clé API. Pour plus d’informations, consultez [Spécification du type d’authentification](#auth) ci-dessous.|
 
    ![Exporter en mode Express vers PowerApps et Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-express.png)

3. Cliquez sur **OK**. Le connecteur personnalisé est maintenant créé et ajouté à l’environnement que vous avez spécifié.

<a name="manual"></a>
## <a name="use-manual-export"></a>Utiliser le mode d’exportation manuel

Pour effectuer l’exportation en mode **Manuel**, suivez les étapes ci-après :

1. Cliquez sur **Télécharger** et enregistrez le fichier, ou cliquez sur le bouton Copier et enregistrez l’URL. Vous allez utiliser le fichier de téléchargement ou l’URL pendant l’importation.
 
    ![Exporter en mode manuel vers PowerApps et Microsoft Flow](media/app-service-export-api-to-powerapps-and-flow/export-manual.png)
 
2. Si votre définition de l’API comprend des définitions de sécurité, celles-ci sont exigées à l’étape n° 2. Pendant l’importation, PowerApps et Microsoft Flow détectent ces définitions et demandent les informations de sécurité. Collectez les informations d’identification associées à chaque définition pour les utiliser dans la section suivante. Pour plus d’informations, consultez [Spécification du type d’authentification](#auth) ci-dessous.

    ![Sécurité de l’exportation en mode manuel](media/app-service-export-api-to-powerapps-and-flow/export-manual-security.png)

    Cet exemple montre que la définition de la sécurité de la clé API a été incluse dans la définition OpenAPI.

La définition de l’API étant exportée, vous l’importez pour créer un connecteur personnalisé dans PowerApps et Microsoft Flow. Les connecteurs personnalisés étant partagés entre les deux services, vous devez uniquement importer la définition une seule fois.

Pour importer la définition de l’API dans PowerApps et Microsoft Flow, suivez les étapes ci-après :

1. Accédez à [powerapps.com](https://web.powerapps.com) ou [flow.microsoft.com](https://flow.microsoft.com).

2. Dans le coin supérieur droit, cliquez sur l’icône d’engrenage, puis sur **Connecteurs personnalisés**.

   ![Icône d’engrenage en service](media/app-service-export-api-to-powerapps-and-flow/icon-gear.png)

3. Cliquez sur **Créer un connecteur personnalisé**, puis sur **Importer une définition OpenAPI**.

   ![Créer un connecteur personnalisé](media/app-service-export-api-to-powerapps-and-flow/flow-apps-create-connector.png)

4. Entrez un nom pour le connecteur personnalisé, accédez à la définition OpenAPI que vous avez exportée, puis cliquez sur **Continuer**.

   ![Charger la définition OpenAPI](media/app-service-export-api-to-powerapps-and-flow/flow-apps-upload-definition.png)

4. Sous l’onglet **Général**, vérifiez les informations provenant de la définition OpenAPI.

5. Sous l’onglet **Sécurité**, si vous êtes invité à fournir des informations d’authentification, entrez les valeurs appropriées pour le type d’authentification. Cliquez sur **Continuer**.

    ![Onglet Sécurité](media/app-service-export-api-to-powerapps-and-flow/tab-security.png)

    Cet exemple montre les champs obligatoires de l’authentification des clés API. Les champs diffèrent selon le type d’authentification.

6. Dans l’onglet **Définitions**, toutes les opérations définies dans votre fichier OpenAPI sont remplies automatiquement. Si toutes vos opérations requises sont définies, vous pouvez accéder à l’étape suivante. Si ce n’est pas le cas, vous pouvez ajouter et modifier les opérations ici.

    ![Onglet Définitions](media/app-service-export-api-to-powerapps-and-flow/tab-definitions.png)

    Cet exemple comprend une seule opération, nommée `CalculateCosts`. Les métadonnées, telles que la **Description**, proviennent toutes du fichier OpenAPI.

7. Cliquez sur **Créer un connecteur** en haut de la page.

Vous pouvez désormais vous connecter au connecteur personnalisé dans PowerApps et Microsoft Flow. Pour plus d’informations sur la création de connecteurs dans les portails PowerApps et Microsoft Flow, consultez [Inscrire votre connecteur personnalisé (PowerApps)](https://powerapps.microsoft.com/tutorials/register-custom-api/#register-your-custom-connector) et [Inscrire votre connecteur personnalisé (Microsoft Flow)](/power-automate/get-started-flow-dev#create-a-custom-connector).

<a name="auth"></a>
## <a name="specify-authentication-type"></a>Spécification du type d’authentification

PowerApps et Microsoft Flow prennent en charge une collection de fournisseurs d’identité qui fournissent l’authentification pour les connecteurs personnalisés. Si votre API nécessite une authentification, assurez-vous qu’elle est capturée en tant que _définition de sécurité_ dans votre document OpenAPI, comme dans l’exemple suivant :

```json
"securityDefinitions": {
    "AAD": {
    "type": "oauth2",
    "flow": "accessCode",
    "authorizationUrl": "https://login.windows.net/common/oauth2/authorize",
    "scopes": {}
    }
}
``` 
Pendant l’exportation, vous fournissez des valeurs de configuration permettant à PowerApps et Microsoft Flow d’authentifier les utilisateurs.

Cette section décrit les types d’authentification qui sont pris en charge par le mode **Express** : clé API, Azure Active Directory et Generic OAuth 2.0. PowerApps et Microsoft Flow prennent également en charge l’authentification de base et OAuth 2.0 pour des services spécifiques tels que Dropbox, Facebook et SalesForce.

### <a name="api-key"></a>Clé API
Si vous utilisez une clé API, les utilisateurs de votre connecteur sont invités à fournir la clé quand ils créent une connexion. Vous spécifiez un nom de clé API pour les aider à comprendre quelle clé est requise. Dans l’exemple précédent, nous utilisons le nom `API Key (contact meganb@contoso.com)` afin que les utilisateurs sachent où obtenir plus d’informations sur la clé API. Pour Azure Functions, la clé est généralement l’une des clés d’hôte, couvrant plusieurs fonctions au sein de l’application de fonction.

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)
Quand vous utilisez Azure AD, vous avez besoin de deux inscriptions d’application Azure AD : une pour l’API elle-même, l’autre pour le connecteur personnalisé :

- Pour configurer l’inscription de l’API, utilisez la fonctionnalité [Autorisation/Authentification App Service](../app-service/configure-authentication-provider-aad.md).

- Pour configurer l’inscription pour le connecteur, suivez les étapes de la section [Ajout d’une application Azure AD](../active-directory/develop/quickstart-register-app.md). L’inscription doit disposer d’un accès délégué à votre API et de l’URL de réponse `https://msmanaged-na.consent.azure-apim.net/redirect`. 

Pour plus d’informations, consultez les exemples d’inscription Azure AD pour [PowerApps](https://powerapps.microsoft.com/tutorials/customapi-azure-resource-manager-tutorial/) et [Microsoft Flow](/connectors/custom-connectors/azure-active-directory-authentication). Ces exemples utilisent Azure Resource Manager comme API ; substituez votre API si vous suivez les étapes.

Les valeurs de configuration suivantes sont requises :
- **ID client** : ID client de l’inscription Azure AD de votre connecteur
- **Clé secrète client** : clé secrète client de l’inscription Azure AD de votre connecteur
- **URL de connexion** : URL de base pour Azure AD. Dans Azure, généralement `https://login.windows.net`.
- **ID de locataire** : l’ID du locataire à utiliser pour la connexion. Cet ID doit être « commun » ou l’ID du locataire dans lequel le connecteur est créé.
- **URL de la ressource** : URL de ressource de l’inscription Azure AD pour votre API

> [!IMPORTANT]
> Si une autre personne importe la définition de l’API dans PowerApps et Microsoft Flow dans le cadre du flux manuel, vous devez lui fournir l’ID client et le secret client de *l’inscription du connecteur*, ainsi que l’URL de ressource de votre API. Assurez-vous que ces informations secrètes sont gérées de façon sécurisée. **Ne partagez pas les informations d’identification de sécurité de l’API.**

### <a name="generic-oauth-20"></a>Generic OAuth 2.0
Si vous utilisez Generic OAuth 2.0, vous pouvez effectuer une intégration à n’importe quel fournisseur OAuth 2.0. Cela vous permet d’utiliser des fournisseurs personnalisés qui ne sont pas pris en charge en mode natif.

Les valeurs de configuration suivantes sont requises :
- **ID client** : l’ID client OAuth 2.0
- **Clé secrète client** : la clé secrète client OAuth 2.0
- **URL d’autorisation** : l’URL d’autorisation OAuth 2.0
- **URL du jeton** : l’URL du jeton OAuth 2.0
- **Actualiser l’URL** : l’URL d’actualisation OAuth 2.0
