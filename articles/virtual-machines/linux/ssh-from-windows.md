---
title: Utiliser des clés SSH pour se connecter à des machines virtuelles Linux
description: Apprenez à créer et à utiliser des clés SSH à partir d’un ordinateur Windows pour vous connecter à une machine virtuelle Linux dans Azure.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.date: 10/31/2020
ms.topic: how-to
ms.author: cynthn
ms.openlocfilehash: 183b601a4521c3ff3e4578784f7adadd01045b0e
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93147145"
---
# <a name="how-to-use-ssh-keys-with-windows-on-azure"></a>Comment utiliser des clés SSH avec Windows sur Azure

Cet article est destiné aux utilisateurs Windows qui souhaitent [créer](#create-an-ssh-key-pair) et utiliser des clés *Secure Shell (SSH)* pour [se connecter](#connect-to-your-vm) à des machines virtuelles Linux dans Azure. Vous pouvez également [générer et stocker des clés SSH dans le Portail Azure](../ssh-keys-portal.md) à utiliser lors de la création de machines virtuelles dans le portail.


Pour utiliser des clés SSH à partir d’un client Linux ou macOS, consultez les [étapes rapides](mac-create-ssh-keys.md). Pour obtenir une vue d’ensemble plus détaillée de SSH, consultez [Procédure détaillée : Créer et gérer des clés SSH pour l’authentification sur une machine virtuelle Linux dans Azure](create-ssh-keys-detailed.md).

## <a name="overview-of-ssh-and-keys"></a>Vue d’ensemble de SSH et des clés

[SSH](https://www.ssh.com/ssh/) est un protocole de connexion chiffré qui permet d’ouvrir des sessions sécurisées via des connexions non sécurisées. SSH est le protocole de connexion par défaut pour les machines virtuelles Linux hébergées dans Azure. Bien que le protocole SSH lui-même offre une connexion chiffrée, l’utilisation de mots de passe avec SSH laisse néanmoins la machine virtuelle vulnérable aux attaques en force brute. Nous vous recommandons de vous connecter à une machine virtuelle via SSH à l’aide d’une paire de clés publique/privée, également appelées clés *SSH*. 

La paire de clés publique-privée est semblable au verrou sur votre porte d’entrée. Le verrou est exposé au **public** , toute personne disposant de la bonne clé peut ouvrir la porte. La clé est **privée** et fournie uniquement aux personnes auxquelles vous faites confiance, car elle peut être utilisée pour déverrouiller la porte. 

- La *clé publique* est placée sur votre machine virtuelle Linux lorsque vous créez la machine virtuelle. 

- La *clé privée* reste sur votre système local. Protégez cette clé privée. Ne la partagez pas.

Lorsque vous vous connectez à votre machine virtuelle Linux, la machine virtuelle teste le client SSH pour s’assurer qu’il dispose de la clé privée appropriée. Si le client a la clé privée, il est autorisé à accéder à la machine virtuelle. 

Selon les stratégies de sécurité de votre organisation, vous pouvez réutiliser une paire de clés unique pour accéder à plusieurs services et machines virtuelles Azure. Vous n’avez pas besoin d’une paire de clés distincte pour chaque machine virtuelle. 

Votre clé publique peut être partagée avec n’importe qui, mais vous seul (ou votre infrastructure de sécurité locale) devez avoir accès à votre clé privée.

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="ssh-clients"></a>Clients SSH

Les versions récentes de Windows 10 incluent des [commandes client OpenSSH](https://blogs.msdn.microsoft.com/commandline/2018/03/07/windows10v1803/) permettant de créer et d’utiliser des clés SSH, ainsi que d’établir des connexions SSH à partir de PowerShell ou d’une invite de commandes. Il s’agit du moyen le plus simple de créer une connexion SSH à votre machine virtuelle Linux à partir d’un ordinateur Windows. 

Vous pouvez également utiliser Bash dans [Azure Cloud Shell](../../cloud-shell/overview.md) pour vous connecter à votre machine virtuelle. Vous pouvez utiliser Cloud Shell dans un [navigateur Web](https://shell.azure.com/bash), à partir du [Portail Azure](https://portal.azure.com) ou en tant que terminal dans Visual Studio Code à l’aide de l’[extension de compte Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account).

Vous pouvez également installer le [sous-système Windows pour Linux](/windows/wsl/about) pour vous connecter à votre machine virtuelle via SSH et utiliser d’autres outils Linux natifs dans un interpréteur de commandes Bash.

## <a name="create-an-ssh-key-pair"></a>Création d’une paire de clés SSH

Créez une paire de clés SSH à l’aide de la commande `ssh-keygen`. Entrez un nom de fichier ou utilisez la valeur par défaut indiquée entre parenthèses (par exemple `C:\Users\username/.ssh/id_rsa`).  Entrez une phrase secrète pour le fichier ou laissez la phrase secrète vide si vous ne souhaitez pas utiliser de phrase secrète. 

```powershell
ssh-keygen -m PEM -t rsa -b 4096
```

## <a name="create-a-vm-using-your-key"></a>Créer une machine virtuelle à l’aide de votre clé

Pour créer une machine virtuelle Linux qui utilise des clés SSH pour l’authentification, fournissez votre clé publique SSH lors de la création de la machine virtuelle.

À l’aide de l’interface de ligne de commande Azure, vous spécifiez le chemin d’accès et le nom de fichier de la clé publique à l’aide de `az vm create` et du paramètre `--ssh-key-value`.

```azurecli
az vm create \
   --resource-group myResourceGroup \
   --name myVM \
   --image UbuntuLTS\
   --admin-username azureuser \
   --ssh-key-value ~/.ssh/id_rsa.pub
```

Avec PowerShell, utilisez `New-AzVM` et ajoutez la clé SSH à la configuration de la machine virtuelle à l’aide de `. Pour obtenir un exemple, consultez [Démarrage rapide : Créer une machine virtuelle Linux dans Azure avec PowerShell](quick-create-powershell.md).

Si vous effectuez un grand nombre de déploiements à l’aide du portail, vous souhaiterez peut-être charger votre clé publique dans Azure, où elle peut être facilement sélectionnée lors de la création d’une machine virtuelle à partir du portail. Pour plus d’informations, consultez [Charger une clé SSH](../ssh-keys-portal.md#upload-an-ssh-key).


## <a name="connect-to-your-vm"></a>Connexion à votre machine virtuelle

Une fois la clé publique déployée sur votre machine virtuelle Azure et la clé privée sur votre système local, établissez une connexion SSH sur votre machine virtuelle à l’aide de l’adresse IP ou du nom DNS de votre machine virtuelle. Remplacez *azureuser* et *10.111.12.123* dans la commande suivante par le nom d’utilisateur de l’administrateur, l’adresse IP (ou le nom de domaine complet) et le chemin d’accès à votre clé privée :

```bash
ssh -i ~/.ssh/id_rsa.pub azureuser@10.111.12.123
```

Si vous avez configuré une phrase secrète quand vous avez créé votre paire de clés, entrez-la quand vous y êtes invité.

Si la machine virtuelle utilise la stratégie juste-à-temps, vous devez demander à y accéder avant de pouvoir vous connecter à cette machine virtuelle. Pour plus d’informations sur la stratégie juste-à-temps, consultez [Gérer l’accès à la machine virtuelle à l’aide de la stratégie juste-à-temps](../../security-center/security-center-just-in-time.md).


## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les clés SSH dans le Portail Azure, consultez [Générer et stocker des clés SSH dans le Portail Azure](../ssh-keys-portal.md) à utiliser lors de la création de machines virtuelles dans le portail.

- Pour connaître les étapes détaillées, les options et des exemples avancés d’utilisation de clés SSH, consultez [Étapes détaillées pour créer des paires de clés SSH](create-ssh-keys-detailed.md).

- Vous pouvez également utiliser PowerShell dans Azure Cloud Shell pour générer des clés SSH et établir des connexions SSH sur des machines virtuelles Linux. Consultez [Démarrage rapide de PowerShell](../../cloud-shell/quickstart-powershell.md#ssh).

- Si vous avez des difficultés à utiliser SSH pour vous connecter à vos machines virtuelles Linux, consultez [Résoudre les problèmes de connexion SSH à une machine virtuelle Linux Azure](../troubleshooting/troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json).
