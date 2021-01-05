---
title: Configuration d’un DNS pour Avere vFXT - Azure
description: Découvrez comment configurer un serveur DNS pour l’équilibrage de charge en tourniquet (round-robin) avec Avere vFXT pour Azure
author: ekpgh
ms.service: avere-vfxt
ms.topic: conceptual
ms.date: 12/19/2019
ms.author: rohogue
ms.openlocfilehash: 81b53904f85e2ac936195b1e39d7586fd1d47524
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009022"
---
# <a name="avere-cluster-dns-configuration"></a>Configuration d’un DNS pour le cluster Avere

Cette section explique les principes de base de la configuration d’un système DNS pour l’équilibrage de charge de votre cluster Avere vFXT.

Ce document *n’inclut pas* d’instructions pour la configuration et la gestion d’un serveur DNS dans l’environnement Azure.

Au lieu d’utiliser le DNS en tourniquet pour équilibrer la charge d’un cluster vFXT dans Azure, assignez manuellement les adresses IP en les répartissant entre les différents clients montés. Plusieurs méthodes d’assignation sont décrites dans [Monter le cluster Avere](avere-vfxt-mount-clients.md).

Pour déterminer si vous devez ou non utiliser un serveur DNS, prenez en compte les considérations suivantes :

* Si votre système est accessible par des clients NFS uniquement, l’utilisation d’un DNS n’est pas nécessaire, car il est possible de spécifier toutes les adresses réseau à l’aide d’adresses IP numériques.

* Si votre système prend en charge l’accès SMB (CIFS), un DNS est nécessaire, car vous devez alors spécifier un domaine DNS pour le serveur Active Directory.

* Un DNS est nécessaire pour utiliser l’authentification Kerberos.

## <a name="load-balancing"></a>Équilibrage de la charge

Pour répartir la charge globale, configurez votre domaine DNS afin qu’il utilise l’équilibrage de charge en tourniquet (round-robin) pour répartir les adresses IP accessibles par les clients.

## <a name="configuration-details"></a>Détails de configuration

Quand les clients accèdent au cluster, RRDNS répartit automatiquement leurs requêtes entre toutes les interfaces disponibles.

Pour obtenir des performances optimales, configurez votre serveur DNS afin qu’il gère les adresses de cluster accessibles par les clients comme indiqué dans le diagramme suivant.

Le diagramme présente un vserver de cluster à gauche, et des adresses IP au centre et à droite. Configurez chaque point d’accès client avec des enregistrements A et des pointeurs comme illustré ci-dessous.

![Diagramme d’un DNS en tourniquet (round-robin) pour le cluster Avere](media/avere-vfxt-rrdns-diagram.png)
<!--- separate text description file provided  [diagram text description](avere-vfxt-rrdns-alt-text.md) -->

Chaque adresse IP accessible par un client doit avoir un nom unique, qui est utilisé en interne par le cluster. (Dans ce diagramme, les adresses IP clientes sont nommées vs1-client-IP-* pour plus de clarté, mais dans un environnement de production, vous devrez sans doute utiliser des noms plus courts, comme client*.)

Les clients montent le cluster en utilisant le nom vserver comme argument serveur.

Modifiez le fichier ``named.conf`` de votre serveur DNS pour définir l’ordre cyclique des requêtes envoyées à vserver. Cette option garantit que toutes les valeurs disponibles sont utilisées de manière cyclique. Ajoutez une instruction similaire à celle-ci :

```
options {
    rrset-order {
        class IN A name "vserver1.example.com" order cyclic;
    };
};
```

Les commandes ``nsupdate`` suivantes sont un exemple de configuration correcte d’un système DNS :

```
update add vserver1.example.com. 86400 A 10.0.0.10
update add vserver1.example.com. 86400 A 10.0.0.11
update add vserver1.example.com. 86400 A 10.0.0.12
update add vs1-client-IP-10.example.com. 86400 A 10.0.0.10
update add vs1-client-IP-11.example.com. 86400 A 10.0.0.11
update add vs1-client-IP-12.example.com. 86400 A 10.0.0.12
update add 10.0.0.10.in-addr.arpa. 86400 PTR vs1-client-IP-10.example.com
update add 11.0.0.10.in-addr.arpa. 86400 PTR vs1-client-IP-11.example.com
update add 12.0.0.10.in-addr.arpa. 86400 PTR vs1-client-IP-12.example.com
```

## <a name="cluster-dns-settings"></a>Paramètres du DNS du cluster

Spécifiez le serveur DNS utilisé par le cluster vFXT dans la page de paramètres **Cluster** > **Administrative Network** (Cluster > Réseau d’administration). Cette page contient les paramètres suivants :

* Adresse du serveur DNS
* Nom du domaine DNS
* Domaines de recherche DNS

Pour plus d’informations sur l’utilisation de cette page, consultez [DNS Settings](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_admin_network.html#gui-dns>) dans le guide de configuration du cluster Avere.
