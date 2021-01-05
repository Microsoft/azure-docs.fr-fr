---
title: Présentation de l’agent Connected Machine Windows
description: Cet article fournit une présentation détaillée de l’agent des serveurs avec Azure Arc disponible, qui prend en charge la surveillance de machines virtuelles hébergées dans des environnements hybrides.
ms.date: 12/15/2020
ms.topic: conceptual
ms.openlocfilehash: 531041b7d7439dd2a48fa9e06eb82796f470e9ed
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563022"
---
# <a name="overview-of-azure-arc-enabled-servers-agent"></a>Présentation de l’agent des serveurs activés par Azure Arc

L’agent Connected Machine des serveurs activés par Azure Arc vous permet de gérer vos machines Windows et Linux hébergées en dehors d’Azure sur votre réseau d’entreprise ou un autre fournisseur de cloud. Cet article propose une présentation détaillée des exigences en matière d'agent, de système et de réseau, ainsi que des différentes méthodes de déploiement.

>[!NOTE]
>À partir de la version générale des serveurs Azure Arc activés en septembre 2020, toutes les versions préliminaires d’Azure Connected Machine Agent (les agents dont la version est inférieure à 1.0) sont **déconseillées** d’ici le **2 février 2021**.  Ce délai d’exécution vous permet d’effectuer une mise à niveau vers la version 1 0 ou une version ultérieure avant que les agents prédéfinis ne soient plus en mesure de communiquer avec le service Azure Arc activé.

## <a name="agent-component-details"></a>Détails du composant Agent

Le package d’agent Azure Connected Machine contient plusieurs composants logiques qui sont regroupés ensemble.

* Le service HIMDS (Hybrid Instance Metadata service) gère la connexion à Azure et l’identité Azure de l’ordinateur connecté.

* L’agent Guest Configuration fournit les fonctionnalités de stratégie dans l’invité (In-Guest Policy) et de configuration de l’invité (Guest Configuration), comme l’évaluation de la conformité de l’ordinateur avec les stratégies requises.

    Notez le comportement suivant avec Azure Policy [Guest Configuration](../../governance/policy/concepts/guest-configuration.md) pour un ordinateur déconnecté :

    * Une attribution de stratégie Guest Configuration qui cible les ordinateurs déconnectés n’est pas affectée.
    * L’attribution d’invité est stockée localement pendant 14 jours. Pendant la période de 14 jours, si l’agent Connected Machine se reconnecte au service, les attributions de stratégie sont réappliquées.
    * Les attributions sont supprimées au bout de 14 jours et ne sont pas réattribuées à l’ordinateur après la période de 14 jours.

* L’agent Extension gère les extensions de machine virtuelle, notamment l’installation, la désinstallation et la mise à niveau. Les extensions sont téléchargées à partir d’Azure et copiées dans le dossier `%SystemDrive%\%ProgramFiles%\AzureConnectedMachineAgent\ExtensionService\downloads` sur Windows et `/opt/GC_Ext/downloads` pour Linux. Sur Windows, l’extension est installée dans le chemin suivant `%SystemDrive%\Packages\Plugins\<extension>` et sur Linux, l’extension est installée dans `/var/lib/waagent/<extension>`.

## <a name="download-agents"></a>Télécharger les agents

Vous pouvez télécharger le package de l’agent Azure Connected Machine pour Windows et Linux à partir des emplacements listés ci-dessous.

* [Package Windows Installer de l’agent pour Windows](https://aka.ms/AzureConnectedMachineAgent) à partir du centre de téléchargement Microsoft.

* Le package de l’agent pour Linux est distribué à partir du [dépôt de packages](https://packages.microsoft.com/) de Microsoft à l’aide du format de package préféré pour la distribution (.RPM ou .DEB).

L’agent Azure Connected Machine pour Windows et Linux peut être mis à niveau vers la dernière version de façon manuelle ou automatique selon vos besoins. Vous pourrez trouver plus d’informations [ici](manage-agent.md).

## <a name="prerequisites"></a>Prérequis

### <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Les versions suivantes des systèmes d’exploitation Windows et Linux sont officiellement prises en charge pour l’agent Azure Connected Machine :

- Windows Server 2012 R2 et versions ultérieures (y compris Windows Server Core)
- Ubuntu 16.04 et 18.04 LTS (x64)
- CentOS Linux 7 (x64)
- SUSE Linux Enterprise Server (SLES) 15 (x64)
- Red Hat Enterprise Linux (RHEL) 7 (x64)
- Amazon Linux 2 (x64)

> [!WARNING]
> Le nom d’hôte Linux ou le nom de l’ordinateur Windows ne peuvent pas contenir de mots réservés ni de marques dans le nom. Dans le cas contraire, la tentative d’inscription de la machine connectée auprès d’Azure se solde par un échec. Consultez [Résoudre les erreurs de nom de ressource réservé](../../azure-resource-manager/templates/error-reserved-resource-name.md) pour obtenir la liste des mots réservés.

### <a name="required-permissions"></a>Autorisations requises

* Pour intégrer des ordinateurs, vous devez être membre du rôle **Intégration de machine connectée à Azure**.

* Pour lire, modifier et supprimer un ordinateur, vous devez être membre du rôle **Administrateur des ressources Azure Connected Machine**. 

### <a name="azure-subscription-and-service-limits"></a>Limites du service et de l’abonnement Azure

Avant de configurer vos machines à l’aide de serveurs activés par Azure Arc, examinez les [limites de l’abonnement](../../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits) et les [limites des groupes de ressources](../../azure-resource-manager/management/azure-subscription-service-limits.md#resource-group-limits) d’Azure Resource Manager pour planifier le nombre de machines à connecter.

### <a name="transport-layer-security-12-protocol"></a>Protocole Transport Layer Security 1.2

Pour garantir la sécurité des données en transit vers Azure, nous vous encourageons vivement à configurer la machine de manière à utiliser le protocole TLS (Transport Layer Security) 1.2. Les versions antérieures de TLS/SSL (Secure Sockets Layer) se sont avérées vulnérables et bien qu’elles fonctionnent encore pour assurer la compatibilité descendante, elles sont **déconseillées**.

|Plateforme/Langage | Support | Informations complémentaires |
| --- | --- | --- |
|Linux | Les distributions de Linux s’appuient généralement sur [OpenSSL](https://www.openssl.org) pour la prise en charge de TLS 1.2. | Vérifiez [OpenSSL Changelog](https://www.openssl.org/news/changelog.html) pour vous assurer que votre version d’OpenSSL est prise en charge.|
| Windows Server 2012 R2 et versions ultérieures | Pris en charge, activé par défaut. | Pour confirmer que vous utilisez toujours les [paramètres par défaut](/windows-server/security/tls/tls-registry-settings).|

### <a name="networking-configuration"></a>Configuration de la mise en réseau

L’agent Connected Machine pour Linux et Windows communique en sortie de manière sécurisée vers Azure Arc sur le port TCP 443. Si la machine se connecte via un pare-feu ou un serveur proxy pour communiquer sur Internet, consultez les éléments ci-dessous pour comprendre la configuration réseau requise.

Si la connectivité sortante est restreinte par votre pare-feu ou votre serveur proxy, vérifiez que les URL listées ci-dessous ne sont pas bloquées. Lorsque vous n’autorisez que les plages d’adresses IP ou les noms de domaine nécessaires pour que l’agent communique avec le service, vous devez également autoriser l’accès aux URL et étiquettes de service suivantes.

Balises de service :

* AzureActiveDirectory
* AzureTrafficManager
* AzureResourceManager
* AzureArcInfrastructure

URL :

| Ressource de l’agent | Description |
|---------|---------|
|`management.azure.com`|Azure Resource Manager|
|`login.windows.net`|Azure Active Directory|
|`dc.services.visualstudio.com`|Application Insights|
|`*.guestconfiguration.azure.com` |Guest Configuration|
|`*.his.arc.azure.com`|Service d’identité hybride|

Les agents de préversion (version 0.11 et antérieure) nécessitent également l’accès aux URL suivantes :

| Ressource de l’agent | Description |
|---------|---------|
|`agentserviceapi.azure-automation.net`|Guest Configuration|
|`*-agentservice-prod-1.azure-automation.net`|Guest Configuration|

Pour obtenir la liste d’adresses IP de chaque balise de service/région, consultez le fichier JSON - [Plages d’adresses IP Azure et balises de service – Cloud Public](https://www.microsoft.com/download/details.aspx?id=56519). Microsoft publie chaque semaine une mise à jour contenant chacun des services Azure et les plages d’adresses IP qu’il utilise. Pour plus d’informations, consultez [Étiquettes de service](../../virtual-network/network-security-groups-overview.md#service-tags).

Les URL indiquées dans le tableau précédent sont nécessaires en plus des informations relatives à la plage d’adresses IP des balises de service, car la plupart des services n’ont pas actuellement d’inscription de balise de service. Ainsi, les adresses IP sont sujettes à modification. Si des plages d’adresses IP sont requises par la configuration de votre pare-feu, utilisez l’étiquette de service **AzureCloud** pour autoriser l’accès à tous les services Azure. Ne désactivez pas la supervision ou l’inspection de la sécurité de ces URL ; autorisez-les comme vous le feriez pour tout autre trafic Internet.

### <a name="register-azure-resource-providers"></a>Inscrire des fournisseurs de ressources Azure

Pour utiliser ce service, les serveurs activés par Azure Arc dépendent des fournisseurs de ressources Azure suivants dans votre abonnement :

* **Microsoft.HybridCompute**
* **Microsoft.GuestConfiguration**

S’ils ne sont pas inscrits, vous pouvez les inscrire à l’aide des commandes suivantes :

Azure PowerShell :

```azurepowershell-interactive
Login-AzAccount
Set-AzContext -SubscriptionId [subscription you want to onboard]
Register-AzResourceProvider -ProviderNamespace Microsoft.HybridCompute
Register-AzResourceProvider -ProviderNamespace Microsoft.GuestConfiguration
```

Azure CLI :

```azurecli-interactive
az account set --subscription "{Your Subscription Name}"
az provider register --namespace 'Microsoft.HybridCompute'
az provider register --namespace 'Microsoft.GuestConfiguration'
```

Vous pouvez également inscrire les fournisseurs de ressources dans le portail Azure en suivant les étapes décrites sous [Portail Azure](../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal).

## <a name="installation-and-configuration"></a>Installation et configuration

Selon vos besoins, plusieurs méthodes vous permettent de connecter des machines de votre environnement hybride directement à Azure. Le tableau suivant décrit chacune d’entre elle, pour vous permettre d’identifier la plus adaptée à votre organisation.

> [!IMPORTANT]
> L’agent Connected Agent ne peut pas être installé sur une machine virtuelle Windows Azure. Si vous tentez d’effectuer cette opération, l’installation la détecte et la restaure.

| Méthode | Description |
|--------|-------------|
| de manière interactive, | Installer manuellement l’agent sur une seule machine ou un petit nombre de machines en suivant les étapes décrites dans [Connecter des machines à partir du portail Azure](onboard-portal.md).<br> À partir du portail Azure, vous pouvez générer un script et l’exécuter sur la machine pour automatiser les étapes d’installation et de configuration de l’agent.|
| À grande échelle | Installer et configurer l’agent pour plusieurs machines en suivant les instructions indiquées dans [Connecter des machines à l’aide d’un principal de service](onboard-service-principal.md).<br> Cette méthode crée un principal de service pour connecter des machines de manière non interactive.|
| À grande échelle | Installer et configurer l’agent pour plusieurs machines en suivant les instructions de la méthode [Utilisation de Windows PowerShell DSC](onboard-dsc.md).<br> Cette méthode utilise un principal de service pour connecter des machines de manière non interactive à PowerShell DSC. |

## <a name="connected-machine-agent-technical-overview"></a>Présentation technique de l’agent Machine connectée

### <a name="windows-agent-installation-details"></a>Détails de l’installation de l’agent Windows

Vous pouvez installer l’agent Connected Machine pour Windows à l’aide de l’une des trois méthodes suivantes :

* En double-cliquant sur le fichier `AzureConnectedMachineAgent.msi`.
* Manuellement en exécutant le package Windows Installer `AzureConnectedMachineAgent.msi` à partir de l’interpréteur de commandes.
* À partir d’une session PowerShell à l’aide d’une méthode de script.

Une fois l’agent Connected Machine pour Windows installé, les modifications de configuration système suivantes sont appliquées.

* Les dossiers d’installation suivants sont créés lors de la configuration.

    |Dossier |Description |
    |-------|------------|
    |%ProgramFiles%\AzureConnectedMachineAgent |Chemin d’installation par défaut contenant les fichiers de support de l’agent.|
    |%ProgramData%\AzureConnectedMachineAgent |Contient les fichiers de configuration de l’agent.|
    |%ProgramData%\AzureConnectedMachineAgent\Tokens |Contient les jetons acquis.|
    |%ProgramData%\AzureConnectedMachineAgent\Config |Contient le fichier de configuration de l’agent `agentconfig.json` qui enregistre ses informations d’inscription auprès du service.|
    |%ProgramFiles%\ArcConnectedMachineAgent\ExtensionService\GC | Chemin d’installation contenant les fichiers de l’agent Guest Configuration. |
    |%ProgramData%\GuestConfig |Contient les stratégies (appliquées) à partir d’Azure.|
    |%ProgramFiles%\AzureConnectedMachineAgent\ExtensionService\downloads | Les extensions sont téléchargées à partir d’Azure et copiées ici.|

* Les services Windows suivants sont créés sur la machine cible lors de l’installation de l’agent.

    |Nom du service |Nom complet |Nom du processus |Description |
    |-------------|-------------|-------------|------------|
    |himds |Azure Hybrid Instance Metadata Service |himds |Ce service implémente le service IMDS (Instance Metadata service) Azure pour gérer la connexion à Azure et l’identité Azure de l’ordinateur connecté.|
    |GCArcService |Service Arc de configuration d’invité |gc_service |Analyse la configuration de l’état souhaité de la machine.|
    |ExtensionService |Service d’extension de la configuration d’invité | gc_service |Installe les extensions requises ciblant la machine.|

* Les variables d’environnement suivantes sont créées lors de l’installation de l’agent.

    |Nom |Valeur par défaut |Description |
    |-----|--------------|------------|
    |IDENTITY_ENDPOINT |http://localhost:40342/metadata/identity/oauth2/token ||
    |IMDS_ENDPOINT |http://localhost:40342 ||

* Plusieurs fichiers journaux sont disponibles pour la résolution des problèmes. Ils sont décrits dans le tableau suivant.

    |Journal |Description |
    |----|------------|
    |%ProgramData%\AzureConnectedMachineAgent\Log\himds.log |Enregistre les détails du service des agents (HIMDS) et l’interaction avec Azure.|
    |%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log |Contient la sortie des commandes de l’outil azcmagent, lorsque l’argument détaillé (-v) est utilisé.|
    |%ProgramData%\GuestConfig\gc_agent_logs\gc_agent.log |Enregistre les détails de l’activité du service DSC,<br> en particulier, la connectivité entre le service HIMDS et Azure Policy.|
    |%ProgramData%\GuestConfig\gc_agent_logs\gc_agent_telemetry.txt |Enregistre les détails relatifs à la télémétrie du service DSC et à la journalisation détaillée.|
    |%ProgramData%\GuestConfig\ext_mgr_logs|Enregistre les détails concernant le composant de l’agent Extension.|
    |%ProgramData%\GuestConfig\extension_logs\<Extension>|Enregistre les détails de l’extension installée.|

* Le groupe de sécurité local **Applications d’extension de l’agent hybride** est créé.

* Lors de la désinstallation de l’agent, les artefacts suivants ne sont pas supprimés.

    * %ProgramData%\AzureConnectedMachineAgent\Log
    * %ProgramData%\AzureConnectedMachineAgent et sous-répertoires
    * %ProgramData%\GuestConfig

### <a name="linux-agent-installation-details"></a>Détails de l’installation de l’agent Linux

L’agent Connected Machine pour Linux est fourni dans le format de package par défaut pour la distribution (. RPM ou. DEB) qui est hébergée dans le [dépôt de packages](https://packages.microsoft.com/) de Microsoft. L’agent est installé et configuré avec le groupe de scripts d’interpréteur de commandes [Install_linux_azcmagent.sh](https://aka.ms/azcmagent).

Une fois l’agent Connected Machine pour Linux installé, les modifications de configuration système suivantes sont appliquées.

* Les dossiers d’installation suivants sont créés lors de la configuration.

    |Dossier |Description |
    |-------|------------|
    |/var/opt/azcmagent/ |Chemin d’installation par défaut contenant les fichiers de support de l’agent.|
    |/opt/azcmagent/ |
    |/opt/GC_Ext | Chemin d’installation contenant les fichiers de l’agent Guest Configuration.|
    |/opt/DSC/ |
    |/var/opt/azcmagent/tokens |Contient les jetons acquis.|
    |/var/lib/GuestConfig |Contient les stratégies (appliquées) à partir d’Azure.|
    |/opt/GC_Ext/downloads|Les extensions sont téléchargées à partir d’Azure et copiées ici.|

* Les démons suivants sont créés sur la machine cible lors de l’installation de l’agent.

    |Nom du service |Nom complet |Nom du processus |Description |
    |-------------|-------------|-------------|------------|
    |himdsd.service |Service Azure Connected Machine Agent |himds |Ce service implémente le service IMDS (Instance Metadata service) Azure pour gérer la connexion à Azure et l’identité Azure de l’ordinateur connecté.|
    |gcad.servce |Service Arc GC |gc_linux_service |Analyse la configuration de l’état souhaité de la machine. |
    |extd.service |Service d’extension |gc_linux_service | Installe les extensions requises ciblant la machine.|

* Plusieurs fichiers journaux sont disponibles pour la résolution des problèmes. Ils sont décrits dans le tableau suivant.

    |Journal |Description |
    |----|------------|
    |/var/opt/azcmagent/log/himds.log |Enregistre les détails du service des agents (HIMDS) et l’interaction avec Azure.|
    |/var/opt/azcmagent/log/azcmagent.log |Contient la sortie des commandes de l’outil azcmagent, lorsque l’argument détaillé (-v) est utilisé.|
    |/opt/logs/dsc.log |Enregistre les détails de l’activité du service DSC,<br> en particulier, la connectivité entre le service himds et Azure Policy.|
    |/opt/logs/dsc.telemetry.txt |Enregistre les détails relatifs à la télémétrie du service DSC et à la journalisation détaillée.|
    |/var/lib/GuestConfig/ext_mgr_logs |Enregistre les détails concernant le composant de l’agent Extension.|
    |/var/lib/GuestConfig/extension_logs|Enregistre les détails de l’extension installée.|

* Les variables d’environnement suivantes sont créées lors de l’installation de l’agent. Les variables suivantes sont définies dans `/lib/systemd/system.conf.d/azcmagent.conf`.

    |Nom |Valeur par défaut |Description |
    |-----|--------------|------------|
    |IDENTITY_ENDPOINT |http://localhost:40342/metadata/identity/oauth2/token ||
    |IMDS_ENDPOINT |http://localhost:40342 ||

* Lors de la désinstallation de l’agent, les artefacts suivants ne sont pas supprimés.

    * /var/opt/azcmagent
    * /opt/logs

## <a name="next-steps"></a>Étapes suivantes

* Pour commencer l’évaluation des serveurs activés par Azure Arc, suivez les instructions de l’article [Connecter des machines hybrides à Azure à partir du portail Azure](onboard-portal.md).

* Pour plus d’informations sur la résolution des problèmes, consultez le [guide Résoudre les problèmes de l’agent Connected Machine](troubleshoot-agent-onboard.md).
