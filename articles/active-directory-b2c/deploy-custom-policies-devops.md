---
title: Déployer des stratégies personnalisées avec Azure Pipelines
titleSuffix: Azure AD B2C
description: Découvrez comment déployer des stratégies personnalisées Azure AD B2C dans un pipeline CI/CD en utilisant Azure Pipelines dans Azure DevOps Services.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 13f5f8da0bd58cef0974e8ea8f5f3c5172daa0ba
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928730"
---
# <a name="deploy-custom-policies-with-azure-pipelines"></a>Déployer des stratégies personnalisées avec Azure Pipelines

En utilisant un pipeline d’intégration et de livraison continues (CI/CD) que vous configurez dans [Azure Pipelines][devops-pipelines], vous pouvez inclure vos stratégies personnalisées Azure AD B2C dans la livraison et l’automatisation du contrôle de code de vos logiciels. Lorsque vous déployez dans différents environnements Azure AD B2C, par exemple de dev, de test et de production, nous vous recommandons de supprimer les processus manuels et d’effectuer des tests automatisés à l’aide d’Azure Pipelines.

Trois étapes principales sont nécessaires pour permettre à Azure Pipelines de gérer des stratégies personnalisées dans Azure AD B2C :

1. Créer une inscription d’application web dans votre client AAD B2C
1. Configurer un référentiel Azure
1. Configurer un pipeline Azure

> [!IMPORTANT]
> La gestion de stratégies Azure AD B2C personnalisées avec un pipeline Azure utilise actuellement des opérations en **préversion** disponibles sur le point de terminaison `/beta` de l’API Microsoft Graph. L’utilisation de ces API dans les applications de production n’est pas prise en charge. Pour plus d’informations, voir la [référence du point de terminaison beta de l’API REST Microsoft Graph](/graph/api/overview?toc=.%2fref%2ftoc.json&view=graph-rest-beta).

## <a name="prerequisites"></a>Prérequis

* [Locataire Azure AD B2C](tutorial-create-tenant.md) et informations d'identification d'un utilisateur de l'annuaire disposant du rôle [Administrateur de stratégies B2C IEF](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator)
* [Stratégies personnalisées](custom-policy-get-started.md) téléchargées sur votre locataire
* [Application de gestion](microsoft-graph-get-started.md) inscrite dans votre locataire avec l’autorisation de l’API Microsoft Graph *Policy.ReadWrite.TrustFramework*
* [Pipeline Azure](https://azure.microsoft.com/services/devops/pipelines/) et accès à un [projet Azure DevOps Services][devops-create-project]

## <a name="client-credentials-grant-flow"></a>Flux d’octroi d’informations d’identification de client

Le scénario décrit ici utilise des appels de service à service entre Azure Pipelines et Azure AD B2C à l’aide du [flux d’octroi d’informations d’identification de client](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) OAuth 2.0. Le flux d’octroi permet à un service web tel qu’Azure Pipelines (le client confidentiel) d’utiliser ses propres informations d’identification au lieu d’emprunter l’identité d’un utilisateur pour s’authentifier lorsqu’il appelle un autre service web (en l’occurrence l’API Microsoft Graph). Azure Pipelines obtient un jeton de manière non interactive, puis adresse des demandes à l’API Microsoft Graph.

## <a name="register-an-application-for-management-tasks"></a>Inscrire une application pour des tâches de gestion

Comme indiqué dans [Conditions préalables](#prerequisites), vous avez besoin d’une inscription d’application que vos scripts PowerShell (exécutés par Azure Pipelines) peuvent utiliser pour accéder aux ressources de votre locataire.

Si vous disposez déjà d’une inscription d’application que vous utilisez pour des tâches d’automatisation, assurez-vous qu’elle dispose de l’autorisation **Microsoft Graph** > **Stratégie** > **Policy.ReadWrite.TrustFramework** dans les **autorisations d’API** de l’inscription de l’application.

Pour obtenir des instructions sur l’inscription d’une application de gestion, voir [Gérer Azure AD B2C avec Microsoft Graph](microsoft-graph-get-started.md).

## <a name="configure-an-azure-repo"></a>Configurer un référentiel Azure

Avec une application de gestion inscrite, vous êtes prêt à configurer un référentiel pour vos fichiers de stratégie.

1. Connectez-vous à votre organisation Azure DevOps Services.
1. [Créez un projet][devops-create-project] ou sélectionnez un projet existant.
1. Dans votre projet, accédez à **Référentiels**, puis sélectionnez la page **Fichiers**. Sélectionnez un référentiel existant ou créez-en un pour cet exercice.
1. Créez un dossier intitulé *B2CAssets*. Nommez le fichier d’espace réservé requis *README.md*, puis **validez** le fichier. Vous pouvez supprimer ce fichier ultérieurement si nécessaire.
1. Ajoutez vos fichiers de stratégie Azure AD B2C au dossier *B2CAssets*. Cette opération comprend les fichiers de stratégie *TrustFrameworkBase.xml*, *TrustFrameWorkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* et *PasswordReset.xml*, ainsi que toutes les autres stratégies que vous avez créées. Enregistrez le nom de fichier de chaque fichier de stratégie Azure AD B2C à utiliser dans une étape ultérieure (ils sont utilisés en tant qu’arguments de script PowerShell).
1. Créez un dossier nommé *Scripts* dans le répertoire racine du référentiel, nommez le fichier d’espace réservé *DeployToB2c.ps1*. Ne validez pas le fichier à ce stade. Vous le ferez à une étape ultérieure.
1. Collez le script PowerShell suivant dans le fichier *DeployToB2c.ps1*, puis **validez** le fichier. Le script acquiert un jeton à partir de Azure AD et appelle l’API Microsoft Graph pour charger les stratégies dans le dossier *B2CAssets* sur votre locataire Azure AD B2C.

    ```PowerShell
    [Cmdletbinding()]
    Param(
        [Parameter(Mandatory = $true)][string]$ClientID,
        [Parameter(Mandatory = $true)][string]$ClientSecret,
        [Parameter(Mandatory = $true)][string]$TenantId,
        [Parameter(Mandatory = $true)][string]$PolicyId,
        [Parameter(Mandatory = $true)][string]$PathToFile
    )

    try {
        $body = @{grant_type = "client_credentials"; scope = "https://graph.microsoft.com/.default"; client_id = $ClientID; client_secret = $ClientSecret }

        $response = Invoke-RestMethod -Uri https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token -Method Post -Body $body
        $token = $response.access_token

        $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
        $headers.Add("Content-Type", 'application/xml')
        $headers.Add("Authorization", 'Bearer ' + $token)

        $graphuri = 'https://graph.microsoft.com/beta/trustframework/policies/' + $PolicyId + '/$value'
        $policycontent = Get-Content $PathToFile
        $response = Invoke-RestMethod -Uri $graphuri -Method Put -Body $policycontent -Headers $headers

        Write-Host "Policy" $PolicyId "uploaded successfully."
    }
    catch {
        Write-Host "StatusCode:" $_.Exception.Response.StatusCode.value__

        $_

        $streamReader = [System.IO.StreamReader]::new($_.Exception.Response.GetResponseStream())
        $streamReader.BaseStream.Position = 0
        $streamReader.DiscardBufferedData()
        $errResp = $streamReader.ReadToEnd()
        $streamReader.Close()

        $ErrResp

        exit 1
    }

    exit 0
    ```

## <a name="configure-your-azure-pipeline"></a>Configurer votre pipeline Azure AD

Une fois votre référentiel initialisé et rempli avec vos fichiers de stratégie personnalisés, vous êtes prêt à configurer le pipeline de mise en production.

### <a name="create-pipeline"></a>Création d’un pipeline

1. Connectez-vous à votre organisation Azure DevOps et accédez à votre projet.
1. Dans votre projet, sélectionnez **Pipelines** > **Versions** > **Nouveau pipeline**.
1. Sous **Sélectionnez un modèle**, choisissez **Travail vide**.
1. Entrez le **Nom de l’étape**, par exemple, *DeployCustomPolicies*, puis fermez le volet.
1. Sélectionnez **Ajouter un artefact**, puis, sous **Type de source**, choisissez **Référentiel Azure**.
    1. Choisissez le référentiel source contenant le dossier *Scripts* que vous avez rempli avec le script PowerShell.
    1. Choisissez une **Branche par défaut**. Si vous avez créé un référentiel dans la section précédente, la branche par défaut est la branche *master*.
    1. Conservez le paramètre **Version par défaut** de *La dernière de la branche par défaut*.
    1. Entrez un **Alias source** pour le référentiel. Par exemple, *policyRepo*. N’incluez pas d’espace dans le nom de l’alias.
1. Sélectionnez **Ajouter**
1. Renommez le pipeline afin de refléter son intention. Par exemple, *Pipeline de déploiement de stratégie personnalisée*.
1. Sélectionnez **Enregistrer** pour enregistrer la configuration du pipeline.

### <a name="configure-pipeline-variables"></a>Configurer des variables de pipeline

1. Sélectionnez l’onglet **Variables**.
1. Ajoutez les variables suivantes sous **Variables de pipeline**, puis définissez leurs valeurs comme indiqué :

    | Nom | Valeur |
    | ---- | ----- |
    | `clientId` | **ID d’application (client)** pour l’application que vous avez inscrite. |
    | `clientSecret` | Valeur de la **clé secrète client** que vous avez créée précédemment. <br /> Modifiez le type de variable en **secret** (sélectionnez l’icône de verrou). |
    | `tenantId` | `your-b2c-tenant.onmicrosoft.com`, où *your-b2c-tenant* est le nom de votre locataire Azure AD B2C. |

1. Sélectionnez **Enregistrer** pour enregistrer les variables.

### <a name="add-pipeline-tasks"></a>Ajouter des tâches de pipeline

Ensuite, ajoutez une tâche pour déployer un fichier de stratégie.

1. Cliquez sur l’onglet **Tâches**.
1. Sélectionnez **Travail de l’agent**, puis le signe plus ( **+** ) pour ajouter une tâche au travail de l’agent.
1. Recherchez et sélectionnez **PowerShell**. Ne sélectionnez pas « Azure PowerShell », « PowerShell sur des machines cibles » ou une autre entrée PowerShell.
1. Sélectionnez la tâche de **Script PowerShell** que vous venez d’ajouter.
1. Entrez les valeurs suivantes pour la tâche de Script PowerShell :
    * **Version de la tâche** : 2.*
    * **Nom d'affichage** : Nom de la stratégie que cette tâche doit charger. Par exemple, *B2C_1A_TrustFrameworkBase*.
    * **Type** : Chemin d'accès au fichier
    * **Chemin d’accès au script** : Sélectionnez les points de suspension (**_..._* _), accédez au dossier _Scripts*, puis sélectionnez le fichier *DeployToB2C.ps1*.
    * **Arguments :**

        Entrez les valeurs suivantes pour les **Arguments**. Remplacez `{alias-name}` par l’alias que vous avez spécifié dans la section précédente.

        ```PowerShell
        # Before
        -ClientID $(clientId) -ClientSecret $(clientSecret) -TenantId $(tenantId) -PolicyId B2C_1A_TrustFrameworkBase -PathToFile $(System.DefaultWorkingDirectory)/{alias-name}/B2CAssets/TrustFrameworkBase.xml
        ```

        Par exemple, si l’alias que vous avez spécifié est *policyRepo*, la ligne d’argument doit être :

        ```PowerShell
        # After
        -ClientID $(clientId) -ClientSecret $(clientSecret) -TenantId $(tenantId) -PolicyId B2C_1A_TrustFrameworkBase -PathToFile $(System.DefaultWorkingDirectory)/policyRepo/B2CAssets/TrustFrameworkBase.xml
        ```

1. Sélectionnez **Enregistrer** pour enregistrer le Travail de l’agent.

La tâche que vous venez d’ajouter charge *un* fichier de stratégie sur Azure AD B2C. Avant de continuer, déclenchez manuellement le travail (**Créer une mise en production**) pour vous assurer qu’il soit correctement accompli avant la création de tâches supplémentaires.

Si la tâche se termine avec succès, ajoutez des tâches de déploiement en suivant les étapes précédentes pour chacun des fichiers de stratégie personnalisée. Modifiez les valeurs des arguments `-PolicyId` et `-PathToFile` pour chaque stratégie.

L’ID `PolicyId` est une valeur trouvée au début d’un fichier de stratégie XML au sein du nœud TrustFrameworkPolicy. Par exemple, l’ID `PolicyId` dans le fichier XML de stratégie suivant est *B2C_1A_TrustFrameworkBase* :

```xml
<TrustFrameworkPolicy
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:xsd="http://www.w3.org/2001/XMLSchema"
xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
PolicySchemaVersion="0.3.0.0"
TenantId="contoso.onmicrosoft.com"
PolicyId= "B2C_1A_TrustFrameworkBase"
PublicPolicyUri="http://contoso.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
```

Lors de l’exécution des agents et du chargement des fichiers de stratégie, assurez-vous que ceux-ci sont chargés dans l’ordre suivant :

1. *TrustFrameworkBase.xml*
1. *TrustFrameworkExtensions.xml*
1. *SignUpOrSignin.xml*
1. *ProfileEdit.xml*
1. *PasswordReset.xml*

L’infrastructure Identity Experience Framework applique cet ordre lorsque la structure de fichier est construite sur une chaîne hiérarchique.

## <a name="test-your-pipeline"></a>Tester votre pipeline

Pour tester votre pipeline de mise en production :

1. Sélectionnez **Pipelines**, puis **Mises en production**.
1. Sélectionnez le pipeline précédemment, par exemple *DeployCustomPolicies*.
1. Sélectionnez **Créer une mise en production**, puis **Créer** pour mettre en file d’attente la mise en production.

Vous devez voir une bannière de notification indiquant qu’une mise en production a été mise en file d’attente. Pour afficher son état, sélectionnez le lien dans la bannière de notification ou dans la liste sous l’onglet **Mises en production**.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus :

* [Appels de service à service utilisant des informations d’identification de client](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md)
* [Azure DevOps Services](/azure/devops/user-guide/)

<!-- LINKS - External -->
[devops]: /azure/devops/
[devops-create-project]:  /azure/devops/organizations/projects/create-project
[devops-pipelines]: /azure/devops/pipelines
