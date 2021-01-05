---
title: Contrôle de sécurité Azure - Réponse aux incidents
description: Réponse aux incidents du contrôle de sécurité Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: fb4c4c5a0cf6610af17aabc562c42d2e0eb4e6a4
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94409091"
---
# <a name="security-control-incident-response"></a>Contrôle de sécurité : Réponse aux incidents

Protégez les informations de l’organisation, ainsi que sa réputation, en développant et en implémentant une infrastructure de réponse aux incidents (par exemple, des plans, des rôles définis, une formation, des communications, une supervision de gestion) pour repérer rapidement une attaque et en contenir les dommages, en éliminant la présence de l’attaquant et en restaurant l’intégrité du réseau et des systèmes.

## <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 10.1 | 19.1, 19.2, 19.3 | Customer |

Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.  

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

## <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 10.2 | 19,8 | Customer |

Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte. 

En outre, marquez clairement les abonnements (par ex. production, non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](../../azure-resource-manager/management/tag-resources.md)

## <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 10.3 | 19 | Customer |

Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://csrc.nist.gov/publications/detail/sp/800-84/final)

## <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 10,4 | 19.5 | Customer |

Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le centre de réponse aux incidents de sécurité Microsoft (MSRC, Microsoft Security Response Center) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../../security-center/security-center-provide-security-contact-details.md)

## <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 10.5 | 19.6 | Customer |

Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../../sentinel/connect-azure-security-center.md)

## <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 10.6 | 19 | Customer |

Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

- [Comment configurer l’automatisation des workflows et Logic Apps](../../security-center/workflow-automation.md)


## <a name="next-steps"></a>Étapes suivantes

- Consultez le contrôle de sécurité suivant : [Tests d’intrusion et exercices Red Team](security-control-penetration-tests-red-team-exercises.md)