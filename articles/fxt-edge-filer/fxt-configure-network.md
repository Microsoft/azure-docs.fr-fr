---
title: 'Tutoriel : Configurer un réseau dans un cluster Azure FXT Edge Filer'
description: Découvrez comment personnaliser les paramètres réseau après la création du cluster Microsoft Azure FXT Edge Filer.
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.openlocfilehash: 4ce7ffc66e0b6164b2e4ca9725b3f26403292a4a
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220769"
---
# <a name="tutorial-configure-the-clusters-network-settings"></a>Tutoriel : Configurer les paramètres réseau du cluster

Avant d’utiliser un nouveau cluster Azure FXT Edge Filer, vous devez vérifier et personnaliser plusieurs paramètres réseau pour votre flux de travail.

Ce tutoriel décrit les paramètres réseau que vous devrez peut-être ajuster pour un nouveau cluster.

Vous apprendrez à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Quels paramètres réseau devront être mis à jour après la création d’un cluster.
> * Quels cas d’usage Azure FXT Edge Filer nécessitent un serveur Active Directory ou un serveur DNS.
> * Comment configurer le tourniquet DNS (RRDNS) pour équilibrer automatiquement la charge des requêtes des clients sur le cluster FXT.

La durée nécessaire pour effectuer ces étapes dépend du nombre de modifications de configuration nécessaires dans votre système :

* Si vous avez uniquement besoin de lire le tutoriel et de vérifier quelques paramètres, il vous faudra de 10 à 15 minutes.
* Si vous avez besoin de configurer le tourniquet DNS, cette tâche peut prendre une heure ou plus.

## <a name="adjust-network-settings"></a>Ajuster les paramètres réseau

La configuration d’un nouveau cluster Azure FXT Edge Filer implique plusieurs tâches liées au réseau. Vérifiez cette liste et choisissez celles qui s’appliquent à votre système.

Pour en savoir plus sur les paramètres réseau pour le cluster, consultez la section sur la [configuration des services réseau](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/network_overview.html) dans le Guide de configuration de cluster.

* Configurer le tourniquet DNS pour le réseau utilisé par le client (facultatif)

  Équilibrez la charge du trafic de cluster en configurant le système DNS comme décrit dans [Configurer le système DNS pour le cluster FXT Edge Filer](#configure-dns-for-load-balancing).

* Vérifier les paramètres NTP

* Configurer Active Directory et les téléchargements de noms d’utilisateurs/groupes (si nécessaire)

  Si vos hôtes réseau utilisent Active Directory ou un autre type de service d’annuaire externe, vous devez modifier la configuration des services d’annuaire du cluster pour configurer comment le cluster télécharge les informations sur les noms d’utilisateurs et les groupes. Pour plus d’informations, consultez **Cluster** > **Directory Services** dans le Guide de configuration de cluster.

  Un serveur AD est obligatoire si vous souhaitez prendre en charge SMB. Configurez AD avant de commencer à configurer SMB.

* Définir des réseaux locaux virtuels (facultatif)
  
  Configurez les réseaux locaux virtuels supplémentaires nécessaires avant de définir les serveurs virtuels et l’espace de noms global de votre cluster. Pour plus d’informations, consultez [Working with VLANs](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/network_overview.html#vlan-overview) dans le Guide de configuration de cluster.

* Configurer les serveurs proxy (si nécessaire)

  Si votre cluster utilise un serveur proxy pour atteindre des adresses externes, effectuez ces étapes pour le configurer :

  1. Définissez le serveur proxy dans la page de paramètres **Configuration du proxy**.
  1. Appliquez la configuration du serveur proxy avec la page **Cluster** > **General Setup** ou **Core Filer Details**.
  
  Pour plus d’informations, consultez [Using web proxies](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/proxy_overview.html) dans le Guide de configuration de cluster.

* Charger les [certificats de chiffrement](#encryption-certificates) pour le cluster (facultatif)

### <a name="encryption-certificates"></a>Certificats de chiffrement

Le cluster FXT Edge Filer utilise des certificats X.509 pour ces fonctions :

* Pour chiffrer le trafic d’administration de cluster

* Pour authentifier un client auprès des serveurs KMIP tiers

* Pour vérifier les certificats de serveur des fournisseurs de cloud

Si vous avez besoin de charger des certificats sur le cluster, utilisez la page de paramètres **Cluster** > **Certificates**. Pour plus d’informations, consultez la page [Cluster > Certificates](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_certificates.html) dans le Guide de configuration de cluster.

Pour chiffrer la communication de gestion de cluster, utilisez la page de paramètres **Cluster** > **General Setup** (Configuration générale) afin de sélectionner le certificat à utiliser pour le protocole TLS d’administration.

> [!Note]
> Les clés d’accès au service cloud sont stockés à l’aide de la page de configuration **Cloud Credentials**. La section [Ajouter un Core Filer](fxt-add-storage.md#add-a-core-filer) montre un exemple. Pour plus d’informations, consultez la section [Cloud Credentials](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cloud_credentials.html) du Guide de configuration de cluster.

## <a name="configure-dns-for-load-balancing"></a>Configurer le système DNS pour l’équilibrage de charge

Cette section explique les principes fondamentaux de la configuration d’un tourniquet DNS (RRDNS) pour répartir la charge client parmi toutes les adresses IP exposées aux clients dans votre cluster FXT Edge Filer.

### <a name="decide-whether-or-not-to-use-dns"></a>Décider s’il faut utiliser DNS ou non

L’équilibrage de charge est toujours recommandée, mais vous n’êtes pas obligé de toujours utiliser le système DNS. Par exemple, avec certains types de flux de travail client, il peut être plus judicieux d’utiliser un script pour affecter les adresses IP de cluster uniformément parmi les clients quand ils montent le cluster. Certaines méthodes sont décrites dans [Monter le cluster](fxt-mount-clients.md).

Pour déterminer si vous devez ou non utiliser un serveur DNS, prenez en compte les considérations suivantes :

* Si votre système est sollicité uniquement par des clients NFS, le système DNS n’est pas nécessaire. Il est possible de spécifier toutes les adresses réseau à l’aide d’adresses IP numériques.

* Si votre système prend en charge l’accès SMB (CIFS), un DNS est nécessaire, car vous devez alors spécifier un domaine DNS pour le serveur Active Directory.

* Un DNS est nécessaire pour utiliser l’authentification Kerberos.

### <a name="round-robin-dns-configuration-details"></a>Détails de configuration du tourniquet DNS

Quand les clients accèdent au cluster, RRDNS répartit automatiquement leurs requêtes entre toutes les interfaces disponibles.

Pour obtenir des performances optimales, configurez votre serveur DNS afin qu’il gère les adresses de cluster accessibles par les clients comme indiqué dans le diagramme suivant.

Le diagramme présente un vserver de cluster à gauche, et des adresses IP au centre et à droite. Configurez chaque point d’accès client avec des enregistrements A et des pointeurs comme illustré ci-dessous.

![Diagramme du tourniquet DNS de cluster - texte détaillé de remplacement du lien après l’image](media/fxt-cluster-config/fxt-rrdns-diagram.png) 
[description textuelle détaillée](https://azure.github.io/Avere/legacy/Azure-FXT-EdgeFilerDNSconfiguration-alt-text.html)

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

### <a name="enable-dns-in-the-cluster"></a>Activer le système DNS dans le cluster

Spécifiez le serveur DNS utilisé par le cluster dans la page de paramètres **Cluster** > **Administrative Network**. Cette page contient les paramètres suivants :

* Adresse du serveur DNS
* Nom du domaine DNS
* Domaines de recherche DNS

Pour plus d’informations, consultez [DNS Settings](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_admin_network.html#gui-dns>) dans le Guide de configuration de cluster.

## <a name="next-steps"></a>Étapes suivantes

Il s’agit de la dernière étape de configuration de base pour le cluster Azure FXT Edge Filer.

* Apprenez-en davantage sur les voyants et autres indicateurs du système dans [Superviser l’état du matériel](fxt-monitor.md).
* Apprenez-en davantage sur la façon dont les clients doivent monter le cluster FXT Edge Filer dans [Monter le cluster](fxt-mount-clients.md).
* Pour plus d’informations sur l’exploitation et la gestion d’un cluster FXT Edge Filer, consultez le [Guide de configuration de cluster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/ops_conf_index.html).
