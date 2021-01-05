---
title: Déployer des applications sécurisées sur Microsoft Azure
description: Cet article présente les meilleures pratiques à prendre en compte pendant les phases de mise en production et de réponse de votre projet d’application web.
author: TerryLanfear
manager: barbkess
ms.author: terrylan
ms.date: 06/12/2019
ms.topic: article
ms.service: security
ms.subservice: security-develop
services: azure
ms.assetid: 521180dc-2cc9-43f1-ae87-2701de7ca6b8
ms.devlang: na
ms.tgt_pltfrm: na
ms.workload: na
ms.openlocfilehash: 4c71ddbf1d2b435697b2707acf0b1262f2c5dc31
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517187"
---
# <a name="deploy-secure-applications-on-azure"></a>Déployer des applications sécurisées sur Azure
Cet article présente les activités et contrôles de sécurité à prendre en compte lorsque vous déployez des applications pour le cloud. Les questions et concepts de sécurité à prendre en compte pendant les phases de mise en production et de réponse du [Microsoft Security Development Lifecycle](/previous-versions/windows/desktop/cc307891(v=msdn.10)) y sont abordés. L’objectif est de vous aider à définir les activités et services Azure que vous pouvez utiliser pour déployer une application plus sécurisée.

Les phases de Microsoft Security Development Lifecycle suivantes sont traitées dans cet article :

- Libérer
- response

## <a name="release"></a>Libérer
La phase de mise en production consiste à préparer un projet pour la version publique.
Il s’agit notamment de planifier des méthodes permettant d’effectuer efficacement des tâches de maintenance post-lancement et de corriger les failles de sécurité susceptibles de se produire plus tard.

### <a name="check-your-applications-performance-before-you-launch"></a>Vérifier les performances de l’application avant de procéder à son lancement

Vérifiez les performances de l’application avant de la lancer ou de déployer des mises à jour en production. Exécutez des [tests de charge](https://www.visualstudio.com/docs/test/performance-testing/getting-started/getting-started-with-performance-testing) dans le cloud à l’aide de Visual Studio pour détecter les problèmes de performances de l’application, améliorer la qualité du déploiement, vous assurer que l’application est toujours fonctionnelle ou disponible et qu’elle peut gérer le trafic correspondant à votre lancement.

### <a name="install-a-web-application-firewall"></a>Installer un pare-feu d’application web

Les applications Web sont de plus en plus la cible d’attaques malveillantes qui exploitent des vulnérabilités connues. Les types d’attaques les plus courantes sont l’injection de code SQL et les attaques de script site à site. Il peut s’avérer difficile d’empêcher ces attaques dans le code de l’application. Cela peut nécessiter une maintenance rigoureuse, une mise à jour corrective et la surveillance au niveau de nombreuses couches de la topologie de l’application. Un pare-feu d’applications web (WAF) centralisé permet de simplifier la gestion de la sécurité. Une solution WAF peut également réagir à une menace de sécurité en exécutant la mise à jour corrective d’une vulnérabilité connue dans un emplacement central plutôt que de sécuriser individuellement chaque application web.

La fonctionnalité [WAF d’Azure Application Gateway](../../web-application-firewall/ag/ag-overview.md) permet de protéger de manière centralisée vos applications web contre les vulnérabilités courantes et le code malveillant exploitant. Le WAF suit les règles des [ensembles de règles de base OWASP](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 ou 2.2.9.

### <a name="create-an-incident-response-plan"></a>Créer un plan de réponse aux incidents

La préparation d’un plan de réponse aux incidents est cruciale pour vous aider à déjouer les nouvelles menaces susceptibles d’émerger au fil du temps. Cette préparation inclut l’identification des personnes appropriées à contacter en cas d’urgence liée à la sécurité ainsi que l’établissement de plans de maintenance de sécurité pour le code tiers sous licence et pour le code hérité d’autres groupes de l’organisation.

### <a name="conduct-a-final-security-review"></a>Effectuer un examen de final de la sécurité

Examiner délibérément toutes les activités de sécurité qui ont été effectuées permet de s’assurer que l’application ou la version du logiciel sont opérationnelles. L’examen final de la sécurité comprend généralement l’examen des modèles de menace, des sorties d’outils ainsi que des performances par rapport aux critères de qualité et barres de bogues qui ont été définis dans la phase des spécifications.

### <a name="certify-release-and-archive"></a>Certifier la mise en production et l’archivage

La certification logicielle avant une mise en production permet de s’assurer que les exigences de sécurité et de confidentialité sont satisfaites. L’archivage de toutes les données pertinentes est essentiel pour la réalisation des tâches de maintenance post-lancement. Il contribue également à réduire les coûts à long terme associés au génie logiciel soutenu.

## <a name="response"></a>response
La phase post-lancement de réponse se concentre sur l’équipe de développement disponible et à même de répondre correctement à tous les rapports des nouvelles menaces et vulnérabilités logicielles.

### <a name="execute-the-incident-response-plan"></a>Exécuter le plan de réponse aux incidents

Pouvoir implémenter le plan de réponse aux incidents créé dans la phase de mise en production est essentiel pour aider à protéger les clients contre les nouvelles vulnérabilités qui touchent la confidentialité ou la sécurité des logiciels.

### <a name="monitor-application-performance"></a>Analyse des performances d’une application

La surveillance continue de l’application après son déploiement vous aide potentiellement à détecter les problèmes de performances ainsi que les failles de sécurité.
Services Azure permettant d’aider à la surveillance des applications :

  - Azure Application Insights
  - Azure Security Center

#### <a name="application-insights"></a>Application Insights

[Application Insights](../../azure-monitor/app/app-insights-overview.md) est un service extensible de gestion des performances des applications (APM) destiné aux développeurs web sur de multiples plateformes. Utilisez-le pour analyser votre application web en direct. Application Insights détecte automatiquement les anomalies de performances. Il intègre de puissants outils d’analyse pour vous aider à diagnostiquer les problèmes et à comprendre ce que font les utilisateurs avec votre application. Il a été conçu pour vous permettre d’améliorer continuellement les performances et la convivialité.

#### <a name="azure-security-center"></a>Azure Security Center

[Azure Security Center](../../security-center/security-center-introduction.md) vous aide à prévenir, détecter et résoudre les menaces grâce à une visibilité et un contrôle accrus de la sécurité de vos ressources Azure, notamment des applications web. Azure Security Center vous aide à détecter les menaces qui pourraient passer inaperçues. Il fonctionne avec différentes solutions de sécurité.

Le niveau gratuit de Security Center offre une sécurité limitée pour vos ressources Azure uniquement. Le [niveau Standard de Security Center](../../security-center/security-center-get-started.md) étend ces fonctionnalités aux ressources locales et à d’autres clouds.
Le niveau Standard de Security Center vous aide à :

  - détecter et corriger les failles de sécurité ;
  - appliquer des contrôles d’accès et d’application pour bloquer les activités malveillantes ;
  - détecter les menaces à l’aide de l’analytique et de l’analyse décisionnelle ;
  - réagir rapidement en cas d’attaque.

## <a name="next-steps"></a>Étapes suivantes
Dans les articles suivants, nous recommandons des contrôles et des activités de sécurité qui peuvent vous aider à concevoir et développer des applications sécurisées.

- [Concevoir des applications sécurisées](secure-design.md)
- [Développer des applications sécurisées](secure-develop.md)