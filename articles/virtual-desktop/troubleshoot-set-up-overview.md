---
title: 'Vue d’ensemble du dépannage de Windows Virtual Desktop : Azure'
description: Vue d’ensemble de la résolution des problèmes lors de la configuration d’un environnement Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 12/04/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: d051594572f742b290867d12e0fb148f2abbf370
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621312"
---
# <a name="troubleshooting-overview-feedback-and-support-for-windows-virtual-desktop"></a>Vue d’ensemble de la résolution des problèmes, commentaires et support pour Windows Virtual Desktop

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/troubleshoot-set-up-overview-2019.md).

Cet article fournit une vue d’ensemble des problèmes que vous pouvez rencontrer lors de la configuration d’un environnement Windows Virtual Desktop et fournit des moyens de résoudre les problèmes.

## <a name="report-issues"></a>Signaler des problèmes

Pour signaler des problèmes ou suggérer des fonctionnalités pour Windows Virtual Desktop avec intégration d’Azure Resource Manager, consultez la [Communauté technique de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop). Vous pouvez utiliser la Communauté technique pour discuter des bonnes pratiques ou bien pour suggérer et soutenir de nouvelles fonctionnalités.

Quand vous demandez de l’aide ou que vous proposez une nouvelle fonctionnalité, veillez à décrire votre sujet aussi précisément que possible. Des informations détaillées peuvent aider les autres utilisateurs à répondre à votre question ou à comprendre la fonctionnalité pour laquelle vous proposez un vote.

## <a name="escalation-tracks"></a>Chemins de réaffectation

Avant de passer à autre chose, veillez à consulter la [page des états Azure](https://status.azure.com/status) et [Azure Service Health](https://azure.microsoft.com/features/service-health/) pour vérifier que votre service Azure fonctionne correctement.

Référez-vous au tableau suivant pour identifier et résoudre les problèmes que vous pouvez rencontrer quand vous configurez un environnement avec le client Bureau à distance. Une fois votre environnement configuré, vous pouvez utiliser notre nouveau [service Diagnostics](diagnostics-role-service.md) pour identifier les problèmes pour les scénarios courants.

| **Problème**                                                            | **Solution suggérée**  |
|----------------------------------------------------------------------|-------------------------------------------------|
| Paramètres de Réseau virtuel Azure et d’ExpressRoute des pools d’hôtes de session               | [Ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/), puis sélectionnez le service approprié (sous la catégorie Réseaux). |
| Création d’une machine virtuelle de pool d’hôtes quand les modèles Azure Resource Manager fournis avec Windows Virtual Desktop ne sont pas utilisés | [Ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/), puis sélectionnez **Windows Virtual Desktop** comme service. <br> <br> Pour les problèmes avec les modèles Azure Resource Manager fournis avec Windows Virtual Desktop, consultez la section Erreurs liées au modèle Azure Resource Manager dans [Création d’un pool d’hôtes](troubleshoot-set-up-issues.md). |
| Gestion des environnements d’hôtes de session Windows Virtual Desktop à partir du portail Azure    | [Ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/). <br> <br> Pour les problèmes de gestion survenant quand vous utilisez les services Bureau à distance/Windows Virtual Desktop PowerShell, consultez [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md) ou [ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/), sélectionnez **Windows Virtual Desktop** comme service, sélectionnez **Configuration et gestion** comme type de problème, puis sélectionnez **Problèmes avec la configuration de l’environnement à l’aide de PowerShell** comme sous-type de problème. |
| Gestion de la configuration de Windows Virtual Desktop liée à des pools d’hôtes et à des groupes d’applications      | Consultez [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md) ou [ouvrez une de demande de support Azure](https://azure.microsoft.com/support/create-ticket/), sélectionnez **Windows Virtual Desktop** pour le service, puis sélectionnez le type de problème approprié.|
| Déploiement et gestion de conteneurs de profils FSLogix | Consultez le [Guide de résolution des problèmes relatifs aux produits FSLogix](/fslogix/fslogix-trouble-shooting-ht/). Si ce guide ne vous permet pas de résoudre le problème, [ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/), sélectionnez **Windows Virtual Desktop** pour le service, sélectionnez **FSLogix** pour le type de problème, puis sélectionnez le sous-type de problème approprié. |
| Dysfonctionnement des clients Bureau à distance au démarrage                                                 | Consultez [Résoudre des problèmes du client Bureau à distance](troubleshoot-client.md). Si vous ne parvenez toujours pas à résoudre le problème, [ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/), sélectionnez **Windows Virtual Desktop** pour le service, puis sélectionnez **Clients Bureau à distance** pour le type de problème.  <br> <br> S’il s’agit d’un problème réseau, vos utilisateurs doivent contacter leur administrateur réseau. |
| Connecté mais pas de flux                                                                 | Résolvez les problèmes en suivant la section [Lorsqu’un utilisateur se connecte, rien ne s’affiche (aucun flux)](troubleshoot-service-connection.md#user-connects-but-nothing-is-displayed-no-feed) de l’article [Connexions au service Windows Virtual Desktop](troubleshoot-service-connection.md). <br> <br> Si vos utilisateurs ont été attribués à un groupe d’applications, [ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/), sélectionnez **Windows Virtual Desktop** pour le service, puis sélectionnez **Clients Bureau à distance** pour le type de problème. |
| Problèmes de découverte de flux en raison du réseau                                            | Vos utilisateurs doivent contacter leur administrateur réseau. |
| Connexion des clients                                                                    | Consultez [Connexions au service Windows Virtual Desktop](troubleshoot-service-connection.md) et, si cela ne résout pas votre problème, consultez [Configuration d’une machine virtuelle hôte de sessions](troubleshoot-vm-configuration.md). |
| Réactivité des applications ou du Bureau à distance                                      | Si les problèmes sont liés à une application ou un produit spécifique, contactez l’équipe responsable de ce produit. |
| Messages ou erreurs concernant les licences                                                          | Si les problèmes sont liés à une application ou un produit spécifique, contactez l’équipe responsable de ce produit. |
| Problèmes liés aux méthodes d’authentification ou aux outils de tiers | Vérifiez que votre fournisseur tiers prend en charge les scénarios Windows Virtual Desktop en tenant compte des problèmes connus. |
| Problèmes lors de l’utilisation de Log Analytics pour Windows Virtual Desktop | Pour les problèmes liés au schéma de diagnostics, [ouvrez une demande de support Azure](https://azure.microsoft.com/support/create-ticket/).<br><br>Pour les requêtes, la visualisation ou d’autres problèmes dans Log Analytics, sélectionnez le type de problème approprié sous Log Analytics. |
| Problèmes liés à l’utilisation des applications M365 | Contactez le centre d’administration M365 grâce à l’une des [options d’aide du centre d’administration M365](/microsoft-365/admin/contact-support-for-business-products/). |

## <a name="next-steps"></a>Étapes suivantes

- Pour résoudre les problèmes de création d’un pool d’hôtes dans un environnement Windows Virtual Desktop, consultez [Création d’un pool d’hôtes](troubleshoot-set-up-issues.md).
- Pour résoudre les problèmes de configuration d’une machine virtuelle dans Windows Virtual Desktop, consultez [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration.md).
- Pour résoudre les problèmes de connexion au client Windows Virtual Desktop, consultez [Connexions au service Windows Virtual Desktop](troubleshoot-service-connection.md).
- Pour résoudre les problèmes liés aux clients Bureau à distance, consultez [Résoudre des problèmes du client Bureau à distance](troubleshoot-client.md).
- Pour résoudre les problèmes d’utilisation de PowerShell avec Windows Virtual Desktop, consultez [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md).
- Pour plus d’informations sur le service, consultez [Environnement Windows Virtual Desktop](environment-setup.md).
- Suivez le [Didacticiel : Résoudre les problèmes liés aux déploiements de modèles Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Pour en savoir plus sur les actions d’audit, consultez [Opérations d’audit avec Resource Manager](../azure-resource-manager/management/view-activity-logs.md).
- Pour en savoir plus sur les actions visant à déterminer les erreurs au cours du déploiement, consultez [Voir les opérations de déploiement](../azure-resource-manager/templates/deployment-history.md).
