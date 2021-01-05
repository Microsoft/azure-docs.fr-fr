---
title: Créer une ressource de Lecteur immersif
titleSuffix: Azure Cognitive Services
description: Cet article vous montre comment créer une ressource de lecteur immersif avec un sous-domaine personnalisé et comment configurer Azure AD dans votre locataire Azure.
services: cognitive-services
author: rwaller
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: conceptual
ms.date: 07/22/2019
ms.author: rwaller
ms.openlocfilehash: b012da0b2aea4a50002e9adbc0876396ddd4b5e7
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368727"
---
# <a name="create-an-immersive-reader-resource-and-configure-azure-active-directory-authentication"></a>Créer une ressource Lecteur immersif et configurer l’authentification Azure Active Directory

Dans cet article, nous fournissons un script qui va créer une ressource Lecteur immersif et configurer l’authentification Azure Active Directory (Azure AD). Chaque fois qu’une ressource Lecteur immersif est créée, avec ce script ou dans le portail, elle doit aussi être configurée avec les autorisations Azure AD. Ce script va vous y aider.

Le script est conçu pour créer et configurer toutes les ressources Lecteur immersif et Azure AD nécessaires en une seule étape pour vous. Toutefois, vous pouvez aussi juste configurer l’authentification Azure AD pour une ressource Lecteur immersif existante, si par exemple, vous en avez déjà créé une dans le portail Azure.

Pour certains clients, il peut être nécessaire de créer plusieurs ressources Lecteur immersif pour le développement et la production, ou éventuellement pour plusieurs régions différentes dans lesquelles votre service est déployé. Dans ce cas, vous pouvez revenir en arrière et utiliser le script plusieurs fois pour créer différentes ressources Lecteur immersif et les configurer avec les autorisations Azure AD.

Le script est conçu pour être flexible. Il recherche d’abord les ressources Lecteur immersif et Azure AD existantes dans votre abonnement, et les crée seulement si nécessaire, si elles n’existent pas déjà. Si c’est la première fois que vous créez une ressource Lecteur immersif, le script fait tout ce dont vous avez besoin. Si vous voulez l’utiliser uniquement afin de configurer Azure AD pour une ressource Lecteur immersif existante qui a été créée dans le portail, il le fera aussi. Il peut également être utilisé pour créer et configurer plusieurs ressources Lecteur immersif.

## <a name="set-up-powershell-environment"></a>Configurer l’environnement PowerShell

1. Commencez par ouvrir [Azure Cloud Shell](../../cloud-shell/overview.md). Assurez-vous que Cloud Shell est défini sur PowerShell dans la liste déroulante en haut à gauche, ou sinon tapez `pwsh`.

1. Copiez et collez l’extrait de code suivant dans le shell.

    ```azurepowershell-interactive
    function Create-ImmersiveReaderResource(
        [Parameter(Mandatory=$true, Position=0)] [String] $SubscriptionName,
        [Parameter(Mandatory=$true)] [String] $ResourceName,
        [Parameter(Mandatory=$true)] [String] $ResourceSubdomain,
        [Parameter(Mandatory=$true)] [String] $ResourceSKU,
        [Parameter(Mandatory=$true)] [String] $ResourceLocation,
        [Parameter(Mandatory=$true)] [String] $ResourceGroupName,
        [Parameter(Mandatory=$true)] [String] $ResourceGroupLocation,
        [Parameter(Mandatory=$true)] [String] $AADAppDisplayName="ImmersiveReaderAAD",
        [Parameter(Mandatory=$true)] [String] $AADAppIdentifierUri,
        [Parameter(Mandatory=$true)] [String] $AADAppClientSecret,
        [Parameter(Mandatory=$true)] [String] $AADAppClientSecretExpiration
    )
    {
        $unused = ''
        if (-not [System.Uri]::TryCreate($AADAppIdentifierUri, [System.UriKind]::Absolute, [ref] $unused)) {
            throw "Error: AADAppIdentifierUri must be a valid URI"
        }

        Write-Host "Setting the active subscription to '$SubscriptionName'"
        $subscriptionExists = Get-AzSubscription -SubscriptionName $SubscriptionName
        if (-not $subscriptionExists) {
            throw "Error: Subscription does not exist"
        }
        az account set --subscription $SubscriptionName

        $resourceGroupExists = az group exists --name $ResourceGroupName
        if ($resourceGroupExists -eq "false") {
            Write-Host "Resource group does not exist. Creating resource group"
            $groupResult = az group create --name $ResourceGroupName --location $ResourceGroupLocation
            if (-not $groupResult) {
                throw "Error: Failed to create resource group"
            }
            Write-Host "Resource group created successfully"
        }

        # Create an Immersive Reader resource if it doesn't already exist
        $resourceId = az cognitiveservices account show --resource-group $ResourceGroupName --name $ResourceName --query "id" -o tsv
        if (-not $resourceId) {
            Write-Host "Creating the new Immersive Reader resource '$ResourceName' (SKU '$ResourceSKU') in '$ResourceLocation' with subdomain '$ResourceSubdomain'"
            $resourceId = az cognitiveservices account create `
                            --name $ResourceName `
                            --resource-group $ResourceGroupName `
                            --kind ImmersiveReader `
                            --sku $ResourceSKU `
                            --location $ResourceLocation `
                            --custom-domain $ResourceSubdomain `
                            --query "id" `
                            -o tsv

            if (-not $resourceId) {
                throw "Error: Failed to create Immersive Reader resource"
            }
            Write-Host "Immersive Reader resource created successfully"
        }

        # Create an Azure Active Directory app if it doesn't already exist
        $clientId = az ad app show --id $AADAppIdentifierUri --query "appId" -o tsv
        if (-not $clientId) {
            Write-Host "Creating new Azure Active Directory app"
            $clientId = az ad app create --password $AADAppClientSecret --end-date "$AADAppClientSecretExpiration" --display-name $AADAppDisplayName --identifier-uris $AADAppIdentifierUri --query "appId" -o tsv

            if (-not $clientId) {
                throw "Error: Failed to create Azure Active Directory app"
            }
            Write-Host "Azure Active Directory app created successfully."
            Write-Host "NOTE: To manage your Active Directory app client secrets after this Immersive Reader Resource has been created please visit https://portal.azure.com and go to Home -> Azure Active Directory -> App Registrations -> $AADAppDisplayName -> Certificates and Secrets blade -> Client Secrets section" -ForegroundColor Yellow
        }

        # Create a service principal if it doesn't already exist
        $principalId = az ad sp show --id $AADAppIdentifierUri --query "objectId" -o tsv
        if (-not $principalId) {
            Write-Host "Creating new service principal"
            az ad sp create --id $clientId | Out-Null
            $principalId = az ad sp show --id $AADAppIdentifierUri --query "objectId" -o tsv

            if (-not $principalId) {
                throw "Error: Failed to create new service principal"
            }
            Write-Host "New service principal created successfully"
        }

        # Sleep for 5 seconds to allow the new service principal to propagate
        Write-Host "Sleeping for 5 seconds"
        Start-Sleep -Seconds 5

        Write-Host "Granting service principal access to the newly created Immersive Reader resource"
        $accessResult = az role assignment create --assignee $principalId --scope $resourceId --role "Cognitive Services User"
        if (-not $accessResult) {
            throw "Error: Failed to grant service principal access"
        }
        Write-Host "Service principal access granted successfully"

        # Grab the tenant ID, which is needed when obtaining an Azure AD token
        $tenantId = az account show --query "tenantId" -o tsv

        # Collect the information needed to obtain an Azure AD token into one object
        $result = @{}
        $result.TenantId = $tenantId
        $result.ClientId = $clientId
        $result.ClientSecret = $AADAppClientSecret
        $result.Subdomain = $ResourceSubdomain

        Write-Host "Success! " -ForegroundColor Green -NoNewline
        Write-Host "Save the following JSON object to a text file for future reference:"
        Write-Output (ConvertTo-Json $result)
    }
    ```

1. Exécutez la fonction `Create-ImmersiveReaderResource` en fournissant les paramètres appropriés.

    ```azurepowershell-interactive
    Create-ImmersiveReaderResource
      -SubscriptionName '<SUBSCRIPTION_NAME>' `
      -ResourceName '<RESOURCE_NAME>' `
      -ResourceSubdomain '<RESOURCE_SUBDOMAIN>' `
      -ResourceSKU '<RESOURCE_SKU>' `
      -ResourceLocation '<RESOURCE_LOCATION>' `
      -ResourceGroupName '<RESOURCE_GROUP_NAME>' `
      -ResourceGroupLocation '<RESOURCE_GROUP_LOCATION>' `
      -AADAppDisplayName '<AAD_APP_DISPLAY_NAME>' `
      -AADAppIdentifierUri '<AAD_APP_IDENTIFIER_URI>' `
      -AADAppClientSecret '<AAD_APP_CLIENT_SECRET>'
      -AADAppClientSecretExpiration '<AAD_APP_CLIENT_SECRET_EXPIRATION>'
    ```

    | Paramètre | Commentaires |
    | --- | --- |
    | SubscriptionName |Nom de l’abonnement Azure à utiliser pour votre ressource Lecteur immersif. Vous devez avoir un abonnement pour pouvoir créer une ressource. |
    | Nom_ressource |  Doit être de type alphanumérique et peut contenir « - », du moment que « - » ne soit pas le premier ou le dernier caractère. La longueur ne doit pas dépasser 63 caractères.|
    | ResourceSubdomain |Un sous-domaine personnalisé est nécessaire pour votre ressource Lecteur immersif. Le sous-domaine est utilisé par le kit SDK lors de l’appel du service Lecteur immersif pour lancer le Lecteur. Le sous-domaine doit être globalement unique. Le sous-domaine doit être de type alphanumérique et peut contenir « - », du moment que « - » ne soit pas le premier ou le dernier caractère. La longueur ne doit pas dépasser 63 caractères. Ce paramètre est facultatif si la ressource existe déjà. |
    | ResourceSKU |Options : `S0`. Visitez notre [page des prix de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/immersive-reader/) pour en savoir plus sur chaque référence SKU disponible. Ce paramètre est facultatif si la ressource existe déjà. |
    | ResourceLocation |Options : `eastus`, `eastus2`, `southcentralus`, `westus`, `westus2`, `australiaeast`, `southeastasia`, `centralindia`, `japaneast`, `northeurope`, `uksouth`, `westeurope`. Ce paramètre est facultatif si la ressource existe déjà. |
    | ResourceGroupName |Les ressources sont créées dans des groupes de ressources au sein d’abonnements. Donnez le nom d’un groupe de ressources existant. Si le groupe de ressources n’existe pas encore, un nouveau groupe avec ce nom est créé. |
    | ResourceGroupLocation |Si votre groupe de ressources n’existe pas, vous devez fournir un emplacement dans lequel créer le groupe. Pour trouver une liste d’emplacements, exécutez `az account list-locations`. Utilisez la propriété *name* (sans espaces) du résultat retourné. Ce paramètre est facultatif si votre groupe de ressources existe déjà. |
    | AADAppDisplayName |Nom d’affichage de l’application Azure Active Directory. Si une application Azure AD existante est introuvable, une nouvelle application avec ce nom est créée. Ce paramètre est facultatif si l’application Azure AD existe déjà. |
    | AADAppIdentifierUri |URI de l’application Azure AD. Si une application Azure AD existante est introuvable, une nouvelle application avec cet URI est créée. Par exemple : `https://immersivereaderaad-mycompany`. |
    | AADAppClientSecret |Mot de passe que vous créez et qui sera utilisé plus tard pour l’authentification lors de l’acquisition d’un jeton pour lancer le Lecteur immersif. Le mot de passe doit comporter au moins 16 caractères et contenir au moins 1 caractère spécial et 1 caractère numérique. Pour gérer les codes secrets du client d’application Azure AD après avoir créé cette ressource, visitez https://portal.azure.com et accédez à Accueil -> Azure Active Directory -> Inscriptions d’applications-> `[AADAppDisplayName]` -> Certificats et secrets -> section Secrets des clients (comme indiqué dans la capture d’écran « Gérer vos secrets d’application » ci-dessous). |
    | AADAppClientSecretExpiration |Date ou heure à laquelle votre `[AADAppClientSecret]` expire (par exemple, '2020-12-31T11:59:59+00:00' ou '2020-12-31'). |

    Gérer l’application et les secrets de votre application Azure AD

    ![Panneau Certificats et secrets du portail Azure](./media/client-secrets-blade.png)

1. Copiez la sortie JSON dans un fichier texte pour l’utiliser ultérieurement. La sortie doit se présenter comme suit.

    ```json
    {
      "TenantId": "...",
      "ClientId": "...",
      "ClientSecret": "...",
      "Subdomain": "..."
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

* Consulter le [guide de démarrage rapide Node.js](./quickstarts/client-libraries.md?pivots=programming-language-nodejs) pour voir ce que vous pouvez faire d’autre avec le SDK Lecteur immersif pour Node.js
* Consulter le [tutoriel Android](./tutorial-android.md) pour voir ce que vous pouvez faire d’autre avec le SDK Lecteur immersif en utilisant Java ou Kotlin pour Android
* Consulter le [tutoriel iOS](./tutorial-ios.md) pour voir ce que vous pouvez faire d’autre avec le SDK Lecteur immersif en utilisant Swift pour iOS
* Consulter le [didacticiel Python](./tutorial-python.md) pour voir ce que vous pouvez faire d’autre avec le kit SDK Lecteur immersif à l’aide de Python
* Explorer le [SDK Lecteur Immersif](https://github.com/microsoft/immersive-reader-sdk) et la [référence du SDK Lecteur immersif](./reference.md)