---
title: Tester la latence des réseaux de machines virtuelles Azure dans un réseau virtuel Azure | Microsoft Docs
description: Découvrez comment tester la latence du réseau entre des machines virtuelles Azure sur un réseau virtuel
services: virtual-network
documentationcenter: na
author: steveesp
manager: Marina Lipshteyn
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/29/2019
ms.author: steveesp
ms.openlocfilehash: 77ea14097538f722569acb5a0371674776aac8e5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84687801"
---
# <a name="test-vm-network-latency"></a>Tester la latence du réseau des machines virtuelles

Pour obtenir des résultats plus précis, mesurez la latence de votre réseau de machines virtuelles Azure avec un outil spécialement conçu à cet effet. Les outils disponibles publiquement tels que SockPerf (pour Linux) et latte.exe (pour Windows) peuvent isoler et mesurer la latence du réseau tout en excluant d’autres types de latence, tels que la latence des applications. Ces outils se concentrent sur le type de trafic qui nuisent aux performances de l’application (à savoir le trafic TCP, Transmission Control Protocol, et UDP, User Datagram Protocol). 

D’autres outils de connectivité courants, tels que Ping, peuvent mesurer la latence, mais leurs résultats peuvent ne pas représenter le trafic réseau utilisé dans les charges de travail réelles. En effet, la plupart de ces outils utilisent le protocole ICMP (Internet Control Message Protocol), qui peut être traité différemment du trafic d’application et dont les résultats peuvent ne pas être applicables aux charges de travail qui utilisent TCP et UDP. 

Pour un test précis de la latence du réseau des protocoles utilisés par la plupart des applications, SockPerf (pour Linux) et latte.exe (pour Windows) produisent les résultats les plus pertinents. Cet article couvre ces deux outils.

## <a name="overview"></a>Vue d’ensemble

En utilisant deux machines virtuelles, l’une en tant qu’expéditeur et l’autre en tant que récepteur, vous créez un canal de communication bidirectionnel. Grâce à cette approche, vous pouvez envoyer et recevoir des paquets dans les deux sens et mesurer la durée des boucles (RTT).

Vous pouvez utiliser cette approche pour mesurer la latence du réseau entre deux machines virtuelles ou même entre deux ordinateurs physiques. Les mesures de latence peuvent être utiles pour les scénarios suivants :

- Établir une référence pour la latence du réseau entre les machines virtuelles déployées.
- Comparer les effets des modifications de la latence du réseau après les modifications relatives apportées :
  - au logiciel du système d’exploitation ou de la pile réseau, y compris les modifications de configuration ;
  - à la méthode de déploiement de machine virtuelle, telle que le déploiement dans une zone de disponibilité ou un groupe de placement de proximité (PPG) ;
  - aux propriétés des machines virtuelles, telles que les changements de taille ou de mise en réseau accélérée ;
  - à un réseau virtuel, tel que le routage ou le filtrage des modifications ;

### <a name="tools-for-testing"></a>aux outils de test
Pour mesurer la latence, vous disposez de deux options d’outil différentes :

* Pour les systèmes basés sur Windows : [latte.exe (Windows)](https://gallery.technet.microsoft.com/Latte-The-Windows-tool-for-ac33093b)
* Pour les systèmes basés sur Linux : [SockPerf (Linux)](https://github.com/mellanox/sockperf)

En utilisant ces outils, vous vous assurez que seuls les délais de livraison de la charge utile TCP ou UDP sont mesurés et non ICMP (test Ping) ou d’autres types de paquets qui ne sont pas utilisés par les applications et ne nuisent pas à leurs performances.

### <a name="tips-for-creating-an-optimal-vm-configuration"></a>Conseils pour la création d’une configuration optimale des machines virtuelles

Lorsque vous créez la configuration de votre machine virtuelle, gardez à l’esprit les recommandations suivantes :
- Utilisez la dernière version de Windows ou de Linux.
- Activez la mise en réseau accélérée pour de meilleurs résultats.
- Déployer des machines virtuelles avec le [groupe de placement de proximité Azure](https://docs.microsoft.com/azure/virtual-machines/linux/co-location).
- Les machines virtuelles plus volumineuses fonctionnent généralement mieux que celles plus petites.

### <a name="tips-for-analysis"></a>Conseils pour l’analyse

Lorsque vous analysez les résultats des tests, gardez à l’esprit les recommandations suivantes :

- Établissez une ligne de base dès le début, dès que le déploiement, la configuration et les optimisations sont terminés.
- Comparez toujours les nouveaux résultats à une ligne de base ou d’un test à un autre avec des modifications contrôlées.
- Répétez les tests chaque fois que des modifications sont observées ou planifiées.


## <a name="test-vms-that-are-running-windows"></a>Tester les machines virtuelles exécutant Windows

### <a name="get-latteexe-onto-the-vms"></a>Obtenir latte.exe sur les machines virtuelles

Télécharger la [dernière version de latte.exe](https://gallery.technet.microsoft.com/Latte-The-Windows-tool-for-ac33093b).

Envisagez de placer latte.exe dans un dossier distinct, tel que *c:\tools*.

### <a name="allow-latteexe-through-windows-defender-firewall"></a>Autorisez latte.exe via le pare-feu Windows Defender

Sur le *récepteur*, créez une règle Autoriser sur le pare-feu Windows Defender pour autoriser l’entrée du trafic latte.exe. Il est plus facile d’autoriser la totalité du programme latte.exe par nom que d’autoriser des ports entrants TCP spécifiques.

Autorisez latte.exe via le pare-feu Windows Defender en exécutant la commande suivante :

```cmd
netsh advfirewall firewall add rule program=<path>\latte.exe name="Latte" protocol=any dir=in action=allow enable=yes profile=ANY
```

Par exemple, si vous avez copié latte.exe dans le dossier *c:\tools*, voici la commande :

`netsh advfirewall firewall add rule program=c:\tools\latte.exe name="Latte" protocol=any dir=in action=allow enable=yes profile=ANY`

### <a name="run-latency-tests"></a>Tests de latence d’exécution

* Sur le *récepteur*, démarrez latte.exe (exécutez-le à partir de la fenêtre CMD, et non à partir de PowerShell) :

    ```cmd
    latte -a <Receiver IP address>:<port> -i <iterations>
    ```

    Des itérations à hauteur de 65 000 suffisent à renvoyer des résultats représentatifs.

    Tout numéro de port disponible est correct.

    Si la machine virtuelle possède une adresse IP de 10.0.0.4, la commande ressemble à ceci :

    `latte -a 10.0.0.4:5005 -i 65100`

* Sur *l’expéditeur*, démarrez latte.exe (exécutez-le à partir de la fenêtre CMD, et non à partir de PowerShell) :

    ```cmd
    latte -c -a <Receiver IP address>:<port> -i <iterations>
    ```

    La commande qui en résulte est la même que sur le récepteur, exception faite de l’ajout de &nbsp; *-c* pour indiquer qu’il s’agit du *client* ou de *l’expéditeur* :

    `latte -c -a 10.0.0.4:5005 -i 65100`

Attendez les résultats. Selon l’éloignement des machines virtuelles, le test peut prendre quelques minutes. Vous pouvez commencer par un nombre inférieur d'itérations pour vérifier que cela fonctionne avant d'exécuter de plus long tests.

## <a name="test-vms-that-are-running-linux"></a>Tester les machines virtuelles exécutant Linux

Pour tester les machines virtuelles exécutant Linux, utilisez [SockPerf](https://github.com/mellanox/sockperf).

### <a name="install-sockperf-on-the-vms"></a>Installer SockPerf sur les machines virtuelles

Sur les machines virtuelles Linux, *expéditeur* et *récepteur*, exécuter les commandes suivantes pour préparer SockPerf sur les machines virtuelles. Des commandes sont fournies pour le distributions principales.

#### <a name="for-red-hat-enterprise-linux-rhelcentos"></a>Pour Red Hat Enterprise Linux (RHEL)/CentOS

Exécutez les commandes suivantes :

```bash
#RHEL/CentOS - Install Git and other helpful tools
    sudo yum install gcc -y -q
    sudo yum install git -y -q
    sudo yum install gcc-c++ -y
    sudo yum install ncurses-devel -y
    sudo yum install -y automake
    sudo yum install -y autoconf
```

#### <a name="for-ubuntu"></a>Pour Ubuntu

Exécutez les commandes suivantes :

```bash
#Ubuntu - Install Git and other helpful tools
    sudo apt-get install build-essential -y
    sudo apt-get install git -y -q
    sudo apt-get install -y autotools-dev
    sudo apt-get install -y automake
    sudo apt-get install -y autoconf
```

#### <a name="for-all-distros"></a>Pour toutes les distributions

Copiez, compilez et installez SockPerf selon les étapes suivantes :

```bash
#Bash - all distros

#From bash command line (assumes Git is installed)
git clone https://github.com/mellanox/sockperf
cd sockperf/
./autogen.sh
./configure --prefix=

#make is slower, may take several minutes
make

#make install is fast
sudo make install
```

### <a name="run-sockperf-on-the-vms"></a>Exécuter SockPerf sur les machines virtuelles

Une fois l’installation de SockPerf terminée, les machines virtuelles sont prêtes à exécuter les tests de latence. 

Commencez par démarrer SockPerf sur le *récepteur*.

Tout numéro de port disponible est correct. Dans cet exemple, nous utilisons le port 12345 :

```bash
#Server/Receiver - assumes server's IP is 10.0.0.4:
sudo sockperf sr --tcp -i 10.0.0.4 -p 12345
```

Maintenant que le serveur est à l’écoute, le client peut commencer à envoyer des paquets au serveur sur le port sur lequel il écoute (port 12345 dans le cas présent).

Il suffit d’environ 100 secondes pour retourner les résultats représentatifs, comme indiqué dans l’exemple suivant :

```bash
#Client/Sender - assumes server's IP is 10.0.0.4:
sockperf ping-pong -i 10.0.0.4 --tcp -m 350 -t 101 -p 12345  --full-rtt
```

Attendez les résultats. En fonction de la distance entre les machines virtuelles, le nombre d’itérations varie. Vous pouvez commencer par des tests plus courts d’environ 5 secondes pour vérifier que cela fonctionne avant d’exécuter des tests plus longs.

Cet exemple SockPerf utilise une taille de message de 350 octets, car il s’agit d’un paquet standard. Vous pouvez ajuster la taille à une valeur supérieure ou inférieure pour obtenir des résultats qui représentent plus précisément la charge de travail qui s’exécute sur vos machines virtuelles.


## <a name="next-steps"></a>Étapes suivantes
* Améliorer la latence avec le [groupe de placement de proximité Azure](https://docs.microsoft.com/azure/virtual-machines/linux/co-location).
* Découvrez comment [Optimiser la mise en réseau pour les machines virtuelles](../virtual-network/virtual-network-optimize-network-bandwidth.md) pour votre scénario.
* Découvrez [comment la bande passante est allouée aux machines virtuelles](../virtual-network/virtual-machine-network-throughput.md).
* Pour plus d’informations, consultez la [FAQ du réseau virtuel Azure](../virtual-network/virtual-networks-faq.md).
