---
title: Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Automation
description: Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Automation
author: mgoedtel
ms.author: magoedte
ms.topic: conceptual
ms.date: 12/11/2020
ms.subservice: ''
ms.openlocfilehash: 26e7dbf3f5629d4691211b6c9b82446ba4035421
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347617"
---
# <a name="use-azure-private-link-to-securely-connect-networks-to-azure-automation"></a>Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Automation

Azure Private Endpoint est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service Automation dans votre réseau virtuel. Le trafic réseau entre les machines sur le réseau virtuel et le compte Automation traverse le réseau virtuel et une liaison privée sur le réseau principal de Microsoft, ce qui élimine l’exposition sur l’Internet public.

Par exemple, vous avez un réseau virtuel dans lequel vous avez désactivé l’accès Internet sortant. Toutefois, vous souhaitez accéder à votre compte Automation en privé et utiliser des fonctionnalités Automation comme les webhooks, State Configuration et les tâches runbook sur les Runbook Workers hybrides. En outre, vous souhaitez que les utilisateurs aient accès au compte Automation uniquement via le réseau virtuel.  Le déploiement d’un point de terminaison privé atteint ces objectifs.

Cet article explique quand utiliser et comment configurer un point de terminaison privé avec votre compte Automation.

![Vue d’ensemble conceptuelle de la liaison privée pour Azure Automation](./media/private-link-security/private-endpoints-automation.png)

>[!NOTE]
> Le support Private Link avec Azure Automation est uniquement disponible dans les clouds Azure commercial et Azure US Government.

## <a name="advantages"></a>Avantages

Grâce à Azure Private Link, vous pouvez :

- vous connecter en privé à Azure Automation sans ouvrir d’accès au réseau public.
- vous connecter en privé à l’espace de travail Log Analytics Azure Monitor sans ouvrir d’accès au réseau public.

    >[!NOTE]
    >Un point de terminaison privé distinct pour votre espace de travail Log Analytics est obligatoire si votre compte Automation est lié à un espace de travail Log Analytics pour transférer des données de tâche et quand vous avez activé des fonctionnalités telles que Update Management, Suivi des modifications et inventaire, State Configuration ou Start/Stop VMs during off-hours. Pour plus d’informations sur la liaison privée pour Azure Monitor, consultez [Utiliser Azure Private Link pour connecter en toute sécurité des réseaux à Azure Monitor](../../azure-monitor/platform/private-link-security.md).

- Vérifiez que vos données Automation sont accessibles uniquement par le biais de réseaux privés autorisés.
- Empêchez l’exfiltration de données à partir de vos réseaux privés en définissant vos ressources Azure Automation qui se connectent via votre point de terminaison privé
- Connectez en toute sécurité votre réseau local privé à Azure Automation à l’aide de ExpressRoute et de Private Link.
- Conservez tout le trafic au sein du réseau principal Microsoft Azure.

Pour plus d’informations, consultez [Principaux avantages d’Azure Private Link](../../private-link/private-link-overview.md#key-benefits).

## <a name="limitations"></a>Limites

- Dans l’implémentation actuelle de Private Link, les tâches cloud liées au compte Automation ne peuvent pas accéder aux ressources Azure qui sont sécurisées à l’aide d’un point de terminaison privé. Par exemple, Azure Key Vault, Azure SQL, compte de stockage Azure, etc. Pour contourner ce problème, utilisez à la place un [Runbook Worker hybride](../automation-hybrid-runbook-worker.md).
- Vous devez utiliser la dernière version de l’[agent Log Analytics](../../azure-monitor/platform/log-analytics-agent.md) pour Windows ou Linux.
- La [passerelle Log Analytics](../../azure-monitor/platform/gateway.md) ne prend pas en charge Private Link.

## <a name="how-it-works"></a>Fonctionnement

Azure Automation Private Link connecte un ou plusieurs points de terminaison privés (et, par conséquent, les réseaux virtuels dans lesquels ils sont contenus) à votre ressource de compte Automation. Ces points de terminaison sont des machines utilisant des webhooks pour démarrer un runbook, des machines hébergeant le rôle Runbook Worker hybride et des nœuds DSC (Desired State Configuration).

Une fois que vous avez créé des points de terminaison privés pour l’automatisation, chacune des URL Automation accessibles au public est mappée à un point de terminaison privé de votre réseau virtuel. Vous ou une machine pouvez contacter directement les URL Automation.

### <a name="webhook-scenario"></a>Scénario Webhook

Vous pouvez démarrer des runbooks en exécutant une PUBLICATION sur l’URL du webhook. Par exemple, l’URL ressemble à : `https://<automationAccountId>.webhooks.<region>.azure-automation.net/webhooks?token=gzGMz4SMpqNo8gidqPxAJ3E%3d`

### <a name="hybrid-runbook-worker-scenario"></a>Scénario Runbook Worker hybride

La fonctionnalité Runbook Worker hybride utilisateur d’Azure Automation vous permet d’exécuter des runbooks directement sur la machine Azure ou non-Azure, y compris les serveurs inscrits auprès de serveurs avec Azure Arc. Sur l’ordinateur ou le serveur qui héberge le rôle, vous pouvez exécuter les runbooks directement et avec les ressources disponibles dans l’environnement pour gérer ces ressources locales.

Un point de terminaison JRDS est utilisé par le Worker hybride pour démarrer/arrêter des runbooks, télécharger les runbooks sur le Worker et renvoyer le flux du journal de tâches au service Automation. Une fois le point de terminaison JRDS activé, l’URL ressemble à ceci : `https://<automationaccountID>.jobruntimedata.<region>.azure-automation.net`. De la sorte, l’exécution d’un runbook sur le Worker hybride connecté au réseau virtuel Azure est en mesure d’exécuter des tâches sans avoir à ouvrir de connexion sortante vers Internet.  

> [!NOTE]
>Avec l’implémentation actuelle des liaisons privées pour Azure Automation, seule est prise en charge l’exécution de tâches sur le Runbook Worker hybride connecté à un réseau virtuel Azure (les tâches cloud ne sont pas prises en charge).

## <a name="hybrid-worker-scenario-for-update-management"></a>Scénario Worker hybride pour Update Management  

Le Runbook Worker hybride système prend en charge un ensemble de runbooks masqués utilisés par la fonctionnalité Update Management et conçus pour installer des mises à jour spécifiées par l’utilisateur sur des ordinateurs Windows et Linux. Quand la solution Update Management d’Azure Automation est activée, toute machine connectée à votre espace de travail Log Analytics est automatiquement configurée en tant que Runbook Worker hybride système.

Pour comprendre et configurer Update Management, consultez [Vue d’ensemble d’Update Management](../update-management/overview.md). La fonctionnalité Update Management dépend d’un espace de travail Log Analytics et nécessite donc de lier l’espace de travail à un compte Automation. Un espace de travail Log Analytics stocke les données collectées par la solution et héberge ses recherches dans les journaux ainsi que ses vues.

Si vous souhaitez que vos machines configurées pour Update Management se connectent à Automation et à l’espace de travail Log Analytics de manière sécurisée via un canal Private Link, vous devez activer Private Link pour l’espace de travail Log Analytics lié au compte Automation configuré avec Private Link.

Vous pouvez contrôler la façon dont un espace de travail Log Analytics peut être atteint en dehors des étendues Private Link en suivant les étapes décrites dans [Configurer Log Analytics](../../azure-monitor/platform/private-link-security.md#configure-log-analytics). Si vous définissez **Autoriser l’accès au réseau public pour l’ingestion** sur **Non**, les machines en dehors des étendues connectées ne peuvent pas charger de données dans cet espace de travail. Si vous définissez **Autoriser l’accès au réseau public pour les requêtes** sur **Non**, les machines en dehors des étendues ne peuvent pas accéder aux données de cet espace de travail.

Utilisez la sous-ressource cible **DSCAndHybridWorker** afin d’activer Private Link pour les Workers hybrides système et utilisateur.

> [!NOTE]
> Les machines hébergées en dehors d’Azure qui sont managées par Update Management et sont connectées au réseau virtuel Azure via le peering privé ExpressRoute, des tunnels VPN et des réseaux virtuels appairés utilisant des points de terminaison privés prennent en charge Private Link.

### <a name="state-configuration-agentsvc-scenario"></a>Scénario State Configuration (agentsvc)

State Configuration fournit un service de gestion de la configuration Azure qui vous permet d’écrire, de gérer et de compiler des configurations PowerShell Desired State Configuration (DSC) pour les nœuds d’un cloud ou d’un centre de données local.

L’agent sur l’ordinateur s’inscrit auprès du service DSC, puis utilise le point de terminaison de service pour extraire la configuration DSC. Le point de terminaison de service de l’agent ressemble à : `https://<automationAccountId>.agentsvc.<region>.azure-automation.net`.

L’URL du point de terminaison public et privé est la même, mais elle est mappée à une adresse IP privée lorsque la liaison privée est activée.

## <a name="planning-based-on-your-network"></a>Planification en fonction de votre réseau

Avant de configurer votre ressource de compte Automation, tenez compte de vos exigences en matière d’isolement réseau. Évaluez l’accès de vos réseaux virtuels à Internet public et les restrictions d’accès à votre compte Automation (y compris la configuration d’une étendue de groupe de liaisons privées pour des journaux Azure Monitor s’ils sont intégrés à votre compte Automation). Incluez également une révision du service Automation [Enregistrements DNS](./automation-region-dns-records.md) dans le cadre de votre plan pour vous assurer que les fonctionnalités prises en charge fonctionnent sans problème.

### <a name="connect-to-a-private-endpoint"></a>Se connecter à un point de terminaison privé

Créez un point de terminaison privé pour vous connecter à notre réseau. Vous pouvez le faire dans le [Centre Private Link du portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Network/PrivateLinkCenterBlade/privateendpoints). Une fois vos modifications apportées à publicNetworkAccess et à Private Link, la prise en compte peut prendre jusqu’à 35 minutes.

Dans cette section, vous allez créer un point de terminaison privé pour votre compte Automation.

1. En haut à gauche de l’écran, sélectionnez **Créer une ressource > Mise en réseau > Centre de liaisons privées**.

2. Dans **Centre de liaisons privées - Vue d’ensemble**, dans l’option permettant de **Générer une connexion privée à un service**, sélectionnez **Démarrer**.

3. Dans **Créer une machine virtuelle - Notions de base**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **DÉTAILS DU PROJET** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **myResourceGroup**. Vous avez créé cela dans la section précédente.  |
    | **DÉTAILS DE L’INSTANCE** |  |
    | Nom | Entrez votre *PrivateEndpoint*. |
    | Région | Sélectionnez **YourRegion**. |
    |||

4. Sélectionnez **Suivant : Ressource**.

5. Dans **Créer un point de terminaison privé – Ressource**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    |Méthode de connexion  | Sélectionnez Se connecter à une ressource Azure dans mon répertoire.|
    | Abonnement| Sélectionnez votre abonnement. |
    | Type de ressource | Sélectionnez **Microsoft.Automation/automationAccounts**. |
    | Ressource |Sélectionnez *myAutomationAccount*|
    |Sous-ressource cible |Sélectionnez *Webhook* ou *DSCAndHybridWorker* en fonction de votre scénario.|
    |||

6. Sélectionnez **Suivant : Configuration**.

7. Dans **Créer un point de terminaison privé – Configuration**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    |**MISE EN RÉSEAU**| |
    | Réseau virtuel| Sélectionnez *MyVirtualNetwork*. |
    | Subnet | Sélectionnez *mySubnet*. |
    |**INTÉGRATION À DNS PRIVÉ**||
    |Intégrer à une zone DNS privée |Sélectionnez **Oui**. |
    |Zone DNS privée |Sélectionnez *(Nouveau)privatelink.azure-automation.net* |
    |||

8. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers la page **Vérifier + créer** où Azure valide votre configuration.

9. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

Dans le **Centre de liaisons privées**, sélectionnez **Points de terminaison privés** pour afficher toutes votre ressource de liaison privée.

![Liaison privée de la ressource Automation](./media/private-link-security/private-link-automation-resource.png)

Sélectionnez la ressource pour afficher tous les détails. Cela crée un nouveau point de terminaison privé pour votre compte Automation et lui attribue une adresse IP privée à partir de votre réseau virtuel. L’**état de la connexion** indique **Approuvé**.

De même, un nom de domaine complet (FQDN) unique est créé pour State Configuration (agentsvc) et pour le runtime de travail Runbook Worker hybride (jrds). Chacun se voit attribuer une adresse IP distincte de votre réseau virtuel et l’**état de la connexion** indique **Approuvé**.

Si le consommateur du service a des autorisations RBAC sur la ressource Automation, il peut choisir la méthode d’approbation automatique. Dans ce cas, lorsque la demande atteint la ressource du fournisseur Automation, celui-ci n’a rien à faire, car la connexion est automatiquement approuvée.

## <a name="set-public-network-access-flags"></a>Définir des indicateurs d’accès au réseau public

Cet article explique comment configurer un compte Automation pour refuser toutes les configurations publiques et n’autoriser que les connexions établies par le biais de points de terminaison privés afin d’améliorer la sécurité réseau. Si vous souhaitez limiter l’accès au compte Automation uniquement dans le réseau virtuel et ne pas autoriser l’accès à partir d’Internet public, vous pouvez définir le propriété `publicNetworkAccess` sur `$false`.

Quand la configuration de l’**Accès au réseau public** est définie sur `$false`, seules les connexions via des points de terminaison privés sont autorisées et toutes les connexions via les points de terminaison publics sont refusées avec un message d’erreur Non autorisé et un code d’état HTTP 401.

Le script PowerShell suivant montre comment `Get` et `Set` la propriété **Accès au réseau public** au niveau du compte Automation :

```powershell
$account = Get-AzResource -ResourceType Microsoft.Automation/automationAccounts -ResourceGroupName "<resourceGroupName>" -Name "<automationAccountName>" -ApiVersion "2020-01-13-preview"
$account.Properties | Add-Member -Name 'publicNetworkAccess' -Type NoteProperty -Value $false
$account | Set-AzResource -Force -ApiVersion "2020-01-13-preview"
```

Vous pouvez également contrôler la propriété d’accès au réseau public à partir du portail Azure. À partir de votre compte Automation, sélectionnez **Isolement réseau** dans le volet gauche sous la section **Paramètres de compte**. Quand le paramètre Accès au réseau public est défini sur **Non**, seules les connexions par des points de terminaison privés sont autorisées et toutes les connexions par des points de terminaison publics sont refusées.

![Paramètre Accès au réseau public](./media/private-link-security/allow-public-network-access.png)

## <a name="dns-configuration"></a>Configuration DNS

Lors de la connexion à une ressource Private Link à l’aide d’un nom de domaine complet (FQDN) dans la chaîne de connexion, il est important de configurer correctement vos paramètres DNS pour résoudre l’adresse IP privée allouée. Les services Azure existants peuvent déjà avoir une configuration DNS à utiliser lors de la connexion à un point de terminaison public. Votre configuration DNS doit être révisée et mise à jour pour vous connecter à l’aide de votre point de terminaison privé.

L’interface réseau associée à l’instance Private Endpoint contient l’ensemble complet des informations requises pour configurer votre DNS, y compris le nom de domaine complet et les adresses IP privées allouées pour une ressource Private Link donnée.

Vous pouvez utiliser les options suivantes pour configurer vos paramètres DNS pour Private Endpoint :

* Utilisez le fichier d’hôte (recommandé uniquement pour les tests). Vous pouvez utiliser le fichier d’hôte sur une machine virtuelle pour substituer l’utilisation du DNS par la résolution de noms en premier. Votre entrée DNS doit ressembler à l’exemple suivant : `privatelinkFQDN.jrds.sea.azure-automation.net`.

* Utilisez une [zone DNS privée](../../dns/private-dns-privatednszone.md). Vous pouvez utiliser des zones DNS privées pour substituer la résolution DNS par un point de terminaison privé donné. Une zone DNS privée peut être liée à votre réseau virtuel pour résoudre des domaines spécifiques. Pour permettre à l’agent sur votre machine virtuelle de communiquer sur le point de terminaison privé, créez un enregistrement DNS privé en tant que `privatelink.azure-automation.net`. Ajoutez un nouveau mappage d’enregistrements *A* DNS à l’adresse IP du point de terminaison privé.

* Utilisez votre redirecteur DNS (facultatif). Vous pouvez utiliser votre redirecteur DNS pour substituer la résolution DNS pour une ressource Private Link donnée. Si votre [serveur DNS](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) est hébergé sur un réseau virtuel, vous pouvez créer une règle de transfert DNS pour utiliser une zone DNS privée afin de simplifier la configuration de toutes les ressources Private Link.

Pour plus d’informations, consultez [Configuration DNS des points de terminaison privés Azure](../../private-link/private-endpoint-dns.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les points de terminaison privés, consultez [Qu’est-ce qu’Azure Private Endpoint ?](../../private-link/private-endpoint-overview.md).