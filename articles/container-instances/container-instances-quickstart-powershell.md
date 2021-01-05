---
title: Guide de démarrage rapide – Déployer un conteneur Docker dans une instance de conteneur – PowerShell
description: Dans ce guide de démarrage rapide, vous utilisez Azure PowerShell pour déployer rapidement une application web conteneurisée qui s’exécute dans une instance de conteneur Azure isolé.
services: container-instances
manager: gwallace
ms.service: container-instances
ms.topic: quickstart
ms.date: 03/21/2019
ms.custom: seodec18, mvc
ms.openlocfilehash: c7002d8a83e58a9089ee3c3840b0397d63e2f198
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89565580"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-azure-powershell"></a>Démarrage rapide : Déployer une instance de conteneur dans Azure à l’aide d’Azure PowerShell

Utilisez Azure Container Instances pour exécuter, facilement et rapidement, des conteneurs Docker serverless dans Azure. Déployez une application sur une instance de conteneur à la demande lorsque vous n’avez pas besoin d’une plateforme d’orchestration de conteneur complète telle qu’Azure Kubernetes Service.

Dans cette procédure de démarrage rapide, vous utilisez Azure PowerShell pour déployer un conteneur Windows isolé et mettre son application à disposition avec un nom de domaine complet (FQDN). Quelques secondes après l’exécution d’une seule commande de déploiement, vous pouvez accéder à l’application en cours d’exécution dans le conteneur :

![Application déployée dans Azure Container Instances affichée dans le navigateur][qs-powershell-01]

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell pour les besoins de ce tutoriel. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Les instances de conteneur Azure, comme toutes les ressources Azure, doivent être déployées dans un groupe de ressources. Les groupes de ressources vous permettent d’organiser et de gérer les ressources Azure connexes.

Commencez par créer un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*, à l’aide de la commande [New-AzResourceGroup][New-AzResourceGroup] suivante :

 ```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

## <a name="create-a-container"></a>Créez un conteneur.

Maintenant que vous avez un groupe de ressources, vous pouvez exécuter un conteneur dans Azure. Pour créer une instance de conteneur avec Azure PowerShell, fournissez un nom de groupe de ressources, un nom d’instance de conteneur et l’image conteneur Docker à l’applet de commande [New-AzContainerGroup][New-AzContainerGroup]. Dans ce démarrage rapide, vous utilisez l’image `mcr.microsoft.com/windows/servercore/iis:nanoserver` publique. Cette image contient Microsoft Internet Information Services (IIS) à exécuter dans Nano Server.

Vous pouvez exposer vos conteneurs sur Internet en spécifiant un ou plusieurs ports à ouvrir, une étiquette de nom DNS ou les deux. Dans ce guide de démarrage rapide, vous déployez un conteneur avec une étiquette de nom DNS pour que IIS soit publiquement accessible.

Pour démarrer une instance de conteneur, exécutez une commande similaire à la suivante. Définissez une valeur `-DnsNameLabel` être unique au sein de la région Azure dans laquelle vous créez l’instance. Si vous recevez un message d’erreur « Étiquette de nom DNS indisponible », essayez d’utiliser une autre étiquette de nom DNS.

 ```azurepowershell-interactive
New-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer -Image mcr.microsoft.com/windows/servercore/iis:nanoserver -OsType Windows -DnsNameLabel aci-demo-win
```

Après quelques secondes, vous devez recevoir une réponse d’Azure. L’élément `ProvisioningState` du conteneur est initialement défini sur **Création**, mais doit passer à **Réussite** après une ou deux minutes. Vérifiez l’état du déploiement à l’aide de l’applet de commande [Get-AzContainerGroup][Get-AzContainerGroup] :

 ```azurepowershell-interactive
Get-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer
```

L’état d’approvisionnement, le nom de domaine complet et l’adresse IP du conteneur apparaissent dans la sortie de l’applet de commande :

```console
PS Azure:\> Get-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer


ResourceGroupName        : myResourceGroup
Id                       : /subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerInstance/containerGroups/mycontainer
Name                     : mycontainer
Type                     : Microsoft.ContainerInstance/containerGroups
Location                 : eastus
Tags                     :
ProvisioningState        : Creating
Containers               : {mycontainer}
ImageRegistryCredentials :
RestartPolicy            : Always
IpAddress                : 52.226.19.87
DnsNameLabel             : aci-demo-win
Fqdn                     : aci-demo-win.eastus.azurecontainer.io
Ports                    : {80}
OsType                   : Windows
Volumes                  :
State                    : Pending
Events                   : {}
```

Une fois que l’élément `ProvisioningState` du conteneur est défini sur **Réussite**, accédez à `Fqdn` dans votre navigateur. Si vous voyez une page web similaire à l’image suivante, félicitations ! Cela signifie que vous avez réussi à déployer une application s’exécutant dans un conteneur Docker sur Azure.

![IIS déployé à l’aide d’Azure Container Instances affiché dans un navigateur][qs-powershell-01]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous avez fini d’utiliser le conteneur, vous pouvez le supprimer avec l’applet de commande [Remove-AzContainerGroup][Remove-AzContainerGroup] :

 ```azurepowershell-interactive
Remove-AzContainerGroup -ResourceGroupName myResourceGroup -Name mycontainer
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une instance de conteneur Azure à partir d’une image dans le registre du Hub Docker public. Si vous voulez créer une image conteneur et la déployer à partir d’un registre de conteneurs Azure privé, passez au didacticiel Azure Container Instances.

> [!div class="nextstepaction"]
> [Didacticiel Azure Container Instances](./container-instances-tutorial-prepare-app.md)

<!-- IMAGES -->
[qs-powershell-01]: ./media/container-instances-quickstart-powershell/qs-powershell-01.png

<!-- LINKS -->
[New-AzResourceGroup]: /powershell/module/az.resources/new-Azresourcegroup
[New-AzContainerGroup]: /powershell/module/az.containerinstance/new-Azcontainergroup
[Get-AzContainerGroup]: /powershell/module/az.containerinstance/get-Azcontainergroup
[Remove-AzContainerGroup]: /powershell/module/az.containerinstance/remove-Azcontainergroup
