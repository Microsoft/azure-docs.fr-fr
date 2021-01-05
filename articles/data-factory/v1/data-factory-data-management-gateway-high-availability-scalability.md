---
title: Haute disponibilité avec la passerelle de gestion des données dans Azure Data Factory
description: Cet article explique comment effectuer un scale-out d’une passerelle de gestion des données en ajoutant des nœuds et comment effectuer un scale-up en augmentant le nombre de travaux simultanés pouvant s’exécuter sur un nœud.
services: data-factory
documentationcenter: ''
author: nabhishek
manager: anandsub
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: b8d05293359cff16bb6d8c9a629a1fbf68104365
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003614"
---
# <a name="data-management-gateway---high-availability-and-scalability-preview"></a>Passerelle de gestion des données - Haute disponibilité et scalabilité (préversion)
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Guide pratique pour créer et configurer le runtime d’intégration autohébergé dans Data Factory](../create-self-hosted-integration-runtime.md). 


Cet article vous aide à configurer la solution de haute disponibilité et scalabilité avec la passerelle de gestion des données / intégration.    

> [!NOTE]
> Cet article part du principe que vous connaissez déjà les bases du runtime d’intégration (anciennement passerelle de gestion des données). Si ce n’est pas le cas, consultez [Passerelle de gestion des données](data-factory-data-management-gateway.md).
> 
> **Cette fonctionnalité en version préliminaire est officiellement prise en charge sur les versions 2.12.xxxx.x et ultérieures de la passerelle de gestion des données**. Assurez-vous que vous utilisez la version 2.12.xxxx.x ou une version supérieure. Téléchargez [ici](https://www.microsoft.com/download/details.aspx?id=39717) la dernière version de la passerelle de gestion des données.

## <a name="overview"></a>Vue d’ensemble
Vous pouvez associer des passerelles de gestion des données installées sur plusieurs ordinateurs locaux à une seule passerelle logique du portail. Ces ordinateurs sont appelés **nœuds**. Vous pouvez associer jusqu’à **quatre nœuds** à une passerelle logique. Avoir plusieurs nœuds (ordinateurs locaux avec une passerelle installée) procure les avantages suivants à une passerelle logique :  

- Les performances du déplacement des données entre les magasins de données locaux et dans le cloud sont améliorées.  
- Si l’un des nœuds tombe en panne pour une raison quelconque, les autres nœuds sont toujours disponibles pour déplacer les données. 
- Si l’un des nœuds a besoin d’être mis hors connexion à des fins de maintenance, les autres nœuds sont toujours disponibles pour déplacer les données.

Vous pouvez également configurer le nombre de **travaux de déplacement de données simultanés** qui peuvent s’exécuter sur un nœud pour augmenter la capacité de déplacement des données entre les magasins de données locaux et dans le cloud. 

À l’aide du portail Azure, vous pouvez surveiller l’état de ces nœuds, ce qui vous permet de déterminer s’il faut ajouter ou supprimer un nœud dans la passerelle logique. 

## <a name="architecture"></a>Architecture 
Le diagramme suivant offre une vue d’ensemble architecturale de la fonctionnalité Haute disponibilité et scalabilité de la passerelle de gestion des données : 

![Passerelle de gestion des données - Haute disponibilité et scalabilité](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-high-availability-and-scalability.png)

Une **passerelle logique** est une passerelle que vous ajoutez à une fabrique de données dans le portail Azure. Avant, vous ne pouviez associer qu’un seul ordinateur Windows local doté d’une passerelle de gestion des données installée avec une passerelle logique. Cet ordinateur passerelle local est appelé nœud. Maintenant, vous pouvez associer jusqu’à **quatre nœuds physiques** avec une passerelle logique. Une passerelle logique dotée de plusieurs nœuds est appelée **passerelle à plusieurs nœuds**.  

Tous ces nœuds sont **actifs**. Ils peuvent tous traiter des travaux de déplacement des données pour déplacer des données entre des magasins de données locaux et dans le cloud. Un des nœuds joue à la fois le rôle de répartiteur et le rôle de travail. Les autres nœuds inclus dans les groupes sont des nœuds de travail. Un nœud **répartiteur** extrait des tâches/travaux de déplacement des données auprès du service cloud et les répartit entre les nœuds de travail (y compris lui-même). Un nœud **rôle de travail** exécute les travaux de déplacement des données pour déplacer des données entre les magasins de données locaux et dans le cloud. Tous les nœuds sont des rôles de travail. Un seul nœud peut être à la fois répartiteur et rôle de travail.    

En général, vous démarrez avec un seul nœud et vous **effectuez un scale-out** pour ajouter d’autres nœuds quand les nœuds existants sont submergés par la charge de déplacement des données. Vous pouvez également **augmenter la taille des instances** (scale up) pour déplacer des données d’un nœud de passerelle en augmentant le nombre de travaux simultanés qui peuvent s’exécuter sur le nœud. Cette fonctionnalité est également disponible avec une passerelle à nœud unique (même quand la fonctionnalité Haute disponibilité et scalabilité n’est pas activée). 

Une passerelle à plusieurs nœuds maintient les informations d’identification des magasins de données synchronisées sur tous les nœuds. En cas de problème de connectivité nœud à nœud, les informations d’identification risquent de ne pas être synchronisées. Quand vous définissez des informations d’identification pour un magasin de données local qui utilise une passerelle, celui-ci les enregistre sur le nœud répartiteur/rôle de travail. Le nœud répartiteur se synchronise avec les autres nœuds rôles de travail. Ce processus est appelé **synchronisation des informations d’identification**. Le canal de communication entre les nœuds peut être **chiffré** par un certificat SSL/TLS public. 

## <a name="set-up-a-multi-node-gateway"></a>Configurer une passerelle à plusieurs nœuds
Cette section part du principe que vous avez parcouru les deux articles suivants ou que vous connaissez les concepts abordés dans ces articles : 

- [Passerelle de gestion des données](data-factory-data-management-gateway.md) - fournit une vue d’ensemble descriptive de la passerelle.
- [Déplacer des données entre des magasins de données locaux et dans le cloud](data-factory-move-data-between-onprem-and-cloud.md) - contient une procédure pas à pas donnant des instructions sur l’utilisation d’une passerelle à nœud unique.  

> [!NOTE]
> Avant d’installer une passerelle de gestion des données sur un ordinateur Windows local, consultez les prérequis indiqués dans l’[article principal](data-factory-data-management-gateway.md#prerequisites).

1. Dans la [procédure pas à pas](data-factory-move-data-between-onprem-and-cloud.md#create-gateway), quand vous créez une passerelle logique, activez la fonctionnalité **Haute disponibilité et scalabilité**. 

    ![Passerelle de gestion des données - Haute disponibilité et scalabilité](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-enable-high-availability-scalability.png)
2. Dans la page **Configurer**, utilisez le lien **Installation rapide** ou **Installation manuelle** pour installer une passerelle sur le premier nœud (un ordinateur Windows local).

    ![Passerelle de gestion des données - Installation rapide ou manuelle](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-express-manual-setup.png)

    > [!NOTE]
    > Si vous utilisez l’option d’installation rapide, la communication nœud à nœud s’effectue sans chiffrement. Le nom du nœud est identique à celui de l’ordinateur. Utilisez l’installation manuelle si la communication nœud à nœud doit être chiffrée ou si vous voulez indiquer un nom de nœud particulier. Il n’est pas possible de modifier les noms de nœuds ultérieurement.
3. Si vous choisissez **Installation rapide** :
    1. Le message suivant s’affiche une fois que la passerelle est correctement installée :

        ![Passerelle de gestion des données - Installation rapide terminée](media/data-factory-data-management-gateway-high-availability-scalability/express-setup-success.png)
    2. Lancez le Gestionnaire de configuration de passerelle de gestion des données en suivant [ces instructions](data-factory-data-management-gateway.md#configuration-manager). Le nom de la passerelle, le nom du nœud, l’état, etc. s’affichent.

        ![Capture d’écran montrant l’endroit où sont affichés le nom de la passerelle, le nom du nœud et l’état.](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-installation-success.png)
4. Si vous choisissez **Installation manuelle** :
    1. Téléchargez le package d’installation à partir du Centre de téléchargement Microsoft, puis exécutez-le pour installer la passerelle sur votre ordinateur.
    2. Utilisez la **clé d’authentification** indiquée dans la page **Configurer** pour inscrire la passerelle.
    
        ![Capture d’écran montrant l’endroit où utiliser la clé d’authentification.](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-authentication-key.png)
    3. Dans la page **Nouveau nœud de passerelle**, vous pouvez indiquer un **nom** personnalisé pour le nœud de passerelle. Par défaut, un nom de nœud est identique à celui de l’ordinateur.    

        ![Passerelle de gestion des données - Spécifier un nom](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-name.png)
    4. Dans la page suivante, vous pouvez choisir d’**activer ou non le chiffrement pour la communication nœud à nœud**. Cliquez sur **Ignorer** pour désactiver le chiffrement (par défaut).

        ![Passerelle de gestion des données - Activer le chiffrement](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-node-encryption.png)  
    
        > [!NOTE]
        > Le changement de mode de chiffrement est uniquement pris en charge quand vous avez un seul nœud de passerelle dans la passerelle logique. Pour changer de mode de chiffrement quand une passerelle a plusieurs nœuds, procédez comme suit : supprimez tous les nœuds à l’exception d’un seul, changez le mode de chiffrement, puis ajoutez de nouveau les nœuds.
        > 
        > Consultez la section [Configuration requise des certificats TSL/SSL](#tlsssl-certificate-requirements) pour obtenir la liste des conditions préalables à l’utilisation d’un certificat TLS/SSL. 
    5. Une fois que la passerelle est correctement installée, cliquez sur Lancer le Gestionnaire de configuration :
    
        ![Installation manuelle - Lancer le Gestionnaire de configuration](media/data-factory-data-management-gateway-high-availability-scalability/manual-setup-launch-configuration-manager.png)   
    6. Le Gestionnaire de configuration de passerelle de gestion des données (ordinateur Windows local) apparaît et indique l’état de la connectivité, le **nom de la passerelle** et le **nom du nœud**.  

        ![Passerelle de gestion des données - Installation terminée](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-installation-success.png)

        > [!NOTE]
        > Si vous approvisionnez la passerelle sur une machine virtuelle Azure, vous pouvez utiliser [ce modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-mutiple-vms-with-data-management-gateway). Ce script crée une passerelle logique, installe le logiciel de la passerelle de gestion des données sur les machines virtuelles et les inscrit auprès de la passerelle logique. 
6. Dans le portail Azure, lancez la page **Passerelle** : 
    1. Dans la page d’accueil de la fabrique de données dans le portail, cliquez sur **Services liés**.
    
        ![Capture d’écran mettant en évidence la vignette des service liés.](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-home-page.png)
    2. Sélectionnez la **passerelle** pour afficher la page **Passerelle** :
    
        ![Page d’accueil Data Factory](media/data-factory-data-management-gateway-high-availability-scalability/linked-services-gateway.png)
    4. La page **Passerelle** s’affiche :   

        ![Affichage de la passerelle à nœud unique](media/data-factory-data-management-gateway-high-availability-scalability/gateway-first-node-portal-view.png) 
7. Cliquez sur **Ajouter un nœud** dans la barre d’outils pour ajouter un nœud à la passerelle logique. Si vous envisagez d’utiliser l’installation rapide, effectuez cette étape à partir de l’ordinateur local qui sera ajouté en tant que nœud à la passerelle. 

    ![Passerelle de gestion des données - Menu Ajouter un nœud](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-add-node-menu.png)
8. Les étapes sont similaires à celles de la configuration du premier nœud. L’interface utilisateur du Gestionnaire de configuration vous permet de définir le nom du nœud, si vous choisissez l’option d’installation manuelle : 

    ![Gestionnaire de configuration - Installer une deuxième passerelle](media/data-factory-data-management-gateway-high-availability-scalability/install-second-gateway.png)
9. Une fois que la passerelle est correctement installée sur le nœud, l’outil Gestionnaire de configuration affiche l’écran suivant :  

    ![Gestionnaire de configuration - Installation de la deuxième passerelle terminée](media/data-factory-data-management-gateway-high-availability-scalability/second-gateway-installation-successful.png)
10. Si vous ouvrez la page **Passerelle** dans le portail, vous voyez maintenant deux nœuds de passerelle : 

    ![Passerelle à deux nœuds dans le portail](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring.png)
11. Pour supprimer un nœud de passerelle, cliquez sur **Supprimer un nœud** dans la barre d’outils, sélectionnez le nœud à supprimer, puis cliquez sur **Supprimer** dans la barre d’outils. Cette action supprime le nœud sélectionné du groupe. Notez que cette action ne désinstalle pas du nœud (ordinateur Windows local) le logiciel de la passerelle de gestion des données. Utilisez **Ajouter ou supprimer des programmes** dans le Panneau de configuration de l’ordinateur local pour désinstaller la passerelle. Quand vous désinstallez la passerelle du nœud, elle est automatiquement supprimée dans le portail.   

## <a name="upgrade-an-existing-gateway"></a>Mettre à niveau une passerelle existante
Vous pouvez mettre à niveau une passerelle existante pour utiliser la fonctionnalité Haute disponibilité et scalabilité. Cette fonctionnalité est uniquement opérationnelle avec des nœuds dont la version de la passerelle de gestion des données est supérieure ou égale à 2.12.xxxx. Pour connaître la version installée de la passerelle de gestion des données sur un ordinateur, consultez l’onglet **Aide** du Gestionnaire de configuration de passerelle de gestion des données. 

1. Pour mettre à jour la passerelle sur l’ordinateur local en lui appliquant la version la plus récente, téléchargez et exécutez un package d’installation MSI à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=39717). Consultez la section [Installation](data-factory-data-management-gateway.md#installation) pour plus d’informations.  
2. Accédez au portail Azure. Lancez la **page Data Factory** de votre fabrique de données. Cliquez sur la vignette Services liés pour lancer la **page Services liés**. Sélectionnez la passerelle pour lancer la **page Passerelle**. Cliquez sur **Fonctionnalité en préversion** pour l’activer comme indiqué dans l’image suivante : 

    ![Passerelle de gestion des données - Activer la fonctionnalité en préversion](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-existing-gateway-enable-high-availability.png)   
2. Une fois que la fonctionnalité en préversion est activée dans le portail, fermez toutes les pages. Rouvrez la **page Passerelle** pour afficher la nouvelle interface utilisateur (IU) en préversion.
 
    ![Passerelle de gestion des données - Activation de la fonctionnalité en préversion terminée](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-preview-success.png)

    ![Passerelle de gestion des données - Interface utilisateur en préversion](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-preview.png)

    > [!NOTE]
    > Pendant la mise à niveau, le nom du premier nœud correspond au nom de l’ordinateur. 
3. Maintenant, ajoutez un nœud. Dans la page **Passerelle**, cliquez sur **Ajouter un nœud**.  

    ![Passerelle de gestion des données - Menu Ajouter un nœud](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-add-node-menu.png)

    Suivez les instructions de la section précédente pour configurer le nœud. 

### <a name="installation-best-practices"></a>Bonnes pratiques d’installation

- Définissez un plan d'alimentation sur l'ordinateur hôte de la passerelle afin d’empêcher la mise en veille prolongée. Si l’ordinateur hôte est en veille prolongée, la passerelle ne répond pas à la demande de données.
- Sauvegardez le certificat associé à la passerelle.
- Vérifiez que la configuration de tous les nœuds est similaire pour atteindre les performances idéales (recommandé). 
- Ajoutez au moins deux nœuds pour garantir la haute disponibilité.  

### <a name="tlsssl-certificate-requirements"></a>Configuration requise des certificats TLS/SSL
Voici la configuration requise pour le certificat TLS/SSL utilisé pour sécuriser les communications entre les nœuds de runtime d’intégration :

- Le certificat doit être un certificat X509 v3 approuvé publiquement. Nous vous recommandons d’utiliser des certificats émis par une autorité de certification (tierce) publique.
- Chaque nœud de runtime d’intégration doit approuver ce certificat, ainsi que l’ordinateur client qui exécute l’application du gestionnaire d’informations d’identification. 
  > [!NOTE]
  > L’application du gestionnaire d’informations d’identification est utilisée lors de la définition en toute sécurité des informations d’identification à partir de l’Assistant Copie / portail Azure. Et elle peut être déclenchée à partir de n’importe quel ordinateur appartenant au même réseau que le magasin de données local ou privé.
- Les certificats utilisant des caractères génériques sont pris en charge. Si votre nom de domaine complet est **node1.domain.contoso.com**, vous pouvez utiliser **_.domain.contoso.com_* comme nom du sujet du certificat.
- Les certificats SAN ne sont pas recommandés, car seul le dernier élément des Autres noms de l’objet sera utilisé et tous les autres seront ignorés en raison d’une limitation actuelle. Par exemple, si vous avez un certificat SAN dont les noms SAN sont **node1.domain.contoso.com** et **node2.domain.contoso.com**, vous ne pouvez utiliser ce certificat que sur l’ordinateur dont le FQDN est **node2.domain.contoso.com**.
- Prise en charge de toutes les tailles de clé prises en charge par Windows Server 2012 R2 pour les certificats TLS/SSL.
- Les certificat utilisant des clés CNG ne sont pas pris en charge.

#### <a name="faq-when-would-i-not-enable-this-encryption"></a>Questions fréquentes : Quand faut-il ne pas activer ce chiffrement ?
L’activation du chiffrement peut ajouter certains coûts à votre infrastructure (possession d’un certificat public). Par conséquent, vous pouvez ignorer l’activation du chiffrement dans les cas ci-dessous :
- Quand le runtime d’intégration s’exécute sur un réseau approuvé ou sur un réseau avec un chiffrement transparent comme IP/SEC. Étant donné que ce canal de communication se limite uniquement à votre réseau approuvé, vous n’aurez peut-être pas besoin d’un chiffrement supplémentaire.
- Quand le runtime d’intégration ne s’exécute pas dans un environnement de production. Cela peut aider à réduire le coût du certificat TLS/SSL.


## <a name="monitor-a-multi-node-gateway"></a>Surveiller une passerelle à plusieurs nœuds
### <a name="multi-node-gateway-monitoring"></a>Surveillance d’une passerelle à plusieurs nœuds
Dans le portail Azure, vous pouvez afficher un instantané en quasi temps réel de l’utilisation des ressources (processeur, mémoire, réseau (entrée/sortie), etc.) sur chaque nœud, ainsi que les états des nœuds de passerelle. 

![Passerelle de gestion des données - Surveillance de plusieurs nœuds](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring.png)

Vous pouvez activer l’option **Paramètres avancés** dans la page **Passerelle** pour afficher des métriques avancées telles que **Réseau**(entrée/sortie), **État du rôle et des informations d’identification**, ce qui s’avère utile pour résoudre les problèmes liés à la passerelle, ainsi que **Tâches simultanées** (en cours d’exécution/limite) que vous pouvez modifier en conséquence lors du réglage des performances. Le tableau suivant fournit les descriptions des colonnes utilisées dans la liste **Nœuds de passerelle** :  

Propriété de surveillance | Description
:------------------ | :---------- 
Nom | Nom de la passerelle logique et nœuds associés à la passerelle.  
Statut | État de la passerelle logique et des nœuds de passerelle. Exemple : En ligne/Hors connexion/Limité/etc. Pour plus d’informations sur ces états, consultez la section [État de la passerelle](#gateway-status). 
Version | Indique la version de la passerelle logique et de chaque nœud de passerelle. La version de la passerelle logique est déterminée selon la version de la majorité des nœuds dans le groupe. S’il existe des nœuds de différentes versions dans l’installation de la passerelle logique, seuls les nœuds dont le numéro de version est identique à celui de la passerelle logique fonctionnent correctement. Les autres sont en mode limité et ont besoin d’une mise à jour manuelle (uniquement si la mise à jour automatique échoue). 
Mémoire disponible | Mémoire disponible sur un nœud de passerelle. Cette valeur est un instantané en quasi temps réel. 
Utilisation du processeur | Utilisation du processeur d’un nœud de passerelle. Cette valeur est un instantané en quasi temps réel. 
Réseau (entrée/sortie) | Utilisation du réseau d’un nœud de passerelle. Cette valeur est un instantané en quasi temps réel. 
Tâches simultanées (en cours d’exécution/limite) | Nombre de travaux ou tâches qui s’exécutent sur chaque nœud. Cette valeur est un instantané en quasi temps réel. La limite correspond au nombre maximal de travaux simultanés pour chaque nœud. Cette valeur est définie selon la taille de l’ordinateur. Vous pouvez augmenter la limite pour monter en puissance l’exécution de tâches simultanées dans les scénarios avancés, où le processeur/la mémoire /le réseau sont sous-utilisés, alors que les activités expirent. Cette fonctionnalité est également disponible avec une passerelle à nœud unique (même quand la fonctionnalité Haute disponibilité et scalabilité n’est pas activée). Pour plus d’informations, consultez la section [Considérations d’échelle](#scale-considerations). 
Role | Il existe deux types de rôles : répartiteur et rôle de travail. Tous les nœuds sont des rôles de travail, ce qui signifie qu’ils peuvent tous être utilisés pour exécuter des tâches. Il n’existe qu’un seul nœud répartiteur, utilisé pour extraire des tâches/travaux auprès de services cloud et les répartir entre les différents nœuds rôles de travail (y compris lui-même). 

![Passerelle de gestion des données - Surveillance avancée de plusieurs nœuds](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring-advanced.png)

### <a name="gateway-status"></a>État de la passerelle

Le tableau suivant indique les états possibles d’un **nœud de passerelle** : 

Statut  | Commentaires/Scénarios
:------- | :------------------
En ligne | Nœud connecté au service Data Factory.
Hors connexion | Le nœud est hors connexion.
Mise à niveau | Le nœud est en cours de mise à jour automatique.
Limité | Dû à un problème de connectivité. Éventuellement dû à un problème de port HTTP 8050, à un problème de connectivité du bus de service ou à un problème de synchronisation des informations d’identification. 
Inactif | La configuration du nœud est différente de celle de la majorité des autres nœuds.<br/><br/> Un nœud peut être inactif quand il ne parvient pas à se connecter à d’autres nœuds. 


Le tableau suivant indique les états possibles d’une **passerelle logique**. L’état de la passerelle dépend des états des nœuds de passerelle. 

Statut | Commentaires
:----- | :-------
Doit être inscrite | Aucun nœud n’est encore inscrit sur cette passerelle logique.
En ligne | Les nœuds de passerelle sont en ligne.
Hors connexion | Aucun nœud n’est en ligne.
Limité | Tous les nœuds inclus dans cette passerelle ne sont pas dans un état intègre. Cet état est un avertissement pouvant indiquer que certains nœuds sont en panne ! <br/><br/>Peut être dû à un problème de synchronisation des informations d’identification sur le nœud répartiteur/rôle de travail. 

### <a name="pipeline-activities-monitoring"></a>Surveillance de pipeline/des activités
Le portail Azure propose une expérience de surveillance de pipeline avec des détails précis au niveau du nœud. Par exemple, il indique quelles activités se sont exécutées sur quel nœud. Ces informations peuvent s’avérer utiles pour comprendre les problèmes de performances sur un nœud particulier, par exemple en raison d’une limitation du réseau. 

![Passerelle de gestion des données - Surveillance de plusieurs nœuds pour les pipelines](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-monitoring-pipelines.png)

![Passerelle de gestion des données - Détails de pipeline](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-multi-node-pipeline-details.png)

## <a name="scale-considerations"></a>Considérations d’échelle

### <a name="scale-out"></a>Scale-out
Quand la **mémoire disponible est faible** et l’**utilisation du processeur est élevée**, l’ajout d’un nouveau nœud permet d’effectuer un scale-out de la charge sur les ordinateurs. Si des activités échouent en raison d’un délai d’expiration ou de l’état hors connexion d’un nœud de passerelle, il est judicieux d’ajouter un nœud à la passerelle.
 
### <a name="scale-up"></a>Monter en puissance
Quand la mémoire disponible et le processeur ne sont pas correctement utilisés, mais que la capacité inactive s’élève à 0, vous devez augmenter la taille des instances en augmentant le nombre de travaux simultanés pouvant s’exécuter sur un nœud. Vous pouvez également augmenter la taille des instances quand les activités expirent parce que la passerelle est surchargée. Comme le montre l’image suivante, vous pouvez augmenter la capacité maximale pour un nœud. Nous vous suggérons de la doubler dans un premier temps.  

![Passerelle de gestion des données - Considérations d’échelle](media/data-factory-data-management-gateway-high-availability-scalability/data-factory-gateway-scale-considerations.png)


## <a name="known-issuesbreaking-changes"></a>Problèmes connus/nouveautés

- Actuellement, vous pouvez avoir jusqu’à quatre nœuds de passerelle physiques pour une seule passerelle logique. Si vous avez besoin de plus de quatre nœuds pour des raisons de performances, envoyez un e-mail à [DMGHelp@microsoft.com](mailto:DMGHelp@microsoft.com).
- Vous ne pouvez pas réinscrire un nœud de passerelle avec la clé d’authentification d’une autre passerelle logique pour le permuter de la passerelle logique actuelle. Pour le réinscrire, désinstallez la passerelle du nœud, réinstallez la passerelle et inscrivez-le à l’aide de la clé d’authentification de l’autre passerelle logique. 
- Si le proxy HTTP est exigé pour tous les nœuds de passerelle, définissez-le dans diahost.exe.config et diawp.exe.config, puis utilisez le gestionnaire de serveur pour vérifier que tous les nœuds ont les mêmes fichiers diahost.exe.config et diawip.exe.config. Consultez la section [Configurer les paramètres du proxy](data-factory-data-management-gateway.md#configure-proxy-server-settings) pour plus d’informations. 
- Pour modifier le mode de chiffrement pour la communication nœud à nœud dans le Gestionnaire de configuration de passerelle, supprimez tous les nœuds dans le portail, sauf un. Ensuite, rajoutez les nœuds après avoir modifié le mode de chiffrement.
- Utilisez un certificat TLS/SSL officiel si vous choisissez de chiffrer le canal de communication nœud à nœud. Un certificat auto-signé peut entraîner des problèmes de connectivité car le même certificat peut ne pas être approuvé dans la liste de l’autorité de certification sur les autres ordinateurs. 
- Vous ne pouvez pas inscrire un nœud de passerelle sur une passerelle logique quand la version du nœud est inférieure à celle de la passerelle logique. Supprimez tous les nœuds de la passerelle logique du portail pour pouvoir inscrire un nœud de version inférieure (le passer à une version antérieure). Si vous supprimez tous les nœuds d’une passerelle logique, installez et inscrivez manuellement de nouveaux nœuds sur cette passerelle logique. L’installation rapide n’est pas prise en charge dans ce cas.
- Vous ne pouvez pas utiliser l’installation rapide pour installer des nœuds sur une passerelle logique existante, qui utilise encore des informations d’identification du cloud. Vous pouvez vérifier l’emplacement de stockage des informations d’identification à partir du Gestionnaire de configuration de passerelle sous l’onglet Paramètres.
- Vous ne pouvez pas utiliser l’installation rapide pour installer des nœuds sur une passerelle logique existante, pour laquelle le chiffrement nœud à nœud est activé. Puisque la définition du mode de chiffrement implique d’ajouter manuellement des certificats, l’installation rapide n’est plus une option. 
- Pour effectuer une copie de fichiers à partir d’un environnement local, vous ne devez plus utiliser \\localhost ni C:\fichiers car localhost ou le lecteur local ne sont peut-être pas accessibles par le biais de tous les nœuds. Utilisez plutôt \\NomServeur\fichiers pour spécifier l’emplacement des fichiers.


## <a name="rolling-back-from-the-preview"></a>Restauration à partir de la préversion 
Pour effectuer une restauration à partir de la préversion, supprimez tous les nœuds, sauf un. Les nœuds que vous supprimez importent peu, mais vérifiez que vous avez au moins un nœud dans la passerelle logique. Vous pouvez supprimer un nœud en désinstallant la passerelle sur l’ordinateur ou en utilisant le portail Azure. Dans le portail Azure, dans la page **Data Factory**, cliquez sur Services liés pour lancer la page **Services liés**. Sélectionnez la passerelle pour lancer la page **Passerelle**. Dans la page Passerelle, vous pouvez voir les nœuds associés à la passerelle. La page vous permet de supprimer un nœud de la passerelle.
 
Après la suppression, cliquez sur **Fonctionnalités de préversion** dans la même page du portail Azure et désactivez la fonctionnalité de préversion. Vous avez réinitialisé votre passerelle qui redevient alors une passerelle à nœud unique à disponibilité générale.


## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants :
- [Passerelle de gestion des données](data-factory-data-management-gateway.md) - fournit une vue d’ensemble descriptive de la passerelle.
- [Déplacer des données entre des magasins de données locaux et dans le cloud](data-factory-move-data-between-onprem-and-cloud.md) - contient une procédure pas à pas donnant des instructions sur l’utilisation d’une passerelle à nœud unique. 
