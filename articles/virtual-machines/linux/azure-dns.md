---
title: Options de résolution de noms DNS pour les machines virtuelles Linux
description: Scénarios de résolution de noms pour les machines virtuelles Linux dans Azure IaaS, notamment les services DNS fournis, le DNS externe hybride et l’apport de son propre serveur DNS.
author: RicksterCDN
ms.service: virtual-machines-linux
ms.topic: conceptual
ms.date: 10/19/2016
ms.author: rclaus
ms.openlocfilehash: aa007888c68df41242f937e1062a90ec1b7fc3ce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87372822"
---
# <a name="dns-name-resolution-options-for-linux-virtual-machines-in-azure"></a>Options de résolution de noms DNS pour les machines virtuelles Linux dans Azure
Azure fournit une résolution des noms DNS par défaut pour toutes les machines virtuelles d’un même réseau virtuel. Vous pouvez implémenter votre propre solution de résolution de noms DNS en configurant vos propres services DNS sur vos machines virtuelles hébergées sur Azure. Les scénarios suivants vous aideront à choisir la solution qui fonctionne dans votre situation.

* [Résolution de noms fournie par Azure](#name-resolution-that-azure-provides)
* [Résolution de noms à l’aide de votre propre serveur DNS](#name-resolution-using-your-own-dns-server)

Le type de résolution de noms que vous utilisez dépend de la façon dont vos machines virtuelles et vos instances de rôle doivent communiquer entre elles.

Le tableau suivant montre des scénarios et les solutions de résolution de noms correspondantes :

| **Scénario** | **Solution** | **Suffixe** |
| --- | --- | --- |
| Résolution de noms entre des instances de rôle ou des machines virtuelles situées dans le même réseau virtuel |Résolution de noms fournie par Azure |nom d’hôte ou nom de domaine complet |
| Résolution de noms entre des instances de rôle ou des machines virtuelles situées dans des réseaux virtuels différents |Serveurs DNS gérés par le client qui transfèrent les requêtes entre les réseaux virtuels pour une résolution par Azure (proxy DNS). Consultez [Résolution de noms à l’aide de votre propre serveur DNS](#name-resolution-using-your-own-dns-server). |Nom de domaine complet uniquement |
| Résolution des noms de service et d’ordinateur locaux à partir des instances de rôle ou des machines virtuelles dans Azure |Serveurs DNS gérés par le client (par exemple contrôleur de domaine local, contrôleur de domaine en lecture seule local ou serveur DNS secondaire synchronisé via des transferts de zone). Consultez [Résolution de noms à l’aide de votre propre serveur DNS](#name-resolution-using-your-own-dns-server). |Nom de domaine complet uniquement |
| Résolution de noms d’hôte Azure à partir d’ordinateurs locaux |Transfert des requêtes à un serveur proxy DNS géré par le client dans le réseau virtuel correspondant. Le serveur proxy transfère les requêtes à Azure pour résolution. Consultez [Résolution de noms à l’aide de votre propre serveur DNS](#name-resolution-using-your-own-dns-server). |Nom de domaine complet uniquement |
| DNS inversé pour les adresses IP internes |[Résolution de noms à l’aide de votre propre serveur DNS](#name-resolution-using-your-own-dns-server) |n/a |

## <a name="name-resolution-that-azure-provides"></a>Résolution de noms fournie par Azure
En plus de la résolution de noms DNS publics, Azure fournit la résolution de noms interne pour les machines virtuelles et les instances de rôle qui se trouvent dans le même réseau virtuel. Dans les réseaux virtuels basés sur Azure Resource Manager, le suffixe DNS est cohérent à travers le réseau virtuel ; le nom de domaine complet n’est pas nécessaire. Les noms DNS peuvent être affectés à la fois à des cartes réseau et à des machines virtuelles. Bien que la résolution de noms fournie par Azure ne nécessite aucune configuration, elle n’est pas appropriée dans tous les scénarios de déploiement, comme le montre le tableau précédent.

### <a name="features-and-considerations"></a>Fonctionnalités et considérations
**Fonctionnalités :**

* Aucune configuration n’est nécessaire pour utiliser la résolution de noms fournie par Azure.
* Le service de résolution de noms fournie par Azure bénéficie d’une haute disponibilité. Vous n’avez pas besoin de créer et de gérer les clusters de vos propres serveurs DNS.
* Le service de résolution de noms fourni par Azure peut être utilisé avec vos propres serveurs DNS pour résoudre à la fois les noms d’hôte locaux et les noms d’hôte Azure.
* La résolution de noms est effectuée entre les machines virtuelles des réseaux virtuels sans qu’un nom de domaine complet soit nécessaire.
* Vous pouvez utiliser des noms d’hôtes qui décrivent vos déploiements de façon plus appropriée, au lieu de noms générés automatiquement.

**Considérations :**

* Le suffixe DNS créé par Azure ne peut pas être modifié.
* Vous ne pouvez pas enregistrer manuellement vos propres enregistrements.
* WINS et NetBIOS ne sont pas pris en charge.
* Les noms d’hôte doivent être compatibles avec DNS.
    Les noms doivent comporter uniquement les caractères 0-9, a-z et « - », et ils ne peuvent pas commencer ou se terminer par « - ». Consultez RFC 3696 Section 2.
* Le trafic des requêtes DNS est limité pour chaque machine virtuelle. Cette limitation ne devrait pas avoir d’incidence sur la plupart des applications.  Si la limitation de requêtes est respectée, assurez-vous que la mise en cache côté client est activée.  Pour plus d’informations, consultez [Tirer le meilleur parti de la résolution de noms fournie par Azure](#getting-the-most-from-name-resolution-that-azure-provides).

### <a name="getting-the-most-from-name-resolution-that-azure-provides"></a>Tirer le meilleur parti de la résolution de noms fournie par Azure
**Mise en cache côté client :**

Certaines requêtes DNS ne sont pas envoyées sur le réseau. La mise en cache côté client permet de réduire la latence et d’améliorer la résilience face aux incohérences réseau en résolvant les requêtes DNS récurrentes à partir d’un cache local. Les enregistrements DNS ont une durée de vie, qui permet au cache de les stocker aussi longtemps que possible sans impact sur leur actualité. Pour cette raison, la mise en cache côté client convient donc à la plupart des situations.

Certaines distributions Linux n’incluent pas la mise en cache par défaut. Nous vous recommandons d’ajouter un cache à chaque machine virtuelle de Linux après avoir vérifié qu’il n’existe pas déjà un cache local.

Plusieurs packages de mise en cache DNS sont disponibles, comme dnsmasq. Voici les étapes nécessaires pour installer dnsmasq sur les distributions plus courantes :

**Ubuntu (utilise resolvconf)**
  * Installez le package dnsmasq (« sudo apt-get install dnsmasq »).

**SUSE (utilise netconf)** :
1. Installez le package dnsmasq (« sudo zypper install dnsmasq »).
2. Activez le service dnsmasq (« systemctl enable dnsmasq.service »).
3. Démarrez le service dnsmasq (« systemctl start dnsmasq.service »).
4. Éditez « /etc/sysconfig/network/config » et remplacez NETCONFIG_DNS_FORWARDER="" par NETCONFIG_DNS_FORWARDER="dnsmasq".
5. Mettez à jour resolv.conf (« netconfig update ») pour définir le cache en tant que résolveur DNS local.

**CentOS de Rogue Wave Software (anciennement OpenLogic, utilise NetworkManager)**
1. Installez le package dnsmasq (« sudo yum install dnsmasq »).
2. Activez le service dnsmasq (« systemctl enable dnsmasq.service »).
3. Démarrez le service dnsmasq (« systemctl start dnsmasq.service »).
4. Ajoutez « prepend domain-name-servers 127.0.0.1; » à « /etc/dhclient-eth0.conf ».
5. Redémarrez le service réseau (« service network restart ») pour définir le cache en tant que résolveur DNS local.

> [!NOTE]
> : Le package « dnsmasq » constitue l’un des nombreux caches DNS disponibles pour Linux. Avant de l’utiliser, vérifiez son adéquation à vos besoins et qu’aucun autre cache n’est installé.
>
>

**Nouvelles tentatives côté client**

DNS est principalement un protocole UDP. Comme le protocole UDP ne garantit pas la remise des messages, le protocole DNS gère lui-même la logique de nouvelle tentative. Chaque client DNS (son système d’exploitation) peut appliquer une logique de nouvelle tentative différente selon la préférence de son créateur :

* Les systèmes d’exploitation Windows effectuent une nouvelle tentative après une seconde, puis à nouveau après deux secondes, quatre secondes et encore quatre secondes.
* La configuration Linux par défaut effectue une nouvelle tentative après cinq secondes.  Vous devez modifier ce paramètre pour effectuer cinq nouvelles tentatives à des intervalles d’une seconde.  

Pour vérifier les paramètres actuels sur une machine virtuelle Linux, exécutez la commande « cat /etc/resolv.conf » et regardez la ligne « options », par exemple :

```config-conf
options timeout:1 attempts:5
```

Le fichier resolv.conf est généré automatiquement et ne doit pas être modifié. Les étapes spécifiques pour l’ajout de la ligne « options » varient selon la distribution :

**Ubuntu** (utilise resolvconf)
1. Ajoutez la ligne options à « /etc/resolvconf/resolv.conf.d/head ».
2. Exécutez « resolvconf -u » pour mettre à jour.

**SUSE** (utilise netconf)
1. Ajoutez « timeout:1 attempts:5 » au paramètre NETCONFIG_DNS_RESOLVER_OPTIONS="" dans « /etc/sysconfig/network/config ».
2. Exécutez « netconfig update » pour mettre à jour.

**CentOS de Rogue Wave Software (anciennement OpenLogic)** (utilise NetworkManager)
1. Ajoutez ’RES_OPTIONS="timeout:1 attempts:5"’ to ’/etc/sysconfig/network’.
2. Exécutez « service network restart » pour mettre à jour.

## <a name="name-resolution-using-your-own-dns-server"></a>Résolution de noms à l’aide de votre propre serveur DNS
Vos besoins en matière de résolution de noms peuvent aller au-delà des fonctions fournies par Azure. Par exemple, vous pouvez avoir besoin d’une résolution DNS entre des réseaux virtuels. Pour couvrir ce scénario, vous pouvez utiliser vos propres serveurs DNS.  

Les serveurs DNS dans un réseau virtuel peuvent transférer les requêtes DNS vers les résolveurs récursifs d’Azure pour résoudre les noms d’hôtes qui se trouvent dans le même réseau virtuel. Par exemple, un serveur DNS qui s’exécute dans Azure peut répondre aux requêtes DNS concernant les fichiers de sa propre zone DNS et transférer toutes les autres requêtes à Azure. Cette fonctionnalité permet aux machines virtuelles de voir vos entrées dans les fichiers de votre zone et les noms d’hôtes fournis par Azure (via le redirecteur). Les résolveurs récursifs d’Azure sont accessibles via l’adresse IP virtuelle 168.63.129.16.

Le transfert DNS permet aussi la résolution DNS entre des réseaux virtuels et permet à vos machines locales de résoudre les noms d’hôtes fournis par Azure. Pour résoudre le nom d’hôte d’une machine virtuelle, la machine virtuelle du serveur DNS doit se trouver dans le même réseau virtuel et être configurée pour transférer les requêtes de nom d’hôte à Azure. Comme le suffixe DNS est différent dans chaque réseau virtuel, vous pouvez utiliser des règles de redirection conditionnelles pour envoyer les requêtes DNS au réseau virtuel approprié en vue de la résolution. L’image suivante montre deux réseaux virtuels et un réseau local effectuant une résolution DNS entre réseaux virtuels à l’aide de cette méthode :

![Résolution DNS entre réseaux virtuels](./media/azure-dns/inter-vnet-dns.png)

Quand vous utilisez la résolution de noms fournie par Azure, le suffixe DNS interne est fourni à chaque machine virtuelle via DHCP. Quand vous utilisez votre propre solution de résolution de noms, ce suffixe n’est pas fourni aux machines virtuelles, car il interfère avec d’autres architectures DNS. Pour référencer des machines par le nom de domaine complet ou pour configurer le suffixe sur vos machines virtuelles, vous pouvez utiliser PowerShell ou l’API pour déterminer le suffixe :

* Pour les réseaux virtuels gérés par Azure Resource Manager, le suffixe est disponible via la ressource de [carte réseau](/rest/api/virtualnetwork/networkinterfaces). Vous pouvez également exécuter la commande `azure network public-ip show <resource group> <pip name>` pour afficher les détails de votre adresse IP publique, qui inclut le nom de domaine complet de la carte réseau.

Si la redirection des requêtes vers Azure ne suffit pas, vous devez fournir votre propre solution DNS.  Votre solution DNS doit :

* Fournir la résolution de nom d’hôte approprié, par exemple par le biais de [DDNS](../../virtual-network/virtual-networks-name-resolution-ddns.md). Si vous utilisez DDNS, il peut être nécessaire de désactiver le nettoyage des enregistrements DNS. Les baux DHCP d’Azure sont très longs et le nettoyage peut supprimer prématurément des enregistrements DNS.
* Fournir la résolution récursive appropriée pour permettre la résolution de noms de domaines externes.
* Être accessible (TCP et UDP sur le port 53) à partir des clients dont elle traite les demandes et être en mesure d’accéder à Internet.
* Être protégée contre tout accès à partir d’Internet, pour atténuer les menaces posées par les agents externes.

> [!NOTE]
> Pour de meilleures performances, quand vous utilisez des machines virtuelles sur des serveurs DNS Azure, désactivez IPv6 et affectez une [adresse IP publique de niveau instance](/previous-versions/azure/virtual-network/virtual-networks-instance-level-public-ip) à chaque machine virtuelle du serveur DNS.  
>
>
