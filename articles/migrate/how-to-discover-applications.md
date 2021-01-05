---
title: Découvrir des applications sur des serveurs locaux avec Azure Migrate
description: Apprenez à découvrir les applications, rôles et fonctionnalités sur des serveurs locaux à l’aide d’Azure Migrate Server Assessment.
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: how-to
ms.date: 06/10/2020
ms.openlocfilehash: eb589c08122cd47747c005c13d12b336319fd558
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752003"
---
# <a name="discover-machine-apps-roles-and-features"></a>Détection des applications, des rôles et des fonctionnalités de la machine

Cet article décrit comment découvrir les applications, rôles et fonctionnalités sur des serveurs locaux à l’aide d’Azure Migrate : Évaluation de serveurs.

La découverte de l’inventaire des applications, de rôles et de fonctionnalités sur des machines locales permet d’identifier et de personnaliser votre migration vers Azure pour vos charges de travail. La découverte des applications utilise l’appliance Azure Migrate pour effectuer la détection avec les informations d’identification de la machine virtuelle. La découverte des applications se fait sans agent. Rien n’est installé sur les machines virtuelles.

> [!NOTE]
> La détection d’applications est actuellement en préversion pour les machines virtuelles VMware uniquement et est limitée à la seule détection. Nous n’offrons pas encore d’évaluation basée sur les applications. 


## <a name="before-you-start"></a>Avant de commencer

- Assurez-vous d’avoir :
    - [Créé ](./create-manage-projects.md) un projet Azure Migrate.
    - [Ajouté](how-to-assess.md) l’outil Azure Migrate : Évaluation de serveurs au projet.
- Passez en revue [la prise en charge et les exigences de la découverte d’application](migrate-support-matrix-vmware.md#vmware-requirements).
- Assurez-vous que PowerShell version 2.0 ou ultérieure est installé sur les machines virtuelles sur lesquelles vous exécutez la découverte d’application et que les outils VMware (versions ultérieures à 10.2.0) sont installés.
- Vérifiez la [configuration requise](migrate-appliance.md) pour le déploiement de l’appliance Azure Migrate.


## <a name="deploy-the-azure-migrate-appliance"></a>Déployez l’appliance Azure Migrate

1. [Examinez](migrate-appliance.md#appliance---vmware) la configuration requise pour le déploiement de l’appliance Azure Migrate.
2. Passez en revue les URL Azure auxquelles l’appliance devra accéder dans les clouds [publics](migrate-appliance.md#public-cloud-urls) et les [clouds Government](migrate-appliance.md#government-cloud-urls).
3. [Passez en revue les données](migrate-appliance.md#collected-data---vmware) que l’appliance collecte pendant la découverte et l’évaluation.
4. [Notez](migrate-support-matrix-vmware.md#port-access-requirements) les conditions d’accès aux ports pour l’appliance.
5. [Déployez l’appliance Azure Migrate](how-to-set-up-appliance-vmware.md) pour démarrer la découverte. Pour déployer l’appliance, vous téléchargez et importez un modèle OVA dans VMware pour créer l’appliance en tant que machine virtuelle VMware. Vous configurez l’appliance, puis l’inscrivez auprès d’Azure Migrate.
6. Lorsque vous déployez l’appliance, pour démarrer la découverte continue, vous spécifiez les éléments suivants :
    - Le nom du vCenter Server auquel vous souhaitez vous connecter.
    - Les informations d’identification que vous avez créées pour que l’appliance se connecte à vCenter Server.
    - Les informations d’identification de compte que vous avez créées pour que l’appliance se connecte aux machines virtuelles Windows/Linux.

Une fois que l’appliance a été déployée et que vous avez fourni les informations d’identification, l’appliance démarre la découverte continue des métadonnées de la machine virtuelle et des données de performances, ainsi que la découverte des applications, des fonctionnalités et des rôles.  La durée de la découverte d’application dépend du nombre de machines virtuelles dont vous disposez. La découverte d’applications pour 500 machines virtuelles prend généralement une heure.

## <a name="verify-permissions"></a>Vérification des autorisations

Vous avez [créé un compte vCenter Server en lecture seule](./tutorial-discover-vmware.md#prepare-vmware) pour la découverte et l’évaluation. Le compte en lecture seule exige des privilèges pour **Machines virtuelles** > **Opérations d’invité** afin d’interagir avec la machine virtuelle pour la découverte d’application.

### <a name="add-the-user-account-to-the-appliance"></a>Ajouter le compte d’utilisateur à l’appliance

Ajoutez le compte d’utilisateur comme suit :

1. Ouvrez l’application de gestion de l’appliance. 
2. Accédez au panneau **Fournir les détails de vCenter**.
3. Dans **Découvrir l’application et les dépendances sur les machines virtuelles**, cliquez sur **Ajouter les informations d’identification**.
3. Choisissez le **système d’exploitation**, fournissez un nom convivial pour le compte, puis le **nom d’utilisateur**/**mot de passe**.
6. Cliquez sur **Enregistrer**.
7. Cliquez sur **Enregistrer et démarrer la découverte**.

    ![Ajouter un compte d’utilisateur de machine virtuelle](./media/how-to-create-group-machine-dependencies-agentless/add-vm-credential.png)


## <a name="review-and-export-the-inventory"></a>Examiner et exporter l’inventaire

Une fois la détection terminée, si vous avez fourni des informations d’identification pour la découverte d’application, vous pouvez passer en revue et exporter l’inventaire des applications dans le portail Azure.

1. Dans la page **Azure Migrate - Serveurs** > **Azure Migrate : Évaluation de serveur**, cliquez sur le nombre affiché pour ouvrir la page **Serveurs découverts**.

    > [!NOTE]
    > À ce stade, vous pouvez également configurer l’analyse des dépendances pour les ordinateurs découverts, afin de pouvoir visualiser les dépendances entre les machines que vous souhaitez évaluer. [En savoir plus](concepts-dependency-visualization.md) sur l’analyse des dépendances.

2. Dans **Applications découvertes**, cliquez sur le nombre affiché.
3. Dans **Inventaire des applications**, vous pouvez passer en revue les applications, les fonctionnalités et les rôles découverts.
4. Pour exporter l’inventaire, dans **Serveurs détectés**, cliquez sur **Exporter l’inventaire des applications**.

L’inventaire des applications est exporté et téléchargé au format Excel. La feuille **Inventaire des applications** affiche toutes les applications découvertes sur toutes les machines.

## <a name="next-steps"></a>Étapes suivantes

- [Créez une évaluation](how-to-create-assessment.md) pour les serveurs découverts.
- Évaluez des bases de données SQL Server avec [Azure Migrate : Évaluation de serveurs](/sql/dma/dma-assess-sql-data-estate-to-sqldb?view=sql-server-2017).