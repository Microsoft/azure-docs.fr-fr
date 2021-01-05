---
title: Se connecter à une machine virtuelle Linux à l’aide d’Azure Bastion
description: Dans cet article, découvrez comment vous connecter à une machine virtuelle Linux à l’aide d’Azure Bastion.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: how-to
ms.date: 02/24/2020
ms.author: cherylmc
ms.openlocfilehash: 2af63997f9ea74dfce184639d2fedb42d354fb39
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077573"
---
# <a name="connect-using-ssh-to-a-linux-virtual-machine-using-azure-bastion"></a>Connexion SSH à une machine virtuelle Linux à l'aide d'Azure Bastion

Cet article vous montre comment vous connecter en toute sécurité et en toute transparence avec le protocole SSH à vos machines virtuelles Linux dans un réseau virtuel Azure. Vous pouvez vous connecter à une machine virtuelle directement à partir du portail Azure. Lorsque vous utilisez Azure Bastion, les machines virtuelles n’ont pas besoin de client, d’agent ni de logiciel supplémentaire. Pour plus d’informations sur Azure Bastion, consultez la [vue d’ensemble](bastion-overview.md).

Vous pouvez utiliser Azure Bastion pour vous connecter au travers du protocole SSH à une machine virtuelle Linux. Vous pouvez utiliser le nom d’utilisateur/mot de passe et les clés SSH pour l’authentification. Vous pouvez vous connecter à votre machine virtuelle avec des clés SSH en utilisant :

* Une clé privée que vous entrez manuellement
* Un fichier qui contient les informations relatives à la clé privée

La clé privée SSH doit être dans un format qui commence par `"-----BEGIN RSA PRIVATE KEY-----"` et se termine par `"-----END RSA PRIVATE KEY-----"`.

## <a name="prerequisites"></a>Prérequis

Assurez-vous que vous avez configuré un hôte Azure Bastion pour le réseau virtuel dans lequel réside la machine virtuelle. Pour plus d’informations, consultez [Créer un hôte Azure Bastion](./tutorial-create-host-portal.md). Une fois que le service Bastion est approvisionné et déployé dans votre réseau virtuel, vous pouvez l’utiliser pour vous connecter à toutes les machines virtuelles dans ce réseau virtuel. 

Lorsque vous utilisez Bastion pour vous connecter, ce dernier part du principe que vous utilisez le protocole RDP pour vous connecter à une machine virtuelle Windows, et le protocole SSH pour vous connecter à vos machines virtuelles Linux. Pour plus d’informations sur la connexion à une machine virtuelle Windows, consultez [Se connecter à une machine virtuelle Windows](bastion-connect-vm-rdp.md).

### <a name="required-roles"></a>Rôles nécessaires

Pour établir une connexion, les rôles suivants sont nécessaires :

* Rôle de lecteur sur la machine virtuelle
* Rôle de lecteur sur la carte réseau avec adresse IP privée de la machine virtuelle
* Rôle de lecteur sur la ressource Azure Bastion

### <a name="ports"></a>Ports

Pour vous connecter à la machine virtuelle Linux par le biais du protocole SSH, les ports suivants doivent être ouverts sur votre machine virtuelle :

* Port d’entrée : SSH (22)

## <a name="connect-using-username-and-password"></a><a name="username"></a>Connexion : Utilisation du nom d’utilisateur et du mot de passe

1. Ouvrez le [portail Azure](https://portal.azure.com). Accédez à la machine virtuelle à laquelle vous souhaitez vous connecter, puis cliquez sur **Se connecter** et sélectionnez **Bastion** dans la liste déroulante.

   ![Capture d’écran montrant la vue d’ensemble d’une machine virtuelle dans le Portail Azure avec l’option Se connecter sélectionnée.](./media/bastion-connect-vm-ssh/connect.png)
1. Quand vous cliquez sur Bastion, une barre latérale s’affiche avec trois onglets : RDP, SSH et Bastion. Si le service Bastion a été approvisionné pour le réseau virtuel, l’onglet Bastion est actif par défaut. Si vous n’avez pas configuré Bastion pour le réseau virtuel, consultez [Configure Bastion (Configurer Bastion)](./tutorial-create-host-portal.md).

   ![Capture d’écran montrant la boîte de dialogue Se connecter à une machine virtuelle avec l’option BASTION sélectionnée.](./media/bastion-connect-vm-ssh/bastion.png)
1. Entrez le nom d’utilisateur et le mot de passe pour le protocole SSH vers votre machine virtuelle.
1. Cliquez sur le bouton **Se connecter** après avoir entré la clé.

## <a name="connect-manually-enter-a-private-key"></a><a name="privatekey"></a>Connexion : Entrez manuellement une clé privée

1. Ouvrez le [portail Azure](https://portal.azure.com). Accédez à la machine virtuelle à laquelle vous souhaitez vous connecter, puis cliquez sur **Se connecter** et sélectionnez **Bastion** dans la liste déroulante.

   ![Capture d’écran montrant la vue d’ensemble d’une machine virtuelle dans le Portail Azure avec l’option Se connecter sélectionnée.](./media/bastion-connect-vm-ssh/connect.png)
1. Quand vous cliquez sur Bastion, une barre latérale s’affiche avec trois onglets : RDP, SSH et Bastion. Si le service Bastion a été approvisionné pour le réseau virtuel, l’onglet Bastion est actif par défaut. Si vous n’avez pas configuré Bastion pour le réseau virtuel, consultez [Configure Bastion (Configurer Bastion)](./tutorial-create-host-portal.md).

   ![Capture d’écran montrant la boîte de dialogue Se connecter à une machine virtuelle avec l’option BASTION sélectionnée.](./media/bastion-connect-vm-ssh/bastion.png)
1. Entrez le nom d’utilisateur, puis sélectionnez **Clé privée SSH**.
1. Entrez votre clé privée dans la zone de texte **Clé privée SSH** (ou collez-la directement).
1. Cliquez sur le bouton **Se connecter** après avoir entré la clé.

## <a name="connect-using-a-private-key-file"></a><a name="ssh"></a>Connexion : Utilisation d’un fichier de clé privée

1. Ouvrez le [portail Azure](https://portal.azure.com). Accédez à la machine virtuelle à laquelle vous souhaitez vous connecter, puis cliquez sur **Se connecter** et sélectionnez **Bastion** dans la liste déroulante.

   ![Capture d’écran montrant la vue d’ensemble d’une machine virtuelle dans le Portail Azure avec l’option Se connecter sélectionnée.](./media/bastion-connect-vm-ssh/connect.png)
1. Quand vous cliquez sur Bastion, une barre latérale s’affiche avec trois onglets : RDP, SSH et Bastion. Si le service Bastion a été approvisionné pour le réseau virtuel, l’onglet Bastion est actif par défaut. Si vous n’avez pas configuré Bastion pour le réseau virtuel, consultez [Configure Bastion (Configurer Bastion)](./tutorial-create-host-portal.md).

   ![Capture d’écran montrant la boîte de dialogue Se connecter à une machine virtuelle avec l’option BASTION sélectionnée.](./media/bastion-connect-vm-ssh/bastion.png)
1. Entrez le nom d’utilisateur, puis sélectionnez **SSH Private Key from Local File (Clé privée SSH provenant du fichier local)** .
1. Cliquez sur le bouton **Parcourir** (icône de dossier dans le fichier local).
1. Recherchez le fichier, puis cliquez sur **Ouvrir**.
1. Cliquez sur **Se connecter** pour vous connecter à la machine virtuelle. Une fois que vous avez cliqué sur Se connecter, le protocole SSH vers cette machine virtuelle s’ouvre directement dans le portail Azure. La connexion s’effectue au moyen du protocole HTML5 sur le port 443 du service Bastion, sur l’adresse IP privée de votre machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

Lisez les [questions fréquentes sur Bastion](bastion-faq.md).
