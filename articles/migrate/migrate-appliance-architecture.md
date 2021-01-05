---
title: Architecture de l’appliance Azure Migrate
description: Présente une vue d’ensemble de l’utilisation de l’appliance Azure Migrate pour l’évaluation et la migration de serveurs.
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 06/09/2020
ms.openlocfilehash: 42d4a722be25eec4b3e27012350346018fdba0f3
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754111"
---
# <a name="azure-migrate-appliance-architecture"></a>Architecture de l’appliance Azure Migrate

Cet article décrit l’architecture et les processus de l’appliance Azure Migrate. L’appliance Azure Migrate est une appliance légère déployée localement qui permet de découvrir les machines virtuelles et les serveurs physiques à migrer vers Azure. 

## <a name="deployment-scenarios"></a>Scénarios de déploiement

L’appliance Azure Migrate est utilisée dans les scénarios suivants.

**Scénario** | **Outil** | **Utilisé pour** 
--- | --- | ---
**Évaluation des machines virtuelles VMware** | Azure Migrate : Évaluation de serveur | Découvrir des machines virtuelles VMware.<br/><br/> Découvrir des applications et des dépendances de machine.<br/><br/> Collecter les métadonnées des machines et les métadonnées de performances et les envoyer à Azure.
**Migration de machines virtuelles VMware (sans agent)** | Azure Migrate : Évaluation de serveur | Détection des machines virtuelles VMware<br/><br/>  Répliquer des machines virtuelles VMware avec une [migration sans agent](server-migrate-overview.md).
**Évaluation des machines virtuelles Hyper-V** | Azure Migrate : Évaluation de serveur | Découvrir des machines virtuelles Hyper-V.<br/><br/> Collecter les métadonnées des machines et les métadonnées de performances et les envoyer à Azure.
**Machine physique** |  Azure Migrate : Évaluation de serveur |  Découvrir des serveurs physiques.<br/><br/> Collecter les métadonnées des machines et les métadonnées de performances et les envoyer à Azure.

## <a name="appliance-components"></a>Composants de l’appliance

L’appliance est dotée d’un certain nombre de composants.

- **Application de gestion** : il s’agit d’une application web pour l’entrée des utilisateurs pendant le déploiement de l’appliance. Utilisation lors de l’évaluation des machines pour la migration vers Azure.
- **Agent de découverte** : l’agent collecte les données de configuration de la machine. Utilisation lors de l’évaluation des machines pour la migration vers Azure. 
- **Agent collecteur** : l’agent collecte les données de performances. Utilisation lors de l’évaluation des machines pour la migration vers Azure.
- **Agent DRA** : orchestre la réplication des machines virtuelles et coordonne la communication entre les machines répliquées et Azure. Utilisation seulement lors de la réplication de machines virtuelles VMware sur Azure avec la migration sans agent.
- **Passerelle** : envoie des données répliquées vers Azure. Utilisation seulement lors de la réplication de machines virtuelles VMware sur Azure avec la migration sans agent.
- **Service de mise à jour automatique** : met à jour les composants de l’appliance (s’exécute toutes les 24 heures).



## <a name="appliance-deployment"></a>Étapes de déploiement d’appliance

- L’appliance Azure Migrate peut être configurée à l’aide d’un modèle pour [Hyper-V](how-to-set-up-appliance-hyper-v.md) ou [VMware](how-to-set-up-appliance-vmware.md), ou à l’aide d’un programme d’installation de script PowerShell pour [VMware/Hyper-V](deploy-appliance-script.md) et pour les [serveurs physiques](how-to-set-up-appliance-physical.md). 
- La configuration requise pour la prise en charge de l’appliance et les conditions préalables au déploiement sont résumées dans la [matrice de prise en charge de l’appliance](migrate-appliance.md).


## <a name="appliance-registration"></a>Inscription de l’appliance

Lors de la configuration de l’appliance, vous inscrivez l’appliance avec Azure Migrate, et les actions résumées dans le tableau se produisent.

**Action** | **Détails** | **autorisations**
--- | --- | ---
**Inscrire les fournisseurs de source** | Ces fournisseurs de ressources sont enregistrés dans l’abonnement que vous choisissez lors de la configuration de l’appliance : Microsoft.OffAzure, Microsoft.Migrate et Microsoft.KeyVault.<br/><br/> L’inscription d’un fournisseur de ressources configure votre abonnement pour travailler avec le fournisseur de ressources. | Pour inscrire les fournisseurs de ressources, vous avez besoin d’un rôle Contributeur ou Propriétaire sur l’abonnement.
**Créer une application de communication Azure AD** | Azure Migrate crée une application Azure AD (Azure Active Directory). Celle-ci est utilisée pour la communication (authentification et autorisation) entre les agents s’exécutant sur l’appliance et leurs services respectifs s’exécutant sur Azure.<br/><br/> Cette application n’a pas les privilèges nécessaires pour effectuer des appels Azure Resource Manager ou des accès Azure RBAC sur une ressource. | Vous avez besoin de [ces autorisations](./tutorial-discover-vmware.md#prepare-an-azure-user-account) pour qu’Azure Migrate crée l’application.
**Créer une application de coffre de clés Azure AD** | Cette application est créée uniquement pour la migration sans agent des machines virtuelles VMware vers Azure.<br/><br/> Elle est exclusivement utilisée pour accéder au coffre de clés créé dans l’abonnement de l’utilisateur pour la migration sans agent.<br/><br/> Elle dispose d’un accès Azure RBAC sur le coffre de clés Azure (créé dans le locataire du client) lorsque la découverte est lancée à partir de l’appliance. | Vous avez besoin de [ces autorisations](./tutorial-discover-vmware.md#prepare-an-azure-user-account) pour qu’Azure Migrate crée l’application.



## <a name="collected-data"></a>Données collectées

Les données collectées par le client pour tous les scénarios de déploiement sont résumées dans la [matrice de prise en charge de l’appliance](migrate-appliance.md).

## <a name="discovery-and-collection-process"></a>Processus de découverte et de collecte

![Architecture](./media/migrate-appliance-architecture/architecture1.png)

L'appliance communique avec les instances vCenter Server et les hôtes/grappes Hyper-V via le processus suivant.

1. **Démarrer la découverte** :
    - Lorsque vous lancez la découverte sur l’appliance Hyper-V, celle-ci communique avec les hôtes Hyper-V sur les ports WinRM 5985 (HTTP).
    - Lorsque vous lancez la découverte sur l'appliance VMware, elle communique par défaut avec vCenter Server sur le port TCP 443. Si vCenter Server écoute sur un port différent, vous pouvez le configurer dans l'application web de l'appliance.
2. **Recueillir des métadonnées et des données de performances** :
    - L’appliance utilise une session CIM (Common Information Model) pour collecter les données des machines virtuelles Hyper-V de l’hôte Hyper-V sur les ports 5985.
    - Par défaut, l'appliance communique avec le port 443 pour collecter les données des machines virtuelles VMware à partir de vCenter Server.
3. **Envoyer des données** : L’appliance envoie les données collectées aux outils Évaluation de serveur Azure Migrate et Migration de serveur Azure Migrate via le port SSL 443. L’appliance peut se connecter à Azure via Internet ou via ExpressRoute (nécessite un Peering Microsoft).
    - Pour les données de performances, l’appliance collecte les données d'utilisation en temps réel.
        - Les données de performances sont collectées toutes les 20 secondes pour VMware et toutes les 30 secondes pour Hyper-V, pour chaque mesure de performance.
        - Les données collectées sont regroupées pour créer un point de données unique pendant 10 minutes.
        - La valeur d'utilisation maximale est choisie parmi tous les points de données de 20/30 secondes puis envoyée à Azure pour le calcul de l'évaluation.
        - Selon la valeur du percentile spécifiée dans les propriétés de l’évaluation (50e/90e/95e/99e), les points de dix minutes sont triés par ordre croissant, et la valeur appropriée du percentile sert à calculer l'évaluation
    - Pour Migration du serveur, l'appliance commence à collecter les données des machines virtuelles et les réplique vers Azure.
4. **Évaluer et migrer** : Vous pouvez maintenant créer des évaluations à partir des métadonnées collectées par l'appliance en utilisant l’outil Évaluation du serveur Azure Migrate. En outre, vous pouvez également commencer à migrer des machines virtuelles VMware en utilisant l’outil Migration de serveur Azure Migrate pour orchestrer la réplication des machines virtuelles sans agent.

## <a name="appliance-upgrades"></a>Mises à niveau d’appliance

L'appliance est mise à niveau à mesure que les agents Azure Migrate fonctionnant sur l'appliance sont mis à jour. Ceci se produit automatiquement, car la mise à jour automatique est activée par défaut sur l’appliance. Vous pouvez modifier ce paramètre par défaut pour mettre à jour les agents manuellement.

Vous désactivez la mise à jour automatique dans le registre en définissant la clé HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance "AutoUpdate" sur 0 (DWORD).


## <a name="next-steps"></a>Étapes suivantes

[Passez en revue](migrate-appliance.md) la matrice de prise en charge de l’appliance.