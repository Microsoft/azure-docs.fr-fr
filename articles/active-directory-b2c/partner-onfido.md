---
title: Tutoriel pour configurer Azure Active Directory B2C avec Onfido
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer l’authentification Azure AD B2C avec Onfido pour l’ID de document et la vérification de la biométrie faciale.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/03/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 46c6eac80ddbff73d99e05c070e66aa1700da174
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928628"
---
# <a name="tutorial-for-configuring-onfido-with-azure-active-directory-b2c"></a>Tutoriel de configuration d’Onfido avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous fournissons des conseils sur l’intégration d’Azure AD B2C avec [Onfido](https://onfido.com/). Onfido est un ID de document et une application de vérification de la biométrie faciale. Elle permet aux entreprises de répondre aux exigences de *connaître de la clientèle* et d’identité en temps réel. Onfido utilise une vérification d’identité basée sur l’intelligence artificielle, qui vérifie d’abord l’ID d’une photo, puis le compare à sa biométrie faciale. Cette solution lie une identité numérique à la personne réelle correspondante et fournit une expérience d’intégration sûre tout en réduisant la fraude.

Dans cet exemple, nous connectons le service d’Onfido dans le processus d’inscription ou de connexion pour effectuer la vérification de l’identité. Des décisions éclairées concernant le produit et le service auxquels l’utilisateur peut accéder sont prises en fonction des résultats d’Onfido.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md) lié à votre abonnement Azure.

- Un [compte d’essai](https://onfido.com/signup/) Onfido.

## <a name="scenario-description"></a>Description du scénario

L’intégration d’Onfido inclut les composants suivants :

- Locataire Azure AD B2C : serveur d’autorisation chargé de vérifier les informations d’identification de l’utilisateur en fonction de stratégies personnalisées définies dans le locataire. Également connu sous le nom de fournisseur d’identité. Il héberge l’application cliente Onfido qui collecte les documents utilisateur et les transmet au service de l’API Onfido.

- Client Onfido : utilitaire de collecte de documents de client JavaScript configurable déployé dans d’autres pages web. Collecte les documents et effectue des vérifications préliminaires comme la taille et la qualité des documents.

- API Rest intermédiaire : fournit des points de terminaison pour permettre au locataire Azure AD B2C de communique avec le service d’API Onfido, gérer le traitement des données et adhérer aux exigences de sécurité des deux.

- Service d’API Onfido : service principal fourni par Onfido, qui enregistre et vérifie les documents fournis par l’utilisateur.

Le diagramme d’architecture suivant illustre l’implémentation.

![Capture d’écran du diagramme d’architecture d’Onfido](media/partner-onfido/onfido-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | L’utilisateur accède à une page de connexion. L’utilisateur s’inscrit pour créer pour créer un compte et entre des informations dans la page. Azure AD B2C collecte les attributs de l’utilisateur. L’application cliente Onfido hébergée dans Azure AD B2C effectue des vérifications préliminaires des informations utilisateur.
| 2. | Azure AD B2C appelle l’API de couche intermédiaire et transmet les attributs de l’utilisateur.
| 3. | Une API de couche intermédiaire collecte des attributs d’utilisateur et les convertit dans un format consommable par l’API Onfido. Ensuite, elle envoie les informations à Onfido.
| 4. | Onfido utilise les informations et les traite pour valider l’identification de l’utilisateur. Ensuite, il retourne le résultat à l’API de couche intermédiaire.
| 5. | L’API de couche intermédiaire traite les informations et renvoie les informations pertinentes dans le format JSON approprié à Azure AD B2C.
| 6. | Azure AD B2C reçoit les informations envoyées par l’API de couche intermédiaire. S’il affiche une réponse Échec, un message d’erreur est affiché à l’utilisateur. S’il affiche une réponse Réussite, l’utilisateur est authentifié et écrit dans le répertoire.

## <a name="onboard-with-onfido"></a>Intégrer avec Onfido

1. Pour créer un compte Onfido, contactez [Onfido](https://onfido.com/signup/).

2. Une fois qu’un compte est créé, créez une [clé API](https://documentation.onfido.com/). Les clés dynamiques sont facturables. Vous pouvez cependant utiliser les [clés de bac à sable pour tester](https://documentation.onfido.com/?javascript#sandbox-and-live-differences) la solution. Les clés de bac à sable produisent la même structure de résultat que les clés dynamiques. Toutefois, les résultats sont toujours prédéterminés. Les documents ne sont pas traités ou enregistrés.

>[!NOTE]
> Vous aurez besoin de la clé ultérieurement.

Pour plus d’informations sur Onfido, consultez la [documentation sur l’API Onfido](https://documentation.onfido.com) et le [Hub des développeurs Onfido](https://developers.onfido.com).  

## <a name="configure-azure-ad-b2c-with-onfido"></a>Configurer Azure AD B2C avec Onfido

### <a name="part-1---deploy-the-api"></a>Partie 1 : Déploiement de l’API

- Déployez le [code d’API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/OnFido-Combined/API/Onfido.Api) fourni sur un service Azure. Le code peut être publié à partir de Visual Studio, en suivant ces [instructions](/visualstudio/deployment/quickstart-deploy-to-azure).
- Configurez CORS, et ajoutez une **Origine autorisée** en tant que https://{votre_nom_locataire}.b2clogin.com.

>[!NOTE]
>Vous aurez besoin de l’URL du service déployé pour configurer Azure AD avec les paramètres requis.

#### <a name="adding-sensitive-configuration-settings"></a>Ajout de paramètres de configuration sensibles

Les paramètres de l’application peuvent être configurés dans le [service d’application dans Azure](../app-service/configure-common.md#configure-app-settings). Le service d’application permet de configurer les paramètres en toute sécurité sans les archiver dans un référentiel. L’API REST a besoin des paramètres suivants :

| Nom du paramètre d’application | Source | Notes |
|:-------------------------|:-------|:-------|
|OnfidoSettings:AuthToken| Compte Onfido |

### <a name="part-2---deploy-the-ui"></a>Partie 2 : Déploiement de l’interface utilisateur

#### <a name="configure-your-storage-location"></a>Configurer votre emplacement de stockage

1. Configurer un [conteneur de stockage blob dans votre compte de stockage](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)

2. Stockez les fichiers d’interface utilisateur du [dossier d’interface utilisateur](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/OnFido-Combined/UI) dans votre conteneur d’objets blob.

3. Autorisez CORS à accéder au conteneur de stockage que vous avez créé en suivant les instructions ci-dessous :

   a. Accédez à **Paramètres** >**Origine autorisée**, puis entrez `https://{your_tenant_name}.b2clogin.com`. Remplacez « your-tenant-name » par le nom de votre locataire Azure AD B2C. Par exemple, https://fabrikam.b2clogin.com. Utilisez des minuscules quand vous entrez le nom de votre locataire.

   b. Pour **Méthodes autorisées**, sélectionnez `GET` et `PUT`.

   c. Sélectionnez **Enregistrer**.

#### <a name="update-ui-files"></a>Mettre à jour les fichiers d’interface utilisateur

1. Dans les fichiers de l’interface utilisateur, accédez au dossier [**ocean_blue**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/OnFido-Combined/UI/ocean_blue).

2. Ouvrez chaque fichier HTML.

3. Recherchez et remplacez {your-ui-blob-container-url} par l’URL de l’emplacement où se trouvent les dossiers **ocean_blue**, **dist** et **assets** de votre interface utilisateur.

4. Recherchez et remplacez {your-intermediate-api-url} par l’URL du service d’application API intermédiaire.

#### <a name="upload-your-files"></a>Charger vos fichiers

1. Stockez les fichiers d’interface utilisateur du dossier d’interface utilisateur dans votre conteneur d’objets blob.

2. Utilisez l’[Explorateur Stockage Azure](../virtual-machines/disks-use-storage-explorer-managed-disks.md) pour gérer vos fichiers et autorisations d’accès.

### <a name="part-3---configure-azure-ad-b2c"></a>Partie 3 : Configurer Azure AD B2C

#### <a name="replace-the-configuration-values"></a>Remplacer des valeurs de configuration

Dans les [stratégies personnalisées](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/OnFido-Combined/Policies) fournies, recherchez les espaces réservés suivants et remplacez-les par les valeurs correspondantes de votre instance.

| Espace réservé | Remplacer par la valeur | Exemple  |
|:---------------|:----------------|:-------------------|
| {your_tenant_name}  | Nom abrégé de votre locataire |  « yourtenant » de yourtenant.onmicrosoft.com |
| {your_tenantID} | TenantID de votre locataire Azure AD B2C | 01234567-89ab-cdef-0123-456789abcdef           |
| {your_tenant_IdentityExperienceFramework_appid}        | ID de l’application IdentityExperienceFramework configuré dans votre locataire Azure AD B2C      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_ ProxyIdentityExperienceFramework _appid} | ID de l’application ProxyIdentityExperienceFramework configuré dans votre locataire Azure AD B2C | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_appid}                         | ID d’application de l’application de stockage de votre locataire                                      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_app_objectid}                  | ID d’objet de l’application de stockage de votre locataire                                   | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_app_insights_instrumentation_key} | Clé d’instrumentation de votre instance App Insights*| 01234567-89ab-cdef-0123-456789abcdef|
|{your_ui_file_base_url}| URL de l’emplacement où se trouvent les dossiers **ocean_blue**, **dist** et **assets** de votre interface utilisateur | https://yourstorage.blob.core.windows.net/UI/|
| {your_app_service_URL}                                 | URL du service d’application que vous avez configurée                                             | `https://yourapp.azurewebsites.net`          |

*Application Insights peut se trouver dans un autre locataire. Cette étape est facultative. Supprimez les éléments TechnicalProfiles et OrchestrationSteps correspondants s’ils ne sont pas nécessaires.

### <a name="part-4---configure-the-azure-ad-b2c-policy"></a>Partie 4 : Configuration de la stratégie Azure AD B2C

Pour obtenir des instructions sur la configuration de votre client Azure AD B2C et de stratégies, consultez ce [document](./custom-policy-get-started.md?tabs=applications#custom-policy-starter-pack).

>[!NOTE]
> Nous recommandons aux clients d’ajouter une notification de consentement sur la page de collection d’attributs. Informez les utilisateurs que des informations seront envoyées à des services tiers à des fins de vérification d’identité.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire Azure AD B2C locataire, puis, sous Stratégies, sélectionnez **Identity Experience Framework**.

2. Sélectionnez votre **SignUpSignIn** créé précédemment.

3. Sélectionnez **Exécuter le flux d’utilisateurs**, puis sélectionnez les paramètres :

   a. **Application** : sélectionnez l’application inscrite (JWT dans l’exemple).

   b. **URL de réponse** : sélectionnez l’**URL de redirection**.

   c. Sélectionnez **Exécuter le flux utilisateur**.

4. Suivez le processus d’inscription et créez un compte

5. Le service Onfido est appelé pendant le flux, après la création de l’attribut utilisateur. Si le flux est incomplet, vérifiez que l’utilisateur n’est pas enregistré dans le répertoire.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)
