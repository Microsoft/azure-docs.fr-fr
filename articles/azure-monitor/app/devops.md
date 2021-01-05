---
title: Supervision des performances des applications web - Azure Application Insights
description: Comment Application Insights s’intègre dans le cycle des opérations de développement
ms.topic: conceptual
ms.date: 12/21/2018
ms.openlocfilehash: d2c3326c3f655d4bdfeaa42ac272658b251b7f82
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87324486"
---
# <a name="deep-diagnostics-for-web-apps-and-services-with-application-insights"></a>Diagnostic approfondi des applications et services web avec Application Insights
## <a name="why-do-i-need-application-insights"></a>Pourquoi ai-je besoin d’Application Insights ?
Application Insights surveille votre application web en cours d’exécution. En plus de signaler les défaillances et problèmes de performance, il permet d’analyser comment les clients utilisent votre application. Résidant dans le cloud ou localement, il fonctionne pour les applications qui s'exécutent sur de nombreuses plateformes (ASP.NET, Java EE, Node.js, etc.). 

![Aspects de la complexité liée à la distribution d’applications web](./media/devops/010.png)

Il est essentiel de surveiller une application moderne pendant qu’elle s’exécute. Vous voudrez surtout détecter les erreurs avant la plupart de vos clients. Mais aussi détecter et résoudre les problèmes de performances qui, sans être catastrophiques, ralentissent peut-être le fonctionnement ou pénalisent certains de vos utilisateurs. Lorsque le système fonctionne comme vous le souhaitez, il peut être intéressant de savoir ce que les utilisateurs en font : utilisent-ils la dernière version ? Sont-ils satisfaits par celle-ci ?

Les applications web modernes sont développées selon un cycle continu : publication d’une nouvelle fonctionnalité ou d’une amélioration ; analyse de son fonctionnement pour les utilisateurs ; planification de la prochaine phase de développement en fonction de ces informations. La phase d’observation de ce cycle est essentielle. Application Insights fournit les outils permettant de surveiller les performances et l’utilisation d’une application web.

L’aspect le plus important de ce processus est le diagnostic. Une application défectueuse se traduit par une perte d’activité. Le rôle principal d’une infrastructure de surveillance est de détecter les défaillances de façon fiable, de vous les signaler immédiatement et de vous présenter les informations nécessaires pour diagnostiquer le problème. C’est exactement ce que fait Application Insights.

### <a name="where-do-bugs-come-from"></a>D'où proviennent les bogues ?
Dans les systèmes web, les défaillances proviennent généralement de problèmes de configuration ou de mauvaises interactions entre les multiples composants. Lors du traitement d’une défaillance de site, la première tâche consiste à identifier l’emplacement du problème : quel composant ou quelle relation est en cause ?

Les plus âgés d’entre nous se rappellent l’époque où un programme s’exécutait sur un ordinateur. Les développeurs le testaient complètement avant de le commercialiser. Une fois celui-ci sur le marché, ils n’y pensaient plus. Les utilisateurs devraient faire avec les bogues pendant de nombreuses années. 

Aujourd’hui, les choses sont bien différentes. Votre application s’exécute sur une multitude d’appareils et il est difficile de garantir un comportement rigoureusement identique sur chacun d’eux. L’hébergement d’applications dans le cloud permet de corriger les bogues rapidement, mais cela se traduit également une concurrence permanente et l’attente de nouvelles fonctionnalités à intervalles réguliers. 

Dans ces conditions, la seule façon de maîtriser le nombre de bogues consiste à automatiser le test unitaire. Tester chaque version manuellement et intégralement est impossible. Le test unitaire fait désormais partie du processus de compilation. Des outils tels que Xamarin Test Cloud permettent d’automatiser le test d’interfaces utilisateur sur plusieurs versions de navigateur. Ils nous permettent de minimiser le nombre de bogues trouvés dans une application.

Les applications web classiques ont de nombreux composants actifs. Outre le client (dans un navigateur ou une application d’apppareil) et le serveur web, il se peut qu’un traitement de serveur principal important soit nécessaire. Par exemple, le serveur principal est un pipeline de composants, ou un ensemble moins imbriqué d’éléments en interrelation. Et bon nombre de ces éléments ne seront pas sous votre contrôle. Il s’agit de services externes dont vous dépendez.

Dans de telles configurations, il peut être difficile et onéreux de tester ou prévoir, chaque mode de défaillance, outre le système lui-même. 

### <a name="questions-"></a>Questions...
Voici quelques questions que nous nous posons lorsque nous développons un système web :

* Mon application se bloque-t-elle ? 
* Que s’est-il passé exactement ? - Si une requête n’a pas pu être traitée, je veux savoir pourquoi. Nous avons d’un suivi des événements...
* Mon application est-elle assez rapide ? Combien de temps lui faut-il pour répondre aux demandes classiques ?
* Le serveur peut-il gérer la charge ? Lorsque le taux de demandes augmente, le temps de réponse reste-t-il stable ?
* Quel est le niveau de réactivité de mes dépendances (API REST, bases de données et autres composants appelés par mon application) ? En particulier, la lenteur du système est-elle due à mon composant ou est-ce que je reçois des réponses lentes de quelqu'un d’autre ?
* Mon application fonctionne-t-elle ou pas ? Le monde entier peut-il le voir ? Signalez-moi tout dysfonctionnement...
* Quelle en est la cause ? S’agissait-il d’un problème dans mon composant ou d’une dépendance ? Est-ce un problème de communication ?
* Combien d’utilisateurs en sont affectés ? Si j’ai plusieurs problèmes à résoudre, lequel est le plus important ?

## <a name="what-is-application-insights"></a>Présentation d’Application Insights
![Flux de travail de base d’Application Insights](./media/devops/020.png)

1. Application Insights analyse votre application et envoie les données télémétriques la concernant pendant son exécution. Vous pouvez soit intégrer le Kit de développement logiciel (SDK) d’Application Insights dans l’application, soit appliquer l’instrumentation lors de l’exécution. La première méthode est plus souple, car vous pouvez ajouter vos propres données de télémétrie aux modules standard.
2. Les données télémétriques sont envoyées au portail Application Insights, où elles sont stockées et traitées. (Bien qu’hébergé dans Microsoft Azure, Application Insights peut surveiller toutes les applications web, et pas seulement les applications Azure.)
3. Les données télémétriques sont présentées sous la forme de graphiques et de tableaux d’événements.

Il existe deux principaux types de données télémétriques : les instances agrégées et les instances brutes. 

* Les données d’instance incluent, par exemple, un rapport d’une demande reçue par votre application web. Vous pouvez rechercher et examiner les détails d’une demande à l’aide de l’outil de recherche dans le portail Application Insights. L’instance inclura, entre autres, des données sur le temps de réponse à la demande, l’URL demandée et l’emplacement approximatif du client.
* Les données agrégées incluent le nombre d’événements par unité de temps, afin de pouvoir comparer le nombre de demandes aux temps de réponse. Elles indiquent également des valeurs moyennes, comme le temps de réponse aux demandes.

Les principales catégories de données sont les suivantes :

* Demandes à votre application (généralement des demandes HTTP), avec des données sur l’URL, le temps de réponse et la réussite ou l’échec.
* Dépendances - Appels REST et SQL effectués par votre application, avec l’URI, les temps de réponse et la réussite
* Exceptions, y compris les traces de pile.
* Données sur les vues de page, provenant des navigateurs des utilisateurs.
* Mesures, comme les compteurs de performance ou les mesures que vous écrivez vous-même. 
* Événements personnalisés que vous pouvez utiliser pour le suivi d’événements commerciaux
* Traces de journal utilisées pour le débogage.

## <a name="case-study-real-madrid-fc"></a>Étude de cas : Real Madrid
Le service web du [club de football Real Madrid](https://www.realmadrid.com/) est visité par environ 450 millions de supporters dans le monde entier. Ils y accèdent à la fois par les navigateurs web et les applications mobiles du Club. Les supporters n’achètent pas seulement des billets, ils consultent des informations et visionnent des vidéos sur les résultats, les joueurs et les prochains matchs. Ils peuvent effectuer des recherches avec des filtres, comme le nombre de buts marqués. Le site contient également des liens vers les réseaux sociaux. L’expérience utilisateur est extrêmement personnalisée et conçue comme une communication bidirectionnelle pour privilégier l’interaction avec les supporters.

La solution [est un système de services et d’applications sur Microsoft Azure](https://www.microsoft.com/inculture/sports/real-madrid/). L’évolutivité est essentielle : le trafic, variable, peut atteindre des sommets avant, pendant et après les matchs.

Pour le Real Madrid, surveiller les performances du système est primordial. Azure Application Insights offre une vue complète du système, ainsi qu’un niveau de service fiable et élevé. 

Le club en retire également une connaissance approfondie de ses supporters : leur lieu de résidence (seulement 3 % vivent en Espagne), leur intérêt vis-à-vis des joueurs, les résultats, les prochains matchs et leurs réactions aux résultats.

La plupart de ces données télémétriques sont recueillies automatiquement sans code ajouté, ce qui a considérablement simplifié la solution et en a réduit la complexité opérationnelle.  Pour le Real Madrid, Application Insights gère 3,8 milliards de points de télémétrie par mois.

Le Real Madrid utilise le module Power BI pour afficher ses données télémétriques.

![Vue Power BI des données télémétriques d’Application Insights](./media/devops/080.png)

## <a name="smart-detection"></a>Détection intelligente
[Diagnostics proactifs](./proactive-diagnostics.md) est une fonctionnalité récente. Sans aucune configuration particulière, Application Insights détecte automatiquement et signale toute hausse inhabituelle du nombre de défaillances dans votre application. L’application est suffisamment intelligente pour ignorer les défaillances occasionnelles et les augmentations qui sont simplement proportionnelles à une hausse des demandes. Par exemple, si un des services dont vous dépendez est défectueux ou si la nouvelle build que vous venez de déployer ne fonctionne pas bien, vous le saurez dès que vous aurez consulté votre messagerie. (Et les webhooks vous permettent de déclencher d’autres applications.)

Par ailleurs, cette fonction effectue quotidiennement une analyse approfondie de vos données télémétriques, en y recherchant des performances inhabituelles difficiles à détecter. Par exemple, elle peut identifier un faible niveau de performance dans une zone géographique ou avec une version particulière d’un navigateur.

Dans les deux cas, l’alerte ne vous signale pas seulement les symptômes détectés, mais elle vous fournit également les données dont vous avez besoin pour diagnostiquer le problème, comme les rapports d’exception appropriés.

![E-mail issus des diagnostics proactifs](./media/devops/030.png)

Notre client Samtec a déclaré : « Récemment, pendant une interruption du service, nous avons trouvé une base de données sous-dimensionnée qui atteignait la limite de ses ressources, provoquant ainsi des délais. Des alertes de détection proactives nous parvenaient pendant que nous étions en temps de traiter le problème, en quasi temps réel comme annoncé par notre publicité. Ces alertes, couplées à celles de la plateforme Azure, nous ont permis de résoudre le problème presque instantanément. Temps d’arrêt total < 10 minutes. »

## <a name="live-metrics-stream"></a>Live Metrics Stream (Flux continu de mesures)
Le déploiement de la dernière build en date peut être très stressant. S’il y a des problèmes, vous voulez en avoir connaissance immédiatement, afin de corriger la situation si nécessaire. Live Metrics Stream (Flux continu de mesures) fournit des mesures clés avec une latence d’environ une seconde.

![Mesures actives](./media/devops/0040.png)

Et vous permet d’inspecter immédiatement un échantillon des échecs ou exceptions éventuels.

![Événements d’échec en direct](./media/devops/002-live-stream-failures.png)

## <a name="application-map"></a>Mise en correspondance d'applications
Application Map (Mise en correspondance d’applications) détecte automatiquement la topologie de votre application et place les informations de performances sur celle-ci pour vous permettre d’identifier facilement les goulots d’étranglement et les flux problématiques dans votre environnement distribué. Elle vous permet de découvrir les dépendances des applications sur les Services Azure. Vous pouvez catégoriser le problème déterminant s’il est lié au code ou aux dépendances et, à partir d’un emplacement unique, effectuer des diagnostics. Par exemple, votre application peut dysfonctionner en raison d’une chute des performances du SQL. Application Map (Mise en correspondance d’applications) vous permet de visualiser le phénomène immédiatement et d’utiliser SQL Index Advisor ou Query Insights.

![Mise en correspondance d'applications](./media/devops/0050.png)

## <a name="application-insights-analytics"></a>Application Insights Analytics
Avec [Analytics](../log-query/log-query-overview.md), vous pouvez écrire des requêtes arbitraires dans un puissant langage de type SQL.  Diagnostiquer toute la pile de l’application devient facile, car plusieurs perspectives sont mises en relation, et vous pouvez poser les bonnes questions pour mettre en corrélation les performances du service avec les mesures de l’activité et l’expérience client. 

Vous pouvez interroger toutes les données des mesures et de l’instance de télémétrie, stockées dans le portail. Le langage comprend des opérations de filtrage, de jointure, d’agrégation et autres. Vous pouvez calculer des champs et effectuer une analyse statistique. Les visualisations peuvent être tabulaires ou graphiques.

![Graphique de l’interrogation d’analyse et des résultats](./media/devops/0025.png)

Par exemple, les opérations suivantes sont faciles :

* Segmenter les données de performances des demandes de votre application par niveaux de client pour comprendre l’expérience de ces derniers.
* Rechercher des codes d’erreur spécifiques ou des noms d’événement personnalisés pendant l’analyse d’un site actif.
* Explorer l’utilisation de l’application par certains clients pour déterminer comment les fonctionnalités sont acquises et adoptées.
* Analyser les sessions et les temps de réponse de certains utilisateurs pour permettre aux équipes de support et d’exploitation d’aider les clients instantanément.
* Déterminer les fonctionnalités d’application fréquemment utilisées pour répondre aux questions sur la hiérarchisation des fonctionnalités.

Notre client DNN a déclaré : « Application Insights a représenté pour nous la pièce manquante du puzzle, en offrant la possibilité de combiner, de trier, d’interroger et de filtrer les données selon les besoins. Le fait que notre équipe puisse utiliser sa propre ingéniosité et sa propre expérience pour trouver des données grâce à un puissant langage d’interrogation nous a permis de mettre au jour des informations et de résoudre des problèmes dont nous-même ignorions l’existence. Beaucoup de réponses intéressantes font suite à des questions commençant par *'' Je me demande si...'' .* »

## <a name="development-tools-integration"></a>Intégration d’outils de développement
### <a name="configuring-application-insights"></a>Configuration d'Application Insights
Visual Studio et Eclipse disposent d’outils pour configurer les packages de kit de développement logiciel (SDK) correspondant au projet que vous développez. Il existe une commande de menu permettant d’ajouter Application Insights.

Si vous utilisez une fonctionnalité de journalisation des traces, telle que Log4N, NLog ou System.Diagnostics.Trace, vous pouvez envoyer les journaux d’activité à Application Insights avec les autres données télémétriques, ce qui facilite la mise en corrélation des traces avec les demandes, les appels de dépendance et les exceptions.

### <a name="search-telemetry-in-visual-studio"></a>Recherche de données télémétriques dans Visual Studio
Lors du développement et du débogage d’une fonctionnalité, vous pouvez afficher les données télémétriques directement dans Visual Studio, et effectuer des recherches à l’aide des mêmes fonctionnalités que dans le portail web.

Et lorsqu’Application Insights consigne une exception, vous pouvez afficher le point de données dans Visual Studio et accéder directement au code concerné.

![Recherche Visual Studio](./media/devops/060.png)

Pendant le débogage, vous pouvez conserver les données de télémétrie dans votre ordinateur de développement, en les affichant dans Visual Studio mais sans les envoyer au portail. Cela évite de mélanger le débogage avec les données télémétriques de production.

### <a name="work-items"></a>Éléments de travail
Lorsqu’une alerte est émise, Application Insights peut automatiquement créer un élément de travail dans votre système de suivi du travail.

## <a name="but-what-about"></a>Mais qu’en est-il de... ?
* [Confidentialité et stockage](./data-retention-privacy.md) - Vos données télémétriques sont conservées sur des serveurs Azure sécurisés.
* Performances - L’impact est très faible. Les données télémétriques sont traitées par lot.
* [Tarification](./pricing.md) - Vous pouvez démarrer gratuitement tant que votre volume reste faible.


## <a name="video"></a>Vidéo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>Étapes suivantes
La prise en main d’Application Insights est simple. Les principales options sont les suivantes :

* [Serveurs IIS](./monitor-performance-live-website-now.md) ainsi que pour [Azure App Service](./app-insights-overview.md).
* Instrumenter votre projet pendant le développement. Vous pouvez effectuer cette tâche pour des applications [ASP.NET](./asp-net.md), [Java](./java-get-started.md) et [Node.js](./nodejs.md) et une multitude d’[autres types](./platforms.md). 
* Instrumenter [n’importe quelle page web](./javascript.md) en y ajoutant un court extrait de code.

