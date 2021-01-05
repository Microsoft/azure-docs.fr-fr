---
title: Tutoriel `:` Utiliser une identité managée pour accéder à Azure Data Lake Store - Windows - Azure AD
description: Un didacticiel qui vous montre comment utiliser une identité managée attribuée par le système d’une machine virtuelle Windows pour accéder à Azure Data Lake Store.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 908fb1ac869ec2b22085af2e07ced6ff64229308
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97592481"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-data-lake-store"></a>Didacticiel : Utiliser une identité managée attribuée par le système de la machine virtuelle Windows pour accéder à Azure Data Lake Store

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Ce didacticiel vous indique comment utiliser une identité managée attribuée par le système pour une machine virtuelle Windows afin d’accéder à Azure Data Lake Store. Les identités MSI sont gérées automatiquement par Azure et vous permettent de vous authentifier auprès des services prenant en charge l’authentification Azure AD sans avoir à insérer des informations d’identification dans votre code. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Accorder à votre machine virtuelle l’accès à Azure Data Lake Store
> * Obtenir un jeton d’accès à l’aide de l’identité de machine virtuelle, et l’utiliser pour accéder à Azure Data Lake Store

## <a name="prerequisites"></a>Conditions préalables requises

- Compréhension des identités managées. Si vous n’êtes pas familiarisé de la fonctionnalité identités managées pour ressources Azure, consultez cette [Vue d’ensemble](overview.md). 
- Un compte Azure. [Inscrivez-vous pour obtenir un compte gratuit](https://azure.microsoft.com/free/).
- Des autorisations « Propriétaire » avec l’étendue appropriée (votre abonnement ou groupe de ressources) pour effectuer les étapes de création de ressource et de gestion de rôles nécessaires. Si vous avez besoin d’aide concernant l’attribution de rôle, consultez [Utiliser le contrôle d’accès en fonction du rôle pour gérer l’accès aux ressources d’un abonnement Azure](../../role-based-access-control/role-assignments-portal.md).
- Vous avez également besoin d’une machine virtuelle Windows sur laquelle les identités managées attribuées par le système sont activées.
  - Si vous devez créer une machine virtuelle pour ce tutoriel, vous pouvez suivre les instructions de l’article intitulé [Créer une machine virtuelle avec une identité affectée par le système activée](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity)




## <a name="enable"></a>Activer

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]



## <a name="grant-access"></a>Accorder l'accès

Maintenant, vous pouvez accorder à votre machine virtuelle l’accès à des fichiers et des dossiers dans un Azure Data Lake Store.  Pour cette étape, vous pouvez utiliser un Data Lake Store existant ou bien en créer un.  Pour créer un nouveau Data Lake Store via le portail Azure, suivez ce [Démarrage rapide de Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md). Des procédures de démarrage rapide utilisant Azure CLI et Azure PowerShell sont également décrites dans la [Documentation Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md).

Dans Data Lake Store, créez un dossier et autorisez l’identité managée attribuée par le système de la machine virtuelle à lire, écrire et exécuter des fichiers dans ce dossier :

1. Dans le portail Azure, cliquez sur **Data Lake Store** dans le volet de navigation gauche.
2. Cliquez sur le Data Lake Store que vous souhaitez utiliser pour ce didacticiel.
3. Cliquez sur **Explorateur de données** dans la barre de commandes.
4. Le dossier racine du Data Lake Store est sélectionné.  Cliquez sur **Accéder** dans la barre de commandes.
5. Cliquez sur **Add**.  Dans le champ **Sélectionner**, saisissez le nom de votre machine virtuelle, par exemple **DevTestVM**.  Sélectionnez votre machine virtuelle à partir des résultats de recherche, puis cliquez sur **Sélectionner**.
6. Cliquez sur **Sélectionner les autorisations**.  Sélectionnez **Lire** et **Exécuter**, ajoutez à **Ce dossier** et ajoutez en tant qu’une **Autorisation d’accès uniquement**.  Cliquez sur **OK**.  L’autorisation doit être ajoutée avec succès.
7. Fermez le panneau **Accès**.
8. Pour ce didacticiel, créons un nouveau dossier.  Cliquez sur **Nouveau dossier** dans la barre de commandes et donnez-lui un nom, par exemple **TestFolder**.  Cliquez sur **OK**.
9. Cliquez sur le dossier créé, puis cliquez sur **Accès** dans la barre de commandes.
10. Semblable à l’étape 5, cliquez sur **Ajouter**, saisissez le nom de votre machine virtuelle dans le champ **Sélectionner**, sélectionnez-la puis cliquez sur **Sélectionner**.
11. Semblable à l’étape 6, cliquez sur **Sélectionner les autorisations**, sélectionnez **Lire**, **Écrire**, et **Exécuter**, ajoutez à **Ce dossier** et ajoutez en tant qu’**une entrée d’autorisation accès et une entrée d’autorisation par défaut**.  Cliquez sur **OK**.  L’autorisation doit être ajoutée avec succès.

Votre identité managée attribuée par le système de la machine virtuelle peut désormais effectuer toutes les opérations sur les fichiers du dossier que vous avez créé.  Pour plus d’informations sur la gestion de l’accès à Data Lake Store, lisez cet article sur le [Contrôle d’accès dans Data Lake Store](../../data-lake-store/data-lake-store-access-control.md).

## <a name="access-data"></a>Accéder aux données

Azure Data Lake Store prenant en charge Azure AD Authentication en mode natif, il peut accepter directement des jetons d’accès obtenus à l’aide d’identités managées attribuées par le système pour les ressources Azure.  Pour s’authentifier sur le système de fichiers de Data Lake Store, vous envoyez un jeton d’accès émis par Azure AD pour votre point de terminaison de système de fichiers de Data Lake Store dans un en-tête d’autorisation au format « Porteur < ACCESS_TOKEN_VALUE > ».  Pour en savoir plus sur la prise en charge de Data Lake Store pour l’authentification à l’aide de Azure AD, lire [Authentification auprès de Data Lake Store à l’aide de Azure Active Directory](../../data-lake-store/data-lakes-store-authentication-using-azure-active-directory.md)

> [!NOTE]
> Les kits de développement logiciel clients du système de fichiers de Data Lake Store ne gèrent pas encore les identités managées pour les ressources Azure.  Ce didacticiel sera mis à jour lorsque cette prise en charge sera ajoutée aux kits de développements logiciel.

Dans ce didacticiel, vous vous authentifiez sur l’API REST du système de fichiers de Data Lake Store à l’aide de PowerShell pour effectuer des requêtes REST. Pour utiliser l’identité managée attribuée par le système de la machine virtuelle pour l’authentification, vous devez effectuer les requêtes à partir de la machine virtuelle.

1. Dans le portail, accédez à **Machines virtuelles** et accédez à votre machine virtuelle Windows puis, dans **Vue d’ensemble**, cliquez sur **Connecter**.
2. Entrez le **Nom d’utilisateur** et le **Mot de passe** que vous avez ajoutés lorsque vous avez créé la machine virtuelle Windows. 
3. Maintenant que vous avez créé une **Connexion au Bureau à distance** avec la machine virtuelle, ouvrez **PowerShell** dans la session à distance. 
4. À l’aide de `Invoke-WebRequest` de Powershell, adressez une requête au point de terminaison des identités managées locales pour les ressources Azure pour obtenir un jeton d’accès pour Azure Data Lake Store.  L’identificateur de ressources pour Data Lake Store est `https://datalake.azure.net/`.  Data Lake effectue une correspondance exacte sur l’identificateur de ressource et la barre oblique est importante.

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatalake.azure.net%2F' -Method GET -Headers @{Metadata="true"}
   ```
    
   Convertissez la réponse d’objet JSON en objet PowerShell. 
    
   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```

   Extrayez le jeton d’accès de la réponse.
    
   ```powershell
   $AccessToken = $content.access_token
   ```

5. À l’aide « Invoke-WebRequest » de PowerShell, faites une requête au point de terminaison REST de votre Data Lake Store pour répertorier les dossiers dans le dossier racine.  Il s’agit d’un moyen simple de vérifier que tout est correctement configuré.  Il est important de que la chaîne « Porteur » dans l’en-tête d’autorisation ait un « P » majuscule.  Vous pouvez rechercher le nom de votre Data Lake Store dans la section **Vue d’ensemble** du panneau Data Lake Store dans le portail Azure.

   ```powershell
   Invoke-WebRequest -Uri https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/?op=LISTSTATUS -Headers @{Authorization="Bearer $AccessToken"}
   ```

   Une réponse correcte se présente ainsi :

   ```powershell
   StatusCode        : 200
   StatusDescription : OK
   Content           : {"FileStatuses":{"FileStatus":[{"length":0,"pathSuffix":"TestFolder","type":"DIRECTORY", "blockSize":0,"accessTime":1507934941392, "modificationTime":1507944835699,"replication":0, "permission":"770","ow..."
   RawContent        : HTTP/1.1 200 OK
                       Pragma: no-cache
                       x-ms-request-id: b4b31e16-e968-46a1-879a-3474aa7d4528
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosniff
                       Strict-Transport-Security: ma...
   Forms             : {}
   Headers           : {[Pragma, no-cache], [x-ms-request-id, b4b31e16-e968-46a1-879a-3474aa7d4528],
                       [x-ms-webhdfs-version, 17.04.22.00], [Status, 0x0]...}
   Images            : {}
   InputFields       : {}
   Links             : {}
   ParsedHtml        : System.__ComObject
   RawContentLength  : 556
   ```

6. Vous pouvez maintenant essayer de charger un fichier sur votre Data Lake Store.  Commencez par créer un fichier à charger.

   ```powershell
   echo "Test file." > Test1.txt
   ```

7. À l’aide de `Invoke-WebRequest` de PowerShell, faites une requête au point de terminaison REST de votre Data Lake Store pour charger le fichier dans le dossier créé précédemment.  Cette requête prend deux étapes.  Dans la première étape, vous faites une requête et vous obtenez une redirection vers l’emplacement où le fichier doit être téléchargé.  Dans la deuxième étape, vous chargez réellement le fichier.  N’oubliez pas définir le nom du dossier et du fichier de façon appropriée si vous avez utilisé des valeurs différentes de celles du didacticiel. 

   ```powershell
   $HdfsRedirectResponse = Invoke-WebRequest -Uri https://<YOUR_ADLS_NAME>.azuredatalakestore.net/webhdfs/v1/TestFolder/Test1.txt?op=CREATE -Method PUT -Headers @{Authorization="Bearer $AccessToken"} -Infile Test1.txt -MaximumRedirection 0
   ```

   Si vous examinez la valeur de `$HdfsRedirectResponse`, elle doit ressembler à la réponse suivante :

   ```powershell
   PS C:\> $HdfsRedirectResponse

   StatusCode        : 307
   StatusDescription : Temporary Redirect
   Content           : {}
   RawContent        : HTTP/1.1 307 Temporary Redirect
                       Pragma: no-cache
                       x-ms-request-id: b7ab492f-b514-4483-aada-4aa0611d12b3
                       ContentLength: 0
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosn...
   Headers           : {[Pragma, no-cache], [x-ms-request-id, b7ab492f-b514-4483-aada-4aa0611d12b3], 
                       [ContentLength, 0], [x-ms-webhdfs-version, 17.04.22.00]...}
   RawContentLength  : 0
   ```

   Terminez le chargement en envoyant une requête au point de terminaison de redirection :

   ```powershell
   Invoke-WebRequest -Uri $HdfsRedirectResponse.Headers.Location -Method PUT -Headers @{Authorization="Bearer $AccessToken"} -Infile Test1.txt -MaximumRedirection 0
   ```

   Une réponse correcte se présente ainsi :

   ```powershell
   StatusCode        : 201
   StatusDescription : Created
   Content           : {}
   RawContent        : HTTP/1.1 201 Created
                       Pragma: no-cache
                       x-ms-request-id: 1e70f36f-ead1-4566-acfa-d0c3ec1e2307
                       ContentLength: 0
                       x-ms-webhdfs-version: 17.04.22.00
                       Status: 0x0
                       X-Content-Type-Options: nosniff
                       Strict...
   Headers           : {[Pragma, no-cache], [x-ms-request-id, 1e70f36f-ead1-4566-acfa-d0c3ec1e2307],
                       [ContentLength, 0], [x-ms-webhdfs-version, 17.04.22.00]...}
   RawContentLength  : 0
   ```

À l’aide d’autres API de système de fichiers de Data Lake Store, vous pouvez ajouter à des fichiers, télécharger des fichiers et bien plus encore.


## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser une identité managée attribuée par le système pour une machine virtuelle Windows afin d’accéder à Azure Data Lake Store. Pour en savoir plus sur Azure Data Lake Store, consultez :

> [!div class="nextstepaction"]
>[Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md)