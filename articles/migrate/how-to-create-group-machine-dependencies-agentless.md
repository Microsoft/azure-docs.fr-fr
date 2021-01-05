---
title: Configurer l’analyse des dépendances sans agent dans l’outil d’évaluation de serveur Azure Migrate
description: Configurez l’analyse des dépendances sans agent dans l’outil d’évaluation de serveur Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: how-to
ms.date: 6/08/2020
ms.openlocfilehash: d84c85326c6f5d87189a2c24a3b13654f157cb05
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754281"
---
# <a name="analyze-machine-dependencies-agentless"></a>Analyser les dépendances des machines (sans agent)

Cet article explique comment configurer l’analyse des dépendances sans agent dans l’outil Azure Migrate : évaluation de serveur. L’[analyse des dépendances](concepts-dependency-visualization.md) vous permet d’identifier et de comprendre les dépendances entre les machines pour l’évaluation et la migration vers Azure.


> [!IMPORTANT]
> La visualisation des dépendances sans agent est actuellement en préversion pour les machines virtuelles VMware découvertes avec l’outil Azure Migrate : Server Assessment.
> Les fonctionnalités peuvent être limitées ou incomplètes.
> Cette préversion est couverte par le support client et peut être utilisée pour les charges de travail de production.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="current-limitations"></a>Limites actuelles

- Dans la vue d’analyse des dépendances, actuellement vous ne pouvez pas ajouter ou supprimer de serveur dans un groupe.
- Aucune carte des dépendances pour un groupe de serveurs n’est disponible actuellement.
- Dans un projet Azure Migrate, la collecte de données de dépendance peut être configurée pour 1 000 serveurs simultanément. Vous pouvez analyser un plus grand nombre de serveurs par un séquençage en lots de 1 000.

## <a name="before-you-start"></a>Avant de commencer

- [Passez en revue](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless) les systèmes d’exploitation pris en charge et les autorisations requises.
- Vérifiez les points suivants :
    - Vous disposez d’un projet Azure Migrate. Si ce n’est pas le cas, [créez-en un](./create-manage-projects.md) maintenant.
    - Vous avez [ajouté](how-to-assess.md) l’outil Azure Migrate : Server Assessment au projet.
    - Vous avez configuré une [appliance Azure Migrate](migrate-appliance.md) pour découvrir les machines locales. Vous avez [configuré une appliance](how-to-set-up-appliance-vmware.md) pour les machines virtuelles VMware. L’appliance découvre les machines locales, puis envoie les métadonnées et les données de performances à Azure Migrate : évaluation de serveur.
- Vérifiez que VMware Tools (version supérieure à 10.2) est installé sur chaque machine virtuelle que vous souhaitez analyser.


## <a name="create-a-user-account-for-discovery"></a>Créer un compte d’utilisateur pour la découverte

Configurez un compte d’utilisateur pour que Server Assessment puisse accéder à la machine virtuelle afin de découvrir les dépendances. [Apprenez-en davantage](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless) sur les exigences relatives aux comptes pour les machines virtuelles Windows et Linux.


## <a name="add-the-user-account-to-the-appliance"></a>Ajouter le compte d’utilisateur à l’appliance

Ajoutez le compte d’utilisateur à l’appliance.

1. Ouvrez l’application de gestion de l’appliance. 
2. Accédez au panneau **Fournir les détails de vCenter**.
3. Dans **Découvrir l’application et les dépendances sur les machines virtuelles**, cliquez sur **Ajouter les informations d’identification**.
3. Choisissez le **système d’exploitation**, fournissez un nom convivial pour le compte, puis le **nom d’utilisateur**/**mot de passe**.
6. Cliquez sur **Enregistrer**.
7. Cliquez sur **Enregistrer et démarrer la découverte**.

    ![Ajouter un compte d’utilisateur de machine virtuelle](./media/how-to-create-group-machine-dependencies-agentless/add-vm-credential.png)

## <a name="start-dependency-discovery"></a>Démarrer la découverte de dépendance

Choisissez les machines sur lesquelles vous souhaitez activer la découverte de dépendance. 

1. Dans **Azure Migrate : Server Assessment**, cliquez sur **Serveurs découverts**.
2. Cliquez sur l’icône **Analyse des dépendances**.
3. Cliquez sur **Ajouter des serveurs**.
4. Dans la page **Ajouter des serveurs**, choisissez l’appliance qui découvre les machines appropriées.
5. Dans la liste des machines, opérez votre sélection.
6. Cliquez sur **Ajouter des serveurs**.

    ![Démarrer la découverte de dépendance](./media/how-to-create-group-machine-dependencies-agentless/start-dependency-discovery.png)

Vous pouvez visualiser les dépendances environ six heures après le démarrage de la découverte des dépendances. Si vous souhaitez activer plusieurs ordinateurs, vous pouvez utiliser [PowerShell](#start-or-stop-dependency-discovery-using-powershell).

## <a name="visualize-dependencies"></a>Visualiser les dépendances

1. Dans **Azure Migrate : Server Assessment**, cliquez sur **Serveurs découverts**.
2. Recherchez la machine que vous souhaitez afficher.
3. Dans la colonne **Dépendances**, cliquez sur **Afficher les dépendances**.
4. Modifiez la période pendant laquelle vous souhaitez afficher la carte à l’aide de la liste déroulante **Durée**.
5. Développez le groupe **Client** pour afficher la liste des machines qui ont une dépendance sur la machine sélectionnée.
6. Développez le groupe **Port** pour répertorier les machines qui ont une dépendance de la machine sélectionnée.
7. Pour accéder à la carte d’une machine dépendante, cliquez sur le nom de la machine > **Charger la carte des serveurs**.

    ![Développer un groupe de ports du serveur et charger une carte des serveurs](./media/how-to-create-group-machine-dependencies-agentless/load-server-map.png)

    ![Développer un groupe de clients ](./media/how-to-create-group-machine-dependencies-agentless/expand-client-group.png)

8. Développez la machine sélectionnée pour afficher les détails au niveau processus pour chaque dépendance.

    ![Développer un serveur pour afficher les processus](./media/how-to-create-group-machine-dependencies-agentless/expand-server-processes.png)

> [!NOTE]
> Les informations de processus pour une dépendance ne sont pas toujours disponibles. Si elles ne sont pas disponible, la dépendance est représentée par le processus marqué comme « Processus inconnu ».

## <a name="export-dependency-data"></a>Exporter les données de dépendance

1. Dans **Azure Migrate : Server Assessment**, cliquez sur **Serveurs découverts**.
2. Cliquez sur l’icône **Analyse des dépendances**.
3. Cliquez sur **Exporter les dépendances d’application**.
4. Dans la page **Exporter les dépendances d’application**, choisissez l’appliance qui découvre les machines appropriées.
5. Sélectionnez l’heure de début et l’heure de fin. Notez que vous pouvez télécharger les données uniquement pour les 30 derniers jours.
6. Cliquez sur **Exporter la dépendance**.

Les données de dépendances sont exportées et téléchargées dans un format CSV. Le fichier téléchargé contient les données de dépendances sur toutes les machines activées pour l’analyse des dépendances. 

![Exporter les dépendances](./media/how-to-create-group-machine-dependencies-agentless/export.png)

### <a name="dependency-information"></a>Informations sur les dépendances

Chaque ligne du fichier CSV exporté correspond à une dépendance observée dans le créneau spécifié. 

Le tableau suivant récapitule les champs du fichier CSV exporté. Notez que les champs de nom de serveur, d’application et de processus sont uniquement renseignés pour les serveurs sur lesquels l’analyse des dépendances sans agent est activée.

**Nom du champ** | **Détails**
--- | --- 
Créneau | Créneau pendant lequel la dépendance a été observée. <br/> Les données de dépendances sont actuellement capturées sur des créneaux de six heures.
Nom du serveur source | Nom de la machine source 
Application source | Nom de l’application sur la machine source 
Processus source | Nom du processus sur la machine source 
Nom du serveur de destination | Nom de la machine de destination
IP de destination | Adresse IP de la machine de destination
Application de destination | Nom de l’application sur la machine de destination
Processus de destination | Nom du processus sur la machine de destination 
Port de destination | Numéro de port sur la machine de destination


## <a name="stop-dependency-discovery"></a>Arrêter la découverte de dépendance

Choisissez les machines sur lesquelles vous souhaitez arrêter la découverte de dépendance. 

1. Dans **Azure Migrate : Server Assessment**, cliquez sur **Serveurs découverts**.
2. Cliquez sur l’icône **Analyse des dépendances**.
3. Cliquez sur **Supprimer des serveurs**.
3. Dans la page **Supprimer des serveurs**, choisissez l’**appliance** qui découvre les machines virtuelles sur lesquelles vous souhaitez arrêter la découverte de dépendance.
4. Dans la liste des machines, opérez votre sélection.
5. Cliquez sur **Supprimer des serveurs**.

Si vous souhaitez interrompre la dépendant sur plusieurs ordinateurs, vous pouvez utiliser [PowerShell](#start-or-stop-dependency-discovery-using-powershell).


## <a name="start-or-stop-dependency-discovery-using-powershell"></a>Démarrer ou arrêter la découverte de dépendances à l’aide de PowerShell

Téléchargez le module PowerShell à partir du dépôt [Exemples Azure PowerShell](https://github.com/Azure/azure-docs-powershell-samples/tree/master/azure-migrate/dependencies-at-scale) sur GitHub.


### <a name="log-in-to-azure"></a>Connexion à Azure

1. Connectez-vous à votre abonnement Azure à l’aide de l’applet de commande Connect-AzAccount.

    ```PowerShell
    Connect-AzAccount
    ```
    Si vous utilisez Azure Government, utilisez la commande suivante.
    ```PowerShell
    Connect-AzAccount -EnvironmentName AzureUSGovernment
    ```

2. Sélectionnez l’abonnement dans lequel vous avez créé le projet Azure Migrate 

    ```PowerShell
    select-azsubscription -subscription "Fabrikam Demo Subscription"
    ```

3. Importer le module PowerShell AzMig_Dependencies téléchargé

    ```PowerShell
    Import-Module .\AzMig_Dependencies.psm1
    ```

### <a name="enable-or-disable-dependency-data-collection"></a>Activer ou désactiver la collecte de données de dépendance

1. Récupérez la liste des machines virtuelles VMware détectées dans votre projet Azure Migrate à l’aide des commandes suivantes. Dans l’exemple ci-dessous, le nom du projet est FabrikamDemoProject et le groupe de ressources auquel il appartient est FabrikamDemoRG. La liste des machines sera enregistrée dans FabrikamDemo_VMs.csv

    ```PowerShell
    Get-AzMigDiscoveredVMwareVMs -ResourceGroupName "FabrikamDemoRG" -ProjectName "FabrikamDemoProject" -OutputCsvFile "FabrikamDemo_VMs.csv"
    ```

    Dans le fichier, vous pouvez voir le nom complet de la machine virtuelle, l’état actuel de la collection de dépendances et l’ID ARM de toutes les machines virtuelles découvertes. 

2. Pour activer ou désactiver les dépendances, créez un fichier CSV d’entrée. Le fichier doit contenir une colonne avec l’en-tête « ID ARM ». Tous les en-têtes supplémentaires du fichier CSV seront ignorés. Vous pouvez créer le CSV à l’aide du fichier généré à l’étape précédente. Créez une copie du fichier en conservant les machines virtuelles sur lesquelles vous souhaitez activer ou désactiver les dépendances. 

    Dans l’exemple suivant, l’analyse des dépendances est activée sur la liste des machines virtuelles dans le fichier d’entrée FabrikamDemo_VMs_Enable.csv.

    ```PowerShell
    Set-AzMigDependencyMappingAgentless -InputCsvFile .\FabrikamDemo_VMs_Enable.csv -Enable
    ```

    Dans l’exemple suivant, l’analyse des dépendances est désactivée sur la liste des machines virtuelles dans le fichier d’entrée FabrikamDemo_VMs_Disable.csv.

    ```PowerShell
    Set-AzMigDependencyMappingAgentless -InputCsvFile .\FabrikamDemo_VMs_Disable.csv -Disable
    ```

## <a name="visualize-network-connections-in-power-bi"></a>Visualiser les connexions réseau dans Power BI

Azure Migrate propose un modèle de Power BI que vous pouvez utiliser pour visualiser simultanément les connexions réseau de nombreux serveurs, et filtrer par processus et par serveur. Pour visualiser, chargez le Power BI avec des données de dépendance, comme indiqué dans les instructions ci-dessous.

1. Téléchargez le module PowerShell et le modèle Power Bi à partir du dépôt [Exemples Azure PowerShell](https://github.com/Azure/azure-docs-powershell-samples/tree/master/azure-migrate/dependencies-at-scale) sur GitHub.

2. Connectez-vous à Azure à l’aide des instructions ci-dessous : 
- Connectez-vous à votre abonnement Azure à l’aide de l’applet de commande Connect-AzAccount.

    ```PowerShell
    Connect-AzAccount
    ```

- Si vous utilisez Azure Government, utilisez la commande suivante.

    ```PowerShell
    Connect-AzAccount -EnvironmentName AzureUSGovernment
    ```

- Sélectionnez l’abonnement dans lequel vous avez créé le projet Azure Migrate 

    ```PowerShell
    select-azsubscription -subscription "Fabrikam Demo Subscription"
    ```

3. Importer le module PowerShell AzMig_Dependencies téléchargé

    ```PowerShell
    Import-Module .\AzMig_Dependencies.psm1
    ```

4. Exécutez la commande suivante : Cette commande télécharge les données de dépendances dans un CSV et les traite pour générer une liste de dépendances uniques qui peuvent être utilisées pour la visualisation dans Power BI. Dans l’exemple ci-dessous, le nom du projet est FabrikamDemoProject et le groupe de ressources auquel il appartient est FabrikamDemoRG. Les dépendances sont téléchargées pour les machines découvertes par FabrikamAppliance. Les dépendances uniques seront enregistrées dans FabrikamDemo_Dependencies.csv

    ```PowerShell
    Get-AzMigDependenciesAgentless -ResourceGroup FabrikamDemoRG -Appliance FabrikamAppliance -ProjectName FabrikamDemoProject -OutputCsvFile "FabrikamDemo_Dependencies.csv"
    ```

5. Ouvrir le modèle Power BI téléchargé

6. Chargez les données de dépendance téléchargées dans Power BI.
    - Ouvrez le modèle dans Power BI.
    - Cliquez sur **Obtenir les données** dans la barre d’outils. 
    - Choisissez **Texte/CSV** à partir de sources de données communes.
    - Choisissez le fichier de dépendances téléchargé.
    - Cliquez sur **Charger**.
    - Vous verrez qu’une table est importée avec le nom du fichier CSV. Vous pouvez voir la table dans la barre des champs située à droite. Renommez-la AzMig_Dependencies
    - Cliquez sur Actualiser dans la barre d’outils.

    Le graphique Connexions réseau et le nom du serveur source, le nom du serveur de destination, le nom du processus source, les segments de nom du processus de destination doivent s’allumer avec les données importées.

7. Visualisez la carte du filtrage des connexions réseau par les serveurs et les processus. Enregistrez votre fichier.


## <a name="next-steps"></a>Étapes suivantes

[Grouper des machines](how-to-create-a-group.md) pour l’évaluation.