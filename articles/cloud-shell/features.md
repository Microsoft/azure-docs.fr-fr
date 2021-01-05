---
title: Fonctionnalités d’Azure Cloud Shell | Microsoft Docs
description: Vue d'ensemble des fonctionnalités d'Azure Cloud Shell
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
ms.date: 04/26/2019
ms.author: damaerte
ms.openlocfilehash: a052364b06ac1b9b30cef76db10a79e8ed85b9a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89470151"
---
# <a name="features--tools-for-azure-cloud-shell"></a>Fonctionnalités et outils pour Azure Cloud Shell

[!INCLUDE [features-introblock](../../includes/cloud-shell-features-introblock.md)]

Azure Cloud Shell s’exécute sur `Ubuntu 16.04 LTS`.

## <a name="features"></a>Fonctionnalités

### <a name="secure-automatic-authentication"></a>Authentification automatique sécurisée

Cloud Shell authentifie automatiquement et de manière sécurisée l’accès aux comptes pour Azure CLI et Azure PowerShell.

### <a name="home-persistence-across-sessions"></a>Persistance de $HOME entre les sessions

Pour conserver les fichiers entre les sessions, Cloud Shell vous guide à travers le processus d’association d’un partage de fichiers Azure au premier lancement.
Par la suite, Cloud Shell associera automatiquement votre espace de stockage (monté sous forme de `$HOME\clouddrive`) pour toutes les sessions à venir.
De plus, votre répertoire `$HOME` est conservé en tant que fichier .img dans votre partage de fichiers Azure.
Les fichiers en dehors de `$HOME` et de l’état de la machine ne sont pas conservés entre les sessions. Utilisez les bonnes pratiques lors du stockage de secrets comme des clés SSH. [Des tutoriels existent pour la configuration de services comme Azure Key Vault](../key-vault/general/manage-with-cli2.md#prerequisites).

[Découvrez plus d’informations sur les fichiers persistants dans Cloud Shell.](persisting-shell-storage.md)

### <a name="azure-drive-azure"></a>Lecteur Azure (Azure :)

PowerShell dans Cloud Shell fournit le lecteur Azure (`Azure:`). Vous pouvez basculer vers le lecteur Azure à l'aide de `cd Azure:` et revenir à votre répertoire personnel à l'aide de `cd  ~`.
Le lecteur Azure facilite la détection et la navigation dans des ressources Azure, comme Calcul, Réseau, ou Stockage, de façon similaire à la navigation dans un système de fichiers.
Vous pouvez continuer à utiliser les [applets de commande Azure PowerShell](/powershell/azure) que vous connaissez pour gérer ces ressources quel que soit le lecteur où vous vous trouvez.
Toutes les modifications apportées aux ressources Azure, effectuées directement dans le portail Azure ou via des applets de commande Azure PowerShell, sont répercutées dans le lecteur Azure.  Vous pouvez exécuter `dir -Force` pour actualiser vos ressources.

![Capture d’écran d’un interpréteur de commandes Azure Cloud Shell initialisé et d’une liste de ressources de répertoire.](media/features-powershell/azure-drive.png)

### <a name="manage-exchange-online"></a>Gérer Exchange Online

PowerShell dans Cloud Shell contient une build privée du module Exchange Online.  Exécutez `Connect-EXOPSSession` pour obtenir vos cmdlets Exchange.

![Capture d’écran d’un interpréteur de commandes Azure Cloud Shell exécutant les commandes Connect-EXOPSSession et Get-User.](media/features-powershell/exchangeonline.png)

 Exécutez `Get-Command -Module tmp_*`
> [!NOTE]
> Le nom du module doit commencer par `tmp_`. Si vous avez installé des modules avec le même préfixe, leurs cmdlets seront également visibles. 

![Capture d’écran d’un interpréteur de commandes Azure Cloud Shell exécutant la commande Get-Command -Module tmp_*.](media/features-powershell/exchangeonlinecmdlets.png)

### <a name="deep-integration-with-open-source-tooling"></a>Étroite intégration à des outils open source

Cloud Shell inclut une authentification préconfigurée pour des outils open source comme Terraform, Ansible et Chef InSpec. Faites un essai à partir des exemples de procédures pas à pas.

## <a name="tools"></a>Outils

|Category   |Nom   |
|---|---|
|Outils Linux            |bash<br> zsh<br> sh<br> tmux<br> dig<br>               |
|Outils Azure            |[Azure CLI](https://github.com/Azure/azure-cli) et [Azure Classic CLI](https://github.com/Azure/azure-xplat-cli)<br> [AZCopy](../storage/common/storage-use-azcopy-v10.md)<br> [Interface de ligne de commande Azure Functions](https://github.com/Azure/azure-functions-core-tools)<br> [Interface de ligne de commande de Service Fabric](../service-fabric/service-fabric-cli.md)<br> [Lot chantier](https://github.com/Azure/batch-shipyard)<br> [blobxfer](https://github.com/Azure/blobxfer)|
|Éditeurs de texte           |code (Éditeur Cloud Shell)<br> Vim<br> nano<br> emacs    |
|Contrôle de code source         |git                    |
|Outils de génération            |make<br> maven<br> npm<br> pip         |
|Containers             |[Machine Docker](https://github.com/docker/machine)<br> [Kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)<br> [Helm](https://github.com/kubernetes/helm)<br> [CLI DC/OS](https://github.com/dcos/dcos-cli)         |
|Bases de données              |Client MySQL<br> Client PostgreSQL<br> [Utilitaire sqlcmd](/sql/tools/sqlcmd-utility)<br> [mssql-scripter](https://github.com/Microsoft/sql-xplat-cli) |
|Autres                  |Client iPython<br> [CLI Cloud Foundry](https://github.com/cloudfoundry/cli)<br> [Terraform](https://www.terraform.io/docs/providers/azurerm/)<br> [Ansible](https://www.ansible.com/microsoft-azure)<br> [Chef InSpec](https://www.chef.io/inspec/)<br> [Puppet Bolt](https://puppet.com/docs/bolt/latest/bolt.html)<br> [HashiCorp Packer](https://www.packer.io/)<br> [Interface CLI Office 365](https://pnp.github.io/office365-cli/)|

## <a name="language-support"></a>Support multilingue

|Langage   |Version   |
|---|---|
|.NET Core  |[3.1.302](https://github.com/dotnet/core/blob/master/release-notes/3.1/3.1.6/3.1.302-download.md)       |
|Go         |1.9        |
|Java       |1.8        |
|Node.js    |8.16.0      |
|PowerShell |[7.0.0](https://github.com/PowerShell/powershell/releases)       |
|Python     |2.7 et 3.5 (par défaut)|

## <a name="next-steps"></a>Étapes suivantes
[Démarrage rapide de Bash dans Cloud Shell](quickstart.md) <br>
[Démarrage rapide de PowerShell dans Cloud Shell](quickstart-powershell.md) <br>
[En savoir plus sur Azure CLI](/cli/azure/) <br>
[En savoir plus sur Azure PowerShell](/powershell/azure/) <br>