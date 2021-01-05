---
title: 'Tutoriel : Créer le cluster du cache Azure FXT Edge Filer'
description: Découvrez comment créer un cluster de cache de stockage hybride avec Azure FXT Edge Filer et comment vous connecter au panneau de configuration du cluster pour configurer les paramètres du cluster.
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 07/01/2019
ms.openlocfilehash: d019f5df4bba6d223076c8ce35151510afedf2e9
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220803"
---
# <a name="tutorial-create-the-azure-fxt-edge-filer-cluster"></a>Tutoriel : Créer le cluster Azure FXT Edge Filer

Après avoir installé et initialisé les nœuds matériels Azure FXT Edge Filer pour votre cache, utilisez le logiciel de cluster FXT pour créer le cluster de cache.

Ce tutoriel vous guide tout au long des étapes de configuration de vos nœuds matériels en tant que cluster.

Ce tutoriel vous apprendra à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Rassembler les informations nécessaires avant de commencer à créer le cluster
> * Faire la différence entre le réseau de gestion du cluster, le réseau en cluster et le réseau exposé aux clients
> * Se connecter à un nœud de cluster
> * Créer un cluster initial à l’aide d’un seul nœud Azure FXT Edge Filer
> * Se connecter au panneau de configuration du cluster pour en configurer les paramètres

Cette procédure prend entre 15 et 45 minutes, en fonction des recherches que vous avez besoin de faire pour identifier les IP adresses et les ressources réseau.

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, effectuez ces prérequis :

* Installez vos systèmes matériels Azure FXT Edge Filer dans votre centre de données.

  Il n’avez besoin que d’un seul nœud pour créer le cluster, mais vous devez [ajouter au moins deux nœuds supplémentaires](fxt-add-nodes.md) pour pouvoir le configurer et le rendre prêt à l’emploi.

* Branchez les câbles d’alimentation et réseau appropriés sur le système.  
* Mettez sous tension au moins un nœud Azure FXT Edge Filer et [définissez son mot de passe racine](fxt-node-password.md).

## <a name="gather-information-for-the-cluster"></a>Collecter des informations pour le cluster

Vous avez besoin des informations suivantes pour créer le cluster Azure FXT Edge Filer :

* Nom du cluster

* Mot de passe d’administration à définir pour le cluster

* Adresses IP :

  * Une adresse IP unique pour la gestion du cluster, ainsi que le masque de réseau et le routeur à utiliser pour le réseau de gestion.
  * La première et la dernière adresse IP dans une plage contigüe d’adresses IP pour la communication du cluster (nœud à nœud). Consultez [Distribution des adresses IP](#ip-address-distribution) ci-dessous pour plus d’informations.
  * (Les adresses IP exposées aux clients sont définies après la création du cluster.)

* Informations sur l’infrastructure réseau :

  * Adresse IP d’un serveur DNS pour le cluster.
  * Nom du domaine DNS pour le cluster.
  * Nom ou adresse IP des serveurs NTP du cluster (soit un seul serveur, soit trois serveurs ou plus).
  * Activation ou non de l’agrégation des liens IEEE 802.1AX-2008 sur les interfaces du cluster.
  * Si vous activez l’agrégation des liens, utilisation ou non de l’agrégation dynamique IEEE 802.3ad (LACP).

Vous pouvez configurer ces éléments d’infrastructure réseau après avoir créé le cluster, mais il est préférable de le faire au moment de la création.

### <a name="ip-address-distribution"></a>Distribution des adresses IP

Le cluster de cache de stockage hybride Azure FXT Edge Filer utilise des adresses IP dans trois catégories :

* Adresse IP de gestion : une adresse IP unique pour la gestion du cluster

  Cette adresse sert de point d’entrée pour accéder aux utilitaires de configuration du cluster (panneau de configuration web ou API XML-RPC). Elle est automatiquement attribuée au nœud principal du cluster et se modifie automatiquement si le nœud principal change.

  Il est possible d’utiliser d’autres adresses IP pour accéder au panneau de configuration, mais l’adresse IP de gestion est conçue pour permettre un accès même quand des nœuds individuels basculent.

* Réseau en cluster : une plage d’adresses IP pour la communication du cluster

  Le réseau en cluster est utilisé pour la communication entre les nœuds du cluster et pour récupérer des fichiers dans le stockage back-end (système de stockage principal).

  **Bonne pratique :** allouez une seule adresse IP par port physique utilisé pour la communication du cluster sur chaque nœud Azure FXT Edge Filer. Le cluster attribue automatiquement les adresses incluses dans la plage spécifiée à des nœuds individuels.

* Réseau exposé aux clients : plage d’adresses IP que les clients utilisent pour demander et écrire des fichiers

  Les adresses réseau de client sont utilisées par les clients pour accéder aux données du système de stockage principal par l’intermédiaire du cluster. Par exemple, un client NFS peut monter une de ces adresses.

  **Bonne pratique :** allouez une seule adresse IP par port physique utilisé pour la communication client sur chaque nœud FXT Series.

  Le cluster distribue les adresses IP exposées aux clients parmi ses nœuds constitutifs aussi équitablement que possible.

  Par souci de simplicité, de nombreux administrateurs configurent un nom DNS unique avec une configuration DNS par tourniquet (round-robin) (RRDNS) pour faciliter la distribution des demandes des clients dans la plage d’adresses. Cette configuration permet également à tous les clients d’utiliser la même commande de montage pour accéder au cluster. Consultez [Configurer DNS](fxt-configure-network.md#configure-dns-for-load-balancing) pour plus d’informations.

L’adresse IP de gestion et une plage d’adresses de réseau en cluster doivent être spécifiées pour créer un cluster. Les adresses exposées aux clients sont spécifiées après la création du cluster.

## <a name="connect-to-the-first-node"></a>Se connecter au premier nœud

Vous pouvez vous connecter à l’un des nœuds FXT installés et utiliser son système d’exploitation pour configurer le cluster.

Si vous ne l’avez pas déjà fait, mettez sous tension au moins l’un des nœuds FXT de votre cluster, puis vérifiez qu’il a une connexion réseau et une adresse IP. Vous devez définir un nouveau mot de passe racine pour activer le nœud, alors suivez les étapes décrites dans [Définir des mots de passe pour le matériel](fxt-node-password.md) si vous ne l’avez pas déjà fait.

Pour vérifier la connexion réseau, assurez-vous que les LED de liaison réseau du nœud sont allumées (et, si nécessaire, les indicateurs situés sur le commutateur réseau auquel il est raccordé). Les LED sont décrites dans [Superviser l’état du matériel Azure FXT Edge Filer](fxt-monitor.md).

Quand le nœud démarre, il demande une adresse IP. S’il est connecté à un serveur DHCP, il accepte l’adresse IP fournie par ce dernier. (Cette adresse IP est temporaire. Elle va changer quand vous allez créer le cluster.)

S’il n’est pas connecté à un serveur DHCP ou ne reçoit pas de réponse, le nœud va utiliser le logiciel Bonjour pour définir une adresse IP auto-affectée sous la forme 169.254.\*.\*. Toutefois, il est préférable de définir une adresse IP statique temporaire sur une des cartes réseau du nœud avant de l’utiliser pour créer un cluster. Des instructions sont données dans ce document de référence. Contactez le support technique de Microsoft pour obtenir des informations mises à jour : [Appendix A: Setting a Static IP Address on an FXT Node](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/static_ip.html) (Annexe A : Définition d’une adresse IP statique sur un nœud FXT).

### <a name="find-the-ip-address"></a>Trouver l’adresse IP

Connectez-vous au nœud Azure FXT Edge Filer pour trouver son adresse IP. Vous pouvez utiliser un câble série, une connexion directe aux ports USB et VGA ou vous connecter par le biais d’un commutateur KVM. (Pour plus d’informations sur la connexion aux ports, consultez [Définir des mots de passe initiaux](fxt-node-password.md).)

Une fois raccordé, connectez-vous avec le nom d’utilisateur `root` et le mot de passe que vous avez définis quand vous avez démarré le nœud pour la première fois.  

Une fois connecté, vous devez déterminer l’adresse IP du nœud.

Utilisez la commande `ifconfig` pour voir les adresses attribuées à ce système.

Par exemple, la commande `ifconfig | grep -B5 inet` recherche les ports dotés d’adresses Internet et donne cinq lignes de contexte pour indiquer l’identificateur du port.

Notez l’une des adresses IP indiquées dans le rapport ifconfig. Les adresses listées avec des noms de port comme e0a ou e0b sont de bonnes options. N’utilisez pas les adresses IP listées avec des noms e7*, car ces noms sont utilisés uniquement pour les ports de service iDRAC/IPMI.

## <a name="load-the-cluster-configuration-wizard"></a>Charger l’Assistant Configuration de cluster

Utilisez l’outil de configuration de cluster basé sur un navigateur pour créer le cluster.

Entrez l’adresse IP du nœud dans un navigateur web. Si le navigateur affiche un message stipulant que le site n’est pas fiable, accédez-y quand même. (Les nœuds Azure FXT Edge Filer individuels n’ont pas de certificats de sécurité fournis par une autorité de certification.)

![Page de connexion du panneau de configuration dans la fenêtre de navigateur](media/fxt-cluster-create/unconfigured-node-gui.png)

Laissez les champs **Nom d’utilisateur** et **Mot de passe** vides. Cliquez sur **Login** (Connexion) pour charger la page de création du cluster.

![Écran de configuration initiale d’un nœud non configuré dans le panneau de configuration de l’interface graphique utilisateur basée sur un navigateur. Celui-ci présente des options pour mettre à jour le logiciel, configurer un cluster manuellement ou configurer un cluster à partir d’un fichier d’installation.](media/fxt-cluster-create/setup-first-screen.png)

## <a name="create-the-cluster"></a>Créer le cluster

L’outil de configuration de cluster vous guide tout au long d’un ensemble d’écrans pour créer le cluster Azure FXT Edge Filer. Vérifiez que vous disposez des [informations nécessaires](#gather-information-for-the-cluster) avant de commencer.

### <a name="creation-options"></a>Options de création

Le premier écran propose trois options. Utilisez l’option de configuration manuelle, sauf si l’équipe de support vous a donné des instructions spéciales.

Cliquez sur **I will configure the cluster manually** (Je vais configurer le cluster manuellement) pour charger l’écran des options de configuration du nouveau cluster.

Les autres options sont rarement utilisées :

* L’option qui permet de mettre à jour l’image système vous invite à installer un nouveau système d’exploitation avant de créer le cluster. (La version du logiciel actuellement installé est listée en haut de l’écran.) Vous devez fournir le fichier du package logiciel (soit une URL et un nom d’utilisateur/mot de passe, soit un fichier chargé à partir de votre ordinateur).

* L’option qui utilise un fichier de configuration de cluster est parfois utilisée par le support technique de Microsoft.

## <a name="cluster-options"></a>Options du cluster

L’écran suivant vous invite à configurer des options pour le nouveau cluster.

La page est divisée en deux sections principales, **Basic Configuration** (Configuration de base) et **Networking Configuration** (Configuration de mise en réseau). La section de configuration de mise en réseau a également des sous-sections : une pour le réseau de **gestion** et l’autre pour le réseau en **cluster**.

### <a name="basic-configuration"></a>Configuration de base

Dans la section supérieure, renseignez les informations de base du nouveau cluster.

![Section de configuration de base dans la page de l’interface graphique utilisateur du navigateur. Cette section contient trois champs (un pour le nom du cluster, un pour le mot de passe administrateur, un pour la confirmation du mot de passe).](media/fxt-cluster-create/basic-configuration.png)

* **Cluster Name** (Nom du cluster) : saisissez un nom unique pour le cluster.

  Le nom du cluster doit respecter ces critères :
  
  * Longueur comprise entre 1 et 16 caractères.
  * Peut inclure des lettres, des chiffres, des tirets (-) et des traits de soulignement (_).
  * Ne doit pas inclure d’autres signes de ponctuation ou caractères spéciaux.
  
  Vous pouvez modifier ce nom par la suite dans la page de configuration **Cluster** > **Configuration générale**. (Pour plus d’informations sur les paramètres du cluster, lisez le [guide de configuration d’un cluster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/ops_conf_index.html), qui ne fait pas partie de cette documentation.)

  > [!NOTE]
  > Le nom de votre cluster est utilisé pour identifier les informations système envoyées au support technique à des fins de supervision ou de résolution des problèmes, il est donc judicieux d’inclure le nom de votre entreprise.

* **Admin password** (Mot de passe administrateur) : définissez le mot de passe de l’utilisateur administrateur par défaut, `admin`.
  
  Vous devez configurer des comptes d’utilisateur individuels pour chaque personne qui administre le cluster, mais vous ne pouvez pas supprimer l’utilisateur `admin`. Connectez-vous en tant que `admin` si vous avez besoin de créer d’autres utilisateurs.

  Vous pouvez modifier le mot de passe pour `admin` dans la page de paramètres **Administration** > **Utilisateurs** du panneau de configuration du cluster. Pour plus d’informations, consultez la documentation sur les **utilisateurs** dans le [guide de configuration d’un cluster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_users.html).

<!-- to do: update "legacy" URLs when docs are ported to Microsoft site -->

### <a name="network-configuration"></a>Configuration réseau

La section **Networking** (Mise en réseau) vous invite à spécifier l’infrastructure réseau que le cluster va utiliser.

Il existe deux réseaux distincts à configurer :

* Le *réseau de gestion* fournit un accès administrateur au cluster à des fins de configuration et de supervision. L’adresse IP spécifiée ici est utilisée lors de la connexion au panneau de configuration ou pour l’accès SSH.

  La plupart des clusters utilisent une seule adresse IP de gestion, mais si vous voulez ajouter des interfaces, vous pouvez le faire après avoir créé le cluster.

* Le *réseau en cluster* est utilisé pour la communication entre les nœuds de cluster et la communication entre les nœuds de cluster et le stockage back-end (système de stockage principal).

Le réseau exposé aux clients est configuré plus tard, une fois que le cluster est créé.

Cette section inclut également la configuration des serveurs DNS et NTP utilisés par les deux réseaux.

### <a name="configure-the-management-network"></a>Configurer le réseau de gestion

Les paramètres de la section **Management** (Gestion) concernent le réseau qui offre un accès administrateur au cluster.

![Section « Management » (Gestion), avec les champs des 5 options et une case à cocher pour le réseau de gestion de 1 Go.](media/fxt-cluster-create/management-network-filled.png)

* **Management IP** (Adresse IP de gestion) : spécifiez l’adresse IP que vous allez utiliser pour accéder au panneau de configuration du cluster. Cette adresse est revendiquée par le nœud principal du cluster, mais elle est automatiquement transférée à un nœud sain si le nœud principal d’origine devient indisponible.

  La plupart des clusters utilisent une seule adresse IP de gestion. Si vous en voulez plusieurs, vous pouvez les ajouter après avoir créé le cluster à l’aide de la page de paramètres **Cluster** > **Réseau administratif**. Pour plus d’informations, consultez le [guide de configuration d’un cluster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_admin_network.html).

* **Netmask** (Masque de réseau) : spécifiez le masque de réseau du réseau de gestion.

* **Router** (Routeur) : entrez l’adresse de passerelle par défaut utilisée par le réseau de gestion.

* **VLAN tag (optional)** (Balise VLAN (facultative)) : si votre cluster utilise des balises VLAN, spécifiez celle du réseau de gestion.

  D’autres paramètres VLAN sont configurés dans la page **Cluster** > **VLAN**. Consultez les sections sur l’[utilisation des VLAN](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/network_overview.html#vlan-overview) et [Cluster > VLAN](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_vlan.html) dans le guide de configuration d’un cluster pour en savoir plus.

* **MTU** : si nécessaire, ajustez l’unité de transmission maximale (MTU) du réseau de gestion de votre cluster.

* **Use 1Gb mgmt network** (Utiliser un réseau de gestion de 1 Go) : cochez cette case si vous voulez affecter les deux ports réseau 1GbE sur vos nœuds FXT au réseau de gestion uniquement. (Vous devez avoir des ports 25GbE/10GbE disponibles pour tout autre trafic.) Si vous ne cochez pas cette case, le réseau de gestion utilise le port disponible dont la vitesse est la plus élevée.

### <a name="configure-the-cluster-network"></a>Configurer le réseau en cluster

Les paramètres du réseau en cluster s’appliquent au trafic parmi les nœuds de cluster et au trafic entre les nœuds de cluster et le système de stockage principal.

![Section « Cluster », avec des champs pour entrer six valeurs.](media/fxt-cluster-create/cluster-network-filled.png)

* **First IP** (Première adresse IP) et **Last IP** (Dernière adresse IP) : entrez les adresses IP qui définissent la plage à utiliser pour les communications internes du cluster. Les adresses IP utilisées ici doivent être contigües et ne pas être attribuées par DHCP.

  Vous pouvez ajouter d’autres adresses IP après la création du cluster. Utilisez la page de paramètres **Cluster** > **Réseaux en cluster** ([documentation du guide de configuration d’un cluster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cluster_networks.html#gui-cluster-networks)).

  La valeur indiquée dans **Number of IPs in Range** (Nombre d’adresses IP dans la plage) est automatiquement calculée et affichée.

* **Non-mgmt netmask (optional)** (Masque de réseau hors gestion (facultatif)) : spécifiez le masque de réseau du réseau en cluster.

  Le système suggère automatiquement la valeur du masque de réseau que vous avez entrée pour le réseau de gestion ; modifiez-la si nécessaire.

* **Cluster router (optional)** (Routeur de cluster (facultatif)) : spécifiez l’adresse de passerelle par défaut utilisée par le réseau en cluster.

  Le système suggère automatiquement la même adresse de passerelle que celle que vous avez fournie pour le réseau de gestion.

* **Cluster VLAN tag (optional)** (Balise VLAN du cluster (facultative)) : si votre cluster utilise des balises VLAN, spécifiez celle du réseau en cluster.

* **Non-mgmt MTU (optional)** (MTU hors gestion (facultatif)) : si nécessaire, ajustez l’unité de transmission maximale (MTU) de votre réseau en cluster.

### <a name="configure-cluster-dns-and-ntp"></a>Configurer les serveurs DNS et NTP du cluster

Sous la section **Cluster** se trouvent des champs permettant de spécifier les serveurs DNS et NTP et d’activer l’agrégation des liens. Ces paramètres s’appliquent à tous les réseaux que le cluster utilise.

![Section destinée à la configuration DNS/NTP, avec trois champs pour les serveurs DNS, des champs pour le domaine DNS et la recherche DNS, trois champs pour les serveurs NTP et un menu déroulant pour les options d’agrégation des liens.](media/fxt-cluster-create/dns-ntp-filled.png)

* **DNS server(s)** (Serveur(s) DNS) : entrez l’adresse IP d’un ou plusieurs serveurs DNS (Domain Name System).

  Un serveur DNS est recommandé pour tous les clusters et obligatoire que si vous voulez utiliser SMB, AD ou Kerberos.
  
  Pour obtenir des performances optimales, configurez le serveur DNS du cluster à des fins d’équilibrage de charge par tourniquet (round-robin) comme décrit dans [Configurer un serveur DNS pour le cluster Azure FXT Edge Filer](fxt-configure-network.md#configure-dns-for-load-balancing).

* **DNS domain** (Domaine DNS) : entrez le nom de domaine du réseau que le cluster va utiliser.

* **DNS search** (Recherche DNS) : entrez éventuellement d’autres noms de domaine dans lesquels le système doit rechercher pour résoudre les requêtes DNS. Vous pouvez ajouter jusqu’à six noms de domaine, séparés par des espaces.

* **NTP server(s)** (Serveur(s) NTP) : spécifiez un ou trois serveurs NTP (Network Time Protocol) dans les champs fournis. Vous pouvez utiliser des noms d’hôte ou des adresses IP.

* **Link aggregation** (Agrégation des liens) : l’agrégation des liens vous permet de personnaliser la façon dont les ports Ethernet situés sur les nœuds FXT du cluster gèrent divers types de trafic. Pour en savoir plus, consultez la section sur l’[agrégation des liens](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_cluster_general_setup.html#link-aggregation) dans le guide de configuration d’un cluster.

### <a name="click-the-create-button"></a>Cliquer sur le bouton Créer

Après avoir fourni tous les paramètres nécessaires dans le formulaire, cliquez sur le bouton **Create Cluster**  (Créer le cluster).

![Bas du formulaire rempli avec le curseur au-dessus du bouton Créer en bas à droite.](media/fxt-cluster-create/create-cluster.png)

Le système affiche un message pendant la création du cluster.

![Message sur l’état de configuration du cluster dans le navigateur : « The FXT node is now creating the cluster. This will take several minutes. When the cluster is created, visit this link to complete the configuration. » (Le nœud FXT est maintenant en train de créer le cluster. Ceci peut prendre plusieurs minutes. Une fois que le cluster est créé, visitez ce lien pour terminer la configuration.) Un lien hypertexte se trouve sur « visit this link ».](media/fxt-cluster-create/creating-message.png)

Au bout d’un moment, vous pouvez cliquer sur le lien dans le message pour accéder au panneau de configuration du cluster. (Ce lien vous fait accéder à l’adresse IP que vous avez spécifiée dans **Management IP** (Adresse IP de gestion).) Un délai compris entre 15 secondes et une minute est nécessaire pour que le lien devienne actif une fois que vous avez cliqué sur le bouton Créer. Si l’interface web ne se charge pas, attendez quelques secondes de plus, puis recliquez sur le lien.

La création du cluster prend une minute environ, mais vous pouvez vous connecter au panneau de configuration pendant le processus. Il est normal que la page de tableau de bord du panneau de configuration affiche des avertissements avant la fin du processus de création du cluster.

## <a name="open-the-settings-pages"></a>Ouvrir les pages de paramètres

Après avoir créé le cluster, vous devez personnaliser sa configuration pour votre réseau et votre workflow.

Utilisez l’interface web du panneau de configuration pour configurer votre nouveau cluster. Suivez le lien figurant sur l’écran de l’état de la création du cluster ou accédez à l’adresse IP de gestion que vous avez définie sur le cluster.

Connectez-vous à l’interface web avec le nom d’utilisateur `admin` et le mot de passe que vous avez défini lors de la création du cluster.

![Champs de connexion au panneau de configuration dans le navigateur web](media/fxt-cluster-create/admin-login.png)

Le panneau de configuration s’ouvre et présente la page **Dashboard** (Tableau de bord). Une fois que la création du cluster est terminée, tous les messages d’avertissement doivent disparaître.

Cliquez sur l’onglet **Settings** (Paramètres) pour configurer le cluster.

Sous l’onglet **Settings** (Paramètres), la barre latérale de gauche présente un menu de pages de configuration. Les pages sont organisées par catégorie. Cliquez sur le contrôle + ou - situé en regard du nom de la catégorie pour développer ou masquer les pages individuelles.

![Onglet Settings (Paramètres) du panneau de configuration (dans le navigateur) avec la page Cluster > General Setup (Configuration générale) chargée.](media/fxt-cluster-create/settings-tab-populated.png)

## <a name="cluster-setup-steps"></a>Étapes de configuration du cluster

À ce stade du processus, votre cluster existe, mais il ne comporte qu’un seul nœud, aucune adresse IP exposée aux clients et aucun stockage back-end. Des étapes de configuration supplémentaires sont nécessaires pour passer d’un cluster nouvellement créé à un système de cache prêt à gérer votre workflow.

### <a name="required-configuration"></a>Configuration nécessaire

Ces étapes sont nécessaires pour la plupart des clusters voire tous les clusters.

* Ajouter des nœuds au cluster

  La norme s’élève à trois nœuds, mais de nombreux clusters de production en ont plus, jusqu’à un maximum de 24 nœuds.

  Consultez [Ajouter des nœuds de cluster](fxt-add-nodes.md) pour apprendre à ajouter d’autres unités Azure FXT Edge Filer à votre cluster et pour activer la haute disponibilité.

* Spécifier un stockage back-end

  Ajoutez des définitions de *système de stockage principal* pour chaque système de stockage back-end que le cluster va utiliser. Consultez [Ajouter du stockage back-end et configurer un espace de noms virtuel](fxt-add-storage.md#about-back-end-storage) pour en savoir plus.

* Configurer l’accès client et l’espace de noms virtuel

  Créez au moins un serveur virtuel et attribuez-lui une plage d’adresses IP que les ordinateurs clients vont utiliser. Vous devez également configurer l’espace de noms du cluster (parfois appelé GNS ou espace de noms global), une fonctionnalité du système de fichiers virtuel qui vous permet de mapper les exportations de stockage back-end sur des chemins virtuels. L’espace de noms du cluster donne aux clients une structure de système de fichiers cohérente et accessible même si vous changez de support de stockage back-end. L’espace de noms peut également fournir une hiérarchie de stockage virtuel conviviale pour les conteneurs d’objets blob Azure ou un autre stockage d’objets cloud pris en charge.

  Consultez [Configurer l’espace de noms](fxt-add-storage.md#configure-the-namespace) pour plus d’informations. Cette étape inclut :
  * Création de serveurs virtuels
  * Configuration de jonctions entre la vue du réseau client et le stockage back-end
  * Définition des adresses IP servies par chaque serveur virtuel

  > [!Note]
  > Une planification approfondie est recommandée avant de commencer à configurer l’espace de noms global du cluster. Consultez les sections [Utilisation d’un espace de noms global](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gns_overview.html) et [Création et utilisation de serveurs virtuels](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/settings_overview.html#creating-and-working-with-vservers) dans le guide de configuration d’un cluster pour obtenir de l’aide.

* [Ajuster les paramètres réseau](fxt-configure-network.md)

  Il existe plusieurs paramètres réseau à vérifier ou personnaliser pour un nouveau cluster. Consultez [Ajuster les paramètres réseau](fxt-configure-network.md) pour plus d’informations sur ces éléments :

  * Vérification de la configuration DNS et NTP
  * Configuration des services d’annuaire, si nécessaire
  * Configuration de VLAN
  * Configuration de serveurs proxy
  * Ajout d’adresses IP au réseau en cluster
  * Stockage des certificats de chiffrement

* [Configurer la supervision du support technique](#enable-support)

  Vous devez accepter la politique de confidentialité de l’outil de configuration et vous devez configurer vos paramètres de chargement en même temps.

  Le cluster peut télécharger automatiquement les données de résolution des problèmes sur votre cluster, notamment les statistiques et les fichiers de débogage. Ces chargements permettent au support technique de Microsoft de fournir le meilleur service possible. Vous pouvez personnaliser ce qui est supervisé et éventuellement activer le support technique proactif et le service de résolution des problèmes à distance.  

### <a name="optional-configuration"></a>Configuration facultative

Ces étapes ne sont pas nécessaires pour tous les clusters. Elle le sont pour certains types de workflow ou pour certains styles de gestion du cluster.

* Personnaliser les paramètres de nœud

  Vous pouvez définir des noms de nœud et configurer des ports IPMI de nœud à l’échelle d’un cluster ou individuellement. Si vous configurez ces paramètres avant d’ajouter des nœuds au cluster, les nouveaux nœuds peuvent choisir les paramètres automatiquement quand ils se joignent. Les options sont décrites dans le document sur la création d’un cluster intitulé [Customizing Node Settings](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/config_node.html) (Personnalisation des paramètres de nœud).

  > [!TIP]
  > La documentation sur ce produit n’est pas encore disponible sur le site de documentation Microsoft Azure. Les liens vers le [guide de configuration d’un cluster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/ops_conf_index.html) et vers l’ancienne version du [guide de création d’un cluster](https://azure.github.io/Avere/legacy/create_cluster/4_8/html/create_index.html) vous font accéder à un site web distinct hébergé par GitHub.

* Configurer SMB

  Si vous voulez autoriser l’accès SMB à votre cluster, ainsi que NFS, vous devez configurer SMB et l’activer. SMB (parfois appelé CIFS) est généralement utilisé pour prendre en charge des clients Microsoft Windows.

  La planification et la configuration de SMB impliquent plus que de cliquer sur quelques boutons dans le panneau de configuration. Selon les exigences de votre système, SMB peut influencer la façon dont vous définissez le système de stockage principal, le nombre de serveurs virtuels que vous créez, la manière de configurer vos jonctions et votre espace de noms, les autorisations d’accès et d’autres paramètres.

  Pour plus d’informations, lisez la section sur la [configuration de l’accès SMB](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/smb_overview.html) dans le guide de configuration d’un cluster.

* Installer des licences supplémentaires

  Si vous voulez utiliser un stockage cloud autre que Stockage Blob Azure, vous devez installer une licence supplémentaire. Contactez votre représentant Microsoft pour plus d’informations sur l’achat d’une licence FlashCloud<sup>TM</sup>. D’autres détails sont expliqués dans [Ajouter du stockage back-end et configurer un espace de noms virtuel](fxt-add-storage.md#about-back-end-storage).

### <a name="enable-support"></a>Activer le support technique

Le cluster Azure FXT Edge Filer peut charger automatiquement les données de support sur votre cluster. Ces chargements permettent aux équipes de délivrer un meilleur service client.

Suivez ces étapes pour configurer les chargements de support.

1. Accédez à la page de paramètres **Cluster** > **Support**. Acceptez la politique de confidentialité.

   ![Capture d’écran montrant le panneau de configuration et une fenêtre contextuelle avec le bouton de confirmation pour accepter la politique de confidentialité](media/fxt-cluster-create/fxt-privacy-policy.png)

1. Cliquez sur le triangle à gauche de **Customer Info** (Informations client) pour développer la section.
1. Cliquez sur le bouton **Revalidate upload information** (Revalider les informations de chargement).
1. Nommez le support du cluster dans **Unique Cluster Name** (Nom de cluster unique) en veillant à choisir un nom qui identifie de manière unique votre cluster auprès de l’équipe de support.
1. Cochez les cases **Statistics Monitoring** (Analyse des statistiques), **General Information Upload** (Chargement des informations générales) et **Crash Information Upload** (Chargement des informations d’incident).
1. Cliquez sur **Envoyer**.  

   ![Capture d’écran montrant la section des informations du client complétée dans la page des paramètres de support](media/fxt-cluster-create/fxt-support-info.png)

1. Cliquez sur le triangle à gauche de **Secure Proactive Support (SPS)** pour développer la section.
1. Cochez la case **Enable SPS Link** (Activer le lien SPS).
1. Cliquez sur **Envoyer**.

   ![Capture d’écran montrant la section Secure Proactive Support complétée dans la page des paramètres de support](media/fxt-cluster-create/fxt-support-sps.png)

## <a name="next-steps"></a>Étapes suivantes

Après avoir créé le cluster de base et accepté la politique de confidentialité, ajoutez le reste des nœuds de cluster.

> [!div class="nextstepaction"]
> [Ajouter des nœuds de cluster](fxt-add-nodes.md)
