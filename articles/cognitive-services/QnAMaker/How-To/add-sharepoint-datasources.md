---
title: Fichiers SharePoint – QnA Maker
description: Ajoutez des sources de données SharePoint sécurisées à votre base de connaissances afin d’enrichir celle-ci de questions et de réponses pouvant être sécurisées avec Active Directory.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: c231ac95841043e5576f064e683dd86d9695b108
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353185"
---
# <a name="add-a-secured-sharepoint-data-source-to-your-knowledge-base"></a>Ajouter une source de données SharePoint sécurisée à votre base de connaissances

Ajoutez des sources de données SharePoint informatiques sécurisées à votre base de connaissances afin d’enrichir celle-ci de questions et de réponses pouvant être sécurisées avec Active Directory.

Lorsque vous ajoutez un document SharePoint sécurisé à votre base de connaissances, en tant que gestionnaire QnA Maker, vous devez demander une autorisation Active Directory pour QnA Maker. Une fois que le gestionnaire Active Directory a accordé à QnA Maker cette autorisation d’accès à SharePoint, il n’est plus nécessaire de la redonner. Chaque ajout de document suivant à la base de connaissances ne nécessite pas d’autorisation si ce document figure dans la même ressource SharePoint.

Si le Gestionnaire de la Base de connaissances QnA Maker n’est pas le Gestionnaire Active Directory, vous devez communiquer avec le Gestionnaire Active Directory pour terminer ce processus.

## <a name="prerequisites"></a>Prérequis

* SharePoint informatique : QnA Maker utilise Microsoft Graph pour les autorisations. Si votre SharePoint est local, vous ne pourrez pas procéder à l’extraction à partir de SharePoint, car Microsoft Graph ne sera pas en mesure de déterminer les autorisations.
* Format d’URL – QnA Maker prend en charge uniquement les URL SharePoint qui sont générées pour le partage et qui sont au format `https://\*.sharepoint.com`

## <a name="add-supported-file-types-to-knowledge-base"></a>Ajouter des types de fichiers pris en charge à la Base de connaissances

Vous pouvez ajouter tout [type de fichier](../index.yml) pris en charge par QnA Maker à partir d’un site SharePoint à votre base de connaissances. Si la ressource de fichier est sécurisée, il se peut que vous deviez accorder des [autorisations](#permissions).

1. À partir de la bibliothèque avec le site SharePoint, sélectionnez le menu symbolisé par des points de suspension (`...`) du fichier.
1. Copiez l’URL du fichier.

   ![Obtenez l’URL du fichier SharePoint en sélectionnant le menu symbolisé par des points de suspension du fichier, puis en copiant l’URL.](../media/add-sharepoint-datasources/get-sharepoint-file-url.png)

1. Dans le portail QnA Maker, sur la page **Paramètres**, ajoutez l’URL à la Base de connaissances.

### <a name="images-with-sharepoint-files"></a>Images représentant des fichiers SharePoint

Les images contenues dans les fichiers ne sont pas extraites. Vous pouvez ajouter l’image à partir du portail QnA Maker une fois le fichier extrait dans des paires QnA.

Ajoutez l’image à l’aide de la syntaxe Markdown suivante :

```markdown
![Explanation or description of image](URL of public image)
```

Le texte entre crochets `[]` décrit l’image. L’URL entre parenthèses `()` est le lien direct vers l’image.

Lorsque vous testez la paire QnA dans le panneau de test interactif, sur le portail QnA Maker, l’image s’affiche à la place du texte Markdown. Cela valide le fait que l’image peut être extraite publiquement de votre application cliente.

## <a name="permissions"></a>Autorisations

Un octroi d’autorisations a lieu quand un fichier sécurisé d’un serveur SharePoint est ajouté à une base de connaissances. Selon la configuration de SharePoint et les autorisations accordées à la personne qui ajoute le fichier, cette opération peut nécessiter :

* Aucune étape supplémentaire si la personne qui ajoute le fichier dispose de toutes les autorisations nécessaires.
* Des actions du [Gestionnaire de la Base de connaissances](#knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal) et du [ Gestionnaire Active Directory](#active-directory-manager-grant-file-read-access-to-qna-maker).

Consultez les étapes répertoriées ci-dessous.

### <a name="knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal"></a>Gestionnaire de la base de connaissances : ajouter la source de données SharePoint dans le portail QnA Maker

Lorsque le **gestionnaire QnA Maker** ajoute un document SharePoint sécurisé à une base de connaissances, le gestionnaire de la base de connaissances lance une demande d’autorisation à laquelle le gestionnaire Active Directory doit répondre.

La demande commence par une fenêtre contextuelle permettant de s’authentifier auprès d’un compte Active Directory.

![Authentifier un compte d’utilisateur](../media/add-sharepoint-datasources/authenticate-user-account.png)

Lorsque le gestionnaire QnA Maker sélectionne le compte, l’administrateur Azure Active Directory reçoit un avis indiquant qu’il doit autoriser l’application QnA Maker (et non le gestionnaire QnA Maker) à accéder à la ressource SharePoint. Le gestionnaire Azure Active Directory doit effectuer cette opération pour chaque ressource SharePoint, mais pas pour chaque document dans cette ressource.

### <a name="active-directory-manager-grant-file-read-access-to-qna-maker"></a>Gestionnaire Active directory : accorder l’accès en lecture de fichier à QnA Maker

Le gestionnaire Active Directory (et non le gestionnaire QnA Maker) doit autoriser QnA Maker à accéder à la ressource SharePoint en sélectionnant [ce lien](https://login.microsoftonline.com/common/oauth2/v2.0/authorize?response_type=id_token&scope=Files.Read%20Files.Read.All%20Sites.Read.All%20User.Read%20User.ReadBasic.All%20profile%20openid%20email&client_id=c2c11949-e9bb-4035-bda8-59542eb907a6&redirect_uri=https%3A%2F%2Fwww.qnamaker.ai%3A%2FCreate&state=68) pour autoriser l’application d’entreprise SharePoint du portail QnA Maker à recevoir des autorisations de lecture de fichier.

![Le Gestionnaire Active Directory accorde une autorisation de façon interactive](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)

<!--
The Active Directory manager must grant QnA Maker access either by application name, `QnAMakerPortalSharePoint`, or by application ID, `c2c11949-e9bb-4035-bda8-59542eb907a6`.
-->
<!--
### Grant access from the interactive pop-up window

The Active Directory manager will get a pop-up window requesting permissions to the `QnAMakerPortalSharePoint` app. The pop-up window includes the QnA Maker Manager email address that initiated the request, an `App Info` link to learn more about **QnAMakerPortalSharePoint**, and a list of permissions requested. Select **Accept** to provide those permissions.

![Azure Active Directory manager grants permission interactively](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)
-->
<!--

### Grant access from the App Registrations list

1. The Active Directory manager signs in to the Azure portal and opens **[App registrations list](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ApplicationsListBlade)**.

1. Search for and select the **QnAMakerPortalSharePoint** app. Change the second filter box from **My apps** to **All apps**. The app information will open on the right side.

    ![Select QnA Maker app in App registrations list](../media/add-sharepoint-datasources/select-qna-maker-app-in-app-registrations.png)

1. Select **Settings**.

    [![Select Settings in the right-side blade](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png)](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png#lightbox)

1. Under **API access**, select **Required permissions**.

    ![Select 'Settings', then under 'API access', select 'Required permission'](../media/add-sharepoint-datasources/select-required-permissions-in-settings-blade.png)

1. Do not change any settings in the **Enable Access** window. Select **Grant Permission**.

    [![Under 'Grant Permission', select 'Yes'](../media/add-sharepoint-datasources/grant-app-required-permissions.png)](../media/add-sharepoint-datasources/grant-app-required-permissions.png#lightbox)

1. Select **YES** in the pop-up confirmation windows.

    ![Grant required permissions](../media/add-sharepoint-datasources/grant-required-permissions.png)
-->
### <a name="grant-access-from-the-azure-active-directory-admin-center"></a>Autoriser l’accès à partir du Centre d’administration Azure Active Directory

1. Le Gestionnaire Active Directory se connecte au portail Azure et ouvre **[Applications d’entreprise](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps)** .

1. Recherchez `QnAMakerPortalSharePoint` pour sélectionner l’application de QnA Maker.

    [![Rechercher QnAMakerPortalSharepoint dans la liste Applications d’entreprise](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png)](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png#lightbox)

1. Sous **Sécurité**, accédez à **Autorisations**. Sélectionnez **Accorder le consentement de l’administrateur pour l’organisation**.

    [![Sélectionner un utilisateur authentifié pour Administrateur Active Directory](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png)](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png#lightbox)

1. Sélectionnez un compte d’authentification autorisé à accorder des autorisations pour Active Directory.



<!--

## Add SharePoint data source with APIs

You need to get the SharePoint file's URI before adding it to QnA Maker.

## Get SharePoint File URI

Use the following steps to transform the SharePoint URL into a sharing token.

1. Encode the URL using [base64](https://en.wikipedia.org/wiki/Base64).

1. Convert the base64-encoded result to an unpadded base64url format with the following character changes.

    * Remove the equal character, `=` from the end of the value.
    * Replace `/` with `_`.
    * Replace `+` with `-`.
    * Append `u!` to be beginning of the string.

1. Sign in to Graph explorer and run the following query, where `sharedURL` is ...:

    ```
    https://graph.microsoft.com/v1.0/shares/<sharedURL>/driveitem
    ```

    Get the **@microsoft.graph.downloadUrl** and use this as `fileuri` in the QnA Maker APIs.

### Add or update a SharePoint File URI to your knowledge base

Use the **@microsoft.graph.downloadUrl** from the previous section as the `fileuri` in the QnA Maker API for [adding a knowledge base](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase) or [updating a knowledge base](/rest/api/cognitiveservices/qnamaker/knowledgebase/update). The following fields are mandatory: name, fileuri, filename, source.

```
{
    "name": "Knowledge base name",
    "files": [
        {
            "fileUri": "<@microsoft.graph.downloadURL>",
            "fileName": "filename.xlsx",
            "source": "<SharePoint link>"
        }
    ],
    "urls": [],
    "users": [],
    "hostUrl": "",
    "qnaList": []
}
```



## Remove QnA Maker app from SharePoint authorization

1. Use the steps in the previous section to find the Qna Maker app in the Active Directory admin center.
1. When you select the **QnAMakerPortalSharePoint**, select **Overview**.
1. Select **Delete** to remove permissions.

-->

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Collaborer sur votre Base de connaissances](../index.yml)