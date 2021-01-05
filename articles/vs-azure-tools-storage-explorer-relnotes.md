---
title: Notes de publication de l’Explorateur Stockage Microsoft Azure
description: Lisez les notes de publication de la dernière version d’Explorateur Stockage Microsoft Azure. Les notes de publication pour les versions précédentes sont également affichées.
services: storage
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: ''
ms.assetid: ''
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/12/2018
ms.author: cawa
ms.openlocfilehash: 56654f09d3a92eec5b86d8c8e4cd76fec0639394
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97653042"
---
# <a name="microsoft-azure-storage-explorer-release-notes"></a>Notes de publication de l’Explorateur Stockage Microsoft Azure

Cet article contient les dernières notes de publication de l’Explorateur Stockage Azure, ainsi que celles des versions précédentes. 

[L’Explorateur Stockage Microsoft Azure](./vs-azure-tools-storage-manage-with-storage-explorer.md) est une application autonome qui vous permet d’utiliser facilement les données du Stockage Azure sur Windows, maOS et Linux.

Pour télécharger les versions précédentes de l’Explorateur Stockage, visitez la [page des versions](https://github.com/microsoft/AzureStorageExplorer/releases) de notre dépôt GitHub.

## <a name="version-1110"></a>Version 1.11.0
4/11/2019

### <a name="new"></a>Nouveau
* Les opérations pour les objets blob, ADLS Gen2 et les disques managés utilisent l’outil AzCopy intégré. Plus spécifiquement, les opérations suivantes sont effectuées à l’aide d’AzCopy :
   * Objets blob
      * Ouverture pour modification + Chargement
      * Chargement, y compris glisser-déplacer
      * Téléchargement
      * Copier-coller #1249
      * DELETE
   * Objets blob ADLS Gen2
      * Chargement, y compris glisser-déplacer
      * Téléchargement
      * Copier-coller
      * Suppression, y compris la suppression de dossier
   * Disques managés
      * Télécharger
      * Téléchargement
      * Copier-coller

   En outre, plusieurs fonctionnalités fréquemment demandées ont été ajoutées à l’expérience AzCopy intégrée :
   * Résolutions de conflit – Vous serez invité à résoudre les conflits pendant les transferts. #1455
   * Charger en tant qu’objets blob de page – Vous pouvez choisir si AzCopy doit télécharger ou non les fichiers .vhd et .vhdx en tant qu’objets blob de page. #1164 et #1601
   * Paramètres AzCopy configurables – Plusieurs paramètres ont été ajoutés pour régler les performances d’AzCopy et l’utilisation des ressources. Pour plus d’informations, voir ci-dessous.

* Pour permettre un accès multi-protocole à ADLS Gen2 et aux objets blob, et améliorer encore les expériences ADLS Gen2, nous avons ajouté les fonctionnalités suivantes pour les comptes ADLS Gen2 :
   * Recherche à l’aide de noms conviviaux pour définir les autorisations ACL
   * Affichage des conteneurs masqués, tels que $logs et $web
   * Acquisition et arrêt du bail de conteneur
   * Acquisition et arrêt du bail d’objet blob #848
   * Gestion des stratégies d’accès au conteneur
   * Configuration des niveaux d’accès aux objets blob
   * Copier-coller des objets blob

* Dans cette version, nous proposons en préversion 17 langues supplémentaires. Vous pouvez basculer vers la langue de votre choix dans la page Paramètres sous « Application » → « Paramètres régionaux » → « Langue (préversion) ». Nous travaillons toujours activement à la traduction de chaînes supplémentaires et à l’amélioration de la qualité de la traduction. Si vous avez des commentaires sur une traduction ou si vous remarquez une chaîne qui n’est pas encore traduite, [ouvrez un ticket d’incident dans GitHub](https://github.com/microsoft/AzureStorageExplorer/issues/new?assignees=&labels=%F0%9F%8C%90%20localization&template=bug-report.md&title=).
* Dans chaque version, nous essayons d’intégrer quelques paramètres pour permettre un réglage précis de l’Explorateur Stockage. Dans cette version, nous avons ajouté des paramètres pour mieux configurer AzCopy, ainsi que pour masquer les nœuds de service :
   * Limite de bande passante AzCopy – Permet de contrôler la quantité de réseau utilisée par AzCopy. Ce paramètre se trouve dans « Transferts » → « AzCopy » → « Taux de transfert maximal ». #1099
   * AzCopy MD5 Check – Vous permet de configurer si AzCopy recherche des hachages MD5 lors du téléchargement, et de quelle manière. Ce paramètre se trouve dans « Transferts » → « AzCopy » → « Vérifier MD5 ».
   * Taille de mémoire tampon et concurrence AzCopy – Par défaut, AzCopy analyse votre ordinateur pour déterminer des valeurs par défaut raisonnables pour ces paramètres. Toutefois, si vous rencontrez des problèmes de performances, vous pouvez utiliser ces paramètres avancés pour personnaliser davantage la façon dont AzCopy s’exécute sur votre ordinateur. Ces paramètres sont disponibles sous « Transferts » → « AzCopy ». #994
   * Afficher et masquer les nœuds de service – Ces paramètres vous permettent d’afficher ou de masquer tous les services Azure pris en charge par l’Explorateur Stockage. Ces paramètres sont disponibles sous la section « Services ». #1877

* Quand vous créez un instantané d’un disque managé, un nom par défaut est maintenant fourni. #1847
* Lors de l’attachement avec Azure AD, si vous attachez un conteneur d’objets blob ADLS Gen2, « (ADLS Gen2) » apparaît en regard du nœud. #1861

### <a name="fixes"></a>Correctifs
* Lors de la copie, du chargement ou du téléchargement de disques volumineux, l’Explorateur Stockage ne parvient parfois pas à révoquer l’accès aux disques impliqués dans l’opération. Ce problème a été résolu. #2048
* Échec des statistiques de table lors de l’affichage d’une requête de clé de partition. Ce problème a été résolu. #1886

### <a name="known-issues"></a>Problèmes connus
* L’Explorateur Stockage 1.11.0 requiert désormais un point de terminaison DFS (tel que « myaccount.dfs.core.windows.net ») à attacher aux conteneurs ADLS Gen2. Les versions précédentes de l’Explorateur Stockage vous permettaient d’utiliser un point de terminaison d’objet blob. Ces attachements peuvent ne plus fonctionner après la mise à niveau vers 1.11.0. Si vous rencontrez ce problème, rattachez à l’aide du point de terminaison DFS.
* L’appartenance des paramètres numériques à une plage valide n’est pas vérifiée.#2140
* La copie de conteneurs d’objets blob d’un compte de stockage vers un autre dans l’arborescence peut échouer. Nous étudions actuellement ce problème.#2124
* Le paramètre Actualisation automatique n’affecte pas encore toutes les opérations de l’Explorateur d’objets blob.
* Les fonctionnalités des disques managés ne sont pas encore prises en charge dans Azure Stack.
* Si un chargement ou un collage sur disque échoue et qu’un disque a été créé avant l’échec, l’Explorateur Stockage ne supprime pas le disque pour vous.
* Selon que vous annulez un chargement ou un collage sur disque, il est possible de conserver le nouveau disque dans un état endommagé. Dans ce cas, vous devez supprimer le nouveau disque ou appeler manuellement les API de disque pour remplacer le contenu du disque de manière à ce qu’il ne soit plus endommagé.
* Lorsque vous utilisez RBAC, l’Explorateur Stockage requiert certaines autorisations de couche de gestion afin d’accéder à vos ressources de stockage. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md) pour plus d’informations.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Voir #537 pour plus d’informations.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
   * ADLS Gen2
   * Disques managés
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* L’exécution de l’Explorateur Stockage sous Linux requiert l’installation préalable de certaines dépendances. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md?tabs=1804#linux-dependencies) de l’Explorateur Stockage pour plus d’informations.

## <a name="previous-releases"></a>Versions précédentes

* [Version 1.10.1](#version-1101)
* [Version 1.10.0](#version-1100)
* [Version 1.9.0](#version-190)
* [Version 1.8.1](#version-181)
* [Version 1.8.0](#version-180)
* [Version 1.7.0](#version-170)
* [Version 1.6.2](#version-162)
* [Version 1.6.1](#version-161)
* [Version 1.6.0](#version-160)
* [Version 1.5.0](#version-150)
* [Version 1.4.4](#version-144)
* [Version 1.4.3](#version-143)
* [Version 1.4.2](#version-142)
* [Version 1.4.1](#version-141)
* [Version 1.3.0](#version-130)
* [Version 1.2.0](#version-120)
* [Version 1.1.0](#version-110)
* [Version 1.0.0](#version-100)
* [Version 0.9.6](#version-096)
* [Version 0.9.5](#version-095)
* [Versions 0.9.4 et 0.9.3](#version-094-and-093)
* [Version 0.9.2](#version-092)
* [Versions 0.9.1 et 0.9.0](#version-091-and-090)
* [Version 0.8.16](#version-0816)
* [Version 0.8.14](#version-0814)
* [Version 0.8.13](#version-0813)
* [Versions 0.8.12, 0.8.11 et 0.8.10](#version-0812-and-0811-and-0810)
* [Versions 0.8.9 et 0.8.8](#version-089-and-088)
* [Version 0.8.7](#version-087)
* [Version 0.8.6](#version-086)
* [Version 0.8.5](#version-085)
* [Version 0.8.4](#version-084)
* [Version 0.8.3](#version-083)
* [Version 0.8.2](#version-082)
* [Version 0.8.0](#version-080)
* [Version 0.7.20160509.0](#version-07201605090)
* [Version 0.7.20160325.0](#version-07201603250)
* [Version 0.7.20160129.1](#version-07201601291)
* [Version 0.7.20160105.0](#version-07201601050)
* [Version 0.7.20151116.0](#version-07201511160)

## <a name="version-1101"></a>Version 1.10.1
19/9/2019

### <a name="hotfix"></a>Correctif logiciel
* Certains utilisateurs ont rencontré une erreur dans la version 1.10.0 en essayant de voir leurs données dans leurs comptes ADLS Gen 1. Cette erreur empêchait le rendu correct du panneau de l’Explorateur. Ce problème a été résolu. #1853 #1865

### <a name="new"></a>Nouveau
* L’Explorateur Stockage a désormais une interface utilisateur dédiée aux paramètres. Vous pouvez y accéder en sélectionnant Modifier → Paramètres ou en cliquant sur l’icône Paramètres (engrenage) dans la barre d’outils verticale située à gauche. Cette fonctionnalité est la première étape que nous prenons pour proposer une variété de [paramètres demandés par l’utilisateur](https://github.com/microsoft/AzureStorageExplorer/labels/%3Abulb%3A%20setting%20candidate). À compter de cette version, les paramètres suivants sont pris en charge :
  * Thème
  * Proxy
  * Déconnexion en quittant #6
  * Activer la connexion du flux de code de l’appareil
  * Actualisation automatique #1526
  * Activer AzCopy
  * Durée SAS AzCopy – Pour ajouter d’autres paramètres, [ouvrez un ticket d’incident sur GitHub](https://github.com/microsoft/AzureStorageExplorer/issues/new?assignees=&labels=%3Abulb%3A%20setting%20candidate&template=feature_request.md&title=) décrivant le paramètre que vous souhaitez afficher.
* L’Explorateur Stockage prend maintenant en charge les disques managés. Vous pouvez :
  * Charger un disque dur virtuel local sur un nouveau disque
  * Télécharger un disque
  * Copier et coller des disques dans des groupes de ressources et des régions
  * Supprimer des disques
  * Créer un instantané d’un disque

Le chargement, le téléchargement et la copie entre régions des disques reposent sur AzCopy v10.
* Vous pouvez désormais installer l’Explorateur Stockage par le biais du Snap Store sur Linux. Si vous effectuez l’installation par le biais du Snap Store, toutes les dépendances sont installées pour vous, même .NET Core ! À l’heure actuelle, nous savons que Explorateur Stockage s’exécute correctement sur Ubuntu et CentOS. Si vous rencontrez des problèmes lors de l’installation à partir du Snap Store sur d’autres distributions Linux, [signalez le problème sur GitHub](https://github.com/microsoft/AzureStorageExplorer/issues/new?assignees=&labels=snaps&template=bug-report.md&title=). Pour en savoir plus sur l’installation à partir du Snap Store, consultez notre [guide de démarrage rapide](./vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=linux). #68
* Deux changements majeurs ont été apportés à l’attachement à Azure Active Directory (Azure AD) pour rendre la fonctionnalité plus utile pour les utilisateurs d’ADLS Gen2 :
  * Vous sélectionnez à présent le locataire dans lequel se trouve la ressource que vous attachez. Vous n’avez donc plus besoin d’un accès RBAC à l’abonnement de la ressource.
  * Si vous attachez un conteneur d’objets blob ADLS Gen2, vous pouvez désormais effectuer l’attachement à un chemin spécifique dans le conteneur.
* Quand vous gérez des listes de contrôle d’accès pour des fichiers et des dossiers ADLS Gen2, l’Explorateur Stockage affiche désormais les noms conviviaux des entités dans la liste de contrôle d’accès. #957
* Lors de l’ajout par le biais d’OID à une liste de contrôle d’accès ADLS Gen2, l’Explorateur Stockage vérifie à présent que l’OID appartient à une entité valide dans votre locataire. #1603
* Les raccourcis clavier permettant de naviguer entre les onglets utilisent désormais plus de combinaisons de touches standard. #1018
* Cliquez sur le bouton central de la souris dans un onglet pour le fermer. #1348
* Si un transfert AzCopy contient des omissions et zéro échec, l’Explorateur Stockage affiche désormais une icône d’avertissement pour mettre en évidence les omissions. #1490
* L’outil AzCopy intégré a été mis à jour vers la version 10.2.1. Par ailleurs, vous pouvez maintenant afficher la version de AzCopy installée dans la boîte de dialogue À propos de. #1343

### <a name="fixes"></a>Correctifs
* De nombreux utilisateurs se heurtent à des erreurs du type « impossible de lire la version de non défini » ou « impossible de lire la connexion de non défini » lors de l’utilisation de comptes de stockage attachés. Bien que nous continuions toujours à étudier la cause racine de ce problème, nous avons amélioré la gestion des erreurs concernant le chargement des comptes de stockage attachés dans la version 1.10.0. #1626, #985 et #1532
* Il était possible pour l’arborescence de l’explorateur (côté gauche) d’entrer dans un état dans lequel le focus passait de manière répétée au nœud supérieur. Ce problème a été résolu. #1596
* Lors de la gestion des instantanés d’un objet blob, les lecteurs d’écran ne lisaient pas l’horodatage associé à l’instantané. Ce problème a été résolu. #1202
* Le paramètre de proxy sur macOS n’était pas défini à temps pour que le processus d’authentification les utilise. Ce problème a été résolu. #1567
* Si un compte de stockage dans un cloud souverain était attaché à l’aide du nom et de la clé, AzCopy ne fonctionnait pas. Ce problème a été résolu. #1544
* En cas d’attachement par le biais d’une chaîne de connexion, l’Explorateur Stockage supprime désormais les espaces de fin. #1387

### <a name="known-issues"></a>Problèmes connus
* Le paramètre Actualisation automatique n’affecte pas encore toutes les opérations de l’Explorateur d’objets blob.
* Les fonctionnalités des disques managés ne sont pas encore prises en charge dans Azure Stack.
* Si un chargement ou un collage sur disque échoue et qu’un disque a été créé avant l’échec, l’Explorateur Stockage ne supprime pas le disque pour vous.
* Selon que vous annulez un chargement ou un collage sur disque, il est possible de conserver le nouveau disque dans un état endommagé. Dans ce cas, vous devez supprimer le nouveau disque ou appeler manuellement les API de disque pour remplacer le contenu du disque de manière à ce qu’il ne soit plus endommagé.
* Selon que vous annulez un chargement ou un collage sur disque, il est possible de conserver le nouveau disque dans un état endommagé. Dans ce cas, vous devez supprimer le nouveau disque ou appeler manuellement les API de disque pour remplacer le contenu du disque de manière à ce qu’il ne soit plus endommagé.
* Lorsque vous effectuez un téléchargement d’objets Blob non-AzCopy, le hachage MD5 pour les fichiers volumineux n’est pas vérifié. Cela est dû à un bogue dans le SDK de stockage. [#1212](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1212)
* Lorsque vous utilisez RBAC, l’Explorateur Stockage requiert certaines autorisations de couche de gestion afin d’accéder à vos ressources de stockage. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md) pour plus d’informations.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Voir #537 pour plus d’informations.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
   * ADLS Gen2
   * Disques managés
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* L’exécution de l’Explorateur Stockage sous Linux requiert l’installation préalable de certaines dépendances. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md?tabs=1804#linux-dependencies) de l’Explorateur Stockage pour plus d’informations.


## <a name="version-1100"></a>Version 1.10.0
12/9/2019

### <a name="new"></a>Nouveau

* L’Explorateur Stockage a désormais une interface utilisateur dédiée aux paramètres. Vous pouvez y accéder en sélectionnant Modifier → Paramètres ou en cliquant sur l’icône Paramètres (engrenage) dans la barre d’outils verticale située à gauche. Cette fonctionnalité est la première étape que nous prenons pour proposer une variété de [paramètres demandés par l’utilisateur](https://github.com/microsoft/AzureStorageExplorer/labels/%3Abulb%3A%20setting%20candidate). À compter de cette version, les paramètres suivants sont pris en charge :
    * Thème
    * Proxy
    * Se déconnecter en quittant [6](https://www.github.com/Microsoft/AzureStorageExplorer/issues/6)
    * Activer la connexion du flux de code de l’appareil
    * Actualisation automatique [1526](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1526)
    * Activer AzCopy
    * Durée SAS AzCopy

    Pour ajouter d’autres paramètres, [ouvrez un problème sur GitHub décrivant le paramètre que vous souhaitez afficher](https://github.com/microsoft/AzureStorageExplorer/issues/new?assignees=&labels=%3Abulb%3A%20setting%20candidate&template=feature_request.md&title=).
* L’Explorateur Stockage prend maintenant en charge les disques managés. Vous pouvez :
    * Charger un disque dur virtuel local sur un nouveau disque
    * Télécharger un disque
    * Copier et coller des disques dans des groupes de ressources et des régions
    * Supprimer des disques
    * Créer un instantané d’un disque

    Le chargement, le téléchargement et la copie entre régions des disques reposent sur AzCopy v10.
* Vous pouvez désormais installer l’Explorateur Stockage par le biais du Snap Store sur Linux. Si vous effectuez l’installation par le biais du Snap Store, toutes les dépendances sont installées pour vous, même .NET Core ! À l’heure actuelle, nous savons que Explorateur Stockage s’exécute correctement sur Ubuntu et CentOS. Si vous rencontrez des problèmes lors de l’installation à partir du Snap Store sur d’autres distributions Linux, [signalez le problème sur GitHub](https://github.com/microsoft/AzureStorageExplorer/issues/new?assignees=&labels=snaps&template=bug-report.md&title=). Pour en savoir plus sur l’installation à partir du Snap Store, consultez notre [guide de démarrage rapide](./vs-azure-tools-storage-manage-with-storage-explorer.md). [68](https://www.github.com/Microsoft/AzureStorageExplorer/issues/68)
* Deux changements majeurs ont été apportés à l’attachement à Azure Active Directory (Azure AD) pour rendre la fonctionnalité plus utile pour les utilisateurs d’ADLS Gen2 : * Vous sélectionnez à présent le locataire dans lequel se trouve la ressource que vous attachez. Vous n’avez donc plus besoin d’un accès RBAC à l’abonnement de la ressource.
        * Si vous attachez un conteneur d’objets BLOB ADLS Gen2, vous pouvez désormais effectuer l’attachement à un chemin spécifique dans le conteneur.
* Quand vous gérez des listes de contrôle d’accès pour des fichiers et des dossiers ADLS Gen2, l’Explorateur Stockage affiche désormais les noms conviviaux des entités dans la liste de contrôle d’accès. [#957](https://www.github.com/Microsoft/AzureStorageExplorer/issues/957)
* Lors de l’ajout par le biais d’OID à une liste de contrôle d’accès ADLS Gen2, l’Explorateur Stockage vérifie à présent que l’OID appartient à une entité valide dans votre locataire. [1603](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1603)
* Les raccourcis clavier permettant de naviguer entre les onglets utilisent désormais plus de combinaisons de touches standard. [#1018](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1018)
* Cliquez sur le bouton central de la souris dans un onglet pour le fermer. [#1348](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1348)
* Si un transfert AzCopy contient des omissions et zéro échec, l’Explorateur Stockage affiche désormais une icône d’avertissement pour mettre en évidence les omissions. [#1490](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1490)
* L’outil AzCopy intégré a été mis à jour vers la version 10.2.1. Par ailleurs, vous pouvez maintenant afficher la version de AzCopy installée dans la boîte de dialogue À propos de. [#1343](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1343)

### <a name="fixes"></a>Correctifs

* De nombreux utilisateurs se heurtent à des erreurs du type « impossible de lire la version de non défini » ou « impossible de lire la connexion de non défini » lors de l’utilisation de comptes de stockage attachés. Bien que nous continuions toujours à étudier la cause racine de ce problème, nous avons amélioré la gestion des erreurs concernant le chargement des comptes de stockage attachés dans la version 1.10.0. [1626](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1626), [985](https://www.github.com/Microsoft/AzureStorageExplorer/issues/985) et [1532](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1532)
* Il était possible pour l’arborescence de l’explorateur (côté gauche) d’entrer dans un état dans lequel le focus passait de manière répétée au nœud supérieur. Ce problème a été résolu. [#1596](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1596)
* Lors de la gestion des instantanés d’un objet blob, les lecteurs d’écran ne lisaient pas l’horodatage associé à l’instantané. Ce problème a été résolu. [#1202](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1202)
* Le paramètre de proxy sur macOS n’était pas défini à temps pour que le processus d’authentification les utilise. Ce problème a été résolu. [1567](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1567)
* Si un compte de stockage dans un cloud souverain était attaché à l’aide du nom et de la clé, AzCopy ne fonctionnait pas. Ce problème a été résolu. [#1544](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1544)
* En cas d’attachement par le biais d’une chaîne de connexion, l’Explorateur Stockage supprime désormais les espaces de fin. [#1387](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1387)

### <a name="known-issues"></a>Problèmes connus

* Le paramètre Actualisation automatique n’affecte pas encore toutes les opérations de l’Explorateur d’objets blob.
* Les fonctionnalités des disques managés ne sont pas encore prises en charge dans Azure Stack.
* Si un chargement ou un collage sur disque échoue et qu’un disque a été créé avant l’échec, l’Explorateur Stockage ne supprime pas le disque pour vous.
* Selon que vous annulez un chargement ou un collage sur disque, il est possible de conserver le nouveau disque dans un état endommagé. Dans ce cas, vous devez supprimer le nouveau disque ou appeler manuellement les API de disque pour remplacer le contenu du disque de manière à ce qu’il ne soit plus endommagé.
* Lorsque vous effectuez un téléchargement d’objets Blob non-AzCopy, le hachage MD5 pour les fichiers volumineux n’est pas vérifié. Cela est dû à un bogue dans le SDK de stockage. [#1212](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1212)
* Lorsque vous utilisez RBAC, l’Explorateur Stockage requiert certaines autorisations de couche de gestion afin d’accéder à vos ressources de stockage. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md) pour plus d’informations.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Voir #537 pour plus d’informations.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
   * ADLS Gen2
   * Disques managés
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* L’exécution de l’Explorateur Stockage sous Linux requiert l’installation préalable de certaines dépendances. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md?tabs=1804#linux-dependencies) de l’Explorateur Stockage pour plus d’informations.

## <a name="version-190"></a>Version 1.9.0
1/7/2019

### <a name="download-azure-storage-explorer-190"></a>Télécharger l’Explorateur Stockage Azure 1.9.0
- [Explorateur Stockage Azure 1.9.0 pour Windows](https://go.microsoft.com/fwlink/?LinkId=708343)
- [Explorateur Stockage Azure 1.9.0 pour Mac](https://go.microsoft.com/fwlink/?LinkId=708342)
- [Explorateur Stockage Azure 1.9.0 pour Linux](https://go.microsoft.com/fwlink/?LinkId=722418)

### <a name="new"></a>Nouveau

* Vous pouvez désormais joindre des conteneurs d’objets blob via Azure AD (autorisations RBAC ou ACL). Cette fonctionnalité est destinée à aider les utilisateurs qui ont accès à des conteneurs, mais non aux comptes de stockage qu’ils incluent. Consultez notre guide de prise en main pour en savoir plus sur cette fonctionnalité.
* Vous pouvez désormais acquérir ou résilier un bail avec le contrôle d’accès en fonction du rôle. [#1354](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1354)
* La gestion des stratégies d’accès et la définition du niveau d’accès public sont désormais possibles avec le contrôle d’accès en fonction du rôle. [#1355](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1355)
* Le contrôle d’accès en fonction du rôle permet désormais de supprimer des dossiers d’objets blob. [#1450](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1450)
* Le contrôle d’accès en fonction du rôle permet désormais de modifier le niveau d’accès aux objets blob. [#1446](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1446)
* Vous pouvez maintenant réinitialiser rapidement l’accès rapide en cliquant sur Aide → Réinitialiser. [#1327](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1327)

### <a name="preview-features"></a>Fonctionnalités de préversion

* La connexion par flux de code d’appareil est à présent disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use Device Code Flow Sign-in » (Utiliser la connexion par flux de code d’appareil). Nous encourageons tous les utilisateurs qui ont eu des problèmes avec des fenêtres de connexion vides à essayer cette fonctionnalité, car elle peut s’avérer être une forme plus fiable de connexion.
* L’Explorateur Stockage intégré à AzCopy est actuellement disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use AzCopy for Improved Blob Upload and Download » (Utiliser AzCopy pour améliorer le chargement et le téléchargement des objets blob). Les transferts d’objets blob effectués avec AzCopy doivent être plus rapides et plus performants.

### <a name="fixes"></a>Correctifs

* Il est désormais possible de charger plus de 50 abonnements pour un seul compte. [#1416](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1416)
* Désormais, le bouton Connexion fonctionne sur la barre d’informations qui s’affiche en cas d’échec d’un lien direct. [#1358](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1358)
* Vous pouvez à présent charger des fichiers .app sur un ordinateur macOS. [#1119](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1119)
* À présent, la fonction Réessayer tout fonctionne lors du changement de nom d’un objet blob ayant échoué. [#992](https://www.github.com/Microsoft/AzureStorageExplorer/issues/992)
* L’opération Annuler fonctionne désormais lors de l’ouverture d’un objet blob. [#1464](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1464)
* Correction de plusieurs fautes d’orthographe et de problèmes liés à des info-bulles dans l’ensemble du produit. Merci à toutes les personnes qui nous ont signalé ces problèmes ! [#1303](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1303), [#1328](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1328), [#1329](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1329), [#1331](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1331), [#1336](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1336), [#1352](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1352), [#1368](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1368), [#1395](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1395)

### <a name="known-issues"></a>Problèmes connus

* Lorsque vous effectuez un téléchargement d’objets Blob non-AzCopy, le hachage MD5 pour les fichiers volumineux n’est pas vérifié. Cela est dû à un bogue dans le SDK de stockage. [#1212](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1212)
* Lorsque vous utilisez RBAC, l’Explorateur Stockage requiert certaines autorisations de couche de gestion afin d’accéder à vos ressources de stockage. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md) pour plus d’informations.
* La tentative d’accès aux objets Blob Gen2 ADLS derrière un proxy peut échouer.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Voir #537 pour plus d’informations.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
   * ADLS Gen2
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* L’exécution de l’Explorateur Stockage sous Linux requiert l’installation préalable de certaines dépendances. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md?tabs=1804#linux-dependencies) de l’Explorateur Stockage pour plus d’informations.

## <a name="version-181"></a>Version 1.8.1
13/05/2019

### <a name="hotfixes"></a>Correctifs logiciels
* Dans certains cas, en cliquant sur « Charger plus » au niveau de la ressource ne permet pas de renvoyer la page suivante de ressources. Ce problème a été résolu. [#1359](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1359)
* Sous Windows, les téléchargements AzCopy échouaient si un seul fichier ou dossier était téléchargé et que son nom contenait un caractère non valide pour un chemin d’accès Windows. Ce problème a été résolu. [#1350](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1350)
* Dans de très rares cas, lors du changement de nom d’un partage de fichiers ou d’un changement de nom dans un partage de fichiers, si les copies pour le changement de nom échouaient, ou si l’Explorateur Stockage ne pouvait pas confirmer la réussite des copies avec Azure, il pouvait arriver que l’Explorateur Stockage supprime des fichiers d’origine avant la fin de la copie. Ce problème a été résolu.

### <a name="new"></a>Nouveau

* La version d’AzCopy intégrée a été mise à jour vers la version 10.1.0.
* Les raccourcis Ctrl/Cmd+R peuvent maintenant être utilisés pour actualiser l’éditeur ayant le focus. [#1097](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1097)
* La version de l’API de Stockage Azure Stack a été remplacée par 2017-04-17.
* La boîte de dialogue de gestion des accès pour ADLS Gen2 maintiendra désormais la synchronisation du masque d’une manière semblable aux autres outils d’autorisations POSIX. L’interface utilisateur vous avertira aussi si une modification fait que les autorisations d’un utilisateur ou d’un groupe dépassent les limites du masque. [#1253](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1253)
* Pour les chargements d’AzCopy, l’indicateur permettant de calculer et de définir le hachage MD5 est maintenant activé. [#1223](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1223)


### <a name="preview-features"></a>Fonctionnalités de préversion

* La connexion par flux de code d’appareil est à présent disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use Device Code Flow Sign-in » (Utiliser la connexion par flux de code d’appareil). Nous encourageons tous les utilisateurs qui ont eu des problèmes avec des fenêtres de connexion vides à essayer cette fonctionnalité, car elle peut s’avérer être une forme plus fiable de connexion.
* L’Explorateur Stockage intégré à AzCopy est actuellement disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use AzCopy for Improved Blob Upload and Download » (Utiliser AzCopy pour améliorer le chargement et le téléchargement des objets blob). Les transferts d’objets blob effectués avec AzCopy doivent être plus rapides et plus performants.

### <a name="fixes"></a>Correctifs

* La boîte de dialogue des stratégies d’accès ne définit plus de date d’expiration sur les stratégies d’accès au stockage qui n’expirent pas. [#764](https://www.github.com/Microsoft/AzureStorageExplorer/issues/764)
* Certaines modifications ont été apportées à la boîte de dialogue de génération de SAP pour garantir que les stratégies d’accès stockées sont utilisées correctement lors de la génération d’une SAP. [#1269](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1269)
* Lors d’une tentative de chargement d’un fichier aligné de 512 octets dans un objet Blob de page, l’Explorateur Stockage affiche à présent une erreur plus pertinente. [#1050](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1050)
* La copie d’un conteneur d’objets Blob qui utilisait un nom d’affichage échouait. À présent, le nom réel du conteneur d’objets Blob est utilisé. [#1166](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1166)
* Il était impossible d’effectuer certaines actions sur un dossier ADLS Gen2 dont le nom contenait des caractères Unicode. Toutes les actions devraient désormais fonctionner. [#980](https://www.github.com/Microsoft/AzureStorageExplorer/issues/980)

### <a name="known-issues"></a>Problèmes connus

* Lorsque vous effectuez un téléchargement d’objets Blob non-AzCopy, le hachage MD5 pour les fichiers volumineux n’est pas vérifié. Cela est dû à un bogue dans le SDK de stockage. [#1212](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1212)
* Lorsque vous utilisez RBAC, l’Explorateur Stockage requiert certaines autorisations de couche de gestion afin d’accéder à vos ressources de stockage. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md) pour plus d’informations.
* La tentative d’accès aux objets Blob Gen2 ADLS derrière un proxy peut échouer.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Voir #537 pour plus d’informations.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
   * ADLS Gen2
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* L’exécution de l’Explorateur Stockage sous Linux requiert l’installation préalable de certaines dépendances. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md?tabs=1804#linux-dependencies) de l’Explorateur Stockage pour plus d’informations.

## <a name="version-180"></a>Version 1.8.0
01/05/2019

### <a name="new"></a>Nouveau

* La version d’AzCopy intégrée a été mise à jour vers la version 10.1.0.
* Les raccourcis Ctrl/Cmd+R peuvent maintenant être utilisés pour actualiser l’éditeur ayant le focus. [#1097](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1097)
* La version de l’API de Stockage Azure Stack a été remplacée par 2017-04-17.
* La boîte de dialogue de gestion des accès pour ADLS Gen2 maintiendra désormais la synchronisation du masque d’une manière semblable aux autres outils d’autorisations POSIX. L’interface utilisateur vous avertira aussi si une modification fait que les autorisations d’un utilisateur ou d’un groupe dépassent les limites du masque. [#1253](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1253)
* Pour les chargements d’AzCopy, l’indicateur permettant de calculer et de définir le hachage MD5 est maintenant activé. [#1223](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1223)


### <a name="preview-features"></a>Fonctionnalités de préversion

* La connexion par flux de code d’appareil est à présent disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use Device Code Flow Sign-in » (Utiliser la connexion par flux de code d’appareil). Nous encourageons tous les utilisateurs qui ont eu des problèmes avec des fenêtres de connexion vides à essayer cette fonctionnalité, car elle peut s’avérer être une forme plus fiable de connexion.
* L’Explorateur Stockage intégré à AzCopy est actuellement disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use AzCopy for Improved Blob Upload and Download » (Utiliser AzCopy pour améliorer le chargement et le téléchargement des objets blob). Les transferts d’objets blob effectués avec AzCopy doivent être plus rapides et plus performants.

### <a name="fixes"></a>Correctifs

* La boîte de dialogue des stratégies d’accès ne définit plus de date d’expiration sur les stratégies d’accès au stockage qui n’expirent pas. [#764](https://www.github.com/Microsoft/AzureStorageExplorer/issues/764)
* Certaines modifications ont été apportées à la boîte de dialogue de génération de SAP pour garantir que les stratégies d’accès stockées sont utilisées correctement lors de la génération d’une SAP. [#1269](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1269)
* Lors d’une tentative de chargement d’un fichier aligné de 512 octets dans un objet Blob de page, l’Explorateur Stockage affiche à présent une erreur plus pertinente. [#1050](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1050)
* La copie d’un conteneur d’objets Blob qui utilisait un nom d’affichage échouait. À présent, le nom réel du conteneur d’objets Blob est utilisé. [#1166](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1166)
* Il était impossible d’effectuer certaines actions sur un dossier ADLS Gen2 dont le nom contenait des caractères Unicode. Toutes les actions devraient désormais fonctionner. [#980](https://www.github.com/Microsoft/AzureStorageExplorer/issues/980)

### <a name="known-issues"></a>Problèmes connus

* Lorsque vous effectuez un téléchargement d’objets Blob non-AzCopy, le hachage MD5 pour les fichiers volumineux n’est pas vérifié. Cela est dû à un bogue dans le SDK de stockage. [#1212](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1212)
* Lorsque vous utilisez RBAC, l’Explorateur Stockage requiert certaines autorisations de couche de gestion afin d’accéder à vos ressources de stockage. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md) pour plus d’informations.
* La tentative d’accès aux objets Blob Gen2 ADLS derrière un proxy peut échouer.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Voir #537 pour plus d’informations.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
   * ADLS Gen2
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* L’exécution de l’Explorateur Stockage sous Linux requiert l’installation préalable de certaines dépendances. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md?tabs=1804#linux-dependencies) de l’Explorateur Stockage pour plus d’informations.

## <a name="version-170"></a>Version 1.7.0
05/03/2019

### <a name="download-azure-storage-explorer-170"></a>Télécharger l’Explorateur Stockage Azure 1.7.0
- [Explorateur Stockage Azure 1.7.0 pour Windows](https://go.microsoft.com/fwlink/?LinkId=708343)
- [Explorateur Stockage Azure 1.7.0 pour Mac](https://go.microsoft.com/fwlink/?LinkId=708342)
- [Explorateur Stockage Azure 1.7.0 pour Linux](https://go.microsoft.com/fwlink/?LinkId=722418)

### <a name="new"></a>Nouveau

* Vous pouvez maintenant modifier le propriétaire et le groupe propriétaire lors de la gestion des accès pour un conteneur ADLS Gen2, un fichier ou un dossier.
* Sous Windows, la mise à jour à partir de l’Explorateur Stockage au sein du produit est désormais une installation incrémentielle. Cela doit entraîner une expérience de mise à jour plus rapide. Si vous préférez une nouvelle installation, vous pouvez télécharger le [programme d’installation](https://azure.microsoft.com/features/storage-explorer/) vous-même, puis l’installez manuellement. #1089

### <a name="preview-features"></a>Fonctionnalités de préversion

* La connexion par flux de code d’appareil est à présent disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use Device Code Flow Sign-in » (Utiliser la connexion par flux de code d’appareil). Nous encourageons tous les utilisateurs qui ont eu des problèmes avec des fenêtres de connexion vides à essayer cette fonctionnalité, car elle peut s’avérer être une forme plus fiable de connexion. #938
* L’Explorateur Stockage intégré à AzCopy est actuellement disponible en préversion. Pour l’activer, accédez à « Préversion » → « Use AzCopy for Improved Blob Upload and Download » (Utiliser AzCopy pour améliorer le chargement et le téléchargement des objets blob). Les transferts d’objets blob effectués avec AzCopy doivent être plus rapides et plus performants.

### <a name="fixes"></a>Correctifs

* Vous pouvez maintenant choisir le type d’objet blob à charger que lorsque AzCopy est activé. #1111
* Auparavant, si vous aviez activé les sites web statiques pour un compte de stockage ADLS Gen2, puis l’aviez associé à un nom et à une clé, l’Explorateur Stockage n’aurait pas détecté que cet espace de noms hiérarchique avait été activé. Ce problème a été résolu. #1081
* Dans l’éditeur d’objets blob, le tri par jours de rétention restants ou état ne fonctionnait pas. Ce problème a été résolu. #1106
* Après la version 1.5.0, L’Explorateur Stockage n’attendait plus la fin des copies côté serveur avant de signaler la réussite d’un changement de nom ou d’un copier-coller. Ce problème a été résolu. #976
* Lorsque vous utilisiez la fonctionnalité AzCopy expérimentale, la commande copiée après avoir cliqué sur « Copier la commande dans le Presse-papiers » n’était pas toujours exécutable telle quelle. À présent, toutes les commandes nécessaires pour exécuter le transfert manuellement seront copiées. #1079
* Auparavant, les objets blob ADLS Gen2 n’étaient pas accessibles si vous étiez derrière un proxy. Cela était dû à un bogue dans une nouvelle bibliothèque de mise en réseau utilisée par le SDK de stockage. Dans la version 1.7.0, nous avons tenté d’atténuer ce problème, mais il persiste pour certaines personnes. Un correctif complet sera disponible dans une prochaine mise à jour. #1090
* Dans la version 1.7.0, la boîte de dialogue d’enregistrement de fichier mémorise désormais correctement le dernier emplacement où vous avez enregistré un fichier. #16
* Dans le volet des propriétés, le niveau de référence (SKU) d’un compte de stockage était affiché en tant que type du compte. Ce problème a été résolu. #654
* Parfois, il était impossible de résilier le bail d’un objet blob, même si vous aviez correctement entré le nom de l’objet blob. Ce problème a été résolu. #1070

### <a name="known-issues"></a>Problèmes connus

* Lorsque vous utilisez RBAC, l’Explorateur Stockage requiert certaines autorisations de couche de gestion afin d’accéder à vos ressources de stockage. Consultez le [guide de dépannage](./storage/common/storage-explorer-troubleshooting.md) pour plus d’informations.
* La tentative d’accès aux objets Blob Gen2 ADLS derrière un proxy peut échouer.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Voir #537 pour plus d’informations.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Pour plus d’informations, consultez #537.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-162"></a>Version 1.6.2
1/9/2019

### <a name="hotfixes"></a>Correctifs logiciels
* Dans la version 1.6.1, les entités ajoutées aux liste de contrôle d’accès ADLS Gen2 par ID d’objet et qui n’étaient pas des utilisateurs étaient toujours ajoutées en tant que groupes. À présent, seuls les groupes sont ajoutés en tant que groupes. Les entités, telles que les applications d’entreprise et les principaux de service, sont ajoutées en tant qu’utilisateurs. [#1049](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1049)
* Si un compte de stockage Gen2 ADLS n’avait aucun conteneur et était joint avec un nom et une clé, l’Explorateur Stockage ne détectait pas que le compte de stockage était ADLS Gen2. Ce problème a été résolu. [#1048](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1048)
* Dans la version 1.6.0, des conflits pendant la copie et le collage n’entraînaient pas l’ouverture d’une invite de résolution. Au lieu de cela, la copie en conflit échouait simplement. À présent, lors du premier conflit, une invite vous demande comment vous souhaitez que celui-ci soit résolu. [#1014](https://www.github.com/Microsoft/AzureStorageExplorer/issues/1014)
* En raison des limitations d’API, toute validation d’ID d’objet dans la boîte de dialogue Gérer l’accès était désactivée. La validation est maintenant possible uniquement pour les noms d’utilisateur principaux. [#954](https://www.github.com/Microsoft/AzureStorageExplorer/issues/954)
* Dans la boîte de dialogue Gérer l’accès ADLS Gen2, les autorisations d’un groupe ne pouvaient pas être modifiées. Ce problème a été résolu. [#958](https://www.github.com/Microsoft/AzureStorageExplorer/issues/958)
* La prise en charge du chargement par glisser-déplacer a été ajoutée dans l’éditeur ADLS Gen2. [#953](https://www.github.com/Microsoft/AzureStorageExplorer/issues/953)
* La propriété URL dans la boîte de dialogue Propriétés des fichiers et dossiers d’ADLS Gen2 n’avait parfois pas de « / ». Ce problème a été résolu. [#960](https://www.github.com/Microsoft/AzureStorageExplorer/issues/960)
* En cas d’échec des autorisations actuelles pour un conteneur, fichier ou dossier ADLS Gen2, l’erreur est désormais correctement affichée dans le journal d’activité. [#965](https://www.github.com/Microsoft/AzureStorageExplorer/issues/965)
* Le chemin temporaire créé pour ouvrir des fichiers a été raccourci pour réduire le risque que la longueur du chemin soit supérieure à MAX_PATH sur Windows. [#93](https://www.github.com/Microsoft/AzureStorageExplorer/issues/93)
* La boîte de dialogue de connexion apparaît désormais correctement quand aucun utilisateur n’est connecté et qu’aucune ressource n’a été attachée. [#944](https://www.github.com/Microsoft/AzureStorageExplorer/issues/944)
* Dans la version 1.6.0, l’enregistrement des propriétés pour les fichiers et les objets blob non-HNS encodait la valeur de chaque propriété. Cela entraînait l’encodage inutile de valeurs qui contenaient uniquement des caractères ASCII. À présent, les valeurs sont encodées uniquement si elles contiennent des caractères non-ASCII. [#986](https://www.github.com/Microsoft/AzureStorageExplorer/issues/986)
* Le chargement d’un dossier dans un conteneur d’objets blob non-HNS échouait si une SAP était utilisée sans autorisation de lecture. Ce problème a été résolu. [#970](https://www.github.com/Microsoft/AzureStorageExplorer/issues/970)
* L’annulation d’un transfert AzCopy ne fonctionnait pas. Ce problème a été résolu. [#943](https://www.github.com/Microsoft/AzureStorageExplorer/issues/943)
* AzCopy échouait pendant la tentative de téléchargement d’un dossier à partir d’un conteneur d’objets blob ADLS Gen2 si le nom du dossier avait des espaces. Ce problème a été résolu. [#990](https://www.github.com/Microsoft/AzureStorageExplorer/issues/990)
* L’éditeur de CosmosDB avait un problème dans la version 1.6.0. Il est maintenant corrigé. [#950](https://www.github.com/Microsoft/AzureStorageExplorer/issues/950)
        
### <a name="new"></a>Nouveau

* Vous pouvez maintenant utiliser l’Explorateur Stockage pour accéder à vos données Blob via [RBAC](./storage/common/storage-auth-aad-rbac-portal.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). Si vous êtes connecté et que l’Explorateur Stockage ne peut pas récupérer les clés de votre compte de stockage, un jeton OAuth est utilisé pour l’authentification quand vous interagissez avec vos données.
* L’Explorateur Stockage prend désormais en charge les comptes de stockage ADLS Gen2. Quand l’Explorateur Stockage détecte que l’espace de noms hiérarchique est activé pour un compte de stockage, vous voyez « (Préversion d’ADLS Gen2) » à côté du nom de votre compte de stockage. L’Explorateur Stockage peut détecter si l’espace de noms hiérarchique est activé ou non quand vous êtes connecté, ou si vous avez attaché votre compte de stockage avec un nom et une clé. Pour les comptes de stockage ADLS Gen2, vous pouvez utiliser l’Explorateur Stockage pour :
  * Créer et supprimer des conteneurs
  * Gérer les propriétés et les autorisations de conteneur (à gauche)
  * Voir et parcourir les données à l’intérieur des conteneurs
  * Créer des dossiers
  * Charger, télécharger, renommer et supprimer des fichiers et dossiers
  * Gérer les propriétés et les autorisations de fichier et dossier (à droite).

    D’autres fonctionnalités Blob classiques, comme la suppression réversible et les instantanés, ne sont actuellement pas disponibles. La gestion des autorisations est également disponible uniquement quand vous êtes connecté. Par ailleurs, quand vous utilisez un compte de stockage ADLS Gen2, l’Explorateur Stockage utilise AzCopy pour tous les chargements et téléchargements, et utilise par défaut le nom et les informations d’identification de clé pour toutes les opérations, s’ils sont disponibles.
* Suite à de nombreux commentaires des utilisateurs, la résiliation de bail peut à nouveau être utilisée pour résilier les baux de plusieurs objets blob à la fois.

### <a name="known-issues"></a>Problèmes connus

* Pendant le téléchargement à partir d’un compte de stockage ADLS Gen2, si un des fichiers transférés existe déjà, AzCopy se bloque parfois. Ce problème sera résolu dans un prochain correctif logiciel.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Pour plus d’informations, consultez #537.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-161"></a>Version 1.6.1
18/12/2018

### <a name="hotfixes"></a>Correctifs logiciels
* En raison des limitations d’API, toute validation d’ID d’objet dans la boîte de dialogue Gérer l’accès était désactivée. La validation est maintenant possible uniquement pour les noms d’utilisateur principaux. [#954](https://www.github.com/Microsoft/AzureStorageExplorer/issues/954)
* Dans la boîte de dialogue Gérer l’accès ADLS Gen2, les autorisations d’un groupe ne pouvaient pas être modifiées. Ce problème a été résolu. [#958](https://www.github.com/Microsoft/AzureStorageExplorer/issues/958)
* La prise en charge du chargement par glisser-déplacer a été ajoutée dans l’éditeur ADLS Gen2. [#953](https://www.github.com/Microsoft/AzureStorageExplorer/issues/953)
* La propriété URL dans la boîte de dialogue Propriétés des fichiers et dossiers d’ADLS Gen2 n’avait parfois pas de « / ». Ce problème a été résolu. [#960](https://www.github.com/Microsoft/AzureStorageExplorer/issues/960)
* En cas d’échec des autorisations actuelles pour un conteneur, fichier ou dossier ADLS Gen2, l’erreur est désormais correctement affichée dans le journal d’activité. [#965](https://www.github.com/Microsoft/AzureStorageExplorer/issues/965)
* Le chemin temporaire créé pour ouvrir des fichiers a été raccourci pour réduire le risque que la longueur du chemin soit supérieure à MAX_PATH sur Windows. [#93](https://www.github.com/Microsoft/AzureStorageExplorer/issues/93)
* La boîte de dialogue de connexion apparaît désormais correctement quand aucun utilisateur n’est connecté et qu’aucune ressource n’a été attachée. [#944](https://www.github.com/Microsoft/AzureStorageExplorer/issues/944)
* Dans la version 1.6.0, l’enregistrement des propriétés pour les fichiers et les objets blob non-HNS encodait la valeur de chaque propriété. Cela entraînait l’encodage inutile de valeurs qui contenaient uniquement des caractères ASCII. À présent, les valeurs sont encodées uniquement si elles contiennent des caractères non-ASCII. [#986](https://www.github.com/Microsoft/AzureStorageExplorer/issues/986)
* Le chargement d’un dossier dans un conteneur d’objets blob non-HNS échouait si une SAP était utilisée sans autorisation de lecture. Ce problème a été résolu. [#970](https://www.github.com/Microsoft/AzureStorageExplorer/issues/970)
* L’annulation d’un transfert AzCopy ne fonctionnait pas. Ce problème a été résolu. [#943](https://www.github.com/Microsoft/AzureStorageExplorer/issues/943)
* AzCopy échouait pendant la tentative de téléchargement d’un dossier à partir d’un conteneur d’objets blob ADLS Gen2 si le nom du dossier avait des espaces. Ce problème a été résolu. [#990](https://www.github.com/Microsoft/AzureStorageExplorer/issues/990)
* L’éditeur de CosmosDB avait un problème dans la version 1.6.0. Il est maintenant corrigé. [#950](https://www.github.com/Microsoft/AzureStorageExplorer/issues/950)
        
### <a name="new"></a>Nouveau

* Vous pouvez maintenant utiliser l’Explorateur Stockage pour accéder à vos données Blob via [RBAC](./storage/common/storage-auth-aad-rbac-portal.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). Si vous êtes connecté et que l’Explorateur Stockage ne peut pas récupérer les clés de votre compte de stockage, un jeton OAuth est utilisé pour l’authentification quand vous interagissez avec vos données.
* L’Explorateur Stockage prend désormais en charge les comptes de stockage ADLS Gen2. Quand l’Explorateur Stockage détecte que l’espace de noms hiérarchique est activé pour un compte de stockage, vous voyez « (Préversion d’ADLS Gen2) » à côté du nom de votre compte de stockage. L’Explorateur Stockage peut détecter si l’espace de noms hiérarchique est activé ou non quand vous êtes connecté, ou si vous avez attaché votre compte de stockage avec un nom et une clé. Pour les comptes de stockage ADLS Gen2, vous pouvez utiliser l’Explorateur Stockage pour :
  * Créer et supprimer des conteneurs
  * Gérer les propriétés et les autorisations de conteneur (à gauche)
  * Voir et parcourir les données à l’intérieur des conteneurs
  * Créer des dossiers
  * Charger, télécharger, renommer et supprimer des fichiers et dossiers
  * Gérer les propriétés et les autorisations de fichier et dossier (à droite).
    
    D’autres fonctionnalités Blob classiques, comme la suppression réversible et les instantanés, ne sont actuellement pas disponibles. La gestion des autorisations est également disponible uniquement quand vous êtes connecté. Par ailleurs, quand vous utilisez un compte de stockage ADLS Gen2, l’Explorateur Stockage utilise AzCopy pour tous les chargements et téléchargements, et utilise par défaut le nom et les informations d’identification de clé pour toutes les opérations, s’ils sont disponibles.
* Suite à de nombreux commentaires des utilisateurs, la résiliation de bail peut à nouveau être utilisée pour résilier les baux de plusieurs objets blob à la fois.

### <a name="known-issues"></a>Problèmes connus

* Pendant le téléchargement à partir d’un compte de stockage ADLS Gen2, si un des fichiers transférés existe déjà, AzCopy se bloque parfois. Ce problème sera résolu dans un prochain correctif logiciel.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Pour plus d’informations, consultez #537.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-160"></a>Version 1.6.0
5/12/2018

### <a name="new"></a>Nouveau

* Vous pouvez maintenant utiliser l’Explorateur Stockage pour accéder à vos données Blob via [RBAC](./storage/common/storage-auth-aad-rbac-portal.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). Si vous êtes connecté et que l’Explorateur Stockage ne peut pas récupérer les clés de votre compte de stockage, un jeton OAuth est utilisé pour l’authentification quand vous interagissez avec vos données.
* L’Explorateur Stockage prend désormais en charge les comptes de stockage ADLS Gen2. Quand l’Explorateur Stockage détecte que l’espace de noms hiérarchique est activé pour un compte de stockage, vous voyez « (Préversion d’ADLS Gen2) » à côté du nom de votre compte de stockage. L’Explorateur Stockage peut détecter si l’espace de noms hiérarchique est activé ou non quand vous êtes connecté, ou si vous avez attaché votre compte de stockage avec un nom et une clé. Pour les comptes de stockage ADLS Gen2, vous pouvez utiliser l’Explorateur Stockage pour :
  * Créer et supprimer des conteneurs
  * Gérer les propriétés et les autorisations de conteneur (à gauche)
  * Voir et parcourir les données à l’intérieur des conteneurs
  * Créer des dossiers
  * Charger, télécharger, renommer et supprimer des fichiers et dossiers
  * Gérer les propriétés et les autorisations de fichier et dossier (à droite).
    
    D’autres fonctionnalités Blob classiques, comme la suppression réversible et les instantanés, ne sont actuellement pas disponibles. La gestion des autorisations est également disponible uniquement quand vous êtes connecté. Par ailleurs, quand vous utilisez un compte de stockage ADLS Gen2, l’Explorateur Stockage utilise AzCopy pour tous les chargements et téléchargements, et utilise par défaut le nom et les informations d’identification de clé pour toutes les opérations, s’ils sont disponibles.
* Suite à de nombreux commentaires des utilisateurs, la résiliation de bail peut à nouveau être utilisée pour résilier les baux de plusieurs objets blob à la fois.

### <a name="known-issues"></a>Problèmes connus

* Pendant le téléchargement à partir d’un compte de stockage ADLS Gen2, si un des fichiers transférés existe déjà, AzCopy se bloque parfois. Ce problème sera résolu dans un prochain correctif logiciel.
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Pour plus d’informations, consultez #537.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-150"></a>Version 1.5.0
29/10/2018

### <a name="new"></a>Nouveau

* Vous pouvez maintenant utiliser [AzCopy v10 (préversion)](https://github.com/Azure/azure-storage-azcopy) pour charger et télécharger des objets blob. Pour activer cette fonctionnalité, accédez au menu « Expérimental », puis cliquez sur « Utiliser AzCopy pour améliorer le chargement et le téléchargement des objets blob ». Quand cette option est activée, AzCopy sera utilisé dans les scénarios suivants :
   * Chargement de dossiers et de fichiers vers des conteneurs d’objets blob, par le biais de la barre d’outils ou de l’opération glisser-déplacer.
   * Téléchargement de dossiers et de fichiers, par le biais de la barre d’outils ou du menu contextuel.

* De plus, lors de l’utilisation d’AzCopy :
   * Vous pouvez copier la commande AzCopy utilisée pour exécuter le transfert vers le Presse-papiers. Cliquez simplement sur « Copier la commande AzCopy dans le Presse-papiers » dans le journal d’activité.
   * Vous devrez actualiser l’éditeur d’objets blob manuellement après le chargement.
   * Le chargement de fichiers pour ajouter des objets blob n’est pas pris en charge. Les fichiers vhd sont chargés comme des objets blob de pages et tous les autres fichiers sont chargés comme des objets blob de blocs.
   * Les erreurs et les conflits qui se produisent pendant le chargement ou téléchargement ne sont visibles qu’une fois le chargement ou téléchargement terminé.

Pour finir, la prise en charge de l’utilisation d’AzCopy avec Partages de fichiers sera disponible prochainement.
* Explorateur Stockage utilise désormais Electron version 2.0.11.
* La résiliation de bail peut désormais être effectuée sur un seul objet blob à la fois. De plus, vous devez entrer le nom de l’objet blob dont vous résiliez le bail. Ce changement a été effectué pour réduire le risque de résiliation accidentelle de bail, en particulier pour les machines virtuelles. #394
* Si vous rencontrez toujours des problèmes de connexion, vous pouvez maintenant essayer de réinitialiser l’authentification. Accédez au menu « Aide », puis cliquez sur « Réinitialiser » pour accéder à cette fonctionnalité. #419

### <a name="fix"></a>Correction

* Suite à de nombreux commentaires des utilisateurs, le nœud d’émulateur par défaut a été réactivé. Vous pouvez toujours ajouter des connexions d’émulateur supplémentaires par le biais de la boîte de dialogue Se connecter, mais si votre émulateur est configuré pour utiliser les ports par défaut vous pouvez également utiliser le nœud « Émulateur * Ports par défaut » sous « Local & attaché/Comptes de stockage ». #669
* Explorateur Stockage ne vous permet plus de définir des valeurs de métadonnées d’objets blob contenant des espaces blancs de début ou de fin. #760
* Le bouton « Se connecter » était toujours activé sur les mêmes pages de la boîte de dialogue Se connecter. Il est désormais désactivé le cas échéant. #761
* Accès rapide ne génère plus d’erreur dans la console quand aucun élément d’Accès rapide n’a été ajouté.

### <a name="known-issues"></a>Problèmes connus

* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Pour plus d’informations, consultez #537.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, commentez ce problème.
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron. Pour contourner ce problème pendant le chargement ou le téléchargement dans un conteneur d’objets blob, vous pouvez utiliser la fonctionnalité expérimentale AzCopy.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes. Toute tentative d’utilisation de ces fonctionnalités lors de l’utilisation de ressources Azure Stack peut provoquer des erreurs inattendues.
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```


## <a name="version-144"></a>Version 1.4.4
15/10/2018

### <a name="hotfixes"></a>Correctifs logiciels
* La version de l’API Gestion des ressources Azure a été restaurée pour débloquer les utilisateurs Azure US Government. [#696](https://github.com/Microsoft/AzureStorageExplorer/issues/696)
* Les boucles de progression de chargement utilisent désormais des animations CSS pour réduire la quantité de GPU utilisée par l’Explorateur Stockage. [#653](https://github.com/Microsoft/AzureStorageExplorer/issues/653)

### <a name="new"></a>Nouveau
* Les attachements de ressources externes, comme pour les émulateurs et les connexions SAS, ont été considérablement améliorés. Vous pouvez désormais :
   * Personnaliser le nom complet de la ressource que vous attachez. [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * Effectuer un attachement à plusieurs émulateurs locaux à l’aide de différents ports. [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * Ajouter des ressources attachées à l’Accès rapide. [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* L’Explorateur Stockage prend maintenant en charge la suppression réversible. Vous pouvez :
   * Configurer une stratégie de suppression réversible en cliquant avec le bouton droit sur le nœud de Conteneurs d’objets blob pour votre compte de stockage.
   * Afficher les objets blob supprimés de manière réversible dans l’éditeur d’objets blob en sélectionnant « Objets blob actifs et supprimés » dans la liste déroulante en regard de la barre de navigation.
   * Annuler la suppression d’objets blob supprimés de manière réversible.

### <a name="fixes"></a>Correctifs
* L’action « Configurer les paramètres CORS » n’est plus disponible sur les comptes de stockage Premium, car ces comptes ne prennent pas en charge le mécanisme CORS. [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* Il existe désormais une propriété Signature d’accès partagé pour les services SAP associés. [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* L’action « Définir le niveau accès par défaut » est désormais disponible pour les comptes de stockage d’objets blob et GPv2 qui ont été épinglés à l’Accès rapide. [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* Parfois, l’Explorateur Stockage ne parvenait pas à afficher les comptes de stockage classiques. [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>Problèmes connus
* Lors vous utilisez des émulateurs, tels que l’émulateur de stockage Azure ou Azurite, vous devez les faire écouter des connexions sur leurs ports par défaut. Autrement, l’Explorateur Stockage n’est pas en mesure de se connecter à ces émulateurs.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, veuillez apporter vos commentaires sur [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/97).
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-143"></a>Version 1.4.3
11/10/2018

### <a name="hotfixes"></a>Correctifs logiciels
* La version de l’API Gestion des ressources Azure a été restaurée pour débloquer les utilisateurs Azure US Government. [#696](https://github.com/Microsoft/AzureStorageExplorer/issues/696)
* Les boucles de progression de chargement utilisent désormais des animations CSS pour réduire la quantité de GPU utilisée par l’Explorateur Stockage. [#653](https://github.com/Microsoft/AzureStorageExplorer/issues/653)

### <a name="new"></a>Nouveau
* Les attachements de ressources externes, comme pour les émulateurs et les connexions SAS, ont été considérablement améliorés. Vous pouvez désormais :
   * Personnaliser le nom complet de la ressource que vous attachez. [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * Effectuer un attachement à plusieurs émulateurs locaux à l’aide de différents ports. [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * Ajouter des ressources attachées à l’Accès rapide. [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* L’Explorateur Stockage prend maintenant en charge la suppression réversible. Vous pouvez :
   * Configurer une stratégie de suppression réversible en cliquant avec le bouton droit sur le nœud de Conteneurs d’objets blob pour votre compte de stockage.
   * Afficher les objets blob supprimés de manière réversible dans l’éditeur d’objets blob en sélectionnant « Objets blob actifs et supprimés » dans la liste déroulante en regard de la barre de navigation.
   * Annuler la suppression d’objets blob supprimés de manière réversible.

### <a name="fixes"></a>Correctifs
* L’action « Configurer les paramètres CORS » n’est plus disponible sur les comptes de stockage Premium, car ces comptes ne prennent pas en charge le mécanisme CORS. [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* Il existe désormais une propriété Signature d’accès partagé pour les services SAP associés. [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* L’action « Définir le niveau accès par défaut » est désormais disponible pour les comptes de stockage d’objets blob et GPv2 qui ont été épinglés à l’Accès rapide. [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* Parfois, l’Explorateur Stockage ne parvenait pas à afficher les comptes de stockage classiques. [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>Problèmes connus
* Lors vous utilisez des émulateurs, tels que l’émulateur de stockage Azure ou Azurite, vous devez les faire écouter des connexions sur leurs ports par défaut. Autrement, l’Explorateur Stockage n’est pas en mesure de se connecter à ces émulateurs.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, veuillez apporter vos commentaires sur [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/97).
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-142"></a>Version 1.4.2
24/09/2018

### <a name="hotfixes"></a>Correctifs logiciels
* Mettez à jour l’API Gestion des ressources Azure avec la version 2018-07-01 pour ajouter la prise en charge des nouveaux types de comptes de stockage Azure. [#652](https://github.com/Microsoft/AzureStorageExplorer/issues/652)

### <a name="new"></a>Nouveau
* Les attachements de ressources externes, comme pour les émulateurs et les connexions SAS, ont été considérablement améliorés. Vous pouvez désormais :
   * Personnaliser le nom complet de la ressource que vous attachez. [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * Effectuer un attachement à plusieurs émulateurs locaux à l’aide de différents ports. [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * Ajouter des ressources attachées à l’Accès rapide. [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* L’Explorateur Stockage prend maintenant en charge la suppression réversible. Vous pouvez :
   * Configurer une stratégie de suppression réversible en cliquant avec le bouton droit sur le nœud de Conteneurs d’objets blob pour votre compte de stockage.
   * Afficher les objets blob supprimés de manière réversible dans l’éditeur d’objets blob en sélectionnant « Objets blob actifs et supprimés » dans la liste déroulante en regard de la barre de navigation.
   * Annuler la suppression d’objets blob supprimés de manière réversible.

### <a name="fixes"></a>Correctifs
* L’action « Configurer les paramètres CORS » n’est plus disponible sur les comptes de stockage Premium, car ces comptes ne prennent pas en charge le mécanisme CORS. [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* Il existe désormais une propriété Signature d’accès partagé pour les services SAP associés. [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* L’action « Définir le niveau accès par défaut » est désormais disponible pour les comptes de stockage d’objets blob et GPv2 qui ont été épinglés à l’Accès rapide. [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* Parfois, l’Explorateur Stockage ne parvenait pas à afficher les comptes de stockage classiques. [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>Problèmes connus
* Lors vous utilisez des émulateurs, tels que l’émulateur de stockage Azure ou Azurite, vous devez les faire écouter des connexions sur leurs ports par défaut. Autrement, l’Explorateur Stockage n’est pas en mesure de se connecter à ces émulateurs.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, veuillez apporter vos commentaires sur [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/97).
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-141"></a>Version 1.4.1
28/08/2018

### <a name="hotfixes"></a>Correctifs logiciels
* Au premier lancement, l’Explorateur Stockage ne pouvait pas générer la clé utilisée pour chiffrer les données sensibles. Cela entraînait des problèmes lors de l’utilisation de l’Accès rapide et de l’attachement de ressources. [#535](https://github.com/Microsoft/AzureStorageExplorer/issues/535)
* Si votre compte n’avait pas besoin de l’authentification MFA pour son locataire de base, mais qu’il en avait besoin pour certains autres locataires, l’Explorateur Stockage ne pouvait pas répertorier les abonnements. Désormais, une fois que la connexion a été effectuée avec ce type de compte, l’Explorateur Stockage vous demandera d’entrer à nouveau vos informations d’identification et d’effectuer un authentification MFA. [#74](https://github.com/Microsoft/AzureStorageExplorer/issues/74)
* L’Explorateur Stockage ne pouvait pas attacher des ressources à partir d’Azure Allemagne et d’Azure US Government. [#572](https://github.com/Microsoft/AzureStorageExplorer/issues/572)
* Si vous vous connectiez à deux comptes ayant la même adresse e-mail, l’Explorateur Stockage ne parvenait parfois pas à afficher vos ressources dans l’arborescence. [#580](https://github.com/Microsoft/AzureStorageExplorer/issues/580)
* Sur les ordinateurs Windows plus lents, l’affichage de l’écran de démarrage prendrait parfois beaucoup de temps. [#586](https://github.com/Microsoft/AzureStorageExplorer/issues/586)
* La boîte de dialogue de connexion s’affichait même s’il existait des comptes joints ou des services associés. [#588](https://github.com/Microsoft/AzureStorageExplorer/issues/588)

### <a name="new"></a>Nouveau
* Les attachements de ressources externes, comme pour les émulateurs et les connexions SAS, ont été considérablement améliorés. Vous pouvez désormais :
   * Personnaliser le nom complet de la ressource que vous attachez. [#31](https://github.com/Microsoft/AzureStorageExplorer/issues/31)
   * Effectuer un attachement à plusieurs émulateurs locaux à l’aide de différents ports. [#193](https://github.com/Microsoft/AzureStorageExplorer/issues/193)
   * Ajouter des ressources attachées à l’Accès rapide. [#392](https://github.com/Microsoft/AzureStorageExplorer/issues/392)
* L’Explorateur Stockage prend maintenant en charge la suppression réversible. Vous pouvez :
   * Configurer une stratégie de suppression réversible en cliquant avec le bouton droit sur le nœud de Conteneurs d’objets blob pour votre compte de stockage.
   * Afficher les objets blob supprimés de manière réversible dans l’éditeur d’objets blob en sélectionnant « Objets blob actifs et supprimés » dans la liste déroulante en regard de la barre de navigation.
   * Annuler la suppression d’objets blob supprimés de manière réversible.

### <a name="fixes"></a>Correctifs
* L’action « Configurer les paramètres CORS » n’est plus disponible sur les comptes de stockage Premium, car ces comptes ne prennent pas en charge le mécanisme CORS. [#142](https://github.com/Microsoft/AzureStorageExplorer/issues/142)
* Il existe désormais une propriété Signature d’accès partagé pour les services SAP associés. [#184](https://github.com/Microsoft/AzureStorageExplorer/issues/184)
* L’action « Définir le niveau accès par défaut » est désormais disponible pour les comptes de stockage d’objets blob et GPv2 qui ont été épinglés à l’Accès rapide. [#229](https://github.com/Microsoft/AzureStorageExplorer/issues/229)
* Parfois, l’Explorateur Stockage ne parvenait pas à afficher les comptes de stockage classiques. [#323](https://github.com/Microsoft/AzureStorageExplorer/issues/323)

### <a name="known-issues"></a>Problèmes connus
* Lors vous utilisez des émulateurs, tels que l’émulateur de stockage Azure ou Azurite, vous devez les faire écouter des connexions sur leurs ports par défaut. Autrement, l’Explorateur Stockage n’est pas en mesure de se connecter à ces émulateurs.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, veuillez apporter vos commentaires sur [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/97).
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-130"></a>Version 1.3.0
09/07/2018

### <a name="new"></a>Nouveau
* L’accès aux conteneurs $web utilisés par des sites web statiques est désormais pris en charge. Cela vous permet de charger et gérer facilement les fichiers et dossiers utilisés par votre site web. [#223](https://github.com/Microsoft/AzureStorageExplorer/issues/223)
* La barre d’applications sur macOS a été réorganisée. Les modifications incluent un menu Fichier, certaines modifications de touches de raccourci, et plusieurs nouvelles commandes dans le menu de l’application. [#99](https://github.com/Microsoft/AzureStorageExplorer/issues/99)
* Le point de terminaison d’autorité pour la connexion à Azure - Gouvernement des États-Unis a été remplacé par https://login.microsoftonline.us/.
* Accessibilité : quand un lecteur d’écran est actif, la navigation au clavier fonctionne désormais avec les tableaux utilisés pour afficher des éléments sur le côté droit. Vous pouvez utiliser les touches de direction pour naviguer dans les lignes et colonnes, la touche Entrée pour appeler des actions par défaut, la touche du menu contextuel pour ouvrir celui-ci pour un élément, et les touches Maj ou Ctrl pour opérer une sélection multiple. [#103](https://github.com/Microsoft/AzureStorageExplorer/issues/103)

### <a name="fixes"></a>Correctifs
*  Sur certains ordinateurs, le démarrage des processus enfants prenait beaucoup de temps. Lorsque cela se produisait, le message d’erreur « child process failed to start in a timely manner » (Échec du démarrage en temps opportun du processus enfant) s’affichait. Le temps alloué au démarrage d’un processus enfant a été augmenté de 20 à 90 secondes. Si vous continuez à rencontrer ce problème, veuillez commenter le problème GitHub lié. [#281](https://github.com/Microsoft/AzureStorageExplorer/issues/281)
* Lorsque vous utilisiez une signature d’accès partagé qui n’avait pas d’autorisation de lecture, il n’était pas possible de charger un objet blob volumineux. La logique de chargement a été modifiée pour fonctionner dans ce scénario. [#305](https://github.com/Microsoft/AzureStorageExplorer/issues/305)
* La définition du niveau d’accès public pour un conteneur avait pour effet de supprimer toutes les stratégies d’accès, et inversement. Désormais, le niveau d’accès public et les stratégies d’accès public sont conservés lors de la définition de l’une des deux options. [#197](https://github.com/Microsoft/AzureStorageExplorer/issues/197)
* « AccessTierChangeTime » était tronqué dans la boîte de dialogue de Propriétés. Ce problème a été résolu. [#145](https://github.com/Microsoft/AzureStorageExplorer/issues/145)
* Le préfixe « Explorateur Stockage Microsoft Azure - » était manquant dans la boîte de dialogue Créer un nouvel annuaire. Ce problème a été résolu. [#299](https://github.com/Microsoft/AzureStorageExplorer/issues/299)
* Accessibilité : la navigation à l’aide de VoiceOver dans la boîte de dialogue Ajouter une entité était difficile. Des améliorations ont été apportées. [#206](https://github.com/Microsoft/AzureStorageExplorer/issues/206)
* Accessibilité : la couleur d’arrière-plan du bouton réduire/développer dans les volet Actions et Propriétés était incompatible avec les contrôles d’interface utilisateur similaires dans le thème Contraste noir élevé. La couleur a été modifiée. [#123](https://github.com/Microsoft/AzureStorageExplorer/issues/123)
* Accessibilité : dans le thème Contraste noir élevé, le focus pour le bouton « X » dans la boîte de dialogue Propriétés n’était pas visible. Ce problème a été résolu. [#243](https://github.com/Microsoft/AzureStorageExplorer/issues/243)
* Accessibilité : plusieurs valeurs aria étaient absentes des onglets Actions et Propriétés, ce qui altérait l’expérience de lecteur d’écran. Les valeurs aria manquantes ont été ajoutées. [#316](https://github.com/Microsoft/AzureStorageExplorer/issues/316)
* Accessibilité : les nœuds d’arborescence réduits sur le côté gauche n’avaient pas de valeur développée d’aria false. Ce problème a été résolu. [#352](https://github.com/Microsoft/AzureStorageExplorer/issues/352)

### <a name="known-issues"></a>Problèmes connus
* Le détachement d’une ressource attachée par le biais d’un URI SAS, comme un conteneur d’objets blob, peut provoquer une erreur qui empêche les autres attachements de s’afficher correctement. Pour contourner ce problème, actualisez simplement le nœud du groupe. Pour plus d’informations, consultez la rubrique relative à [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/537).
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, veuillez apporter vos commentaires sur [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/97).
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Azure Stack ne prend pas en charge les fonctionnalités suivantes, et toute tentative de les utiliser lors de l’utilisation d’Azure Stack peut entraîner des erreurs inattendues :
   * Partages de fichiers
   * Niveaux d’accès
   * Suppression réversible
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-120"></a>Version 1.2.0
12/06/2018

### <a name="new"></a>Nouveau
* Si l’Explorateur Stockage ne parvient pas à charger uniquement les abonnements d’un sous-ensemble de vos locataires, tous les abonnements correctement chargés sont affichés avec un message d’erreur particulier pour les locataires qui ont échoué. [#159](https://github.com/Microsoft/AzureStorageExplorer/issues/159)
* Sur Windows, lorsqu’une mise à jour est disponible, vous pouvez désormais choisir de « Update on Close » (Mettre à jour lors de la fermeture). Lorsque cette option est sélectionnée, le programme d’installation de la mise à jour s’exécute après la fermeture de l’Explorateur Stockage. [#21](https://github.com/Microsoft/AzureStorageExplorer/issues/21)
* L’option Restaurer l’instantané est ajoutée au menu contextuel de l’éditeur de partage de fichiers lors de l’affichage d’un instantané de partage de fichiers.[#131](https://github.com/Microsoft/AzureStorageExplorer/issues/131)
* Le bouton Effacer la file d’attente est désormais toujours activé.[#135](https://github.com/Microsoft/AzureStorageExplorer/issues/135)
* La prise en charge pour la connexion à ADFS d’Azure Stack a été réactivée. Azure Stack version 1804 ou supérieure est nécessaire. [#150](https://github.com/Microsoft/AzureStorageExplorer/issues/150)

### <a name="fixes"></a>Correctifs
* Si vous affichiez les instantanés d’un partage de fichiers dont le nom était le préfixe d’un autre partage de fichiers dans le même compte de stockage, les captures instantanées de l’autre partage de fichiers étaient également répertoriées. Ce problème est à présent résolu. [#255](https://github.com/Microsoft/AzureStorageExplorer/issues/255)
* Lorsqu’elle était attachée par le biais de SAS, la restauration d’un fichier à partir d’une capture instantanée de partage de fichiers aboutissait à une erreur. Ce problème est à présent résolu. [#211](https://github.com/Microsoft/AzureStorageExplorer/issues/211)
* Lors de l’affichage d’instantanés pour un objet blob, l’action Promouvoir l’instantané était activée lorsque l’objet blob de base et un seul instantané étaient sélectionnés. L’action n’est désormais activée que si un seul instantané est sélectionné. [#230](https://github.com/Microsoft/AzureStorageExplorer/issues/230)
* Si une seule tâche (par exemple, un téléchargement d’objet blob) était démarrée et échouait, une nouvelle tentative d’exécution de la tâche ne se lançait pas automatiquement à moins que vous ne démarriez une autre tâche du même type. Une nouvelle tentative d’exécution de tâche doit désormais s’effectuer automatiquement, peu importe le nombre de tâches que vous avez placées dans la file d’attente.
* Les éditeurs ouverts pour les conteneurs d’objets blob récemment créés dans les comptes Stockage Blob et GPV2 ne disposaient pas de colonne Niveau d’accès. Ce problème est à présent résolu. [#109](https://github.com/Microsoft/AzureStorageExplorer/issues/109)
* Parfois, il manquait une colonne Niveau d’accès lorsqu’un compte Stockage ou un conteneur d’objets blob était attaché par l’intermédiaire de SAS. La colonne est désormais toujours affichée, mais elle comporte une valeur vide si aucun Niveau d’accès n’est défini. [#160](https://github.com/Microsoft/AzureStorageExplorer/issues/160)
* La définition du niveau d’accès d’un objet blob de blocs récemment chargé était désactivée. Ce problème est à présent résolu. [#171](https://github.com/Microsoft/AzureStorageExplorer/issues/171)
* Si le bouton « Conserver l’onglet ouvert » était appelé à l’aide du clavier, le focus clavier était perdu. À présent, le focus se positionne sur l’onglet qui est resté ouvert. [#163](https://github.com/Microsoft/AzureStorageExplorer/issues/163)
* Pour une requête dans le Générateur de requêtes, aucune description utilisable de l’opérateur actif n’était donnée par VoiceOver. Elle est maintenant plus descriptive. [#207](https://github.com/Microsoft/AzureStorageExplorer/issues/207)
* Les liens de pagination des différents éditeurs n’étaient pas descriptifs. Ils ont été modifiés pour être plus précis. [#205](https://github.com/Microsoft/AzureStorageExplorer/issues/205)
* Dans la boîte de dialogue Ajouter une entité, VoiceOver n’annonçait pas à quelle colonne l’élément d’entrée était destiné. Le nom de la colonne active est désormais inclus dans la description de l’élément. [#206](https://github.com/Microsoft/AzureStorageExplorer/issues/206)
* Les boutons radio et les cases à cocher n’avaient pas de bordure visible lorsqu’ils étaient actifs. Ce problème est à présent résolu. [#237](https://github.com/Microsoft/AzureStorageExplorer/issues/237)

### <a name="known-issues"></a>Problèmes connus
* Lors vous utilisez des émulateurs, tels que l’émulateur de stockage Azure ou Azurite, vous devez les faire écouter des connexions sur leurs ports par défaut. Autrement, l’Explorateur Stockage n’est pas en mesure de se connecter à ces émulateurs.
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, veuillez apporter vos commentaires sur [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/97).
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-110"></a>Version 1.1.0
09/05/2018

### <a name="new"></a>Nouveau
* L’Explorateur Stockage prend maintenant en charge l’utilisation d’Azurite. Remarque : La connexion à Azurite est codée en dur pour les points de terminaison de développement par défaut.
* L’Explorateur Stockage prend maintenant en charge les niveaux d’accès pour les objets blob uniquement et les comptes de stockage GPV2. Explorez plus en détail les niveaux d’accès [ici](./storage/blobs/storage-blob-storage-tiers.md).
* Une heure de début n’est plus nécessaire lors de la génération d’une signature SAP.

### <a name="fixes"></a>Correctifs
* La récupération d’abonnements pour les comptes US Government ne fonctionnait plus. Ce problème est à présent résolu. [#61](https://github.com/Microsoft/AzureStorageExplorer/issues/61)
* Le délai d’expiration des stratégies d’accès n’était pas correctement enregistré. Ce problème est à présent résolu. [#50](https://github.com/Microsoft/AzureStorageExplorer/issues/50)
* Quand une URL SAP était générée pour un élément de conteneur, le nom de l’élément n’était pas ajouté à l’URL. Ce problème est à présent résolu. [#44](https://github.com/Microsoft/AzureStorageExplorer/issues/44)
* Lors de la création d’une signature SAP, les délais d’expiration situés dans le passé étaient parfois la valeur par défaut. C’est parce que l’Explorateur Stockage utilisait la dernière heure de début et d’expiration comme valeurs par défaut. Maintenant, chaque fois que vous ouvrez la boîte de dialogue SAP, un nouvel ensemble de valeurs par défaut est généré. [#35](https://github.com/Microsoft/AzureStorageExplorer/issues/35)
* Lors d’une copie entre comptes de stockage, une signature SAP de 24 heures était générée. Si la copie durait plus de 24 heures, elle échouait. Nous avons prolongé la signature SAP de 1 semaine afin de réduire le risque d’un échec de copie à cause d’une signature SAP expirée. [#62](https://github.com/Microsoft/AzureStorageExplorer/issues/62)
* Pour certaines activités, les clics sur « Annuler » ne fonctionnaient pas toujours. Ce problème est à présent résolu. [#125](https://github.com/Microsoft/AzureStorageExplorer/issues/125)
* Pour certaines activités, la vitesse de transfert était incorrecte. Ce problème est à présent résolu. [#124](https://github.com/Microsoft/AzureStorageExplorer/issues/124)
* Dans le menu Affichage, « Previous » était mal orthographié. Il est maintenant correctement orthographié. [#71](https://github.com/Microsoft/AzureStorageExplorer/issues/71)
* La page finale de Windows Installer avait un bouton « Suivant ». Il a été remplacé par un bouton « Terminer ». [#70](https://github.com/Microsoft/AzureStorageExplorer/issues/70)
* Le focus de la touche TAB n’était pas visible pour les boutons des boîtes de dialogue lors de l’utilisation du thème HC Black. Il est maintenant visible.[#64](https://github.com/Microsoft/AzureStorageExplorer/issues/64)
* La casse de « Résolution automatique » pour les actions du journal d’activité était incorrecte. Elle est maintenant correcte. [#51](https://github.com/Microsoft/AzureStorageExplorer/issues/51)
* Lors de la suppression d’une entité d’une table, la boîte de dialogue vous demandant confirmation affichait une icône d’erreur. La boîte de dialogue utilise maintenant une icône d’avertissement. [#148](https://github.com/Microsoft/AzureStorageExplorer/issues/148)

### <a name="known-issues"></a>Problèmes connus
* Si vous utilisez Visual Studio pour Mac et que vous avez créé une configuration AAD personnalisée, vous n’avez peut-être pas pu vous connecter. Pour contourner ce problème, supprimez le contenu de ~/.IdentityService/AadConfigurations. Si vous êtes toujours bloqué, veuillez apporter vos commentaires sur [ce problème](https://github.com/Microsoft/AzureStorageExplorer/issues/97).
* Azurite n’a pas encore totalement implémenté toutes les API de stockage. C’est pourquoi vous risquez de rencontrer des erreurs ou un comportement inattendus quand vous utilisez Azurite pour le stockage de développement.
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Le chargement à partir de votre dossier OneDrive ne fonctionne pas en raison d’un bogue dans NodeJS. Le bogue a été résolu, mais pas encore intégré à Electron.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```


## <a name="version-100"></a>Version 1.0.0
16/04/2018

### <a name="new"></a>Nouveau
* Authentification améliorée qui permet à l’Explorateur Stockage d’utiliser le même magasin de comptes que Visual Studio 2017. Pour utiliser cette fonctionnalité, vous devez vous reconnecter à vos comptes et redéfinir vos abonnements filtrés.
* Pour les comptes Azure Stack soutenus par AAD, l’Explorateur Stockage récupère à présent les abonnements Azure Stack lorsque l’option Target Azure Stack (Cibler Azure Stack) est activée. Vous n’avez plus besoin de créer un environnement de connexion personnalisée.
* Plusieurs raccourcis ont été ajoutés pour accélérer la navigation. Il s’agit notamment de l’activation/de la désactivation de divers panneaux et du basculement d’un éditeur à l’autre. Pour plus d’informations, consultez le menu Affichage.
* Les commentaires sur l’Explorateur Stockage sont maintenant actifs sur GitHub. Vous pouvez accéder à notre page Problèmes en cliquant sur le bouton Commentaires dans la partie inférieure gauche ou en accédant à [https://github.com/Microsoft/AzureStorageExplorer/issues](https://github.com/Microsoft/AzureStorageExplorer/issues). N’hésitez pas à effectuer des suggestions, à signaler des problèmes, à poser des questions ou à laisser toute autre forme de commentaires.
* Si vous rencontrez des problèmes de certificat TLS/SSL et que vous ne parvenez pas à trouver le certificat qui pose problème, vous pouvez maintenant lancer l’Explorateur Stockage à partir de la ligne de commande avec l’indicateur `--ignore-certificate-errors`. Une fois lancé avec cet indicateur, l’Explorateur Stockage ignore les erreurs de certificat TLS/SSL.
* Il existe à présent une option Télécharger dans le menu contextuel pour les éléments blob et de fichier.
* Prise en charge améliorée de l’accessibilité et des lecteurs d’écrans. Si vous vous appuyez sur des fonctionnalités d’accessibilité, consultez notre [documentation à ce sujet](./vs-azure-tools-storage-explorer-accessibility.md) pour plus d’informations.
* L’Explorateur Stockage utilise à présent Electron 1.8.3

### <a name="breaking-changes"></a>Dernières modifications
* L’Explorateur Stockage a basculé vers une nouvelle bibliothèque d’authentification. Dans le cadre du basculement vers la bibliothèque, vous devez vous reconnecter à vos comptes et redéfinir vos abonnements filtrés.
* La méthode utilisée pour chiffrer les données sensibles a changé. Pour cette raison, il se peut que certains de vos éléments Accès rapide doivent être rajoutés et/ou que certaines de vos ressources doivent être rattachées.

### <a name="fixes"></a>Correctifs
* Pour certains utilisateurs, un message d’erreur « Résolution impossible » interrompait les opérations de chargement ou téléchargement de blobs effectuées par certains utilisateurs. Ce problème est à présent résolu.
* Si des informations d’identification étaient demandées lors de l’utilisation d’un lien direct, le fait de cliquer sur l’invite « Se connecter » ouvrait une boîte de dialogue vide. Ce problème est à présent résolu.
* Sur Linux, si l’Explorateur Stockage ne peut pas démarrer en raison d’un arrêt du processus GPU, vous êtes à présent informé de l’incident, vous recevez l’instruction d’utiliser le commutateur --disable-gpu et l’Explorateur Stockage redémarre automatiquement avec le commutateur activé.
* Les stratégies d’accès non valides étaient difficiles à identifier dans la boîte de dialogue Stratégies d’accès. Les ID de stratégie d’accès non valides sont maintenant indiqués en rouge pour plus de visibilité.
* Le journal d’activité avait parfois de grandes zones avec des espaces vides entre les différentes parties d’une activité. Ce problème est à présent résolu.
* Dans l’éditeur de requête de table, si vous laissiez une clause timestamp dans un état non valide puis que vous tentiez de modifier une autre clause, l’éditeur se figeait. L’éditeur restaure maintenant la clause timestamp sur son dernier état valide lorsqu’une modification dans une autre clause est détectée.
* Si vous vous interrompiez lors de la saisie de votre requête de recherche dans l’arborescence, la recherche commençait et le focus disparaissait de la zone de texte. Maintenant, vous devez démarrer explicitement la recherche en appuyant sur la touche « Entrée » ou en cliquant sur le bouton de lancement de la recherche.
* La commande Get Shared Access Signature (Obtenir une signature d’accès partagé) était parfois désactivée lors d’un clic avec le bouton droit sur un fichier situé dans un partage de fichiers. Ce problème est à présent résolu.
* Si le nœud d’arbre de ressource avec le focus était filtré lors de la recherche, vous ne pouviez pas accéder via la touche tabulation à l’arborescence de la ressource et vous deviez utiliser les touches de direction pour parcourir l’arborescence de la ressource. Maintenant, si le nœud d’arbre de ressource ayant le focus est masqué, le premier nœud dans l’arborescence de la ressource a automatiquement le focus.
* Un séparateur supplémentaire était parfois visible dans la barre d’outils de l’éditeur. Ce problème est à présent résolu.
* La zone de texte de barre de navigation dépassait parfois. Ce problème est à présent résolu.
* Les éditeurs d’objets blob et de partage de fichiers étaient parfois constamment actualisés lors du chargement simultané de nombreux fichiers. Ce problème est à présent résolu.
* La fonctionnalité de statistiques des dossiers n’avait aucune utilité dans la vue de gestion des instantanés du partage de fichiers. Elle est maintenant désactivée.
* Sur Linux, le menu Fichier n’apparaissait pas. Ce problème est à présent résolu.
* Lors du chargement d’un dossier vers un partage de fichiers, par défaut, seul le contenu du dossier était chargé. Le comportement par défaut est maintenant de charger le contenu du dossier dans le dossier correspondant dans le partage de fichiers.
* L’ordre des boutons de plusieurs boîtes de dialogue était inversé. Ce problème est à présent résolu.
* Divers correctifs liés à la sécurité.

### <a name="known-issues"></a>Problèmes connus
* Dans de rares cas, le focus de l’arborescence peut être bloqué sur un accès rapide. Pour débloquer le focus, vous pouvez tout actualiser.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Les utilisateurs Linux doivent installer [.NET Core 2.0](https://dotnet.microsoft.com/download/dotnet-core/2.0).
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-096"></a>Version 0.9.6
28/02/2018

### <a name="fixes"></a>Correctifs
* Un problème a empêché de répertorier les fichiers/objets blob attendus dans l’éditeur. Ce problème est à présent résolu.
* Un problème a provoqué le basculement entre les vues des captures instantanées, et l’affichage incorrect des éléments. Ce problème est à présent résolu.

### <a name="known-issues"></a>Problèmes connus
* L’explorateur de stockage ne prend pas en charge les comptes ADFS.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite [ici](https://github.com/Azure/azure-storage-node/issues/317).
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Le panneau des paramètres de compte peut indiquer qu’il vous faut entrer à nouveau vos informations d’identification pour filtrer les abonnements.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-095"></a>Version 0.9.5
02/06/2018

### <a name="new"></a>Nouveau

* Prise en charge des instantanés de partages de fichiers :
    * Créez et gérez les instantanés pour les partages de fichiers.
    * Basculez facilement entre les instantanés de vos partages de fichiers lorsque vous les explorez.
    * Restaurez les versions précédentes de vos fichiers.
* Prise en charge d’Azure Data Lake Store (préversion) :
    * Connectez-vous à vos ressources ADLS sur plusieurs comptes.
    * Connectez-vous à et partagez des ressources ADLS avec des URI ADL.
    * Effectuez des opérations de fichier/dossier de base de manière récursive.
    * Épingles des dossiers individuels dans Accès rapide.
    * Affichez les statistiques des dossiers.

### <a name="fixes"></a>Correctifs
* Meilleures améliorations de démarrage.
* Correction de différents bogues.

### <a name="known-issues"></a>Problèmes connus
* L’explorateur de stockage ne prend pas en charge les comptes ADFS.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Le panneau des paramètres de compte peut indiquer qu’il vous faut entrer à nouveau vos informations d’identification pour filtrer les abonnements.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer.exe --disable-gpu
    ```

* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-094-and-093"></a>Versions 0.9.4 et 0.9.3
21/01/2018

### <a name="new"></a>Nouveau
* La fenêtre Explorateur Stockage existante est réutilisée dans les cas suivants :
    * À l’ouverture de liens directs générés dans l’Explorateur Stockage.
    * À l’ouverture de l’Explorateur Stockage à partir du portail.
    * À l’ouverture de l’Explorateur Stockage à partir de l’extension Stockage Azure pour Visual Studio Code (bientôt disponible).
* L’ouverture d’une nouvelle fenêtre Explorateur Stockage est maintenant possible directement à partir de l’Explorateur Stockage.
    * Pour Windows, une nouvelle option « Nouvelle fenêtre » est disponible sous le menu Fichier et dans le menu contextuel de la barre des tâches.
    * Pour Mac, une nouvelle option « Nouvelle fenêtre » est disponible sous le menu App.

### <a name="fixes"></a>Correctifs
* Correction d’un problème de sécurité. Veuillez mettre à niveau vers 0.9.4 dès que possible.
* Les anciennes activités n’étaient pas correctement nettoyées. Ce problème impactait les performances des travaux de longue durée. Ces activités sont maintenant correctement nettoyées.
* Les actions impliquant beaucoup de fichiers et de répertoires entraînaient parfois le gel de l’Explorateur Stockage. Les demandes de partages de fichiers à Azure sont désormais limitées pour restreindre l’utilisation des ressources système.

### <a name="known-issues"></a>Problèmes connus
* L’explorateur de stockage ne prend pas en charge les comptes ADFS.
* Les touches de raccourci pour les options « View Explorer » (Afficher l’explorateur) et « View Account Management » (Afficher la gestion des comptes) sont Ctrl/Cmd + Maj + E ou Ctrl/Cmd + Maj + A, respectivement.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Le panneau des paramètres de compte peut indiquer qu’il vous faut entrer à nouveau vos informations d’identification pour filtrer les abonnements.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer --disable-gpu
    ```

* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-092"></a>Version 0.9.2
11/01/2017

### <a name="hotfixes"></a>Correctifs logiciels
* Des modifications inattendues de données pouvaient se produire quand des valeurs Edm.DateTime étaient modifiées pour des entités de table en fonction du fuseau horaire local. L’éditeur utilise maintenant une zone de texte brut, qui permet un contrôle précis et continu des valeurs Edm.DateTime.
* Le chargement ou téléchargement d’un groupe d’objets blob ne démarrait pas quand il était joint avec un nom et une clé. Ce problème est à présent résolu.
* Auparavant, l’Explorateur Stockage vous invitait à réauthentifier un compte obsolète uniquement si un ou plusieurs abonnements du compte étaient sélectionnés. L’Explorateur Stockage affiche maintenant une invite même si le compte est entièrement filtré.
* Le domaine de points de terminaison pour Azure US Government était incorrect. Il a été corrigé.
* Cliquer sur le bouton Appliquer dans le panneau Gérer les comptes était parfois difficile. Cela a été corrigé.

### <a name="new"></a>Nouveau
* Prise en charge de la préversion pour Azure Cosmos DB :
    * [Documentation en ligne](./cosmos-db/storage-explorer.md)
    * Créer des bases de données et des collections
    * Manipuler des données
    * Interroger, créer ou supprimer des documents
    * Mettre à jour les procédures stockées, les fonctions définies par l’utilisateur ou les déclencheurs
    * Utiliser des chaînes de connexion pour se connecter à vos bases de données et les gérer
* Amélioration des performances de chargement/téléchargement d’un grand nombre d’objets blob de petite taille.
* Ajout d’une action « Retry All » (Tout réessayer) s’il existe des erreurs dans un groupe de chargement ou de téléchargement d’objets blob.
* L’explorateur de stockage interrompt désormais l’itération lors du chargement/téléchargement d’objets blob s’il détecte une perte de votre connexion réseau. Vous pouvez ensuite reprendre l’itération, une fois que la connexion réseau a été rétablie.
* Ajout des options « Close All » (Tout fermer), « Fermer les autres » et « Fermer » pour les onglets via le menu contextuel.
* L’explorateur de stockage utilise désormais des boîtes de dialogue et des menus contextuels natifs.
* L’explorateur de stockage est désormais plus accessible. Les améliorations incluent :
    * la prise en charge améliorée des lecteurs d’écran : NVDA sur Windows et VoiceOver sur Mac ;
    * l’amélioration des thèmes à contraste élevé ;
    * les correctifs apportés à la tabulation et au focus de clavier.

### <a name="fixes"></a>Correctifs
* Si vous tentiez d’ouvrir ou de télécharger un objet blob avec un nom de fichier Windows non valide, l’opération entraînait un échec. L’explorateur de stockage sera désormais capable de détecter si un nom d’objet blob n’est pas valide et vous demandera si vous souhaitez encoder ou ignorer cet objet blob. L’explorateur de stockage pourra également détecter si un nom de fichier semble être encodé et vous demandera si souhaitez le décoder avant de le charger.
* Lors du chargement de l’objet blob, l’éditeur du conteneur d’objets blob cible ne s’actualisait parfois pas correctement. Ce problème est à présent résolu.
* La prise en charge de plusieurs formats de chaînes de connexion et d’URI SAP a été améliorée. Nous avons résolu tous les problèmes connus, mais n’hésitez pas à nous envoyer vos commentaires si vous en rencontrez d’autres.
* La notification de mises à jour ne fonctionnait pas pour certains utilisateurs dans la version 0.9.0. Ce problème a été résolu. Ceux qui ont été affectés par ce bogue peuvent télécharger manuellement la dernière version de l’Explorateur Stockage [ici](https://azure.microsoft.com/features/storage-explorer/).

### <a name="known-issues"></a>Problèmes connus
* L’explorateur de stockage ne prend pas en charge les comptes ADFS.
* Les touches de raccourci pour les options « View Explorer » (Afficher l’explorateur) et « View Account Management » (Afficher la gestion des comptes) sont Ctrl/Cmd + Maj + E ou Ctrl/Cmd + Maj + A, respectivement.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Le panneau des paramètres de compte peut indiquer qu’il vous faut entrer à nouveau vos informations d’identification pour filtrer les abonnements.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer --disable-gpu
    ```

* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-091-and-090"></a>Versions 0.9.1 et 0.9.0
20/10/2017
### <a name="new"></a>Nouveau
* Prise en charge de la préversion pour Azure Cosmos DB :
    * [Documentation en ligne](./cosmos-db/storage-explorer.md)
    * Créer des bases de données et des collections
    * Manipuler des données
    * Interroger, créer ou supprimer des documents
    * Mettre à jour les procédures stockées, les fonctions définies par l’utilisateur ou les déclencheurs
    * Utiliser des chaînes de connexion pour se connecter à vos bases de données et les gérer
* Amélioration des performances de chargement/téléchargement d’un grand nombre d’objets blob de petite taille.
* Ajout d’une action « Retry All » (Tout réessayer) s’il existe des erreurs dans un groupe de chargement ou de téléchargement d’objets blob.
* L’explorateur de stockage interrompt désormais l’itération lors du chargement/téléchargement d’objets blob s’il détecte une perte de votre connexion réseau. Vous pouvez ensuite reprendre l’itération, une fois que la connexion réseau a été rétablie.
* Ajout des options « Close All » (Tout fermer), « Fermer les autres » et « Fermer » pour les onglets via le menu contextuel.
* L’explorateur de stockage utilise désormais des boîtes de dialogue et des menus contextuels natifs.
* L’explorateur de stockage est désormais plus accessible. Les améliorations incluent :
    * la prise en charge améliorée des lecteurs d’écran : NVDA sur Windows et VoiceOver sur Mac ;
    * l’amélioration des thèmes à contraste élevé ;
    * les correctifs apportés à la tabulation et au focus de clavier.

### <a name="fixes"></a>Correctifs
* Si vous tentiez d’ouvrir ou de télécharger un objet blob avec un nom de fichier Windows non valide, l’opération entraînait un échec. L’explorateur de stockage sera désormais capable de détecter si un nom d’objet blob n’est pas valide et vous demandera si vous souhaitez encoder ou ignorer cet objet blob. L’explorateur de stockage pourra également détecter si un nom de fichier semble être encodé et vous demandera si souhaitez le décoder avant de le charger.
* Lors du chargement de l’objet blob, l’éditeur du conteneur d’objets blob cible ne s’actualisait parfois pas correctement. Ce problème est à présent résolu.
* La prise en charge de plusieurs formats de chaînes de connexion et d’URI SAP a été améliorée. Nous avons résolu tous les problèmes connus, mais n’hésitez pas à nous envoyer vos commentaires si vous en rencontrez d’autres.
* La notification de mises à jour ne fonctionnait pas pour certains utilisateurs dans la version 0.9.0. Ce problème a été résolu. Ceux qui ont été affectés par ce bogue peuvent télécharger manuellement la version la plus récente de l’explorateur de stockage [ici](https://azure.microsoft.com/features/storage-explorer/).

### <a name="known-issues"></a>Problèmes connus
* L’explorateur de stockage ne prend pas en charge les comptes ADFS.
* Les touches de raccourci pour les options « View Explorer » (Afficher l’explorateur) et « View Account Management » (Afficher la gestion des comptes) sont Ctrl/Cmd + Maj + E ou Ctrl/Cmd + Maj + A, respectivement.
* Lorsque vous ciblez Azure Stack, le chargement de certains fichiers en tant qu’objets blob ajoutés peut échouer.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». C’est parce que nous utilisons la solution de contournement du filtre Annuler décrite ici.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Le panneau des paramètres de compte peut indiquer qu’il vous faut entrer à nouveau vos informations d’identification pour filtrer les abonnements.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* L’interpréteur de commandes Électron utilisé par l’explorateur de stockage rencontre des difficultés avec l’accélération matérielle de certains processeurs graphiques (GPU). Si la fenêtre principale de l’explorateur de stockage est vide, vous pouvez essayer de lancer l’explorateur de stockage à partir de la ligne de commande et de désactiver l’accélération GPU en ajoutant le commutateur `--disable-gpu` :

    ```
    ./StorageExplorer --disable-gpu
    ```

* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

## <a name="version-0816"></a>Version 0.8.16
8/21/2017

### <a name="new"></a>Nouveau
* Lorsque vous ouvrez un objet blob, l’explorateur de stockage vous invite à charger le fichier téléchargé si une modification est détectée
* Expérience de connexion Azure Stack améliorée
* Amélioration des performances de chargement/téléchargement d’un grand nombre de fichiers de petite taille en même temps


### <a name="fixes"></a>Correctifs
* Pour certains types d’objets blob, choisir l’option « remplacer » pendant un conflit de chargement pouvait parfois entraîner le redémarrage du chargement.
* Dans la version 0.8.15, les chargements se bloquaient quelquefois à 99 %.
* Si, au cours d’un chargement de fichiers sur un partage de fichiers, vous choisissiez de charger dans un répertoire qui n’existait pas encore, le chargement échouait.
* L’explorateur de stockage créait de façon incorrecte des horodatages pour les requêtes de table et les signatures d’accès partagé.


### <a name="known-issues"></a>Problèmes connus
* Actuellement, l’utilisation d’une chaîne de connexion clé et nom ne fonctionne pas. Une solution est prévue dans la prochaine version. En attendant, vous pouvez utiliser l’attachement avec le nom et la clé.
* Si vous essayez d’ouvrir un fichier avec un nom de fichier Windows non valide, le téléchargement se solde par une erreur de fichier introuvable.
* L’annulation d’une tâche peut prendre un certain temps après avoir cliqué sur « Annuler ». Il s’agit d’une limitation de la bibliothèque de nœuds de stockage Azure.
* Après avoir effectué un chargement de l’objet blob, l’onglet qui a initié le chargement est actualisé. Il s’agit d’une modification du comportement précédent, qui entraîne également un retour vers la racine du conteneur dans lequel vous vous trouvez.
* Si vous choisissez un certificat de code PIN/carte à puce incorrect, vous devez redémarrer pour que l’explorateur de stockage oublie cette décision.
* Le panneau des paramètres de compte peut indiquer qu’il vous faut entrer à nouveau vos informations d’identification pour filtrer les abonnements.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* Pour les utilisateurs sur Ubuntu 14.04, vous devez vous assurer que GCC est à jour, ce qui peut être fait en exécutant les commandes suivantes et en redémarrant votre machine :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

* Pour les utilisateurs sur Ubuntu 17.04, l’installation de GConf est nécessaire. Elle peut être effectuée en exécutant les commandes suivantes, puis en redémarrant votre ordinateur :

    ```
    sudo apt-get install libgconf-2-4
    ```

### <a name="version-0814"></a>Version 0.8.14
06/22/2017

### <a name="new"></a>Nouveau

* Mise à jour vers la version Électron 1.7.2 afin de bénéficier de plusieurs mises à jour de sécurité critiques
* Vous pouvez maintenant accéder rapidement au guide de dépannage en ligne depuis le menu d’aide
* [Guide][2] de dépannage de l’Explorateur Stockage Microsoft Azure
* [Instructions][3] sur la connexion à un abonnement Azure Stack

### <a name="known-issues"></a>Problèmes connus

* Les boutons de la boîte de dialogue de confirmation de la suppression des dossiers ne sont pas référencés par des clics de souris sur Linux. une solution de contournement est d’utiliser la touche Entrée
* Si vous choisissez un certificat de code PIN/carte à puce incorrect vous devrez redémarrer afin que Storage Explorer oublie cette décision
* Des erreurs peuvent survenir si vous chargez plus de trois groupes d’objets blob ou fichiers simultanément
* Le panneau des paramètres de compte peut indiquer que vous devez entrer à nouveau vos informations d’identification pour filtrer les abonnements
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien qu’Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* La version gcc doit être mise à jour ou mise à niveau lors de l’installation d’Ubuntu 14.04 : vous trouverez les étapes de mise à niveau ci-dessous :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```

### <a name="version-0813"></a>Version 0.8.13
05/12/2017

#### <a name="new"></a>Nouveau

* [Guide][2] de dépannage de l’Explorateur Stockage Microsoft Azure
* [Instructions][3] sur la connexion à un abonnement Azure Stack

#### <a name="fixes"></a>Correctifs

* Problème résolu : le chargement du fichier peut provoquer très probablement une erreur de mémoire insuffisante
* Problème résolu : vous pouvez maintenant vous connecter avec le code PIN/carte à puce
* Résolu : L’ouverture dans le portail fonctionne à présent avec Azure Chine 21Vianet, Azure Allemagne, Azure US Government et Azure Stack
* Problème résolu : lors du téléchargement d’un dossier dans un conteneur d’objets blob, une erreur « Opération non conforme » peut se produire
* Problème résolu : sélectionner tout est désactivé lors de gestion des instantanés
* Problème résolu : les métadonnées de l’objet blob de base peuvent être remplacées lorsque vous avez affiché les propriétés de ses instantanés

#### <a name="known-issues"></a>Problèmes connus

* Si vous choisissez un certificat de code PIN/carte à puce incorrect vous devrez redémarrer afin que Storage Explorer oublie cette décision
* En mode zoom avant ou arrière, le niveau de zoom peut être momentanément rétabli sur le niveau par défaut
* Des erreurs peuvent survenir si vous chargez plus de trois groupes d’objets blob ou fichiers simultanément
* Le panneau des paramètres de compte peut indiquer que vous devez entrer à nouveau vos informations d’identification pour filtrer les abonnements
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* La version gcc doit être mise à jour ou mise à niveau lors de l’installation d’Ubuntu 14.04 : vous trouverez les étapes de mise à niveau ci-dessous :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```


### <a name="version-0812-and-0811-and-0810"></a>Versions 0.8.12, 0.8.11 et 0.8.10
04/07/2017

#### <a name="new"></a>Nouveau

* Storage Explorer se ferme désormais automatiquement lorsque vous installez une mise à jour depuis la notification de mise à jour
* L’accès rapide sur place fournit une expérience améliorée pour utiliser vos ressources fréquemment sollicitées
* Dans l’éditeur de conteneur d’objets Blob, vous pouvez maintenant voir à quelle machine virtuelle appartient un objet blob loué
* Vous pouvez réduire maintenant le panneau de gauche
* La détection s’exécute maintenant en même temps que le téléchargement
* Utilisez des statistiques dans les éditeurs de conteneurs d’objets Blob, partages de fichiers et tables pour afficher la taille de vos ressources ou de la sélection
* Vous pouvez maintenant vous connecter à Azure Active Directory (AAD) basé sur les comptes Azure Stack.
* Vous pouvez maintenant télécharger les fichiers d’archive de plus de 32 Mo sur les comptes de stockage Premium
* Prise en charge de l’accessibilité améliorée
* Vous pouvez maintenant ajouter des certificats TLS/SSL X.509 encodés de base 64 approuvés en accédant à Modifier -&gt; Certificats SSL -&gt; Importer certificats

#### <a name="fixes"></a>Correctifs

* Problème résolu : après l’actualisation des informations d’identification d’un compte, l’arborescence n’est parfois pas actualisée automatiquement
* Problème résolu : générer une SAP pour les tables et les files d’attente de l’émulateur peut entraîner une URL non valide
* Problème résolu : les comptes de stockage premium peuvent maintenant être développés lorsqu’un proxy est activé
* Problème résolu : le bouton Appliquer de la page de gestion de comptes ne fonctionne pas si vous avez 1 ou 0 compte sélectionné
* Problème résolu : le téléchargement d’objets blob qui nécessitent des résolutions de conflit peut échouer - résolu dans 0.8.11
* Problème résolu : l’envoi de commentaires a été interrompu dans 0.8.11 - résolu dans 0.8.12

#### <a name="known-issues"></a>Problèmes connus

* Après la mise à niveau vers 0.8.10, vous devez actualiser toutes vos informations d’identification.
* En mode zoom avant ou arrière, le niveau de zoom peut être momentanément rétabli sur le niveau par défaut.
* Des erreurs peuvent survenir si vous chargez plus de trois groupes d’objets blob ou de fichiers simultanément.
* Le panneau des paramètres de compte peut indiquer que vous devez entrer à nouveau vos informations d’identification pour filtrer les abonnements.
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Lors d’un changement de nom, toutes les autres propriétés et métadonnées des objets blob, fichiers et entités sont conservées.
* Bien que Azure Stack ne prend actuellement pas en charge les partages de fichiers, un nœud de partages de fichiers apparaît toujours sous un compte de stockage d’Azure Stack joint.
* La version gcc doit être mise à jour ou mise à niveau lors de l’installation d’Ubuntu 14.04 : vous trouverez les étapes de mise à niveau ci-dessous :

    ```
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
    ```


### <a name="version-089-and-088"></a>Versions 0.8.9 et 0.8.8
02/23/2017

>[!VIDEO https://www.youtube.com/embed/R6gonK3cYAc?ecver=1]

>[!VIDEO https://www.youtube.com/embed/SrRPCm94mfE?ecver=1]


#### <a name="new"></a>Nouveau

* Storage Explorer 0.8.9 télécharge automatiquement la dernière version des mises à jour.
* Correctif : l’utilisation d’un portail URI SAP généré pour attacher un compte de stockage entraîne une erreur.
* Vous pouvez désormais créer, gérer et promouvoir des instantanés d’objets blob.
* Vous pouvez désormais vous connecter aux comptes Azure Chine 21Vianet, Azure Allemagne et Azure US Government.
* Vous pouvez maintenant modifier le niveau de zoom. Utilisez les options dans le menu Affichage pour effectuer un zoom avant, un zoom arrière et réinitialiser le zoom.
* Les caractères Unicode sont désormais pris en charge dans les métadonnées de l’utilisateur pour les fichiers et les objets blob.
* Améliorations de l’accessibilité.
* Les notes de publication de la version suivante peuvent être affichées depuis la notification de mise à jour. Vous pouvez également afficher les notes de publication courantes dans le menu Aide.

#### <a name="fixes"></a>Correctifs

* Problème résolu : le numéro de version s’affiche désormais correctement dans le panneau de configuration sur Windows
* Problème résolu : la recherche n’est plus limitée à 50 000 nœuds
* Problème résolu : le téléchargement vers un partage de fichiers tourne indéfiniment si le répertoire de destination n’existe pas déjà
* Problème résolu : stabilité améliorée pour de longs chargements et téléchargements

#### <a name="known-issues"></a>Problèmes connus

* En mode zoom avant ou arrière, le niveau de zoom peut être momentanément rétabli sur le niveau par défaut.
* L’Accès rapide fonctionne uniquement avec les éléments basés sur l’abonnement. Les ressources locales ou attachées par le biais d’une clé ou d’un jeton SAP ne sont pas prises en charge dans cette version.
* L’Accès rapide peut prendre quelques secondes pour accéder à la ressource cible, selon le nombre de ressources dont vous disposez.
* Des erreurs peuvent survenir si vous chargez plus de trois groupes d’objets blob ou de fichiers simultanément.

12/16/2016
### <a name="version-087"></a>Version 0.8.7

>[!VIDEO https://www.youtube.com/embed/Me4Y4jxoer8?ecver=1]

#### <a name="new"></a>Nouveau

* Vous pouvez choisir la façon dont vous souhaitez résoudre les conflits au début d’une session de mise à jour, de téléchargement ou de copie dans la fenêtre Activités
* Pointez sur un onglet pour afficher le chemin d’accès complet de la ressource de stockage
* Lorsque vous cliquez sur un onglet, il se synchronise avec son emplacement dans le volet de navigation de gauche

#### <a name="fixes"></a>Correctifs

* Problème résolu : Storage Explorer est désormais une application approuvée sur Mac
* Problème résolu : Ubuntu 14.04 est de nouveau pris en charge
* Problème résolu : l’interface utilisateur de compte d’ajout clignote parfois lors du chargement des abonnements
* Problème résolu : certaines ressources de stockage ne sont pas répertoriées dans le volet de navigation de gauche
* Problème résolu : le volet d’action affiche parfois des actions vides
* Problème résolu : la taille de la fenêtre de la dernière session fermée est désormais conservée
* Problème résolu : vous pouvez ouvrir plusieurs onglets pour la même ressource à l’aide du menu contextuel

#### <a name="known-issues"></a>Problèmes connus

* L’Accès rapide fonctionne uniquement avec les éléments basés sur l’abonnement. Les ressources locales ou attachées par le biais d’une clé ou d’un jeton SAP ne sont pas prises en charge dans cette version
* L’Accès rapide peut prendre quelques secondes pour accéder à la ressource cible, selon le nombre de ressources dont vous disposez
* Des erreurs peuvent survenir si vous chargez plus de trois groupes d’objets blob ou fichiers simultanément
* Les recherches peuvent porter sur 50 000 nœuds environ. Au-delà, elles affectent les performances ou peuvent entraîner des exceptions non prises en charge
* Lorsque vous utilisez Storage Explorer sur macOS pour la première fois, il se peut que plusieurs invites vous demandent l’autorisation de l’utilisateur pour accéder au trousseau. Nous vous suggérons de sélectionner Toujours autoriser, de sorte que l’invite cesse de s’afficher

11/18/2016
### <a name="version-086"></a>Version 0.8.6

#### <a name="new"></a>Nouveau

* Vous pouvez désormais épingler les services les plus fréquemment utilisés à l’Accès rapide pour faciliter la navigation
* Vous pouvez désormais ouvrir plusieurs éditeurs dans différents onglets. Cliquez une fois pour ouvrir un onglet temporaire et double-cliquez pour ouvrir un onglet permanent. Vous pouvez également cliquer sur l’onglet temporaire pour le rendre permanent
* Des améliorations notables en termes de stabilité et de performances pour les chargements et téléchargements ont été réalisés, en particulier pour les fichiers volumineux sur des machines rapides
* Des dossiers « virtuels » vides peuvent maintenant être créés dans des conteneurs d’objets blob
* Nous avons réintroduit une recherche étendue avec notre nouvelle recherche de sous-chaîne améliorée. Vous disposez désormais de deux options de recherche :
    * Recherche globale : il suffit d’entrer un terme à rechercher dans la zone de texte de recherche
    * Recherche étendue : cliquez sur l’icône de loupe, en regard d’un nœud, puis ajouter un terme de recherche à la fin du chemin ou cliquez avec le bouton droit et sélectionnez « Rechercher d’ici »
* Nous avons ajouté différents thèmes : Clair (par défaut), Foncé, Contraste noir élevé et Contraste blanc élevé. Allez à Modifier -&gt; Thèmes pour modifier vos préférences de thème
* Vous pouvez modifier les propriétés des objets blob et des fichiers
* Nous prenons désormais en charge les messages des files d’attente codés (base 64) et non codés
* Sous Linux, un système d’exploitation 64 bits est désormais obligatoire. Pour cette version, nous ne prenons en charge que Ubuntu 16.04.1 LTS 64 bits
* Nous avons mis à jour notre logo.

#### <a name="fixes"></a>Correctifs

* Problème résolu : blocage de l’écran
* Sécurité améliorée
* Problème résolu : il peut arriver que des comptes joints en double s’affichent
* Problème résolu : un objet blob avec un type de contenu non défini peut générer une exception
* Problème résolu : impossible d’ouvrir le panneau de requête sur une table vide
* Problème résolu : bogues différents dans la recherche
* Problème résolu : augmentation du nombre de ressources chargées de 50 à 100 lorsque vous cliquez sur « Charger plus »
* Problème résolu : lors de la première exécution, si un compte est connecté, tous les abonnements sont sélectionnés pour ce compte par défaut

#### <a name="known-issues"></a>Problèmes connus

* Cette version de Storage Explorer ne fonctionne pas sur Ubuntu 14.04
* Pour ouvrir plusieurs onglets pour la même ressource, ne cliquez pas toujours sur la même ressource. Cliquez sur une autre ressource, puis revenez en arrière et cliquez sur la ressource d’origine pour l’ouvrir dans un autre onglet
* L’Accès rapide fonctionne uniquement avec les éléments basés sur l’abonnement. Les ressources locales ou attachées par le biais d’une clé ou d’un jeton SAP ne sont pas prises en charge dans cette version
* L’Accès rapide peut prendre quelques secondes pour accéder à la ressource cible, selon le nombre de ressources dont vous disposez
* Des erreurs peuvent survenir si vous chargez plus de trois groupes d’objets blob ou fichiers simultanément
* Les recherches peuvent porter sur 50 000 nœuds environ. Au-delà, elles affectent les performances ou peuvent entraîner des exceptions non prises en charge

10/03/2016
### <a name="version-085"></a>Version 0.8.5

#### <a name="new"></a>Nouveau

* Il est maintenant possible d’utiliser les clés SAP générées par le portail pour les joindre à des comptes de stockage et de ressources

#### <a name="fixes"></a>Correctifs

* Problème résolu : la condition de concurrence lors de la recherche provoque parfois la non extensibilité des nœuds
* Problème résolu : « Utiliser HTTP » ne fonctionne pas lors de la connexion à des comptes de stockage avec la clé et le nom du compte
* Problème résolu : les clés SAP (en particulier celles qui sont générées par le portail) renvoient une erreur « barre oblique »
* Problème résolu : problèmes d’importation de tables
    * Parfois, la clé de partition et la clé de ligne ont été inversées
    * Impossible de lire les clés de partition « nul »

#### <a name="known-issues"></a>Problèmes connus

* Les recherches peuvent porter sur 50 000 nœuds environ. Au-delà, elles affectent les performances
* Actuellement, Azure Stack ne prend pas en charge les fichiers. Par conséquent, toute tentative de développement des Fichiers entraîne une erreur.

09/12/2016
### <a name="version-084"></a>Version 0.8.4

>[!VIDEO https://www.youtube.com/embed/cr5tOGyGrIQ?ecver=1]

#### <a name="new"></a>Nouveau

* Générez des liens directs vers des comptes, conteneurs de stockage, files d’attente, tables ou partages de fichiers pour le partage et l’accès facile à vos ressources. Prise en charge Windows et Mac OS
* Recherchez vos conteneurs d’objets blob, tables, files d’attente, partages de fichiers ou comptes de stockage à partir de la zone de recherche
* Vous pouvez maintenant regrouper des clauses dans le générateur de requêtes de tables
* Renommer et copier/coller des conteneurs d’objets blob, des partages de fichiers, des tables, des objets blob, des dossiers d’objets blob, des fichiers et répertoires depuis des comptes et conteneurs joint par SAP
* Le changement de nom et la& copie des conteneurs d’objets blob et des partages de fichiers préservent désormais les métadonnées et propriétés

#### <a name="fixes"></a>Correctifs

* Problème résolu : impossible de modifier des entités de tables si elles contiennent des propriétés booléennes ou binaires

#### <a name="known-issues"></a>Problèmes connus

* Les recherches peuvent porter sur 50 000 nœuds environ. Au-delà, elles affectent les performances

08/03/2016
### <a name="version-083"></a>Version 0.8.3

>[!VIDEO https://www.youtube.com/embed/HeGW-jkSd9Y?ecver=1]

#### <a name="new"></a>Nouveau

* Renommez des conteneurs, tables et partages de fichiers
* Meilleure expérience de générateur de requête
* Possibilité d’enregistrer et de charger des requêtes
* Liens directs à des conteneurs ou comptes de stockage, files d’attente, tables ou partages de fichiers pour partager vos ressources et y accéder facilement (Windows uniquement, prise en charge macOS très prochainement !)
* Possibilité de gérer et configurer des règles CORS

#### <a name="fixes"></a>Correctifs

* Problème résolu : les comptes Microsoft nécessitent une nouvelle authentification toutes les 8 à 12 heures

#### <a name="known-issues"></a>Problèmes connus

* Parfois, l’interface utilisateur peut sembler bloquée, l’agrandissement de la fenêtre permet de résoudre ce problème
* L’installation de macOS peut nécessiter des autorisations élevées
* Le panneau des paramètres de compte peut indiquer que vous devez entrer à nouveau vos informations d’identification pour filtrer les abonnements
* Le changement de nom des partages de fichiers, conteneurs d’objets blob et tables ne conserve pas les métadonnées ou autres propriétés sur le conteneur, tels que le quota de partage de fichiers, le niveau d’accès public ou les stratégies d’accès
* Les captures instantanées ne sont pas conservées lorsque les blobs sont renommés (individuellement ou dans un conteneur d’objets blob renommé). Cependant, lors d’un changement de nom, toutes les autres propriétés et métadonnées des blobs, fichiers et entités sont conservées.
* Copier ou renommer des ressources ne fonctionne pas à l’intérieur des comptes joints par SAP

07/07/2016
### <a name="version-082"></a>Version 0.8.2

>[!VIDEO https://www.youtube.com/embed/nYgKbRUNYZA?ecver=1]

#### <a name="new"></a>Nouveau

* Les comptes de stockage sont regroupés par abonnements ; le stockage de développement et les ressources jointes via la clé ou SAP sont affichés sous le nœud (local et joint)
* Se déconnecter de comptes dans le panneau « Paramètres de compte Azure »
* Configurer les paramètres de proxy pour activer et gérer la connexion
* Créer et annuler des baux d’objets blob
* Ouvrir des conteneurs d’objets blob, des files d’attente, des tables et des fichiers avec un simple clic

#### <a name="fixes"></a>Correctifs

* Problème résolu : les messages de files d’attente insérés à l’aide des bibliothèques .NET ou Java ne sont pas correctement décodés depuis base64
* Problème résolu : les tables $metrics ne figurent pas pour les comptes de stockage d’objets blob
* Problème résolu : le nœud de la table ne fonctionne pas pour le stockage local (développement)

#### <a name="known-issues"></a>Problèmes connus

* L’installation de macOS peut nécessiter des autorisations élevées

06/15/2016
### <a name="version-080"></a>Version 0.8.0

>[!VIDEO https://www.youtube.com/embed/ycfQhKztSIY?ecver=1]

>[!VIDEO https://www.youtube.com/embed/k4_kOUCZ0WA?ecver=1]

>[!VIDEO https://www.youtube.com/embed/3zEXJcGdl_k?ecver=1]

#### <a name="new"></a>Nouveau

* Prise en charge du partage de fichiers : affichage, téléchargement, chargement, copie de fichiers et de répertoires, URI SAP (créer et connecter)
* Amélioration de l’expérience utilisateur pour la connexion au stockage avec des URI SAP ou des clés de compte
* Exporter des résultats de requêtes de table
* Réorganisation et personnalisation des colonnes de table
* Affichage des conteneurs d’objets blob $logs et de tables $metrics pour les comptes de stockage avec les mesures activées
* L’amélioration du comportement d’exportation et d’importation, inclut désormais le type de valeur de propriété

#### <a name="fixes"></a>Correctifs

* Problème résolu : le chargement ou le téléchargement d’objets blob volumineux peut entraîner des chargements/téléchargements incomplets
* Problème résolu : la modification, l’ajout ou l’importation d’une entité avec une valeur de chaîne numérique (« 1 ») la convertit en double
* Problème résolu : impossible de développer le nœud de la table dans l’environnement de développement local

#### <a name="known-issues"></a>Problèmes connus

* Les tables $metrics ne figurent pas pour les comptes de stockage d’objets blob
* Les messages de files d’attente ajoutés par programme ne peuvent pas s’afficher correctement si les messages sont encodés à l’aide de l’encodage Base64

05/17/2016
### <a name="version-07201605090"></a>Version 0.7.20160509.0

#### <a name="new"></a>Nouveau

* Une meilleure gestion des erreurs pour les incidents d’applications

#### <a name="fixes"></a>Correctifs

* Bogue résolu dans lequel les messages de la barre d’informations parfois n’apparaissent pas lorsque les informations d’identification de connexion sont requises

#### <a name="known-issues"></a>Problèmes connus

* Tables : Ajout, modification ou importation d’une entité qui a une propriété avec une valeur numérique ambiguë, comme « 1 » ou « 1.0 » et l’utilisateur tente de l’envoyer en tant que `Edm.String`, la valeur revient via l’API client comme un Edm.Double

03/31/2016

### <a name="version-07201603250"></a>Version 0.7.20160325.0

>[!VIDEO https://www.youtube.com/embed/imbgBRHX65A?ecver=1]

>[!VIDEO https://www.youtube.com/embed/ceX-P8XZ-s8?ecver=1]

#### <a name="new"></a>Nouveau

* Prise en charge de la table : opérations d’affichage, d’interrogation, d’exportation, d’importation et CRUD pour les entités
* Prise en charge de la file d’attente : affichage, ajout, retrait de la file d’attente des messages
* Génération d’URI SAP pour les comptes de stockage
* Connexion à des comptes de stockage avec des URI SAP
* Mettre à jour des notifications pour les futures mises à jour de Storage Explorer
* Apparence mise à jour

#### <a name="fixes"></a>Correctifs

* Améliorations des performances et de la fiabilité

### <a name="known-issues-amp-mitigations"></a>Problèmes connus &amp; atténuations des risques

* Le téléchargement de fichiers d’objets blob volumineux ne fonctionne pas correctement, nous vous recommandons d’utiliser AzCopy pendant que nous résolvons ce problème
* Les informations d’identification ne sont pas récupérées ni mises en cache s’il est impossible de trouver ou d’écrire sur le dossier de base
* Si nous ajoutons, modifions ou importons une entité qui a une propriété avec une valeur numérique ambiguë, comme « 1 » ou « 1.0 » et que l’utilisateur tente de l’envoyer en tant que `Edm.String`, la valeur revient via l’API client comme un Edm.Double
* Lorsque vous importez des fichiers CSV avec des enregistrements multilignes, les données peuvent être tronquées ou embrouillées

02/03/2016

### <a name="version-07201601291"></a>Version 0.7.20160129.1

#### <a name="fixes"></a>Correctifs

* Amélioration des performances globales lors du chargement, du téléchargement et de la copie des objets blob

01/14/2016

### <a name="version-07201601050"></a>Version 0.7.20160105.0

#### <a name="new"></a>Nouveau

* Prise en charge de Linux (fonctionnalités de parité vers OSX)
* Ajouter des conteneurs d’objets blob avec une clé Signatures d’accès partagé (SAP)
* Ajouter des comptes de stockage pour Azure Chine 21Vianet
* Ajouter des comptes de stockage avec des points de terminaison personnalisés
* Ouvrez et affichez les blobs de texte et d’image du contenu
* Affichez et modifiez les propriétés et métadonnées des blobs

#### <a name="fixes"></a>Correctifs

* Problème résolu : le chargement ou le téléchargement d’un grand nombre d’objets blob (plus de 500) peut parfois provoquer l’affichage d’un écran blanc sur l’application
* Problème résolu : lors de la définition du niveau d’accès public du conteneur d’objets blob, la nouvelle valeur n'est pas mise à jour tant vous n’avez pas redéfini le focus sur le conteneur. En outre, la boîte de dialogue affiche toujours par défaut « Aucun accès public » et non la valeur actuelle réelle.
* Une meilleure prise en charge globale du clavier/accessibilité et de l’interface utilisateur
* Le texte de l’historique des vues miniatures est encapsulé lorsqu’il est long avec un espace blanc
* La boîte de dialogue SAP prend en charge la validation d’entrée
* Le stockage local reste disponible même si les informations d’identification de l’utilisateur ont expiré
* Lorsqu’un conteneur d’objets blob ouvert est supprimé, l’explorateur d’objets blob sur le côté droit est fermé

#### <a name="known-issues"></a>Problèmes connus

* La version gcc doit être mise à jour ou mise à niveau lors de l’installation de Linux : vous trouverez les étapes de mise à niveau ci-dessous :
    * `sudo add-apt-repository ppa:ubuntu-toolchain-r/test`
    * `sudo apt-get update`
    * `sudo apt-get upgrade`
    * `sudo apt-get dist-upgrade`

11/18/2015
### <a name="version-07201511160"></a>Version 0.7.20151116.0

#### <a name="new"></a>Nouveau

* Versions macOS et Windows
* Connectez-vous pour afficher vos comptes de stockage : utilisez votre compte professionnel, Microsoft, 2FA, etc.
* Stockage de développement local (utilisez l’émulateur de stockage, Windows uniquement)
* Prise en charge d’Azure Resource Manager et des ressources classiques
* Créez et supprimez des objets blob, des files d’attente ou des tables
* Recherchez des objets blob, des files d’attente ou des tables spécifiques
* Explorez le contenu de conteneurs d’objets blob
* Affichez et parcourez les répertoires
* Chargez, téléchargez et supprimez des objets blob et des dossiers
* Affichez et modifiez les propriétés et métadonnées des blobs
* Générez des clés SAP
* Gérez et créez des stratégies d’accès stockés (SAS)
* Recherchez les blobs par préfixe
* Glissez-déplacez les fichiers à charger ou télécharger

#### <a name="known-issues"></a>Problèmes connus

* Lors de la définition du niveau d’accès public au conteneur d’objets blob, la nouvelle valeur n'est pas mise à jour tant vous n’avez pas redéfini le focus sur le conteneur
* Lorsque vous ouvrez la boîte de dialogue pour définir le niveau d’accès public, « Aucun accès public » est toujours affichée en tant que valeur par défaut et pas la valeur actuelle réelle
* Impossible de renommer des objets blob téléchargés
* Les entrées du journal d’activité sont parfois « coincées » dans un état En cours lorsqu’une erreur se produit et l’erreur n’est pas affichée
* Il se bloque parfois ou devient blanc lorsque vous tentez de charger ou télécharger un grand nombre d’objets blob
* Parfois, l’annulation d’une opération de copie ne fonctionne pas
* Au cours de la création d’un conteneur (objet blob/file d’attente/table), si vous entrez un nom non valide et passez à la création d’un autre sous un type de conteneur différent, il est impossible de définir le focus sur le nouveau type
* Impossible de créer un nouveau dossier ou de renommer un dossier




[2]: https://support.microsoft.com/en-us/help/4021389/storage-explorer-troubleshooting-guide
[3]: vs-azure-tools-storage-manage-with-storage-explorer.md