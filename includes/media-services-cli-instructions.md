---
title: Fichier include
description: Fichier include
services: media-services
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 01/28/2019
ms.author: juliako
ms.custom: include file
ms.openlocfilehash: 1327ac49920af353d6adbbe592ab4189417e60e8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996891"
---
## <a name="use-cli-shell"></a>Utiliser l’interpréteur de commandes CLI

Il est recommandé d’utiliser [Azure Cloud Shell](../articles/cloud-shell/overview.md?view=azure-cli-latest) pour exécuter des commandes CLI. **Cloud Shell** est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour effectuer les étapes mentionnées dans cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Il vous donne la possibilité de choisir l’expérience d’interpréteur de commandes la plus adaptée à votre façon de travailler. Les utilisateurs Linux peuvent choisir une expérience Bash, et les utilisateurs Windows l’option PowerShell.

Vous pouvez également installer l’interface CLI localement. Consultez [Installer Azure CLI](/cli/azure/install-azure-cli) pour obtenir des instructions pour votre plateforme.

### <a name="sign-in"></a>Se connecter

L’utilisation d’une installation locale de l’interface CLI exige une connexion à Azure. Cette étape n’est pas obligatoire pour Azure Cloud Shell. Connectez-vous avec la commande `az login`.

Si l’interface CLI peut ouvrir votre navigateur par défaut, elle le fera et chargera une page de connexion par la même occasion. Sinon, vous devez ouvrir une page de navigateur et suivre les instructions sur la ligne de commande pour entrer un code d’autorisation après avoir accédé à https://aka.ms/devicelogin dans votre navigateur.

### <a name="specify-location-of-files"></a>Spécifier l’emplacement des fichiers

De nombreuses commandes CLI Media Services vous permettent de passer un paramètre avec un nom de fichier. Si vous utilisez **Cloud Shell**, vous pouvez charger votre fichier sur votre clouddrive (à l'aide de Bash ou de PowerShell). 

![Charger des fichiers]

Que vous utilisiez une interface CLI locale ou **Cloud Shell**, vous devez spécifier le chemin du fichier en fonction du système d'exploitation ou du Cloud Shell (Bash ou PowerShell) que vous utilisez. Vous trouverez quelques exemples ci-dessous :

Chemin d'accès relatif du fichier (tous les systèmes d'exploitation)

* `@"mytestfile.json"`
* `@"../mytestfile.json"`

Chemin d'accès absolu du fichier sous Linux/Mac et Windows

* `@ "/usr/home/mytestfile.json"`
*    `@"c:\tmp\user\mytestfile.json"`

Utilisez `{file}` si la commande demande un chemin vers le fichier. Par exemple : `az ams transform create -a amsaccount -g resourceGroup -n custom --preset .\customPreset.json`. <br/> Utilisez `@{file}` si la commande va charger le fichier spécifié. Par exemple : `az ams account-filter create -a amsaccount -g resourceGroup -n filterName --tracks @tracks.json`.

[Charger des fichiers]: ./media/media-services-cli/upload-download-files.png