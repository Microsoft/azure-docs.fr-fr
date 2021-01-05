---
title: Résolution des problèmes des cibles de stockage NFS Azure HPC Cache
description: Conseils pour éviter et résoudre les erreurs de configuration et d’autres problèmes susceptibles de provoquer un échec lors de la création d’une cible de stockage NFS.
author: ekpgh
ms.service: hpc-cache
ms.topic: troubleshooting
ms.date: 03/18/2020
ms.author: v-erkel
ms.openlocfilehash: efa163a2c10a7dc93bf5d26865a0e7eb43f11dea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87082764"
---
# <a name="troubleshoot-nas-configuration-and-nfs-storage-target-issues"></a>Résolution des problèmes de configuration NAS et des cibles de stockage NFS

Cet article apporte des solutions à certaines erreurs de configuration courantes ainsi qu’à d’autres problèmes susceptibles d’empêcher Azure HPC Cache d’ajouter un système de stockage NFS comme cible de stockage.

Cet article explique comment vérifier les ports et autoriser l’accès racine à un système NAS. Il contient également des informations détaillées sur des problèmes moins courants susceptibles de provoquer l’échec de la création de cibles de stockage NFS.

> [!TIP]
> Avant de suivre ce guide, consultez [Prérequis des cibles de stockage NFS](hpc-cache-prerequisites.md#nfs-storage-requirements).

Si la solution à votre problème n’est pas indiquée ici, veuillez [ouvrir un ticket de support](hpc-cache-support-ticket.md) pour que le Support technique et Service Microsoft puisse examiner et résoudre le problème avec vous.

## <a name="check-port-settings"></a>Vérification des paramètres de port

Azure HPC Cache a besoin d’un accès en lecture/écriture à plusieurs ports UDP/TCP sur le système de stockage NAS back-end. Vérifiez que ces ports sont accessibles sur le système NAS et que le trafic vers ces ports est autorisé à travers n’importe quel pare-feu entre le système de stockage et le sous-réseau de cache. Vous devrez peut-être vous rapprocher des administrateurs réseau et pare-feu de votre centre de données pour vérifier cette configuration.

Les ports des systèmes de stockage sont différents selon les fournisseurs. Vérifiez les exigences de votre système lorsque vous configurez une cible de stockage.

En général, le cache a besoin d’accéder à ces ports :

| Protocol | Port  | Service  |
|----------|-------|----------|
| TCP/UDP  | 111   | rpcbind  |
| TCP/UDP  | 2049  | NFS      |
| TCP/UDP  | 4045  | nlockmgr |
| TCP/UDP  | 4046  | mountd   |
| TCP/UDP  | 4047  | status   |

Pour connaître les ports nécessaires à votre système, utilisez la commande ``rpcinfo`` suivante. Elle permet de lister les ports en présentant les résultats pertinents sous forme de tableau. (Utilisez l’adresse IP de votre système à la place du terme *<storage_IP>* .)

Vous pouvez lancer cette commande à partir de tout client Linux doté d’une infrastructure NFS. Si vous utilisez un client à l’intérieur du sous-réseau du cluster, il peut également vous aider à vérifier la connectivité entre le sous-réseau et le système de stockage.

```bash
rpcinfo -p <storage_IP> |egrep "100000\s+4\s+tcp|100005\s+3\s+tcp|100003\s+3\s+tcp|100024\s+1\s+tcp|100021\s+4\s+tcp"| awk '{print $4 "/" $3 " " $5}'|column -t
```

Vérifiez que tous les ports retournés par la requête ``rpcinfo`` autorisent sans restrictions le trafic provenant du sous-réseau d’Azure HPC Cache.

Vérifiez ces paramètres sur le NAS lui-même et sur tous les pare-feu entre le système de stockage et le sous-réseau de cache.

## <a name="check-root-access"></a>Vérification de l’accès racine

Azure HPC Cache a besoin d’accéder aux exportations de votre système de stockage pour créer la cible de stockage. Plus précisément, il monte les exportations sous l’identifiant utilisateur 0.

Le moyen utilisé pour autoriser cet accès varie selon les systèmes de stockage :

* Les serveurs Linux ajoutent généralement ``no_root_squash`` au chemin d’accès exporté dans ``/etc/exports``.
* Les systèmes NetApp et EMC contrôlent généralement l’accès avec des règles d’exportation liées à des adresses IP ou à des réseaux spécifiques.

Si vous utilisez des règles d’exportation, n’oubliez pas que le cache peut utiliser plusieurs adresses IP différentes du sous-réseau de cache. Autorisez l’accès à partir de toute la plage des adresses IP de sous-réseau possibles.

> [!NOTE]
> Par défaut, Azure HPC cache écrase l’accès racine. Pour plus d’informations, consultez [Configurer des paramètres de cache supplémentaires](configuration.md#configure-root-squash).

Avec votre fournisseur de stockage NAS, activez le niveau d’accès approprié pour le cache.

### <a name="allow-root-access-on-directory-paths"></a>Autorisation de l’accès racine sur les chemins des répertoires
<!-- linked in prereqs article -->

Dans le cas de systèmes NAS qui exportent des répertoires hiérarchiques, Azure HPC Cache a besoin d’un accès racine à chaque niveau d’exportation.

Prenons par exemple un système qui affiche trois exportations :

* ``/ifs``
* ``/ifs/accounting``
* ``/ifs/accounting/payroll``

L’exportation ``/ifs/accounting/payroll`` est un enfant de ``/ifs/accounting``, et ``/ifs/accounting`` un enfant de ``/ifs``.

Si vous ajoutez l’exportation ``payroll`` en tant que cible de stockage du cache HPC, le cache monte en fait ``/ifs/`` et accède au répertoire payroll à partir de là. Azure HPC Cache a donc besoin d’un accès racine à ``/ifs`` pour accéder à l’exportation ``/ifs/accounting/payroll``.

Cette exigence est liée à la façon dont le cache indexe les fichiers et évite les collisions de fichiers, à l’aide des descripteurs de fichiers fournis par le système de stockage.

Un système NAS qui effectue des exportations hiérarchiques peut fournir différents descripteurs pour le même fichier si ce dernier est récupéré à partir de différentes exportations. Supposons par exemple qu’un client monte ``/ifs/accounting`` et accède au fichier ``payroll/2011.txt``. Un autre client monte ``/ifs/accounting/payroll`` et accède au fichier ``2011.txt``. Selon la façon dont le système de stockage attribue les descripteurs de fichiers, ces deux clients peuvent recevoir le même fichier avec des descripteurs différents (un pour ``<mount2>/payroll/2011.txt`` et un pour ``<mount3>/2011.txt``).

Le système de stockage back-end conserve les alias internes des descripteurs de fichiers, mais Azure HPC Cache ne peut pas déterminer quels descripteurs de fichiers de son index font référence au même élément. Il est donc possible que le cache comporte différentes écritures pour le même fichier et applique mal les modifications, ne sachant pas qu’il s’agit du même fichier.

Pour éviter cette éventuelle collision de fichiers dans plusieurs exportations, Azure HPC Cache monte automatiquement l’exportation la plus superficielle dans le chemin (``/ifs`` dans l’exemple) et utilise le descripteur de fichier fourni à partir de cette exportation. Si plusieurs exportations ont recours au même chemin de base, Azure HPC Cache a besoin d’un accès racine à ce chemin.

## <a name="enable-export-listing"></a>Activation du listing des exportations
<!-- link in prereqs article -->

Le NAS doit lister ses exportations lorsque Azure HPC Cache le demande.

Sur la plupart des systèmes de stockage NFS, vous pouvez tester ce comportement en envoyant la requête suivante à partir d’un client Linux : ``showmount -e <storage IP address>``.

Utilisez si possible un client Linux du même réseau virtuel que votre cache.

Si cette commande ne liste pas les exportations, le cache aura des difficultés à se connecter à votre système de stockage. Rapprochez-vous de votre fournisseur NAS pour activer le listing des exportations.

## <a name="adjust-vpn-packet-size-restrictions"></a>Ajustement des restrictions de taille des paquets VPN
<!-- link in prereqs article and configuration article -->

Si un VPN se trouve entre le cache et votre appareil NAS, il risque de bloquer les paquets Ethernet de 1 500 octets de taille complète. Vous rencontrez peut-être ce problème si les gros échanges entre le NAS et l’instance Azure HPC Cache n’aboutissent pas, mais que des mises à jour plus petites fonctionnent correctement.

Il n’existe pas de moyen simple de savoir si votre système est confronté à ce problème, sauf si vous connaissez les détails de votre configuration VPN. Voici quelques méthodes qui peuvent vous aider à vérifier ce problème.

* Utilisez des renifleurs de paquets des deux côtés du VPN pour détecter les paquets qui ont été transférés avec succès.
* Si votre VPN autorise les commandes ping, vous pouvez tester l’envoi d’un paquet de taille complète.

  Exécutez une commande ping sur le VPN vers le NAS avec ces options. (Utilisez l’adresse IP de votre système de stockage à la place de la valeur *<storage_IP>* .)

   ```bash
   ping -M do -s 1472 -c 1 <storage_IP>
   ```

  Voici les options de la commande :

  * ``-M do`` : ne pas fragmenter.
  * ``-c 1`` : envoyer un seul paquet.
  * ``-s 1472`` : définir la taille de la charge utile sur 1 472 octets. Il s’agit de la taille maximale pour un paquet de 1 500 octets en tenant compte de la surcharge Ethernet.

  Une réponse correcte se présente ainsi :

  ```bash
  PING 10.54.54.11 (10.54.54.11) 1472(1500) bytes of data.
  1480 bytes from 10.54.54.11: icmp_seq=1 ttl=64 time=2.06 ms
  ```

  Si la commande ping échoue avec 1472 octets, il existe probablement un problème de taille des paquets.

Pour corriger le problème, vous devrez peut-être configurer le verrouillage MSS sur le VPN pour que le système distant détecte correctement la taille de trame maximale. Pour plus d’informations, consultez la [documentation sur les paramètres IPsec/IKE des passerelles VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md#ipsec).

Dans certains cas, la modification du paramètre MTU du cache HPC Azure sur 1400 peut être utile. Toutefois, si vous limitez la valeur MTU sur le cache, vous devez également limiter les paramètres MTU pour les clients et les systèmes de stockage principaux qui interagissent avec le cache. Pour plus d’informations, consultez [Configurer des paramètres supplémentaires de cache HPC Azure](configuration.md#adjust-mtu-value).

## <a name="check-for-acl-security-style"></a>Vérification du style de sécurité ACL

Certains systèmes NAS appliquent un style de sécurité hybride qui associe les listes de contrôle d’accès (ACL) à la sécurité POSIX ou UNIX traditionnelle.

Si votre système signale le style de sécurité UNIX ou POSIX sans inclure le sigle « ACL », ce problème ne vous concerne pas.

Dans le cas des systèmes qui utilisent des listes de contrôle d’accès, Azure HPC Cache doit effectuer un suivi de valeurs supplémentaires propres à l’utilisateur pour contrôler l’accès aux fichiers. Pour cela, il active un cache d’accès. Il n’existe pas de contrôle accessible à l’utilisateur permettant d’activer le cache d’accès, mais vous pouvez ouvrir un ticket de support pour demander qu’il soit activé pour les cibles de stockage concernées sur votre système de cache.

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez un problème non traité dans cet article, [ouvrez un ticket de support](hpc-cache-support-ticket.md) pour obtenir l’aide d’experts.
