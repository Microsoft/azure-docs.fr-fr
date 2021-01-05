---
title: Azure Lab Services – Guide de l’administrateur | Microsoft Docs
description: Ce guide aide les administrateurs qui créent et gèrent des comptes lab à l’aide d’Azure Lab Services.
ms.topic: article
ms.date: 10/20/2020
ms.openlocfilehash: 3ad3ee38a6c08a6af85822d76012cc6dfc34ff4e
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462475"
---
# <a name="azure-lab-services---administrator-guide"></a>Azure Lab Services – Guide de l’administrateur
Les administrateurs informatiques qui gèrent les ressources cloud d’une université sont généralement responsables de la configuration du compte lab de leur établissement. Après avoir configuré un compte lab, les administrateurs ou enseignants créent des labos dans ce compte. Cet article offre une vue d’ensemble globale des ressources Azure en question ainsi que des conseils pour les créer.

![Diagramme de vue globale des ressources Azure d’un compte lab.](./media/administrator-guide/high-level-view.png)

- Les labos sont hébergés dans un abonnement Azure dont le service Azure Lab Services est propriétaire.
- Les comptes lab, une galerie d’images partagées, et les versions d’images sont hébergés dans votre abonnement.
- Le compte lab et la galerie d’images partagées peuvent se trouver dans le même groupe de ressources. Dans ce schéma, ils se trouvent dans des groupes de ressources différents.

Pour plus d’informations relatives à l’architecture, consultez [Bases de l’architecture des labos](./classroom-labs-fundamentals.md).

## <a name="subscription"></a>Abonnement
Votre université peut disposer d’un ou plusieurs abonnements Azure. Un abonnement vous permet de gérer la facturation et la sécurité de l’ensemble des ressources/services Azure qui y sont utilisés, notamment les comptes lab.

La relation entre un compte Lab et son abonnement est importante pour les raisons suivantes :

- La facturation est établie via l’abonnement qui contient le compte Lab.
- Vous pouvez accorder aux utilisateurs du locataire Azure Active Directory (Azure AD) de l’abonnement un accès au service Azure Lab Services. Vous pouvez ajouter un utilisateur en tant que Propriétaire/Contributeur de compte lab, Créateur de labo ou Propriétaire de labo.

Les labos et leurs machines virtuelles sont gérés et hébergés pour vous dans un abonnement dont le service Azure Lab Services est propriétaire.

## <a name="resource-group"></a>Resource group
Un abonnement contient un ou plusieurs groupes de ressources. Les groupes de ressources servent à créer des regroupements logiques des ressources Azure utilisées ensemble dans une même solution.  

Quand vous créez un compte Lab, vous devez configurer le groupe de ressources qui le contient. 

Un groupe de ressources est également nécessaire lorsque vous créez une [galerie d’images partagées](#shared-image-gallery). Vous pouvez placer votre compte lab et la galerie d’images partagées dans le même groupe de ressources ou dans deux groupes de ressources distincts. Vous souhaiterez peut-être adapter cette seconde approche si vous envisagez de partager la galerie d’images entre plusieurs solutions.

Lorsque vous créez un compte lab, vous pouvez créer et attacher automatiquement une galerie d’images partagées au même moment.  Cette option permet de créer le compte lab et la galerie d’images partagées dans des groupes de ressources distincts. Ce comportement se manifeste lorsque vous suivez la procédure décrite dans le tutoriel [Configurer la galerie d’images partagées lors de la création du compte lab](how-to-attach-detach-shared-image-gallery.md#configure-at-the-time-of-lab-account-creation). L’image située au début cet article utilise cette configuration. 

Nous vous recommandons de prendre le temps de réfléchir à la manière de structurer vos groupes de ressources car, une fois le groupe de ressources d’un compte lab ou d’une galerie d’images partagées créé, il n’est *plus* possible de le modifier. S’il vous faut modifier le groupe de ressources pour ces ressources, vous devrez supprimer et recréer votre compte lab ou la galerie d’images partagées.

## <a name="lab-account"></a>Compte Lab

Un compte lab sert de conteneur pour un ou plusieurs laboratoires. Lors des premiers pas avec Azure Lab Services, il est courant de n’avoir qu’un compte lab. À mesure que l’utilisation du lab évolue, vous pouvez décider de créer des comptes lab supplémentaires.

La liste suivante met en avant les scénarios dans lesquels la présence de plusieurs comptes lab peut s’avérer intéressante :

- **Gérer des besoins de stratégies différents entre les labos**

    Quand vous configurez un compte lab, vous définissez des stratégies qui s’appliquent à *tous* les laboratoires relevant du compte lab, par exemple :
    - Le réseau virtuel Azure avec des ressources partagées auxquelles le labo peut accéder. Par exemple, vous pouvez disposer d’un ensemble de labos qui doivent pouvoir accéder à un jeu de données partagées au sein d’un réseau virtuel.
    - Les images de machine virtuelle qui permettent aux labos de créer des machines virtuelles. Par exemple, vous pouvez disposer d’un ensemble de labos qui doivent pouvoir accéder à l’image de la Place de marché Azure [Data Science VM for Linux](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804).

    Si chacun de vos labos présente des exigences de stratégie uniques, il peut être intéressant de créer des comptes lab distincts afin de gérer chaque lab séparément.

- **Attribuer un budget distinct à chaque compte lab**
  
    Plutôt que de signaler les coûts de tous les labos via un seul compte lab, vous serez peut-être amené à ventiler le budget plus clairement. Par exemple, vous pouvez créer des comptes lab distincts pour le département mathématiques, le département informatique, ainsi que d’autres départements de votre université, afin de répartir le budget entre les différents départements.  Vous pouvez alors voir le coût de chaque compte lab en utilisant [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md).

- **Isoler les labos pilotes des labos actifs ou de production**
  
    Dans certains cas, vous pouvez piloter des changements de stratégies pour un compte lab sans risquer d’affecter vos labos actifs ou de production. Dans ce type de scénario, la création d’un compte Lab distinct à des fins de pilotage permet d’isoler les modifications. 

## <a name="lab"></a>Laboratoire

Un labo contient des machines virtuelles qui sont chacune attribuée à un étudiant déterminé.  En règle générale, vous pouvez vous attendre à ce que :

- Un labo existe pour chaque classe.
- Créez un nouvel ensemble de labos pour chaque semestre, trimestre ou autre système universitaire utilisé. Pour les classes qui doivent utiliser la même image, vous devez utiliser une [galerie d’images partagées](#shared-image-gallery). Ainsi, vous pouvez réutiliser des images entre les labos et les périodes universitaires.

Lorsque vous réfléchissez à la manière de structurer vos labos, prenez en compte les points suivants :

- **Toutes les machines virtuelles d’un labo sont déployées avec la même image publiée**

    De ce fait, si vous devez publier simultanément plusieurs images de labo pour une classe, un labo distinct devra être créé pour chaque image.
  
- **Le quota d’utilisation est défini au niveau du labo et s’applique à tous ses utilisateurs**

    Pour définir des quotas différents pour les utilisateurs, vous devez créer des labos distincts. Cependant, avant de pouvoir ajouter des heures supplémentaires à des utilisateurs spécifiques, vous devez avoir défini le quota.
  
- **La planification de démarrage ou d’arrêt est définie au niveau du labo et s’applique à toutes ses machines virtuelles**

    Comme la définition du quota, s’il vous faut définir des planifications différentes pour les utilisateurs, vous devez créer un labo distinct pour chaque planification.

Par défaut, chaque labo dispose de son propre réseau virtuel.  Si l’appairage de réseaux virtuels est activé, chaque labo dispose de son propre sous-réseau, associé au réseau virtuel spécifié.

## <a name="shared-image-gallery"></a>Galerie d’images partagées

Une galerie d’images partagées est attachée à un compte Lab et sert de référentiel central pour le stockage d’images. Une image est enregistrée dans la galerie dès qu’un enseignant choisit de l’exporter à partir d’un modèle de machine virtuelle de labo. Chaque fois qu’un enseignant apporte des modifications au modèle de machine virtuelle et l’exporte, de nouvelles versions de l’image sont enregistrées et les versions précédentes sont conservées.

Les formateurs peuvent publier une version d’image à partir de la galerie d’images partagées au moment de créer un labo. Même si la galerie stocke plusieurs versions d’une image, les enseignants ne peuvent sélectionner que la dernière version lors de la création du labo.

Le service Shared Image Gallery est une ressource facultative dont vous n’aurez peut-être pas besoin de suite compte tenu du nombre limité de labos dont vous disposerez dans un premier temps. Cela étant, le service Shared Image Gallery présente de nombreux avantages à mesure que vos besoins évoluent vers davantage de labos :

- **Vous pouvez enregistrer et gérer différentes versions d’une image de modèle de machine virtuelle**

    Cela s’avère utile pour créer une image personnalisée ou apporter des modifications (de logiciels, de configuration, etc.) à une image de la galerie de la Place de marché publique Azure.  Par exemple, il est fréquent que les enseignants réclament l’installation de différents logiciels ou outils. Plutôt que de demander aux étudiants d’installer manuellement les composants requis, il est possible d’exporter différentes versions de l’image du modèle de machine virtuelle dans une galerie d’images partagées. Vous pouvez ensuite utiliser ces versions d’images lorsque vous créez labos.

- **Vous pouvez partager et réutiliser des images de machines virtuelles de modèles entre les labos**

    Vous pouvez enregistrer et réutiliser une image pour ne pas devoir la configurer à partir de zéro chaque fois que vous créez un labo. Par exemple, si plusieurs classes doivent utiliser la même image, vous pouvez la créer, puis l’exporter dans la galerie d’images partagées pour permettre aux labos de la partager.

- **La disponibilité des images est garantie via la réplication automatique**

    Lorsque vous enregistrez une image de labo dans la galerie d’images partagées, elle est automatiquement répliquée dans les autres [régions au sein de la même zone géographique](https://azure.microsoft.com/global-infrastructure/regions/). En cas de panne dans une région, la publication de l’image dans votre labo n’est pas affectée, car un réplica d’image d’une autre région peut être utilisé.  La publication de machines virtuelles à partir de plusieurs réplicas peut également contribuer à améliorer les performances.

Pour regrouper logiquement des images partagées, vous pouvez effectuer l’une des opérations suivantes :

- Créez plusieurs galeries d’images partagées. Sachant que chaque compte lab peut se connecter à une seule galerie d’images partagées, cette option vous engage aussi à créer plusieurs comptes lab.
- Utilisez une seule galerie d’images partagées commune à plusieurs comptes lab. Dans ce cas, chaque compte lab peut activer uniquement les images applicables aux labos qu’il contient.

## <a name="naming"></a>Dénomination

Durant vos premiers pas avec Azure Lab Services, nous vous recommandons d’établir des conventions d’affectation de noms pour les groupes de ressources, les comptes Lab, les labos et la galerie d’images partagées. Bien que les conventions d’affectation de noms que vous établissez soient destinées à répondre aux besoins uniques de votre organisation, le tableau suivant en décrit les principes généraux :

| Type de ressource | Role | Modèle suggéré | Exemples |
| ------------- | ---- | ----------------- | -------- | 
| Resource group | Contient un ou plusieurs comptes Lab et une ou plusieurs galeries d’images partagées | \<organization short name\>-\<environment\>-rg<ul><li>**Nom abrégé de l’organisation** identifie l’organisation dans laquelle le groupe de ressources est utilisé.</li><li>**Environnement** identifie l’environnement de la ressource, par exemple, *pilote* ou *production*.</li><li>**Rg** est l’abréviation du type de ressource : *resource group* (groupe de ressources).</li></ul> | labosuniversitecontoso-rg<br/>contosouniversitylabs-pilot-rg<br/>labosuniversitecontoso-prod-rg |
| Compte Lab | Contient un ou plusieurs labos | \<organization short name\>-\<environment\>-la<ul><li>**Nom abrégé de l’organisation** identifie l’organisation dans laquelle le groupe de ressources est utilisé.</li><li>**Environnement** identifie l’environnement de la ressource, par exemple, *pilote* ou *production*.</li><li>**La** est l’abréviation du type de ressource : *lab account* (compte lab).</li></ul> | labosuniversitecontoso-la<br/>labosdeptmaths-la<br/>sciencedeptlabs-pilot-la<br/>labosdeptsciences-prod-la |
| Laboratoire | Contient une ou plusieurs machines virtuelles |\<class name\>-\<timeframe\>-\<educator identifier\><ul><li>**Nom de la classe** identifie la classe rattachée au labo.</li><li>**Période** identifie la période au cours de laquelle la classe est assurée.</li>**Identificateur du formateur** identifie le formateur propriétaire du lab.</li></ul> | CS1234-automne2019-jeandupont<br/>CS1234-printemps2019-jeandupont |
| Galerie d’images partagées | Contient une ou plusieurs versions d’image de machine virtuelle | galerie\<organization short name\> | galerielabosuniversitecontoso |

Pour plus d’informations sur le nommage des autres ressources Azure, consultez [Conventions d’affectation de noms pour les ressources Azure](/azure/architecture/best-practices/naming-conventions).

## <a name="regionslocations"></a>Régions ou localisations

Lors de la configuration de vos ressources du service Azure Lab Services, vous devez indiquer la région ou l’emplacement du centre de données appelé à héberger les ressources. Les sections suivantes décrivent comment une région ou un emplacement peut affecter chaque ressource impliquée dans la configuration d’un labo.

### <a name="resource-group"></a>Resource group

La région désigne le centre de données où sont stockées les informations sur un groupe de ressources. Les ressources Azure contenues dans le groupe de ressources peuvent se trouver dans une région différente de celle de leur parent.

### <a name="lab-account"></a>Compte Lab

L’emplacement d’un compte lab désigne la région où se trouve une ressource.  

### <a name="lab"></a>Laboratoire

L’emplacement d’un labo varie en fonction des facteurs suivants :

  - **Le compte lab est appairé à un réseau virtuel**
  
    Vous pouvez [appairer un compte lab à un réseau virtuel](./how-to-connect-peer-virtual-network.md) lorsqu’ils se trouvent dans la même région.  Lorsqu’un compte lab est appairé à un réseau virtuel, les labos sont automatiquement créés dans la même région que le compte lab et le réseau virtuel.

    > [!NOTE]
    > Lorsqu’un compte lab est appairé à un réseau virtuel, le paramètre **Autoriser le créateur du labo à choisir l’emplacement du labo** est désactivé. Pour plus d’informations, consultez [Autoriser un créateur de labo à choisir l’emplacement du labo](./allow-lab-creator-pick-lab-location.md).
    
  - **Aucun réseau virtuel n’est appairé *et* les créateurs de labo ne sont pas autorisés à choisir l’emplacement du labo**
  
    Quand *aucun* réseau virtuel n’est appairé au compte lab et que les [créateurs de labo ne sont *pas autorisés* à choisir l’emplacement du labo](./allow-lab-creator-pick-lab-location.md), les labos sont automatiquement créés dans une région disposant d’une capacité de machines virtuelles.  Plus précisément, le service Azure Lab Services recherche la disponibilité dans des [régions qui se trouvent dans la même zone géographique que le compte lab](https://azure.microsoft.com/global-infrastructure/regions).

  - **Aucun réseau virtuel n’est appairé *et* les créateurs de labo sont autorisés à choisir l’emplacement du labo**
       
    Quand *aucun* réseau virtuel n’est appairé et que [les créateurs de labo sont *autorisés* à choisir l’emplacement du labo](./allow-lab-creator-pick-lab-location.md), les emplacements pouvant être sélectionnés par le créateur de labo dépendent de la capacité disponible.

> [!NOTE]
> Pour s’assurer qu’une région dispose de suffisamment de capacité de machines virtuelles, il est important de commencer par demander de la capacité par le biais du compte lab ou lors de la création du labo.

Une règle générale consiste à attribuer à une ressource la région la plus proche de ses utilisateurs. Pour les labos, cela signifie que le labo doit être créé au plus près de vos étudiants. Pour les cours en ligne dont les étudiants sont situés dans le monde entier, faites appel à votre meilleur jugement pour créer un labo central. Vous pouvez également scinder une classe en plusieurs labos selon la région de vos étudiants.

## <a name="vm-sizing"></a>Dimensionnement des machines virtuelles

Au moment de créer un labo, les administrateurs ou les créateurs de labo ont le choix entre diverses tailles de machine virtuelle selon les besoins de leur classe. N’oubliez pas que les tailles de calcul disponibles dépendent de la région dans laquelle se trouve votre compte lab.

| Taille | Spécifications | Série | Utilisation suggérée |
| ---- | ----- | ------ | ------------- |
| Petite| <ul><li>2&nbsp;cœurs</li><li>3,5 gigaoctets (Go) de RAM</li> | [Standard_A2_v2](../virtual-machines/av2-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Taille idéale pour la ligne de commande, l’ouverture d’un navigateur web, les serveurs web à faible trafic et les bases de données de petite et moyenne taille. |
| Moyenne | <ul><li>4&nbsp;cœurs</li><li>7&nbsp;Go&nbsp;de RAM</li> | [Standard_A4_v2](../virtual-machines/av2-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Taille idéale pour les bases de données relationnelles, la mise en cache en mémoire et l’analyse. |
| Moyenne (virtualisation imbriquée) | <ul><li>4&nbsp;cœurs</li><li>16&nbsp;Go&nbsp;de RAM</li></ul> | [Standard_D4s_v3](../virtual-machines/dv3-dsv3-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#dsv3-series) | Taille idéale pour les bases de données relationnelles, la mise en cache en mémoire et l’analyse.
| grand | <ul><li>8&nbsp;cœurs</li><li>16&nbsp;Go&nbsp;de RAM</li></ul>  | [Standard_A8_v2](../virtual-machines/av2-series.md) | Taille idéale pour les applications exigeant des processeurs plus rapides, un niveau de performance du disque local plus élevé, des bases de données volumineuses et des caches mémoire volumineux.  Cette taille prend également en charge la virtualisation imbriquée. |
| Grande (virtualisation imbriquée) | <ul><li>8&nbsp;cœurs</li><li>32&nbsp;Go&nbsp;de RAM</li></ul>  | [Standard_D8s_v3](../virtual-machines/dv3-dsv3-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#dsv3-series) | Taille idéale pour les applications exigeant des processeurs plus rapides, un niveau de performance du disque local plus élevé, des bases de données volumineuses et des caches mémoire volumineux. |
| GPU de petite taille (visualisation) | <ul><li>6&nbsp;cœurs</li><li>56&nbsp;Go de&nbsp;RAM</li>  | [Standard_NV6](../virtual-machines/nv-series.md) | Taille idéale pour la visualisation à distance, le streaming, les jeux et l'encodage à l’aide d’infrastructures de type OpenGL et DirectX. |
| GPU de petite taille (calcul) | <ul><li>6&nbsp;cœurs</li><li>56&nbsp;Go de&nbsp;RAM</li></ul>  | [Standard_NC6](../virtual-machines/nc-series.md) |Taille idéale pour les applications nécessitant beaucoup de ressources informatiques comme l’IA et le Deep Learning. |
| GPU de taille moyenne (visualisation) | <ul><li>12&nbsp;cœurs</li><li>112&nbsp;Go de&nbsp;RAM</li></ul>  | [Standard_NV12](../virtual-machines/nv-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Taille idéale pour la visualisation à distance, le streaming, les jeux et l'encodage à l’aide d’infrastructures de type OpenGL et DirectX. |

## <a name="manage-identity"></a>Gérer l’identité

Le [contrôle d’accès en fonction du rôle Azure (RBAC)](../role-based-access-control/overview.md) permet d’attribuer les rôles suivants pour accorder l’accès à des comptes lab et à des labos :

- **Propriétaire** de compte lab

    L’administrateur qui crée un compte lab est automatiquement ajouté au rôle Propriétaire de ce compte. Le rôle Propriétaire peut :
     - Modifier les paramètres du compte lab.
     - Donner à d’autres administrateurs l’accès au compte lab en tant que Propriétaires ou Contributeurs.
     - Donner à des enseignants l’accès à des labos en tant que Créateurs, Propriétaires ou Contributeurs.
     - Créer et gérer tous les labos au sein du compte lab.

- **Contributeur** de compte lab

    Un administrateur auquel le rôle Contributeur est attribué peut :
    - Modifier les paramètres du compte lab.
    - Créer et gérer tous les labos au sein du compte lab.

    Toutefois, le Contributeur *ne peut pas* accorder à d’autres utilisateurs l’accès à des comptes lab ou à des labos.

- **Créateur Lab**

    Pour créer des laboratoires au sein d’un compte lab, un enseignant doit être membre du rôle Créateur de laboratoire.  Un enseignant qui crée un labo est automatiquement ajouté en tant que Propriétaire de labo. Pour plus d’informations, consultez [Ajouter un utilisateur au rôle Créateur de labo](./tutorial-setup-lab-account.md#add-a-user-to-the-lab-creator-role). 

- **Propriétaire** ou **Contributeur** de labo
  
    Un enseignant doté d’un rôle de Propriétaire ou de Contributeur de labo peut afficher et modifier les paramètres d’un labo. Il doit également disposer du rôle de Lecteur de compte lab.

    Différence d’importance entre les rôles de Propriétaire et de Contributeur de labo, seul le Propriétaire peut autoriser d’autres utilisateurs à gérer un labo. Un Contributeur *ne peut pas* autoriser d’autres utilisateurs à gérer un labo.

- **Galerie d’images partagées**

    Lorsque vous attachez une galerie d’images partagées à un compte lab, les Propriétaires de compte lab et les Contributeurs et Créateurs de labo, ainsi que les Propriétaires de labo se voient automatiquement autorisés à afficher et enregistrer des images dans la galerie.

Lorsque vous attribuez des rôles, vous pouvez suivre ces conseils :

   - En règle générale, seuls les administrateurs doivent être Propriétaires ou Contributeurs d’un compte lab. Le compte lab peut présenter plusieurs Propriétaires ou Contributeurs.
   - Pour permettre aux enseignants de créer et de gérer des labos, il suffit de leur accorder le rôle Créateur de labo.
   - Pour permettre aux enseignants de gérer des labos spécifiques, mais *pas* de créer des labos, attribuez-leur le rôle de Propriétaire ou de Contributeur pour chacun des labos qu’ils seront amenés à gérer. Par exemple, vous pouvez permettre qu’un professeur et un assistant soient copropriétaires d’un labo. Pour plus d’informations, consultez [Ajouter des Propriétaires à un labo](./how-to-add-user-lab-owner.md).

## <a name="pricing"></a>Tarifs

### <a name="azure-lab-services"></a>Azure Lab Services

Pour en savoir plus sur la tarification, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).


### <a name="shared-image-gallery"></a>Galerie d’images partagées

Vous devez aussi prendre en compte la tarification du service Shared Image Gallery si vous prévoyez d’utiliser des galeries d’images partagées pour stocker et gérer des versions d’images. 

Créer une galerie d’images partagées et l’attacher à votre compte Lab sont des opérations gratuites. Aucun coût n’est induit tant que vous n’enregistrez pas de version d’image dans la galerie. Le coût d’utilisation d’une galerie d’images partagées s’avère généralement négligeable, mais il est important de comprendre la façon dont il est calculé, car il n’est pas inclus dans le tarif d’Azure Lab Services.  

#### <a name="storage-charges"></a>Frais de stockage

Pour stocker des versions d’images, une galerie d’images partagées utilise des disques managés HDD standard. La taille du disque managé HDD utilisé dépend de la taille de la version d’image stockée. Pour plus d’informations sur la tarification, consultez [Tarification de la fonctionnalité Disques managés](https://azure.microsoft.com/pricing/details/managed-disks/).

#### <a name="replication-and-network-egress-charges"></a>Frais de réplication et de sortie réseau

Lorsque vous enregistrez une version d’image en utilisant le modèle de machine virtuelle d’un labo, le service Azure Lab Services la stocke d’abord dans une région source avant de la répliquer automatiquement dans une ou plusieurs régions cibles. 

Il est important de noter qu’Azure Lab Services réplique automatiquement la version d’image source dans toutes les [régions cibles au sein de la zone géographique](https://azure.microsoft.com/global-infrastructure/regions/) dans laquelle se trouve le labo. Par exemple, si votre labo se trouve dans la zone géographique États-Unis, une version d’image est répliquée dans chacune des huit régions des États-Unis.

La sortie réseau est facturée du moment qu’une version d’image est répliquée de la région source vers d’autres régions cibles. Le montant facturé dépend de la taille de la version d’image au moment où les données de l’image sont initialement transférées en sortie depuis la région source.  Pour plus d’informations sur la tarification, consultez [Détails sur les tarifs de bande passante](https://azure.microsoft.com/pricing/details/bandwidth/).

Les clients de [solutions Éducation](https://www.microsoft.com/licensing/licensing-programs/licensing-for-industries?rtc=1&activetab=licensing-for-industries-pivot:primaryr3) peuvent être exonérés de frais de sortie. Pour en savoir plus, contactez votre responsable de compte. 

Pour plus d’informations, consultez « Quels sont les programmes de transfert de données destinés aux clients du secteur de l’enseignement et les critères d’éligibilité ? » dans la section FAQ de la page [Programmes destinés aux établissements d’enseignement](https://azure.microsoft.com/pricing/details/bandwidth/).

#### <a name="pricing-example"></a>Exemple de tarif

Examinons un exemple de coût d’enregistrement portant sur une image de modèle de machine virtuelle dans une galerie d’images partagées. Les scénarios sont les suivants :

- Vous disposez d’une seule image de machine virtuelle personnalisée.
- Vous enregistrez deux versions de l’image.
- Votre labo se trouve aux États-Unis, où il existe au total huit régions.
- La taille de chaque version d’image est de 32 Go ; par conséquent, le coût du disque managé HDD est de 1,54 USD par mois.

Le coût total estimé est calculé comme suit :

* *Nombre d’images &times; nombre de versions &times; nombre de réplicas &times; prix du disque managé = coût total par mois*

Dans cet exemple, le coût est le suivant :

* 1 image personnalisée (32 Go) &times; 2 versions &times; 8 régions des États-Unis &times; 1,54 USD = 24,64 USD par mois

> [!NOTE]
> Le calcul précédent est fourni à titre d’exemple uniquement. Il couvre les coûts de stockage associés à l’utilisation du service Shared Image Gallery et n’inclut *pas* les coûts de sortie. Pour obtenir la tarification réelle du stockage, consultez [Tarification des disques managés](https://azure.microsoft.com/en-us/pricing/details/managed-disks/).

#### <a name="cost-management"></a>la gestion des coûts ;

L’administrateur de compte lab doit contenir les coûts en supprimant régulièrement les versions d’images inutiles de la galerie. 

Ne supprimez pas la réplication vers des régions spécifiques dans le but de réduire les coûts, même si cette option est possible dans la galerie d’images partagées. Le fait de modifier la réplication peut avoir des conséquences négatives sur la capacité d’Azure Lab Services à publier des machines virtuelles à partir d’images enregistrées dans une galerie d’images partagées.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la configuration et la gestion des labos, consultez :

- [Guide de configuration de compte de laboratoire](account-setup-guide.md)  
- [Guide de configuration d’un laboratoire](setup-guide.md)  
- [Gestion des coûts pour les labos](cost-management-guide.md)  
- [Utiliser Azure Lab Services dans Teams](lab-services-within-teams-overview.md)
