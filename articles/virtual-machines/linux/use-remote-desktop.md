---
title: Utiliser le Bureau à distance sur une machine virtuelle Linux dans Azure
description: Découvrez comment installer et configurer le Bureau à distance (xrdp) pour effectuer une connexion à une machine virtuelle Linux dans Azure à l’aide des outils graphiques
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: how-to
ms.date: 09/12/2019
ms.author: cynthn
ms.openlocfilehash: efcfb2fd431dd9626ddcf918127964672b0e5ce0
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500476"
---
# <a name="install-and-configure-remote-desktop-to-connect-to-a-linux-vm-in-azure"></a>Installer et configurer le Bureau à distance pour effectuer une connexion à une machine virtuelle Linux dans Azure
Les machines virtuelles (VM) Linux dans Azure sont généralement gérées à partir de la ligne de commande à l’aide d’une connexion Secure Shell (SSH). Si vous découvrez Linux, ou si vous souhaitez des scénarios de dépannage rapides, l’utilisation du Bureau à distance peut se révéler plus facile. Cet article explique comment installer et configurer un environnement de bureau ([xfce](https://www.xfce.org)) et le Bureau à distance ([xrdp](http://xrdp.org)) pour votre machine virtuelle Linux à l’aide du modèle de déploiement Resource Manager.


## <a name="prerequisites"></a>Prérequis
Cet article nécessite que vous disposiez d’une machine virtuelle Ubuntu 18.04 LTS dans Azure. Si vous avez besoin créer une machine virtuelle, utilisez l’une des méthodes suivantes :

- [Azure CLI](quick-create-cli.md)
- [Portail Azure](quick-create-portal.md)


## <a name="install-a-desktop-environment-on-your-linux-vm"></a>Installation d’un environnement de bureau sur votre machine virtuelle Linux
La plupart des machines virtuelles Linux dans Azure n’ont pas d’environnement de bureau installé par défaut. Les machines virtuelles Linux sont généralement gérées à l’aide des connexions SSH plutôt que d’un environnement de bureau. Il existe divers environnements de bureau sous Linux que vous pouvez choisir. Selon l’environnement de bureau que vous choisissez, ce dernier peut consommer entre 1 et 2 Go d’espace disque et nécessiter 5 à 10 minutes pour installer et configurer tous les packages requis.

L’exemple suivant installe l’environnement de bureau léger [xfce4](https://www.xfce.org/) sur une machine virtuelle Ubuntu 18.04 LTS. Les commandes pour les autres distributions varient légèrement (utilisez `yum` pour effectuer une installation sur Red Hat Enterprise Linux et configurez les règles `selinux` appropriées, ou utilisez `zypper` pour une installation sur SUSE, par exemple).

Tout d’abord, connectez-vous avec SSH à votre machine virtuelle. L’exemple suivant se connecte à la machine virtuelle nommée *myvm.westus.cloudapp.azure.com* avec le nom d’utilisateur *azureuser*. Utilisez vos propres valeurs :

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

Si vous utilisez Windows et souhaitez plus d’informations sur l’utilisation de SSH, consultez [Utilisation de clés SSH avec Windows sur Azure](ssh-from-windows.md).

Ensuite, installez xfce en utilisant `apt` comme suit :

```bash
sudo apt-get update
sudo apt-get -y install xfce4
```

## <a name="install-and-configure-a-remote-desktop-server"></a>Installation et configuration d’un serveur de bureau à distance
Maintenant que vous avez installé un environnement de bureau, configurez un service de bureau à distance pour écouter les connexions entrantes. [xrdp](http://xrdp.org) est un serveur RDP (Remote Desktop Protocol) open source qui est disponible sur la plupart des distributions Linux et fonctionne bien avec xfce. Installez xrdp sur votre machine virtuelle Ubuntu comme suit :

```bash
sudo apt-get -y install xrdp
sudo systemctl enable xrdp
```

Indiquez à xrdp quel environnement de bureau utiliser lorsque vous démarrez votre session. Configurez xrdp pour utiliser xfce en tant qu’environnement de bureau comme suit :

```bash
echo xfce4-session >~/.xsession
```

Redémarrez le service xrdp pour appliquer les modifications comme suit :

```bash
sudo service xrdp restart
```


## <a name="set-a-local-user-account-password"></a>Définition d’un mot de passe de compte d’utilisateur
Si vous avez créé un mot de passe pour votre compte d’utilisateur lorsque vous avez créé votre machine virtuelle, ignorez cette étape. Si vous utilisez uniquement l’authentification par clé SSH et que vous n’avez pas de mot de passe défini pour le compte local, spécifiez un mot de passe avant d’utiliser xrdp pour vous connecter à votre machine virtuelle. xrdp ne peut pas accepter de clés SSH pour l’authentification. L’exemple suivant spécifie un mot de passe pour le compte d’utilisateur *azureuser* :

```bash
sudo passwd azureuser
```

> [!NOTE]
> La spécification d’un mot de passe ne met pas à jour votre configuration SSHD pour autoriser les connexions par mot de passe si ce n’est pas le cas actuellement. Du point de vue de la sécurité, vous souhaiterez peut-être vous connecter à votre machine virtuelle avec un tunnel SSH à l’aide de l’authentification par clé, puis vous connecter à xrdp. Dans ce cas, ignorez l’étape suivante sur la création d’une règle de groupe de sécurité réseau pour autoriser le trafic du bureau à distance.


## <a name="create-a-network-security-group-rule-for-remote-desktop-traffic"></a>Création d’une règle de groupe de sécurité réseau pour le trafic du Bureau à distance
Pour autoriser le trafic du Bureau à distance à atteindre votre machine virtuelle Linux, une règle de groupe de sécurité réseau doit être créée pour permettre au TCP sur le port 3389 d’atteindre votre machine virtuelle. Pour plus d’informations sur les règles de groupe de sécurité réseau, consultez [Présentation du groupe de sécurité réseau](../../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) Vous pouvez également [utiliser le portail Azure pour créer une règle de groupe de sécurité réseau](../windows/nsg-quickstart-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

L’exemple suivant permet de créer une règle de groupe de sécurité réseau avec [az vm open-port](/cli/azure/vm#az-vm-open-port) sur le port *3389*. À partir d’Azure CLI, et non de la session SSH sur votre machine virtuelle, ouvrez la règle de groupe de sécurité réseau suivante :

```azurecli
az vm open-port --resource-group myResourceGroup --name myVM --port 3389
```


## <a name="connect-your-linux-vm-with-a-remote-desktop-client"></a>Connexion de votre machine virtuelle Linux à un client Bureau à distance
Ouvrez votre client Bureau à distance local et connectez-vous à l’adresse IP ou au nom DNS de votre machine virtuelle Linux. Entrez le nom d’utilisateur et le mot de passe du compte d’utilisateur sur votre machine virtuelle comme suit :

![Connexion à xrdp à l’aide de votre client Bureau à distance](./media/use-remote-desktop/remote-desktop-client.png)

Une fois l’authentification effectuée, l’environnement de bureau xfce se charge et ressemble à l’exemple suivant :

![Environnement de bureau xfce via xrdp](./media/use-remote-desktop/xfce-desktop-environment.png)

Si votre client RDP local utilise l’authentification au niveau du réseau, vous devrez peut-être désactiver ce paramètre de connexion. XRDP ne prend pas en charge l’authentification au niveau du réseau. Vous pouvez également consulter les solutions RDP alternatives qui prennent en charge l’authentification au niveau du réseau, telles que [FreeRDP](https://www.freerdp.com).


## <a name="troubleshoot"></a>Dépanner
Si vous ne pouvez pas vous connecter à votre machine virtuelle Linux à l’aide d’un client Bureau à distance, utilisez `netstat` sur votre machine virtuelle Linux pour vérifier que votre machine virtuelle écoute les connexions RDP comme suit :

```bash
sudo netstat -plnt | grep rdp
```

L’exemple suivant montre la machine virtuelle à l’écoute du port TCP 3389 comme prévu :

```bash
tcp     0     0      127.0.0.1:3350     0.0.0.0:*     LISTEN     53192/xrdp-sesman
tcp     0     0      0.0.0.0:3389       0.0.0.0:*     LISTEN     53188/xrdp
```

Si le service *xrdp-sesman* n’écoute pas, sur une machine virtuelle Ubuntu, redémarrez le service comme suit :

```bash
sudo service xrdp restart
```

Examinez les journaux d’activité dans */var/log* sur votre machine virtuelle Ubuntu pour savoir pourquoi le service ne répond pas. Vous pouvez également surveiller le journal système lors d’une tentative de connexion au Bureau à distance pour afficher les erreurs :

```bash
tail -f /var/log/syslog
```

Les autres distributions de Linux telles que Red Hat Enterprise Linux et SUSE peuvent avoir différentes façons de redémarrer les services et d’autres emplacements de fichiers journaux pour la vérification.

Si vous ne recevez pas de réponse dans votre client Bureau à distance et que vous ne voyez aucun événement dans le journal système, cela indique que le trafic du bureau à distance ne peut pas joindre la machine virtuelle. Passez en revue vos règles de groupe de sécurité réseau pour vous assurer que vous disposez d’une règle autorisant TCP sur le port 3389. Pour plus d’informations, consultez [Résoudre les problèmes de connectivité des applications sur une machine virtuelle Linux Azure](../troubleshooting/troubleshoot-app-connection.md).


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la création et l’utilisation de clés SSH avec les machines virtuelles Linux, consultez [Create SSH keys for Linux VMs in Azure](mac-create-ssh-keys.md) (Création de clés SSH pour les machines virtuelles Linux dans Azure).

Pour plus d’informations sur l’utilisation de SSH avec Windows, consultez la page [Utilisation de clés SSH avec Windows sur Azure](ssh-from-windows.md).