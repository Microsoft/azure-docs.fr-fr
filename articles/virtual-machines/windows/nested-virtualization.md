---
title: Guide d’activation de la virtualisation imbriquée dans les machines virtuelles Azure
description: Guide d’activation de la virtualisation imbriquée dans les machines virtuelles Azure
author: cynthn
ms.author: cynthn
ms.date: 10/09/2017
ms.topic: how-to
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.openlocfilehash: 03df7db13ebd3ebec407bb046cc735c835e01068
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87074245"
---
# <a name="how-to-enable-nested-virtualization-in-an-azure-vm"></a>Guide d’activation de la virtualisation imbriquée dans une machine virtuelle Azure

La virtualisation imbriquée est prise en charge dans plusieurs familles de machine virtuelle Azure. Cette fonctionnalité offre une grande souplesse pour la prise en charge de scénarios comme les environnements de développement, de test, de formation et de démonstration.   

Cet article parcourt l’activation de Hyper-V sur une machine virtuelle Azure et la configuration de la connectivité Internet pour cette machine virtuelle invitée.

## <a name="create-a-nesting-capable-azure-vm"></a>Créer une machine virtuelle Azure prenant en charge l’imbrication

Créer une machine virtuelle Azure Windows Server 2016. Pour obtenir la liste complète des tailles de machine virtuelle prenant en charge l’imbrication, consultez l’[article sur l’unité Compute Azure](../acu.md).

Veillez à choisir une taille de machine virtuelle suffisante pour prendre en charge les demandes d’une machine virtuelle invitée. Dans cet exemple, nous utilisons une machine virtuelle Azure de taille D3_v3. 

Vous pouvez afficher la disponibilité régionale de machines virtuelles des séries Dv3 et Ev3 [ici](https://azure.microsoft.com/regions/services/).

>[!NOTE]
>
>Pour plus d’instructions sur la création d’une nouvelle machine virtuelle, consultez [Créer et gérer des machines virtuelles Windows avec le module Azure PowerShell](./tutorial-manage-vm.md)
    
## <a name="connect-to-your-azure-vm"></a>Se connecter à votre machine virtuelle Azure

Créez une connexion Bureau à distance à la machine virtuelle.

1. Cliquez sur le bouton **Connexion** dans les propriétés de la machine virtuelle. Un fichier de protocole Remote Desktop Protocol (fichier .rdp) est créé et téléchargé.

2. Pour vous connecter à votre machine virtuelle, ouvrez le fichier RDP téléchargé. À l’invite, cliquez sur **Se connecter**. Sur un Mac, vous avez besoin d’un client RDP similaire à ce [Client Bureau à distance](https://apps.apple.com/app/microsoft-remote-desktop/id1295203466?mt=12) disponible sur le Mac App Store.

3. Entrez le nom d’utilisateur et le mot de passe spécifiés lors de la création de la machine virtuelle, puis cliquez sur **OK**.

4. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Cliquez sur **Oui** ou **Continuer** pour continuer le processus de connexion.

## <a name="enable-the-hyper-v-feature-on-the-azure-vm"></a>Activer la fonctionnalité Hyper-V sur la machine virtuelle Azure
Vous pouvez configurer ces paramètres manuellement, ou utiliser le script PowerShell que nous vous fournissons pour automatiser la configuration.

### <a name="option-1-use-a-powershell-script-to-configure-nested-virtualization"></a>Option 1 : Utiliser un script PowerShell pour configurer la virtualisation imbriquée
Un script PowerShell pour activer la virtualisation imbriquée sur un hôte Windows Server 2016 est disponible sur [GitHub](https://github.com/charlieding/Virtualization-Documentation/tree/live/hyperv-tools/Nested). Le script vérifie les conditions préalables, puis configure la virtualisation imbriquée sur la machine virtuelle Azure. Un redémarrage de la machine virtuelle Azure est nécessaire pour terminer la configuration. Ce script pourrait fonctionner dans d’autres environnements, mais cela n’est pas garanti. Consultez le billet de blog Azure avec une démonstration vidéo sur la virtualisation imbriquée exécutée sur Azure ! https://aka.ms/AzureNVblog.

### <a name="option-2-configure-nested-virtualization-manually"></a>Option n°2 : Configurer manuellement la virtualisation imbriquée

1. Sur la machine virtuelle Azure, ouvrez PowerShell en tant qu’administrateur. 

2. Activez la fonctionnalité Hyper-V et les outils de gestion.

    ```powershell
    Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
    ```

    >[!WARNING] 
    >
    >Cette commande redémarre la machine virtuelle Azure. Vous perdrez votre connexion RDP au cours du processus de redémarrage.
    
3. Après le redémarrage de la machine virtuelle Azure, reconnectez-vous à votre machine virtuelle à l’aide du protocole RDP.

## <a name="set-up-internet-connectivity-for-the-guest-virtual-machine"></a>Configurer une connectivité Internet pour les machines virtuelles invitées
Créez une nouvelle carte réseau virtuelle pour la machine virtuelle invitée et configurez une passerelle NAT pour activer la connectivité Internet.

### <a name="create-a-nat-virtual-network-switch"></a>Créer un commutateur réseau virtuel NAT

1. Sur la machine virtuelle Azure, ouvrez PowerShell en tant qu’administrateur.
   
2. Créez un commutateur interne.

    ```powershell
    New-VMSwitch -Name "InternalNAT" -SwitchType Internal
    ```

3. Affichez les propriétés du commutateur et notez l’ifIndex de la nouvelle carte.

    ```powershell
    Get-NetAdapter
    ```

    ![NetAdapter](./media/virtual-machines-nested-virtualization/get-netadapter.png)

    >[!NOTE] 
    >
    >Prenez note de la valeur « ifIndex » pour le commutateur virtuel que vous venez de créer.
    
4. Créez une adresse IP pour la passerelle NAT.
    
Pour configurer la passerelle, vous avez besoin d’informations sur votre réseau :    
  * Adresse IP : l’adresse IP de la passerelle NAT spécifie l’adresse IPv4 ou IPv6 à utiliser comme adresse de passerelle par défaut pour le sous-réseau du réseau virtuel. La forme générique est a.b.c.1 (par exemple, « 192.168.0.1 »). Si la dernière position n’a pas à être.1, c’est généralement le cas (sur la base de la longueur de préfixe). En général, vous devez utiliser un espace d’adressage de réseau privé RFC 1918. 
  * PrefixLength : la longueur de préfixe de sous-réseau définit la taille du sous-réseau local (masque de sous-réseau). La longueur de préfixe de sous-réseau sera une valeur entière comprise entre 0 et 32. 0 mappe la totalité d’Internet, 32 n’autorise qu’une adresse IP mappée. Les valeurs courantes sont comprises entre 24 et 12 en fonction du nombre d’adresses IP devant être attachées à NAT. Une valeur PrefixLength courante est 24 : il s’agit d’un masque de sous-réseau 255.255.255.0.
  * InterfaceIndex : **ifIndex** est l’index d’interface du commutateur virtuel créé à l’étape précédente. 

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 13
    ```

### <a name="create-the-nat-network"></a>Créer le réseau NAT

Pour configurer la passerelle, vous devrez fournir des informations sur le réseau et la passerelle NAT :
  * Nom : Il s’agit du nom du réseau NAT. 
  * InternalIPInterfaceAddressPrefix : Le préfixe de sous-réseau NAT décrit à la fois le préfixe IP de la passerelle NAT ci-dessus, ainsi que la longueur du préfixe de sous-réseau NAT ci-dessus. La forme générique sera a.b.c.0/longueur de préfixe de sous-réseau NAT. 

Dans PowerShell, créez un nouveau réseau NAT.
```powershell
New-NetNat -Name "InternalNat" -InternalIPInterfaceAddressPrefix 192.168.0.0/24
```


## <a name="create-the-guest-virtual-machine"></a>Créer la machine virtuelle invitée

>[!IMPORTANT] 
>
>L’agent invité Azure n’est pas pris en charge sur les machines virtuelles imbriquées et peut entraîner des problèmes à la fois sur l’hôte et sur les machines virtuelles imbriquées. N’installez pas l’agent Azure sur des machines virtuelles imbriquées et n’utilisez pas une image pour la création de machines virtuelles imbriquées qui ont déjà installé l’agent invité Azure.

1. Ouvrez le Gestionnaire Hyper-V et créez une machine virtuelle. Configurez la machine virtuelle pour utiliser le nouveau réseau interne que vous avez créé.
    
    ![NetworkConfig](./media/virtual-machines-nested-virtualization/configure-networking.png)
    
2. Installez un système d’exploitation sur la machine virtuelle invitée.
    
    >[!NOTE] 
    >
    >Vous avez besoin d’un support d’installation pour le système d’exploitation à installer sur la machine virtuelle. Dans ce cas, nous utilisons Windows 10 Entreprise.

## <a name="assign-an-ip-address-to-the-guest-virtual-machine"></a>Affecter une adresse IP à la machine virtuelle invitée

Vous pouvez affecter une adresse IP à la machine virtuelle invitée en définissant une adresse IP statique sur la machine virtuelle invitée manuellement ou en configurant DHCP sur la machine virtuelle Azure pour affecter l’adresse IP de façon dynamique.

###  <a name="option-1-configure-dhcp-to-dynamically-assign-an-ip-address-to-the-guest-virtual-machine"></a>Option 1 : Configurer DHCP pour affecter dynamiquement une adresse IP à la machine virtuelle invitée
Suivez les étapes ci-dessous pour configurer DHCP sur la machine virtuelle hôte pour l’attribution dynamique d’adresses.

#### <a name="install-dhcp-server-on-the-azure-vm"></a>Installer le serveur DHCP sur la machine virtuelle Azure

1. Ouvrez le Gestionnaire de serveurs. Dans le tableau de bord, cliquez sur **Ajouter des rôles et fonctionnalités**. La fenêtre de l’Assistant Ajouter des rôles et fonctionnalités s’affiche.
  
2. Dans l’assistant, cliquez sur **Suivant** jusqu'à la page de Rôles de serveur.
  
3. Cliquez pour sélectionner la case **Serveur DHCP**, puis sur **Ajouter des fonctionnalités**, et enfin sur **Suivant** jusqu'à ce que vous terminiez l’assistant.
  
4. Cliquez sur **Installer**.

#### <a name="configure-a-new-dhcp-scope"></a>Configurer une nouvelle étendue DHCP

1. Ouvrez le gestionnaire DHCP.
  
2. Dans le volet de navigation, développez le nom du serveur, cliquez avec le bouton droit sur **IPv4**, puis cliquez sur **Nouvelle étendue**. L’assistant Nouvelle étendue s’affiche, cliquez sur **Suivant**.
  
3. Spécifiez le nom et la description de l’étendue, puis cliquez sur **Suivant**.
  
4. Définissez une plage IP pour votre serveur DHCP (par exemple, 192.168.0.100 à 192.168.0.200).
  
5. Cliquez sur **Suivant** jusqu'à la page de la passerelle par défaut. Entrez l’adresse IP que vous avez créée précédemment (par exemple, 192.168.0.1) en tant que passerelle par défaut, puis cliquez sur **Ajouter**.
  
6. Cliquez sur **Suivant** jusqu'à la fin de l’assistant, en laissant toutes les valeurs par défaut, puis cliquez sur **Terminer**.
    
### <a name="option-2-manually-set-a-static-ip-address-on-the-guest-virtual-machine"></a>Option n°2 : Définir manuellement une adresse IP statique sur la machine virtuelle invitée
Si vous ne configurez pas le protocole DHCP pour attribuer dynamiquement une adresse IP à la machine virtuelle invitée, procédez comme suit pour définir une adresse IP statique.

1. Sur la machine virtuelle Azure, ouvrez PowerShell en tant qu’administrateur.

2. Cliquez avec le bouton droit sur la machine virtuelle invitée et cliquez sur Se connecter.

3. Connectez-vous à la machine virtuelle invitée.

4. Sur la machine virtuelle invitée, ouvrez le centre réseau et partage.

5. Configurez la carte réseau pour une adresse dans la plage du réseau NAT que vous avez créé à la section précédente.

Dans cet exemple, vous utiliserez une adresse dans la plage 192.168.0.0/24.

## <a name="test-connectivity-in-guest-virtual-machine"></a>Tester la connectivité de la machine virtuelle invitée

Dans la machine virtuelle invitée, ouvrez votre navigateur et accédez à une page web.
    ![GuestVM](./media/virtual-machines-nested-virtualization/guest-virtual-machine.png)

Pour obtenir des instructions sur l’activation de la connectivité transparente entre les machines virtuelles Azure et les machines virtuelles invitées, consultez [ce document](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).
