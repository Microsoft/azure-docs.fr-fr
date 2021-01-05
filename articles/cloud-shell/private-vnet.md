---
title: Cloud Shell dans un réseau virtuel Azure
description: Déployer Cloud Shell dans un réseau virtuel Azure
services: Azure
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/15/2020
ms.author: damaerte
ms.openlocfilehash: 722d935c242a51ddfc01377676f026b71a8951b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89468536"
---
# <a name="deploy-cloud-shell-into-an-azure-virtual-network"></a>Déployer Cloud Shell dans un réseau virtuel Azure
> [!NOTE]
> Cette fonctionnalité n’existe qu’en préversion publique.

Une session Cloud Shell normale s’exécute dans le conteneur d’un réseau Microsoft distinct de vos ressources. Cela signifie que les commandes qui s’exécutent dans le conteneur ne peuvent pas accéder aux ressources qui sont accessibles uniquement à partir d’un réseau virtuel spécifique. Par exemple, vous ne pouvez pas utiliser SSH pour vous connecter depuis Cloud Shell à une machine virtuelle qui dispose uniquement d’une adresse IP privée ni utiliser kubectl pour vous connecter à un cluster Kubernetes dont l’accès est verrouillé. 

Cette fonctionnalité facultative répond à ces limitations et vous permet de déployer Cloud Shell dans un réseau virtuel Azure que vous contrôlez. De là, le conteneur est capable d’interagir avec les ressources du réseau virtuel que vous sélectionnez.  

Vous pouvez voir ci-dessous l’architecture des ressources qui sera déployée et utilisée dans ce scénario.

![Illustre l’architecture de réseau virtuel isolé de Cloud Shell.](media/private-vnet/data-diagram.png)

Avant de pouvoir utiliser Cloud Shell dans votre propre réseau virtuel Azure, vous devrez créer plusieurs ressources pour prendre en charge cette fonctionnalité. Cet article explique comment configurer les ressources requises à l’aide d’un modèle Resource Manager.

> [!NOTE]
> Ces ressources ne doivent être configurées qu’une seule fois pour le réseau virtuel. Elles peuvent ensuite être partagées par tous les administrateurs ayant accès au réseau virtuel.

## <a name="required-network-resources"></a>Ressources réseau requises

### <a name="virtual-network"></a>Réseau virtuel
Un réseau virtuel définit l’espace d’adressage dans lequel un ou plusieurs sous-réseaux sont créés.

Le réseau virtuel à utiliser pour Cloud Shell doit être identifié. Il s’agit généralement d’un réseau virtuel existant qui contient des ressources que vous souhaitez gérer ou un réseau qui s’appaire à des réseaux qui contiennent vos ressources.

### <a name="subnet"></a>Subnet
Dans le réseau virtuel sélectionné, un sous-réseau dédié doit être utilisé pour les conteneurs Cloud Shell. Le sous-réseau est délégué au service Azure Container Instances (ACI).  Lorsqu’un utilisateur demande un conteneur Cloud Shell dans un réseau virtuel, Cloud Shell utilise ACI pour créer un conteneur dans ce sous-réseau délégué.  Aucune autre ressource ne peut être créée dans ce sous-réseau.

### <a name="network-profile"></a>Profil réseau
Un profil réseau est un modèle de configuration réseau pour les ressources Azure qui spécifie certaines propriétés réseau pour la ressource.

### <a name="azure-relay"></a>Azure Relay
Une instance [Azure Relay](../azure-relay/relay-what-is-it.md) permet à deux points de terminaison qui ne sont pas directement accessibles de communiquer. Ici, elle est utilisée pour permettre au navigateur de l’administrateur de communiquer avec le conteneur dans le réseau privé.

L’instance Azure Relay utilisée pour Cloud Shell peut être configurée pour contrôler quels réseaux peuvent accéder aux ressources des conteneurs : 
- Accessible à partir de l’Internet public : Dans cette configuration, Cloud Shell offre un moyen d’atteindre des ressources internes depuis l’extérieur. 
- Accessible à partir des réseaux spécifiés : Dans cette configuration, les administrateurs doivent accéder au portail Azure à partir d’un ordinateur fonctionnant dans le réseau approprié pour pouvoir utiliser Cloud Shell.

## <a name="storage-requirements"></a>Exigences de stockage
Comme lors de l’utilisation standard de Cloud Shell, un compte de stockage est requis lors de l’utilisation de Cloud Shell dans un réseau virtuel. Chaque administrateur a besoin d’un partage de fichiers pour stocker ses fichiers.  Le compte de stockage doit être accessible à partir du réseau virtuel utilisé par Cloud Shell. 

## <a name="virtual-network-deployment-limitations"></a>Limitations concernant le déploiement de réseau virtuel
* En raison des ressources réseau supplémentaires impliquées, le démarrage de Cloud Shell dans un réseau virtuel est généralement plus lent qu’une session Cloud Shell standard.

* Pendant la préversion, moins de régions sont prises en charge par Cloud Shell dans un réseau virtuel. Le service est actuellement limité à : USA Ouest et USA Centre-Ouest.

* [Azure Relay](../azure-relay/relay-what-is-it.md) n’est pas un service gratuit, veuillez consulter la [grille tarifaire](https://azure.microsoft.com/pricing/details/service-bus/). Dans le scénario de Cloud Shell, une connexion hybride est utilisée par chaque administrateur lorsque celui-ci utilise Cloud Shell. La connexion est automatiquement arrêtée une fois la session Cloud Shell terminée.

## <a name="register-the-resource-provider"></a>Inscrire le fournisseur de ressources

Le fournisseur de ressources Microsoft.ContainerInstances doit être inscrit dans l’abonnement qui contient le réseau virtuel que vous souhaitez utiliser. Sélectionnez l’abonnement approprié avec `Set-AzContext -Subscription {subscriptionName}`, puis exécutez :

```powershell
PS> Get-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance | select ResourceTypes,RegistrationState

ResourceTypes                             RegistrationState
-------------                             -----------------
{containerGroups}                         Registered
...
```

Si **RegistrationState** est `Registered`, aucune action n’est requise. S’il est `NotRegistered`, exécutez `Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance`. 

## <a name="deploy-network-resources"></a>Déployer des ressources réseau
 
### <a name="create-a-resource-group-and-virtual-network"></a>Créer un groupe de ressources et un réseau virtuel
Si vous disposez déjà d’un réseau virtuel auquel vous souhaitez vous connecter, ignorez cette section.

Dans le portail Azure, ou en utilisant Azure CLI, Azure PowerShell, etc., créez un groupe de ressources et un réseau virtuel dans le nouveau groupe de ressources ; **le groupe de ressources et le réseau virtuel doivent se trouver dans la même région**.

> [!NOTE]
> En préversion publique, le groupe de ressources et le réseau virtuel doivent se trouver dans les régions USA Centre-Ouest ou USA Ouest.

### <a name="arm-templates"></a>Modèles ARM
Utilisez le [modèle de démarrage rapide Azure](https://aka.ms/cloudshell/docs/vnet/template) permettant de créer des ressources Cloud Shell dans un réseau virtuel et le [modèle de démarrage rapide Azure](https://aka.ms/cloudshell/docs/vnet/template/storage) permettant de créer le stockage nécessaire. Prenez note des noms de vos ressources, principalement le nom de votre partage de fichiers.

### <a name="open-relay-firewall"></a>Ouvrir le pare-feu du relais
Accédez au relais créé à l’aide du modèle ci-dessus, sélectionnez « Mise en réseau » dans les paramètres et autorisez l’accès au relais à partir du réseau de votre navigateur. Par défaut, le relais est uniquement accessible à partir du réseau virtuel dans lequel il a été créé. 

### <a name="configuring-cloud-shell-to-use-a-virtual-network"></a>Configuration de Cloud Shell pour utiliser un réseau virtuel
> [!NOTE]
> Vous devez effectuer cette étape pour chaque administrateur qui utilisera Cloud Shell.

Après avoir effectué les étapes ci-dessus, accédez à Cloud Shell dans le portail Azure ou à l’adresse https://shell.azure.com. L’une de ces expériences doit être utilisée chaque fois que vous souhaitez vous connecter à une expérience Cloud Shell isolée.

> [!NOTE]
> Si Cloud Shell a été utilisé dans le passé, le clouddrive existant doit être démonté. Pour ce faire, exécutez `clouddrive unmount` à partir d’une session Cloud Shell active, puis actualisez votre page.

Connectez-vous à Cloud Shell, vous serez alors invité à effectuer la première exécution. Sélectionnez votre expérience d’interpréteur de commandes préférée, sélectionnez « Afficher les paramètres avancés », puis cochez la case « Afficher les paramètres d’isolement du réseau virtuel ». Renseignez les champs de la fenêtre contextuelle.  La plupart des champs se remplissent automatiquement selon les ressources disponibles qui peuvent être associées à Cloud Shell dans un réseau virtuel.  Le nom du partage de fichiers doit être rempli par l’utilisateur.


![Illustre les paramètres de la première expérience de réseau virtuel isolé de Cloud Shell.](media/private-vnet/vnet-settings.png)

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur les réseaux virtuels Azure](../virtual-network/virtual-networks-overview.md)