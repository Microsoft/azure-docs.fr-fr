---
title: 'Tutoriel : Ajouter un stockage à un cluster Azure FXT Edge Filer'
description: Découvrez comment configurer un stockage back-end et le pseudo-espace de nom exposé aux clients pour Azure FXT Edge Filer
author: ekpgh
ms.author: rohogue
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.openlocfilehash: 7d10c6c1ce440b2ffe964dc78379ef3ab108e78e
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92217522"
---
# <a name="tutorial-add-back-end-storage-and-configure-the-virtual-namespace"></a>Tutoriel : Ajouter un stockage back-end et configurer l’espace de noms virtuel

Ce tutoriel explique comment ajouter un stockage back-end pour le cache et comment configurer le système de fichiers virtuel exposé aux clients.

Le cluster se connecte à des systèmes de stockage back-end pour accéder aux données demandées par les clients et pour y stocker les changements sur une plus longue durée que dans le cache.

L’espace de noms est le pseudo-système de fichiers exposé aux clients qui vous permet de permuter le stockage back-end sans changer les workflows côté client.

Ce didacticiel vous apprendra à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Ajouter un stockage back-end au cluster Azure FXT Edge Filer
> * Définir le chemin exposé aux clients pour le stockage

## <a name="about-back-end-storage"></a>Présentation du stockage back-end

Le cluster Azure FXT Edge Filer utilise une définition de *core filer* pour connecter un système de stockage back-end au cluster FXT.

Azure FXT Edge Filer est compatible avec plusieurs systèmes matériels NAS parmi les plus courants, et peut utiliser des conteneurs vides du stockage Blob Azure ou d’autres stockages cloud.

Les conteneurs de stockage cloud doivent être vides au moment de leur ajout afin que le système d’exploitation FXT puisse gérer intégralement toutes les données sur le volume de stockage cloud. Vous pouvez déplacer vos données existantes vers le conteneur cloud après avoir ajouté le conteneur au cluster comme core filer.

Utilisez le Panneau de configuration pour ajouter un core filer à votre système.

> [!NOTE]
>
> Si vous souhaitez utiliser le stockage Amazon AWS ou Google Cloud, vous devez installer une licence pour la fonctionnalité FlashCloud<sup>TM</sup>. Obtenez une clé de licence auprès de votre représentant Microsoft, puis suivez les instructions fournies dans la section [Adding or removing feature licenses](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/install_licenses.html#install-licenses) (Ajout ou suppression de licences de fonctionnalités) du Guide de configuration du cluster.
>
> La prise en charge du stockage Blob Azure est incluse dans la licence logicielle d’Azure FXT Edge Filer.

Pour plus d’informations sur l’ajout de core filers, lisez les sections suivantes dans le Guide de configuration du cluster :

* Pour plus d’informations sur le choix d’un core filer et les étapes préalables à son ajout, consultez [Working With Core Filers](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/core_filer_overview.html#core-filer-overview) (Utilisation des core filers).
* Pour connaître le détail des prérequis et obtenir des instructions pas à pas, lisez ces articles :

  * [Adding a New NAS Core Filer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_nas.html#create-core-filer-nas) (Ajout d’un nouveau core filer NAS)
  * [Adding a New Cloud Core Filer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_cloud.html#create-core-filer-cloud) (Ajout d’un nouveau core filer cloud)

Après avoir ajouté un core filer, vous pouvez mettre à jour ses paramètres dans la page de paramètres Core Filer Details (Détails du core filer).

## <a name="add-a-core-filer"></a>Ajouter un core filer

Définissez un core filer en cliquant sur le bouton **Create** (Créer) dans la page de paramètres **Core Filer** > **Manage Core Filers** (Core filer > Gérer les core filers).

![Clic sur le bouton Create au-dessus de la liste des core filers dans la page Manage Core Filers](media/fxt-cluster-config/create-core-filer-button.png)

L’Assistant **Add New Core Filer** (Ajouter un nouveau core filer) vous explique pas à pas comment créer un core filer connecté à votre stockage back-end. Le Guide de configuration du cluster détaille toutes des étapes du processus, qui est différent pour le stockage NFS/NAS et le stockage cloud (voir les liens ci-dessus).

Les tâches subordonnées sont les suivantes :

* Spécifiez le type de core filer (NAS ou cloud)

  ![Première page de l’Assistant d’ajout d’un nouveau core filer pour un NAS matériel. L’option Cloud comme type de core filer est désactivée et affiche un message d’erreur en raison de l’absence de licence.](media/fxt-cluster-config/new-nas-1.png)

* Définissez le nom du core filer. Choisissez un nom qui permette aux administrateurs du cluster d’identifier facilement quel système de stockage il représente.

* Pour les core filers NAS, indiquez le nom de domaine complet (FQDN) ou l’adresse IP. Le nom de domaine complet est recommandé pour tous les core filers et obligatoire pour un accès SMB.

* Sélectionnez une stratégie de cache. La deuxième page de l’Assistant liste toutes les stratégies de cache disponibles pour le nouveau core filer. Pour plus d’informations, consultez la [section sur les stratégies de cache dans le Guide de configuration du cluster](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_manage_cache_policies.html).

  ![Deuxième page de l’Assistant d’ajout d’un nouveau core filer pour un NAS matériel. Le menu déroulant des stratégies de cache est développé ; il affiche plusieurs options désactivées et trois options de stratégie de cache valides (contournement, mise en cache en lecture et mise en cache en lecture/écriture).](media/fxt-cluster-config/new-nas-choose-cache-policy.png)

* Pour le stockage cloud, vous devez spécifier le service cloud et les informations d’identification pour y accéder, entre autres paramètres. Pour plus d’informations, consultez [Cloud service and protocol](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_cloud.html#cloud-service-and-protocol) (Service cloud et protocole) dans le Guide de configuration du cluster.

  ![Informations sur le core filer de type cloud dans l’Assistant New Core Filer](media/fxt-cluster-config/new-core-filer-cloud3.png)
  
  Si vous avez déjà ajouté les informations d’identification de l’accès cloud pour ce cluster, elles apparaissent dans la liste. Mettez à jour et ajoutez les informations d’identification dans la page de paramètres **Cluster** > **Cloud Credentials** (Cluster > Informations d’identification cloud).

Après avoir renseigné tous les paramètres requis dans l’Assistant, cliquez sur le bouton **Add Filer** (Ajouter un filer) pour envoyer la modification.

Après un court instant, le système de stockage apparaît dans la liste des core filers sous **Dashboard** (Tableau de bord) et est accessible par le biais des pages de paramètres du core filer.

![Core filer « Flurry-NAS » dans la page de paramètres Manage Core Filers, avec la vue des détails du core filer développée](media/fxt-cluster-config/core-filer-in-manage-page.png)

Il manque un serveur virtuel pour le core filer illustré dans cette capture d’écran. Vous devez associer le core filer à un serveur virtuel et créer une jonction pour permettre aux clients d’accéder au stockage. Ces étapes sont décrites ci-dessous dans [Configurer l’espace de noms](#configure-the-namespace).

## <a name="configure-the-namespace"></a>Configurer l’espace de noms

Le cluster Azure FXT Edge Filer crée un système de fichiers virtuel, appelé *espace de noms du cluster*, qui simplifie l’accès des clients aux données stockées sur différents systèmes back-end. Du fait que les clients demandent les fichiers en utilisant un chemin virtuel, les systèmes de stockage peuvent être ajoutés ou remplacés sans avoir à changer le workflow côté client.

L’espace de noms du cluster vous permet également de présenter les systèmes de stockage NAS et cloud dans une structure de fichiers similaire.

Les serveurs virtuels du cluster tiennent à jour l’espace de noms et le contenu fourni aux clients. Deux étapes sont nécessaires pour créer l’espace de noms du cluster :

1. Créer un serveur virtuel
1. Configurer des jonctions entre les systèmes de stockage back-end et les chemins des systèmes de fichiers exposés aux clients

### <a name="create-a-vserver"></a>Créer un serveur virtuel

Les serveurs virtuels sont des serveurs de fichiers virtuels qui contrôlent les flux de données entre un client et les core filers du cluster :

* Les serveurs virtuels hébergent les adresses IP exposées au client
* Les serveurs virtuels créent l’espace de noms et configurent les jonctions qui mappent la structure de répertoires virtuels côté client aux exportations sur le stockage back-end
* Les serveurs virtuels appliquent les contrôles d’accès aux fichiers, y compris les stratégies d’exportation des core filers et les systèmes d’authentification utilisateur
* Les serveurs virtuels fournissent l’infrastructure SMB

Avant de commencer à configurer un serveur virtuel pour le cluster, lisez la documentation connexe et rapprochez-vous de votre représentant Microsoft pour obtenir des explications supplémentaires sur l’espace de noms et les serveurs virtuels. Si vous utilisez des réseaux VLAN, [créez-les](fxt-configure-network.md#adjust-network-settings) avant de créer le serveur virtuel.

Consultez ces sections du Guide de configuration du cluster pour vous familiariser avec les fonctionnalités du serveur virtuel FXT et de l’espace de noms global :

* [Creating and Working with VServers](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/settings_overview.html#creating-and-working-with-vservers) (Création et utilisation de serveurs virtuels)
* [Using a Global Namespace](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gns_overview.html) (Utilisation d’un espace de noms global)
* [Creating a VServer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_vserver_manage.html#creating-a-vserver) (Création d’un serveur virtuel)

Vous devez avoir au moins un serveur virtuel pour votre cluster.

Pour créer un serveur virtuel, vous devez indiquer les informations suivantes :

* Le nom souhaité pour le serveur virtuel

* La plage d’adresses IP exposées aux clients et gérées par le serveur virtuel

  Vous devez indiquer une plage unique d’adresses IP contiguës quand vous créez le serveur virtuel. Vous pourrez ajouter d’autres adresses ultérieurement dans la page de paramètres **Client Facing Network** (Réseau exposé aux clients).

* Le réseau VLAN à utiliser pour ce serveur virtuel (si votre réseau en comporte plusieurs)

Utilisez la page de paramètres **VServer** > **Manage VServers** (Serveur virtuel > Gérer les serveurs virtuels) pour créer un autre serveur virtuel. Pour plus d’informations, consultez [Creating a VServer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_vserver_manage.html#creating-a-vserver) (Création d’un serveur virtuel) dans le Guide de configuration du cluster.

![Fenêtre indépendante pour créer un serveur virtuel](media/fxt-cluster-config/new-vserver.png)

### <a name="create-a-junction"></a>Créer une jonction

Une *jonction* mappe un chemin de stockage back-end à l’espace de noms exposé aux clients.

Ce système vous permet de simplifier le chemin utilisé dans les points de montage clients, et de mettre facilement à l’échelle la capacité, car un seul chemin virtuel peut prendre en charge le stockage à partir de plusieurs core filers.

![Page de l’Assistant Add New Junction avec les paramètres définis](media/fxt-cluster-config/add-junction-full.png)

Pour plus d’informations sur la création d’une jonction d’espace de noms, consultez [**VServer** > **Namespace**](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_namespace.html) (Serveur virtuel > Espace de noms) dans le Guide de configuration du cluster.

![Page de paramètres VServer > Namespace avec les détails d’une jonction](media/fxt-cluster-config/namespace-populated.png)

## <a name="configure-export-rules"></a>Configurer les règles d’exportation

Une fois que vous avez créé un serveur virtuel et un core filer, vous devez personnaliser les règles d’exportation et exporter les stratégies qui contrôlent l’accès des clients aux fichiers lors des exportations du core filer.

Tout d’abord, utilisez la page **VServer** > **Export Rules** (Serveur virtuel > Règles d’exportation) pour ajouter de nouvelles règles, modifier la stratégie par défaut ou créer votre stratégie d’exportation personnalisée.

Ensuite, utilisez la page **VServer** > **Export Policies** (Serveur virtuel > Exporter les stratégies) pour appliquer la stratégie personnalisée aux exportations du core filer auxquelles les clients accèdent par le biais de ce serveur virtuel.

Pour plus de détails, lisez l’article [Controlling Access to Core Filer Exports](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/export_rules_overview.html) (Contrôle de l’accès aux exportations du core filer) dans le Guide de configuration du cluster.

## <a name="next-steps"></a>Étapes suivantes

Après avoir ajouté le stockage et configuré l’espace de noms exposé aux clients, effectuez la configuration initiale de votre cluster : 

> [!div class="nextstepaction"]
> [Configurer les paramètres réseau du cluster](fxt-configure-network.md)
