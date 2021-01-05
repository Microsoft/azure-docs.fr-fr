---
title: Mettre à jour le script de déploiement de modèle de Visual Studio pour utiliser Az PowerShell
description: Mettre à jour le script de déploiement de modèle de Visual Studio depuis AzureRM vers Az PowerShell
author: cweining
ms.topic: conceptual
ms.date: 01/31/2020
ms.author: cweining
ms.openlocfilehash: 357e0289f3237ed32b0801280316225ba5530282
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76963867"
---
# <a name="update-visual-studio-template-deployment-script-to-use-az-powershell-module"></a>Mettre à jour le script de déploiement de modèle de Visual Studio pour utiliser le module Az PowerShell

Visual Studio 16.4 prend en charge l’utilisation du module Az PowerShell dans le script de déploiement de modèle. Toutefois, Visual Studio n’installe pas automatiquement ce module. Pour utiliser le module Az, vous devez suivre quatre étapes :

1. [Désinstaller le module AzureRM](/powershell/azure/uninstall-az-ps#uninstall-the-azurerm-module)
1. [Installer le module Az](/powershell/azure/install-az-ps)
1. Mettre à jour Visual Studio vers la version 16.4
1. Mettre à jour le script de déploiement dans votre projet

## <a name="update-visual-studio-to-164"></a>Mettre à jour Visual Studio vers la version 16.4

Mettez à jour votre installation de Visual Studio vers la version 16.4 ou une version ultérieure. Pendant la mise à niveau, assurez-vous que le composant Azure PowerShell n’est pas activé. Comme vous avez installé le module Az par le biais de la galerie, vous ne souhaitez pas réinstaller le module AzureRM.

Si vous avez déjà effectué la mise à niveau vers la version 16.4 et que le composant Azure PowerShell était activé, vous pouvez le désinstaller en exécutant Visual Studio Installer. Ne sélectionnez pas le composant Azure PowerShell dans la charge de travail Azure ou sur la page des composants individuels.

## <a name="update-the-deployment-script-in-your-project"></a>Mettre à jour le script de déploiement dans votre projet

Remplacez toutes les occurrences de la chaîne « AzureRm » par « Az » dans votre script de déploiement. Vous pouvez consulter les révisions dans ce dépôt [Gist](https://gist.github.com/cweining/d2da2479418ea403499c4306dcf4f619) pour voir les changements. Pour plus d’informations sur la mise à niveau des scripts vers le module Az, consultez [Migrer Azure PowerShell depuis AzureRM vers Az](/powershell/azure/migrate-from-azurerm-to-az).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du projet Visual Studio, consultez [Création et déploiement des projets de groupes de ressources Azure via Visual Studio](create-visual-studio-deployment-project.md).
