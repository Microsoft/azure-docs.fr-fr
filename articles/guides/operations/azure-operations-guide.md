---
title: Guide de prise en main pour les opérateurs informatiques Azure | Microsoft Docs
description: Guide de prise en main pour les opérateurs informatiques Azure
author: RicksterCDN
ms.author: rclaus
tags: azure-resource-manager
ms.service: azure
ms.topic: overview
ms.workload: infrastructure
ms.date: 08/24/2018
ms.openlocfilehash: 7b9e95c06dd48e78e42244d27d27d063bf5f0be7
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92327771"
---
# <a name="get-started-for-azure-it-operators"></a>Prise en main pour les opérateurs informatiques Azure

Ce guide présente les principaux concepts associés au déploiement et à la gestion d’une infrastructure Microsoft Azure. Si vous débutez dans le cloud computing ou ne connaissez pas encore Azure, ce guide vous aide à vous familiariser rapidement avec les concepts, le déploiement et les détails de gestion. De nombreuses sections de ce guide présentent des activités, comme le déploiement d’une machine virtuelle, qu’elles accompagnent de liens vers des articles techniques approfondis et détaillés.

## <a name="cloud-computing-overview"></a>Présentation du cloud computing

Le cloud computing offre une alternative moderne aux centres de données locaux classiques. Les fournisseurs de cloud public proposent et gèrent toute l’infrastructure informatique et ses logiciels de gestion sous-jacents. Ils offrent un large éventail de services cloud. En l’occurrence, un service cloud peut être une machine virtuelle, un serveur web ou un moteur de base de données hébergé dans le cloud. En tant que client auprès d’un prestataire de cloud, vous louez des services cloud en fonction de vos besoins. Ce faisant, vous convertissez les charges d’immobilisation de la maintenance matérielle en charge de fonctionnement. Un service cloud fournit également les avantages suivants :

- Déploiement rapide de grands environnements de calcul

- Désallocation rapide des systèmes qui ne sont plus nécessaires

- Déploiement simple des systèmes habituellement complexes, tels que les équilibreurs de charge

- Capacité à fournir une capacité de calcul ou une mise à l’échelle souples, au besoin

- Environnements informatiques plus rentables

- Accès où que soyez par le biais d’un portail web ou d’une automatisation par programmation

- Services cloud pour répondre à la plupart des besoins en calcul et en application

Avec l’infrastructure locale, vous disposez d’un contrôle total sur le matériel et les logiciels qui sont déployés. Depuis toujours, cela a conduit à des décisions d’approvisionnement en matériel qui se polarisent sur l’évolution verticale (« scaling up »). Prenons l’exemple de l’achat d’un serveur équipé de cœurs supplémentaires pour répondre aux besoins de performances de pointe. Cette infrastructure, malheureusement, peut être sous-utilisée en dehors de cette fenêtre de forte demande. Avec Azure, vous déployez uniquement l’infrastructure qui vous est nécessaire, en l’adaptant à tout moment en fonction de vos périodes hautes ou basses. Cet état de fait mène à privilégier une montée en puissance parallèle (« scaling out ») en déployant des nœuds de calcul supplémentaires pour répondre à un besoin de performances. La montée en puissance parallèle des services cloud est plus économique que l’évolution verticale qui implique l’acquisition d’un matériel coûteux.

Microsoft a déployé de nombreux centres de données Azure dans le monde entier et en a déjà prévu d’autres. Microsoft augmente aussi le nombre de clouds souverains dans des régions telles que la Chine et l’Allemagne. Seules les plus grandes entreprises mondiales peuvent ainsi déployer des centres de données, c’est la raison pour laquelle l’utilisation d’Azure permet aux entreprises de toutes tailles de déployer des services proches de leurs clients.

Aux petites entreprises, Azure propose un point d’entrée à bas coût, avec une mise à l’échelle rapide à mesure que la demande en capacité de calcul augmente. Un investissement initial important est alors évité au niveau de l’infrastructure tandis que la souplesse nécessaire est assurée pour concevoir et modifier en fonction des besoins l’architecture des systèmes. L’utilisation du cloud computing convient bien au modèle de croissance et de décroissance rapides (« scale-fast and Fail-fast ») d’une start-up.

Pour plus d’informations sur les régions Azure disponibles, consultez [Régions Azure](https://azure.microsoft.com/regions/).

### <a name="cloud-computing-model"></a>Modèle de cloud computing

Azure utilise un modèle de cloud computing basé sur les catégories de service fournies aux clients. Les trois catégories de service incluent l’IaaS (infrastructure as a service), le PaaS (platform as a service) et le SaaS (software as a service). Les fournisseurs partagent tout ou partie de la responsabilité pour les composants dans la pile informatique dans chacune de ces catégories. Jetons un œil à chacune des catégories pour le cloud computing.
![Comparaison de pile de cloud computing](./media/cloud-computing-comparison.png)

#### <a name="iaas-infrastructure-as-a-service"></a>IaaS : Infrastructure as a service

Un fournisseur de cloud IaaS exécute et gère toutes les ressources de calcul physiques ainsi que les logiciels nécessaires à la virtualisation de l’ordinateur. Un client de ce service déploie les machines virtuelles dans ces centres de données hébergés. Même si les machines virtuelles sont situées dans un centre de données hors site, le consommateur IaaS garde le contrôle sur la configuration et la gestion du système d’exploitation laissant l’infrastructure sous-jacente au fournisseur de cloud.

Azure comprend plusieurs solutions IaaS, notamment les machines virtuelles, les groupes de machines virtuelles identiques et l’infrastructure de réseau associée. Le choix des machines virtuelles est largement plébiscité lorsqu’il s’agit au départ de faire migrer des services sur Azure, car il permet un modèle de migration « lift-and-shift » qui se résume à un simple déplacement. Vous pouvez configurer une machine virtuelle, comme l’infrastructure exécutant vos services dans votre centre de données, et ensuite migrer vos logiciels sur la nouvelle machine virtuelle. Des mises à jour de configuration peuvent s’avérer nécessaires, telles que les URL vers d’autres services ou stockages, mais vous pouvez migrer beaucoup d’applications de cette façon-là.

Les groupes de machines virtuelles identiques reposent sur des machines virtuelles Microsoft Azure et offrent un moyen facile de déployer des clusters de machines virtuelles identiques. Les groupes de machines virtuelles identiques prennent également en charge la mise à l’échelle automatique afin de pouvoir déployer les nouvelles machines virtuelles automatiquement à la demande. Les groupes de machines virtuelles identiques offrent une plateforme idéale pour héberger les clusters de calcul de microservice plus importants, tels que Azure Service Fabric et Azure Container Service.

#### <a name="paas-platform-as-a-service"></a>PaaS : Platform as a service

Avec PaaS, vous déployez votre application dans un environnement qui est fourni par le prestataire de services cloud. Le fournisseur s’occupe de toute la gestion de l’infrastructure pour vous permettre de vous consacrer au développement d’applications et à la gestion de données.

Azure fournit plusieurs offres de calcul PaaS, notamment le composant Web Apps d’Azure App Service et d’Azure Cloud Services (rôles web et de travail). Dans un cas comme dans l’autre, les développeurs ont à leur disposition plusieurs moyens de déployer leur application sans avoir à se préoccuper de l’infrastructure qui les prend en charge. Les développeurs n’ont aucun besoin de créer des machines virtuelles, ni d’utiliser le protocole RDP (Remote Desktop Protocol) pour se connecter à chacune d’elles, ni même d’installer l’application. Ils appuient simplement sur un bouton (ou peu s’en faut) et les outils fournis par Microsoft approvisionnent les machines virtuelles, puis déploient et installent l’application sur celles-ci.

#### <a name="saas-software-as-a-service"></a>SaaS : Software as a service

SaaS est un logiciel qui est hébergé et géré de manière centralisée. Il s’appuie généralement sur une architecture multilocataire, où une seule version de l’application est utilisée pour tous les clients. Il peut être monté en charge parallèle sur plusieurs instances pour garantir des performances optimales dans tous les emplacements. Le logiciel SaaS est généralement concédé sous licence, par abonnement mensuel ou annuel. Les éditeurs de logiciels SaaS sont responsables de tous les composants de la pile logicielle afin de vous faire gérer uniquement les services fournis.

Microsoft 365 constitue un bon exemple d’offre SaaS. Les abonnés payent un abonnement mensuel ou annuel pour obtenir Microsoft Exchange, Microsoft OneDrive et le reste de la suite Microsoft Office en tant que service. Les abonnés bénéficient toujours de la version la plus récente et n’ont pas besoin de gérer le serveur Exchange. Par rapport à l’installation et à la mise à jour annuelle d’Office, cette solution est moins onéreuse et demande moins d’efforts.

## <a name="azure-services"></a>Services Azure

Azure met de nombreux services à disposition dans sa plateforme de cloud computing. Ces services sont les suivants :

### <a name="compute-services"></a>Services de calcul

Services d’hébergement et d’exécution de la charge de travail des applications :

- Machines virtuelles Azure pour Linux et Windows

- App Services (Web Apps, Mobile Apps, Logic Apps, API Apps et Function Apps)

- Azure Batch (pour les tâches de calcul en lot et en parallèle à grande échelle)

- Azure Service Fabric

- Azure Container Service

### <a name="data-services"></a>Services de données

Services de stockage et de gestion des données :

- Stockage Azure (comprend les services Stockage Blob, Stockage File d’attente, Stockage Table et Stockage Fichier)

- Azure SQL Database

- Azure Cosmos DB

- Microsoft Azure StorSimple

- Cache Azure pour Redis

### <a name="application-services"></a>Services d’application

Services de création et de fonctionnement des applications :

- Azure Active Directory (Azure AD)

- Azure Service Bus pour la connexion des systèmes distribués

- Azure HDInsight pour le traitement de big data

- Azure Logic Apps pour les workflows d’intégration et d’orchestration

- Azure Media Services

### <a name="network-services"></a>Services réseau

Services de mise en réseau au sein d’Azure et entre Azure et les centres de données locaux :

- Réseau virtuel Azure

- Azure ExpressRoute

- Service DNS fourni par Azure

- Azure Traffic Manager

- Azure Content Delivery Network

Pour obtenir une documentation détaillée des services Azure, consultez [Documentation des services Azure](/azure).

## <a name="azure-key-concepts"></a>Principaux concepts Azure

### <a name="datacenters-and-regions"></a>Centres de données et régions

Azure est une plateforme cloud mondiale, disponible dans de nombreuses régions du monde. Lorsque vous configurez un service, une application ou une machine virtuelle dans Azure, vous êtes invité à sélectionner une région. La région sélectionnée représente un centre de données spécifique dans lequel votre application s’exécute. Pour plus d’informations, consultez [Régions Azure](https://azure.microsoft.com/regions/).

L’un des avantages à utiliser Azure est de pouvoir déployer des applications dans différents centres de données du monde. La région que vous choisissez peut affecter les performances de votre application. Il est préférable de choisir une région plus proche de la majorité de vos clients pour réduire la latence des demandes réseau. Vous pouvez également sélectionner une région pour respecter les exigences légales de distribution de votre application dans certains pays et certaines régions.

### <a name="azure-portal"></a>Portail Azure

Le portail Azure est une application web qui peut être utilisée pour créer, gérer et supprimer des services et des ressources Azure. Le portail Azure se trouve à l’adresse [portal.azure.com](https://portal.azure.com). Il comprend un tableau de bord personnalisable et des outils de gestion des ressources Azure. Il fournit également des informations de facturation et d’abonnement. Pour plus d’informations, consultez [Présentation du portail Microsoft Azure](https://azure.microsoft.com/documentation/articles/azure-portal-overview/) et [Gérer des ressources Azure sur le portail](../../azure-resource-manager/management/manage-resources-portal.md).

### <a name="resources"></a>Ressources

Les ressources Azure se présentent sous la forme de services individuels de calcul, de mise en réseau, de données ou d’hébergement d’applications qui sont déployés dans un abonnement Azure. Les ressources parmi les plus courantes se retrouvent dans les machines virtuelles, les comptes de stockage ou les bases de données SQL. Les services Azure comportent souvent plusieurs ressources Azure connexes. Par exemple, une machine virtuelle Azure peut inclure une machine virtuelle, un compte de stockage, une carte réseau et une adresse IP publique. Ces ressources peuvent être créées, gérées et supprimées individuellement ou en tant que groupe. Les ressources Azure sont détaillées plus loin dans ce guide.

### <a name="resource-groups"></a>Groupes de ressources

Un groupe de ressources Azure est un conteneur réunissant les ressources associées d’une solution Azure. Le groupe de ressources peut inclure toutes les ressources de la solution, ou uniquement celles que vous souhaitez gérer en tant que groupe. Les groupes de ressources Azure sont détaillés plus loin dans ce guide.

### <a name="resource-manager-templates"></a>Modèles Resource Manager

Un modèle Resource Manager est un fichier JSON (JavaScript Objet Notation) qui définit une ou plusieurs ressources à déployer sur un groupe de ressources. Il définit également les dépendances entre les ressources déployées. Les modèles Resource Manager sont détaillés plus loin dans ce guide.

### <a name="automation"></a>Automatisation

En plus de créer, gérer et supprimer des ressources sur le portail Azure, vous pouvez automatiser ces activités à l’aide de PowerShell ou de l’interface de ligne de commande (CLI) Azure.

#### <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell est un ensemble de modules fournissant des applets de commande pour gérer Azure. Vous pouvez utiliser les applets de commande pour créer, gérer et supprimer des services Azure. Les applets de commande vous permettent de réaliser des déploiements cohérents et reproductibles sans avoir à intervenir. Pour plus d’informations, consultez [Installer et configurer Azure PowerShell](/powershell/azure/install-Az-ps).

#### <a name="azure-command-line-interface"></a>Interface de ligne de commande Azure

L’interface de ligne de commande Azure est un outil que vous pouvez utiliser pour créer, gérer et supprimer des ressources Azure à partir de la ligne de commande. Azure CLI, l’interface de ligne de commande Azure, est disponible pour Linux, Mac OS X et Windows. Pour obtenir plus d’informations et de détails techniques, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

#### <a name="rest-apis"></a>API REST

Azure repose sur un ensemble d’API REST qui prennent en charge l’interface utilisateur du portail Azure. La plupart de ces API REST sont également prises en charge pour vous permettre de configurer et de gérer par programmation vos ressources et applications Azure à partir de n’importe quel appareil connecté à Internet. Pour plus d’informations, consultez [Informations de référence sur les API REST Azure](/rest/api/index).

### <a name="azure-cloud-shell"></a>Azure Cloud Shell

Les administrateurs peuvent accéder à Azure PowerShell et Azure CLI par l’intermédiaire d’une expérience appelée Azure Cloud Shell et accessible via le navigateur. Cette interface interactive fournit un outil flexible pour que les administrateurs Linux et Windows utilisent leur interface de ligne de commande de choix, Bash ou PowerShell. Azure Cloud Shell est accessible via le portail, comme une interface web autonome à [shell.azure.com](https://shell.azure.com), ou à partir de plusieurs autres points d’accès. Pour plus d’informations, voir la [présentation d’Azure Cloud Shell](../../cloud-shell/overview.md).

## <a name="azure-subscriptions"></a>Abonnements Azure

Un abonnement est un regroupement logique de services Azure, associé à un compte Azure. Un seul compte Azure peut contenir plusieurs abonnements. La facturation des services Azure est effectuée par abonnement. Les abonnements Azure sont dotés d’un Administrateur de compte ayant un contrôle total sur l’abonnement et d’un Administrateur de service ayant le contrôle sur tous les services de l’abonnement. Pour plus d’informations sur les administrateurs d’abonnements classiques, consultez [Ajouter ou changer des administrateurs d’abonnements Azure](../../cost-management-billing/manage/add-change-subscription-administrator.md). En plus des administrateurs, les comptes individuels peuvent bénéficier d’un contrôle détaillé des ressources Azure en utilisant un [Contrôle d’accès en fonction du rôle (RBAC) Azure](../../role-based-access-control/overview.md).

### <a name="select-and-enable-an-azure-subscription"></a>Sélectionner et activer un abonnement Azure

Avant de pouvoir utiliser les services Azure, vous devez vous abonner. Plusieurs types d’abonnements sont disponibles.

**Comptes gratuits**  : Le lien pour s’abonner à un compte gratuit se trouve sur le [site web Azure](https://azure.microsoft.com/). Vous disposez d’un crédit valable sur une période de 30 jours, pour essayer la combinaison de ressources de votre choix dans Azure. Si vous dépassez le montant du crédit alloué, votre compte est suspendu. À la fin de la période d’essai, vos services sont désactivés et ne fonctionnent plus. Vous pouvez à tout moment passer à un abonnement avec paiement à l’utilisation.

**Abonnements MSDN**  : Si vous avez un abonnement MSDN, vous obtenez un crédit Azure d’un montant spécifique tous les mois. Par exemple, si vous utilisez une édition de Microsoft Visual Studio Enterprise avec un abonnement MSDN, vous bénéficiez d’un crédit Azure de \$150 USD par mois.

Si vous dépassez le montant du crédit, vos services sont désactivés jusqu’au début du mois suivant. Vous pouvez désactiver le plafond des dépenses et ajouter une carte de crédit à utiliser pour les dépassements. Certains de ces coûts bénéficient de remise pour les comptes MSDN. Par exemple, vous pouvez payer le prix Linux pour les machines virtuelles exécutant Windows Server, sans aucuns frais supplémentaires pour les serveurs Microsoft tels que Microsoft SQL Server. Les comptes MSDN s’avèrent ainsi parfaits pour les scénarios de développement et de test.

**Comptes BizSpark**  : Le programme Microsoft BizSpark présente de nombreux avantages pour les start-ups. Vous bénéficiez notamment de l’accès à tous les logiciels Microsoft dans les environnements de développement et de test pour cinq comptes MSDN maximum. Vous obtenez 150 USD de crédit Azure pour chacun de ces cinq comptes MSDN et profitez de tarifs réduits pour plusieurs services Azure tels que Machines virtuelles.

**Paiement à l’utilisation**  : Avec cet abonnement, vous payez uniquement ce que vous utilisez en associant une carte de crédit ou de débit au compte. Si vous êtes une organisation, vous pouvez également être approuvé pour la facturation.

**Contrats Entreprise**  : Avec un contrat Entreprise, vous vous engagez à utiliser un certain nombre de services dans Azure sur l’année à venir et vous payez ce montant à l’avance. L’engagement que vous contractez court tout au long de l’année. Si vous dépassez le montant de l’engagement, vous pouvez régler le surplus en retard. Selon le montant de l’engagement, vous bénéficiez d’une remise sur les services dans Azure.

### <a name="grant-administrative-access-to-an-azure-subscription"></a>Accorder un accès administratif à un abonnement Azure

Azure RBAC présente plusieurs rôles intégrés que vous pouvez utiliser pour attribuer des autorisations. Pour faire d’un utilisateur un administrateur d’un abonnement Azure, attribuez-lui le rôle [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) au niveau de l’abonnement. Le rôle Propriétaire donne à l’utilisateur un accès total à toutes les ressources de l’abonnement, ainsi que le droit de déléguer l’accès à d’autres personnes.

Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).

### <a name="view-billing-information-in-the-azure-portal"></a>Afficher les informations de facturation dans le portail Azure

Un aspect essentiel de l’utilisation d’Azure est la possibilité d’afficher les informations de facturation. Le portail Azure fournit un aperçu détaillé des informations de facturation Azure.

Pour plus d’informations, consultez [Comment télécharger votre facture Azure et vos données d’utilisation quotidienne](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md).

### <a name="get-billing-information-from-billing-apis"></a>Obtenir des informations de facturation à partir des API de facturation

En plus de l’affichage de la facturation dans le portail, vous pouvez accéder aux informations de facturation à l’aide d’un script ou d’un programme par le biais des API REST de facturation Azure :

- Vous pouvez utiliser l’API d’usage Azure pour récupérer vos données d’utilisation. Vous affinez les informations propres à la facturation par le marquage des ressources Azure associées. Par exemple, vous pouvez marquer chacune des ressources dans un groupe de ressources avec un nom de service ou celui d’un projet, puis suivre spécifiquement les coûts de ce marqueur.

- Vous pouvez utiliser l’API de carte de tarifs Azure pour lister toutes les ressources disponibles, ainsi que les métadonnées et les informations tarifaires de chacune de ces ressources.

Pour plus d’informations, consultez [Obtenir une vue d’ensemble de votre consommation des ressources Microsoft Azure](../../cost-management-billing/manage/usage-rate-card-overview.md).

### <a name="forecast-cost-with-the-pricing-calculator"></a>Prévoir le coût à l’aide de la calculatrice de prix

Le tarif de chaque service dans Azure est différent. De nombreux services Azure fournissent des niveaux De base, Standard et Premium. En règle générale, chaque niveau propose plusieurs tranches de prix et de performances. À l’aide de la [Calculatrice de prix en ligne](https://azure.microsoft.com/pricing/calculator), vous pouvez faire des estimations. La calculatrice offre une grande souplesse en permettant d’estimer le coût sur une seule ressource ou sur un groupe de ressources.

## <a name="azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager est un système de déploiement, de gestion et d’organisation des ressources Azure. En l’utilisant, vous pouvez rassembler de nombreuses ressources individuelles dans un groupe de ressources.

Il comporte également des fonctionnalités de déploiement permettant un déploiement personnalisable et la configuration des ressources associées. Ainsi, en utilisant Resource Manager, vous pouvez déployer une application composée de plusieurs machines virtuelles, d’un équilibreur de charge et d’une base de données dans Azure SQL Database sous la forme d’une unité unique. Vous développez ces déploiements à l’aide d’un modèle Azure Resource Manager.

Resource Manager offre plusieurs avantages :

- Vous pouvez déployer, gérer et surveiller toutes les ressources de votre solution comme un groupe, plutôt que de les gérer individuellement.

- Vous pouvez déployer votre solution plusieurs fois, tout au long du cycle de vie de développement, et avoir ainsi l’assurance que vos ressources déployées sont constantes.

- Vous pouvez gérer votre infrastructure à l’aide de modèles déclaratifs plutôt que de scripts.

- Vous pouvez définir les dépendances entre les ressources afin de les déployer dans le bon ordre.

- Vous pouvez appliquer un contrôle d’accès à tous les services de votre groupe de ressources, car Azure RBAC est intégré en mode natif à la plateforme de gestion.

- Vous pouvez appliquer des marqueurs aux ressources pour organiser logiquement toutes les ressources de votre abonnement.

- Vous pouvez clarifier la facturation de votre organisation en affichant les coûts d’un groupe de ressources partageant le même marqueur.

### <a name="tips-for-creating-resource-groups"></a>Conseils pour la création de groupes de ressources

Lorsque vous prenez des décisions concernant vos groupes de ressources, tenez compte des conseils suivants :

- Toutes les ressources d’un groupe de ressources doivent avoir le même cycle de vie.

- Vous pouvez attribuer une ressource à un seul groupe à la fois.

- Vous pouvez à tout moment ajouter ou supprimer une ressource dans un groupe de ressources. Toutes les ressources doivent appartenir à un groupe de ressources. Par conséquent, si vous supprimez une ressource d’un groupe, vous devez l’ajouter à un autre.

- Vous pouvez à tout moment déplacer la plupart des types de ressources vers un autre groupe de ressources.

- Les ressources d’un groupe de ressources peuvent se trouver dans différentes régions.

- Vous pouvez utiliser un groupe de ressources pour contrôler l’accès aux ressources qu’il contient.

### <a name="building-resource-manager-templates"></a>Création de modèles Resource Manager

Les modèles Resource Manager définissent de façon déclarative les ressources et les configurations de ressources à déployer dans un seul groupe de ressources. Vous pouvez utiliser les modèles Resource Manager pour orchestrer les déploiements complexes sans nécessiter trop de script ou de configuration manuelle. Après avoir développé un modèle, vous pouvez le déployer plusieurs fois, chaque fois avec des résultats identiques.

Un modèle Resource Manager se compose de quatre sections :

- **Paramètres**  : Il s’agit d’entrées pour le déploiement. Les valeurs des paramètres peuvent être fournies par un être humain ou un processus automatisé. Un exemple de paramètre peut être le nom d’utilisateur de l’administrateur et le mot de passe associé pour une machine virtuelle Windows. Les valeurs des paramètres sont utilisées dans le déploiement lorsqu’elles sont spécifiées.

- **Variables**  : Elles servent à contenir les valeurs qui sont utilisées dans le déploiement. Contrairement aux paramètres, une valeur de variable n’est pas fournie au moment du déploiement. Au lieu de cela, elle est codée en dur ou générée de façon dynamique.

- **Ressources**  : Cette section du modèle définit les ressources à déployer, comme les machines virtuelles, les comptes de stockage et les réseaux virtuels.

- **Sortie** : Après un déploiement, Resource Manager peut retourner des données, comme des chaînes de connexion générées dynamiquement.

Les mécanismes suivants sont disponibles pour l’automatisation du déploiement :

- **Fonctions**  : Vous pouvez utiliser plusieurs fonctions dans les modèles Resource Manager. Celles-ci incluent des opérations, telles que la conversion d’une chaîne en minuscules, le déploiement de plusieurs instances d’une ressource définie, et le retour de façon dynamique du groupe de ressources cible. Les fonctions Resource Manager permettent de créer des déploiements dynamiques.

- **Dépendances des ressources**  : Quand vous déployez plusieurs ressources, certaines ont une dépendance sur d’autres. Pour faciliter le déploiement, vous pouvez utiliser une déclaration de dépendance, afin que les ressources dépendantes soient déployées avant les autres.

- **Liaison de modèles**  : À partir d’un modèle Resource Manager, vous pouvez lier un autre modèle. Cela permet une décomposition du déploiement en un ensemble de modèles dont les usages sont spécifiques et ciblés.

Vous pouvez générer des modèles Resource Manager dans n’importe quel éditeur de texte. Cela dit, le kit SDK Azure pour Visual Studio comprend des outils pour vous aider. À l’aide de Visual Studio, vous pouvez ajouter des ressources au modèle par l’intermédiaire d’un Assistant, puis déployer et déboguer le modèle directement à partir de Visual Studio. Pour plus d’informations, consultez [Création de modèles Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).

Enfin, vous pouvez convertir des groupes de ressources existants en modèles réutilisables à partir du portail Azure. Cela peut être utile si vous souhaitez créer un modèle de déploiement pour un groupe de ressources existant, ou que vous voulez simplement examiner le JSON sous-jacent. Pour exporter un groupe de ressources, sélectionnez le bouton **Script d’automatisation** dans les paramètres du groupe de ressources.

## <a name="security-of-azure-resources-azure-rbac"></a>Sécurité des ressources Azure (Azure RBAC)

Vous pouvez accorder l’accès fonctionnel aux comptes d’utilisateur dans une étendue spécifiée : abonnement, groupe de ressources ou ressources individuelles. Cela signifie que vous pouvez déployer un ensemble de ressources dans un groupe de ressources, telles qu’une machine virtuelle et toutes les ressources associées, et accorder des autorisations à un utilisateur ou un groupe spécifique. Cette approche limite l’accès aux seules ressources qui appartiennent au groupe de ressources cible. Vous pouvez également accorder l’accès à une ressource unique, telle qu’une machine virtuelle ou un réseau virtuel.

Pour accorder l’accès, vous assignez un rôle à l’utilisateur ou au groupe d’utilisateurs. Il existe de nombreux rôles prédéfinis. Vous pouvez également définir vos propres rôles personnalisés.

Voici quelques exemples de [rôles intégrés dans Azure](../../role-based-access-control/built-in-roles.md) :

- **Propriétaire** : Un utilisateur avec ce rôle peut tout gérer, y compris l’accès.

- **Lecteur** : Un utilisateur avec ce rôle peut lire les ressources de tous les types (à l’exception des secrets), mais ne peut pas faire de changements.

- **Contributeur de machine virtuelle** : Un utilisateur avec ce rôle peut gérer des machines virtuelles, mais ne peut pas gérer le réseau virtuel auxquelles elle sont connectées, ni le compte de stockage où réside le fichier VHD.

- **Contributeur de base de données SQL**  : Un utilisateur avec ce rôle peut gérer des bases de données SQL, mais pas leurs stratégies de sécurité.

- **Gestionnaire de sécurité SQL**  : Un utilisateur avec ce rôle peut gérer les stratégies de sécurité des serveurs et bases de données SQL.

- **Contributeur de compte de stockage**  : Un utilisateur avec ce rôle peut gérer les comptes de stockage, mais ne peut pas en gérer l’accès.

Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).

## <a name="azure-virtual-machines"></a>Machines virtuelles Azure

Parmi les services IaaS d’Azure, Machines virtuelles Azure est l’un des plus importants. Ce service prend en charge le déploiement des machines virtuelles de Windows ou de Linux dans un centre de données Microsoft Azure. Avec le service Machines virtuelles Azure, vous avez le contrôle total sur la configuration des machines virtuelles et êtes responsable de l’installation, de la configuration et de la maintenance de tous les logiciels.

Lorsque vous déployez une machine virtuelle Azure, vous pouvez sélectionner une image à partir de la Place de marché Azure, ou fournir votre propre image généralisée. Cette image est utilisée pour appliquer le système d’exploitation et la configuration initiale. Lors du déploiement, Resource Manager gère certains paramètres de configuration, tels que l’attribution du nom de l’ordinateur, les informations d’identification d’administration et la configuration réseau. Vous pouvez utiliser des extensions de machine virtuelle Azure pour automatiser davantage les configurations, telles que l’installation de logiciels, la configuration d’antivirus et la surveillance des solutions.

Vous pouvez créer des machines virtuelles de tailles très différentes. La taille d’une machine virtuelle détermine l’allocation des ressources, telles que les capacités de traitement, de mémoire et de stockage. Dans certains cas, des composants spécifiques, tels que les cartes réseau compatibles RDMA et les disques SSD, sont uniquement disponibles dans certaines tailles de machine virtuelle. Pour obtenir une liste complète des possibilités et des tailles de machine virtuelle, consultez « Tailles des machines virtuelles dans Azure » pour [Windows](../../virtual-machines/sizes.md) et [Linux](../../virtual-machines/sizes.md).

### <a name="use-cases"></a>Cas d'utilisation

Du fait que les machines virtuelles Azure offrent un contrôle total sur la configuration, elles représentent une solution idéale pour un large éventail de charges de travail de serveur qui ne tiennent pas dans un modèle PaaS. Sur la plateforme Microsoft Azure, il est désormais possible d’exécuter les charges de travail des serveurs, comme les serveurs de base de données (SQL Server, Oracle ou MongoDB), Windows Server Active Directory, Microsoft SharePoint et bien d’autres. Si vous le souhaitez, vous pouvez déplacer ces charges de travail depuis un centre de données local vers une ou plusieurs régions Azure, sans qu’une reconfiguration importante soit nécessaire.

### <a name="deployment-of-virtual-machines"></a>Déploiement de machines virtuelles

Vous pouvez déployer Machines virtuelles Azure par l’intermédiaire du portail Azure, en utilisant l’automatisation avec le module Azure PowerShell ou l’interface CLI multiplateforme.

#### <a name="portal"></a>Portail

Pour déployer une machine virtuelle en utilisant le portail Azure, vous avez besoin uniquement d’un abonnement Azure actif et d’un accès à un navigateur web. Vous pouvez sélectionner de nombreuses images de systèmes d’exploitation avec des configurations différentes. Toutes les exigences au niveau du stockage et de la mise en réseau sont configurées au cours du déploiement. Pour plus d’informations, consultez « Créer une machine virtuelle dans le portail Azure » pour [Windows](../../virtual-machines/windows/quick-create-portal.md) et [Linux](../../virtual-machines/linux/quick-create-portal.md).

En plus de déployer une machine virtuelle à partir du portail Azure, vous pouvez déployer un modèle Azure Resource Manager à partir de ce même portail. Toutes les ressources, telles que définies dans le modèle, sont alors déployées et configurées. Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et du portail Azure](../../azure-resource-manager/templates/deploy-portal.md).

#### <a name="powershell"></a>PowerShell

Le déploiement d’une machine virtuelle Azure à l’aide de PowerShell permet l’automatisation du déploiement complet de toutes les ressources de machine virtuelle associées, notamment le stockage et la mise en réseau. Pour plus d’informations, consultez [Créer une machine virtuelle Windows à l’aide de Resource Manager et de PowerShell](../../virtual-machines/windows/quick-create-powershell.md).

En plus du déploiement individuel des ressources de calcul Azure, vous pouvez utiliser le module Azure PowerShell pour déployer un modèle Azure Resource Manager. Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md).

#### <a name="command-line-interface-cli"></a>Interface de ligne de commande (CLI)

À l’instar du module PowerShell, l’interface de ligne de commande Azure assure l’automatisation du déploiement et s’utilise sur les systèmes Windows, OS X ou Linux. Lorsque vous utilisez la commande **vm quick-create** de l’interface de ligne de commande Azure, toutes les ressources de machine virtuelle associées (notamment le stockage et la mise en réseau), et la machine virtuelle elle-même, sont déployées. Pour plus d’informations, consultez [Créer une machine virtuelle Linux dans Azure à l’aide de l’interface de ligne de commande](../../virtual-machines/linux/quick-create-cli.md).

De même, vous pouvez utiliser l’interface de ligne de commande Azure pour déployer un modèle Azure Resource Manager. Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure CLI](../../azure-resource-manager/templates/deploy-cli.md).

### <a name="access-and-security-for-virtual-machines"></a>Accès et sécurité des machines virtuelles

L’accès à une machine virtuelle depuis Internet nécessite que l’interface du réseau associé, ou le cas échéant de l’équilibreur de charge, soit configuré avec une adresse IP publique. L’adresse IP publique inclut un nom DNS qui se résout sur la machine virtuelle ou l’équilibreur de charge. Pour plus d’informations, consultez [Adresses IP dans Azure](../../virtual-network/public-ip-addresses.md).

Vous gérez l’accès à la machine virtuelle sur l’adresse IP publique à l’aide d’une ressource de groupe de sécurité réseau. Un groupe de sécurité réseau (NSG, Network Security Group) agit comme un pare-feu et autorise ou refuse le trafic via l’interface réseau ou le sous-réseau sur un ensemble de ports définis. Par exemple, pour créer une session Bureau à distance avec une machine virtuelle Azure, vous devez configurer le NSG de manière à autoriser le trafic entrant sur le port 3389. Pour plus d’informations, consultez [Ouverture de ports sur une machine virtuelle dans Azure à l’aide du portail Azure](../../virtual-machines/windows/nsg-quickstart-portal.md).

Enfin, comme c’est le cas pour la gestion de tout système informatique, vous devez assurer la sécurité d’une machine virtuelle Azure au niveau du système d’exploitation, à l’aide des informations d’identification de sécurité et des pare-feu logiciels.

## <a name="azure-storage"></a>Stockage Azure

Stockage Azure est un service géré par Microsoft qui fournit un stockage redondant, évolutif et durable. En tant que ressource, vous pouvez ajouter un compte de stockage Azure à n’importe quel groupe de ressources au moyen d’une méthode de déploiement de ressource. Azure comporte quatre types de stockage : Stockage Blob, stockage Fichier, stockage Table et stockage File d’attente. Lors du déploiement d’un compte de stockage, deux types de comptes sont disponibles, l’un à usage général, l’autre de stockage blob. Un compte de stockage à usage général vous donne accès aux quatre types de stockage. Les comptes de stockage blob sont similaires aux comptes à usage général, mais contiennent des objets blob spécialisés qui incluent les niveaux d’accès à chaud et à froid. Pour plus d’informations sur le stockage d’objets blob, consultez [Stockage Blob Azure](../../storage/blobs/storage-blob-storage-tiers.md).

Les comptes de stockage Azure peuvent être configurés selon différents niveaux de redondance :

- Le **stockage localement redondant** offre une haute disponibilité en vous assurant que trois copies de l’ensemble des données sont effectuées de façon synchrone avant qu’une opération d’écriture soit considérée comme réussie. Ces copies sont stockées dans un établissement unique situé dans une seule région. Les réplicas se trouvent dans des domaines d’erreur et de mise à niveau distincts. Cela signifie que les données sont disponibles même si un nœud de stockage contenant vos données est défaillant ou est mis hors connexion pour une mise à jour.

- Le **stockage géo-redondant** réalise trois copies synchrones des données dans la région principale pour la haute disponibilité, puis trois réplicas de façon asynchrone dans une région associée pour la récupération d’urgence.

- Le **stockage géo-redondant avec accès en lecture** correspond à un stockage géo-redondant, avec la possibilité de lire les données dans la région secondaire. Cette capacité le rend approprié pour une reprise d’activité partielle. S’il y a un problème avec la région principale, vous pouvez modifier votre application pour avoir un accès en lecture seule à la région associée.

### <a name="use-cases"></a>Cas d'utilisation

Chaque type de stockage convient à une utilisation différente.

#### <a name="blob-storage"></a>Stockage d'objets blob

Le mot *blob* est l’acronyme de « binary large object », un *objet binaire volumineux* . Les objets blob sont des fichiers non structurés, comme ceux que vous stockez sur votre ordinateur. Ce service peut stocker tout type de données texte ou binaires, par exemple, un document, un fichier multimédia ou un programme d’installation d’application. Le stockage d’objets blob est également appelé Stockage Blob. Le Stockage Blob Azure conserve également les disques de données des machines virtuelles Azure.

Ce type de stockage Azure prend en charge trois types d’objets blob :

- **Objets blob de blocs** Ces objets servent à stocker des fichiers ordinaires d’une taille maximale de 195 Go (4 Mo × avec 50 000 blocs). L’utilisation principale des objets blob de blocs est le stockage de fichiers lus du début jusqu’à la fin, tels que les fichiers multimédias ou les fichiers d’image de sites Web. Ils sont nommés objets blob de blocs, parce que les fichiers supérieurs à 64 Mo doivent être chargés sous forme de petits blocs. Ces blocs sont ensuite consolidés (ou validés) dans l’objet blob final.

- **Objets blob de pages** Ces objets servent à stocker les fichiers à accès aléatoire d’une taille maximale de 1 To. Les objets blob de pages sont utilisés principalement comme stockage de sauvegarde pour les disques durs virtuels qui fournissent des disques durables à Machines virtuelles Azure, le service de calcul IaaS dans Azure. Ils sont nommés objets blob de pages, car ils fournissent un accès en lecture/écriture aléatoire vers des pages de 512 octets.

- **Objets blob d’ajout** Ces objets se composent de blocs, comme les objets blob de blocs, mais sont optimisés pour les opérations d’ajout. Leur utilisation est fréquente pour l’enregistrement d’informations à partir d’une ou de plusieurs sources dans le même objet blob. Vous pouvez, par exemple, écrire toute la journalisation du suivi dans le même objet blob d’ajout pour une application qui s’exécute sur plusieurs machines virtuelles. La taille d’un seul objet blob d’ajout peut atteindre 195 Go.

Pour plus d’informations, consultez [Bien démarrer avec le Stockage Blob Azure à l’aide de .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md).

#### <a name="file-storage"></a>Stockage Fichier

Le Stockage Fichier Azure est un service qui propose des partages de fichiers dans le cloud en utilisant le protocole SMB (Server Message Block) standard. Le service prend en charge SMB 2.1 et SMB 3.0. Avec Stockage Fichier Azure, vous pouvez migrer des applications qui s’appuient sur des partages de fichiers sur Azure, rapidement et sans réécritures onéreuses. Les applications s’exécutant sur des machines virtuelles Azure, dans des services cloud ou à partir de clients locaux, peuvent monter un partage de fichiers dans le cloud. La façon de faire est semblable à celle d’une application de bureau montant un partage SMB classique. Un nombre illimité de composants d’application peuvent ensuite monter un partage de stockage de fichiers et y accéder simultanément.

Un partage Stockage Fichier étant un partage de fichiers SMB standard, les applications exécutées dans Azure peuvent accéder aux données du partage par le biais d’API d’E/S de système de fichiers. Les développeurs peuvent ainsi utiliser leur code et leurs compétences actuelles pour migrer les applications existantes. Les professionnels de l’informatique peuvent utiliser les applets de commande PowerShell pour créer, monter et gérer les partages Stockage Fichier dans le cadre de l’administration des applications Azure.

Pour plus d’informations, consultez [Bien démarrer avec Stockage Fichier Azure sur Windows](../../storage/files/storage-how-to-use-files-windows.md) ou [Utilisation de Stockage Fichier Azure avec Linux](../../storage/files/storage-how-to-use-files-linux.md).

#### <a name="table-storage"></a>Stockage de tables

Le stockage de tables Azure est un service qui stocke des données NoSQL structurées dans le cloud. Le Stockage Table est un magasin de clés/attributs doté d’une conception sans schéma. Du fait que le Stockage Table ne comporte pas de schéma, il est aisé d’adapter vos données en fonction de l’évolution des besoins de votre application. L'accès aux données est rapide et peu coûteux pour tous les types d'applications. Normalement, le stockage de tables est considérablement moins coûteux que le SQL traditionnel pour des volumes de données similaires.

Vous pouvez utiliser le stockage de tables pour stocker des jeux de données flexibles, par exemple, des données utilisateur pour des applications Web, des carnets d'adresses, des informations sur les périphériques et tout autre type de métadonnées requis par votre service. Vous pouvez stocker le nombre d’entités que vous voulez dans une table. Un compte de stockage peut contenir n’importe quel nombre de tables, jusqu’à la limite de capacité du compte de stockage.

Pour plus d’informations, consultez [Bien démarrer avec Stockage Table Azure](../../cosmos-db/tutorial-develop-table-dotnet.md).

#### <a name="queue-storage"></a>Stockage de files d'attente

Le Stockage File d’attente Azure fournit une messagerie cloud entre les composants d’application. Lors de la conception d’applications pour la mise à l’échelle, des composants d’application sont souvent découplés, de sorte qu’ils peuvent être mis à l’échelle indépendamment. Le Stockage File d’attente offre une messagerie asynchrone pour la communication entre les composants d’application, qu’ils soient exécutés dans le cloud, sur le bureau, sur un serveur local ou sur un appareil mobile. Le Stockage File d'attente prend également en charge la gestion des tâches asynchrones et la création des workflows de processus.

Pour plus d’informations, consultez [Bien démarrer avec Stockage File d’attente Azure](../../storage/queues/storage-dotnet-how-to-use-queues.md).

### <a name="deploying-a-storage-account"></a>Déploiement d’un compte de stockage

Il existe plusieurs options de déploiement pour un compte de stockage.

#### <a name="portal"></a>Portail

Le déploiement d’un compte de stockage par le portail Azure ne nécessite qu’un abonnement Azure actif et l’accès à un navigateur web. Vous pouvez déployer un nouveau compte de stockage dans un groupe de ressources existant ou nouveau. Dès que vous avez créé le compte de stockage, vous pouvez créer un conteneur d’objets blob ou un partage de fichiers à l’aide du portail. Vous pouvez créer des entités de Stockage Table et Stockage File d’attente par programmation. Pour plus d’informations, consultez la rubrique [Création d’un compte de stockage](../../storage/common/storage-account-create.md) .

En plus du déploiement d’un compte de stockage à partir du portail Azure, vous pouvez déployer un modèle Azure Resource Manager à partir de ce même portail. Toutes les ressources sont alors déployées et configurées, comme défini dans le modèle, notamment les comptes de stockage. Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et du portail Azure](../../azure-resource-manager/templates/deploy-portal.md).

#### <a name="powershell"></a>PowerShell

Le déploiement d’un compte de stockage Azure à l’aide de PowerShell permet l’automatisation du déploiement complet du compte de stockage. Pour plus d’informations, consultez [Utilisation d’Azure PowerShell avec Stockage Azure](/powershell/module/az.storage/).

En plus de déployer des ressources Azure individuellement, vous pouvez utiliser le module Azure PowerShell pour déployer un modèle Azure Resource Manager. Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md).

#### <a name="command-line-interface-cli"></a>Interface de ligne de commande (CLI)

À l’instar du module PowerShell, l’interface de ligne de commande Azure assure l’automatisation du déploiement et s’utilise sur les systèmes Windows, OS X ou Linux. Vous pouvez utiliser la commande d’Azure CLI **storage account create** pour créer un compte de stockage. Pour plus d’informations, consultez [Utilisation de l’interface de ligne de commande Azure avec Stockage Azure](../../storage/blobs/storage-quickstart-blobs-cli.md).

De même, vous pouvez utiliser l’interface de ligne de commande Azure pour déployer un modèle Azure Resource Manager. Pour plus d’informations, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure CLI](../../azure-resource-manager/templates/deploy-cli.md).

### <a name="access-and-security-for-azure-storage"></a>Accès et sécurité pour Stockage Azure

Il est possible d’accéder à Stockage Azure de différentes manières, y compris par le portail Azure, au cours de la création et du fonctionnement de machines virtuelles, et à partir de bibliothèques clientes de stockage.

#### <a name="virtual-machine-disks"></a>Disques de machines virtuelles

Lorsque vous déployez une machine virtuelle, vous devez également créer un compte de stockage pour stocker le disque du système d’exploitation de la machine virtuelle et les disques de données supplémentaires. Vous pouvez sélectionner un compte de stockage existant ou en créer un. La taille maximale d’un objet blob étant de 1 024 Go, un seul disque de machine virtuelle peut afficher une taille de 1 023 Go. Pour configurer un disque de données plus volumineux, vous pouvez présenter plusieurs disques de données à la machine virtuelle et les rassembler en un disque logique unique. Pour plus d’informations, consultez les pages « Gestion des disques Azure » pour [Windows](../../virtual-machines/windows/tutorial-manage-data-disk.md) et [Linux](../../virtual-machines/linux/tutorial-manage-disks.md).

#### <a name="storage-tools"></a>Outils de stockage

Les comptes de stockage Azure sont accessibles par le biais de nombreux explorateurs de stockage différents, tels que Visual Studio Cloud Explorer. Ces outils vous permettent de parcourir les données et les comptes de stockage. Pour obtenir plus d’informations et la liste des explorateurs de stockage disponibles, consultez [Outils clients Stockage Azure](../../storage/common/storage-explorers.md).

#### <a name="storage-api"></a>API de stockage

Les ressources de stockage sont accessibles par le biais de n’importe quel langage capable de créer des requêtes HTTP/HTTPS. Par ailleurs, Stockage Azure offre des bibliothèques de programmation pour plusieurs langages répandus. Ces bibliothèques simplifient l’utilisation de Stockage Azure en gérant des détails, tels que l’invocation synchrone et asynchrone, le traitement par lots des opérations, la gestion des exceptions et les nouvelles tentatives automatiques. Pour plus d’informations, consultez [Référence sur l’API REST du service Stockage Azure](/rest/api/storageservices/Azure-Storage-Services-REST-API-Reference).

#### <a name="storage-access-keys"></a>Clés d’accès de stockage

Chaque compte de stockage a deux clés d’authentification, une clé principale et une clé secondaire. Chacune d’elles peut servir dans des opérations d’accès de stockage. Ces clés de stockage sont utilisées pour aider à sécuriser un compte de stockage, elles sont également nécessaires pour accéder aux données par programmation. Il existe deux clés pour autoriser la substitution occasionnelle des clés afin d’améliorer la sécurité. Il est essentiel de sécuriser ces clés, car leur possession, avec le nom du compte, offre un accès illimité à toutes les données du compte de stockage.

#### <a name="shared-access-signatures"></a>Signatures d’accès partagé

Si vous devez autoriser un accès contrôlé à vos ressources de stockage pour des utilisateurs, vous pouvez créer une signature d’accès partagé. Une signature d’accès partagé est un jeton qu’il est possible d’ajouter à une URL qui permet alors un accès délégué à une ressource de stockage. Quiconque possède le jeton peut accéder à la ressource qu’il désigne avec les autorisations qu’il spécifie, pendant son délai de validité. Pour plus d’informations, consultez [Utilisation des signatures d’accès partagé](../../storage/common/storage-sas-overview.md).

## <a name="azure-virtual-network"></a>Réseau virtuel Azure

Des réseaux virtuels sont nécessaires pour prendre en charge les communications entre les machines virtuelles. Vous pouvez définir des sous-réseaux, des adresses IP personnalisées, des paramètres DNS, des filtrages de sécurité et un équilibrage de charge. Azure prend en charge différents cas d’usage : réseaux cloud uniquement ou réseaux virtuels hybrides.

### <a name="cloud-only-virtual-networks"></a>Réseaux virtuels cloud uniquement

Par défaut, un réseau virtuel Azure est accessible uniquement aux ressources stockées dans Azure. Les ressources connectées au même réseau virtuel peuvent communiquer entre elles. Vous pouvez associer des interfaces réseau de machine virtuelle à des équilibreurs de charge au moyen d’une adresse IP publique et rendre ainsi la machine virtuelle accessible sur Internet. Vous pouvez aider à sécuriser l’accès aux ressources exposées publiquement à l’aide d’un groupe de sécurité réseau.

![Réseau virtuel Azure pour une application web à 2 niveaux](/azure/load-balancer/media/load-balancer-internal-overview/ic744147.png)

### <a name="hybrid-virtual-networks"></a>Réseaux virtuels hybrides

Vous pouvez connecter le réseau local à un réseau virtuel Azure par le biais d’une connexion ExpressRoute ou VPN de site à site. Dans cette configuration, le réseau virtuel Azure est essentiellement une extension cloud de votre réseau local.

Le réseau virtuel Azure étant connecté à votre réseau local, les réseaux virtuels entre locaux ne doivent utiliser qu’une partie de l’espace d’adressage utilisé par votre organisation. De la même façon que les différents locaux de l’entreprise se voient affecter un sous-réseau IP spécifique, Azure devient un emplacement supplémentaire lorsque vous étendez votre réseau.
Il existe plusieurs options pour déployer un réseau virtuel.

- [Portail](../..//virtual-network/quick-create-portal.md)

- [PowerShell](../../virtual-network/quick-create-powershell.md)

- [Interface de ligne de commande (CLI)](../../virtual-network/quick-create-cli.md)

- Modèles Azure Resource Manager

> **Quand les utiliser**  : Chaque fois que vous utilisez des machines virtuelles dans Azure, vous utilisez des réseaux virtuels. Ainsi, vous pouvez segmenter vos machines virtuelles en sous-réseaux publics et privés, à l’image des centres de données locaux.
>
> **Prise en main**  : Le déploiement d’un réseau virtuel Azure à l’aide du portail Azure ne demande qu’un abonnement Azure actif et l’accès à un navigateur web. Vous pouvez déployer un nouveau réseau virtuel dans un groupe de ressources existant ou nouveau. Lorsque vous créez une machine virtuelle à partir du portail, vous pouvez sélectionner un réseau virtuel existant ou en créer un. Pour commencer, [créez un réseau virtuel à l’aide du portail Azure](../../virtual-network/quick-create-portal.md).

### <a name="access-and-security-for-virtual-networks"></a>Accès et sécurité des réseaux virtuels

Vous pouvez aider à sécuriser les réseaux virtuels Azure à l’aide d’un groupe de sécurité réseau (NSG). Les groupes NSG contiennent des règles de liste de contrôle d’accès (ACL) qui autorisent ou rejettent le trafic réseau vers vos instances de machine virtuelle dans un réseau virtuel. Vous pouvez associer des NSG à des sous-réseaux ou à des instances de machine virtuelle individuelle au sein de ce sous-réseau. Lorsque vous associez un NSG à un sous-réseau, les règles ACL s’appliquent à toutes les instances de machine virtuelle présentes dans ce sous-réseau. Qui plus est, vous pouvez limiter davantage le trafic vers une machine virtuelle individuelle par l’association directe d’un NSG avec la machine virtuelle. Pour plus d’informations, consultez [Filtrer le trafic réseau avec les groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- [Créer une machine virtuelle Windows](../../virtual-machines/windows/quick-create-portal.md)
- [Créer une machine virtuelle Linux](../../virtual-machines/linux/quick-create-portal.md)