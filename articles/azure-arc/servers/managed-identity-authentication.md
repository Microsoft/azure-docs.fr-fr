---
title: S’authentifier auprès de ressources Azure au moyen de serveurs avec Arc
description: Cet article décrit la prise en charge d’Azure Instance Metadata Service pour les serveurs avec Arc et la façon dont vous pouvez vous authentifier auprès de ressources Azure et locales à l’aide d’un secret.
ms.topic: conceptual
ms.date: 12/09/2020
ms.openlocfilehash: 49b70928ae972da8e0a0d14d711e4b6f246cca6a
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96939015"
---
# <a name="authenticate-against-azure-resources-with-arc-enabled-servers"></a>S’authentifier auprès de ressources Azure au moyen de serveurs avec Arc

Les applications ou les processus qui s’exécutent directement sur des serveurs avec Azure Arc peuvent tirer parti des identités managées pour accéder à d’autres ressources Azure qui prennent en charge l’authentification basée sur Azure Active Directory. Une application peut obtenir un [jeton d’accès](../../active-directory/develop/developer-glossary.md#access-token) représentant son identité, qui est affectée par le système pour les serveurs avec Arc, et l’utiliser en tant que jeton de « porteur » pour s’authentifier auprès d’un autre service.

Reportez-vous à la [documentation de présentation des identités managées](../../active-directory/managed-identities-azure-resources/overview.md) pour obtenir une description détaillée des identités managées, ainsi que la distinction entre les identités affectées par le système et celles affectées par l’utilisateur.

Dans cet article, nous vous montrons comment un serveur peut utiliser une identité managée affectée par le système pour accéder à Azure [Key Vault](../../key-vault/general/overview.md). En agissant comme un amorçage, Key Vault permet à votre application cliente d’utiliser ensuite un secret pour accéder à des ressources non sécurisées par Azure Active Directory (AD). Par exemple, les certificats TLS/SSL utilisés par vos serveurs web IIS peuvent être stockés dans Azure Key Vault et être déployés de façon sécurisée sur des serveurs Windows ou Linux en dehors d’Azure.

## <a name="security-overview"></a>Présentation de la sécurité

Lors de l’intégration de votre serveur à des serveurs avec Azure Arc, plusieurs actions sont effectuées pour configurer l’utilisation d’une identité managée, à l’instar de ce qui est effectué pour une machine virtuelle Azure :

- Azure Resource Manager reçoit une demande pour activer l’identité managée attribuée par le système sur le serveur avec Arc.

- Azure Resource Manager crée un principal de service dans Azure AD pour représenter l’identité du serveur. Le principal de service est créé dans le locataire Azure AD approuvé par cet abonnement.

- Azure Resource Manager configure l’identité sur le serveur en mettant à jour le point de terminaison d’identité Azure Instance Metadata Service (IMDS) pour [Windows](../../virtual-machines/windows/instance-metadata-service.md) ou [Linux](../../virtual-machines/linux/instance-metadata-service.md) avec l’ID client et le certificat du principal de service. Le point de terminaison est un point de terminaison REST accessible uniquement à partir du serveur à l’aide d’une adresse IP non routable connue. Ce service fournit un sous-ensemble d’informations de métadonnées sur le serveur avec Arc pour faciliter sa gestion et sa configuration.

L’environnement d’un serveur avec identité managée est configuré avec les variables suivantes sur un serveur Windows avec Arc :

- **IMDS_ENDPOINT** : adresse IP du point de terminaison IMDS `http://localhost:40342` pour les serveurs avec Arc.

- **IDENTITY_ENDPOINT** : point de terminaison localhost correspondant à l’identité managée `http://localhost:40342/metadata/identity/oauth2/token` du service.

Le code exécuté sur le serveur peut demander un jeton au point de terminaison d’Azure Instance Metadata Service, qui est accessible uniquement à partir du serveur.

La variable d’environnement système **IDENTITY_ENDPOINT** permet la découverte du point de terminaison d’identité par les applications. Les applications doivent essayer de récupérer les valeurs **IDENTITY_ENDPOINT** et **IMDS_ENDPOINT** et les utiliser. Les applications avec n’importe quel niveau d’accès sont autorisées à effectuer des demandes aux points de terminaison. Les réponses de métadonnées sont gérées normalement et accordées à n’importe quel processus sur la machine. Toutefois, quand une demande effectuée est susceptible d’exposer un jeton, nous exigeons que le client fournisse un secret pour attester qu’il est en mesure d’accéder aux données uniquement disponibles pour les utilisateurs disposant de privilèges plus élevés.

## <a name="prerequisites"></a>Prérequis

- Compréhension des identités managées.
- Un serveur connecté et inscrit auprès des serveurs avec Arc.
- Vous êtes membre du [groupe Propriétaire](../../role-based-access-control/built-in-roles.md#owner)** dans l’abonnement ou le groupe de ressources, pour effectuer les étapes de création de ressource et de gestion de rôles nécessaires.
- Un coffre de clés Azure pour stocker et récupérer vos informations d’identification, et pour attribuer l’accès à l’identité Azure Arc au coffre de clés.

    - Si vous n’avez pas de coffre de clés créé, consultez [Créer un coffre de clés](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad.md#create-a-key-vault-).
    - Pour configurer l’accès par l’identité managée utilisée par le serveur, consultez [Accorder l’accès pour Linux](../../active-directory/managed-identities-azure-resources/tutorial-linux-vm-access-nonaad.md#grant-access) ou [Accorder l’accès pour Windows](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad.md#grant-access). Pour l’étape 5, vous allez entrer le nom du serveur avec Arc. Pour effectuer cette opération à l’aide de PowerShell, consultez [Attribuer une stratégie d’accès à l’aide de PowerShell](../../key-vault/general/assign-access-policy-powershell.md).

## <a name="acquiring-an-access-token-using-rest-api"></a>Acquisition d’un jeton d’accès à l’aide de l’API REST

La méthode permettant d’obtenir et d’utiliser une identité managée affectée par le système pour s’authentifier auprès de ressources Azure est similaire à celle suivie avec une machine virtuelle Azure.

Pour un serveur Windows avec Arc, à l’aide de PowerShell, vous appelez la requête web afin d’obtenir le jeton de l’hôte local dans le port spécifique. Spécifiez la requête à l’aide de l’adresse IP ou de la variable d’environnement **IDENTITY_ENDPOINT**.

```powershell
$apiVersion = "2020-06-01"
$resource = "https://management.azure.com/"
$endpoint = "{0}?resource={1}&api-version={2}" -f $env:IDENTITY_ENDPOINT,$resource,$apiVersion
$secretFile = ""
try
{
    Invoke-WebRequest -Method GET -Uri $endpoint -Headers @{Metadata='True'} -UseBasicParsing
}
catch
{
    $wwwAuthHeader = $_.Exception.Response.Headers["WWW-Authenticate"]
    if ($wwwAuthHeader -match "Basic realm=.+")
    {
        $secretFile = ($wwwAuthHeader -split "Basic realm=")[1]
    }
}
Write-Host "Secret file path: " $secretFile`n
$secret = cat -Raw $secretFile
$response = Invoke-WebRequest -Method GET -Uri $endpoint -Headers @{Metadata='True'; Authorization="Basic $secret"} -UseBasicParsing
if ($response)
{
    $token = (ConvertFrom-Json -InputObject $response.Content).access_token
    Write-Host "Access token: " $token
}
```

La réponse suivante est un exemple qui est retourné :

:::image type="content" source="media/managed-identity-authentication/powershell-token-output-example.png" alt-text="Récupération réussie du jeton d’accès à l’aide de PowerShell":::

Pour un serveur Linux avec Arc, à l’aide de Bash, vous appelez la requête web afin d’obtenir le jeton de l’hôte local dans le port spécifique. Spécifiez la requête suivante à l’aide de l’adresse IP ou de la variable d’environnement **IDENTITY_ENDPOINT**. Pour effectuer cette étape, vous avez besoin d’un client SSH.

```bash
ChallengeTokenPath=$(curl -s -D - -H Metadata:true "http://127.0.0.1:40342/metadata/identity/oauth2/token?api-version=2019-11-01&resource=https%3A%2F%2Fmanagement.azure.com" | grep Www-Authenticate | cut -d "=" -f 2 | tr -d "[:cntrl:]")
ChallengeToken=$(cat $ChallengeTokenPath)
if [ $? -ne 0 ]; then
    echo "Could not retrieve challenge token, double check that this command is run with root privileges."
else
    curl -s -H Metadata:true -H "Authorization: Basic $ChallengeToken" "http://127.0.0.1:40342/metadata/identity/oauth2/token?api-version=2019-11-01&resource=https%3A%2F%2Fmanagement.azure.com"
fi
```

La réponse suivante est un exemple qui est retourné :

:::image type="content" source="media/managed-identity-authentication/bash-token-output-example.png" alt-text="Récupération réussie du jeton d’accès à l’aide de Bash":::

La réponse inclut le jeton d’accès dont vous avez besoin pour accéder à une ressource dans Azure. Pour effectuer la configuration de l’authentification auprès d’Azure Key Vault, consultez [Accéder à Key Vault avec Windows](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad.md#access-data) ou [Accéder à Key Vault avec Linux](../../active-directory/managed-identities-azure-resources/tutorial-linux-vm-access-nonaad.md#access-data).

## <a name="next-steps"></a>Étapes suivantes

- Pour en découvrir plus sur Azure Key Vault, consultez [Vue d’ensemble de Key Vault](../../key-vault/general/overview.md).

- Découvrez comment affecter à une identité managée l’accès à une ressource Azure à l’aide de [PowerShell](../../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md) ou d’[Azure CLI](../../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md).