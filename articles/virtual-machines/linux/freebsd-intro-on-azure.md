---
title: Présentation de FreeBSD sur Azure
description: Apprenez à utiliser des machines virtuelles FreeBSD sur Azure
author: thomas1206
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 09/13/2017
ms.author: mimckitt
ms.openlocfilehash: 23f14e14bea665f838919ae357d11db32bddfa7f
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94579417"
---
# <a name="introduction-to-freebsd-on-azure"></a>Présentation de FreeBSD sur Azure
Cet article offre une vue d’ensemble de l’exécution d’une machine virtuelle FreeBSD dans Azure.

## <a name="overview"></a>Vue d’ensemble
FreeBSD pour Microsoft Azure est un système d’exploitation informatique avancé utilisé sur les serveurs, ordinateurs de bureau et plateformes intégrées modernes.

Microsoft Corporation rend disponibles des images de FreeBSD sur Azure avec [l’Agent invité de machine virtuelle Azure](https://github.com/Azure/WALinuxAgent/) préconfiguré. Actuellement, les versions FreeBSD suivantes sont proposées sous forme d’images par Microsoft :

- FreeBSD 10.4 sur la Place de marché Azure
- FreeBSD 11.2 sur la Place de marché Azure
- FreeBSD 12.0 sur la Place de marché Azure

L’agent est responsable de la communication entre la machine virtuelle FreeBSD et Azure Fabric pour les opérations telles que l’approvisionnement de la machine virtuelle à la première utilisation (nom d’utilisateur, mot de passe ou clé SSH, nom d’hôte, etc.) et l’activation de fonctionnalités pour certaines extensions de machines virtuelles.

Comme pour les versions futures de FreeBSD, la stratégie consiste à rester à jour et à rendre disponibles les dernières versions dès qu’elles sont publiées par l’équipe d’ingénieurs de lancement de FreeBSD.

### <a name="create-a-freebsd-vm-through-azure-cli-on-freebsd"></a>Créer une machine virtuelle FreeBSD avec Azure CLI sur FreeBSD
Vous devez d’abord installer [Azure CLI](/cli/azure/get-started-with-azure-cli) à l’aide de la commande suivante sur une machine FreeBSD.

```bash 
curl -L https://aka.ms/InstallAzureCli | bash
```

Si l’interpréteur de commandes n’est pas installé sur votre ordinateur FreeBSD, exécutez la commande suivante avant l’installation. 

```bash
sudo pkg install bash
```

Si python n’est pas installé sur votre ordinateur FreeBSD, exécutez les commandes suivantes avant l’installation. 

```bash
sudo pkg install python35
cd /usr/local/bin 
sudo rm /usr/local/bin/python 
sudo ln -s /usr/local/bin/python3.5 /usr/local/bin/python
```

Pendant l’installation, vous devez répondre à la question suivante : `Modify profile to update your $PATH and enable shell/tab completion now? (Y/n)`. Si vous répondez `y` et que vous entrez `/etc/rc.conf` comme `a path to an rc file to update`, vous pouvez rencontrer le problème suivant : `ERROR: [Errno 13] Permission denied`. Pour résoudre ce problème, vous devez accorder le droit d’écriture à l’utilisateur actif pour le fichier `etc/rc.conf`.

Vous pouvez maintenant vous connecter à Azure et créer votre machine virtuelle FreeBSD. Voici un exemple de création d’une machine virtuelle FreeBSD 11.0. Vous pouvez également ajouter le paramètre `--public-ip-address-dns-name` avec un nom DNS global unique pour une adresse IP publique nouvellement créée. 

```azurecli
az login 
az group create --name myResourceGroup --location eastus
az vm create --name myFreeBSD11 \
    --resource-group myResourceGroup \
    --image MicrosoftOSTC:FreeBSD:11.0:latest \
    --admin-username azureuser \
    --generate-ssh-keys
```

Vous pouvez ensuite vous connecter à votre machine virtuelle FreeBSD via l’adresse IP qui figure dans la sortie du déploiement ci-dessus. 

```bash
ssh azureuser@xx.xx.xx.xx -i /etc/ssh/ssh_host_rsa_key
```   

## <a name="vm-extensions-for-freebsd"></a>Extension de machine virtuelle pour FreeBSD
Voici les extensions de machines virtuelles prises en charge par FreeBSD.

### <a name="vmaccess"></a>VMAccess
L’extension [VMAccess](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) peut :

* Réinitialiser le mot de passe de l’utilisateur sudo d’origine.
* Créer un nouvel utilisateur sudo avec le mot de passe spécifié.
* Définir une clé hôte publique donnée.
* Réinitialiser la clé hôte publique fournie pendant l’approvisionnement de la machine virtuelle si la clé hôte n’est pas fournie.
* Ouvrir le port SSH (22) et restaurer sshd_config si reset_ssh a la valeur True.
* Supprimer l’utilisateur existant.
* Vérifier les disques.
* Réparer un disque ajouté.

### <a name="customscript"></a>CustomScript
L’extension [CustomScript](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript) peut :

* Télécharger les scripts personnalisés dans le stockage Azure ou un stockage public externe (par exemple, GitHub) s’ils sont fournis.
* Exécuter le script de point d’entrée.
* Prendre en charge les commandes inline.
* Convertir automatiquement le saut de ligne Windows dans des scripts Shell et Python.
* Supprimer automatiquement les nomenclatures dans des scripts Shell et Python.
* Protéger les données sensibles dans CommandToExecute.

> [!NOTE]
> La machine virtuelle FreeBSD prend uniquement en charge CustomScript version 1.x à ce stade.  

## <a name="authentication-user-names-passwords-and-ssh-keys"></a>Authentification : noms d’utilisateurs, mots de passe et clés SSH
Lorsque vous créez une machine virtuelle FreeBSD avec le Portail Azure, vous devez fournir un nom d’utilisateur, un mot de passe ou une clé publique SSH.
Le nom d’utilisateur pour le déploiement d’une machine virtuelle FreeBSD sur Azure ne doit pas correspondre au nom des comptes système (UID < 100) déjà présents sur la machine virtuelle (« root », par exemple).
Actuellement, seule la règle RSA SSH est prise en charge. Une clé SSH multiligne doit commencer par `---- BEGIN SSH2 PUBLIC KEY ----` et se terminer par `---- END SSH2 PUBLIC KEY ----`.

## <a name="obtaining-superuser-privileges"></a>Obtention de privilèges de superutilisateur
Le compte utilisateur qui est spécifié pendant le déploiement de l'instance de machine virtuelle dans Azure est un compte privilégié. Le package de « sudo » a été installé dans l’image FreeBSD publiée.
Une fois connecté avec ce compte utilisateur, vous pouvez exécuter les commandes en tant que root avec la syntaxe de commande.

```
$ sudo <COMMAND>
```

Vous pouvez éventuellement obtenir un interpréteur de commandes root avec `sudo -s`.

## <a name="known-issues"></a>Problèmes connus
[L’agent invité de machine virtuelle Azure](https://github.com/Azure/WALinuxAgent/) version 2.2.2 présente un [problème connu](https://github.com/Azure/WALinuxAgent/pull/517) qui provoque l’échec de l’approvisionnement des machines virtuelles FreeBSD sur Azure. Le correctif a été capturé par l’[Agent invité de machine virtuelle Azure](https://github.com/Azure/WALinuxAgent/) 2.2.3 et versions ultérieures. 

## <a name="next-steps"></a>Étapes suivantes
* Accédez à [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/thefreebsdfoundation.freebsd-12_1?tab=Overview) pour créer une machine virtuelle FreeBSD.
