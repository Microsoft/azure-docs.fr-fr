---
title: Place de marché commerciale - Attribution de partenaires et de l’utilisation de client
description: Obtenir une vue d’ensemble du suivi de l’utilisation de client pour les solutions de la place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: vikrambmsft
ms.author: vikramb
ms.date: 11/4/2020
ms.custom: devx-track-terraform
ms.openlocfilehash: e9e630f4199e0bfb67509f008f403c0a7a490887
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436689"
---
# <a name="commercial-marketplace-partner-and-customer-usage-attribution"></a>Place de marché commerciale - Attribution de partenaires et de l’utilisation de client

L’attribution de l’utilisation de client est une méthode permettant d’associer des ressources Azure exécutées dans des abonnements clients, déployée pour exécuter votre solution avec vous en tant que partenaire. La formation de ces associations dans les systèmes Microsoft internes offre une meilleure visibilité de l’empreinte Azure exécutant vos logiciels. En optant pour cette capacité de suivi, vous vous alignez sur les équipes de vente Microsoft et bénéficiez d’un crédit pour les programmes de partenaires Microsoft.

Vous pouvez former cette association via la place de marché Azure, le référentiel de démarrage rapide, les référentiels GitHub privés et lors d’engagements clients individualisés (tels que le développement d’une application) qui créent une adresse IP durable.

L’attribution de l’utilisation de client prend en charge trois options de déploiement :

- via les modèles Azure Resource Manager : Les partenaires peuvent utiliser des modèles Resource Manager pour déployer les services Azure afin d’exécuter les logiciels du partenaire. Les partenaires peuvent créer un modèle Resource Manager qui définit l’infrastructure et la configuration de leur solution Azure. Un modèle Resource Manager vous permet, ainsi qu’à vos clients, de déployer votre solution tout au long de son cycle de vie. Vous pouvez être certain que vos ressources sont déployées dans un état cohérent.
- API Azure Resource Manager : les partenaires peuvent appeler directement les API Resource Manager pour déployer un modèle Resource Manager ou pour générer les appels d’API, afin de provisionner directement des services Azure.
- Terraform : Les partenaires peuvent utiliser Terraform pour déployer un modèle Resource Manager ou déployer directement des services Azure.

>[!IMPORTANT]
>- L’attribution de l’utilisation de client n’est pas destinée à suivre le travail des intégrateurs de systèmes, des fournisseurs de services gérés ou des outils conçus pour déployer et gérer des logiciels s’exécutant sur Azure.
>
>- L’attribution de l’utilisation de client est destinée aux nouveaux déploiements et ne prend PAS en charge le balisage des ressources existantes qui ont déjà été déployées.
>
>- L’attribution de l’utilisation de client est obligatoire pour les offres [Azure Application](./create-new-azure-apps-offer.md) publiées sur la place de marché Azure.
>
>- Tous les services Azure ne sont pas compatibles avec l’attribution de l’utilisation du client. Azure Kubernetes Services (AKS) et Virtual Machine Scale Sets présentent actuellement des problèmes connus qui entraînent la production de rapports d’utilisation insuffisants.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-guids"></a>Créer des GUID

Un GUID désigne un identificateur de référence unique à 32 chiffres hexadécimaux. Pour créer un GUID et effectuer un suivi, vous devez utiliser un générateur GUID, par exemple via PowerShell.

```powershell
[guid]::NewGuid()
```

Nous vous recommandons de créer un GUID unique pour chaque offre et canal de distribution de chaque produit. Si vous ne souhaitez pas que les rapports soient scindés, vous pouvez choisir d'utiliser un GUID unique pour les différents canaux de distribution du produit.

Si vous déployez un produit à l’aide d’un modèle et qu’il est disponible à la fois sur la Place de marché Azure et sur GitHub, vous pouvez créer et enregistrer deux GUID distincts :

- Produit A sur la Place de marché Azure
- Produit A sur GitHub

La création de rapports est assurée par Microsoft Partner Network ID et le GUID.

Vous pouvez également suivre l’utilisation à un niveau plus granulaire en inscrivant des GUID supplémentaires et en modifiant les GUID entre les plans, les plans étant des variantes d’une offre.

## <a name="register-guids"></a>Inscrire les GUID

Les GUID doivent être inscrits dans l’Espace partenaires pour activer l’attribution de l’utilisation de client.

Une fois que vous avez ajouté un GUID à votre modèle ou dans l’agent utilisateur et inscrit le GUID dans l’Espace partenaires, les déploiements futurs font l’objet d’un suivi.

> [!NOTE]
> Si vous publiez votre offre [Azure Application](./create-new-azure-apps-offer.md) sur la Place de marché Azure via l’Espace partenaires, tout nouveau GUID utilisé dans votre modèle est automatiquement inscrit dans votre profil de l’Espace partenaires au moment où le modèle est chargé.  

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard).

1. Inscrivez-vous en tant qu’[éditeur de la place de marché commerciale](https://aka.ms/JoinMarketplace).

   * Les partenaires doivent [avoir un profil dans l’Espace partenaires](./partner-center-portal/create-account.md). Vous êtes invité à répertorier l’offre dans la Place de marché Azure ou AppSource.
   * Les partenaires peuvent inscrire plusieurs GUID.
   * Les partenaires peuvent également enregistrer des GUID pour des modèles et des offres de solutions extérieures à la place de marché.

1. Cliquez sur l’icône d’engrenage **Settings** (Paramètres) dans le coin supérieur droit > **Account settings** (Paramètres du compte).

1. Dans **Organization profile** (Profil d’organisation) > **Identifiers** (Identificateurs), sélectionnez **Add Tracking GUID** (Ajouter un GUID de suivi).

1. Dans la zone **GUID**, entrez votre GUID de suivi. Entrez simplement le GUID sans le préfixe `pid-`. Dans la zone **Description**, entrez le nom ou la description de l’offre.

1. Pour inscrire plusieurs GUID, sélectionnez de nouveau **Add Tracking GUID** (Ajouter un GUID de suivi). Des champs supplémentaires apparaissent sur la page.

1. Sélectionnez **Enregistrer**.

## <a name="use-resource-manager-templates"></a>Utiliser des modèles Resource Manager
De nombreuses solutions de partenaires sont déployées à l’aide des modèles Azure Resource Manager. Si vous disposez d’un modèle Resource Manager sur la Place de marché Azure, sur GitHub ou dans le cadre d’un démarrage rapide, le processus de modification de votre modèle permettant d’autoriser l’attribution de l’utilisation de client est relativement simple.

> [!NOTE]
> Pour plus d’informations sur la création et la publication de modèles de solution, consultez
> * [Créer et déployer votre premier modèle Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).
>* [Offre d’application Azure](./create-new-azure-apps-offer.md).
>* Vidéo : [Création de modèles de solution et d’applications managées pour la Place de marché Azure](https://channel9.msdn.com/Events/Build/2018/BRK3603).


Pour ajouter un identificateur global unique (GUID), vous devez apporter une modification unique au fichier de modèle principal :

1. [Créer un GUID](#create-guids) à l’aide de la méthode conseillée et [enregistrer le GUID](#register-guids).

1. Ouvrez le modèle Resource Manager.

1. Ajoutez une nouvelle ressource de type [Microsoft. Resources/Deployments](/azure/templates/microsoft.resources/deployments) dans le fichier de modèle principal. Cette ressource doit être uniquement dans le fichier **mainTemplate.json** ou **azuredeploy.json**, et pas dans l’un des modèles imbriqués ou liés.

1. Entrez la valeur du GUID derrière le préfixe `pid-`, en tant que nom de la ressource. Par exemple, si le GUID est eb7927c8-dd66-43e1-b0cf-c346a422063, le nom de la ressource est _PID-eb7927c8-dd66-43e1-b0cf-c346a422063_.

1. Recherchez d’éventuelles erreurs dans le modèle.

1. Republiez le modèle dans les référentiels appropriés.

1. [Vérifiez que le GUID a fonctionné dans le déploiement du modèle](#verify-the-guid-deployment).

### <a name="sample-resource-manager-template-code"></a>Exemple de code de modèle Resource Manager

Pour activer les ressources de suivi pour votre modèle, vous devez ajouter la ressource supplémentaire suivante dans la section des ressources. Veillez à modifier l’exemple de code ci-dessous avec vos propres entrées lorsque vous l’ajoutez au fichier modèle principal.
Cette ressource doit être ajoutée uniquement dans le fichier **mainTemplate.json** ou **azuredeploy.json**, et pas dans l’un des modèles imbriqués ou liés.

```json
// Make sure to modify this sample code with your own inputs where applicable

{ // add this resource to the resources section in the mainTemplate.json (do not add the entire file)
    "apiVersion": "2020-06-01",
    "name": "pid-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX", // use your generated GUID here
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "resources": []
        }
    }
} // remove all comments from the file when complete
```

## <a name="use-the-resource-manager-apis"></a>Utiliser les API Resource Manager

Dans certains cas, les partenaires préfèrent effectuer directement des appels d’API REST Resource Manager afin de déployer des services Azure. [Azure prend en charge plusieurs kits SDK](../index.yml?pivot=sdkstools) pour permettre ces appels. Vous pouvez utiliser l’un des kits SDK ou appeler les API REST directement pour déployer des ressources.

Si vous utilisez un modèle Resource Manager, vous devez baliser votre solution en vous conformant aux instructions précédentes. Si vous n’utilisez pas un modèle Resource Manager et n’effectuez pas d’appels directs d’API, vous pouvez toujours baliser votre déploiement pour associer l’utilisation des ressources Azure.

### <a name="tag-a-deployment-with-the-resource-manager-apis"></a>Baliser un déploiement avec les API Resource Manager

Pour autoriser l’attribution de l’utilisation de client, lorsque vous concevez vos appels d’API, incluez un GUID dans l’en-tête d’agent utilisateur au sein de la demande. Ajoutez le GUID pour chaque offre ou référence SKU. Mettez la chaîne en forme avec le préfixe `pid-` et incluez le GUID généré par le partenaire. Voici un exemple de format GUID à insérer dans l’agent utilisateur :

![Exemple de format GUID](media/marketplace-publishers-guide/tracking-sample-guid-for-lu-2.PNG)

> [!NOTE]
> Le format de la chaîne est important. Si le préfixe `pid-` n’est pas inclus, il est impossible d’interroger les données. D’autres kits SDK procèdent au suivi différemment. Pour implémenter cette méthode, passez en revue la prise en charge et le suivi de votre kit SDK Azure préféré.

#### <a name="example-the-python-sdk"></a>Exemple : SDK Python

Pour Python, utilisez l’attribut **config**. Vous pouvez uniquement ajouter l’attribut à un UserAgent. Voici un exemple :

![Ajouter l’attribut à un agent utilisateur](media/marketplace-publishers-guide/python-for-lu.PNG)

> [!NOTE]
> Ajoutez l’attribut à chaque client. Il n’existe aucune configuration statique globale. Vous pouvez baliser une fabrique de clients pour vous assurer que chaque client effectue un suivi. Pour plus d’informations, consultez cet [exemple de fabrique de clients sur GitHub](https://github.com/Azure/azure-cli/blob/7402fb2c20be2cdbcaa7bdb2eeb72b7461fbcc30/src/azure-cli-core/azure/cli/core/commands/client_factory.py#L70-L79).

#### <a name="example-the-net-sdk"></a>Exemple : Le kit de développement logiciel (SDK) .NET

Pour .NET, veillez à définir l’agent utilisateur. La bibliothèque [Microsoft.Azure.Management.Fluent](/dotnet/api/microsoft.azure.management.fluent) peut être utilisée pour définir l’agent utilisateur avec le code suivant (exemple en C#) :

```csharp

var azure = Microsoft.Azure.Management.Fluent.Azure
    .Configure()
    // Add your pid in the user agent header
    .WithUserAgent("pid-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX", String.Empty) 
    .Authenticate(/* Credentials created via Microsoft.Azure.Management.ResourceManager.Fluent.SdkContext.AzureCredentialsFactory */)
    .WithSubscription("<subscription ID>");
```

#### <a name="tag-a-deployment-by-using-the-azure-powershell"></a>Baliser un déploiement à l’aide d’Azure PowerShell

Si vous déployez des ressources par le biais d’Azure PowerShell, ajoutez votre GUID à l’aide de la méthode suivante :

```powershell
[Microsoft.Azure.Common.Authentication.AzureSession]::ClientFactory.AddUserAgent("pid-eb7927c8-dd66-43e1-b0cf-c346a422063")
```

#### <a name="tag-a-deployment-by-using-the-azure-cli"></a>Baliser un déploiement à l’aide d’Azure CLI

Lorsque vous utilisez Azure CLI pour ajouter votre GUID, définissez la variable d’environnement **AZURE_HTTP_USER_AGENT**. Vous pouvez définir cette variable dans la portée d’un script. Vous pouvez également définir la variable de manière globale dans la portée de l’interpréteur de commandes :

```powershell
export AZURE_HTTP_USER_AGENT='pid-eb7927c8-dd66-43e1-b0cf-c346a422063'
```

Pour plus d’informations, consultez la page [Kit de développement logiciel (SDK) Azure pour Go](/azure/developer/go/).

## <a name="use-terraform"></a>Utiliser Terraform

La prise en charge pour Terraform est disponible via la version 1.21.0 du fournisseur Azure : [https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/CHANGELOG.md#1210-january-11-2019](https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/CHANGELOG.md#1210-january-11-2019).  Cette prise en charge s’applique à tous les partenaires qui déploient leur solution via Terraform ainsi qu’à toutes les ressources déployées et mesurées par le fournisseur Azure (version 1.21.0 ou ultérieure).

Le fournisseur Azure pour Terraform a ajouté un nouveau champ facultatif intitulé [*partner_id*](https://www.terraform.io/docs/providers/azurerm/#partner_id) vous permettant de spécifier le GUID de suivi que vous utilisez pour votre solution. La valeur de ce champ peut également être approvisionnée depuis la variable d’environnement *ARM_PARTNER_ID*.

```
provider "azurerm" {
          subscription_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          client_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          ……
          # new stuff for ISV attribution
          partner_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"}
```
Les partenaires qui souhaitent que leur déploiement via Terraform soit suivi en fonction de l’attribution de l’utilisation de client doivent procéder comme suit :

* Créez un GUID (le GUID doit être ajouté pour chaque offre ou référence SKU)
* Mettez à jour le fournisseur Azure pour définir la valeur de *partner_id* sur le GUID (n’ajoutez PAS le préfixe « pid- » au GUID, affectez uniquement le GUID tel qu’il est)

## <a name="verify-the-guid-deployment"></a>Vérifier le déploiement du GUID

Une fois que vous avez modifié votre modèle et effectué un déploiement de tests, utilisez le script PowerShell suivant pour récupérer les ressources déployées et balisées.

Vous pouvez utiliser ce script pour vérifier que le GUID a été ajouté avec succès à votre modèle Resource Manager. Le script ne s’applique pas au déploiement de l’API Resource Manager ou Terraform.

Connectez-vous à Azure. Sélectionnez l’abonnement avec le déploiement que vous souhaitez vérifier avant d’exécuter le script. Exécutez le script dans le contexte d’abonnement du déploiement.

Le **GUID** et le nom **resourceGroup** du déploiement sont des paramètres requis.

Vous pouvez obtenir [le script d’origine](https://gist.github.com/bmoore-msft/ae6b8226311014d6e7177c5127c7eba1#file-verify-deploymentguid-ps1) sur GitHub.

```powershell
Param(
    [GUID][Parameter(Mandatory=$true)]$guid,
    [string][Parameter(Mandatory=$true)]$resourceGroupName
)

# Get the correlationId of the pid deployment

$correlationId = (Get-AzResourceGroupDeployment -ResourceGroupName
$resourceGroupName -Name "pid-$guid").correlationId

# Find all deployments with that correlationId

$deployments = Get-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName | Where-Object{$_.correlationId -eq $correlationId}

# Find all deploymentOperations in a deployment by name
# PowerShell doesn't surface outputResources on the deployment
# or correlationId on the deploymentOperation

foreach ($deployment in $deployments){

# Get deploymentOperations by deploymentName
# then the resourceId for any create operation

($deployment | Get-AzResourceGroupDeploymentOperation | Where-Object{$_.properties.provisioningOperation -eq "Create" -and $_.properties.targetResource.resourceType -ne "Microsoft.Resources/deployments"}).properties.targetResource.id

}
```

## <a name="report"></a>Rapport

Vous trouverez le rapport relatif à l’attribution de l’utilisation de client dans le tableau de bord de votre Espace partenaires ([https://partner.microsoft.com/dashboard/partnerinsights/analytics/overview](https://partner.microsoft.com/dashboard/partnerinsights/analytics/overview)). Pour afficher le rapport, vous devez connecter à l’aide des informations d’identification de votre Espace partenaires. Si vous rencontrez des problèmes de rapport ou de connexion, créez une [demande de support](#get-support).

## <a name="notify-your-customers"></a>Informer vos clients

Les partenaires doivent informer leurs clients des déploiements utilisant l’attribution de l’utilisation de client. Microsoft signale au partenaire l’utilisation d’Azure associée à ces déploiements. Les exemples suivants incluent le contenu que vous pouvez utiliser pour informer vos clients de ces déploiements. Dans les exemples, remplacez \<PARTNER> par le nom de votre entreprise. Les partenaires doivent s’assurer que la notification s’aligne sur leurs stratégies de confidentialité et de collecte de données, y compris sur les options relatives aux clients à exclure du suivi.

### <a name="notification-for-resource-manager-template-deployments"></a>Notification des déploiements de modèle Resource Manager

Quand vous déployez ce modèle, Microsoft peut identifier l’installation du logiciel \<PARTNER> avec les ressources Azure déployées. Microsoft peut mettre en corrélation les ressources Azure utilisées pour prendre en charge le logiciel. Microsoft collecte ces informations pour fournir les meilleures expériences possibles avec leurs produits et pour gérer leur activité. Les données sont collectées et régies par les stratégies de confidentialité de Microsoft, qui se trouvent à l’adresse [https://www.microsoft.com/trustcenter](https://www.microsoft.com/trustcenter).

### <a name="notification-for-sdk-or-api-deployments"></a>Notification de déploiement des kits SDK ou d’API

Quand vous déployez le logiciel \<PARTNER>, Microsoft peut identifier l’installation du logiciel \<PARTNER> avec les ressources Azure déployées. Microsoft peut mettre en corrélation les ressources Azure utilisées pour prendre en charge le logiciel. Microsoft collecte ces informations pour fournir les meilleures expériences possibles avec leurs produits et pour gérer leur activité. Les données sont collectées et régies par les stratégies de confidentialité de Microsoft, qui se trouvent à l’adresse [https://www.microsoft.com/trustcenter](https://www.microsoft.com/trustcenter).

## <a name="get-support"></a>Obtenir de l’aide

Découvrez les options de support disponibles via la place de marché commerciale dans [Support technique pour le programme Place de marché commerciale dans l’Espace partenaires](support.md).

### <a name="how-to-submit-a-technical-consultation-request"></a>Comment envoyer une demande de consultation technique

1. Visitez [Services techniques des partenaires](https://aka.ms/TechnicalJourney).
1. Sélectionnez Gestion et infrastructure cloud, puis une nouvelle page affichant le parcours technique s’ouvrira pour vous.
1. Sous Services de déploiement, cliquez sur le bouton Envoyer une demande.
1. Connectez-vous à l’aide de votre MSA (compte MPN) ou de votre AAD (compte Tableau de bord du partenaire). En fonction de vos informations d’identification de connexion, un formulaire de demande en ligne s’affichera :
    * Renseignez/vérifiez les informations de contact.
    * Les informations de consultation peuvent être préremplies. Sinon sélectionnez-les dans les listes déroulantes.
    * Entrez un titre et la description du problème (fournissez autant de détails que possible).

1. Cliquez sur Envoyer.

Consultez des instructions pas à pas avec captures d’écran dans [Utilisation des services techniques de prévente et de déploiement](https://aka.ms/TechConsultInstructions).

### <a name="whats-next"></a>Étapes suivantes

Vous serez contacté par un consultant technique partenaire Microsoft afin d’organiser un appel destiné à évaluer l’étendue de vos besoins.

## <a name="faq"></a>Questions fréquentes (FAQ)

**Quel avantage l’ajout du GUID apporte-t-il au modèle ?**

Microsoft permet aux partenaires de visualiser les déploiements de leurs solutions par les clients, ainsi que les informations détaillées sur leur utilisation influencée. Microsoft et le partenaire peuvent utiliser ces informations pour favoriser un engagement plus proche entre les équipes de vente. Microsoft et le partenaire peuvent utiliser ces données pour obtenir une vue plus cohérente de l’impact d’un partenaire individuel sur la croissance Azure.

**Une fois ajouté, un GUID peut-il être modifié ?**

Oui, un client ou un partenaire d’implémentation peut personnaliser le modèle et modifier ou supprimer le GUID. Nous conseillons aux partenaires de décrire de façon proactive le rôle de la ressource et le GUID à leurs clients et partenaires pour empêcher la suppression ou la modification du GUID. La modification du GUID affecte uniquement les ressources et les déploiements nouveaux, qui n’existent pas.

**Puis-je effectuer le suivi de modèles déployés à partir d’un dépôt autre que Microsoft tel que GitHub ?**

Oui, tant que le GUID est présent lorsque le modèle est déployé, l’utilisation fait l’objet d’un suivi. Les partenaires doivent toujours inscrire leurs GUID.

**Le client reçoit-il également la création de rapports ?**

Les clients peuvent effectuer le suivi de leur utilisation des ressources individuelles ou des groupes de ressources définis par le client sur le Portail Azure. Les clients ne voient pas l’utilisation détaillée par le GUID.

**Est-ce que cette méthode est similaire au partenaire de référence numérique (DPOR) ?**

Cette nouvelle méthode de connexion du déploiement et de l’utilisation à la solution d’un partenaire fournit un mécanisme permettant de lier une solution de partenaire à l’utilisation d’Azure. Le partenaire de référence numérique (DPOR) est destiné à associer un partenaire de conseil (intégrateur de systèmes) ou de gestion (fournisseur de services managés) à l’abonnement Azure d’un client.

**Puis-je utiliser un disque dur virtuel privé personnalisé dans le cadre d’une offre de modèle de solution sur la Place de marché Azure ?**

Non, vous ne pouvez pas. L’image de machine virtuelle doit provenir de la Place de marché Azure. Consultez [Guide de publication d’offres de machine virtuelle sur la Place de marché Azure](marketplace-virtual-machines.md).

Vous pouvez créer une offre de machine virtuelle dans la place de marché à l’aide de votre disque dur virtuel personnalisé et le marquer comme privé afin que personne ne puisse le voir. Ajoutez ensuite une référence à cette machine virtuelle dans votre modèle de solution.

**La mise à jour de la propriété *contentVersion* du modèle principal a échoué ?**

Il s’agit probablement d’un bogue quand le modèle est déployé à l’aide d’un élément TemplateLink issu d’un autre modèle, qui attend une version plus ancienne de contentVersion pour une raison quelconque. La solution de contournement consiste à utiliser la propriété de métadonnées :

```
"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "metadata": {
        "contentVersion": "1.0.1.0"
    },
    "parameters": {
```
