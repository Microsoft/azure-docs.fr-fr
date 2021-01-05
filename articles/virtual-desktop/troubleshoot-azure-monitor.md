---
title: Résolution des problèmes liés à Monitor pour Windows Virtual Desktop (préversion) – Azure
description: Guide pratique pour résoudre les problèmes liés à Azure Monitor pour Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 12/01/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 91cf6729911cdb674c5451f172e76a2e9d5943e4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465387"
---
# <a name="troubleshoot-azure-monitor-for-windows-virtual-desktop-preview"></a>Résolution des problèmes liés à Azure Monitor pour Windows Virtual Desktop (préversion)

>[!IMPORTANT]
>Azure Monitor pour Windows Virtual Desktop est en préversion publique. Cette préversion est fournie sans contrat de niveau de service, c’est pourquoi nous déconseillons son utilisation pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Cet article présente les problèmes connus dans Azure Monitor pour Windows Virtual Desktop (préversion) et les solutions aux problèmes courants.

## <a name="the-configuration-workbook-isnt-working-properly"></a>Le classeur de configuration ne fonctionne pas correctement

Si le classeur de configuration Azure Monitor ne fonctionne pas, vous pouvez utiliser ces ressources pour configurer ses éléments manuellement :

- Pour activer manuellement les diagnostics ou accéder à l’espace de travail Log Analytics, consultez [Envoi de diagnostics Windows Virtual Desktop à Log Analytics](diagnostics-log-analytics.md).
- Pour installer manuellement l’extension Log Analytics sur un hôte, consultez [Extension de machine virtuelle Log Analytics pour Windows](../virtual-machines/extensions/oms-windows.md).
- Pour configurer un nouvel espace de travail Log Analytics, consultez [Création d’un espace de travail Log Analytics sur le Portail Azure](../azure-monitor/learn/quick-create-workspace.md).
- Pour ajouter ou supprimer des compteurs de performances, consultez [Configuration des compteurs de performances](../azure-monitor/platform/data-sources-performance-counters.md).
- Pour configurer des événements pour un espace de travail Log Analytics, consultez [Collecte de sources de données du journal des événements Windows avec l’agent Log Analytics](../azure-monitor/platform/data-sources-windows-events.md).

Le problème peut également être dû à un manque de ressources ou à un défaut d’autorisations requises.

Si l’abonnement ne comporte pas de ressources Windows Virtual Desktop, il n’apparaît pas dans le paramètre *Abonnement*.

Si vous ne disposez pas d’un accès en lecture aux abonnements concernés, ils n’apparaissent pas dans le paramètre *Abonnement*. Leurs données ne figurent pas dans le tableau de bord. Pour résoudre ce problème, contactez le propriétaire de votre abonnement et demandez un accès en lecture.

## <a name="my-data-isnt-displaying-properly"></a>Mes données ne s’affichent pas correctement

Si vos données ne s’affichent pas correctement, cela peut être dû à un problème survenu au cours du processus de configuration Azure Monitor. Tout d’abord, vérifiez que vous avez rempli tous les champs du classeur de configuration (cf. [Monitoring d’un déploiement avec Azure Monitor pour Windows Virtual Desktop](azure-monitor.md)). Vous pouvez à tout moment modifier les paramètres des environnements existants comme des nouveaux. S’il vous manque des compteurs ou des événements, les données associées n’apparaissent pas sur le Portail Azure.

S’il ne manque aucune information, mais que les données ne s’affichent toujours pas correctement, il peut y avoir un problème dans la requête ou les sources de données. 

Si vous ne trouvez pas d’erreurs de configuration et que vous ne voyez toujours pas les données attendues, vous pouvez attendre 15 minutes et actualiser le flux. Azure Monitor présente une période de latence de 15 minutes pour le remplissage des données de journal. Pour plus d’informations, consultez [Durée d’ingestion des données de journal dans Azure Monitor](../azure-monitor/platform/data-ingestion-time.md).

Enfin, s’il ne manque aucune information, mais que les données n’apparaissent toujours pas, il peut y avoir un problème dans la requête ou les sources de données. Vous devrez peut-être alors contacter le support pour résoudre le problème.

## <a name="i-want-to-customize-azure-monitor-for-windows-virtual-desktop"></a>Je veux personnaliser Azure Monitor pour Windows Virtual Desktop

Azure Monitor pour Windows Virtual Desktop utilise des classeurs Azure Monitor. Les classeurs vous permettent d’enregistrer une copie du modèle de classeur Windows Virtual Desktop et d’effectuer vos propres personnalisations.

Les modèles personnalisés ne sont pas mis à jour lorsque le groupe de produits met à jour le modèle d’origine. Ce comportement est lié à la conception de l’outil de classeurs : vous devez en effet enregistrer une copie du modèle mis à jour et générer à nouveau vos personnalisations pour adopter les mises à jour. Pour plus d’informations, consultez [Résolution des problèmes liés aux insights issus des classeurs](../azure-monitor/insights/troubleshoot-workbooks.md) et [Vue d’ensemble des classeurs](../azure-monitor/platform/workbooks-overview.md).

## <a name="i-cant-interpret-the-data"></a>Je ne parviens pas à interpréter les données

Pour plus d’informations sur les termes liés aux données, consultez le [glossaire Azure Monitor pour Windows Virtual Desktop](azure-monitor-glossary.md).

## <a name="the-data-i-need-isnt-available"></a>Les données dont j’ai besoin ne sont pas disponibles

Il vous manque un point de données pour diagnostiquer un problème ? Envoyez-nous vos commentaires.

- Pour savoir comment laisser des commentaires, consultez [Vue d’ensemble de la résolution des problèmes, des commentaires et du support pour Windows Virtual Desktop](troubleshoot-set-up-overview.md).
- Vous pouvez également laisser vos commentaires sur Windows Virtual Desktop sur le [Hub de commentaires Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app) ou notre [forum UserVoice](https://windowsvirtualdesktop.uservoice.com/forums/921118-general).

## <a name="known-issues"></a>Problèmes connus

Voici les problèmes connus à l’heure actuelle et en cours de résolution :

- Il n’est possible de sélectionner qu’un seul abonnement, un seul groupe de ressources et un seul pool d’hôtes à surveiller à la fois. Lorsque vous utilisez la page Rapports utilisateur pour comprendre l’expérience d’un utilisateur, vous devez donc vérifier que vous disposez bien du pool d’hôtes dont se sert l’utilisateur ; sinon, ses données n’apparaîtront pas dans les visuels.

- Il n’est pas possible d’enregistrer les paramètres favoris dans Azure Monitor, à moins d’enregistrer un modèle personnalisé du classeur. Les administrateurs informatiques doivent donc entrer le nom de leur abonnement, celui des groupes de ressources et leurs préférences de pool d’hôtes chaque fois qu’ils ouvrent Azure Monitor pour Windows Virtual Desktop.

- Il n’existe actuellement aucun moyen d’exporter des données d’Azure Monitor pour Windows Virtual Desktop dans Excel.

- Toutes les alertes Azure Monitor de gravité 1 de tous les produits de l’abonnement sélectionné s’affichent sur la page Vue d’ensemble. Ce comportement est lié à la conception des alertes. Il est en effet possible que les alertes d’autres produits de l’abonnement aient une incidence sur Windows Virtual Desktop. Pour le moment, la requête est limitée aux alertes de gravité 1. Sont exclues les alertes de gravité 0 prioritaires de la page Vue d’ensemble.

- Certains messages d’erreur ne sont pas formulés de manière conviviale. Par ailleurs, tous ne sont pas décrits dans la documentation.

## <a name="next-steps"></a>Étapes suivantes

Si vous ne savez pas comment interpréter les données ou souhaitez en savoir plus sur les termes courants, consultez le [glossaire Azure Monitor pour Windows Virtual Desktop](azure-monitor-glossary.md). Si vous voulez apprendre à configurer et à utiliser Azure Monitor pour Windows Virtual Desktop, consultez [Monitoring d’un déploiement avec Azure Monitor pour Windows Virtual Desktop](azure-monitor.md).