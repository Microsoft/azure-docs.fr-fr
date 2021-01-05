---
title: Guide de résolution des problèmes d’Azure Security Center | Microsoft Docs
description: Ce guide s’adresse aux professionnels de l’informatique, analystes de la sécurité et administrateurs du cloud qui doivent détecter les problèmes liés à Azure Security Center.
services: security-center
author: v-miegge
manager: dcscontentpm
ms.service: security-center
ms.topic: conceptual
ms.date: 09/10/2019
ms.author: memildin
ms.openlocfilehash: 64b39dfa581b242fbb490d61b388f2bf260976ef
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460416"
---
# <a name="azure-security-center-troubleshooting-guide"></a>Guide de résolution des problèmes d’Azure Security Center

Ce guide s’adresse aux informaticiens professionnels, aux analystes de la sécurité des informations et aux administrateurs de cloud dont les entreprises utilisent Azure Security Center et qui doivent résoudre des problèmes liés à ce service.

Security Center utilise l'agent Log Analytics pour collecter et stocker les données. Pour plus d’informations, consultez l’article [Migration de plateforme Azure Security Center](./security-center-enable-data-collection.md). Les informations contenues dans cet article représentent la fonctionnalité Security Center après la transition vers l'agent Log Analytics.

## <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

Ce guide explique comment résoudre les problèmes liés au Centre de sécurité.

Types d’alertes :

* Analyse comportementale de la machine virtuelle (VMBA)
* Analyse du réseau
* SQL Database et analyse Azure Synapse Analytics
* Informations contextuelles

En fonction des types d’alertes, les clients peuvent collecter les informations nécessaires pour enquêter sur l’alerte à l’aide des ressources suivantes :

* Journaux de sécurité dans l’observateur d’événements de la machine virtuelle dans Windows
* AuditD dans Linux
* Les journaux d’activité Azure et l’activation des journaux de diagnostic sur la ressource de l’attaque.

Les clients peuvent partager des commentaires sur la description et la pertinence de l’alerte. Accédez à l’alerte, sélectionnez le bouton **Est-ce que ceci a été utile**, indiquez la raison et entrez un commentaire. Nous surveillons régulièrement ce canal de retour pour améliorer nos alertes.

## <a name="audit-log"></a>Journal d’audit

Vous pouvez résoudre la majorité des problèmes rencontrés dans Security Center, en commençant par examiner les enregistrements du [journal d’audit](../azure-monitor/platform/platform-logs-overview.md) pour le composant en échec. Les journaux d’audit vous permettent de déterminer :

* Les opérations ayant eu lieu.
* Qui a initié l’opération.
* Le moment où a eu lieu l’opération.
* L’état de l’opération.
* Les valeurs d’autres propriétés qui peuvent vous aider à effectuer des recherches sur l’opération.

Le journal d’audit contient toutes les opérations d’écriture (PUT, POST, DELETE) effectuées sur vos ressources, mais n’inclut pas les opérations de lecture (GET).

## <a name="log-analytics-agent"></a>Agent Log Analytics

Security Center utilise l'agent Log Analytics (le même agent que celui utilisé par le service Azure Monitor) pour collecter les données relatives à la sécurité sur vos machines virtuelles Azure. Une fois la collecte des données activée et l’agent installé correctement sur l’ordinateur cible, le processus suivant doit être en cours d’exécution :

* HealthService.exe

Si vous ouvrez la console de management des services (services.msc), vous verrez également l'agent Log Analytics s'exécuter comme illustré ci-dessous :

![Services](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig5.png)

Pour vérifier votre version de l'agent, ouvrez le **Gestionnaire des tâches** et, dans l'onglet **Processus**, localisez l'**agent Log Analytics**. Cliquez dessus avec le bouton droit de la souris, puis cliquez sur **Propriétés**. Dans l’onglet **Détails**, recherchez la version du fichier, comme indiqué ci-dessous :

![Fichier](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig6.png)

## <a name="log-analytics-agent-installation-scenarios"></a>Scénarios d'installation de l'agent Log Analytics

Il existe deux scénarios qui peuvent produire des résultats différents lors de l'installation de l'agent Log Analytics sur votre ordinateur. Les scénarios pris en charge sont les suivants :

* **Agent installé automatiquement par Security Center** : dans ce scénario, vous serez en mesure d’afficher les alertes dans Security Center et dans la recherche de journal. Vous recevrez des notifications par courrier électronique à l’adresse de messagerie qui a été configurée dans la stratégie de sécurité associée à l’abonnement auquel la ressource appartient.

* **Agent installé manuellement sur une machine virtuelle dans Azure** : dans ce scénario, si vous utilisez des agents téléchargés et installés manuellement avant février 2017, vous ne pouvez afficher les alertes dans le portail Security Center que si vous filtrez sur l’abonnement auquel appartient l’espace de travail. Si vous filtrez sur l’abonnement auquel appartient la ressource, vous ne voyez pas les alertes. Vous recevrez des notifications par courrier électronique à l’adresse de messagerie qui a été configurée dans la stratégie de sécurité associée à l’abonnement auquel l’espace de travail appartient.

> [!NOTE]
> Pour éviter le comportement expliqué dans le second scénario, veillez à télécharger la dernière version de l’agent.

## <a name="monitoring-agent-health-issues"></a>Problèmes d’intégrité de l’agent de surveillance <a name="mon-agent"></a>

**L’état de surveillance** définit la raison pour laquelle Security Center ne peut pas surveiller correctement les machines virtuelles et ordinateurs initialisées pour l’approvisionnement automatique. Le tableau suivant présente les valeurs, descriptions et étapes de résolution de **l’état de surveillance**.

| État de surveillance | Description | Étapes de résolution |
|---|---|---|
| Installation de l’agent en attente | L'installation de l'agent Log Analytics est toujours en cours d'exécution.  Cette installation peut prendre plusieurs heures. | Attendez que l’installation automatique soit terminée. |
| État d’alimentation hors tension | La machine virtuelle est arrêtée.  L'agent Log Analytics ne peut être installé que sur une machine virtuelle en cours d'exécution. | Redémarrez la machine virtuelle. |
| Agent de machine virtuelle Azure manquant ou non valide | L'agent Log Analytics n'est pas encore installé.  Un agent de machine virtuelle Azure valide est requis pour que Security Center installe l’extension. | Installer, réinstaller ou mettre à niveau l’agent de machine virtuelle Azure sur la machine virtuelle. |
| État de la machine virtuelle non prêt pour l’installation  | L'agent Log Analytics n'est pas encore installé car la machine virtuelle n'est pas prête pour l'installation. La machine virtuelle n’est pas prête pour l’installation en raison d’un problème avec l’agent de machine virtuelle ou l’approvisionnement de la machine virtuelle. | Vérifiez l’état de votre machine virtuelle. Revenez sur **Machines virtuelles** dans le portail, puis sélectionnez la machine virtuelle pour les informations sur l’état. |
|Échec de l’installation - erreur générale | L'agent Log Analytics a été installé, mais a échoué en raison d'une erreur. | [Installez manuellement l’extension](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension) ou désinstallez-la pour que Security Center tente de l’installer à nouveau. |
| Échec de l'installation : agent local déjà installé | L'installation de l'agent Log Analytics a échoué. Security Center a identifié un agent local (Log Analytics ou System Center Operations Manager) déjà installé sur la machine virtuelle. Pour éviter une configuration multihébergement, où la machine virtuelle fait un rapport à deux espaces de travail distincts, l'installation de l'agent Log Analytics est arrêtée. | Il existe deux manières de résoudre ce problème : [installer manuellement l’extension](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension) et la connecter à l’espace de travail souhaité. Ou, définir l’espace de travail souhaité comme espace de travail par défaut et activer l’approvisionnement automatique de l’agent.  Consultez la section [Activer l’approvisionnement automatique](security-center-enable-data-collection.md). |
| L’agent ne peut pas se connecter à l’espace de travail | L'agent Log Analytics a été installé, mais a échoué suite à un problème de connectivité réseau.  Vérifiez que l’agent dispose d’un accès Internet ou qu’un proxy HTTP valide lui a été défini. | Consultez la section Configuration réseau requise pour Monitoring Agent. |
| Agent connecté à un espace de travail manquant ou inconnu | Security Center a identifié que l'agent Log Analytics installé sur la machine virtuelle est connecté à un espace de travail auquel il n'a pas accès. | Cela peut se produire dans deux cas. L’espace de travail a été supprimé et n’existe plus. Réinstallez l’agent avec le bon espace de travail ou désinstallez l’agent et autorisez Security Center à terminer l’installation de l’approvisionnement automatique. Dans le deuxième cas, l’espace de travail fait partie d’un abonnement pour lequel Security Center ne possède pas d’autorisation. Security Center requiert des abonnements pour permettre au fournisseur de ressources de Microsoft Security d’y accéder. Pour l’activer, enregistrez l’abonnement dans le fournisseur de ressources de Microsoft Security. Cela peut être réalisé à l’aide d’une API, PowerShell, d’un portail ou simplement en filtrant l’abonnement dans le tableau de bord **Vue d’ensemble** de Security Center. Pour plus d’informations, consultez [Fournisseurs et types de ressources](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal). |
| L’agent ne répond pas ou l’ID est manquant | Security Center ne peut pas récupérer les données de sécurité analysées de la machine virtuelle, même si l’agent est installé. | L’agent ne rapporte aucune donnée, y compris les pulsations. L’agent peut être endommagé ou quelque chose bloque le trafic. Ou, l’agent rapporte des données mais il manque un ID de ressource Azure. Il est donc impossible de faire correspondre les données à la machine virtuelle Azure. Pour résoudre les problèmes associés à Linux, consultez [Troubleshooting Guide for OMS Agent for Linux](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/Troubleshooting.md#im-not-seeing-any-linux-data-in-the-oms-portal) (Guide de résolution des problèmes pour Agent OMS pour Linux). Pour résoudre les problèmes associés à Windows, consultez [Troubleshooting Windows Virtual Machines](../virtual-machines/extensions/oms-windows.md#troubleshoot-and-support) (Résolution des problèmes de machines virtuelles Windows). |
| Agent non installé | La collecte de données est désactivée. | Activez la collecte de données dans la stratégie de sécurité ou installez manuellement l'agent Log Analytics. |

## <a name="troubleshooting-monitoring-agent-network-requirements"></a>Résolution des problèmes de configuration réseau requise de l’agent de surveillance <a name="mon-network-req"></a>

Pour que les agents se connectent et s’inscrivent auprès de Security Center, ils doivent accéder aux ressources réseau, y compris les numéros de port et les URL de domaine.

* Pour les serveurs proxy, vous devez vous assurer que les ressources de serveur proxy appropriées sont configurées dans les paramètres de l’agent. Lisez cet article pour obtenir plus d’informations sur la façon de [modifier les paramètres proxy](../azure-monitor/platform/agent-windows.md).
* Si un pare-feu restreint l’accès à Internet, vous devez configurer votre pare-feu pour autoriser l’accès à Log Analytics. Aucune action n’est nécessaire dans les paramètres de l’agent.

Le tableau suivant présente les ressources nécessaires pour la communication.

| Ressource de l'agent | Ports | Ignorer l’inspection HTTPS |
|---|---|---|
| *.ods.opinsights.azure.com | 443 | Oui |
| *.oms.opinsights.azure.com | 443 | Oui |
| *.blob.core.windows.net | 443 | Oui |
| \* .azure-automation.net | 443 | Oui |

Si vous rencontrez des problèmes d’intégration avec l’agent, lisez l’article [Comment faire pour résoudre les problèmes d’intégration de Microsoft Operations Management Suite](https://support.microsoft.com/help/3126513/how-to-troubleshoot-operations-management-suite-onboarding-issues).

## <a name="troubleshooting-endpoint-protection-not-working-properly"></a>Dépannage d’une protection du point de terminaison qui ne fonctionne pas correctement

L’agent invité représente le processus parent de toutes les actions effectuées par l’extension [Microsoft Antimalware](../security/fundamentals/antimalware.md). Lorsque le processus de l’agent invité échoue, le programme Microsoft Antimalware exécuté comme un processus enfant de l’agent invité peut également échouer.  Dans les scénarios recommandés pour vérifier les options suivantes :

* Si la machine virtuelle cible est une image personnalisée et que le créateur de la machine virtuelle n’a jamais installé l’agent invité.
* Si la cible est une machine virtuelle Linux au lieu d’une machine virtuelle Windows, l’installation de la version Windows de l’extension anti-programme malveillant sur une machine virtuelle Linux échoue. L’agent invité Linux a des exigences spécifiques en termes de version du système d’exploitation et de packages requis, et si ces conditions ne sont pas remplies, l’agent de la machine virtuelle ne fonctionnera pas ici non plus.
* Si la machine virtuelle a été créée avec une ancienne version de l’agent invité. Dans ce cas, notez que certains anciens agents ne pourront pas se mettre automatiquement à jour avec la version la plus récente, et cela peut entraîner ce problème. Utilisez toujours la dernière version de l’agent invité si vous créez vos propres images.
* Certains logiciels d’administration tiers peuvent désactiver l’agent invité ou bloquer l’accès à certains emplacements de fichiers. Si vous avez installé un logiciel tiers sur votre machine virtuelle, assurez-vous que l’agent figure sur la liste d’exclusion.
* Certains paramètres de pare-feu ou d’un groupe de sécurité réseau (NSG) peuvent bloquer le trafic réseau vers et depuis l’agent invité.
* Certaines listes de contrôle d’accès (ACL) peuvent empêcher l’accès au disque.
* Un espace disque insuffisant peut empêcher l’agent invité de fonctionner correctement.

Par défaut, l’interface utilisateur de Microsoft Antimalware est désactivée. Consultez l’article [Activation de l’interface utilisateur Microsoft Antimalware sur des machines virtuelles Azure Resource Manager après le déploiement](/archive/blogs/azuresecurity/enabling-microsoft-antimalware-user-interface-post-deployment) pour plus d’informations sur l’activation de l’interface utilisateur, le cas échéant.

## <a name="troubleshooting-problems-loading-the-dashboard"></a>Résolution des problèmes de chargement du tableau de bord

Si vous rencontrez des problèmes de chargement du tableau de bord de Security Center, vérifiez que l’utilisateur qui inscrit l’abonnement auprès de Security Center (c’est-à-dire le premier utilisateur qui a ouvert Security Center avec l’abonnement) et l’utilisateur qui souhaite activer la collecte des données ont le rôle *Propriétaire* ou *Contributeur* pour l’abonnement. À partir de ce moment, les utilisateurs ayant le rôle *Lecteur* pour l’abonnement peuvent également consulter le tableau de bord, les alertes, les recommandations et les stratégies.

## <a name="contacting-microsoft-support"></a>Contacter le support Microsoft

Vous pouvez identifier certains problèmes en suivant les recommandations fournies dans cet article, tandis que d’autres problèmes sont documentés dans la [page Microsoft Q&A](/answers/topics/azure-security-center.html) publique de Security Center. Toutefois, si vous avez encore besoin d’aide pour résoudre votre problème, vous pouvez ouvrir une nouvelle demande de support à l’aide du **portail Azure**, comme indiqué ci-dessous :

![Support Microsoft](./media/security-center-troubleshooting-guide/security-center-troubleshooting-guide-fig2.png)

## <a name="see-also"></a>Voir aussi

Dans ce document, vous avez appris à configurer des stratégies de sécurité dans le Centre de sécurité Azure. Pour plus d’informations sur le Centre de sécurité Azure, consultez les rubriques suivantes :

* [Guide des opérations et de planification d’Azure Security Center](security-center-planning-and-operations-guide.md) : découvrez comment planifier l’adoption d’Azure Security Center et prenez connaissance des considérations relatives à la conception.
* [Surveillance de l’intégrité de la sécurité dans Azure Security Center](security-center-monitoring.md) : découvrez comment surveiller l’intégrité de vos ressources Azure.
* [Gestion et résolution des alertes de sécurité dans Azure Security Center](security-center-managing-and-responding-alerts.md) : découvrez comment gérer et résoudre les alertes de sécurité.
* [Présentation des alertes de sécurité dans Azure Security Center](./security-center-alerts-overview.md)
* [Tutoriel : Répondre à des incidents de sécurité](tutorial-security-incident.md)
* [Validation des alertes dans Azure Security Center](security-center-alert-validation.md)
* [Notifications par e-mail dans Azure Security Center](security-center-provide-security-contact-details.md)
* [Gestion des incidents de sécurité dans Azure Security Center](security-center-incident.md)
* [Fonctionnalités de détection d’Azure Security Center](./security-center-alerts-overview.md)
* [Surveillance des solutions de partenaire avec Azure Security Center](./security-center-partner-integration.md) : découvrez comment surveiller l’état d’intégrité de vos solutions de partenaire.
* [FAQ d’Azure Security Center](faq-general.md) : découvrez les réponses aux questions les plus souvent posées à propos de l’utilisation de ce service
* [Blog sur la sécurité Azure](/archive/blogs/azuresecurity/) : accédez à des billets de blog sur la sécurité et la conformité Azure.