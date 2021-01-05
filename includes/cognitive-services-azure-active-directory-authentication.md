---
author: erhopf
ms.author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 05/11/2020
ms.openlocfilehash: 1085daca153431a28fdcc2583d0e31308214bf91
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95560968"
---
## <a name="authenticate-with-azure-active-directory"></a>S’authentifier à l’aide d’Azure Active Directory

> [!IMPORTANT]
> 1. Actuellement, **seule** l’API Vision par ordinateur, l’API Visage, l’API Analyse de texte, le Lecteur immersif, Form Recognizer, le Détecteur d’anomalies et tous les services Bing, à l’exception de Recherche personnalisée Bing, prennent en charge l’authentification à l’aide d’Azure Active Directory (AAD).
> 2. L’authentification AAD doit être toujours utilisée avec le nom de sous-domaine personnalisé de votre ressource Azure. Les [points de terminaison régionaux](../articles/cognitive-services/cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) ne prennent pas en charge l’authentification AAD.

Dans les sections précédentes, nous vous avons montré comment vous authentifier auprès d’Azure Cognitive Services en utilisant une clé d’abonnement monoservice ou multiservice. Bien que ces clés offrent un moyen simple et rapide de démarrer le développement, elles ne conviennent pas dans des scénarios plus complexes qui nécessitent le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Voyons ce qui est nécessaire pour s’authentifier avec Azure Active Directory (AAD).

Dans les sections suivantes, vous allez utiliser l’environnement Azure Cloud Shell ou Azure CLI pour créer un sous-domaine, affecter des rôles et obtenir un jeton de porteur pour appeler Azure Cognitive Services. Si vous êtes bloqué, des liens sont fournis dans chaque section avec toutes les options disponibles pour chaque commande dans Azure Cloud Shell/Azure CLI.

### <a name="create-a-resource-with-a-custom-subdomain"></a>Créer une ressource avec un sous-domaine personnalisé

La première étape consiste à créer un sous-domaine personnalisé. Si vous voulez utiliser une ressource Cognitive Services existante n’ayant pas de nom de sous-domaine personnalisé, suivez les instructions de la rubrique [Sous-domaines personnalisés de Cognitive Services](../articles/cognitive-services/cognitive-services-custom-subdomains.md#how-does-this-impact-existing-resources) pour activer un sous-domaine personnalisé pour votre ressource.

1. Commencez par ouvrir Azure Cloud Shell. Ensuite, [sélectionnez un abonnement](/powershell/module/az.accounts/set-azcontext?view=azps-3.3.0) :

   ```powershell-interactive
   Set-AzContext -SubscriptionName <SubscriptionName>
   ```

2. Ensuite, [créez une ressource Cognitive Services](/powershell/module/az.cognitiveservices/new-azcognitiveservicesaccount?view=azps-1.8.0) avec un sous-domaine personnalisé. Le nom de sous-domaine doit être globalement unique et ne peut pas inclure de caractères spéciaux, comme : « . », « ! », « , ».

   ```powershell-interactive
   $account = New-AzCognitiveServicesAccount -ResourceGroupName <RESOURCE_GROUP_NAME> -name <ACCOUNT_NAME> -Type <ACCOUNT_TYPE> -SkuName <SUBSCRIPTION_TYPE> -Location <REGION> -CustomSubdomainName <UNIQUE_SUBDOMAIN>
   ```

3. Si l’opération réussit, le **point de terminaison** doit afficher le nom du sous-domaine unique pour votre ressource.


### <a name="assign-a-role-to-a-service-principal"></a>Attribuer un rôle à un principal de service

Maintenant que vous disposez d’un sous-domaine personnalisé associé à votre ressource, vous devez affecter un rôle à un principal du service.

> [!NOTE]
> Gardez à l’esprit que la propagation des attributions de rôles Azure peut prendre cinq minutes.

1. Nous allons d’abord inscrire une [application AAD](/powershell/module/Az.Resources/New-AzADApplication?view=azps-1.8.0).

   ```powershell-interactive
   $SecureStringPassword = ConvertTo-SecureString -String <YOUR_PASSWORD> -AsPlainText -Force

   $app = New-AzADApplication -DisplayName <APP_DISPLAY_NAME> -IdentifierUris <APP_URIS> -Password $SecureStringPassword
   ```

   Vous aurez besoin de l’**ApplicationId** à l’étape suivante.

2. Ensuite, vous devez [créer un principal du service](/powershell/module/az.resources/new-azadserviceprincipal?view=azps-1.8.0) pour l’application AAD.

   ```powershell-interactive
   New-AzADServicePrincipal -ApplicationId <APPLICATION_ID>
   ```

   >[!NOTE]
   > Si vous inscrivez une application dans le portail Azure, cette étape est effectuée pour vous.

3. La dernière étape consiste à [affecter le rôle « Utilisateur Cognitive Services »](/powershell/module/az.Resources/New-azRoleAssignment?view=azps-1.8.0) au principal du service (délimité à la ressource). En affectant un rôle, vous accordez au principal du service l’accès à cette ressource. Vous pouvez accorder au même principal du service l’accès à plusieurs ressources de votre abonnement.
   >[!NOTE]
   > L’ObjectId du principal du service est utilisé, et non pas l’ObjectId de l’application.
   > ACCOUNT_ID est l’ID de ressource Azure du compte Cognitive Services que vous avez créé. Cet ID est indiqué dans les « propriétés » de la ressource dans le portail Azure.

   ```azurecli-interactive
   New-AzRoleAssignment -ObjectId <SERVICE_PRINCIPAL_OBJECTID> -Scope <ACCOUNT_ID> -RoleDefinitionName "Cognitive Services User"
   ```

### <a name="sample-request"></a>Exemple de requête

Dans cet exemple, un mot de passe est utilisé pour authentifier le principal du service. Le jeton fourni est ensuite utilisé pour appeler l’API Vision par ordinateur.

1. Obtenez votre **TenantId** :
   ```powershell-interactive
   $context=Get-AzContext
   $context.Tenant.Id
   ```

2. Obtenez un jeton :
   > [!NOTE]
   > Si vous utilisez Azure Cloud Shell, la classe `SecureClientSecret` n’est pas disponible. 

   #### <a name="powershell"></a>[PowerShell](#tab/powershell)
   ```powershell-interactive
   $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList "https://login.windows.net/<TENANT_ID>"
   $secureSecretObject = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.SecureClientSecret" -ArgumentList $SecureStringPassword   
   $clientCredential = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential" -ArgumentList $app.ApplicationId, $secureSecretObject
   $token=$authContext.AcquireTokenAsync("https://cognitiveservices.azure.com/", $clientCredential).Result
   $token
   ```
   
   #### <a name="azure-cloud-shell"></a>[Azure Cloud Shell](#tab/azure-cloud-shell)
   ```Azure Cloud Shell
   $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList "https://login.windows.net/<TENANT_ID>"
   $clientCredential = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential" -ArgumentList $app.ApplicationId, <YOUR_PASSWORD>
   $token=$authContext.AcquireTokenAsync("https://cognitiveservices.azure.com/", $clientCredential).Result
   $token
   ``` 

   ---

3. Appelez l’API Vision par ordinateur :
   ```powershell-interactive
   $url = $account.Endpoint+"vision/v1.0/models"
   $result = Invoke-RestMethod -Uri $url  -Method Get -Headers @{"Authorization"=$token.CreateAuthorizationHeader()} -Verbose
   $result | ConvertTo-Json
   ```

Le principal du service peut également être authentifié avec un certificat. En plus du principal du service, l’utilisateur principal est également pris en charge en ayant des autorisations déléguées via une autre application AAD. Dans ce cas, au lieu de mots de passe ou de certificats, les utilisateurs sont invités à fournir une authentification à deux facteurs lors de l’acquisition du jeton.

## <a name="authorize-access-to-managed-identities"></a>Autoriser l'accès aux identités managées
 
Cognitive Services prend en charge l’authentification Azure Active Directory (Azure AD) avec des [identités managées pour ressources Azure](../articles/active-directory/managed-identities-azure-resources/overview.md). Les identités managées pour ressources Azure peuvent autoriser l’accès à des ressources Cognitive Services en utilisant les informations d’identification Azure AD d’applications s’exécutant dans des machines virtuelles Azure, des applications de fonction, le service Virtual Machine Scale Sets et d’autres services. En utilisant des identités managées pour ressources Azure et Azure AD Authentication, vous pouvez éviter de stocker des informations d’identification avec les applications qui s’exécutent dans le cloud.  

### <a name="enable-managed-identities-on-a-vm"></a>Activer les identités managées sur une machine virtuelle

Avant de pouvoir utiliser les identités managées pour ressources Azure en vue d’autoriser les ressources Cognitive Services à partir de votre machine virtuelle, vous devez activer les identités managées pour ressources Azure sur la machine virtuelle. Pour savoir comment activer des identités managées pour ressources Azure, consultez :

- [Azure portal](../articles/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../articles/active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../articles/active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Modèle Azure Resource Manager](../articles/active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Bibliothèques clientes Azure Resource Manager](../articles/active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

Pour plus d’informations sur les identités managées, consultez [Identités managées pour les ressources Azure](../articles/active-directory/managed-identities-azure-resources/overview.md).