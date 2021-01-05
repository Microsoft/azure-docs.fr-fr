---
title: Valider le débit VPN vers un Réseau virtuel Microsoft Azure
description: Cet article vous aide à valider le débit du réseau entre vos ressources locales et une machine virtuelle Azure.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
manager: dcscontentpm
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 09/02/2020
ms.author: radwiv
ms.reviewer: chadmat;genli
ms.openlocfilehash: 2d5b51e8cfbfcb5f771e9da524231f8ddfc40a9e
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660931"
---
# <a name="how-to-validate-vpn-throughput-to-a-virtual-network"></a>Comment valider un débit VPN sur un réseau virtuel

La connexion à la passerelle VPN vous permet d'établir une connectivité entre les locaux sécurisée entre votre réseau virtuel Azure et votre infrastructure informatique locale.

Cet article montre comment valider le débit du réseau des ressources locales vers une machine virtuelle Azure.

> [!NOTE]
> Cet article a pour objet de vous aider à diagnostiquer et résoudre les problèmes courants. Si vous ne parvenez pas à résoudre le problème en utilisant les informations suivantes, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="overview"></a>Vue d’ensemble

La connexion à la passerelle VPN implique les composants suivants :

* Périphérique VPN local (afficher la liste des [ périphériques VPN validés)](vpn-gateway-about-vpn-devices.md#devicetable).
* Internet public
* Passerelle VPN Azure
* Azure VM

Le diagramme suivant illustre la connectivité logique d’un réseau local vers un réseau virtuel Azure par VPN.

![Connectivité logique du réseau client au réseau MSFT via le VPN](./media/vpn-gateway-validate-throughput-to-vnet/VPNPerf.png)

## <a name="calculate-the-maximum-expected-ingressegress"></a>Calcul des valeurs maximales attendues en entrée/sortie

1. Déterminez les exigences de débit de base de votre application.
1. Déterminez les limites de débit de votre passerelle VPN Azure. Pour obtenir de l’aide, consultez la section « Références SKU de passerelle » sur la page [À propos de la passerelle VPN](vpn-gateway-about-vpngateways.md#gwsku).
1. Déterminez les [instructions de débit de machine virtuelle Azure](../virtual-machines/sizes.md) pour la taille de votre machine virtuelle.
1. Déterminez la bande passante de votre fournisseur d’accès à Internet (FAI).
1. Calculez le débit attendu en utilisant la bande passante minimale de la machine virtuelle, de la passerelle VPN ou du fournisseur d'accès à Internet, en mégabits par seconde (/) divisés par huit (8).

Si le débit calculé ne répond pas aux exigences de débit de base de votre application, vous devez augmenter la bande passante de la ressource que vous avez identifiée comme goulot d’étranglement. Pour redimensionner une passerelle VPN Azure, consultez [Modification d’une référence SKU de passerelle](vpn-gateway-about-vpn-gateway-settings.md#gwsku). Pour redimensionner une machine virtuelle, consultez [Redimensionner une machine virtuelle](../virtual-machines/windows/resize-vm.md). Si la bande passante Internet n’est pas celle attendue, vous pouvez également contacter votre fournisseur d’accès à Internet.

> [!NOTE]
> Le débit de la passerelle VPN est un agrégat de toutes les connexions site à site, réseau virtuel à réseau virtuel et point à site.

## <a name="validate-network-throughput-by-using-performance-tools"></a>Validation du débit réseau à l’aide d’outils de performances

Cette validation doit être effectuée pendant les heures creuses, car la saturation de débit de tunnel VPN pendant le test empêche les résultats précis.

L’outil que nous utilisons pour ce test est iPerf, qui fonctionne sur Windows et Linux et possède des modes client et serveur. Il est limité à 3 Gbits/s pour les machines virtuelles Windows.

Cet outil n’effectue pas d’opérations de lecture/écriture sur le disque. Il produit uniquement le trafic TCP généré automatiquement d’une extrémité à l’autre. Il génère des statistiques basées sur le test qui mesure la bande passante disponible entre les nœuds du serveur et du client. Lorsque vous testez deux nœuds, l'un des nœuds joue le rôle du serveur et l’autre celui de client. Une fois ce test terminé, nous vous recommandons d’inverser les rôles des nœuds pour tester le débit du chargement et du téléchargement sur les deux nœuds.

### <a name="download-iperf"></a>Télécharger iPerf

Téléchargez [iPerf](https://iperf.fr/download/iperf_3.1/iperf-3.1.2-win64.zip). Pour plus d’informations, consultez la [Documentation d’iPerf](https://iperf.fr/iperf-doc.php).

 > [!NOTE]
 > Les produits tiers mentionnés dans cet article sont fabriqués par des sociétés indépendantes de Microsoft. Microsoft ne formule aucune garantie, expresse ou implicite, concernant les performances ou la fiabilité de ces produits.

### <a name="run-iperf-iperf3exe"></a>Exécutez iPerf (iperf3.exe)

1. Activez une règle NSG/ACL autorisant le trafic (pour le test d’adresses IP publiques sur une machine virtuelle Azure).

1. Sur les deux nœuds, activez une exception de pare-feu pour le port 5001.

   **Windows :** exécutez la commande suivante en tant qu’administrateur :

   ```CMD
   netsh advfirewall firewall add rule name="Open Port 5001" dir=in action=allow protocol=TCP localport=5001
   ```

   Pour supprimer la règle lorsque le test est terminé, exécutez la commande suivante :

   ```CMD
   netsh advfirewall firewall delete rule name="Open Port 5001" protocol=TCP localport=5001
   ```

   **Linux Azure :** les images Linux Azure sont dotées de pare-feux permissifs. Si une application écoute sur un port, le trafic est autorisé. Les images personnalisées qui sont sécurisées peuvent nécessiter l’ouverture explicite des ports. Les pare-feu de couche système courants pour Linux comprennent `iptables`, `ufw` et `firewalld`.

1. Sur le nœud serveur, accédez au répertoire dans lequel iperf3.exe est extrait. Exécutez ensuite iPerf en mode serveur et configurez-le pour écouter sur le port 5001 avec les commandes suivantes :

   ```CMD
   cd c:\iperf-3.1.2-win65

   iperf3.exe -s -p 5001
   ```

   > [!Note]
   > Le port 5001 peut être personnalisé pour tenir compte des restrictions de pare-feu spécifiques à votre environnement.

1. Sur le nœud client, accédez au répertoire où l’outil iperf est extrait et exécutez la commande suivante :

   ```CMD
   iperf3.exe -c <IP of the iperf Server> -t 30 -p 5001 -P 32
   ```

   Le client dirige 30 secondes de trafic sur le port 5001 vers le serveur. L’indicateur « -P » indique que nous utilisons 32 connexions simultanées pour le nœud serveur.

   L’écran suivant présente la sortie de cet exemple :

   ![Output](./media/vpn-gateway-validate-throughput-to-vnet/06theoutput.png)

1. (FACULTATIF) Pour conserver les résultats des tests, exécutez cette commande :

   ```CMD
   iperf3.exe -c IPofTheServerToReach -t 30 -p 5001 -P 32  >> output.txt
   ```

1. Au terme des étapes précédentes, effectuez les mêmes étapes en inversant les rôles, afin que le nœud du serveur soit désormais le nœud du client et inversement.

> [!Note]
> Iperf n’est pas le seul outil. [NTTTCP peut aussi être utilisé à des fins de test](../virtual-network/virtual-network-bandwidth-testing.md).

## <a name="test-vms-running-windows"></a>Tester des machines virtuelles exécutant Windows

### <a name="load-latteexe-onto-the-vms"></a>Charger Latte.exe sur les machines virtuelles

Télécharger la dernière version de [Latte.exe](https://gallery.technet.microsoft.com/Latte-The-Windows-tool-for-ac33093b)

Envisagez de placer Latte.exe dans un dossier distinct, tel que `c:\tools`

### <a name="allow-latteexe-through-the-windows-firewall"></a>Autoriser Latte.exe via le pare-feu Windows

Sur le récepteur, créez une règle d’autorisation sur le pare-feu Windows pour autoriser l’entrée du trafic Latte.exe. Il est plus facile d’autoriser la totalité du programme Latte.exe par nom que d’autoriser des ports entrants TCP spécifiques.

### <a name="allow-latteexe-through-the-windows-firewall-like-this"></a>Autorisez Latte.exe via le pare-feu Windows comme ceci

`netsh advfirewall firewall add rule program=<PATH>\latte.exe name="Latte" protocol=any dir=in action=allow enable=yes profile=ANY`

Par exemple, si vous avez copié latte.exe dans le dossier « c:\tools », voici la commande

`netsh advfirewall firewall add rule program=c:\tools\latte.exe name="Latte" protocol=any dir=in action=allow enable=yes profile=ANY`

### <a name="run-latency-tests"></a>Tests de latence d’exécution

Démarrez latte.exe sur le RÉCEPTEUR (exécutez à partir de CMD et non à partir de PowerShell) :

`latte -a <Receiver IP address>:<port> -i <iterations>`

Des itérations à hauteur de 65 Ko suffisent à renvoyer des résultats représentatifs.

Tout numéro de port disponible est correct.

Si la machine virtuelle possède une adresse IP de 10.0.0.4, la commande ressemble à ceci

`latte -c -a 10.0.0.4:5005 -i 65100`

Démarrez latte.ex sur l’EXPÉDITEUR (exécutez à partir de CMD et non à partir de PowerShell)

`latte -c -a <Receiver IP address>:<port> -i <iterations>`

La commande qui en résulte est la même que sur le récepteur, exception faite de l'ajout de « -c » pour indiquer qu’il s’agit du « client » ou de l’expéditeur

`latte -c -a 10.0.0.4:5005 -i 65100`

Attendez les résultats. Selon l’éloignement des machines virtuelles, cela peut prendre quelques minutes. Vous pouvez commencer par un nombre inférieur d'itérations pour vérifier que cela fonctionne avant d'exécuter de plus long tests.

## <a name="test-vms-running-linux"></a>Tester des machines virtuelles Azure exécutant Linux

Utilisez [SockPerf](https://github.com/mellanox/sockperf) pour tester les machines virtuelles.

### <a name="install-sockperf-on-the-vms"></a>Installer SockPerf sur les machines virtuelles

Sur les machines virtuelles Linux (EXPÉDITEUR et RÉCEPTEUR), exécutez les commandes suivantes pour préparer SockPerf sur vos machines virtuelles :

#### <a name="centos--rhel---install-git-and-other-helpful-tools"></a>CentOS / RHEL - Installer GIT et autres outils utiles

`sudo yum install gcc -y -q`
`sudo yum install git -y -q`
`sudo yum install gcc-c++ -y`
`sudo yum install ncurses-devel -y`
`sudo yum install -y automake`

#### <a name="ubuntu---install-git-and-other-helpful-tools"></a>Ubuntu - Installer GIT et autres outils utiles

`sudo apt-get install build-essential -y`
`sudo apt-get install git -y -q`
`sudo apt-get install -y autotools-dev`
`sudo apt-get install -y automake`

#### <a name="bash---all"></a>Bash - Tout

À partir de la ligne de commande bash (ce qui suppose l'installation de git)

`git clone https://github.com/mellanox/sockperf`
`cd sockperf/`
`./autogen.sh`
`./configure --prefix=`

Make est plus lent, peut prendre quelques minutes

`make`

L'installation de Make est rapide

`sudo make install`

### <a name="run-sockperf-on-the-vms"></a>Exécuter SockPerf sur les machines virtuelles

#### <a name="sample-commands-after-installation-serverreceiver---assumes-servers-ip-is-10004"></a>Exemples de commandes après installation. Serveur/Récepteur - suppose que l’adresse IP du serveur est 10.0.0.4

`sudo sockperf sr --tcp -i 10.0.0.4 -p 12345 --full-rtt`

#### <a name="client---assumes-servers-ip-is-10004"></a>Client - suppose que l’adresse IP du serveur est 10.0.0.4

`sockperf ping-pong -i 10.0.0.4 --tcp -m 1400 -t 101 -p 12345  --full-rtt`

> [!Note]
> Assurez-vous qu’il n’existe aucun tronçon intermédiaire (appliance virtuelle, par exemple) pendant le test de débit entre la machine virtuelle et la passerelle.
> Si les tests iPERF/NTTTCP ci-dessus génèrent de mauvais résultats (en termes de débit global), reportez-vous à l’article suivant pour comprendre les facteurs clés susceptibles d'être à l'origine du problème : https://docs.microsoft.com/azure/virtual-network/virtual-network-tcpip-performance-tuning

L’analyse des suivis de captures paquets (Wireshark/Moniteur réseau) collectés en parallèle à partir du client et du serveur lors de ces tests vous aidera notamment à évaluer de mauvaises performances. Ces suivis peuvent inclure la perte de paquets, une latence élevée et la taille de la MTU. Fragmentation, fenêtre TCP 0, fragments en désordre, et ainsi de suite.

## <a name="address-slow-file-copy-issues"></a>Résolution des problèmes de copie lente des fichiers

Même si le débit global évalué lors des étapes précédentes (iPERF/NTTTCP/etc.) était correct, vous risquez de constater une lenteur de copie des fichiers lorsque vous utilisez l’Explorateur Windows ou une opération de glisser-déplacer via une session Bureau à distance. Ce problème est généralement dû à un ou deux des facteurs suivants :

* Les applications de copie de fichiers, comme l’Explorateur Windows et le protocole RDP, n’utilisent pas plusieurs threads lors de la copie des fichiers. Pour de meilleures performances, utilisez une application de copie de fichiers multi-thread, comme [Richcopy](/previous-versions/technet-magazine/dd547088(v=msdn.10)), pour copier des fichiers à l’aide de 16 ou 32 threads. Pour modifier le nombre de threads pour la copie de fichiers dans Richcopy, cliquez sur **Action** > **Options de copie** > **Copie de fichiers**.

   ![Problèmes de copie lente de fichiers](./media/vpn-gateway-validate-throughput-to-vnet/Richcopy.png)<br>

   > [!Note]
   > Toutes les applications ne fonctionnent pas de la même façon et tous les processus/applications n’utilisent pas l'ensemble des threads. Lors de l’exécution du test, vous pouvez remarquer des threads vides et dès lors, les résultats en termes de débit manqueront de précision.
   > Pour vérifier les performances de transfert de fichiers de votre application, utilisez le multithread en augmentant le nombre de threads successifs ou en le diminuant pour déterminer le débit optimal de l'application ou du transfert de fichiers.

* La vitesse en lecture/écriture du disque de la machine virtuelle est insuffisante. Pour plus d'informations, consultez [Dépannage Azure Storage](/previous-versions/azure/storage/common/storage-e2e-troubleshooting).

## <a name="on-premises-device-external-facing-interface"></a>Interface externe avec appareil local

Indiquez les sous-réseaux de plages locales que vous souhaitez qu'Azure atteigne via le VPN ou la passerelle de réseau local. Dans le même temps, définissez l’espace d’adressage du réseau virtuel dans Azure sur l’appareil local.

* **Passerelle basée sur un itinéraire** : La stratégie ou le sélecteur de trafic pour les VPN basés sur l'itinéraire sont configurés comme universels (ou en caractères génériques).

* **Passerelle basée sur une stratégie** : Les VPN basés sur des stratégies chiffrent et acheminent les paquets via les tunnels IPsec basés sur des combinaisons de préfixes d’adresses entre votre réseau local et le réseau virtuel Azure. La stratégie (ou le sélecteur de trafic) est généralement définie en tant que liste d'accès dans les configurations VPN.

* Connexions **UsePolicyBasedTrafficSelector** :la définition du paramètre « UsePolicyBasedTrafficSelectors » sur $True sur une connexion a pour effet de configurer la passerelle VPN Azure pour se connecter à un pare-feu VPN basé sur une stratégie en local. Si vous activez UsePolicyBasedTrafficSelectors, vous devez vous assurer que votre périphérique VPN dispose des sélecteurs de trafic correspondant définis avec toutes les combinaisons de préfixes de réseau local (passerelle réseau locale) à destination vers et depuis des préfixes du réseau virtuel Azure, plutôt que de manière indifférenciée.

Une configuration inappropriée peut entraîner des déconnexions fréquentes dans le tunnel, des abandons de paquets, un débit et une latence médiocres.

## <a name="check-latency"></a>Vérifier la latence

Vous pouvez vérifier la latence à l’aide des outils suivants :

* WinMTR
* TCPTraceroute
* `ping` et `psping` (ces outils offrent une bonne estimation de la RTT, mais ne peuvent pas être systématiquement utilisés).

![Vérifier la latence](./media/vpn-gateway-validate-throughput-to-vnet/08checkinglatency.png)

Si vous remarquez un pic de latence au niveau d'un des tronçons avant accès au réseau principal MS, vous pouvez effectuer des investigations supplémentaires avec votre fournisseur de services Internet.

En présence d’un pic de latence anormalement élevé au niveau des tronçons de « msn.net », contactez le support MS pour un examen approfondi.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations ou d'aide, consultez le lien suivant :

* [Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)