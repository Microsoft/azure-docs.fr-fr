---
title: Fichier include
description: Fichier include
services: virtual-network
author: jimdial
ms.service: virtual-network
ms.topic: include
ms.date: 05/10/2019
ms.author: anavin
ms.custom: include file
ms.openlocfilehash: 93caf39216ef0479ec2799267a9ba8181f37f802
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84194223"
---
## <a name="add-ip-addresses-to-a-vm-operating-system"></a><a name="os-config"></a>Ajouter des adresses IP à un système d’exploitation de machine virtuelle

Connectez-vous à une machine virtuelle que vous avez créée à l’aide de plusieurs adresses IP privées. Vous devez ajouter manuellement toutes les adresses IP privées (y compris l’adresse principale) que vous avez ajoutées à la machine virtuelle. Effectuez les étapes suivantes dans le système d’exploitation de votre machine virtuelle.

### <a name="windows-server"></a>Windows Server
<details>
  <summary>Développez</summary>

1. Tapez *ipconfig /all*à l’invite de commandes.  Seule l’adresse IP privée *principale* est visible (via DHCP).
2. Saisissez *ncpa.cpl* dans l’invite de commandes pour ouvrir la fenêtre **Connexions réseau**.
3. Ouvrez les propriétés de la carte réseau appropriée : **Ethernet**.
4. Double-cliquez sur Internet Protocol version 4 (IPv4).
5. Cliquez sur **Utiliser l’adresse IP suivante** et entrez les valeurs suivantes :

    * **Adresse IP** : entrez l’adresse IP privée *principale*.
    * **Masque de sous-réseau** : définissez cette option en fonction de votre sous-réseau. Par exemple, si le sous-réseau est un sous-réseau /24, le masque de sous-réseau est 255.255.255.0.
    * **Passerelle par défaut** : première adresse IP du sous-réseau. Si votre sous-réseau est 10.0.0.0/24, l’adresse IP de la passerelle est 10.0.0.1.
    * Sélectionnez **Utiliser l’adresse de serveur DNS suivante** et saisissez les valeurs ci-dessous :
        * **Serveur DNS préféré** : saisissez 168.63.129.16 si vous n’utilisez pas votre propre serveur DNS.  Si vous utilisez votre propre serveur DNS, entrez l’adresse IP de votre serveur.  (Pour Serveur DNS auxiliaire, vous pouvez choisir n’importe quelle adresse de serveur DNS public libre.)
    * Sélectionnez le bouton **Avancé** et ajoutez des adresses IP supplémentaires. Ajoutez chaque adresse IP privée secondaire que vous avez ajoutée à l’interface réseau Azure dans une étape précédente, à l’interface réseau Windows à laquelle est attribuée l’adresse IP principale assignée à l’interface réseau Azure.

        Vous ne devez jamais assigner manuellement l’adresse IP publique assignée à une machine virtuelle Azure au sein du système d’exploitation de la machine virtuelle. Lorsque vous définissez manuellement l’adresse IP dans le système d’exploitation, assurez-vous qu’il s’agit de la même adresse que l’adresse IP privée assignée à [l’interface réseau](../articles/virtual-network/virtual-network-network-interface-addresses.md#change-ip-address-settings) Azure, ou vous pouvez perdre la connectivité à la machine virtuelle. En savoir plus sur les paramètres [d’adresse IP privée](../articles/virtual-network/virtual-network-network-interface-addresses.md#private). Vous ne devez jamais attribuer une adresse IP publique Azure au sein du système d’exploitation.

    * Cliquez sur **OK** pour fermer les paramètres TCP/IP, puis sur **OK** à nouveau pour fermer les paramètres de la carte réseau. Votre connexion RDP est rétablie.

6. Tapez *ipconfig /all*à l’invite de commandes. Vérifiez que toutes les adresses IP que vous avez ajoutées sont affichées et que le protocole DHCP est désactivé.
7. Configurez Windows pour utiliser l’adresse IP privée de la configuration IP principale dans Azure en tant qu’adresse IP principale pour Windows. Pour plus d’informations, consultez [Aucun accès à Internet à partir de la machine virtuelle Windows Azure qui possède plusieurs adresses IP](https://support.microsoft.com/help/4040882/no-internet-access-from-azure-windows-vm-that-has-multiple-ip-addresse). 

### <a name="validation-windows-server"></a>Validation (Windows Server)

Pour être sûr de pouvoir vous connecter à Internet à partir de votre configuration IP secondaire par le biais de l’adresse IP publique associée, une fois que vous l’avez ajoutée correctement à l’aide des étapes ci-dessus, utilisez la commande suivante (en remplaçant 10.0.0.7 par l’adresse IP privée secondaire) :

```bash
ping -S 10.0.0.7 outlook.com
```
>[!NOTE]
>Pour les configurations IP secondaires, vous pouvez uniquement exécuter une commande ping sur Internet si une adresse IP publique est associée à la configuration. Pour les configurations IP principales, une adresse IP publique n’est pas requise pour exécuter une commande ping sur Internet.
</details>

### <a name="linux-ubuntu-1416"></a>Linux (Ubuntu 14/16)
<details>
  <summary>Développez</summary>
Nous vous recommandons de consulter la documentation la plus récente relative de votre distribution Linux. 

1. Ouvrez une fenêtre de terminal.
2. Assurez-vous d’être l’utilisateur root. Si ce n’est pas le cas, saisissez la commande suivante :

   ```bash
   sudo -i
   ```

3. Mettez à jour le fichier de configuration de l’interface réseau (en supposant que « eth0 » est utilisé).

   * Conservez l’élément de ligne existant pour dhcp. L’adresse IP principale reste configurée telle qu’elle était précédemment.
   * Ajoutez une configuration pour une adresse IP statique supplémentaire à l’aide des commandes suivantes :

     ```bash
     cd /etc/network/interfaces.d/
     ls
     ```

     Un fichier .cfg doit s’afficher.
4. Ouvrez le fichier. Les lignes suivantes doivent figurer à la fin du fichier :

   ```bash
   auto eth0
   iface eth0 inet dhcp
   ```

5. Ajoutez les lignes suivantes après les lignes qui existent dans ce fichier :

   ```bash
   iface eth0 inet static
   address <your private IP address here>
   netmask <your subnet mask>
   ```

6. Enregistrez le fichier à l’aide de la commande suivante :

   ```bash
   :wq
   ```

7. Réinitialisez l’interface réseau à l’aide de la commande suivante :

   ```bash
   sudo ifdown eth0 && sudo ifup eth0
   ```

   > [!IMPORTANT]
   > Exécutez les scripts ifup et ifdown sur la même ligne si vous utilisez une connexion à distance.
   >

8. Vérifiez que l’adresse IP est ajoutée à l’interface réseau à l’aide de la commande suivante :

   ```bash
   ip addr list eth0
   ```

   Vous devez voir l’adresse IP que vous avez ajoutée à la liste.

### <a name="validation-ubuntu-1416"></a>Validation (Ubuntu 14/16)

Pour être sûr de pouvoir vous connecter à Internet à partir de votre configuration IP secondaire via l’adresse IP associée, utilisez la commande suivante :

```bash
ping -I 10.0.0.5 outlook.com
```
>[!NOTE]
>Pour les configurations IP secondaires, vous pouvez uniquement exécuter une commande ping sur Internet si une adresse IP publique est associée à la configuration. Pour les configurations IP principales, une adresse IP publique n’est pas requise pour exécuter une commande ping sur Internet.

Pour les machines virtuelles Linux, lorsque vous tentez de valider la connectivité sortante à partir d’une carte réseau secondaire, il se peut que vous deviez ajouter des itinéraires appropriés. Pour ce faire, de nombreuses options s’offrent à vous. Reportez-vous à la documentation appropriée pour votre distribution Linux. Voici une méthode pour effectuer cette opération :

```bash
echo 150 custom >> /etc/iproute2/rt_tables 

ip rule add from 10.0.0.5 lookup custom
ip route add default via 10.0.0.1 dev eth2 table custom

```
- Veillez à remplacer :
    - **10.0.0.5** par l’adresse IP privée à laquelle une adresse IP publique est associée
    - **10.0.0.1** par votre passerelle par défaut
    - **eth2</details> par le nom de votre carte réseau secondaire**

### <a name="linux-ubuntu-1804"></a>Linux (Ubuntu 18.04+)
<details>
  <summary>Développez</summary>
Les versions 18.04 et ultérieures d’Ubuntu sont passées à `netplan` pour la gestion du réseau du système d’exploitation. Nous vous recommandons de consulter la documentation la plus récente relative de votre distribution Linux. 

1. Ouvrez une fenêtre de terminal.
2. Assurez-vous d’être l’utilisateur root. Si ce n’est pas le cas, saisissez la commande suivante :

    ```bash
    sudo -i
    ```

3. Créez un fichier pour la deuxième interface et ouvrez-le dans un éditeur de texte :

    ```bash
    vi /etc/netplan/60-static.yaml
    ```

4. Ajoutez les lignes suivantes au fichier, en remplaçant `10.0.0.6/24` par votre IP/netmask :

    ```bash
    network:
        version: 2
        ethernets:
            eth0:
                addresses:
                    - 10.0.0.6/24
    ```

5. Enregistrez le fichier à l’aide de la commande suivante :

    ```bash
    :wq
    ```

6. Testez les modifications à l’aide de [netplan try](http://manpages.ubuntu.com/manpages/cosmic/man8/netplan-try.8.html) pour confirmer la syntaxe :

    ```bash
    netplan try
    ```

> [!NOTE]
> `netplan try` applique les modifications temporairement, puis les annule après 120 secondes. En cas de perte de connexion, patientez 120 secondes, puis rétablissez la connexion. Les modifications auront alors été annulées.

7. En supposant qu’aucun problème ne se pose avec `netplan try`, appliquez les modifications de configuration :

    ```bash
    netplan apply
    ```

8. Vérifiez que l’adresse IP est ajoutée à l’interface réseau à l’aide de la commande suivante :

    ```bash
    ip addr list eth0
    ```

    Vous devez voir l’adresse IP que vous avez ajoutée à la liste. Exemple :

    ```bash
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
        link/ether 00:0d:3a:8c:14:a5 brd ff:ff:ff:ff:ff:ff
        inet 10.0.0.6/24 brd 10.0.0.255 scope global eth0
        valid_lft forever preferred_lft forever
        inet 10.0.0.4/24 brd 10.0.0.255 scope global secondary eth0
        valid_lft forever preferred_lft forever
        inet6 fe80::20d:3aff:fe8c:14a5/64 scope link
        valid_lft forever preferred_lft forever
    ```
### <a name="validation-ubuntu-1804"></a>Validation (Ubuntu 18.04+)

Pour être sûr de pouvoir vous connecter à Internet à partir de votre configuration IP secondaire via l’adresse IP associée, utilisez la commande suivante :

```bash
ping -I 10.0.0.5 outlook.com
```
>[!NOTE]
>Pour les configurations IP secondaires, vous pouvez uniquement exécuter une commande ping sur Internet si une adresse IP publique est associée à la configuration. Pour les configurations IP principales, une adresse IP publique n’est pas requise pour exécuter une commande ping sur Internet.

Pour les machines virtuelles Linux, lorsque vous tentez de valider la connectivité sortante à partir d’une carte réseau secondaire, il se peut que vous deviez ajouter des itinéraires appropriés. Pour ce faire, de nombreuses options s’offrent à vous. Reportez-vous à la documentation appropriée pour votre distribution Linux. Voici une méthode pour effectuer cette opération :

```bash
echo 150 custom >> /etc/iproute2/rt_tables 

ip rule add from 10.0.0.5 lookup custom
ip route add default via 10.0.0.1 dev eth2 table custom

```
- Veillez à remplacer :
    - **10.0.0.5** par l’adresse IP privée à laquelle une adresse IP publique est associée
    - **10.0.0.1** par votre passerelle par défaut
    - **eth2</details> par le nom de votre carte réseau secondaire**

### <a name="linux-red-hat-centos-and-others"></a>Linux (Red Hat, CentOS, etc.)
<details>
  <summary>Développez</summary>

1. Ouvrez une fenêtre de terminal.
2. Assurez-vous d’être l’utilisateur root. Si ce n’est pas le cas, saisissez la commande suivante :

    ```bash
    sudo -i
    ```

3. Entrez votre mot de passe et suivez les instructions qui s’affichent. Une fois que vous êtes l’utilisateur root, accédez au dossier Scripts réseau avec la commande suivante :

    ```bash
    cd /etc/sysconfig/network-scripts
    ```

4. Répertoriez les fichiers ifcfg connexes à l’aide de la commande suivante :

    ```bash
    ls ifcfg-*
    ```

    Vous devez voir *ifcfg-eth0* dans la liste de fichiers.

5. Pour ajouter une adresse IP, créez un fichier de configuration pour celle-ci comme indiqué ci-dessous. Notez qu’un fichier doit être créé pour chaque configuration IP.

    ```bash
    touch ifcfg-eth0:0
    ```

6. Ouvrez le fichier *ifcfg-eth0:0* à l’aide de la commande suivante :

    ```bash
    vi ifcfg-eth0:0
    ```

7. Ajoutez du contenu dans le fichier, *eth0:0* dans cet exemple, à l’aide de la commande suivante. Veillez à mettre à jour les informations en fonction de votre adresse IP.

    ```bash
    DEVICE=eth0:0
    BOOTPROTO=static
    ONBOOT=yes
    IPADDR=192.168.101.101
    NETMASK=255.255.255.0
    ```

8. Enregistrez le fichier à l’aide de la commande suivante :

    ```bash
    :wq
    ```

9. Redémarrez les services réseau et vérifiez que les modifications ont été appliquées en exécutant les commandes suivantes :

    ```bash
    /etc/init.d/network restart
    ifconfig
    ```

    Vous devez voir l’adresse IP que vous avez ajoutée à la liste, dans le cas présent *eth0:0*.

### <a name="validation-red-hat-centos-and-others"></a>Validation (Red Hat, CentOS et autres)

Pour être sûr de pouvoir vous connecter à Internet à partir de votre configuration IP secondaire via l’adresse IP associée, utilisez la commande suivante :

```bash
ping -I 10.0.0.5 outlook.com
```
>[!NOTE]
>Pour les configurations IP secondaires, vous pouvez uniquement exécuter une commande ping sur Internet si une adresse IP publique est associée à la configuration. Pour les configurations IP principales, une adresse IP publique n’est pas requise pour exécuter une commande ping sur Internet.

Pour les machines virtuelles Linux, lorsque vous tentez de valider la connectivité sortante à partir d’une carte réseau secondaire, il se peut que vous deviez ajouter des itinéraires appropriés. Pour ce faire, de nombreuses options s’offrent à vous. Reportez-vous à la documentation appropriée pour votre distribution Linux. Voici une méthode pour effectuer cette opération :

```bash
echo 150 custom >> /etc/iproute2/rt_tables 

ip rule add from 10.0.0.5 lookup custom
ip route add default via 10.0.0.1 dev eth2 table custom

```
- Veillez à remplacer :
    - **10.0.0.5** par l’adresse IP privée à laquelle une adresse IP publique est associée
    - **10.0.0.1** par votre passerelle par défaut
    - **eth2</details> par le nom de votre carte réseau secondaire**