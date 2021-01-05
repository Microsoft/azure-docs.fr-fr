---
title: Utiliser des identités managées sur une machine virtuelle Azure pour se connecter – Azure AD
description: Instructions et exemples détaillés sur l’utilisation d’un principal du service d’identités managées pour ressources Azure pour la connexion cliente par script et l’accès aux ressources.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/01/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: 08fe856fd43baad4189d81f7743396a71cd5de48
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266321"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-for-sign-in"></a>Comment utiliser des identités managées pour ressources Azure sur une machine virtuelle Azure pour se connecter 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  
Cet article fournit des exemples de script PowerShell et CLI pour la connexion à l’aide du principal du service d’identités managées pour ressources Azure, et des conseils sur des sujets importants tels que la gestion des erreurs.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

Si vous envisagez d’utiliser les exemples de Azure PowerShell ou Azure CLI dans cet article, veillez à installer la dernière version de [Azure PowerShell](/powershell/azure/install-az-ps) ou bien [Azure CLI](/cli/azure/install-azure-cli). 

> [!IMPORTANT]
> - L’exemple de script présenté dans cet article part du principe que le client de ligne de commande s’exécute sur une machine virtuelle avec des identités managées pour ressources Azure activées. Utilisez la fonctionnalité « Se connecter » de machine virtuelle dans le portail Azure, pour vous connecter à distance à votre machine virtuelle. Pour plus d’informations sur l’activation d’identités managées pour ressources Azure sur une machine virtuelle, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](qs-configure-portal-windows-vm.md), ou l’une des variantes de cet article (en utilisant PowerShell, CLI, un modèle ou un Kit de développement logiciel (SDK) Azure). 
> - Pour éviter les erreurs lors de l’accès aux ressources, l’identité managée de la machine virtuelle doit comporter au moins l’accès « Lecture » à l’étendue appropriée (la machine virtuelle ou plus) pour autoriser les opérations d’Azure Resource Manager sur la machine virtuelle. Pour plus d’informations, voir [Attribuer l’accès aux identités managées pour ressources Azure à une ressource en utilisant le portail Azure](howto-assign-access-portal.md).

## <a name="overview"></a>Vue d’ensemble

Les identités managées pour ressources Azure fournissent un [objet principal du service](../develop/developer-glossary.md#service-principal-object), qui est [créé lors de l’activation d’identités managées pour ressources Azure](overview.md) sur la machine virtuelle. L’accès à des ressources Azure peut être donné au principal du service, et celui-ci peut être utilisé comme identité par des clients de script/ligne de commande pour la connexion et l’accès aux ressources. En règle générale, pour accéder à des ressources sécurisées sous sa propre identité, un client de script doit :  

   - être inscrit et consenti avec Azure AD comme une application cliente web/confidentielle
   - connectez-vous sous son principal du service, à l’aide des informations d’identification de l’application (normalement incorporées dans le script)

Avec des identités managées pour ressources Azure, votre client de script n’a plus besoin de faire quoi que ce soit, car il peut se connecter sous le principal du service d’identités managées pour ressources Azure. 

## <a name="azure-cli"></a>Azure CLI

Le script suivant montre comment :

1. Se connecter à Azure AD sous le principal du service d’identité managée pour ressources Azure de la machine virtuelle  
2. Appelez Azure Resource Manager et obtenez l’ID principal du service de la machine virtuelle. CLI prend en charge automatiquement la gestion de l’acquisition/utilisation des jetons pour vous. Veillez à indiquer le nom de votre machine virtuelle pour `<VM-NAME>`.  

   ```azurecli
   az login --identity
   
   spID=$(az resource list -n <VM-NAME> --query [*].identity.principalId --out tsv)
   echo The managed identity for Azure resources service principal ID is $spID
   ```

## <a name="azure-powershell"></a>Azure PowerShell

Le script suivant montre comment :

1. Se connecter à Azure AD sous le principal du service d’identité managée pour ressources Azure de la machine virtuelle  
2. Appelez une applet de commande de Azure Resource Manager pour obtenir des informations sur la machine virtuelle. PowerShell prend en charge la gestion de l’utilisation de jeton automatiquement pour vous.  

   ```azurepowershell
   Add-AzAccount -identity

   # Call Azure Resource Manager to get the service principal ID for the VM's managed identity for Azure resources. 
   $vmInfoPs = Get-AzVM -ResourceGroupName <RESOURCE-GROUP> -Name <VM-NAME>
   $spID = $vmInfoPs.Identity.PrincipalId
   echo "The managed identity for Azure resources service principal ID is $spID"
   ```

## <a name="resource-ids-for-azure-services"></a>ID de ressource pour les services Azure

Pour obtenir la liste des ressources qui prennent en charge Azure AD et qui ont été testées avec des identités managées pour ressources Azure et leurs ID de ressource respectifs, voir [Services Azure prenant en charge Azure AD Authentication](services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

## <a name="error-handling-guidance"></a>Conseil de gestion des erreurs 

Des réponses telles que les suivantes peuvent indiquer que l’identité managée pour ressources Azure de la machine virtuelle n’a pas été configurée correctement :

- PowerShell : *Invoke-WebRequest : Impossible de se connecter au serveur distant*
- Interface CLI : *MSI : Impossible de récupérer un jeton à partir de `http://localhost:50342/oauth2/token`, erreur 'HTTPConnectionPool(host='localhost', port=50342)* 

Si vous recevez l’une de ces erreurs, revenez à la machine virtuelle Azure dans le [portail Azure](https://portal.azure.com) et :

- Accédez à la page **Identité**, puis assurez-vous que l’option **Affecté(e) par le système** est définie sur « Oui ».
- Allez à la page **Extensions**, puis assurez-vous que l’extension d’identités managées pour ressources Azure **(prévue pour janvier 2019)** a bien été déployée.

Si l’un des paramétrages est incorrect, il se peut que vous deviez redéployer les identités managées pour ressources Azure sur votre ressource, ou résoudre le problème de déploiement. Si vous avez besoin d’aide pour configurer une machine virtuelle, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle en utilisant le portail Azure](qs-configure-portal-windows-vm.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour activer des identités managées pour ressources Azure sur une machine virtuelle Azure, voir [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant PowerShell](qs-configure-powershell-windows-vm.md) ou [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant Azure CLI](qs-configure-cli-windows-vm.md)