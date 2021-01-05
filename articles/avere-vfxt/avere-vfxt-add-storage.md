---
title: Configurer le stockage Avere vFXT - Azure
description: Découvrez comment ajouter un système de stockage principal pour un cluster dans Avere vFXT pour Azure. Si vous avez créé un conteneur d’objets blob Azure avec le cluster, il est prêt à l’emploi.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: c17d3c7cd2cf6fe5bca725cf94344b2d2cb50bf2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001522"
---
# <a name="configure-storage"></a>Configurer le stockage

Cette étape configure un système de stockage back-end pour votre cluster vFXT.

> [!TIP]
> Si vous avez créé un conteneur d’objets blob Azure et le cluster Avere vFXT, le conteneur sera préconfiguré et prêt à l’emploi.

Suivez ces instructions si vous n'avez pas créé de nouveau conteneur d'objets blob avec votre cluster, ou si vous souhaitez ajouter du matériel ou un système de stockage basé sur le cloud.

Il existe deux tâches principales :

1. [Créer un système de stockage principal (core filer)](#create-a-core-filer), qui connecte votre cluster vFXT à un système de stockage existant ou à un conteneur de compte de stockage Azure.

1. [Créer une jonction d’espace de noms](#create-a-junction), qui définit le chemin emprunté par les clients.

Vous effectuez ces étapes à l’aide du panneau de configuration Avere. Lisez [Accéder au cluster vFXT](avere-vfxt-cluster-gui.md) pour savoir comment l’utiliser.

## <a name="create-a-core-filer"></a>Créer un système de stockage principal

« Core filer » est un terme de vFXT qui désigne un système de stockage back-end. Le stockage peut être une appliance NAS matérielle telle que NetApp ou Isilon, ou peut être un magasin d’objets cloud. Vous trouverez plus d’informations sur les systèmes de stockage principaux dans le [guide des paramètres de cluster Avere](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/settings_overview.html#managing-core-filers).

Pour ajouter un système de stockage principal, choisissez l’un des deux types de système de stockage principal suivants :

* [Système de stockage principal NAS](#nas-core-filer) : explique comment ajouter un système de stockage principal NAS
* [Système de stockage principal cloud avec stockage Azure](#azure-blob-storage-cloud-core-filer) : explique comment ajouter un conteneur de stockage Blob Azure en tant que système de stockage principal cloud

### <a name="nas-core-filer"></a>Système de stockage principal NAS

Un système de stockage principal NAS peut être une appliance NetApp ou Isilon locale, ou un point de terminaison NAS dans le cloud. Le système de stockage doit disposer d’une connexion fiable à grande vitesse au cluster Avere vFXT - par exemple, une connexion ExpressRoute 1 Gbit/s (pas un VPN) - et doit donner au cluster un accès root aux exportations NAS utilisées.

Pour ajouter un système de stockage principal NAS, effectuez les étapes suivantes :

1. À partir du panneau de configuration Avere, cliquez sur l’onglet **Settings** (Paramètres) en haut.

1. Cliquez sur **Core Filer** (Système de stockage principal)  > **Manage Core Filers** (Gérer les systèmes de stockage principaux) sur la gauche.

1. Cliquez sur **Créer**.

   ![Capture d’écran de la page Add new core filer (Ajouter un nouveau système de stockage principal) avec le curseur sur le bouton Create (Créer)](media/avere-vfxt-add-core-filer-start.png)

1. Remplissez les informations nécessaires dans l’Assistant :

   * Nommez votre système de stockage principal.
   * Le cas échéant, fournissez un nom de domaine complet (FQDN). Sinon, fournissez une adresse IP ou un nom d’hôte qui correspond à votre système de stockage principal.
   * Choisissez votre classe de système de stockage dans la liste. En cas de doute, choisissez **Other** (Autre).

     ![Capture d’écran de la page d’ajout d’un nouveau système de stockage principal avec le nom du système de stockage principal et son nom de domaine complet](media/avere-vfxt-add-core-filer.png)
  
   * Cliquez sur **Next** (Suivant) et choisissez une stratégie de cache.
   * Cliquez sur **Add Filer** (Ajouter le système de stockage).
   * Pour plus d’informations, reportez-vous à [Adding a new NAS core filer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_nas.html) (Ajout d’un nouveau système de stockage principal NAS) dans le guide des paramètres de cluster Avere.

Ensuite, passez à la section [Créer une jonction](#create-a-junction).  

### <a name="azure-blob-storage-cloud-core-filer"></a>Système de stockage principal cloud avec le stockage Blob Azure

Pour utiliser le stockage Blob Azure comme stockage back-end de votre cluster vFXT, vous avez besoin d’un conteneur vide à ajouter comme système de stockage back-end.

Pour ajouter un stockage d’objets blob à votre cluster, vous devez effectuer les tâches suivantes :

* Créer un compte de stockage (étape 1, ci-dessous)
* Créer un conteneur d’objets blob vide (étapes 2 et 3)
* Ajouter la clé d’accès de stockage en tant qu’information d’identification cloud pour le cluster vFXT (étapes 4 à 6)
* Ajouter le conteneur d’objets blob en tant que système de stockage principal pour le cluster vFXT (étapes 7 à 9)
* Créer une jonction d’espace de noms que les clients utilisent pour monter le système de stockage principal ([Créer une jonction](#create-a-junction), même procédure pour le stockage matériel et cloud)

> [!TIP]
> Si vous créez un conteneur d’objets blob lorsque vous créez un cluster Avere vFXT pour Azure, le modèle de déploiement configure automatiquement le conteneur comme un système de stockage principal (ceci est également vrai si vous utilisez le script de création, qui est disponible à la demande). Vous n’aurez pas besoin de configurer le système de stockage principal.
>
> L’outil de création de cluster se charge des tâches de configuration suivantes :
>
> * Il crée un conteneur d’objets blob dans le compte de stockage fourni
> * Il définit le conteneur comme un système de stockage principal
> * Il crée une jonction d’espace de noms avec le conteneur
> * Il crée un point de terminaison de service de stockage sur le réseau virtuel du cluster

Pour ajouter le stockage d’objets blob après avoir créé le cluster, effectuez les étapes ci-après.

1. Créez un compte de stockage V2 universel avec ces paramètres :

   * **Abonnement** : même chose que le cluster vFXT
   * **Groupe de ressources** : même chose que le groupe de clusters vFXT (facultatif)
   * **Emplacement** : même chose que le cluster vFXT
   * **Performances** : Standard (le stockage Premium n’est pas pris en charge)
   * **Type de compte** : V2 universel (StorageV2)
   * **Réplication** : stockage localement redondant (LRS)
   * **Niveau d’accès** : élevé
   * **Transfert sécurisé requis** : désactivez cette option (valeur non par défaut)
   * **Réseaux virtuels** : facultatif

   Vous pouvez utiliser le portail Azure ou cliquez sur le bouton « Déployer sur Azure » ci-dessous.

   [![Bouton permettant de créer le compte de stockage](media/deploytoazure.png)](https://ms.portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAvere%2Fmaster%2Fsrc%2Fvfxt%2Fstorageaccount%2Fazuredeploy.json)

1. Une fois le compte créé, accédez à la page du compte de stockage.

   ![Nouveau compte de stockage dans le portail Azure](media/avere-vfxt-new-storage-acct.png)

1. Créez un conteneur d’objets blob : Dans la page Vue d’ensemble, cliquez sur **Conteneurs**, puis cliquez sur **+ Conteneur**. Utilisez n’importe quel nom de conteneur et assurez-vous que l’accès est défini sur **Privé**.

   ![Page des objets blob de stockage où le bouton +Conteneur est entouré, et où un nouveau conteneur est créé dans une page contextuelle](media/avere-vfxt-new-blob.png)

1. Obtenez la clé du compte de stockage Azure en cliquant sur **Clés d’accès** sous **Paramètres**. Copiez l’une des clés fournies.

   ![Interface utilisateur du portail Azure pour la copie de la clé](media/avere-vfxt-copy-storage-key.png)

1. Ouvrez le panneau de configuration Avere associé à votre cluster. Cliquez sur **Settings** (Paramètres), puis ouvrez **Cluster** > **Cloud Credentials** (Informations d’identification cloud) dans le volet de navigation de gauche. Dans la page Cloud Credentials (Informations d’identification cloud), cliquez sur **Add Credential** (Ajouter des informations d’identification).

   ![Cliquer sur le bouton Add Credential (Ajouter des informations d’identification) dans la page de configuration des informations d’identification cloud](media/avere-vfxt-new-credential-button.png)

1. Renseignez les informations suivantes afin de créer des informations d’identification pour le système de stockage principal cloud :

   | Champ | Valeur |
   | --- | --- |
   | Credential name (Nom des informations d’identification) | Tout nom descriptif |
   | Type de service | (sélectionnez la clé d’accès du stockage Azure) |
   | Locataire | nom du compte de stockage |
   | Abonnement | ID d’abonnement |
   | Storage Access Key (Clé d’accès de stockage) | Clé du compte de stockage Azure (copiée à l’étape précédente) |

   Cliquez sur **Envoyer**.

   ![Formulaire des informations d’identification cloud rempli dans le panneau de configuration Avere](media/avere-vfxt-new-credential-submit.png)

1. Ensuite, créez le système de stockage principal. Dans la partie gauche du panneau de configuration Avere, cliquez sur **Core Filer** (Système de stockage principal) >  **Manage Core Filers** (Gérer les systèmes de stockage principaux).

1. Cliquez sur le bouton **Create** dans la page de paramètres **Manage Core Filers** (Gérer les systèmes de stockage principaux).

1. Renseignez l’Assistant :

   * Sélectionnez le type de système de stockage **Cloud**.
   * Nommez le nouveau système de stockage principal et cliquez sur **Next** (Suivant).
   * Acceptez la stratégie de cache par défaut et passez à la troisième page.
   * Dans **Service type** (Type de service), choisissez **Azure storage** (Stockage Azure).
   * Choisissez les informations d’identification créées précédemment.
   * Définissez **Bucket contents** (Contenu du compartiment) sur **Empty** (Vide).
   * Définissez **Certificate verification** (Vérification du certificat) sur **Disabled** (Désactivé).
   * Définissez **Compression mode** (Mode de compression) sur **None** (Aucun).
   * Cliquez sur **Suivant**.
   * Dans la quatrième page, entrez le nom du conteneur dans **Bucket name** (nom du compartiment) sous la forme *nom_compte_de_stockage*/*nom_conteneur*.
   * Si vous le souhaitez, définissez **Encryption type** (Type de chiffrement) sur **None** (Aucun).  Stockage Azure est chiffré par défaut.
   * Cliquez sur **Add Filer** (Ajouter le système de stockage).

   Pour plus d’informations, consultez [Adding a new cloud core filer](<https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_cloud.html>) (Ajout d’un nouveau système de stockage principal cloud) dans le guide de configuration des clusters Avere.

La page est actualisée, ou vous pouvez l’actualiser vous-même : votre nouveau système de stockage principal apparaît alors.

Ensuite, vous devez [créer une jonction](#create-a-junction).

## <a name="create-a-junction"></a>Créer une jonction

Une jonction est un chemin que vous créez pour les clients. Les clients empruntent le chemin pour gagner la destination choisie par vos soins.

Par exemple, vous pouvez créer `/vfxt/files` pour établir une correspondance avec l’exportation `/vol0/data` de votre système de stockage principal NetApp et le sous-répertoire `/project/resources`.

Vous trouverez plus d’informations sur les jonctions dans la [section consacrée aux espaces de noms du guide de configuration de clusters Avere](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_namespace.html).

Suivez ces étapes dans l’interface du Panneau de configuration Avere :

* Cliquez sur **VServer** > **Namespace** (Espace de noms) dans le coin supérieur gauche.
* Fournissez un chemin d’espace de noms commençant par / (barre oblique), comme ``/vfxt/data``.
* Choisissez votre système de stockage principal.
* Choisissez l’exportation du système de stockage principal.
* Cliquez sur **Suivant**.

  ![Capture d’écran de la page « Add new junction » (Ajouter une nouvelle jonction) avec les champs renseignés pour la jonction, le système de stockage principal et l’exportation](media/avere-vfxt-add-junction.png)

La jonction s’affiche après quelques secondes. Si nécessaire, créez d’autres jonctions.

Une fois la jonction créée, les clients utilisent le chemin de l’espace de noms pour accéder aux fichiers à partir du système de stockage.

## <a name="next-steps"></a>Étapes suivantes

* [Monter le cluster Avere vFXT](avere-vfxt-mount-clients.md)
* Découvrez comment [déplacer des données vers un nouveau conteneur d’objets blob](avere-vfxt-data-ingest.md) de manière efficace
