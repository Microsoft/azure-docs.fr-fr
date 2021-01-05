---
title: Démarrage rapide pour Azure Cloud Shell - Bash
description: Découvrez comment utiliser la ligne de commande Bash dans votre navigateur avec Azure Cloud Shell.
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 03/12/2018
ms.author: damaerte
ms.openlocfilehash: 91b7c58890518559c046023bd78c9248e9840f9f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89468747"
---
# <a name="quickstart-for-bash-in-azure-cloud-shell"></a>Démarrage rapide de Bash dans Azure Cloud Shell

Ce document explique comment utiliser Bash dans Azure Cloud Shell dans le [portail Azure](https://ms.portal.azure.com/).

> [!NOTE]
> Un démarrage rapide de [PowerShell dans Azure Cloud Shell](quickstart-powershell.md) est également disponible.

## <a name="start-cloud-shell"></a>Démarrer Cloud Shell
1. Lancez **Cloud Shell** dans le volet de navigation en haut du Portail Azure. <br>
![Capture d’écran montrant comment démarrer Azure Cloud Shell dans le Portail Azure.](media/quickstart/shell-icon.png)

2. Sélectionnez un abonnement pour créer un compte de stockage et un partage Microsoft Azure Files.
3. Sélectionnez Créer le stockage

> [!TIP]
> L’authentification sur Azure CLI est automatique à chaque session.

### <a name="select-the-bash-environment"></a>Sélectionnez l’environnement Bash
Vérifiez que la liste déroulante des environnements située à gauche de la fenêtre de l’interpréteur de commandes indique `Bash`. <br>
![Capture d’écran montrant comment sélectionner l’environnement bash pour Azure Cloud Shell.](media/quickstart/env-selector.png)

### <a name="set-your-subscription"></a>Définissez votre abonnement
1. Listez les abonnements auxquels vous avez accès.
   ```azurecli-interactive
   az account list
   ```

2. Définissez votre abonnement préféré :

   ```azurecli-interactive
   az account set --subscription 'my-subscription-name'
   ```

> [!TIP]
> Votre abonnement sera mémorisé pour les sessions ultérieures avec `/home/<user>/.azure/azureProfile.json`.

### <a name="create-a-resource-group"></a>Créer un groupe de ressources
Créez un nouveau groupe de ressources, nommé « MyRG », dans la région États-Unis de l’Ouest.
```azurecli-interactive
az group create --location westus --name MyRG
```

### <a name="create-a-linux-vm"></a>Créer une machine virtuelle Linux
Créez une machine virtuelle Ubuntu dans votre nouveau groupe de ressources. Azure CLI crée des clés SSH et les utilise pour configurer la machine virtuelle. <br>

```azurecli-interactive
az vm create -n myVM -g MyRG --image UbuntuLTS --generate-ssh-keys
```

> [!NOTE]
> Si l’option `--generate-ssh-keys` est utilisée, Azure CLI crée et configure des clés publiques et privées dans le répertoire `$Home` et la machine virtuelle. Par défaut, les clés sont placées dans Cloud Shell aux emplacements `/home/<user>/.ssh/id_rsa` et `/home/<user>/.ssh/id_rsa.pub`. Le dossier `.ssh` est conservé dans l’image de 5 Go du partage de fichiers Azure attaché servant à conserver `$Home`.

Votre nom d’utilisateur sur cette machine virtuelle sera votre nom d’utilisateur utilisé dans Cloud Shell ($User@Azure:).

### <a name="ssh-into-your-linux-vm"></a>Se connecter avec SSH à votre machine virtuelle Linux
1. Recherchez le nom de votre machine virtuelle dans la barre de recherche du Portail Azure.
2. Cliquez sur « Se connecter » pour obtenir le nom de votre machine virtuelle et votre adresse IP publique. <br>
   ![Capture d’écran montrant comment se connecter à une machine virtuelle Linux avc SSH.](media/quickstart/sshcmd-copy.png)

3. Établissez une connexion SSH avec votre machine virtuelle avec la commande `ssh`.
   ```
   ssh username@ipaddress
   ```

Lors de l’établissement de la connexion SSH, vous devriez voir l’invite de bienvenue sur Ubuntu. <br>
![Capture d’écran montrant l’initialisation Ubuntu et l’invite de bienvenue après avoir établi une connexion SSH.](media/quickstart/ubuntu-welcome.png)

## <a name="cleaning-up"></a>Nettoyage 
1. Fermez votre session SSH.
   ```
   exit
   ```

2. Supprimez votre groupe de ressources et toutes les ressources qu’il contient.
   ```azurecli-interactive
   az group delete -n MyRG
   ```

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur les fichiers persistants pour Bash dans Cloud Shell](persisting-shell-storage.md) <br>
[En savoir plus sur Azure CLI](/cli/azure/) <br>
[En savoir plus sur le stockage Azure Files](../storage/files/storage-files-introduction.md) <br>