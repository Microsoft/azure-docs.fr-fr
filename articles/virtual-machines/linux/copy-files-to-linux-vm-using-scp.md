---
title: Déplacer des fichiers vers et depuis des machines virtuelles Linux Azure avec SCP
description: Déplacez, en toute sécurité, des fichiers vers et depuis une machine virtuelle Linux dans Azure à l’aide de SCP et d’une paire de clés SSH.
author: cynthn
ms.service: virtual-machines-linux
ms.workload: infrastructure
ms.topic: how-to
ms.date: 07/12/2017
ms.author: cynthn
ms.subservice: disks
ms.openlocfilehash: f56118750fc980c249c88b796728f4ecb2641a88
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86510954"
---
# <a name="move-files-to-and-from-a-linux-vm-using-scp"></a>Déplacer des fichiers vers et depuis une machine virtuelle Linux à l’aide de SCP

Cet article explique comment déplacer des fichiers à partir de votre poste de travail vers une machine virtuelle Linux Azure ou à partir d’une machine virtuelle Linux Azure vers votre poste de travail, à l’aide de la copie sécurisée (SCP). Le déplacement de fichiers entre votre poste de travail et une machine virtuelle Linux, de manière rapide et sécurisée, constitue une partie essentielle de la gestion de votre infrastructure Azure. 

Pour cet article, vous avez besoin d’une machine virtuelle Linux déployée dans Azure à l’aide de [fichiers de clés publiques et privées SSH](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Vous avez également besoin d’un client SCP pour votre ordinateur local. Celui-ci est construit par-dessus SSH et inclus dans l’interpréteur de commandes Bash par défaut de la plupart des ordinateurs Mac et Linux et dans certains interpréteurs de commandes Windows.

## <a name="quick-commands"></a>Commandes rapides

Copier un fichier vers la machine virtuelle Linux

```bash
scp file azureuser@azurehost:directory/targetfile
```

Copier un fichier à partir de la machine virtuelle Linux

```bash
scp azureuser@azurehost:directory/file targetfile
```

## <a name="detailed-walkthrough"></a>Procédure pas à pas

Par exemple, nous déplaçons un fichier de configuration Azure vers une machine virtuelle Linux et nous extrayons un répertoire de fichiers journaux, en utilisant à la fois SCP et des clés SSH.   

## <a name="ssh-key-pair-authentication"></a>Authentification avec une paire de clés SSH

SCP utilise SSH pour la couche de transport. SSH gère l’authentification sur l’hôte de destination, puis déplace le fichier dans un tunnel chiffré fourni par défaut avec SSH. Pour l’authentification SSH, des noms d’utilisateur et des mots de passe peuvent être utilisés. Toutefois, l’authentification par clé publique et privée SSH est recommandée en tant que meilleure pratique de sécurité. Une fois que SSH a authentifié la connexion, SCP commence à copier le fichier. À l’aide d’une valeur `~/.ssh/config` correctement configurée et de clés publiques et privées SSH, la connexion SCP peut être établie simplement à l’aide d’un nom de serveur (ou d’une adresse IP). Si vous n’avez qu’une seule clé SSH, SCP la recherche dans le répertoire `~/.ssh/` et l’utilise par défaut pour se connecter à la machine virtuelle.

Pour plus d’informations sur la configuration de votre `~/.ssh/config` et les clés publiques et privées SSH, consultez [Créer des clés SSH](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="scp-a-file-to-a-linux-vm"></a>Copie sécurisée (SCP) d’un fichier vers une machine virtuelle Linux

Pour le premier exemple, nous copions un fichier de configuration Azure vers une machine virtuelle Linux qui est utilisée pour déployer l’automatisation. Étant donné que ce fichier contient des informations d’identification d’API Azure, notamment des secrets, sa sécurité est importante. Le tunnel chiffré fourni par SSH protège le contenu du fichier.

La commande suivante copie le fichier local *.azure/config* vers une machine virtuelle Azure avec le nom de domaine complet *myserver.eastus.cloudapp.azure.com*. Le nom d’utilisateur administrateur sur la machine virtuelle Azure est *azureuser*. Le fichier est ciblé dans le répertoire */home/azureuser/* . Substituez vos propres valeurs dans cette commande.

```bash
scp ~/.azure/config azureuser@myserver.eastus.cloudapp.com:/home/azureuser/config
```

## <a name="scp-a-directory-from-a-linux-vm"></a>Copie sécurisée (SCP) d’un répertoire à partir d’une machine virtuelle Linux

Pour cet exemple, nous copions un répertoire de fichiers journaux à partir de la machine virtuelle Linux sur votre poste de travail. Un fichier journal est susceptible de contenir des données sensibles ou secrètes. Toutefois, l’utilisation de SCP garantit que le contenu des fichiers journaux est chiffré. L’utilisation de SCP pour transférer les fichiers constitue le moyen le plus simple de copier le répertoire et les fichiers journaux sur votre poste de travail tout en assurant leur sécurité.

La commande suivante copie les fichiers contenus dans le répertoire */home/azureuser/logs/* sur la machine virtuelle Azure dans le répertoire/tmp local :

```bash
scp -r azureuser@myserver.eastus.cloudapp.com:/home/azureuser/logs/. /tmp/
```

L’indicateur `-r` indique à SCP de copier de manière récursive les fichiers et les répertoires à partir du point du répertoire indiqué dans la commande.  Notez également que la syntaxe de ligne de commande est similaire à une commande de copie `cp`.

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les utilisateurs, SSH et vérifier ou réparer les disques de machines virtuelles Azure Linux à l'aide de l’extension VMAccess](../extensions/vmaccess.md?toc=/azure/virtual-machines/linux/toc.json)
