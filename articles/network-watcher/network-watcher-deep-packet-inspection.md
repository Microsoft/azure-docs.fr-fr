---
title: Inspection de paquets avec Azure Network Watcher | Microsoft Docs
description: Cet article explique comment utiliser Network Watcher pour effectuer une inspection approfondie des paquets collectés à partir d’une machine virtuelle
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: 7b907d00-9c35-40f5-a61e-beb7b782276f
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 2a4debda712636e577639b84845dda6b9b4228a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84725053"
---
# <a name="packet-inspection-with-azure-network-watcher"></a>Inspection de paquets avec Azure Network Watcher

À l’aide de la fonctionnalité de capture de paquets de Network Watcher, vous pouvez lancer et gérer des sessions de captures sur vos machines virtuelles Azure à partir du portail, de PowerShell, de l’interface CLI et par programme via le kit de développement logiciel (SDK) et l’API REST. La capture de paquets vous permet de gérer les scénarios qui requièrent des données au niveau des paquets en fournissant les informations dans un format facilement utilisable. En exploitant les outils disponibles gratuitement pour examiner les données, vous pouvez étudier les communications envoyées en direction et en provenance de vos machines virtuelles et obtenir des informations sur votre trafic réseau. Exemples d’utilisation des données de capture de paquets : étude des problèmes de réseau ou d’application, détection des tentatives d’utilisation abusive et d’intrusion sur le réseau ou maintien de la conformité aux réglementations. Dans cet article, nous expliquons comment ouvrir un fichier de capture de paquets fourni par Network Watcher à l’aide d’un outil open source connu. Nous fournirons également des exemples montrant comment calculer une latence de connexion, identifier un trafic anormal et examiner les statistiques réseau.

## <a name="before-you-begin"></a>Avant de commencer

Cet article aborde certains scénarios préconfigurés dans une capture de paquets exécutée précédemment. Ces scénarios illustrent les fonctionnalités qui sont accessibles en consultant une capture de paquets. Ce scénario utilise [WireShark](https://www.wireshark.org/) pour inspecter la capture des paquets.

Ce scénario part du principe que vous avez déjà exécuté une capture de paquets sur une machine virtuelle. Pour découvrir comment créer une capture de paquets, consultez [Manage packet captures with the portal](network-watcher-packet-capture-manage-portal.md) (Gérer les captures de paquets avec le portail). Pour utiliser l’API REST, consultez [Managing Packet Captures with REST API](network-watcher-packet-capture-manage-rest.md) (Gérer les captures de paquets avec l’API REST).

## <a name="scenario"></a>Scénario

Dans ce scénario, vous allez :

* Examiner une capture de paquets

## <a name="calculate-network-latency"></a>Calculer la latence du réseau

Dans ce scénario, nous expliquons comment afficher la durée initiale des boucles d’une conversation de protocole TCP entre deux points de terminaison.

Lorsqu’une connexion TCP est établie, les trois premiers paquets envoyés via la connexion suivent un modèle dit de « connexion en trois temps ». En examinant les deux premiers paquets envoyés via cette connexion, une demande initiale du client et une réponse du serveur, nous pouvons calculer la latence lors de l’établissement de cette connexion. Cette latence correspond à la durée des boucles. Pour plus d’informations sur le protocole TCP et la connexion en trois temps, reportez-vous à la ressource suivante : [https://support.microsoft.com/en-us/help/172983/explanation-of-the-three-way-handshake-via-tcp-ip](https://support.microsoft.com/en-us/help/172983/explanation-of-the-three-way-handshake-via-tcp-ip)

### <a name="step-1"></a>Étape 1

Lancez WireShark.

### <a name="step-2"></a>Étape 2

Chargez le fichier **.cap** à partir de votre capture de paquets. Ce fichier se trouve dans l’objet blob dans lequel il a été enregistré localement sur la machine virtuelle, selon sa configuration.

### <a name="step-3"></a>Étape 3 :

Pour afficher la durée initiale des boucles dans les conversations TCP, nous examinerons ici uniquement les deux premiers paquets impliqués dans l’établissement de la liaison TCP. Nous allons utiliser les deux premiers paquets de la connexion en trois temps, à savoir les paquets [SYN] et [SYN, ACK]. Ils sont nommés pour les indicateurs définis dans l’en-tête TCP. Le dernier paquet de la connexion, le paquet [ACK], ne sera pas utilisé dans ce scénario. Le paquet [SYN] est envoyé par le client. Une fois le paquet reçu, le serveur envoie le paquet [ACK] en guise d’accusé de réception du paquet SYN du client. Profitant du fait que la réponse du serveur nécessite très peu de traitement, nous calculons la durée des boucles en soustrayant l’heure à laquelle le paquet [SYN, ACK] a été reçu par le client et l’heure à laquelle le paquet [SYN] a été envoyé par le client.

Si vous utilisez WireShark, cette valeur est calculée automatiquement.

Pour afficher plus facilement les deux premiers paquets dans la connexion en trois temps TCP, nous allons utiliser la fonction de filtrage proposée par WireShark.

Pour appliquer le filtre dans WireShark, développez le segment « protocole TCP » d’un paquet [SYN] dans votre capture et examinez les indicateurs définis dans l’en-tête TCP.

Étant donné que nous cherchons à filtrer tous les paquets [SYN] et [SYN, ACK], dans les indicateurs, vérifiez que le bit Syn est défini sur 1, puis cliquez avec le bouton droit sur le bit Syn-> Apply as Filter (Appliquer en tant que filtre) -> Sélectionné.

![figure 7][7]

### <a name="step-4"></a>Étape 4

Maintenant que vous avez filtré la fenêtre pour afficher uniquement les paquets avec le bit [SYN] défini, vous pouvez facilement sélectionner les conversations qui vous intéressent afin de consulter la durée initiale des boucles. Un moyen simple d’afficher la durée des boucles dans WireShark consiste à cliquer sur la liste déroulante nommée « SEQ/ACK analysis » (Analyse SEQ/ACK). La durée des boucles s’affiche alors. Dans ce cas, la durée des boucles a été de 0,0022114 secondes ou 2,211 ms.

![figure 8][8]

## <a name="unwanted-protocols"></a>Protocoles indésirables

De nombreuses applications peuvent être exécutées sur une instance de machine virtuelle que vous avez déployée dans Azure. Plusieurs de ces applications communiquent sur le réseau, parfois sans votre autorisation explicite. En utilisant la capture de paquets pour stocker les communications réseau, nous pouvons étudier la manière dont les applications communiquent sur le réseau et identifier les éventuels problèmes.

Dans cet exemple, nous allons examiner une capture de paquets précédemment exécutée afin d’identifier les protocoles indésirables susceptibles d’indiquer une communication non autorisée à partir d’une application en cours d’exécution sur votre ordinateur.

### <a name="step-1"></a>Étape 1

Dans la même capture que celle du scénario précédent, cliquez sur **Statistiques** > **Protocol Hierarchy** (Hiérarchie des protocoles).

![menu Hiérarchie des protocoles][2]

La fenêtre de la hiérarchie des protocoles s’affiche. Cette vue fournit la liste de tous les protocoles qui étaient en cours d’utilisation lors de la session de capture et le nombre de paquets transmis et reçus à l’aide des protocoles. Cette vue peut être utile pour détecter le trafic réseau indésirable sur vos machines virtuelles ou sur le réseau.

![Hiérarchie des protocoles ouverte][3]

Comme vous pouvez le voir dans la capture d’écran suivante, un trafic utilisant le protocole BitTorrent (utilisé pour le partage de fichiers en pair à pair) a été enregistré. En tant qu’administrateur, vous ne vous attendez pas à voir un trafic BitTorrent sur cette machine virtuelle spécifique. Maintenant que vous avez connaissance de ce trafic, vous pouvez supprimer le logiciel pair à pair installé sur cette machine virtuelle ou bloquer le trafic à l’aide de groupes de sécurité réseau ou d’un pare-feu. En outre, vous pouvez choisir d’exécuter des captures de paquets planifiées, afin de pouvoir vérifier régulièrement l’utilisation des protocoles sur vos machines virtuelles. Pour obtenir un exemple d’automatisation des tâches réseau dans Azure, consultez la page [Monitor network resources with azure automation](network-watcher-monitor-with-azure-automation.md) (Surveiller les ressources réseau à l’aide d’Azure Automation).

## <a name="finding-top-destinations-and-ports"></a>Identification des principaux ports et destinations

Identifier les types de trafic, les points de terminaison et les ports utilisés pour la communication est essentiel lors de la surveillance ou de la résolution des problèmes des applications et des ressources sur votre réseau. En utilisant un fichier de capture de paquets tel que ceux ci-dessus, il est possible de déterminer rapidement les principales destinations avec lesquelles votre machine virtuelle communique ainsi que les ports utilisés.

### <a name="step-1"></a>Étape 1

Dans la même capture que celle du scénario précédent, cliquez sur **Statistiques** > **IPv4 Statistics (Statistiques IPv4)**  > **Destinations and Ports (Destinations et ports)** .

![fenêtre de capture de paquets][4]

### <a name="step-2"></a>Étape 2

En observant les résultats, on peut identifier une ligne particulièrement intéressante : plusieurs connexions ont été enregistrées sur le port 111. Le port le plus utilisé a été le 3389, qui correspond au Bureau à distance, et les autres sont des ports dynamiques RPC.

Bien que ce trafic puisse être insignifiant, il s’agit d’un port qui a été utilisé pour un grand nombre de connexions et qui est inconnu de l’administrateur.

![figure 5][5]

### <a name="step-3"></a>Étape 3 :

Maintenant que nous avons identifié un port utilisé de manière inhabituelle, nous pouvons filtrer notre capture sur celui-ci.

Dans ce scénario, le filtre serait :

```
tcp.port == 111
```

Entrez le texte du filtre ci-dessus dans la zone de texte de filtre, puis appuyez sur ENTRÉE.

![figure 6][6]

Les résultats indiquent que la totalité du trafic provient d’une machine virtuelle locale sur le même sous-réseau. Si vous ne comprenez toujours pas l’origine de ce trafic, vous pouvez inspecter les paquets de manière plus approfondie pour déterminer la raison pour laquelle ces appels sont effectués sur le port 111. Grâce à ces informations, vous pouvez déterminer l’action appropriée.

## <a name="next-steps"></a>Étapes suivantes

Découvrez les autres fonctionnalités de diagnostic de Network Watcher en consultant la page [Azure network monitoring overview](network-watcher-monitoring-overview.md) (Présentation de la surveillance du réseau Azure)

[1]: ./media/network-watcher-deep-packet-inspection/figure1.png
[2]: ./media/network-watcher-deep-packet-inspection/figure2.png
[3]: ./media/network-watcher-deep-packet-inspection/figure3.png
[4]: ./media/network-watcher-deep-packet-inspection/figure4.png
[5]: ./media/network-watcher-deep-packet-inspection/figure5.png
[6]: ./media/network-watcher-deep-packet-inspection/figure6.png
[7]: ./media/network-watcher-deep-packet-inspection/figure7.png
[8]: ./media/network-watcher-deep-packet-inspection/figure8.png













