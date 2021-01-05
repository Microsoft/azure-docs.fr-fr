---
title: Utilisation de Speech Services avec des points de terminaison privés
titleSuffix: Azure Cognitive Services
description: Guide pratique de l’utilisation de Speech Services avec des points de terminaison privés fournis par Azure Private Link
services: cognitive-services
author: alexeyo26
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/04/2020
ms.author: alexeyo
ms.openlocfilehash: 01a0171ed2b660fbabebf4276a74f8a3ea631bde
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516539"
---
# <a name="using-speech-services-with-private-endpoints-provided-by-azure-private-link"></a>Utilisation de Speech Services avec des points de terminaison privés fournis par Azure Private Link

[Azure Private Link](../../private-link/private-link-overview.md) vous permet de vous connecter à différents services PaaS dans Azure par le biais d’un [point de terminaison privé](../../private-link/private-endpoint-overview.md). Un point de terminaison privé est une adresse IP privée au sein d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md) et d’un sous-réseau spécifiques.

Cet article explique comment configurer et utiliser Private Link et des points de terminaison privés avec Azure Cognitive Speech Services. 

> [!NOTE]
> Cet article explique les spécificités de la configuration et de l’utilisation de Private Link avec Azure Cognitive Speech Services. Avant de poursuivre, familiarisez-vous avec l’article général sur l’[utilisation de réseaux virtuels avec Cognitive Services](../cognitive-services-virtual-networks.md).

L’activation d’une ressource Speech pour les scénarios de point de terminaison privé requiert l’exécution des tâches suivantes :
- [Créer un nom de domaine personnalisé de ressource Speech](#create-custom-domain-name)
- [Créer et configurer un ou plusieurs points de terminaison privés](#enabling-private-endpoints)
- [Ajuster les solutions et applications existantes](#using-speech-resource-with-custom-domain-name-and-private-endpoint-enabled)

Si vous décidez ultérieurement de supprimer tous les points de terminaison privés, mais de continuer à utiliser la ressource, effectuez les actions décrites dans [cette section](#using-speech-resource-with-custom-domain-name-without-private-endpoints).

## <a name="create-custom-domain-name"></a>Créer un nom de domaine personnalisé

Les points de terminaison privés requièrent l’utilisation de [noms de sous-domaines personnalisés Cognitive Services](../cognitive-services-custom-subdomains.md). Suivez les instructions ci-dessous pour en créer un pour votre ressource Speech.

> [!WARNING]
> Une ressource Speech avec un nom de domaine personnalisé activé utilise un autre moyen d’interagir avec Speech Services. Il est probable que vous deviez ajuster le votre code d’application pour des scénarios [avec point de terminaison privé](#using-speech-resource-with-custom-domain-name-and-private-endpoint-enabled) et [**sans** point de terminaison privé](#using-speech-resource-with-custom-domain-name-without-private-endpoints).
>
> L’opération d’activation de nom de domaine personnalisé n’est [**pas réversible**](../cognitive-services-custom-subdomains.md#can-i-change-a-custom-domain-name). La seule façon de revenir au [nom régional](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) consiste à créer une ressource Speech. 
>
> En particulier, dans les cas où votre ressource Speech dispose d’un grand nombre de modèles et projets personnalisés associés créés avec [Speech Studio](https://speech.microsoft.com/) nous recommandons **vivement** d’essayer la configuration avec une ressource test avant de modifier la ressource utilisée en production.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

- Accédez au [portail Azure](https://portal.azure.com/) et connectez-vous à votre compte Azure.
- Sélectionnez la ressource Speech requise.
- Sélectionnez *Mise en réseau* (groupe *Gestion des ressources*). 
- Sous l’onglet *Pare-feu et réseaux virtuels* (par défaut), cliquez sur le bouton **Générer un nom de domaine personnalisé**.
- Cela a pour effet d’ouvrir un nouveau panneau contenant des instructions pour créer un sous-domaine personnalisé unique pour votre ressource.
> [!WARNING]
> Après avoir créé un nom de domaine personnalisé, vous ne pouvez **pas** le modifier. Pour plus d’informations, consultez la section Avertissement précédente.
- Une fois l’opération terminée, vous pouvez sélectionner *Clés et points de terminaison* (groupe *Gestion des ressources*) et vérifier le nouveau nom de point de terminaison de votre ressource au format <p />`{your custom name}.cognitiveservices.azure.com`

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Cette section nécessite d’exécuter localement PowerShell version 7.x ou ultérieure avec le module Azure PowerShell version 5.1.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

Avant de poursuivre, exécutez `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="verify-custom-domain-name-availability"></a>Vérifier la disponibilité du nom de domaine personnalisé

Vous devez vérifier si le domaine personnalisé que vous souhaitez utiliser est libre. Nous allons utiliser la méthode [Vérifier la disponibilité du domaine](/rest/api/cognitiveservices/accountmanagement/checkdomainavailability/checkdomainavailability) de l’API REST Cognitive Services. Consultez les commentaires dans le bloc de code ci-dessous expliquant les étapes à suivre.

> [!TIP]
> Le code ci-dessous ne fonctionne **PAS** dans Azure Cloud Shell.

```azurepowershell
$subId = "Your Azure subscription Id"
$subdomainName = "custom domain name"

# Select the Azure subscription containing Speech resource
# If your Azure account has only one active subscription
# you can skip this step
Set-AzContext -SubscriptionId $subId

# Preparing OAuth token which is used in request
# to Cognitive Services REST API
$Context = Get-AzContext
$AccessToken = (Get-AzAccessToken -TenantId $Context.Tenant.Id).Token
$token = ConvertTo-SecureString -String $AccessToken -AsPlainText -Force

# Preparing and executing the request to Cognitive Services REST API
$uri = "https://management.azure.com/subscriptions/" + $subId + `
    "/providers/Microsoft.CognitiveServices/checkDomainAvailability?api-version=2017-04-18"
$body = @{
subdomainName = $subdomainName
type = "Microsoft.CognitiveServices/accounts"
}
$jsonBody = $body | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri $uri -ContentType "application/json" -Authentication Bearer `
    -Token $token -Body $jsonBody | Format-List
```
Si le nom souhaité est disponible, vous obtenez une réponse similaire à celle-ci :
```azurepowershell
isSubdomainAvailable : True
reason               :
type                 :
subdomainName        : my-custom-name
```
Si le nom est déjà utilisé, vous obtenez la réponse suivante :
```azurepowershell
isSubdomainAvailable : False
reason               : Sub domain name 'my-custom-name' is already used. Please pick a different name.
type                 :
subdomainName        : my-custom-name
```
## <a name="enabling-custom-domain-name"></a>Activation d’un nom de domaine personnalisé

Pour activer un nom de domaine personnalisé pour la ressource Speech sélectionnée, nous utilisons la cmdlet [Set-AzCognitiveServicesAccount](/powershell/module/az.cognitiveservices/set-azcognitiveservicesaccount). Consultez les commentaires dans le bloc de code ci-dessous expliquant les étapes à suivre.

> [!WARNING]
> Après avoir exécuté avec succès le code ci-dessous, vous allez créer un nom de domaine personnalisé pour votre ressource Speech. Il n’est **pas possible** de modifier ce nom. Pour plus d’informations, consultez la section Avertissement précédente.

```azurepowershell
$resourceGroup = "Resource group name where Speech resource is located"
$speechResourceName = "Your Speech resource name"
$subdomainName = "custom domain name"

# Select the Azure subscription containing Speech resource
# If your Azure account has only one active subscription
# you can skip this step
$subId = "Your Azure subscription Id"
Set-AzContext -SubscriptionId $subId

# Set the custom domain name to the selected resource
# WARNING! THIS IS NOT REVERSIBLE!
Set-AzCognitiveServicesAccount -ResourceGroupName $resourceGroup `
    -Name $speechResourceName -CustomSubdomainName $subdomainName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cette section nécessite la dernière version d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="verify-custom-domain-name-availability"></a>Vérifier la disponibilité du nom de domaine personnalisé

Vous devez vérifier si le domaine personnalisé que vous souhaitez utiliser est libre. Nous allons utiliser la méthode [Vérifier la disponibilité du domaine](/rest/api/cognitiveservices/accountmanagement/checkdomainavailability/checkdomainavailability) de l’API REST Cognitive Services. 

Copiez le bloc de code ci-dessous, insérez le nom de domaine personnalisé, puis enregistrez le tout dans le fichier `subdomain.json`.

```json
{
    "subdomainName": "custom domain name",
    "type": "Microsoft.CognitiveServices/accounts"
}
```

Copiez le fichier dans votre dossier actif ou chargez-le dans Azure Cloud Shell et exécutez la commande suivante (remplacez `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` par l’identifiant de votre abonnement Azure).

```azurecli-interactive
az rest --method post --url "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.CognitiveServices/checkDomainAvailability?api-version=2017-04-18" --body @subdomain.json
```
Si le nom souhaité est disponible, vous obtenez une réponse similaire à celle-ci :
```azurecli
{
  "isSubdomainAvailable": true,
  "reason": null,
  "subdomainName": "my-custom-name",
  "type": null
}
```

Si le nom est déjà utilisé, vous obtenez la réponse suivante :
```azurecli
{
  "isSubdomainAvailable": false,
  "reason": "Sub domain name 'my-custom-name' is already used. Please pick a different name.",
  "subdomainName": "my-custom-name",
  "type": null
}
```
## <a name="enabling-custom-domain-name"></a>Activation d’un nom de domaine personnalisé

Pour activer le nom de domaine personnalisé pour la ressource Speech sélectionnée, nous utilisons la commande [az cognitiveservices account update](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_update).

Sélectionnez l’abonnement Azure contenant la ressource Speech. Si votre compte Azure ne dispose que d’un seul abonnement actif, vous pouvez ignorer cette étape (remplacez `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` par l’identifiant de votre abonnement Azure).
```azurecli-interactive
az account set --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
Définissez le nom de domaine personnalisé pour la ressource sélectionnée. Remplacez les exemples de valeurs de paramètre par les valeurs réelles, puis exécutez la commande ci-dessous.
> [!WARNING]
> Une fois l’exécution de la commande ci-dessous terminée, vous allez créer un nom de domaine personnalisé pour votre ressource Speech. Il n’est **pas possible** de modifier ce nom. Pour plus d’informations, consultez la section Avertissement précédente.
```azurecli
az cognitiveservices account update --name my-speech-resource-name --resource-group my-resource-group-name --custom-domain my-custom-name
```

**_

## <a name="enabling-private-endpoints"></a>Activation de points de terminaison privés

Activez un point de terminaison privé à l’aide du portail Azure, d’Azure PowerShell ou d’Azure CLI.

Nous vous recommandons d’utiliser la [zone DNS privée](../../dns/private-dns-overview.md) attachée au réseau virtuel avec les mises à jour nécessaires pour les points de terminaison privés que nous créons par défaut pendant le processus d’approvisionnement. Toutefois, si vous utilisez votre propre serveur DNS, il se peut que vous devriez apporter des modifications supplémentaires à la configuration de votre DNS. Consultez la section [DNS pour points de terminaison privés](#dns-for-private-endpoints). La meilleure solution consiste à choisir la stratégie DNS _ *avant** d’approvisionner des points de terminaison privés pour une ressource Speech en production. Nous vous recommandons également d’effectuer des tests préliminaires, en particulier si vous utilisez votre propre serveur DNS.

Consultez les articles suivants pour créer un ou plusieurs points de terminaison privés. Les articles utilisent une application web en guise d’exemple de ressource à activer avec les points de terminaison privés. Utilisez plutôt les paramètres suivants :

| Paramètre             | Valeur                                    |
|---------------------|------------------------------------------|
| Type de ressource       | **Microsoft.CognitiveServices/accounts** |
| Resource            | **\<your-speech-resource-name>**         |
| Sous-ressource cible | **account**                              |

- [Créer un point de terminaison privé en utilisant le portail Azure](../../private-link/create-private-endpoint-portal.md)
- [Créer un point de terminaison privé en utilisant Azure PowerShell](../../private-link/create-private-endpoint-powershell.md)
- [Créer un point de terminaison privé en utilisant Azure CLI](../../private-link/create-private-endpoint-cli.md)

### <a name="dns-for-private-endpoints"></a>DNS pour points de terminaison privés

Familiarisez-vous avec les principes généraux de [DNS pour points de terminaison privés dans des ressources Cognitive Services](../cognitive-services-virtual-networks.md#dns-changes-for-private-endpoints). Vérifiez ensuite que votre configuration DNS fonctionne correctement (voir les sous-sections suivantes).

#### <a name="mandatory-check-dns-resolution-from-the-virtual-network"></a>(Vérification obligatoire). Résolution DNS à partir du réseau virtuel

Nous allons utiliser `my-private-link-speech.cognitiveservices.azure.com` comme exemple de nom DNS de ressource Speech pour cette section.

Connectez-vous à une machine virtuelle située dans le réseau virtuel auquel vous avez attaché votre point de terminaison privé. Ouvrez l’invite de commandes Windows ou l’interpréteur de commandes Bash, exécutez la commande « nslookup » et assurez-vous qu’elle résout correctement le nom de domaine personnalisé de votre ressource :
```dos
C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
Server:  UnKnown
Address:  168.63.129.16

Non-authoritative answer:
Name:    my-private-link-speech.privatelink.cognitiveservices.azure.com
Address:  172.28.0.10
Aliases:  my-private-link-speech.cognitiveservices.azure.com
```
Vérifiez que l’adresse IP résolue correspond à l’adresse de votre point de terminaison privé.

#### <a name="optional-check-dns-resolution-from-other-networks"></a>(Vérification facultative). Résolution DNS à partir d’autres réseaux

Cette vérification est nécessaire si vous envisagez d’utiliser la ressource Speech activée de votre point de terminaison privé en mode « hybride », c’est-à-dire si vous avez activé l’option d’accès *Tous les réseaux* ou *Réseaux sélectionnés et points de terminaison privés* dans la section *Mise en réseau* de votre ressource. Si vous envisagez d’accéder à la ressource en utilisant uniquement un point de terminaison privé, vous pouvez ignorer cette section.

Nous allons utiliser `my-private-link-speech.cognitiveservices.azure.com` comme exemple de nom DNS de ressource Speech pour cette section.

Sur un ordinateur connecté à un réseau à partir duquel vous autorisez l’accès à la ressource, ouvrez l’invite de commandes Windows ou l’interpréteur de commandes Bash, exécutez la commande « nslookup » et assurez-vous qu’elle résout correctement le nom de domaine personnalisé de votre ressource :
```dos
C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
Server:  UnKnown
Address:  fe80::1

Non-authoritative answer:
Name:    vnetproxyv1-weu-prod.westeurope.cloudapp.azure.com
Address:  13.69.67.71
Aliases:  my-private-link-speech.cognitiveservices.azure.com
          my-private-link-speech.privatelink.cognitiveservices.azure.com
          westeurope.prod.vnet.cog.trafficmanager.net
```

Notez que l’adresse IP résolue pointe vers un point de terminaison proxy de réseau virtuel qui est utilisé pour distribuer le trafic réseau vers la ressource Cognitive Services avec point de terminaison privé. Ce comportement est différent pour une ressource avec un nom de domaine personnalisé activé, mais *sans* point de terminaison privé configuré. Consultez [cette section](#dns-configuration).

## <a name="adjusting-existing-applications-and-solutions"></a>Ajustement d’applications et de solutions existantes 

Une ressource Speech avec un domaine personnalisé activé utilise un autre moyen pour interagir avec Speech Services. Cela est vrai pour une ressource Speech avec domaine personnalisé activé [avec](#using-speech-resource-with-custom-domain-name-and-private-endpoint-enabled) et [sans](#using-speech-resource-with-custom-domain-name-without-private-endpoints) point de terminaison privé. Cette section fournit les informations nécessaires pour les deux cas.

### <a name="using-speech-resource-with-custom-domain-name-and-private-endpoint-enabled"></a>Utilisation de ressource Speech avec nom de domaine personnalisé et point de terminaison privé activé

Une ressource Speech avec un nom de domaine personnalisé et un point de terminaison privé activé utilise un autre moyen d’interagir avec Speech Services. Cette section explique comment utiliser une telle ressource avec l’API REST Speech Services et le [Kit de développement logiciel (SDK) Speech](speech-sdk.md).

> [!NOTE]
> Notez qu’une ressource Speech sans point de terminaison privé mais avec un **nom de domaine personnalisé** activé utilise également un moyen spécial d’interagir avec Speech Services, mais que ce moyen diffère du scénario d’une ressource Speech avec point de terminaison privé. Si vous avez une telle ressource (par exemple, si vous aviez une ressource avec points de terminaison privés avant de décider de supprimer ceux-ci), consultez la [section correspondante](#using-speech-resource-with-custom-domain-name-without-private-endpoints).

#### <a name="speech-resource-with-custom-domain-name-and-private-endpoint-usage-with-rest-api"></a>Ressource Speech avec nom de domaine personnalisé et point de terminaison privé. Utilisation avec API REST

Pour cette section, nous allons utiliser `my-private-link-speech.cognitiveservices.azure.com` comme exemple de nom DNS de ressource Speech (domaine personnalisé).

##### <a name="note-on-speech-services-rest-api"></a>Remarque sur l’API REST Speech Services

La solution Speech Services offre une API REST pour la [reconnaissance vocale](rest-speech-to-text.md) et la [synthèse vocale](rest-text-to-speech.md). Pour le scénario avec point de terminaison privé, vous devez prendre en compte les aspects suivants.

La reconnaissance vocale a deux API REST distinctes. Chaque API sert un objectif différent, utilise des points de terminaison différents et nécessite une approche différente lorsqu’elle est utilisée dans un scénario avec point de terminaison privé.

Les API REST de reconnaissance vocale sont les suivantes :
- L’[API REST de reconnaissance vocale v3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30) est utilisée pour la [Transcription par lot](batch-transcription.md) et [Custom Speech](custom-speech-overview.md). La version v3.0 [succède à la version v2.0](/azure/cognitive-services/speech-service/migrate-v2-to-v3).
- [L’API REST de reconnaissance vocale pour audio court](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) est utilisée pour la transcription en ligne. 

L’utilisation de l’API REST de reconnaissance vocale pour audio court et de l’API REST de synthèse vocale dans le scénario avec point de terminaison privé est identique et équivalente au [cas d’usage du Kit de développement logiciel (SDK) Speech](#speech-resource-with-custom-domain-name-and-private-endpoint-usage-with-speech-sdk) décrit plus loin dans cet article. 

L’API REST de reconnaissance vocale v3.0 utilisant un jeu de points de terminaison différent, elle nécessite une approche différente pour le scénario avec point de terminaison privé.

Les deux cas sont décrits dans les sous-sections suivantes.


##### <a name="speech-to-text-rest-api-v30"></a>API REST de reconnaissance vocale v3.0

En général, les ressources Speech utilisent des [points de terminaison régionaux Cognitive Services](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) pour communiquer avec l’[API REST de synthèse vocale v3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30). Ces ressources ont le format d’affectation des noms suivant : <p/>`{region}.api.cognitive.microsoft.com`

Voici un exemple d’URL de demande :

```http
https://westeurope.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions
```
Après l’activation d’un domaine personnalisé pour une ressource Speech (nécessaire pour des points de terminaison privés), une telle ressource utilise le modèle de nom DNS suivant pour le point de terminaison d’API REST de base : <p/>`{your custom name}.cognitiveservices.azure.com`

Cela signifie que, dans notre exemple, le nom du point de terminaison d’API REST sera : <p/>`my-private-link-speech.cognitiveservices.azure.com`

Et l’exemple d’URL de demande ci-dessus doit être converti en :
```http
https://my-private-link-speech.cognitiveservices.azure.com/speechtotext/v3.0/transcriptions
```
Cette URL doit être accessible à partir du réseau virtuel avec le point de terminaison privé attaché (et la [résolution DNS correcte](#mandatory-check-dns-resolution-from-the-virtual-network)).

Donc, en général, après avoir activé un nom de domaine personnalisé pour une ressource Speech, vous devez remplacer le nom d’hôte dans toutes les URL de demande par le nouveau nom d’hôte du domaine personnalisé. Toutes les autres parties de la demande (comme le chemin d’accès `/speechtotext/v3.0/transcriptions` dans l’exemple ci-dessus) restent les mêmes.

> [!TIP]
> Certains clients ont développé des applications qui utilisent la partie région du nom DNS du point de terminaison régional (par exemple, pour envoyer la demande à la ressource Speech déployée dans la région Azure particulière).
>
> Le nom de domaine personnalisé de la ressource Speech ne contient **aucune information** sur la région où la ressource est déployée. Ainsi, la logique d’application décrite ci-dessus **ne fonctionne pas** et doit être modifiée.

##### <a name="speech-to-text-rest-api-for-short-audio-and-text-to-speech-rest-api"></a>API REST de reconnaissance vocale pour audio court et API REST de synthèse vocale

L’[API REST de reconnaissance vocale pour audio court](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) et l’[API REST de synthèse vocale](rest-text-to-speech.md) utilisent deux types de points de terminaison :
- [points de terminaison régionaux Cognitive Services](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) pour communiquer avec l’API REST Cognitive Services afin d’obtenir un jeton d’autorisation ;
- points de terminaison spéciaux pour toutes les autres opérations.

La description détaillée des points de terminaison spéciaux et de la façon dont leur URL doit être transformée pour une ressource Speech avec point de terminaison privé est fournie dans [cette sous-section](#general-principle) de la section « Utilisation avec le Kit de développement logiciel (SDK) Speech » ci-dessous. Le principe décrit pour le Kit de développement logiciel (SDK) s’applique également aux API REST de reconnaissance vocale (v1.0) et de synthèse vocale.

Consultez la sous-section mentionnée dans le paragraphe précédent et examinez l’exemple suivant (l’exemple décrit l’API REST de synthèse vocale ; l’utilisation de l’API REST de reconnaissance vocale pour audio court est rigoureusement identique).

> [!NOTE]
> Lorsque vous utilisez l’**API REST de reconnaissance vocale pour audio court** dans des scénarios avec point de terminaison privé, vous devez utiliser un jeton d’autorisation [transmis via](rest-speech-to-text.md#request-headers) `Authorization` [en-tête](rest-speech-to-text.md#request-headers) ; la transmission de la clé d’abonnement Speech au point de terminaison spécial via un en-tête `Ocp-Apim-Subscription-Key` **ne fonctionne pas** et génère l’erreur 401.

**Exemple d’utilisation de l’API REST de synthèse vocale.**

Nous allons utiliser la région Europe Ouest comme exemple de région Azure et `my-private-link-speech.cognitiveservices.azure.com` comme exemple de nom DNS de ressource Speech (domaine personnalisé). Le nom de domaine personnalisé `my-private-link-speech.cognitiveservices.azure.com` dans notre exemple appartient à la ressource Speech créée dans la région Europe Ouest.

Pour obtenir la liste des voix prises en charge dans la région, vous devez effectuer les deux opérations suivantes :

- Obtenir un jeton d’autorisation via
```http
https://westeurope.api.cognitive.microsoft.com/sts/v1.0/issuetoken
```
- À l’aide du jeton obtenu, obtenir la liste des voix via
```http
https://westeurope.tts.speech.microsoft.com/cognitiveservices/voices/list
```
(Pour plus d’informations sur les étapes ci-dessus, consultez la [documentation sur l’API REST de synthèse vocale](rest-text-to-speech.md))

Pour la ressource Speech avec point de terminaison privé, les URL de point de terminaison pour la même séquence d’opérations doivent être modifiées. La même séquence ressemble à ce qui suit :
- Obtenir un jeton d’autorisation via
```http
https://my-private-link-speech.cognitiveservices.azure.com/v1.0/issuetoken
```
(Consultez l’explication détaillée dans la sous-section [API REST de reconnaissance vocale v3.0](#speech-to-text-rest-api-v30) ci-dessus)
- À l’aide du jeton obtenu, obtenir la liste des voix via
```http
https://my-private-link-speech.cognitiveservices.azure.com/tts/cognitiveservices/voices/list
```
(pour une explication détaillée, consultez la sous-section [Principe général](#general-principle) de la section « Utilisation avec le Kit de développement logiciel (SDK) Speech » ci-dessous)

#### <a name="speech-resource-with-custom-domain-name-and-private-endpoint-usage-with-speech-sdk"></a>Ressource Speech avec nom de domaine personnalisé et point de terminaison privé. Utilisation avec le Kit de développement logiciel (SDK) Speech

L’utilisation du Kit de développement logiciel (SDK) pour des ressources Speech avec nom de domaine personnalisé et point de terminaison privé nécessite une révision et probablement des modifications de votre code d’application. Nous travaillons sur une prise en charge plus transparente du scénario avec point de terminaison privé.

Pour cette section, nous allons utiliser `my-private-link-speech.cognitiveservices.azure.com` comme exemple de nom DNS de ressource Speech (domaine personnalisé).

##### <a name="general-principle"></a>Principe général

Généralement, dans les scénarios de Kit de développement logiciel (SDK) (ainsi que dans les scénarios d’API REST de synthèse vocale), les ressources Speech utilisent les points de terminaison régionaux spéciaux pour différentes offres de service. Le format de nom DNS pour ces points de terminaison est le suivant : </p>`{region}.{speech service offering}.speech.microsoft.com`

Exemple : </p>`westeurope.stt.speech.microsoft.com`

Toutes les valeurs possibles pour la région (premier élément du nom DNS) sont répertoriées [ici](regions.md). Le tableau ci-dessous présente la valeur possible pour l’offre Speech Services (deuxième élément du nom DNS) :

| Valeur du nom DNS | Offre Speech Services                                    |
|----------------|-------------------------------------------------------------|
| `commands`     | [Commandes personnalisées](custom-commands.md)                       |
| `convai`       | [Transcription de conversation](conversation-transcription.md) |
| `s2s`          | [Traduction vocale](speech-translation.md)                 |
| `stt`          | [Reconnaissance vocale](speech-to-text.md)                         |
| `tts`          | [Synthèse vocale](text-to-speech.md)                         |
| `voice`        | [Custom Voice](how-to-custom-voice.md)                      |

Donc, dans l’exemple ci-dessus (`westeurope.stt.speech.microsoft.com`) correspond au point de terminaison de reconnaissance vocale dans la région Europe Ouest.

Les points de terminaison avec point de terminaison privé communiquant avec Speech Services via un proxy spécial, les **URL de connexion de point de terminaison doivent être modifiées**. Le principe suivant est appliqué : une URL de point de terminaison « standard » suit le modèle <p/>`{region}.{speech service offering}.speech.microsoft.com/{URL path}`

Vous devez la changer en : <p/>`{your custom name}.cognitiveservices.azure.com/{speech service offering}/{URL path}`

**Exemple 1.** L’application communique à l’aide de l’URL suivante (reconnaissance vocale à l’aide du modèle de base pour l’anglais des États-Unis dans la région Europe Ouest) : 
```
wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```

Pour l’utiliser dans le scénario avec point de terminaison privé quand le nom de domaine personnalisé de la ressource Speech est `my-private-link-speech.cognitiveservices.azure.com`, cette URL doit être modifiée comme suit :
```
wss://my-private-link-speech.cognitiveservices.azure.com/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```

Voyons cela de plus près :
- Le nom d’hôte `westeurope.stt.speech.microsoft.com` est remplacé par le nom d’hôte du domaine personnalisé `my-private-link-speech.cognitiveservices.azure.com`.
- Le deuxième élément du nom DNS d’origine (`stt`) devient le premier élément du chemin d’accès de l’URL et précède le chemin d’accès d’origine, de sorte que l’URL d’origine `/speech/recognition/conversation/cognitiveservices/v1?language=en-US` devient `/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US`.
 
**Exemple 2.** L’application communique à l’aide de l’URL suivante (synthèse vocale à l’aide d’un modèle vocal personnalisé dans la région Europe Ouest) : 
```http
https://westeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId=974481cc-b769-4b29-af70-2fb557b897c4
```
Pour l’utiliser dans le scénario avec point de terminaison privé quand le nom de domaine personnalisé de la ressource Speech est `my-private-link-speech.cognitiveservices.azure.com`, cette URL doit être modifiée comme suit : 
```http
https://my-private-link-speech.cognitiveservices.azure.com/voice/cognitiveservices/v1?deploymentId=974481cc-b769-4b29-af70-2fb557b897c4
```

Le même principe que dans l’Exemple 1 est appliqué mais, cette fois, l’élément clé est `voice`.

##### <a name="modifying-applications"></a>Modification d’applications

Pour appliquer le principe décrit dans la section précédente à votre code d’application, vous devez effectuer deux opérations majeures :

- Déterminez l’URL de point de terminaison que votre application utilise.
- Modifiez votre URL de point de terminaison comme décrit dans la section précédente, et créez votre instance de classe `SpeechConfig` à l’aide de cette URL modifiée explicitement.

###### <a name="determining-application-endpoint-url"></a>Détermination de l’URL de point de terminaison de l’application

- [Activez la journalisation pour votre application](how-to-use-logging.md), puis exécutez-la pour générer le journal.
- Dans le fichier journal, recherchez `SPEECH-ConnectionUrl`. La chaîne contient le paramètre `value` qui contient à son tour l’URL complète que votre application utilisait.

Exemple de ligne de fichier journal avec l’URL de point de terminaison :
```
(114917): 41ms SPX_DBG_TRACE_VERBOSE:  property_bag_impl.cpp:138 ISpxPropertyBagImpl::LogPropertyAndValue: this=0x0000028FE4809D78; name='SPEECH-ConnectionUrl'; value='wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?traffictype=spx&language=en-US'
```
Par conséquent, l’URL que l’application utilise dans cet exemple est :
```
wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```
###### <a name="creating-speechconfig-instance-using-full-endpoint-url"></a>Création d’une instance `SpeechConfig` à l’aide d’une URL de point de terminaison complète

Modifiez le point de terminaison que vous avez déterminé dans la section précédente en procédant de la manière décrite dans la sous-section [Principe général](#general-principle).

Vous devez à présent modifier la manière dont vous créez l’instance de `SpeechConfig`. Le plus souvent, l’application actuelle utilise quelque chose ressemblant à ceci :
```csharp
var config = SpeechConfig.FromSubscription(subscriptionKey, azureRegion);
```
Cela ne fonctionne pas pour une ressources Speech avec point de terminaison privé en raison des modifications de nom d’hôte et d’URL que nous avons décrites dans les sections précédentes. Si vous essayez d’exécuter votre application existante sans aucune modification à l’aide de la clé d’une ressource avec point de terminaison privé, vous obtenez une erreur d’authentification (401).

Pour que cela fonctionne, vous devez modifier la manière dont vous instanciez la classe `SpeechConfig`, et utiliser l’initialisation « à partir du point de terminaison »/« avec point de terminaison ». Supposons que les deux variables suivantes sont définies :
- `subscriptionKey` contenant la clé de la ressource Speech avec point de terminaison privé ;
- `endPoint` contenant l’URL **modifiée** complète du point de terminaison (utilisant le type requis par le langage de programmation correspondant). Dans notre exemple, cette variable doit contenir
```
wss://my-private-link-speech.cognitiveservices.azure.com/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```
Ensuite, nous devons instancier la classe `SpeechConfig` comme suit :
```csharp
var config = SpeechConfig.FromEndpoint(endPoint, subscriptionKey);
```
```cpp
auto config = SpeechConfig::FromEndpoint(endPoint, subscriptionKey);
```
```java
SpeechConfig config = SpeechConfig.fromEndpoint(endPoint, subscriptionKey);
```
```python
import azure.cognitiveservices.speech as speechsdk
speech_config = speechsdk.SpeechConfig(endpoint=endPoint, subscription=subscriptionKey)
```
```objectivec
SPXSpeechConfiguration *speechConfig = [[SPXSpeechConfiguration alloc] initWithEndpoint:endPoint subscription:subscriptionKey];
```
> [!TIP]
> Les paramètres de requête spécifiés dans l’URI de point de terminaison ne sont pas modifiés, même s’ils sont définis par d’autres API. Par exemple, si la langue de reconnaissance est définie dans l’URI en tant que paramètre de requête « language=en-US » et est également définie sur « ru-RU » via la propriété correspondante, le paramètre de langue dans l’URI est prioritaire, et la langue effective est « en-US ». Seuls les paramètres non spécifiés dans l’URI de point de terminaison peuvent être définis par d’autres API.

Après cette modification, votre application doit fonctionner avec les ressources Speech avec point de terminaison privé. Nous travaillons sur une prise en charge plus transparente du scénario avec point de terminaison privé.

### <a name="using-speech-resource-with-custom-domain-name-without-private-endpoints"></a>Utilisation d’une ressource Speech avec un nom de domaine personnalisé sans point de terminaison privé

Dans cet article, nous avons indiqué plusieurs fois que l’activation d’un domaine personnalisé pour une ressource Speech est **irréversible** et qu’une telle ressource utilise un autre moyen de communiquer avec Speech Services, comparée aux ressources « habituelles » (c’est-à-dire celles qui utilisent des [noms de point de terminaison régionaux](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints)).

Cette section explique comment utiliser une ressource Speech avec un nom de domaine personnalisé activé, mais **sans** point de terminaison privé avec l’API REST Speech Services et le [Kit de développement logiciel (SDK)](speech-sdk.md). Il peut s’agir d’une ressource qui a été utilisée dans un scénario avec point de terminaison privé, mais dont le ou les points de terminaison privés ont été supprimés.

#### <a name="dns-configuration"></a>Configuration DNS

Rappelez-vous qu’un nom DNS de domaine personnalisé de la ressource Speech avec point de terminaison privé est [résolu à partir de réseaux publics](#optional-check-dns-resolution-from-other-networks). Dans ce cas, l’adresse IP résolue pointe vers un point de terminaison proxy de réseau virtuel qui est utilisé pour distribuer le trafic réseau vers la ressource Cognitive Services avec point de terminaison privé.

Toutefois, quand **tous** les points de terminaison privés de la ressource sont supprimés (ou juste après l’activation du nom de domaine personnalisé), l’enregistrement CNAME de la ressource Speech est réapprovisionné et pointe désormais vers l’adresse IP du [point de terminaison régional Cognitive Services](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) correspondant.

La sortie de la commande `nslookup` doit donc ressembler à ceci :
```dos
C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
Server:  UnKnown
Address:  fe80::1

Non-authoritative answer:
Name:    apimgmthskquihpkz6d90kmhvnabrx3ms3pdubscpdfk1tsx3a.cloudapp.net
Address:  13.93.122.1
Aliases:  my-private-link-speech.cognitiveservices.azure.com
          westeurope.api.cognitive.microsoft.com
          cognitiveweprod.trafficmanager.net
          cognitiveweprod.azure-api.net
          apimgmttmdjylckcx6clmh2isu2wr38uqzm63s8n4ub2y3e6xs.trafficmanager.net
          cognitiveweprod-westeurope-01.regional.azure-api.net
```
Comparez-la avec la sortie de [cette section](#optional-check-dns-resolution-from-other-networks).

#### <a name="speech-resource-with-custom-domain-name-without-private-endpoints-usage-with-rest-api"></a>Ressource Speech avec nom de domaine personnalisé sans point de terminaison privé. Utilisation avec API REST

##### <a name="speech-to-text-rest-api-v30"></a>API REST de reconnaissance vocale v3.0

L’utilisation de l’API REST de reconnaissance vocale v3.0 est totalement identique à celle de [ressources Speech avec point de terminaison privé](#speech-to-text-rest-api-v30).

##### <a name="speech-to-text-rest-api-for-short-audio-and-text-to-speech-rest-api"></a>API REST de reconnaissance vocale pour audio court et API REST de synthèse vocale

Dans ce cas, l’utilisation de l’API REST de reconnaissance vocale pour audio court et de l’API REST de synthèse vocale ne diffère pas du cas général, à une exception près pour l’API REST de reconnaissance vocale pour audio court (voir la remarque ci-dessous). Les deux API doivent être utilisées comme décrit dans la documentation sur l’[API REST de reconnaissance vocale pour audio court](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) et l’[API REST de synthèse vocale](rest-text-to-speech.md).

> [!NOTE]
> Lorsque vous utilisez l’**API REST de reconnaissance vocale pour audio court** dans des scénarios de domaine personnalisé, vous devez utiliser un jeton d’autorisation [transmis via](rest-speech-to-text.md#request-headers) `Authorization` [en-tête](rest-speech-to-text.md#request-headers) ; la transmission de la clé d’abonnement Speech au point de terminaison spécial via un en-tête `Ocp-Apim-Subscription-Key` **ne fonctionne pas** et génère l’erreur 401.

#### <a name="speech-resource-with-custom-domain-name-without-private-endpoints-usage-with-speech-sdk"></a>Ressource Speech avec nom de domaine personnalisé sans point de terminaison privé. Utilisation avec le Kit de développement logiciel (SDK) Speech

L’utilisation du Kit de développement logiciel (SDK) pour des ressources Speech avec nom de domaine personnalisé **sans** point de terminaison privé nécessite une révision et probablement des modifications de votre code d’application. Notez que ces modifications **diffèrent** de celles apportées dans le cas d’une [ressources Speech avec point de terminaison privé](#speech-resource-with-custom-domain-name-and-private-endpoint-usage-with-speech-sdk). Nous travaillons sur une prise en charge plus transparente du scénario avec point de terminaison privé/domaine personnalisé.

Pour cette section, nous allons utiliser `my-private-link-speech.cognitiveservices.azure.com` comme exemple de nom DNS de ressource Speech (domaine personnalisé).

Dans la section traitant d’une [ressource Speech avec point de terminaison privé](#speech-resource-with-custom-domain-name-and-private-endpoint-usage-with-speech-sdk), nous avons expliqué comment déterminer l’URL de point de terminaison utilisée, la modifier et la faire fonctionner via une initialisation « à partir du point de terminaison »/« avec point de terminaison » de l’instance de classe `SpeechConfig`.

Toutefois, si vous tentez d’exécuter la même application après avoir supprimé tous les points de terminaison privés (en laissant passer un certain temps pour le réapprovisionnement de l’enregistrement DNS correspondant), vous obtenez une erreur de service interne (404). La raison à cela est que l’[enregistrement DNS](#dns-configuration) pointe désormais vers le point de terminaison Cognitive Services régional au lieu du proxy de réseau virtuel, et que des chemins d’URL tels que `/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US` y sont introuvables, d’où l’erreur « Introuvable » (404).

Si vous « restaurez » votre application vers l’instanciation « standard » de `SpeechConfig` dans le style de
```csharp
var config = SpeechConfig.FromSubscription(subscriptionKey, azureRegion);
```
votre application s’arrête avec l’erreur d’authentification (401).

##### <a name="modifying-applications"></a>Modification d’applications

Pour activer votre application pour le scénario de ressource Speech avec nom de domaine personnalisé sans point de terminaison privé, vous devez procéder comme suit :
- Demandez un jeton d’autorisation via l’API REST Cognitive Services.
- Instanciez la classe `SpeechConfig` à l’aide de la méthode « à partir du jeton d’autorisation »/« avec jeton d’autorisation ». 

###### <a name="requesting-authorization-token"></a>Demande de jeton d’autorisation

Consultez [cet article](../authentication.md#authenticate-with-an-authentication-token) pour savoir comment obtenir le jeton via l’API REST Cognitive Services. 

Utilisez votre nom de domaine personnalisé dans l’URL du point de terminaison. Dans notre exemple, l’URL est la suivante :
```http
https://my-private-link-speech.cognitiveservices.azure.com/sts/v1.0/issueToken
```
> [!TIP]
> Vous pouvez trouver cette URL dans la section *Clés et points de terminaison* (groupe *Gestion des ressources*) de votre ressource Speech dans le portail Azure.

###### <a name="creating-speechconfig-instance-using-authorization-token"></a>Création d’une instance `SpeechConfig` à l’aide d’un jeton d’autorisation

Vous devez instancier la classe `SpeechConfig` à l’aide du jeton d’autorisation que vous avez obtenu dans la section précédente. Supposons que les variables suivantes sont définies :

- `token` contenant le jeton d’autorisation obtenu dans la section précédente ;
- `azureRegion` contenant le nom de la ressource Speech [région](regions.md) (exemple : `westeurope`) ;
- `outError` (uniquement pour la cas [Objective C](/objectivec/cognitive-services/speech/spxspeechconfiguration#initwithauthorizationtokenregionerror)).

Ensuite, nous devons instancier la classe `SpeechConfig` comme suit :
```csharp
var config = SpeechConfig.FromAuthorizationToken(token, azureRegion);
```
```cpp
auto config = SpeechConfig::FromAuthorizationToken(token, azureRegion);
```
```java
SpeechConfig config = SpeechConfig.fromAuthorizationToken(token, azureRegion);
```
```python
import azure.cognitiveservices.speech as speechsdk
speech_config = speechsdk.SpeechConfig(auth_token=token, region=azureRegion)
```
```objectivec
SPXSpeechConfiguration *speechConfig = [[SPXSpeechConfiguration alloc] initWithAuthorizationToken:token region:azureRegion error:outError];
```
> [!NOTE]
> L’appelant doit s’assurer que le jeton d’autorisation est valide. Avant que le jeton d’autorisation expire, l’appelant doit l’actualiser en appelant cet méthode setter avec un nouveau jeton valide. Les valeurs de configuration étant copiées lors de la création d’un module de reconnaissance/synthétiseur, la nouvelle valeur de jeton ne s’applique pas aux modules de reconnaissance qui ont déjà été créés. Pour les modules de reconnaissance/synthétiseurs créés précédemment, vous devez définir le jeton d’autorisation du module de reconnaissance/synthétiseur correspondant pour actualiser le jeton. Dans le cas contraire, les modules de reconnaissance/synthétiseurs rencontreront des erreurs lors de la reconnaissance/synthèse.

Après cette modification, votre application doit fonctionner avec des ressources Speech avec nom de domaine personnalisé sans point de terminaison privé. Nous travaillons sur une prise en charge plus transparente du scénario avec point de terminaison privé/domaine personnalisé.

## <a name="pricing"></a>Tarifs

Pour plus d’informations sur les tarifs, consultez [Tarification Liaison privée Azure](https://azure.microsoft.com/pricing/details/private-link).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [Azure Private Link](../../private-link/private-link-overview.md)
* En savoir plus sur le [Kit de développement logiciel (SDK) Speech](speech-sdk.md)
* En savoir plus sur l’[API REST de reconnaissance vocale](rest-speech-to-text.md)
* En savoir plus sur l’[API REST de synthèse vocale](rest-text-to-speech.md)
