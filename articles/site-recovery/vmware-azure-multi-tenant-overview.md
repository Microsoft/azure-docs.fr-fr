---
title: Récupération d’urgence multilocataire de machines virtuelles VMware avec Azure Site Recovery
description: Offre une vue d’ensemble de la prise en charge d’Azure Site Recovery pour la reprise d’activité de VMware sur Azure dans un programme d’environnement multilocataire (CSP).
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: mayg
ms.openlocfilehash: 64e40341ec56a2e1c561b2bcbb5e584830c14015
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93145580"
---
# <a name="overview-of-multi-tenant-support-for-vmware-disaster-recovery-to-azure-with-csp"></a>Vue d’ensemble de la prise en charge de l’architecture multilocataire pour la reprise d’activité de VMware sur Azure avec CSP

[Azure Site Recovery](site-recovery-overview.md) prend en charge les environnements multilocataires pour les abonnements de locataire. Il prend également en charge une architecture multilocataire pour les abonnements de locataire qui sont créés et gérés via le programme Fournisseur de solutions cloud (CSP) Microsoft.

Cet article offre une vue d’ensemble de l’implémentation et de la gestion de la réplication de l’architecture multilocataire VMware vers Azure.

## <a name="multi-tenant-environments"></a>Environnements multilocataires

Il existe trois modèles multilocataires principaux :

* **Fournisseur de services d’hébergement partagé (HSP)** : le partenaire possède l’infrastructure physique et utilise des ressources partagées (vCenter, centres de données, stockage physique, etc.) pour héberger les machines virtuelles de plusieurs locataires sur la même infrastructure. Le partenaire peut fournir la gestion de la récupération d’urgence en tant que service géré, ou le locataire peut disposer de la récupération d’urgence en tant que solution en libre-service.

* **Fournisseur de services d’hébergement dédié** : le partenaire possède l’infrastructure physique mais utilise des ressources dédiées (plusieurs vCenter, magasins de données physiques, etc.) pour héberger les machines virtuelles de chaque locataire sur une infrastructure distincte. Le partenaire peut fournir la gestion de la récupération d’urgence en tant que service géré, ou le locataire peut en disposer en tant que solution en libre-service.

* **Fournisseur de services gérés (MSP)** : le client possède l’infrastructure physique qui héberge les machines virtuelles, et le partenaire réalise l’activation et la gestion de la reprise d’activité.

## <a name="shared-hosting-services-provider-hsp"></a>Fournisseur de services d’hébergement partagé (HSP)

Les deux autres scénarios sont des sous-ensembles du scénario d’hébergement partagé et ils reposent sur les mêmes principes. Les différences sont décrites à la fin des conseils d’hébergement partagé.

L’exigence de base dans un scénario de multilocation est que les locataires soient isolés. Un locataire ne doit pas être capable de voir ce qu’un autre locataire a hébergé. Dans un environnement géré par un partenaire, cette exigence n’est pas aussi importante que dans un environnement en libre-service où elle peut être critique. Cet article part du principe que l’isolation des locataires est requise.

L’architecture est illustrée dans le diagramme suivant.

![HSP partagé avec un vCenter](./media/vmware-azure-multi-tenant-overview/shared-hosting-scenario.png)  

**Hébergement partagé avec un serveur vCenter**

Dans le diagramme, chaque client possède un serveur d’administration distinct. Cette configuration limite l’accès du locataire aux machines virtuelles propres au locataire et active l’isolation des locataires. Les machines virtuelles VMware utilisent le serveur de configuration pour découvrir des machines virtuelles et installer des agents. Les mêmes principes s’appliquent aux environnements multilocataires, en limitant en plus la détection de machines virtuelles à l’aide du contrôle d’accès vCenter.

L’exigence d’isolement des données signifie qu’aucune information sensible sur l’infrastructure (par exemple, les informations d’identification d’accès) ne soit divulguée aux locataires. Pour cette raison, nous recommandons que tous les composants du serveur d’administration restent sous le contrôle exclusif du partenaire. Les composants de serveur d’administration sont les suivants :

* Serveur de configuration
* Serveur de traitement
* Serveur cible maître

Un serveur de traitement non à l’échelle distinct est également contrôlé par le partenaire.

## <a name="configuration-server-accounts"></a>Comptes serveur de configuration

Dans le scénario multilocataire, chaque serveur de configuration utilise deux comptes :

- **Compte d’accès vCenter**  : ce compte est utilisé pour découvrir les machines virtuelles du locataire. Des autorisations d’accès vCenter lui sont associées. Afin d’éviter les fuites d’accès, nous recommandons aux partenaires d’entrer eux-mêmes ces informations d’identification dans l’outil de configuration.

- **Compte d’accès de machine virtuelle**  : ce compte est utilisé pour installer l’agent du service Mobilité sur les machines virtuelles du locataire, à l’aide d’une transmission automatique. Il s’agit généralement d’un compte de domaine qu’un locataire peut fournir à un partenaire ou un compte que le partenaire peut gérer directement. Si un locataire ne souhaite pas partager directement ses informations avec le partenaire, il peut entrer ses informations d’identification grâce à un accès limité au serveur de configuration. Avec l’aide du partenaire, il peut également l’agent du service Mobilité manuellement.

## <a name="vcenter-account-requirements"></a>Conditions requises pour le compte vCenter

Configurez le serveur de configuration avec un compte auquel un rôle spécial a été affecté.

- L’attribution de rôle doit être appliquée au compte d’accès vCenter pour chaque objet vCenter et elle ne doit pas être propagée sur les objets enfants. Cette configuration garantit l’isolation des locataires, car la propagation d’accès peut entraîner un accès accidentel à d’autres objets.

    ![L’option de propagation sur les objets enfants](./media/vmware-azure-multi-tenant-overview/assign-permissions-without-propagation.png)

- L’approche alternative consiste à affecter le compte d’utilisateur et le rôle à l’objet de centre de données et à les propager sur les objets enfants. Affectez ensuite au compte un rôle **Aucun accès** pour chaque objet (comme les machines virtuelles appartenant à d’autres locataires) qui doit être inaccessible à un locataire particulier. Cette configuration est fastidieuse. Elle entraîne des contrôles d’accès accidentels, car chaque nouvel objet enfant créé hérite automatiquement de l’accès de son parent. Nous vous recommandons donc d’utiliser la première approche.

### <a name="create-a-vcenter-account"></a>Créer un compte vCenter

1. Créez un nouveau rôle en clonant le rôle prédéfini *Lecture seule* , puis donnez-lui un nom pratique (tel que Azure_Site_Recovery comme indiqué dans cet exemple).
2. Attribuez les autorisations suivantes à ce rôle :

   * **Banque de données**  : Allouer de l’espace, Parcourir la banque de données, Opérations de fichier de bas niveau, Supprimer le fichier, Mettre à jour les fichiers de machine virtuelle
   * **Réseau**  : Attribution de réseau
   * **Ressource**  : Affecter les machines virtuelles au pool de ressources, Migrer des machines virtuelles hors tension, Migrer des machines virtuelles sous tension
   * **Tâches**  : Créer une tâche, Mettre à jour une tâche
   * **Machine virtuelle - Configuration**  : Tous
   * **Machine virtuelle - Interagir** > Répondre à la question, Connexion d’appareil, Configurer un support de CD, Configurer une disquette, Mettre hors tension, Mettre sous tension, Installation des outils VMware
   * **Machine virtuelle - Inventaire** > Créer à partir d’un existant, Créer, S’inscrire, Annuler l’inscription
   * **Machine virtuelle - Approvisionnement** > Autoriser le téléchargement de machines virtuelles, Autoriser le chargement de fichiers de machine virtuelle
   * **Machine virtuelle Gestion des instantanés** > Supprimer les instantanés

       ![La boîte de dialogue Modifier le rôle](./media/vmware-azure-multi-tenant-overview/edit-role-permissions.png)

3. Attribuez un niveau d’accès au compte vCenter (utilisé dans le serveur de configuration du locataire) pour divers objets comme suit :

>| Object | Role | Notes |
>| --- | --- | --- |
>| vCenter | Lecture seule | Nécessaire uniquement pour autoriser l’accès vCenter pour la gestion de différents objets. Vous pouvez supprimer cette autorisation si le compte ne sera jamais fourni à un locataire ou utilisé pour des opérations de gestion sur le vCenter. |
>| Centre de données | Azure_Site_Recovery |  |
>| Hôte et cluster hôte | Azure_Site_Recovery | Revérifiez que l’accès se trouve au niveau de l’objet de sorte que seuls les hôtes accessibles comprennent des machines virtuelles de locataire avant le basculement et après la restauration automatique. |
>| Banque de données et cluster de banque de données | Azure_Site_Recovery | Identique à ce qui précède. |
>| Réseau | Azure_Site_Recovery |  |
>| Serveur d’administration | Azure_Site_Recovery | Inclut l’accès à tous les composants (CS, PS et MT) qui ne se trouvent pas sur l’ordinateur du CS. |
>| Machines virtuelles de locataire | Azure_Site_Recovery | Garantit que les nouvelles machines virtuelles d’un locataire disposent également de cet accès. Sinon, elles ne pourront pas être découvertes via le portail Azure. |

L’accès au compte vCenter est maintenant terminé. Cette étape répond aux conditions minimales requises des autorisations pour effectuer des opérations de restauration. Vous pouvez également utiliser ces autorisations d’accès avec vos stratégies existantes. Modifiez simplement vos autorisations existantes pour inclure des autorisations de rôle de l’étape 2 détaillée précédemment.

### <a name="failover-only"></a>Basculement uniquement
Pour limiter les opérations de récupération d’urgence au basculement uniquement (à savoir sans les fonctionnalités de restauration automatique), utilisez la procédure précédente avec les exceptions suivantes :

- Au lieu d’affecter le rôle *Azure_Site_Recovery* au compte d’accès vCenter, affectez uniquement un rôle *Lecture seule* à ce compte. Ce jeu d’autorisations permet la réplication et le basculement de la machine virtuelle, et il n’autorise pas la restauration automatique.
- Tout le reste de la procédure précédente ne change pas. Pour garantir l’isolation des locataires et limiter la découverte de la machine virtuelle, chaque autorisation est toujours affectée au niveau de l’objet uniquement et n’est pas propagée sur les objets enfants.

### <a name="deploy-resources-to-the-tenant-subscription"></a>Déployer des ressources sur l’abonnement locataire

1. Sur le portail Azure, créez un groupe de ressources, puis déployez un coffre Recovery Services en suivant la procédure habituelle.
2. Téléchargez la clé d’inscription du coffre.
3. Inscrivez le serveur de configuration du locataire à l’aide de la clé d’inscription du coffre.
4. Entrez les informations d’identification des deux comptes, le compte permettant d’accéder au serveur vCenter et le compte permettant d’accéder à la machine virtuelle.

    ![Comptes serveur de configuration de gestionnaire](./media/vmware-azure-multi-tenant-overview/config-server-account-display.png)

### <a name="register-servers-in-the-vault"></a>Inscrire les serveurs dans le coffre

1. Dans le portail Azure, dans le coffre que vous avez créé précédemment, inscrivez le serveur vCenter auprès du serveur de configuration, en utilisant le compte vCenter que vous avez créé.
2. Terminez la « Préparation de l’infrastructure » pour la récupération de site en suivant la procédure habituelle.
3. Les machines virtuelles sont maintenant prêtes pour la réplication. Vérifiez que seuls les machines virtuelles du locataire apparaissent dans **Répliquer** > **Sélectionner des machines virtuelles**.

## <a name="dedicated-hosting-solution"></a>Solution d’hébergement dédié

Comme indiqué dans le diagramme suivant, la différence architecturale dans une solution d’hébergement dédié réside dans le fait que l’infrastructure de chaque locataire est configurée pour ce locataire uniquement.

![Diagramme illustrant que la différence architecturale dans une solution d’hébergement dédiée réside dans le fait que l’infrastructure de chaque locataire est configurée pour ce locataire uniquement.](./media/vmware-azure-multi-tenant-overview/dedicated-hosting-scenario.png)  
**Scénario d’hébergement dédié avec plusieurs vCenter**

## <a name="managed-service-solution"></a>Solution de service géré

Comme indiqué dans le diagramme suivant, la différence architecturale dans une solution de service géré réside dans le fait que l’infrastructure de chaque locataire est également séparée physiquement de l’infrastructure d’autres locataires. Ce scénario s’applique généralement lorsque le locataire possède l’infrastructure et souhaite disposer d’un fournisseur de solutions pour gérer la récupération d’urgence.

![architecture-hsp-partagé](./media/vmware-azure-multi-tenant-overview/managed-service-scenario.png)  
**Scénario de service géré avec plusieurs vCenter**

## <a name="next-steps"></a>Étapes suivantes
- [En savoir plus](site-recovery-role-based-linked-access-control.md) sur le contrôle d’accès en fonction du rôle dans Site Recovery.
- Découvrez comment configurer la récupération d’urgence des [machines virtuelles VMware](vmware-azure-tutorial.md) vers Azure.
- Apprenez-en davantage sur [l’Architecture mutualisée avec CSP pour les machines virtuelles VMWare](vmware-azure-multi-tenant-csp-disaster-recovery.md).
