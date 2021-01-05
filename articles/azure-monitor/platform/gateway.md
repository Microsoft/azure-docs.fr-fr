---
title: Connecter des ordinateurs à l’aide de la passerelle Log Analytics | Microsoft Docs
description: Connectez vos appareils et les ordinateurs contrôlés par Operations Manager en utilisant la passerelle Log Analytics pour envoyer des données aux services Azure Automation et Log Analytics lorsqu’ils n’ont pas accès à Internet.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 12/24/2019
ms.openlocfilehash: dc4d1b852b0a498de0834731b2b1cd1225b9748b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008852"
---
# <a name="connect-computers-without-internet-access-by-using-the-log-analytics-gateway-in-azure-monitor"></a>Connecter des ordinateurs sans accès Internet en utilisant la passerelle Log Analytics dans Azure Monitor

>[!NOTE]
>Microsoft Operations Management Suite (OMS) devient Microsoft Azure Monitor. Suite à cette transition, notre terminologie évolue. Cet article fait référence à la passerelle OMS en tant que passerelle Azure Log Analytics. 
>

Cet article décrit comment configurer la communication avec Azure Automation et Azure Monitor à l’aide de la passerelle Log Analytics lorsque les ordinateurs qui sont directement connectés ou qui doivent être analysés par Operations Manager n’ont pas accès à Internet. 

La passerelle Log Analytics est un proxy de transfert HTTP qui prend en charge le tunneling HTTP à l’aide de la commande HTTP CONNECT. Cette passerelle envoie des données à Azure Automation et à un espace de travail Log Analytics dans Azure Monitor pour le compte des ordinateurs qui ne peuvent pas se connecter directement à Internet. 

La passerelle Log Analytics prend en charge ce qui suit :

* La création de rapports vers les mêmes espaces de travail Log Analytics configurés sur chaque agent situé derrière la passerelle et configurés avec des Runbook Workers hybrides d’Azure Automation.  
* Les ordinateurs Windows sur lequel Microsoft Monitoring Agent est directement connecté à un espace de travail Log Analytics dans Azure Monitor.
* Les ordinateurs Linux sur lesquels un agent Log Analytics pour Linux est directement connecté à un espace de travail Log Analytics dans Azure Monitor.  
* System Center Operations Manager 2012 SP1 avec UR7, Operations Manager 2012 R2 avec UR3, ou un groupe d’administration dans Operations Manager 2016 ou ultérieur intégré dans Log Analytics.  

Certaines stratégies de sécurité informatique interdisent aux ordinateurs du réseau de se connecter à Internet. Ces ordinateurs non connectés peuvent être, par exemple, des appareils ou des serveurs de point de vente (PDV) prenant en charge des services informatiques. Pour connecter ces appareils à Azure Automation ou à un espace de travail Log Analytics afin de pouvoir les gérer et les surveiller, vous devez les configurer pour communiquer directement avec la passerelle Log Analytics. La passerelle Log Analytics peut recevoir des informations de configuration et transférer des données pour le compte de ces appareils/serveurs. Si ces ordinateurs sont configurés avec l’agent Log Analytics pour se connecter directement à un espace de travail Log Analytics, tous les ordinateurs communiqueront plutôt avec la passerelle Log Analytics.  

La passerelle de Log Analytics transfère directement les données, des agents au service. Elle n’analyse pas les données en transit et ne met pas en cache les données lorsqu’elle perd la connectivité avec le service. Lorsque la passerelle ne parvient pas à communiquer avec le service, l’agent continue à s’exécuter et place en file d’attente les données collectées sur le disque de l’ordinateur surveillé. Une fois la connexion restaurée, l’agent envoie les données en cache collectées à Azure Monitor.

Lorsqu’un groupe d’administration Operations Manager est intégré à Log Analytics, les serveurs d’administration peuvent être configurés pour se connecter à la passerelle Log Analytics pour recevoir des informations de configuration et envoyer les données collectées en fonction de la solution que vous avez activée.  Les agents Operations Manager envoient certaines données au serveur d’administration. Par exemple, les agents peuvent envoyer des alertes Operations Manager, des données d’évaluation de configuration, des données d’espace d’instance et des données de capacité. D’autres données volumineuses, telles que les journaux d’activité IIS (Internet Information Services), les données de performances et les événements de sécurité, sont envoyées directement à la passerelle Log Analytics. 

Si vous avez un ou plusieurs serveurs de passerelle Operations Manager déployés pour analyser des systèmes non fiables dans un réseau de périmètre ou un réseau isolé, ils ne peuvent pas communiquer avec une passerelle Log Analytics.  Les serveurs de passerelle Operations Manager peuvent uniquement générer des rapports sur un serveur d’administration.  Lorsqu’un groupe d’administration Operations Manager est configuré pour communiquer avec la passerelle Log Analytics, les informations de configuration de proxy sont automatiquement distribuées à tous les ordinateurs gérés par agent qui sont configurés pour collecter des données de journal pour Azure Monitor, même si le paramètre est vide.

Pour fournir une haute disponibilité aux groupes directement connectés ou aux groupes Operations Management qui communiquent avec un espace de travail Log Analytics via la passerelle, vous pouvez utiliser l’équilibrage de charge au niveau du réseau pour rediriger et distribuer le trafic entre plusieurs serveurs de passerelle. Ainsi, en cas de panne de serveur de passerelle, le trafic est redirigé vers un autre nœud disponible.  

L’ordinateur qui exécute la passerelle Log Analytics demande à l’agent Windows Log Analytics d’identifier les points de terminaison de service avec lesquels la passerelle doit communiquer. L’agent doit également diriger les rapports de la passerelle vers les espaces de travail configurés pour les agents ou le groupe d’administration Operations Manager derrière la passerelle. Cette configuration permet à la passerelle et l’agent pour communiquer avec l’espace de travail qui leur est attribué.

Une passerelle peut être multirésidente sur un maximum de quatre espaces de travail. Il s’agit du nombre total d’espaces de travail qu’un agent Windows peut prendre en charge.  

Chaque agent doit disposer d’une connexion réseau avec la passerelle, afin que les agents puissent automatiquement transférer des données vers la passerelle et recevoir des données de cette dernière. Évitez d’installer la passerelle sur un contrôleur de domaine. Les ordinateurs Linux qui se trouvent derrière un serveur de passerelle ne peuvent pas utiliser la méthode d’[installation de script Wrapper](agent-linux.md#install-the-agent-using-wrapper-script) pour installer l’agent Log Analytics pour Linux. L’agent doit être téléchargé manuellement, copié sur l’ordinateur et installé manuellement, car la passerelle ne prend en charge que la communication avec les services Azure mentionnés précédemment.

Le diagramme suivant affiche le flux de données entre les agents directs et Azure Automation avec Log Analytics utilisant le serveur de passerelle. La configuration de proxy des agents doit correspondre au même port que celui avec lequel la passerelle Log Analytics est configurée.  

![Diagramme de communication directe entre l’agent et les services](./media/gateway/oms-omsgateway-agentdirectconnect.png)

Le diagramme suivant illustre le flux de données entre un groupe d’administration Operations Manager et Log Analytics.   

![Diagramme de communication entre Operations Manager et Log Analytics](./media/gateway/log-analytics-agent-opsmgrconnect.png)

## <a name="set-up-your-system"></a>Configurer votre système

Les ordinateurs désignés pour exécuter la passerelle Log Analytics doivent respecter la configuration suivante :

* Windows 10, Windows 8.1 ou Windows 7
* Windows Server 2019, Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 ou Windows Server 2008
* Microsoft .NET Framework 4.5
* Au moins un processeur 4 cœurs et 8 Go de mémoire 
* Un [agent Log Analytics pour Windows](agent-windows.md) configuré pour transmettre ses rapports vers l’espace de travail avec lequel les agents communiquent via la passerelle

### <a name="language-availability"></a>Langues disponibles

La passerelle Log Analytics est disponible dans les langues suivantes :

- Chinois (simplifié)
- Chinois (traditionnel)
- Tchèque
- Néerlandais
- Anglais
- Français
- Allemand
- Hongrois
- Italien
- Japonais
- Coréen
- Polonais
- Portugais (Brésil)
- Portugais (Portugal)
- Russe
- Espagnol (international)

### <a name="supported-encryption-protocols"></a>Protocoles de chiffrement pris en charge

La passerelle Log Analytics prend uniquement en charge les versions 1.0, 1.1 et 1.2 du protocole TLS.  Elle ne prend pas en charge le protocole SSL.  Pour garantir la sécurité des données en transit vers Log Analytics, configurez la passerelle pour qu’elle utilise au moins la version 1.2 du protocole TLS. Les versions antérieures des protocoles SSL ou TLS sont vulnérables. Même si elles permettent la compatibilité descendante, évitez de les utiliser.  

Pour plus d’informations, passez en revue [Envoi sécurisé de données via TLS 1.2](./data-security.md#sending-data-securely-using-tls-12). 

### <a name="supported-number-of-agent-connections"></a>Nombre de connexion d’agent prises en charge

Le tableau suivant indique le nombre approximatif d’agents pouvant communiquer avec un serveur de passerelle. Cette prise en charge est basée sur des agents qui chargent environ 200 Ko de données toutes les 6 secondes. Pour chaque agent testé, le volume de données est d’environ 2,7 Go par jour.

|Passerelle |Nombre (approximatif) d’agents pris en charge|  
|--------|----------------------------------|  
|Processeur : Intel Xeon Processor E5-2660 v3 \@ 2,6 GHz 2 cœurs<br> Mémoire : 4 Go<br> Bande passante réseau : 1 Gbit/s| 600|  
|Processeur : Intel Xeon Processor E5-2660 v3 \@ 2,6 GHz 4 cœurs<br> Mémoire : 8 Go<br> Bande passante réseau : 1 Gbit/s| 1 000|  

## <a name="download-the-log-analytics-gateway"></a>Téléchargement de la passerelle Log Analytics

La dernière version du fichier de configuration de la passerelle Log Analytics est disponible dans le Centre de téléchargement Microsoft ([Lien de téléchargement](https://go.microsoft.com/fwlink/?linkid=837444)) ou sur le Portail Azure.

Pour obtenir la passerelle Log Analytics sur le portail Microsoft Azure, procédez comme suit :

1. Parcourez la liste des services, puis sélectionnez **Log Analytics**. 
1. Sélectionnez un espace de travail.
1. Dans le panneau de votre espace de travail, sous **Général**, sélectionnez **Démarrage rapide**. 
1. Sous **Choisir une source de données pour se connecter à l’espace de travail**, sélectionnez **Ordinateurs**.
1. Dans le panneau **Agent direct**, sélectionnez **Télécharger la passerelle Log Analytics**.
 
   ![Capture d’écran indiquant la procédure de téléchargement de la passerelle Log Analytics](./media/gateway/download-gateway.png)

or 

1. Dans le panneau de votre espace de travail, sous **Paramètres**, sélectionnez **Paramètres avancés**.
1. Accédez à **Sources connectées** > **Serveurs Windows**, puis sélectionnez **Télécharger la passerelle Log Analytics**.

## <a name="install-log-analytics-gateway-using-setup-wizard"></a>Installer la passerelle Log Analytics à l’aide d’Assistant Installation

Pour installer une passerelle à l’aide de l’Assistant Installation, procédez comme suit. 

1. Dans le dossier de destination, double-cliquez sur **Log Analytics gateway.msi**.
1. Sur la page d’**accueil**, sélectionnez **Suivant**.

   ![Capture d’écran de la page d’accueil de l’Assistant d’installation de la passerelle](./media/gateway/gateway-wizard01.png)

1. Sur la page du **contrat de licence**, sélectionnez **J’accepte les termes du contrat de licence** pour accepter les termes du contrat de licence logiciel Microsoft, puis sélectionnez **Suivant**.
1. Sur la page relative au **port et à l’adresse proxy** :

   a. Entrez le numéro de port TCP à utiliser pour la passerelle. Le programme d’installation utilise ce numéro de port pour configurer une règle entrante sur le pare-feu Windows.  La valeur par défaut est 8080.
      Les numéros de port valides sont compris entre 1 et 65535. Si la valeur saisie n’est pas comprise dans cette plage, un message d’erreur s’affiche.

   b. Si le serveur sur lequel la passerelle est installée doit communiquer par le biais d’un proxy, saisissez l’adresse proxy à laquelle la passerelle doit se connecter. Par exemple, entrez `http://myorgname.corp.contoso.com:80`.  Si vous laissez ce champ vide, la passerelle tentera de se connecter directement à Internet.  Si votre serveur proxy requiert une authentification, entrez un nom d'utilisateur et un mot de passe.

   c. Sélectionnez **Suivant**.

   ![Capture d’écran indiquant la configuration du proxy de la passerelle](./media/gateway/gateway-wizard02.png)

1. Si vous n’avez pas activé Microsoft Update, la page Microsoft Update s’affiche et vous pouvez choisir de l’activer. Effectuez une sélection, puis cliquez sur **Suivant**. Sinon, passez à l’étape suivante.
1. Sur la page **Dossier de destination**, conservez le dossier par défaut C:\Program Files\OMS Gateway ou indiquez l’emplacement où vous souhaitez installer la passerelle. Sélectionnez ensuite **Suivant**.
1. Sur la page **Prêt pour l’installation**, sélectionnez **Installer**. Si le contrôle de compte utilisateur envoie une requête d’autorisation d’installation, veuillez sélectionner **Oui**.
1. Une fois l’installation terminée, sélectionnez **Terminer**. Pour vérifier que le service est en cours d’exécution, ouvrez le composant logiciel enfichable services.msc et vérifiez que **Passerelle OMS** apparaît dans la liste des services et que son statut est défini sur **Exécution en cours**.

   ![Capture d’écran indiquant les services locaux, montrant que la passerelle OMS est en cours d’exécution](./media/gateway/gateway-service.png)

## <a name="install-the-log-analytics-gateway-using-the-command-line"></a>Installer la passerelle Log Analytics en utilisant l’interface en ligne de commande

Le fichier téléchargé pour la passerelle est un package Windows Installer qui prend en charge une installation sans assistance à partir de l’interface en ligne de commande ou d’une autre méthode automatisée. Si vous n’êtes pas familiarisé avec les options standards de l’interface de ligne de commande pour Windows Installer, veuillez consulter la page [Options de l’interface en ligne de commande](/windows/desktop/msi/command-line-options).
 
Le tableau suivant répertorie les paramètres pris en charge par le programme d’installation.

|Paramètres| Notes|
|----------|------| 
|PORTNUMBER | Numéro de port TCP pour la passerelle à écouter |
|PROXY | Adresse IP du serveur proxy |
|INSTALLDIR | Chemin d’accès complet pour spécifier le répertoire d’installation des fichiers du logiciel de passerelle |
|USERNAME | Identifiant utilisateur nécessaire à l’authentification auprès du serveur proxy |
|PASSWORD | Mot de passe associé à l’identifiant utilisateur pour l’authentification auprès du proxy |
|LicenseAccepted | Entrez la valeur **1** pour valider votre acceptation du contrat de licence |
|HASAUTH | Entrez la valeur **1** lorsque les paramètres USERNAME/PASSWORD sont spécifiés |
|HASPROXY | Entrez la valeur **1** lors de la spécification de l’adresse IP pour le paramètre **PROXY** |

Pour installer la passerelle en mode sans assistance et la configurer avec une adresse de serveur proxy et un numéro de port spécifiques, tapez la commande suivante :

```dos
Msiexec.exe /I "oms gateway.msi" /qn PORTNUMBER=8080 PROXY="10.80.2.200" HASPROXY=1 LicenseAccepted=1 
```

L’option de ligne de commande /qn masque le programme d’installation. L’option de ligne de commande /qb affiche le programme d’installation lors de l’installation sans assistance.  

Si vous devez fournir des informations d’identification pour l’authentification auprès du proxy, tapez la commande suivante :

```dos
Msiexec.exe /I "oms gateway.msi" /qn PORTNUMBER=8080 PROXY="10.80.2.200" HASPROXY=1 HASAUTH=1 USERNAME="<username>" PASSWORD="<password>" LicenseAccepted=1 
```

Après l’installation, vous pouvez vérifier que les paramètres sont acceptés (à l’exception du nom d’utilisateur et du mot de passe) en utilisant les cmdlets PowerShell suivantes :

- **Get-OMSGatewayConfig** : retourne le port TCP que la passerelle est configurée pour écouter.
- **Get-OMSGatewayRelayProxy** : retourne l’adresse IP du serveur proxy que vous avez configuré pour la communication.

## <a name="configure-network-load-balancing"></a>Configuration de l’équilibrage de la charge réseau

Vous pouvez configurer la passerelle pour la haute disponibilité à l’aide de [l’équilibrage de la charge réseau (NLB) Microsoft](/windows-server/networking/technologies/network-load-balancing), [Azure Load Balancer](../../load-balancer/load-balancer-overview.md) ou les équilibreurs de charge basés sur le matériel. L’équilibreur de charge gère le trafic en redirigeant les connexions demandées à partir des agents Log Analytics ou des serveurs d’administration Operations Manager sur ses nœuds. Si un serveur de passerelle tombe en panne, le trafic est redirigé vers d’autres nœuds.

### <a name="microsoft-network-load-balancing"></a>Équilibrage de la charge réseau Microsoft

Pour apprendre à concevoir et déployer un cluster d’équilibrage de charge réseau Windows Server 2016, consultez [Équilibrage de charge réseau](/windows-server/networking/technologies/network-load-balancing). Les étapes suivantes décrivent comment configurer un cluster d’équilibrage de charge réseau Microsoft.  

1. Connectez-vous au serveur Windows qui est membre du cluster d’équilibrage de charge réseau avec un compte d’administration.  
2. Ouvrez le Gestionnaire d’équilibrage de charge réseau dans le Gestionnaire de serveur, cliquez sur **Outils**, puis sur **Gestionnaire d’équilibrage de charge réseau**.
3. Pour vous connecter à un serveur de passerelle Log Analytics sur lequel Microsoft Monitoring Agent est installé, faites un clic droit sur l’adresse IP du cluster, puis cliquez sur **Ajouter l’hôte au cluster**. 

    ![Gestionnaire d’équilibrage de la charge réseau : ajoutez l’hôte au cluster](./media/gateway/nlb02.png)
 
4. Entrez l’adresse IP du serveur de passerelle que vous voulez connecter. 

    ![Gestionnaire d’équilibrage de charge réseau – Ajouter l’hôte au cluster : Se connecter](./media/gateway/nlb03.png) 

### <a name="azure-load-balancer"></a>Azure Load Balancer

Pour apprendre à concevoir et déployer une instance d’Azure Load Balancer, veuillez consulter l’article [Qu’est-ce qu’Azure Load Balancer ?](../../load-balancer/load-balancer-overview.md). Pour déployer un équilibreur de charge de base, veuillez suivre les étapes décrites dans ce [guide de démarrage rapide](../../load-balancer/quickstart-load-balancer-standard-public-portal.md), excepté la procédure décrite dans la section **Créer des serveurs principaux**.   

> [!NOTE]
> Si vous souhaitez configurer Azure Load Balancer à l’aide de la **référence SKU De base**, vos machines virtuelles doivent appartenir à un groupe à haute disponibilité. Pour en savoir plus sur les groupes à haute disponibilité, veuillez consulter l’article [Gestion de la disponibilité des machines virtuelles Windows dans Azure](../../virtual-machines/manage-availability.md). Pour ajouter des ordinateurs virtuels existants à un groupe à haute disponibilité, veuillez consulter l’article [Définir une machine virtuelle Azure Resource Manager dans un groupe à haute disponibilité](https://gallery.technet.microsoft.com/Set-Azure-Resource-Manager-f7509ec4).
> 

Après avoir créé l’équilibreur de charge, vous devez créer un pool principal. Ce dernier distribuera le trafic vers un ou plusieurs serveurs de passerelle. Pour cela, veuillez suivre la procédure décrite dans la section du guide de démarrage rapide [Créer des ressources pour l’équilibreur de charge](../../load-balancer/quickstart-load-balancer-standard-public-portal.md).  

>[!NOTE]
>La sonde d’intégrité doit être configurée pour utiliser le port TCP du serveur de passerelle. La sonde d’intégrité ajoute ou supprime de façon dynamique des serveurs de passerelle dans la rotation de l’équilibreur de charge en fonction de leur réponse aux contrôles d’intégrité. 
>

## <a name="configure-the-log-analytics-agent-and-operations-manager-management-group"></a>Configurer l’agent Log Analytics et le groupe d’administration Operations Manager

Cette section explique comment configurer les agents Log Analytics connectés directement, d’un groupe d’administration Operations Manager ou Runbook Worker hybride Azure Automation avec la passerelle Log Analytics pour communiquer avec Azure Automation ou Log Analytics.  

### <a name="configure-a-standalone-log-analytics-agent"></a>Configuration d’un agent Log Analytics autonome

Lorsque vous configurez l’agent Log Analytics, remplacez la valeur du serveur proxy par l’adresse IP du serveur de passerelle Log Analytics et son numéro de port. Si vous avez déployé plusieurs serveurs de passerelle derrière un équilibreur de charge, la configuration de proxy de l’agent Log Analytics est l’adresse IP virtuelle de l’équilibreur de charge.  

>[!NOTE]
>Pour installer l’agent Log Analytics sur la passerelle et les ordinateurs Windows qui se connectent directement à Log Analytics, veuillez consulter l’article [Connecter des ordinateurs Windows au service Log Analytics dans Azure](agent-windows.md). Pour connecter des ordinateurs Linux, consultez [Connecter des ordinateurs Linux à Azure Monitor](agent-linux.md). 
>

Après avoir installé l’agent sur le serveur de passerelle, configurez-le pour transférer ses rapports à l’espace de travail ou aux agents de l’espace de travail qui communiquent avec la passerelle. Si l’agent Windows Log Analytics n’est pas installé sur la passerelle, l’événement 300 est écrit dans le journal des événements de la Passerelle OMS pour signaler que l’agent doit être installé. Si l’agent est installé mais pas configuré pour transférer ses rapports au même espace de travail que les agents communiquant par son intermédiaire, l’événement 105 est écrit dans le même journal, pour signaler que l’agent sur la passerelle doit être configuré pour transférer ses rapports au même espace de travail que les agents communiquant avec la passerelle.

Quand vous avez effectué la configuration, redémarrez le service de **passerelle OMS** pour appliquer les modifications. Autrement, la passerelle rejettera les agents tentant de communiquer avec Log Analytics et écrira l’événement 105 dans le journal des événements de la Passerelle OMS. Ceci peut également arriver si vous ajoutez ou supprimez un espace de travail de la configuration de l’agent sur le serveur de passerelle.

Pour plus d’informations sur le Runbook Worker hybride Automation, veuillez consulter l’article [Automatiser les ressources de votre centre de données ou de votre cloud à l’aide d’un Runbook Worker hybride](../../automation/automation-hybrid-runbook-worker.md).

### <a name="configure-operations-manager-where-all-agents-use-the-same-proxy-server"></a>Configurer Operations Manager quand tous les agents utilisent le même serveur proxy

La configuration du proxy Operations Manager est automatiquement appliquée à tous les agents générant un rapport pour Operations Manager, même si le paramètre est vide.  

Pour que la passerelle OMS prenne en charge Operations Manager, vous devez disposer des éléments suivants :

* Microsoft Monitoring Agent (version 8.0.10900.0 ou ultérieure) installé sur le serveur de passerelle OMS et configuré avec les mêmes espaces de travail Log Analytics que ceux auxquels votre groupe d’administration est configuré pour rendre compte.
* Une connexion à Internet. Sinon, la passerelle OMS doit être connectée à un serveur proxy connecté à Internet.

> [!NOTE]
> Si vous ne spécifiez pas de valeur pour la passerelle, les valeurs vides sont transférées à tous les agents.
>

S’il s’agit de la première fois que votre groupe d’administration Operations Manager s’inscrit auprès d’un espace de travail Log Analytics, l’option permettant de spécifier la configuration du proxy pour le groupe d’administration n’est pas disponible dans la console Opérations. Pour que cette option soit disponible, le groupe d’administration doit être inscrit auprès du service.  

Pour configurer l’intégration, mettez à jour la configuration du proxy système à l’aide de Netsh sur le système à partir duquel vous exécutez la console Opérations, puis sur tous les serveurs d’administration du groupe d’administration. Procédez comme suit :

1. Ouvrez une invite de commandes avec élévation de privilèges :

   a. Sélectionnez **Démarrer**, puis entrez **cmd**.  

   b. Cliquez avec le bouton droit sur **Invite de commandes**, puis sélectionnez **Exécuter en tant qu’administrateur**.  

1. Entrez la commande suivante :

   `netsh winhttp set proxy <proxy>:<port>`

À l’issue de l’intégration avec Log Analytics, supprimez la modification en exécutant `netsh winhttp reset proxy`. Ensuite, dans la console Opérations, utilisez l’option **Configurer le serveur proxy** pour spécifier le serveur de passerelle Log Analytics. 

1. Dans la console Operations Manager, sous **Operations Management Suite**, sélectionnez **Connexion**, puis **Configurer le serveur Proxy**.

   ![Capture d’écran d’Operations Manager, indiquant la sélection Configurer le serveur Proxy](./media/gateway/scom01.png)

1. Sélectionnez **Utiliser un serveur proxy pour accéder à Operations Management Suite**, puis entrez l’adresse IP du serveur de la passerelle Log Analytics ou l’adresse IP virtuelle de l’équilibreur de charge réseau. Lorsque vous entrez l’adresse, n’oubliez pas le préfixe `http://`.

   ![Capture d’écran d’Operations Manager, indiquant l’adresse du serveur proxy](./media/gateway/scom02.png)

1. Sélectionnez **Terminer**. Votre groupe d’administration Operations Manager est maintenant configuré pour communiquer via le serveur de passerelle pour le service Log Analytics.

### <a name="configure-operations-manager-where-specific-agents-use-a-proxy-server"></a>Configurer Operations Manager quand certains agents utilisent un serveur proxy

Si votre environnement est complexe ou volumineux, vous pouvez souhaiter que seuls certains serveurs (ou groupes) utilisent le serveur de passerelle Log Analytics.  Vous ne pouvez pas mettre à jour l’agent Operations Manager directement pour ces serveurs. En effet, cette valeur est remplacée par la valeur globale du groupe d’administration.  Au lieu de cela, vous devez remplacer la règle utilisée pour le transfert de ces valeurs.  

> [!NOTE] 
> Utilisez cette technique de configuration si vous souhaitez permettre l’utilisation de plusieurs serveurs de passerelle Log Analytics dans votre environnement.  Par exemple, vous pouvez exiger que certains serveurs de passerelle Log Analytics soient définis selon leur zone géographique.
>

Pour configurer certains serveurs ou groupes afin qu’ils utilisent le serveur de passerelle Log Analytics : 

1. Ouvrez la console Operations Manager, puis sélectionnez l'espace de travail **Création**.  
1. Dans l’espace de travail Création, sélectionnez **Règles**. 
1. Dans la barre d’outils Operations Manager, sélectionnez le bouton **Étendue**. Si ce bouton n’est pas disponible, vérifiez que vous avez sélectionné un objet, et non un dossier, dans le panneau **Analyse**. La boîte de dialogue **Objets du pack de gestion de l’étendue** affiche une liste des classes, groupes ou objets couramment ciblés. 
1. Dans le champ **Rechercher**, entrez **Service de contrôle d’intégrité**, puis sélectionnez-le dans la liste. Sélectionnez **OK**.  
1. Recherchez la règle **Advisor Proxy Setting Rule** (Règle de configuration du paramètre proxy Advisor). 
1. Dans la barre d’outils Operations Manager, sélectionnez **Remplace**, puis pointez sur **Remplacer la règle\Pour un objet spécifique de la classe : Service de contrôle d’intégrité**, puis sélectionnez un objet dans la liste.  Ou créez un groupe personnalisé contenant l’objet Service de contrôle d’intégrité, pour les serveurs auxquels vous souhaitez appliquer ce remplacement. Ensuite, appliquez le remplacement à votre groupe personnalisé.
1. Dans la boîte de dialogue **Propriétés du remplacement**, cochez la case dans la colonne **Remplacer** en regard du paramètre **WebProxyAddress**.  Dans le champ **Remplacer la valeur**, entrez l’URL du serveur de passerelle Log Analytics. Lorsque vous entrez l’adresse, n’oubliez pas le préfixe `http://`.  

    >[!NOTE]
    > Vous n’avez pas besoin d’activer la règle. Elle est déjà gérée automatiquement avec un remplacement dans le pack d’administration Microsoft System Center Advisor Secure Reference Override qui cible le groupe Microsoft System Center Advisor Monitoring Server.
    > 

1. Sélectionnez un pack d’administration dans la liste **Sélectionner un pack d’administration de destination** ou créez un pack d’administration non scellé en sélectionnant **Nouveau**. 
1. Lorsque vous avez terminé, sélectionnez **OK**. 

### <a name="configure-for-automation-hybrid-runbook-workers"></a>Configurer des Runbooks Workers hybrides Automation

Si votre environnement contient des Runbooks Workers hybrides Automation, veuillez suivre ces étapes pour configurer la passerelle de sorte qu’elle prenne en charge les rôles de travail.

Reportez-vous à la section [Configurer votre réseau](../../automation/automation-hybrid-runbook-worker.md#network-planning) de la documentation Automation pour trouver l’URL propre à chaque région.

Si votre ordinateur est automatiquement inscrit en tant que Runbook Worker hybride, par exemple si la solution de gestion des mises à jour est activée pour une ou plusieurs machines virtuelles, suivez ces étapes :

1. Ajoutez les URL de service de données d’exécution de la tâche à la liste d’hôtes autorisés sur la passerelle Log Analytics. Par exemple : `Add-OMSGatewayAllowedHost we-jobruntimedata-prod-su1.azure-automation.net`
1. Redémarrez le service de passerelle Log Analytics en utilisant la cmdlet PowerShell suivante : `Restart-Service OMSGatewayService`

Si votre ordinateur est joint à Azure Automation à l’aide de la cmdlet d’inscription Runbook Worker hybride, suivez ces étapes :

1. Ajoutez l’URL d’enregistrement de service de l’agent à la liste d’hôtes autorisés sur la passerelle Log Analytics. Par exemple : `Add-OMSGatewayAllowedHost ncus-agentservice-prod-1.azure-automation.net`
1. Ajoutez les URL de service de données d’exécution de la tâche à la liste d’hôtes autorisés sur la passerelle Log Analytics. Par exemple : `Add-OMSGatewayAllowedHost we-jobruntimedata-prod-su1.azure-automation.net`
1. Redémarrez le service de passerelle Log Analytics.
    `Restart-Service OMSGatewayService`

## <a name="useful-powershell-cmdlets"></a>Applets de commande PowerShell utiles

Certaines cmdlets peuvent vous aider à effectuer les tâches nécessaires à la mise à jour des paramètres de configuration de la passerelle Log Analytics. Avant d’utiliser des cmdlets, veillez à :

1. Installer la passerelle Log Analytics (Microsoft Windows Installer).
1. Ouvrez une fenêtre de console PowerShell.
1. Importez le module en tapant la commande : `Import-Module OMSGateway`
1. Si aucune erreur ne s’est produite lors des étapes précédentes, cela signifie que le module a été importé, et que les cmdlets peuvent être utilisées. Entrez `Get-Module OMSGateway`
1. Après avoir utilisé les cmdlets pour apporter des modifications, veuillez redémarrer le service de passerelle OMS.

Si une erreur survient lors de l’étape 3, cela signifie que le module n’a pas été importé. L’erreur peut se produire lorsque PowerShell ne peut pas trouver le module. Vous pouvez rechercher le module dans le chemin d’installation de la passerelle OMS : *C:\Program Files\Microsoft OMS Gateway\PowerShell\OmsGateway*.

| **Applet de commande** | **Paramètres** | **Description** | **Exemple** |
| --- | --- | --- | --- |  
| `Get-OMSGatewayConfig` |Clé |Récupère la configuration du service |`Get-OMSGatewayConfig` |  
| `Set-OMSGatewayConfig` |Clé (obligatoire) <br> Valeur |Modifie la configuration du service |`Set-OMSGatewayConfig -Name ListenPort -Value 8080` |  
| `Get-OMSGatewayRelayProxy` | |Récupère l’adresse du proxy de relais (en amont) |`Get-OMSGatewayRelayProxy` |  
| `Set-OMSGatewayRelayProxy` |Adresse<br> Nom d’utilisateur<br> Mot de passe (chaîne sécurisée) |Définit l’adresse (et les informations d’identification) du proxy de relais (en amont) |1. Définir un proxy relais et des informations d’identification :<br> `Set-OMSGatewayRelayProxy`<br>`-Address http://www.myproxy.com:8080`<br>`-Username user1 -Password 123` <br><br> 2. Définir un proxy relais n’exigeant pas d’authentification : `Set-OMSGatewayRelayProxy`<br> `-Address http://www.myproxy.com:8080` <br><br> 3. Effacer le paramètre de proxy relais :<br> `Set-OMSGatewayRelayProxy` <br> `-Address ""` |  
| `Get-OMSGatewayAllowedHost` | |Récupère l’hôte actuellement autorisé (uniquement l’hôte configuré localement, pas les hôtes autorisés téléchargés automatiquement) |`Get-OMSGatewayAllowedHost` | 
| `Add-OMSGatewayAllowedHost` |Hôte (obligatoire) |Ajoute l’hôte à la liste d’éléments autorisés |`Add-OMSGatewayAllowedHost -Host www.test.com` |  
| `Remove-OMSGatewayAllowedHost` |Hôte (obligatoire) |Supprime l’hôte de la liste d’éléments autorisés |`Remove-OMSGatewayAllowedHost`<br> `-Host www.test.com` |  
| `Add-OMSGatewayAllowedClientCertificate` |Objet (obligatoire) |Ajoute l’objet du certificat client à la liste d’éléments autorisés |`Add-OMSGatewayAllowed`<br>`ClientCertificate` <br> `-Subject mycert` |  
| `Remove-OMSGatewayAllowedClientCertificate` |Objet (obligatoire) |Supprime l’objet du certificat client de la liste d’éléments autorisés |`Remove-OMSGatewayAllowed` <br> `ClientCertificate` <br> `-Subject mycert` |  
| `Get-OMSGatewayAllowedClientCertificate` | |Récupère les objets de certificat client actuellement autorisés (uniquement les objets configurés localement, pas les objets autorisés téléchargés automatiquement) |`Get-`<br>`OMSGatewayAllowed`<br>`ClientCertificate` |  

## <a name="troubleshooting"></a>Dépannage

Pour collecter des événements journalisés par la passerelle, vous devez avoir installé l’agent Log Analytics.

![Capture d’écran de la liste de l’observateur d’événements dans le journal de la passerelle Log Analytics](./media/gateway/event-viewer.png)

### <a name="log-analytics-gateway-event-ids-and-descriptions"></a>Identifiants et descriptions des événements de la passerelle Log Analytics

Le tableau suivant montre les identifiants et descriptions des événements du journal de la passerelle Log Analytics.

| **Identifiant** | **Description** |
| --- | --- |
| 400 |Toute erreur d’application ne disposant pas d’un identifiant spécifique. |
| 401 |Configuration incorrecte. Par exemple : listenPort = "texte" au lieu d’un nombre entier. |
| 402 |Exception lors de l’analyse des messages de liaison TLS. |
| 403 |Erreur de mise en réseau. Par exemple : connexion impossible au serveur cible. |
| 100 |Informations générales. |
| 101 |Le service a démarré. |
| 102 |Le service s’est arrêté. |
| 103 |Le client a reçu une commande HTTP CONNECT. |
| 104 |L’élément n’est pas une commande HTTP CONNECT. |
| 105 |Le serveur de destination ne figure pas dans la liste autorisée ou le port de destination n’est pas un port sécurisé (443). <br> <br> Vérifiez que l’agent MMA sur votre serveur de passerelle OMS et les agents communiquant avec la passerelle OMS sont connectés au même espace de travail Log Analytics. |
| 105 |ERREUR TcpConnection – Certificat client non valide : CN=passerelle. <br><br> Vérifiez que vous utilisez la version 1.0.395.0 ou ultérieure de la passerelle OMS. Vérifiez aussi que l’agent MMA sur votre serveur de passerelle OMS et les agents communiquant avec la passerelle OMS sont connectés au même espace de travail Log Analytics. |
| 106 |La version du protocole TLS/SSL n’est pas prise en charge.<br><br> La passerelle Log Analytics prend uniquement en charge Les version 1.0, 1.1 et 1.2 de TLS. Elle ne prend pas en charge le protocole SSL.|
| 107 |La session TLS a été vérifiée. |

### <a name="performance-counters-to-collect"></a>Compteurs de performances à collecter

Le tableau suivant montre les compteurs de performances disponibles pour la passerelle Log Analytics. Utiliser l’Analyseur de performances pour ajouter les compteurs.

| **Nom** | **Description** |
| --- | --- |
| Passerelle Log Analytics/Connexion du client actif |Nombre de connexions du réseau client actif (TCP) |
| Passerelle Log Analytics/Nombre d’erreurs |Nombre d’erreurs |
| Passerelle Log Analytics/Client connecté |Nombre de clients connectés |
| Passerelle Log Analytics/Nombre de rejets |Nombre de rejets dus à des erreurs de validation de TLS |

![Capture d’écran de l’interface de passerelle Log Analytics, indiquant des compteurs de performances](./media/gateway/counters.png)

## <a name="assistance"></a>Assistance

Lorsque vous êtes connecté au portail Microsoft Azure, vous pouvez obtenir de l’aide avec la passerelle Log Analytics ou tout autre service Azure ou fonctionnalité.
Pour obtenir de l’aide, sélectionnez l’icône en forme de point d’interrogation, située dans le coin supérieur droit du portail. Puis sélectionnez **Nouvelle demande de support**. Puis remplissez le formulaire de nouvelle demande de support.

![Capture d’écran d’une nouvelle demande de support](./media/gateway/support.png)

## <a name="next-steps"></a>Étapes suivantes

[Ajoutez des sources de données](./agent-data-sources.md) pour collecter des données provenant de vos sources connectées et les stocker dans votre espace de travail Log Analytics.