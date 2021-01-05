---
title: Créer un cluster Service Fabric dans le portail Azure
description: Apprenez à configurer un cluster Service Fabric sécurisé dans Azure à l’aide du portail Azure et d’Azure Key Vault.
ms.topic: conceptual
ms.date: 09/06/2018
ms.openlocfilehash: c679a804db09b1034f31e9d8da1f7d2ad206f684
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90563724"
---
# <a name="create-a-service-fabric-cluster-in-azure-using-the-azure-portal"></a>Création d’un cluster Service Fabric dans Azure à partir du portail Azure
> [!div class="op_single_selector"]
> * [Azure Resource Manager](service-fabric-cluster-creation-via-arm.md)
> * [Azure portal](service-fabric-cluster-creation-via-portal.md)
> 
> 

Ce guide vous mène pas à pas à travers les étapes de configuration d’un cluster Service Fabric (Linux ou Windows) dans Azure à l’aide du portail Azure. Il vous permet de vous familiariser avec les procédures suivantes :

* Créez un cluster dans le portail Azure.
* authentification d’administrateurs à l’aide de certificat.

> [!NOTE]
> Pour accéder à des options de sécurité plus avancées, telles que l’authentification des utilisateurs avec Azure Active Directory et la configuration de certificats pour la sécurité des applications, [créez votre cluster à l’aide d’Azure Resource Manager][create-cluster-arm].
> 
> 

## <a name="cluster-security"></a>Sécurité des clusters 
Les certificats sont utilisés dans Service Fabric à des fins d’authentification et de chiffrement pour sécuriser les divers aspects d’un cluster et de ses applications. Pour plus d’informations sur l’utilisation de certificats dans Service Fabric, consultez la page [Scénarios de sécurité d’un cluster Service Fabric][service-fabric-cluster-security].

Si c’est la première fois que vous créez un cluster Service Fabric ou que vous déployez un cluster pour des charges de travail de test, vous pouvez passer à la section suivante (**Créer un cluster dans le portail Azure**) et configurer le système pour qu’il génère les certificats requis pour vos clusters qui exécutent des charges de travail de test. Si vous configurez un cluster pour des charges de production, continuez à lire.

#### <a name="cluster-and-server-certificate-required"></a>Certificat de cluster et de serveur (obligatoire)
Ce certificat est nécessaire pour sécuriser un cluster et empêcher un accès non autorisé à ce dernier. Il assure la sécurité du cluster de différentes manières :

* **Authentification du cluster :** authentifie la communication nœud à nœud pour la fédération du cluster. Seuls les nœuds qui peuvent prouver leur identité avec ce certificat peuvent être ajoutés au cluster.
* **Authentification du serveur :** authentifie les points de terminaison de gestion du cluster sur un client de gestion, afin que le client de gestion sache qu'il communique avec le véritable cluster. Ce certificat fournit également un protocole TLS pour l’API de gestion HTTPS et pour Service Fabric Explorer via HTTPS.

Pour cela, le certificat doit répondre aux exigences suivantes :

* Le certificat doit contenir une clé privée.
* Le certificat doit être créé pour l'échange de clés et pouvoir faire l'objet d'un export au format Personal Information Exchange (.pfx).
* Le **nom d'objet du certificat doit correspondre au domaine** servant à accéder au cluster Service Fabric. Cela est nécessaire pour fournir un protocole TLS pour les points de terminaison de gestion HTTPS du cluster et Service Fabric Explorer. Vous ne pouvez pas obtenir de certificat TLS/SSL auprès d’une autorité de certification pour le domaine `.cloudapp.azure.com`. Vous devez obtenir un nom de domaine personnalisé pour votre cluster. Lorsque vous demandez un certificat auprès d'une autorité de certification, le nom d'objet du certificat doit correspondre au nom de domaine personnalisé que vous utilisez pour votre cluster.

#### <a name="client-authentication-certificates"></a>Authentification de certificat client
Les certificats client supplémentaires authentifient les administrateurs pour les tâches de gestion de cluster. Service Fabric dispose de deux niveaux d’accès : **admin** et **read-only user**. Au minimum, un seul certificat pour un accès administrateur doit être utilisé. Pour un accès de niveau utilisateur supplémentaire, un certificat distinct doit être fourni. Pour en savoir plus sur les rôles d’accès, consultez la page [Contrôle d’accès en fonction du rôle pour les clients de Service Fabric][service-fabric-cluster-security-roles].

Vous n’avez pas besoin de charger les certificats d’authentification client dans le Key Vault pour pouvoir travailler avec Service Fabric. Il suffit de fournir ces certificats aux utilisateurs qui sont autorisés à effectuer la gestion des clusters. 

> [!NOTE]
> Azure Active Directory est la méthode recommandée pour authentifier les clients pour des opérations de gestion de cluster. Pour utiliser Azure Active Directory, vous devez [créer un cluster à l’aide d’Azure Resource Manager][create-cluster-arm].
> 
> 

#### <a name="application-certificates-optional"></a>Certificats d’application (facultatif)
Un nombre quelconque de certificats supplémentaires peut être installé sur un cluster pour sécuriser une application. Avant de créer votre cluster, examinez les scénarios de sécurité d’application qui nécessitent l’installation d’un certificat sur les nœuds, notamment :

* Le chiffrement et déchiffrement de valeurs de configuration d’applications.
* Le chiffrement des données sur les nœuds lors de la réplication. 

Les certificats d’application ne peuvent pas être configurés lors de la [création d’un cluster via le portail Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/service-fabric/service-fabric-cluster-creation-via-portal.md). Pour configurer les certificats d’application au moment de la configuration du cluster, vous devez [créer un cluster à l’aide d’Azure Resource Manager][create-cluster-arm]. Vous pouvez également ajouter des certificats d’application au cluster après sa création.

## <a name="create-cluster-in-the-azure-portal"></a>Création d’un cluster dans le portail Azure

La création d’un cluster de production pour répondre aux besoins de votre application implique une planification. Pour faciliter cela, il est fortement recommandé de lire et de comprendre le document [Considérations de planification pour les clusters Service Fabric][service-fabric-cluster-capacity]. 

### <a name="search-for-the-service-fabric-cluster-resource"></a>Rechercher la ressource de cluster Service Fabric

Connectez-vous au [portail Azure][azure-portal].
Cliquez sur **Créer une ressource** pour ajouter un nouveau modèle de ressource. Recherchez le modèle de cluster Service Fabric dans **Marketplace** sous **Tout**.
Sélectionnez **Cluster Service Fabric** dans la liste.

![Rechercher le modèle de cluster Service Fabric sur le portail Azure.][SearchforServiceFabricClusterTemplate]

Accédez au panneau **Cluster Service Fabric**, puis cliquez sur **Créer**.

Le panneau **Créer un cluster Service Fabric** inclut les quatre étapes suivantes :

### <a name="1-basics"></a>1. Concepts de base
![Capture d’écran de la création d’un groupe de ressources.][CreateRG]

Dans le volet De base, vous devez fournir les informations de base de votre cluster.

1. Entrez le nom de votre cluster.
2. Choisissez un **Nom d’utilisateur** et un **Mot de passe** pour le bureau à distance des machines virtuelles.
3. Veillez à sélectionner **l’Abonnement** sur lequel vous souhaitez que votre cluster soit déployé, surtout si vous avez plusieurs abonnements.
4. Créez un **groupe de ressources**. Il est préférable de lui attribuer le même nom que le cluster, car cela vous permet de le retrouver plus facilement, surtout lorsque vous essayez d’apporter des modifications à votre déploiement ou de supprimer votre cluster.
   
   > [!NOTE]
   > Bien que vous puissiez décider d’utiliser un groupe de ressources existant, nous vous conseillons de créer un groupe de ressources. Cela facilite la suppression des clusters et des ressources qu’ils utilisent.
   > 
   > 
5. Sélectionnez l’**emplacement** dans lequel vous souhaitez créer le cluster. Si vous envisagez d’utiliser un certificat existant que vous avez déjà téléchargé vers un coffre de clés, vous devez utiliser la même région que celle figurant dans le coffre de clés. 

### <a name="2-cluster-configuration"></a>2. Configuration de clusters
![Création d’un type de nœud][CreateNodeType]

Configurez vos nœuds de cluster. Les types de nœuds définissent les tailles de machine virtuelle, le nombre de machines virtuelles et leurs propriétés. Votre cluster peut avoir plusieurs types de nœud, mais le type de nœud principal (le premier que vous définissez sur le portail) doit avoir au moins cinq machines virtuelles, car il s’agit du type de nœud dans lequel les services du système Service Fabric sont placés. Ne configurez pas la zone **Propriétés de sélection élective** , car une propriété de sélection élective par défaut de « NodeTypeName » est ajoutée automatiquement.

> [!NOTE]
> Un scénario répandu si vous avez plusieurs types de nœuds est une application qui contient un service front-end et un service back-end. Vous souhaitez placer le service front-end sur des machines virtuelles plus petites (tailles de machine virtuelle de type D2_V2) avec leurs ports ouverts à Internet, et vous souhaitez placer le service back-end sur des machines virtuelles de plus grandes tailles (dont les tailles sont de type D3_V2, D6_V2, D15_V2 etc.) dont les ports ne sont pas accessibles sur Internet.
> 

1. Choisissez un nom pour votre type de nœud (1 à 12 caractères contenant uniquement des lettres et des chiffres).
2. La **taille** minimale des machines virtuelles pour le type de nœud principal dépend de la **couche de durabilité** que vous choisissez pour le cluster. La valeur par défaut du niveau de durabilité est Bronze. Pour en savoir plus sur la durabilité, consultez [Comment choisir la durabilité du cluster Service Fabric][service-fabric-cluster-durability].
3. Sélectionnez la **taille de la machine virtuelle**. Les machines virtuelles de série D ont des lecteurs de disques SSD et sont vivement recommandées pour les applications avec état. N’utilisez pas les références de machine virtuelle incluant des cœurs partiels ou présentant une capacité de disque disponible inférieure à 10 Go. Reportez-vous au document [Considérations de planification pour les clusters Service Fabric][service-fabric-cluster-capacity] pour obtenir de l’aide concernant la sélection de la taille de la machine virtuelle.
4.  **Le cluster à un seul nœud et les clusters à trois nœuds** ne servent qu’à effectuer des tests. Ils ne sont pas pris en charge pour l’exécution des charges de travail de production.
5. Choisissez la **capacité du groupe identique de machines virtuelles initiale** pour le type de nœud. Vous pouvez effectuer un scale-in ou un scale-out du nombre de machines virtuelles d’un type de nœud. Toutefois, pour le type de nœud principal, le minimum est de cinq pour les charges de travail de production. Les autres types de nœuds peuvent avoir au minimum une machine virtuelle. Le **nombre** minimal de machines virtuelles du type de nœud principal détermine la **fiabilité** de votre cluster.  
6. Configurez des **points de terminaison personnalisés**. Ce champ vous permet d’entrer une liste séparée par des virgules des ports que vous souhaitez exposer par le biais de l’Azure Load Balancer à l’Internet public pour vos applications. Par exemple, si vous envisagez de déployer une application web dans votre cluster, saisissez « 80 » pour autoriser le trafic sur le port 80 dans votre cluster. Pour en savoir plus sur les points de terminaison, consultez la page [Communiquer avec des applications][service-fabric-connect-and-communicate-with-services].
7. **Activez un proxy inverse**.  Le [proxy inverse Service Fabric](service-fabric-reverseproxy.md) permet aux microservices exécutés dans un cluster Service Fabric de découvrir d’autres services dotés de points de terminaison http et de communiquer avec ces services.
8. De retour dans le panneau **Configuration du Cluster**, sous **+Afficher les paramètres facultatifs**, configurer le cluster **diagnostics**. Par défaut, les diagnostics sont activés sur votre cluster afin de faciliter la résolution des problèmes. Si vous souhaitez désactiver les diagnostics, définissez **l’État** sur **Désactivé**. Nous vous recommandons de **ne pas** désactiver les diagnostics. Si vous avez déjà créé le projet Application Insights, donnez sa clé, afin que les traces d’application soient acheminées vers lui.
9. **Incluez le service DNS**.  Le [service DNS](service-fabric-dnsservice.md) est un service facultatif qui vous permet de trouver d’autres services en utilisant le protocole DNS.
10. Sélectionnez le **mode de mise à niveau Fabric** que vous souhaitez associer à votre cluster. Sélectionnez **Automatique**si vous souhaitez que le système récupère automatiquement la dernière version disponible et essaye de mettre à niveau votre cluster vers cette version. Définissez le mode sur **Manuel**si vous souhaitez choisir une version prise en charge. Pour en savoir plus sur le mode de mise à niveau de Service Fabric, consultez le document [Mettre à niveau un cluster Service Fabric][service-fabric-cluster-upgrade].

> [!NOTE]
> Nous prenons uniquement en charge les clusters qui exécutent des versions prises en charge de Service Fabric. Si vous sélectionnez le mode **Manuel** , vous êtes responsable de la mise à niveau de votre cluster vers une version prise en charge.
> 

### <a name="3-security"></a>3. Sécurité
![Capture d’écran des configurations de sécurité sur le portail Azure.][BasicSecurityConfigs]

Pour faciliter la configuration d’un cluster de test sécurisé pour vous, nous vous avons fourni l’option **De base**. Si vous disposez déjà d’un certificat et que vous l’avez téléchargé dans votre [coffre de clés](../key-vault/index.yml) (et activé le coffre de clés pour le déploiement), utilisez l’option **Personnalisé**.

#### <a name="basic-option"></a>Option de base
Suivez les écrans suivants pour ajouter ou réutiliser un coffre de clés existant et ajouter un certificat. L’ajout du certificat est un processus synchrone, et par conséquent, vous devez attendre que le certificat soit créé.

Résistez à la tentation de quitter l’écran jusqu'à la fin de la procédure précédente.

![La capture d’écran montre la page Sécurité avec l’option De base sélectionnée ainsi que le volet Coffre de clés et le volet Créer un coffre de clés.][CreateKeyVault]

Maintenant que le coffre de clés est créé, modifiez les stratégies d’accès pour votre coffre de clés. 

![La capture d’écran montre le volet Créer un cluster Service Fabric avec l’option 3, Sécurité, sélectionnée et une explication indiquant que le coffre de clés n’est pas activé.][CreateKeyVault2]

Cliquez sur **Modifier les stratégies d'accès**, puis sur **Afficher les stratégies d'accès avancé** et activez l’accès aux machines virtuelles Azure pour le déploiement. Il est recommandé d’activer aussi le déploiement du modèle. Une fois que vous avez effectué vos sélections, n’oubliez pas de cliquer sur le bouton **Enregistrer** et de fermer le panneau **Stratégies d’accès**.

![La capture d’écran montre le volet Créer un cluster Service Fabric avec le volet Sécurité ouvert et le volet Stratégies d’accès ouvert.][CreateKeyVault3]

Entrez le nom du certificat, puis cliquez sur **OK**.

![La capture d’écran montre le volet Créer un cluster Service Fabric avec l’option Sécurité sélectionnée comme avant, mais sans l’explication indiquant que le coffre de clés n’est pas activé.][CreateKeyVault4]

#### <a name="custom-option"></a>Option Personnalisé
Ignorez cette section si vous avez déjà effectué les étapes décrites dans l’option **De base**.

![La capture d’écran montre la boîte de dialogue Sécurité, Configurer les paramètres de sécurité du cluster.][SecurityCustomOption]

Vous aurez besoin du coffre de clés source, de l’URL du certificat et des informations relatives à l’empreinte du certificat pour la page de sécurité. Si vous ne les avez pas sous la main, ouvrez une autre fenêtre de navigateur et effectuez les opérations suivantes dans le portail Azure

1. Accédez à votre service de coffre de clés.
2. Sélectionnez l’onglet « Propriétés » et copiez l’ID de ressource pour « Coffre de clés source » sur l’autre fenêtre de navigateur 

    ![La capture d’écran montre la fenêtre Propriétés pour le coffre de clés.][CertInfo0]

3. Sélectionnez maintenant l’onglet Certificats.
4. Cliquez sur l’empreinte numérique du certificat, ce qui vous redirige vers la page Versions.
5. Cliquez sur les GUID que vous voyez sous la version actuelle.

    ![La capture d’écran montre la fenêtre Certificats pour le coffre de clés.][CertInfo1]

6. Vous devez maintenant sur l’écran ci-dessous. Copiez l’empreinte numérique hexadécimale SHA-1 dans « Empreinte numérique du certificat » sur l’autre fenêtre de navigateur
7. Copiez l’Identificateur Secret' pour l’« URL du certificat » dans l’autre fenêtre de navigateur.

    ![La capture d’écran montre la boîte de dialogue Version du certificat avec une option permettant de copier l’identificateur de certificat.][CertInfo2]

Activez la case **Configurer les paramètres avancés** pour saisir les certificats clients pour le **Client d’administration** et le **Client en lecture seule**. Dans ces champs, saisissez l’empreinte de votre certificat de client d’administration et l’empreinte de votre certificat de client en lecture seule, le cas échéant. Lorsque les administrateurs tentent de se connecter au cluster, ils se voient attribuer l’accès uniquement s’ils disposent d’un certificat avec une empreinte qui correspond aux valeurs entrées ici.  

### <a name="4-summary"></a>4. Résumé

Vous êtes maintenant prêt à déployer le cluster. Avant cela, téléchargez le certificat, regardez à l’intérieur de la grande zone bleue d’informations pour le lien. Veillez à conserver le certificat en lieu sûr. Vous en aurez besoin pour vous connecter à votre cluster. Étant donné que le certificat que vous avez téléchargé n’a pas de mot de passe, il est recommandé que vous en ajoutiez un.

Pour terminer la création du cluster, cliquez sur **Créer**. Vous pouvez éventuellement télécharger le modèle.

![La capture d’écran montre la page Résumé du volet Créer un cluster Service Fabric avec un lien permettant d’afficher et de télécharger un certificat.][Summary]

Vous pouvez voir la progression de la création dans les notifications. (Cliquez sur l’icône représentant une cloche près de la barre d’état dans l’angle supérieur droit de votre écran.) Si vous avez cliqué sur **Épingler au tableau d’accueil** pendant la création du cluster, **Déploiement du cluster Service Fabric** apparaît épinglé au **tableau d’accueil**. Ce processus prend un certain temps. 

Pour effectuer des opérations de gestion sur votre cluster à l’aide de PowerShell ou CLI, vous devez vous connecter à votre cluster. En savoir plus sur la façon de [vous connecter à votre cluster](service-fabric-connect-to-secure-cluster.md).

## <a name="view-your-cluster-status"></a>Afficher l’état de votre cluster
![Capture d’écran des détails du cluster dans le tableau de bord.][ClusterDashboard]

Une fois votre cluster créé, vous pouvez l’inspecter dans le portail :

1. Accédez à **Parcourir**, puis cliquez sur **Clusters Service Fabric**.
2. Recherchez votre cluster et cliquez dessus.
3. Vous pouvez maintenant voir les détails de votre cluster dans le tableau de bord, notamment le point de terminaison du cluster et un lien vers Service Fabric Explorer.

La section **Surveillance des nœuds** du panneau du tableau de bord du cluster indique le nombre de machines virtuelles intègres et de machines virtuelles non intègres. Pour plus d’informations sur l’intégrité du cluster, consultez [Présentation du modèle d’intégrité de Service Fabric][service-fabric-health-introduction].

> [!NOTE]
> Les clusters Service Fabric nécessitent un certain nombre de nœuds actifs en permanence pour maintenir la disponibilité et préserver l’état. Cette situation est appelée « conservation du quorum ». Ainsi, il est généralement déconseillé d’arrêter tous les ordinateurs du cluster, sauf si vous avez d’abord effectué une [sauvegarde complète de votre état][service-fabric-reliable-services-backup-restore].
> 
> 

## <a name="remote-connect-to-a-virtual-machine-scale-set-instance-or-a-cluster-node"></a>Connexion distante à une instance de jeu de mise à l’échelle de machine virtuelle ou à un nœud de cluster
Chacune des valeurs NodeTypes que vous spécifiez dans votre cluster entraîne la configuration d’un groupe de machines virtuelles identiques. <!--See [Remote connect to a Virtual Machine Scale Set instance][remote-connect-to-a-vm-scale-set] for details. -->

## <a name="next-steps"></a>Étapes suivantes
À ce stade, vous avez un cluster sécurisé à l’aide de certificats pour l’authentification de la gestion. Ensuite, [connectez-vous à votre cluster](service-fabric-connect-to-secure-cluster.md) et découvrez comment [gérer les secrets d’application](service-fabric-application-secret-management.md).  Découvrez également les [options de support de Service Fabric](service-fabric-support.md).

<!-- Links -->
[azure-powershell]: /powershell/azure/
[azure-portal]: https://portal.azure.com/
[key-vault-get-started]: ../key-vault/general/overview.md
[create-cluster-arm]: service-fabric-cluster-creation-via-arm.md
[service-fabric-cluster-security]: service-fabric-cluster-security.md
[service-fabric-cluster-security-roles]: service-fabric-cluster-security-roles.md
[service-fabric-cluster-capacity]: service-fabric-cluster-capacity.md
[service-fabric-cluster-durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
[service-fabric-connect-and-communicate-with-services]: service-fabric-connect-and-communicate-with-services.md
[service-fabric-health-introduction]: service-fabric-health-introduction.md
[service-fabric-reliable-services-backup-restore]: service-fabric-reliable-services-backup-restore.md
[remote-connect-to-a-vm-scale-set]: service-fabric-cluster-nodetypes.md
[service-fabric-cluster-upgrade]: service-fabric-cluster-upgrade.md

<!--Image references-->
[SearchforServiceFabricClusterTemplate]: ./media/service-fabric-cluster-creation-via-portal/SearchforServiceFabricClusterTemplate.png
[CreateRG]: ./media/service-fabric-cluster-creation-via-portal/CreateRG.png
[CreateNodeType]: ./media/service-fabric-cluster-creation-via-portal/NodeType.png
[BasicSecurityConfigs]: ./media/service-fabric-cluster-creation-via-portal/BasicSecurityConfigs.PNG
[CreateKeyVault]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault.PNG
[CreateKeyVault2]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault2.PNG
[CreateKeyVault3]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault3.PNG
[CreateKeyVault4]: ./media/service-fabric-cluster-creation-via-portal/CreateKeyVault4.PNG
[CertInfo0]: ./media/service-fabric-cluster-creation-via-portal/CertInfo0.PNG
[CertInfo1]: ./media/service-fabric-cluster-creation-via-portal/CertInfo1.PNG
[CertInfo2]: ./media/service-fabric-cluster-creation-via-portal/CertInfo2.PNG
[SecurityCustomOption]: ./media/service-fabric-cluster-creation-via-portal/SecurityCustomOption.PNG
[DownloadCert]: ./media/service-fabric-cluster-creation-via-portal/DownloadCert.PNG
[Summary]: ./media/service-fabric-cluster-creation-via-portal/Summary.PNG
[SecurityConfigs]: ./media/service-fabric-cluster-creation-via-portal/SecurityConfigs.png
[Notifications]: ./media/service-fabric-cluster-creation-via-portal/notifications.png
[ClusterDashboard]: ./media/service-fabric-cluster-creation-via-portal/ClusterDashboard.png
[cluster-security-cert-installation]: ./media/service-fabric-cluster-creation-via-arm/cluster-security-cert-installation.png
