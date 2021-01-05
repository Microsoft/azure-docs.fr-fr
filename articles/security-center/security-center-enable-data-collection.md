---
title: Déployer automatiquement des agents pour Azure Security Center | Microsoft Docs
description: Cet article explique comment configurer le provisionnement automatique de l’agent Log Analytics et d’autres agents utilisés par Azure Security Center.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: quickstart
ms.date: 11/15/2020
ms.author: memildin
ms.openlocfilehash: f5cf432e3824ca0bb441a458a08fc7353291cf1f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490055"
---
# <a name="auto-provisioning-agents-and-extensions-from-azure-security-center"></a>Provisionnement automatique d’agents et d’extensions à partir d’Azure Security Center

Security Center collecte des données à partir de vos machines virtuelles Azure, groupes de machines virtuelles identiques, conteneurs IaaS et ordinateurs non-Azure (y compris locaux) pour surveiller les menaces et vulnérabilités de sécurité. 

La collecte de données est requise pour fournir une visibilité des mises à jour manquantes, des paramètres de sécurité du système d’exploitation mal configurés, de l’état de protection du point de terminaison ainsi que de l’intégrité et de la protection contre les menaces. La collecte de données est nécessaire uniquement pour les ressources de calcul (machines virtuelles, groupes de machines virtuelles identiques, conteneurs IaaS et ordinateurs autres qu’Azure). Vous pouvez bénéficier d’Azure Security Center même si vous ne configurez pas d’agents ; toutefois, vous disposerez d’une sécurité limitée et les fonctionnalités répertoriées ci-dessus ne sont pas pris en charge.  

Les données sont collectées à l’aide des éléments suivants :

- L’**agent Log Analytics** lit divers journaux d’événements et configurations liées à la sécurité de la machine et copie ces données dans votre espace de travail à des fins d’analyse. Il peut s’agir des données suivantes : type et version de système d’exploitation, journaux d’activité de système d’exploitation (journaux d’événements Windows), processus en cours d’exécution, nom de machine, adresses IP et utilisateur connecté.
- Les **extensions de sécurité**, telles que le [module complémentaire Azure Policy pour Kubernetes](../governance/policy/concepts/policy-for-kubernetes.md), peuvent également fournir des données à Security Center relatives à des types de ressources spécialisés.

> [!TIP]
> Au fil de la croissance de Security Center, les types de ressources pouvant être surveillés ont également augmenté. Le nombre d’extensions a également augmenté. Le provisionnement automatique s’est développé pour prendre en charge des types de ressources supplémentaires en tirant parti des fonctionnalités Azure Policy.

:::image type="content" source="./media/security-center-enable-data-collection/auto-provisioning-options.png" alt-text="Page des paramètres de provisionnement automatique de Security Center":::


## <a name="why-use-auto-provisioning"></a>Pourquoi utiliser le provisionnement automatique ?
Les agents et les extensions décrits dans cette page *peuvent* être installés manuellement (voir [Installation manuelle de l’agent Log Analytics](#manual-agent)). Toutefois, le **provisionnement automatique** réduit la charge de gestion en installant tous les agents et extensions requis sur les machines existantes et nouvelles pour garantir plus rapidement la couverture de sécurité de toutes les ressources prises en charge. 

Nous vous recommandons d’activer le provisionnement automatique, mais il est désactivé par défaut.

## <a name="how-does-auto-provisioning-work"></a>Comment fonctionne le provisionnement automatique ?
Les paramètres de provisionnement automatique de Security Center ont un bouton bascule pour chaque type d’extension pris en charge. Lorsque vous activez le provisionnement automatique d’une extension, vous affectez la **stratégie de déploiement si inexistant** appropriée pour vous assurer que l’extension est provisionnée sur toutes les ressources existantes et futures de ce type.

> [!TIP]
> Apprenez-en davantage sur les effets d’Azure Policy, notamment le déploiement si inexistant, dans [Comprendre les effets d’Azure Policy](../governance/policy/concepts/effects.md).

## <a name="enable-auto-provisioning-of-the-log-analytics-agent"></a>Activer le provisionnement automatique de l’agent Log Analytics<a name="auto-provision-mma"></a>
Lorsque le provisionnement automatique est activé pour l’agent Log Analytics, Security Center déploie l’agent sur toutes les machines virtuelles Azure prises en charge et toutes celles nouvellement créées. Pour obtenir la liste des plateformes prises en charge, consultez [Plateformes prises en charge dans Azure Security Center](security-center-os-coverage.md).

Pour activer le provisionnement automatique de l’agent Log Analytics :

1. Dans le menu de Security Center, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’abonnement approprié.
1. Dans la page **Provisionnement automatique**, définissez l’état de l’agent sur **Activé**.
1. Dans le volet des options de configuration, définissez l’espace de travail à utiliser.

    :::image type="content" source="./media/security-center-enable-data-collection/log-analytics-agent-deploy-options.png" alt-text="Options de configuration du provisionnement automatique des agents Log Analytics sur les machines virtuelles" lightbox="./media/security-center-enable-data-collection/log-analytics-agent-deploy-options.png":::

    - **Connecter les machines virtuelles Azure aux espaces de travail par défaut créés par Security Center** – Security Center crée un groupe de ressources et un espace de travail par défaut dans la même zone géographique, et connecte l’agent à cet espace de travail. Si un abonnement contient des machines virtuelles de plusieurs zones géographiques, Security Center crée plusieurs espaces de travail pour garantir la conformité aux exigences de confidentialité des données.

        La convention d’affectation de noms pour l’espace de travail et le groupe de ressources est la suivante : 
        - Espace de travail : DefaultWorkspace-[ID d’abonnement]-[zone géographique] 
        - Groupe de ressources : DefaultResourceGroup-[geo] 

        Security Center active automatiquement une solution Security Center sur l’espace de travail, en fonction du niveau tarifaire défini pour l’abonnement. 

        > [!TIP]
        > Pour toute question concernant les espaces de travail par défaut, consultez :
        >
        > - [Est-ce que je suis facturé pour Journaux Azure Monitor sur des espaces de travail créés par Security Center ?](faq-data-collection-agents.md#am-i-billed-for-azure-monitor-logs-on-the-workspaces-created-by-security-center)
        > - [Où est créé l’espace de travail Log Analytics par défaut ?](faq-data-collection-agents.md#where-is-the-default-log-analytics-workspace-created)
        > - [Puis-je supprimer les espaces de travail par défaut créés par Security Center ?](faq-data-collection-agents.md#can-i-delete-the-default-workspaces-created-by-security-center)

    - **Connecter les machines virtuelles Azure à un autre espace de travail** – Dans la liste déroulante, sélectionnez l’espace de travail où stocker les données collectées. La liste déroulante comprend tous les espaces de travail de tous vos abonnements. Vous pouvez utiliser cette option pour collecter les données de machines virtuelles qui s’exécutent dans différents abonnements et les stocker dans l’espace de travail que vous avez sélectionné.  

        Si vous disposez déjà d’un espace de travail Log Analytics existant, vous souhaiterez peut-être utiliser le même espace de travail (des autorisations en lecture et écriture sur l’espace de travail sont requises). Cette option est utile si vous utilisez un espace de travail centralisé dans votre organisation et que vous souhaitez l’utiliser pour la collecte de données de sécurité. Apprenez-en davantage dans [Gérer l’accès aux données de journal et aux espaces de travail dans Azure Monitor](../azure-monitor/platform/manage-access.md).

        Si l’espace de travail sélectionné a déjà une solution Security ou SecurityCenterFree activée, la tarification sera définie automatiquement. Dans le cas contraire, installez une solution Security Center sur l’espace de travail :

        1. Dans le menu de Security Center, ouvrez **Tarification et paramètres**.
        1. Sélectionnez l’espace de travail auquel vous connecterez les agents.
        1. Sélectionnez **Azure Defender activé** ou **Azure Defender désactivé**.

1. Dans la configuration **Événements de sécurité Windows**, sélectionnez la quantité de données d’événement brutes à stocker :
    - **Aucun** : désactiver le stockage d’événements de sécurité. Il s'agit du paramètre par défaut.
    - **Minimal** : petit ensemble d’événements pour quand vous voulez réduire au maximum le volume d’événements.
    - **Commun** : ensemble d’événements qui satisfait la plupart des clients et fournit une piste d’audit complète.
    - **Tous les événements** : pour les clients qui souhaitent s’assurer que tous les événements sont stockés.

    > [!TIP]
    > Pour définir ces options au niveau de l’espace de travail, consultez [Définition de l’option d’événement de sécurité au niveau de l’espace de travail](#setting-the-security-event-option-at-the-workspace-level).
    > 
    > Pour plus d’informations sur ces options, consultez [Options des événements de sécurité Windows pour l’agent Log Analytics](#data-collection-tier).

1. Sélectionnez **Appliquer** dans le volet de configuration.

1. Sélectionnez **Enregistrer**. Si un espace de travail doit être approvisionné, l’installation de l’agent peut prendre jusqu’à 25 minutes.

1. Il vous sera demandé si vous voulez reconfigurer les machines virtuelles surveillées précédemment connectées à un espace de travail par défaut :

    :::image type="content" source="./media/security-center-enable-data-collection/reconfigure-monitored-vm.png" alt-text="Passer en revue les options pour reconfigurer les machines virtuelles surveillées":::

    - **Non** – Vos nouveaux paramètres d’espace de travail s’appliqueront uniquement aux machines virtuelles nouvellement détectées sur lesquelles l’agent Log Analytics n’est pas installé.
    - **Oui** – Vos nouveaux paramètres d’espace de travail s’appliqueront à toutes les machines virtuelles et chaque machine virtuelle actuellement connectée à un espace de travail créé par Security Center sera reconnectée au nouvel espace de travail cible.

   > [!NOTE]
   > Si vous sélectionnez **Oui**, ne supprimez pas le ou les espaces de travail créés par Security Center tant que toutes les machines virtuelles n’ont pas été reconnectées au nouvel espace de travail cible. Cette opération échoue si un espace de travail est supprimé trop tôt.


## <a name="enable-auto-provisioning-of-extensions"></a>Activer le provisionnement automatique des extensions

Pour activer le provisionnement automatique d’une extension autre que l’agent Log Analytics : 

1. Dans le menu de Security Center dans le portail Azure, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’abonnement approprié.
1. Sélectionnez **Provisionnement automatique**.
1. Si vous activez le provisionnement automatique pour Microsoft Dependency Agent, assurez-vous que l’agent Log Analytics est également configuré pour un déploiement automatique. 
1. Basculez l’état sur **Activé** pour l’extension appropriée.

    :::image type="content" source="./media/security-center-enable-data-collection/toggle-kubernetes-add-on.png" alt-text="Activer le provisionnement automatique pour le module complémentaire de stratégie K8s":::

1. Sélectionnez **Enregistrer**. La stratégie Azure est affectée et une tâche de correction est créée.

    |Extension  |Policy  |
    |---------|---------|
    |Module complémentaire de stratégie pour Kubernetes|[Déployer l’extension Azure Policy sur les clusters Azure Kubernetes Service](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fa8eff44f-8c92-45c3-a3fb-9880802d67a7)|
    |Microsoft Dependency Agent (préversion) (machines virtuelles Windows)|[Déployer Dependency Agent pour les machines virtuelles Windows](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f1c210e94-a481-4beb-95fa-1571b434fb04)         |
    |Microsoft Dependency Agent (préversion) (machines virtuelles Linux)|[Déployer Dependency Agent pour les machines virtuelles Linux](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f4da21710-ce6f-4e06-8cdb-5cc4c93ffbee)|



## <a name="windows-security-event-options-for-the-log-analytics-agent"></a>Options des événements de sécurité Windows pour l’agent Log Analytics <a name="data-collection-tier"></a> 

La sélection d’un niveau de collecte de données dans Azure Security Center n’affecte que le *stockage* d’événements de sécurité dans votre espace de travail Log Analytics. L’agent Log Analytics continuera de collecter et d’analyser les événements de sécurité requis pour la protection de Security Center contre les menaces, quel que soit le niveau des événements de sécurité que vous choisissez de stocker dans votre espace de travail. Le choix de stocker des événements de sécurité permet l’investigation, la recherche et l’audit de ces événements dans votre espace de travail.

### <a name="requirements"></a>Spécifications 
Azure Defender est requis pour le stockage des données d’événements de sécurité Windows. [En savoir plus sur Azure Defender](azure-defender.md).

Le stockage de données dans Log Analytics peut occasionner des frais supplémentaires de stockage de données. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/security-center/).

### <a name="information-for-azure-sentinel-users"></a>Informations pour les utilisateurs Azure Sentinel 
Utilisateurs d’Azure Sentinel : notez que la collecte des événements de sécurité dans le contexte d’un seul espace de travail peut être configurée à partir d’Azure Security Center ou d’Azure Sentinel, mais pas des deux à la fois. Si vous envisagez d’ajouter Azure Sentinel à un espace de travail qui reçoit déjà des alertes d’Azure Security Center, et qui est défini pour collecter des événements de sécurité, vous disposez de deux options :
- Laissez la collecte d’événements de sécurité dans Azure Security Center telle quelle. Vous êtes alors en mesure d’interroger et d’analyser ces événements dans Azure Sentinel ainsi que dans Azure Defender. Toutefois, vous n’êtes pas en mesure de surveiller l’état de connectivité du connecteur ou de modifier sa configuration dans Azure Sentinel. Si cela est important pour vous, envisagez la deuxième option.
- Désactivez la collecte des événements de sécurité dans Azure Security Center (en définissant **Événements de sécurité Windows** sur **Aucun** dans la configuration de votre agent Log Analytics). Ensuite, ajoutez le connecteur Événements de sécurité dans Azure Sentinel. Comme pour la première option, vous êtes en mesure d’interroger et d’analyser les événements à la fois dans Azure Sentinel et Azure Defender/ASC. Cependant, vous pouvez maintenant surveiller l’état de connectivité du connecteur ou modifier sa configuration dans, et uniquement dans, Azure Sentinel.

### <a name="what-event-types-are-stored-for-common-and-minimal"></a>Quels types d’événements sont stockés pour « Commun » et « Minimal » ?
Ces ensembles ont été conçus pour des scénarios classiques. Veillez à évaluer celui qui correspond à vos besoins avant de l’implémenter.

Pour déterminer les événements pour les options **Commun** et **Minimal**, nous avons travaillé avec les clients et les normes industrielles pour en savoir plus sur la fréquence non filtrée de chaque événement et leur utilisation. Nous avons utilisé les instructions suivantes dans ce processus :

- **Minimal** : vérifiez que cet ensemble couvre uniquement les événements qui peuvent indiquer une violation avérée et des événements importants qui ont un volume très faible. Par exemple, cet ensemble contient une connexion utilisateur ayant réussi et ayant échoué (ID d’événement 4624, 4625), mais il ne contient pas de déconnexion, ce qui est important pour l’audit, mais pas pour la détection, et a un volume relativement élevé. La plupart du volume de données de cet ensemble est constituée d’événements de connexion et d’un événement de création de processus (ID d’événement 4688).
- **Commun** : fournissez une piste d’audit utilisateur complète dans cet ensemble. Par exemple, cet ensemble contient les connexions et déconnexions de l’utilisateur (ID d’événement 4634). Nous incluons les actions d’audit, telles que les modifications de groupe de sécurité, les opérations Kerberos du contrôleur de domaine clé et les autres événements recommandés par les organisations du secteur.

Les événements qui ont un volume très faible ont été inclus dans l’ensemble Commun, car la motivation principale à préférer cet ensemble à l’ensemble Tous les événements est de réduire le volume et de ne pas filtrer d’événements spécifiques.

Voici le détail complet des ID d’événement App Locker et de sécurité pour chaque ensemble :

| Couche Données | Indicateurs d’événements collectés |
| --- | --- |
| Minimales | 1102,4624,4625,4657,4663,4688,4700,4702,4719,4720,4722,4723,4724,4727,4728,4732,4735,4737,4739,4740,4754,4755, |
| | 4756,4767,4799,4825,4946,4948,4956,5024,5033,8001,8002,8003,8004,8005,8006,8007,8222 |
| Courant | 1,299,300,324,340,403,404,410,411,412,413,431,500,501,1100,1102,1107,1108,4608,4610,4611,4614,4622, |
| |  4624,4625,4634,4647,4648,4649,4657,4661,4662,4663,4665,4666,4667,4688,4670,4672,4673,4674,4675,4689,4697, |
| | 4700,4702,4704,4705,4716,4717,4718,4719,4720,4722,4723,4724,4725,4726,4727,4728,4729,4733,4732,4735,4737, |
| | 4738,4739,4740,4742,4744,4745,4746,4750,4751,4752,4754,4755,4756,4757,4760,4761,4762,4764,4767,4768,4771, |
| | 4774,4778,4779,4781,4793,4797,4798,4799,4800,4801,4802,4803,4825,4826,4870,4886,4887,4888,4893,4898,4902, |
| | 4904,4905,4907,4931,4932,4933,4946,4948,4956,4985,5024,5033,5059,5136,5137,5140,5145,5632,6144,6145,6272, |
| | 6273,6278,6416,6423,6424,8001,8002,8003,8004,8005,8006,8007,8222,26401,30004 |

> [!NOTE]
> - Si vous utilisez l’objet de stratégie de groupe (GPO), il est recommandé d’activer les stratégies d’audit d’événement de création de processus 4688 et le champ *CommandLine* à l’intérieur de l’événement 4688. Pour plus d’informations sur l’événement de création de processus 4688, consultez la [FAQ](faq-data-collection-agents.md#what-happens-when-data-collection-is-enabled) de Security Center. Pour plus d’informations sur ces stratégies d’audit, consultez les [recommandations pour la stratégie d’audit](/windows-server/identity/ad-ds/plan/security-best-practices/audit-policy-recommendations).
> -  Pour activer la collecte de données pour les [Contrôles d’application adaptatifs](security-center-adaptive-application.md), Security Center configure une stratégie AppLocker locale en mode Audit pour autoriser toutes les applications. Cela amène AppLocker à générer des événements qui sont ensuite recueillis et exploités par Security Center. Il est important de noter que cette stratégie ne sera configurée sur aucun ordinateur sur lequel une stratégie AppLocker est déjà configurée. 
> - Pour collecter la plateforme de filtrage Windows [ID d’événement 5156](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=5156), vous devez activer [Connexion de la plateforme de filtrage d’audits](/windows/security/threat-protection/auditing/audit-filtering-platform-connection) (Auditpol /set /subcategory:"Filtering Platform Connection" /Success:Enable)
>

### <a name="setting-the-security-event-option-at-the-workspace-level"></a>Définition de l’option d’événement de sécurité au niveau de l’espace de travail

Vous pouvez définir le niveau des données d’événement de sécurité à stocker au niveau de l’espace de travail.

1. Dans le menu de Security Center dans le portail Azure, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’espace de travail approprié. Les seuls événements de collecte de données pour un espace de travail sont les événements de sécurité Windows décrits dans cette page.

    :::image type="content" source="media/security-center-enable-data-collection/event-collection-workspace.png" alt-text="Définition des données d’événement de sécurité à stocker dans un espace de travail":::

1. Sélectionnez la quantité de données d’événement brutes à stocker, puis sélectionnez **Enregistrer**.

## <a name="manual-agent-provisioning"></a>Approvisionnement manuel d’un agent <a name="manual-agent"></a>
 
Pour installer manuellement l’agent Log Analytics :

1. Désactivez le provisionnement automatique.

1. Si vous le souhaitez, vous pouvez créer un espace de travail.

1. Activez Azure Defender sur l’espace de travail sur lequel vous installez l’agent Log Analytics :

    1. Dans le menu de Security Center, sélectionnez **Tarification et paramètres**.

    1. Définissez l’espace de travail sur lequel vous installez l’agent. Assurez-vous que l’espace de travail est dans le même abonnement que vous utilisez dans Security Center et que vous disposez d’autorisations en lecture/écriture pour l’espace de travail.

    1. Sélectionnez **Azure Defender activé**, puis **Enregistrer**.

       >[!NOTE]
       >Si l’espace de travail a déjà une solution **Security** ou **SecurityCenterFree** activée, la tarification sera définie automatiquement. 

1. Pour déployer les agents sur de nouvelles machines virtuelles en utilisant un modèle Resource Manager, installez l’agent Log Analytics :

   - [Installer l’agent Log Analytics pour Windows](../virtual-machines/extensions/oms-windows.md)
   - [Installer l’agent Log Analytics pour Linux](../virtual-machines/extensions/oms-linux.md)

1. Pour déployer des agents sur vos machines virtuelles existantes, suivez les instructions fournies dans [Collecter des données sur les machines virtuelles Azure](../azure-monitor/learn/quick-collect-azurevm.md) (la section **Collecter les données d’événements et de performances** est facultative).

1. Pour utiliser PowerShell afin de déployer les agents, suivez les instructions de la documentation relative aux machines virtuelles :

    - [Pour les machines Windows](../virtual-machines/extensions/oms-windows.md?toc=%2fazure%2fazure-monitor%2ftoc.json#powershell-deployment)
    - [Pour les machines Linux](../virtual-machines/extensions/oms-linux.md?toc=%2fazure%2fazure-monitor%2ftoc.json#azure-cli-deployment)

> [!TIP]
> Pour obtenir des instructions sur la manière d’intégrer Security Center à l’aide de PowerShell, voir [Automatiser l’intégration d’Azure Security Center à l’aide de PowerShell](security-center-powershell-onboarding.md).


## <a name="automatic-provisioning-in-cases-of-a-pre-existing-agent-installation"></a>Approvisionnement automatique en cas d’installation d’un agent préexistant <a name="preexisting"></a> 

Les cas d’usage suivants spécifient la manière dont l’approvisionnement automatique fonctionne lorsqu’un agent ou une extension sont déjà installés. 

- **Agent Log Analytics est installé sur la machine, mais pas comme extension (agent direct)** – Si l’agent Log Analytics est installé directement sur la machine virtuelle (non pas comme extension Azure), Security Center installe l’extension de l’agent Log Analytics et il est possible que l’agent Log Analytics soit mis à niveau vers sa version la plus récente.
L’agent installé continue de rendre compte à ses espaces de travail déjà configurés et à l’espace de travail configuré dans Security Center (multihébergement pris en charge sur les machines Windows).
Si l’espace de travail configuré est un espace de travail utilisateur (pas un espace de travail par défaut de Security Center), vous devez installer la solution « security/securityFree » sur ce dernier pour que Security Center démarre le traitement des événements à partir des machines virtuelles et ordinateurs rendant compte à cet espace de travail.

    Pour les machines Linux, le multihébergement d’agent n’est pas encore pris en charge, par conséquent, si une installation existante de l’agent est détectée, l’approvisionnement automatique n’a pas lieu et la configuration de la machine n’est pas modifiée.

    Pour les machines existantes sur les abonnements intégrés à Security Center avant le 17 mars 2019, lorsqu’un agent existant est détecté, l’extension de l’agent Log Analytics n’est pas installée et la machine n’est pas affectée. Pour ces machines, consultez la recommandation « Résoudre les problèmes d’intégrité de l’agent d’analyse sur vos machines » pour résoudre les problèmes d’installation de l’agent sur ces machines.
  
- **L’agent System Center Operations Manager est installé sur la machine** – Security Center installe l’extension de l’agent Log Analytics parallèlement au gestionnaire Operations Manager existant. L’agent Operations Manager existant continue de rendre compte normalement au serveur Operations Manager. L’agent Operations Manager et l’agent Log Analytics partagent des bibliothèques runtime communes, qui sont mises à jour vers la dernière version lors de ce processus. Si la version 2012 de l’agent Operations Manager est installée, n’activez **pas** l’approvisionnement automatique.

- **Une extension de machine virtuelle préexistante est présente** :
    - Lorsque l’Agent Monitoring est installé en tant qu’extension, la configuration de l’extension permet de rendre compte à un seul espace de travail. Security Center n’écrase pas les connexions existantes des espaces de travail utilisateur. Security Center stocke les données de sécurité à partir d’une machine virtuelle dans un espace de travail qui est déjà connecté, sous réserve que la solution « security » ou « securityFree » y soit installée. Security Center peut mettre à niveau la version de l'extension vers la dernière version lors de ce processus.  
    - Pour voir à quel espace de travail l’extension existante envoie des données, exécutez le test pour [Valider la connectivité avec Azure Security Center](/archive/blogs/yuridiogenes/validating-connectivity-with-azure-security-center). Vous pouvez également ouvrir des espaces de travail Log Analytics, sélectionner un espace de travail, sélectionner la machine virtuelle, puis rechercher la connexion de l'agent Log Analytics. 
    - Si vous disposez d’un environnement où l'agent Log Analytics est installé sur les stations de travail clientes et rapportent à un espace de travail Log Analytics existant, consultez la liste des [ systèmes d’exploitation pris en charge par Azure Security Center](security-center-os-coverage.md) pour vous assurer que votre système d’exploitation est pris en charge. Pour plus d’informations, voir [Clients Log Analytics actuels](./faq-azure-monitor-logs.md).
 

## <a name="disable-auto-provisioning"></a>Désactiver le provisionnement automatique<a name="offprovisioning"></a>

Lorsque vous désactivez le provisionnement automatique, les agents ne sont pas provisionnés sur les nouvelles machines virtuelles.

Pour désactiver le provisionnement automatique d’un agent :

1. Dans le menu de Security Center dans le portail, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’abonnement approprié.
1. Sélectionnez **Provisionnement automatique**.
1. Basculez l’état sur **Désactivé** pour l’agent approprié.

    :::image type="content" source="./media/security-center-enable-data-collection/agent-toggles.png" alt-text="Désactiver le provisionnement automatique par type d’agent":::

1. Sélectionnez **Enregistrer**. Quand le provisionnement automatique est désactivé, la section de configuration de l’espace de travail par défaut n’est pas affichée :

    :::image type="content" source="./media/security-center-enable-data-collection/empty-configuration-column.png" alt-text="Lorsque le provisionnement automatique est désactivé, la cellule de configuration est vide":::


> [!NOTE]
>  La désactivation de l’approvisionnement automatique ne supprime pas le Log Analytics Agent des machines virtuelles Azure sur lesquelles l’agent était approvisionné. Pour plus d’informations sur la suppression de l’extension OMS, consultez [Comment supprimer des extensions OMS installées par Security Center](faq-data-collection-agents.md#remove-oms).
>


## <a name="troubleshooting"></a>Dépannage

-   Pour identifier des problèmes d’approvisionnement automatique, consultez [Problèmes d’intégrité de l’agent de surveillance](security-center-troubleshooting-guide.md#mon-agent).

-  Pour identifier la configuration réseau requise pour l’agent de surveillance, consultez [Résolution des problèmes de configuration réseau requise de l’agent de surveillance](security-center-troubleshooting-guide.md#mon-network-req).
-   Pour identifier des problèmes d’intégration manuelle, voir [Comment faire pour résoudre les problèmes d’intégration de Microsoft Operations Management Suite](https://support.microsoft.com/help/3126513/how-to-troubleshoot-operations-management-suite-onboarding-issues).

- Pour identifier les problèmes liés aux machines virtuelles et ordinateurs non surveillés :

    Une machine virtuelle ou un ordinateur n’est pas supervisé par Security Center si la machine n’exécute pas l’extension de l’agent Log Analytics. Un agent local peut être déjà installé sur une machine, par exemple l’agent direct OMS ou l’agent System Center Operations Manager. Les machines sur lesquelles sont installés ces agents sont considérées comme non surveillées, car ces agents ne sont pas entièrement pris en charge par Security Center. Pour tirer pleinement parti de toutes les fonctionnalités de Security Center, l’extension de l’agent Log Analytics est nécessaire.

    Pour en savoir plus sur la raison pour laquelle Security Center ne peut pas superviser correctement les machines virtuelles et ordinateurs initialisés pour le provisionnement automatique, consultez [Problèmes d’intégrité de l’agent de supervision](security-center-troubleshooting-guide.md#mon-agent).




## <a name="next-steps"></a>Étapes suivantes
Cet article vous a montré le fonctionnement de la collecte de données et de l’approvisionnement automatique dans Security Center. Pour plus d’informations sur Security Center, consultez les pages suivantes :

- [FAQ de Azure Security Center](faq-general.md): forum aux questions concernant l’utilisation de ce service.
- [Surveillance de l’intégrité de la sécurité dans Azure Security Center](security-center-monitoring.md): découvrez comment surveiller l’intégrité de vos ressources Azure.

Cet article explique comment installer un Log Analytics Agent et de définir un espace de travail Log Analytics dans lequel stocker les données collectées. Les deux opérations sont nécessaires pour activer la collecte de données. Le stockage de données dans Log Analytics, que vous utilisiez un espace de travail nouveau ou existant, peut occasionner des frais supplémentaires. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/security-center/).

