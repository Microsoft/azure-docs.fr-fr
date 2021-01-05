---
title: Modes et exigences de connectivité
description: Décrit en détail les options de connectivité des services de données Azure Arc depuis votre environnement vers Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: conceptual
ms.openlocfilehash: c7bff21a17af3c908caeed6a1e60de8e2fe4efc9
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287582"
---
# <a name="connectivity-modes-and-requirements"></a>Modes et exigences de connectivité

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="connectivity-modes"></a>Modes de connectivité

Il existe plusieurs options pour le degré de connectivité entre votre environnement de services de données compatible avec Azure Arc vers Azure. À mesure que vos besoins varient en fonction de la stratégie d’entreprise, du règlement gouvernemental ou de la disponibilité de la connectivité réseau à Azure, vous pouvez choisir parmi les modes de connectivité suivants.

Les services de données Azure Arc activés vous permettent de vous connecter à Azure dans deux *modes de connectivité* différents : 

- Connecté directement 
- Connecté indirectement

Ce mode de connectivité vous offre la possibilité de choisir la quantité de données envoyées à Azure et la manière dont les utilisateurs interagissent avec le contrôleur de données Arc. Selon le mode de connectivité choisi, certaines fonctionnalités des services de données compatibles avec Azure Arc peuvent être disponibles ou non.

Notez que, si les services de données compatibles avec Azure Arc sont directement connectés à Azure, les utilisateurs peuvent utiliser des [API Azure Resource Manager](/rest/api/resources/), l’interface de ligne de commande Azure et le portail Azure pour utiliser les services de données Azure Arc. L’expérience en mode directement connecté est très similaire à l’utilisation d’un autre service Azure avec l’approvisionnement, le désapprovisionnement, la mise à l’échelle, la configuration, etc. dans le Portail Azure.  Si les services de données compatibles avec Azure Arc sont indirectement connectés à Azure, le Portail Azure est un affichage en lecture seule. Vous pouvez afficher l’inventaire des instances managées SQL et des instances Postgres Hyperscale que vous avez déployées, ainsi que les détails les concernant, mais vous ne pouvez pas effectuer d’action sur ceux-ci dans le Portail Azure.  En mode connecté indirectement, toutes les actions doivent être effectuées localement à l’aide d’Azure Data Studio, de l’outil [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] ou d’outils natifs Kubernetes comme kubectl.

En outre, Azure Active Directory et le contrôle d’accès en fonction du rôle Azure peuvent être utilisés en mode connecté directement, car il existe une dépendance d’une connexion continue et directe à Azure pour fournir cette fonctionnalité.

Certains services attachés à Azure sont disponibles uniquement lorsqu’ils peuvent être directement accessibles, tels que les services de sécurité Azure Defender, les Insights de conteneur et la Sauvegarde Azure sur stockage blob.

Actuellement, dans l’aperçu, seul le mode connecté indirectement est pris en charge. 

||**Connecté indirectement**|**Connecté directement**|**Jamais connecté**|
|---|---|---|---|
|**Description**|Le mode connecté indirectement offre la plupart des services de gestion localement dans votre environnement, sans connexion directe à Azure.  Une quantité minimale de données doit être envoyée à Azure à des fins d’inventaire et de facturation _uniquement_. Elle est exportée vers un fichier et chargée vers Azure au moins une fois par mois.  Aucune connexion directe ou continue à Azure n’est requise.  Certains services et fonctionnalités qui nécessitent une connexion à Azure ne seront pas disponibles.|Le mode connecté directement offre tous les services disponibles quand une connexion directe peut être établie avec Azure. Les connexions sont toujours lancées _depuis_ votre environnement vers Azure et utilisent des ports standard et des protocoles tels que HTTPS/443.|Aucune donnée ne peut être envoyée vers ou à partir d’Azure.|
|**Disponibilité actuelle**| disponible en préversion.|Prévue pour l’aperçu à l’avenir.|Actuellement non pris en charge.|
|**Études de cas classiques**|Centres de données locaux qui n’autorisent pas la connectivité dans ou à l’extérieur de la région de données du centre de données en raison de stratégies de conformité commerciales ou réglementaires ou de problèmes d’attaques externes ou d’exfiltration de données.  Exemples types : Institutions financières, soins médicaux, secteur public. <br/><br/>Sites de périmètre où le site de périphérie ne dispose généralement pas d’une connectivité à Internet.  Exemples typiques : applications de terrain, pétrolières, gazières ou militaires.  <br/><br/>Emplacements de site de périphérie disposant d’une connectivité intermittente avec de longues périodes de panne.  Exemples typiques : stades, bateaux de croisière. | Organisations qui utilisent des clouds publics.  Exemples types : Azure, AWS ou Google Cloud.<br/><br/>Emplacements de sites de périphérie où la connectivité Internet est généralement présente et autorisée.  Exemples typiques : magasins de vente au détail, fabrication.<br/><br/>Centres de données d’entreprise avec des stratégies plus permissives pour la connectivité vers Internet et depuis leur région de données du centre de données.  Exemples types : Entreprises non réglementées, petites et moyennes entreprises|Environnements parfaitement hermétiques dans lesquels aucune donnée n’est susceptible de se trouver dans l’environnement de données. Exemples typiques : principales fonctionnalités gouvernementales des clés secrètes.|
|**Comment les données sont envoyées à Azure**|Il existe trois options pour l’envoi des données de facturation et d’inventaire vers Azure :<br><br> 1) Les données sont exportées hors de la région de données par un processus automatisé qui est connecté à la fois à la région de données sécurisée et à Azure.<br><br>2) Les données sont exportées hors de la région de données par un processus automatisé dans la région de données, copiées automatiquement vers une région moins sécurisée, et un processus automatisé dans la région moins sécurisée charge les données vers Azure.<br><br>3) Les données sont exportées manuellement par un utilisateur dans la région sécurisée, extraites manuellement de la région sécurisée et chargées manuellement vers Azure. <br><br>Les deux premières options sont un processus continu automatisé qui peut être planifié pour s’exécuter fréquemment, de sorte qu’il y a un délai minimal dans le transfert de données vers Azure, qui est uniquement soumis à la connectivité disponible à Azure.|Les données sont envoyées automatiquement et en continu à Azure.|Les données ne sont jamais envoyées à Azure.|

## <a name="feature-availability-by-connectivity-mode"></a>Disponibilité des fonctionnalités par mode de connectivité

|**Fonctionnalité**|**Connecté indirectement**|**Connecté directement**|
|---|---|---|
|**Haute disponibilité automatique**|Prise en charge|Prise en charge|
|**Approvisionnement en libre-service**|Prise en charge<br/>La création peut être effectuée par le biais d’Azure Data Studio, de l’outil [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] ou d’outils natifs Kubernetes (Helm, kubectl, OC, etc.), ou encore à l’aide d’une approvisionnement Kubernetes GitOps activé par Azure Arc.|Prise en charge<br/>Outre les options de création en mode connecté indirectement, vous pouvez créer via le portail Azure, des API Azure Resource Manager, Azure CLI ou des modèles Resource Manager. **Disponibilité en attente du mode connecté directement**
|**Extensibilité élastique**|Prise en charge|Prise en charge<br/>**Disponibilité en attente du mode connecté directement**|
|**Billing**|Prise en charge<br/>Les données de facturation sont régulièrement exportées et envoyées à Azure.|Prise en charge<br/>Les données de facturation sont envoyées automatiquement et en permanence à Azure, et elles sont reflétées en temps quasi réel. **Disponibilité en attente du mode connecté directement**|
|**Gestion des stocks**|Prise en charge<br/>Les données d’inventaire sont régulièrement exportées et envoyées à Azure.<br/><br/>Utilisez des outils clients comme Azure Data Studio, Azure Data CLI ou `kubectl` pour afficher et gérer l’inventaire localement.|Prise en charge<br/>Les données d’inventaire sont envoyées automatiquement et en permanence à Azure, et elles sont reflétées en temps quasi réel. Vous pouvez ainsi gérer l’inventaire directement à partir du portail Azure. **Disponibilité en attente du mode connecté directement**|
|**Mises à niveau et mises à jour correctives automatiques**|Prise en charge<br/>Le contrôleur de données doit avoir un accès direct à Microsoft Container Registry (MCR) ou les images de conteneur doivent être extraites de la valeur de la clé et transmises à un registre de conteneurs privé local auquel le contrôleur de données a accès.|Prise en charge<br/>**Disponibilité en attente du mode connecté directement**|
|**Sauvegarde et restauration automatiques**|Prise en charge<br/>Sauvegarde et restauration locales automatiques.|Prise en charge<br/>En plus de la sauvegarde et de la restauration locales automatisées, vous pouvez _éventuellement_ envoyer des sauvegardes à Sauvegarde Azure pour une conservation des données hors site à long terme. **Disponibilité en attente du mode connecté directement**|
|**Monitoring**|Prise en charge<br/>Analyse locale à l’aide des tableaux de bord Grafana et Kibana.|Prise en charge<br/>Outre les tableaux de bord de surveillance locaux, vous pouvez _éventuellement_ envoyer des données et des journaux de surveillance à Azure Monitor pour une surveillance à l’échelle de plusieurs sites dans un même emplacement. **Disponibilité en attente du mode connecté directement**|
|**Authentification**|Utilisez le nom d’utilisateur/mot de passe local pour l’authentification du contrôleur de données et du tableau de bord. Utilisez les connexions SQL et Postgres ou Active Directory pour la connectivité aux instances de base de données.  Utilisez les fournisseurs d’authentification K8s pour l’authentification auprès de l’API Kubernetes.|En plus ou à la place des méthodes d’authentification pour le mode connecté indirectement, vous pouvez _éventuellement_ utiliser Azure Active Directory. **Disponibilité en attente du mode connecté directement**|
|**Contrôle d’accès en fonction du rôle (RBAC)**|Utilisez le contrôle d’accès en fonction du rôle Kubernetes sur l’API Kubernetes. Utilisez le contrôle d’accès en fonction du rôle SQL et Postgres pour les instances de base de données.|Vous pouvez éventuellement intégrer Azure Active Directory et Azure RBAC. **Disponibilité en attente du mode connecté directement**|
|**Azure Defender**|Non pris en charge|Sortie planifiée|

## <a name="connectivity-requirements"></a>Connectivité requise

**Certaines fonctionnalités nécessitent une connexion à Azure.**

**Toutes les communications avec Azure sont toujours lancées à partir de votre environnement.** Cela est vrai même pour les opérations, qui sont initiées par un utilisateur dans le Portail Azure.  Dans ce cas, il y a effectivement une tâche, qui est mise en file d’attente dans Azure.  Un agent de votre environnement lance la communication avec Azure pour voir quelles tâches sont dans la file d’attente, exécute les tâches et renvoie l’état/l’achèvement/l’échec à Azure.

|**Type de données**|**Sens**|**Obligatoire/facultatif**|**Coûts supplémentaires**|**Mode requis**|**Notes**|
|---|---|---|---|---|---|
|**Images conteneur**|Microsoft Container Registry -> Client|Obligatoire|Non|Indirect ou direct|Les images conteneur sont la méthode de distribution du logiciel.  Dans un environnement qui peut se connecter à Microsoft Container Registry (MCR) par le biais d’Internet, les images de conteneur peuvent être extraites directement à partir de la valeur de la valeur.  Dans le cas où l’environnement de déploiement ne dispose pas d’une connectivité directe, vous pouvez extraire les images de la clé et les envoyer vers un registre de conteneurs privé dans l’environnement de déploiement.  Au moment de la création, vous pouvez configurer le processus de création pour effectuer l’extraction à partir du registre de conteneurs privé au lieu de MCR. Cela s’applique également aux mises à jour automatisées.|
|**Inventaire des ressources**|Environnement client-> Azure|Obligatoire|Non|Indirect ou direct|Un inventaire des contrôleurs de données, des instances de base de données (PostgreSQL et SQL) est conservé dans Azure à des fins de facturation, ainsi qu’à des fins de création d’un inventaire de tous les contrôleurs de données et d’instances de base de données dans un seul emplacement, ce qui est particulièrement utile si vous avez plusieurs environnements avec des services de données Azure Arc.  À mesure que les instances sont approvisionnées, annulées, mises à l’échelle (scale-up/scale-down), l’inventaire est mis à jour dans Azure.|
|**Données de télémétrie de facturation**|Environnement client-> Azure|Obligatoire|Non|Indirect ou direct|L’utilisation des instances de base de données doit être envoyée à Azure à des fins de facturation.  Il n’y a aucun coût pour les services de données compatibles avec Azure Arc pendant la période de la version préliminaire.|
|**Analyse des données et des journaux**|Environnement client-> Azure|Facultatif|Peut-être en fonction du volume de données (voir [Tarification Azure Monitor](https://azure.microsoft.com/en-us/pricing/details/monitor/))|Indirect ou direct|Vous pouvez envoyer les données de surveillance et les journaux collectés localement à Azure Monitor pour agréger des données dans plusieurs environnements dans un même emplacement et pour utiliser des services Azure Monitor comme des alertes, à l’aide des données de Azure Machine Learning, etc.|
|**Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**|Environnement client-> Azure -> environnement client|Facultatif|Non|Directe uniquement|Si vous souhaitez utiliser Azure RBAC, la connectivité doit être établie avec Azure à tout moment.  Si vous ne souhaitez pas utiliser Azure RBAC, vous pouvez utiliser le contrôle d’accès en fonction du rôle Kubernetes local.  **Disponibilité en attente du mode connecté directement**|
|**Azure Active Directory (AD)**|Environnement client-> Azure -> environnement client|Facultatif|Peut-être, mais vous payez peut-être déjà pour Azure AD|Directe uniquement|Si vous souhaitez utiliser Azure AD pour l’authentification, la connectivité doit être établie avec Azure à tout moment. Si vous ne souhaitez pas utiliser Azure AD pour l’authentification, vous pouvez utiliser les services de fédération Active Directory (AD FS) sur Active Directory. **Disponibilité en attente du mode connecté directement**|
|**Sauvegarde et restauration**|Environnement client -> Environnement client|Obligatoire|Non|Direct ou indirect|Le service de sauvegarde et de restauration peut être configuré pour pointer vers des classes de stockage locales. |
|**Sauvegarde Azure – Rétention à long terme**| Environnement client-> Azure | Facultatif| Oui, pour Stockage Azure | Directe uniquement |Vous souhaiterez peut-être envoyer des sauvegardes effectuées localement à Sauvegarde Azure pour la rétention à long terme et hors site des sauvegardes, puis les remettre dans l’environnement local pour la restauration. **Disponibilité en attente du mode connecté directement**|
|**Services de sécurité Azure Defender**|Environnement client-> Azure -> environnement client|Facultatif|Oui|Directe uniquement|**Disponibilité en attente du mode connecté directement**|
|**Mise en service et modification de la configuration à partir du Portail Azure**|Environnement client-> Azure -> environnement client|Facultatif|Non|Directe uniquement|Les modifications d’approvisionnement et de configuration peuvent être effectuées localement à l’aide d’Azure Data Studio ou de l’outil [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)].  En mode directement connecté, vous serez également en mesure d’approvisionner et d’apporter des modifications de configuration à partir du portail Azure. **Disponibilité en attente du mode connecté directement**|


## <a name="details-on-internet-addresses-ports-encryption-and-proxy-server-support"></a>Détails sur les adresses Internet, les ports, le chiffrement et la prise en charge du serveur proxy

Actuellement, dans la phase de préversion, seul le mode connecté indirectement est pris en charge. Dans ce mode, seules trois connexions sont requises pour les services disponibles sur Internet. Ces connexions sont les suivantes :

- [Microsoft Container Registry (MCR)](#microsoft-container-registry-mcr)
- [API Azure Resource Manager](#azure-resource-manager-apis)
- [API Azure Monitor](#azure-monitor-apis)

Toutes les connexions HTTPS à Azure et à Microsoft Container Registry sont chiffrées à l’aide de SSL/TLS à l’aide de certificats officiellement signés et vérifiables.

Pour plus d’informations sur ces connexions, lisez les sections suivantes. 

### <a name="microsoft-container-registry-mcr"></a>Microsoft Container Registry (MCR)

Microsoft Container Registry héberge les images de conteneur des services de données compatibles avec Azure Arc.  Vous pouvez extraire ces images de MCR et les pousser vers un registre de conteneurs privé et configurer le processus de déploiement du contrôleur de données pour extraire les images des conteneurs de ce registre de conteneurs privé.

#### <a name="connection-source"></a>Source de connexion

Kubernetes kubelet sur chacun des nœuds Kubernetes extrayant les images de conteneur.

#### <a name="connection-target"></a>Cible de connexion

`mcr.microsoft.com`

#### <a name="protocol"></a>Protocol

HTTPS

#### <a name="port"></a>Port

443

#### <a name="can-use-proxy"></a>Peut utiliser le proxy

Oui

#### <a name="authentication"></a>Authentification

Aucun

### <a name="azure-resource-manager-apis"></a>API Azure Resource Manager
Azure Data Studio, [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] et Azure CLI se connectent aux API Azure Resource Manager pour envoyer et récupérer des données vers et depuis Azure pour certaines fonctionnalités.

#### <a name="connection-source"></a>Source de connexion

Ordinateur exécutant Azure Data Studio, [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] ou Azure CLI qui se connecte à Azure.

#### <a name="connection-target"></a>Cible de connexion

- `login.microsoftonline.com`
- `management.azure.com`
- `san-af-eastus-prod.azurewebsites.net`
- `san-af-eastus2-prod.azurewebsites.net`
- `san-af-australiaeast-prod.azurewebsites.net`
- `san-af-centralus-prod.azurewebsites.net`
- `san-af-westus2-prod.azurewebsites.net`
- `san-af-westeurope-prod.azurewebsites.net`
- `san-af-southeastasia-prod.azurewebsites.net`
- `san-af-koreacentral-prod.azurewebsites.net`
- `san-af-northeurope-prod.azurewebsites.net`
- `san-af-westeurope-prod.azurewebsites.net`
- `san-af-uksouth-prod.azurewebsites.net`
- `san-af-francecentral-prod.azurewebsites.net`

#### <a name="protocol"></a>Protocol

HTTPS

#### <a name="port"></a>Port

443

#### <a name="can-use-proxy"></a>Peut utiliser le proxy

Oui

#### <a name="authentication"></a>Authentification 

Azure Active Directory

### <a name="azure-monitor-apis"></a>API Azure Monitor

Azure Data Studio, [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] et Azure CLI se connectent aux API Azure Resource Manager pour envoyer et récupérer des données vers et depuis Azure pour certaines fonctionnalités.

#### <a name="connection-source"></a>Source de connexion

Un ordinateur exécutant [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] ou Azure CLI qui charge les métriques d’analyse ou les journaux vers Azure Monitor.

#### <a name="connection-target"></a>Cible de connexion

- `login.microsoftonline.com`
- `management.azure.com`
- `*.ods.opinsights.azure.com`
- `*.oms.opinsights.azure.com`
- `*.monitoring.azure.com`

#### <a name="protocol"></a>Protocol

HTTPS

#### <a name="port"></a>Port

443

#### <a name="can-use-proxy"></a>Peut utiliser le proxy

Oui

#### <a name="authentication"></a>Authentification 

Azure Active Directory

> [!NOTE]
> Pour le moment, toutes les connexions HTTPS/443 du navigateur aux tableaux de bord Grafana et Kibana et de l’outil [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] à l’API du contrôleur de données sont chiffrées par SSL à l’aide de certificats auto-signés.  À l’avenir, une fonctionnalité vous permettra de fournir vos propres certificats pour le chiffrement de ces connexions SSL.

La connectivité entre Azure Data Studio et [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] d’un côté et le serveur d’API Kubernetes de l’autre utilise l’authentification et le chiffrement Kubernetes que vous avez établis.  Chaque utilisateur qui utilise Azure Data Studio et [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] doit disposer d’une connexion authentifiée à l’API Kubernetes pour effectuer la plupart des actions liées aux services de données Azure Arc.

