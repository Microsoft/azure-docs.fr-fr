---
title: Introduction aux microservices sur Azure
description: Pourquoi créer des applications cloud avec une approche de microservices est important pour le développement d’applications modernes et comment Azure Service Fabric fournit une plateforme pour y parvenir.
ms.topic: conceptual
ms.date: 01/07/2020
ms.openlocfilehash: d20e04820c87a7390d9c20e511259ee9860c27f5
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575650"
---
# <a name="why-use-a-microservices-approach-to-building-applications"></a>Pourquoi utiliser une approche de microservices pour la conception d’applications ?

Pour les développeurs de logiciels, il n’y a rien de nouveau dans la conception de l’affacturage d’une application en composants. En règle générale, une approche hiérarchisée est utilisée avec un magasin principal, une logique métier de couche intermédiaire et une interface utilisateur frontale. Ce qui *a* changé au cours des dernières années, c’est que les développeurs créent des applications distribuées pour le cloud.

Les nouveaux besoins métier sont les suivants :

* Un service créé et fonctionnant à l’échelle pour atteindre les clients situés dans de nouvelles régions géographiques.
* Distribution plus rapide des fonctionnalités pour répondre aux demandes des clients d’une manière agile.
* Meilleure utilisation des ressources pour réduire les coûts.

Ces besoins de l’entreprise affectent *la manière* dont nous créons des applications.

Pour plus d’informations sur l’approche d’Azure à l’égard des microservices, veuillez consulter [Microservices: An application revolution powered by the cloud (Microservices : une approche révolutionnaire des applications reposant sur le cloud)](https://azure.microsoft.com/blog/microservices-an-application-revolution-powered-by-the-cloud/).

## <a name="monolithic-vs-microservices-design-approach"></a>Approches de conception monolithique et de microservices

Les applications évoluent au fil du temps. Les applications réussies évoluent en étant utiles aux utilisateurs. Les applications ratées n’évoluent pas et sont, en fin de compte, déconseillées. La question est : que savez-vous de vos besoins actuels et que seront-ils à l’avenir ? Par exemple, supposons que vous créez une application de création de rapports pour un service de votre entreprise. Vous êtes certain que l’application s’applique uniquement dans l’étendue de votre entreprise et que les rapports ont une durée de vie limitée. Votre approche sera différente de celle adoptée pour la création d’un service de distribution de contenu vidéo à des dizaines de millions de clients par exemple.

Parfois, la mise en œuvre d’un élément en tant que preuve de concept constitue un facteur déterminant. Vous savez que l’application peut être repensée ultérieurement. Il n’est pas très pertinent de lancer une ingénierie complexe pour quelque chose qui ne sera jamais utilisé. Par ailleurs, lorsque des entreprises développent pour le cloud, leurs attentes concernent la croissance et l’utilisation. La croissance et l’évolutivité sont imprévisibles. Nous voulons créer des prototypes rapidement, mais, dans le même temps, savoir que nous sommes sur la voie du succès. C’est l’approche d’une lean startup : créer, mesurer, apprendre et itérer.

À l’ère client/serveur, nous avions tendance à nous concentrer sur la création d’applications hiérarchisées à l’aide de technologies spécifiques dans chaque niveau. Le qualificatif *monolithique* a vu le jour pour décrire cette approche. Les interfaces avaient tendance à se trouver entre les niveaux, et une conception plus étroitement couplée était utilisée entre les composants de chaque niveau. Les développeurs concevaient et factorisaient des classes compilées dans des bibliothèques et liaient celles-ci dans quelques fichiers exécutables et DLL.

Une approche de conception monolithique présente des avantages. Les applications monolithiques sont souvent plus simple à concevoir, et les appels entre ses composants sont plus rapides dans la mesure où ils sont souvent effectués via IPC (communication entre processus). En outre, chacun teste un produit unique, ce qui est permet une meilleure utilisation des ressources humaines. L’inconvénient est la création d’un couplage étroit entre les couches hiérarchisées et l’impossibilité de mettre à l’échelle les composants individuels. Si vous devez appliquer des correctifs ou des mises à niveau, vous devez attendre que les autres terminent leurs tests. Une approche agile est plus difficile.

Les microservices gèrent ces inconvénients et s’alignent plus étroitement sur les exigences métier précédentes. Mais ils présentent également des avantages et inconvénients. Avantages des microservices : en général, chacun encapsule des fonctionnalités métier plus simples, dont vous pouvez effectuer indépendamment le scale-out, le scale-in, le test, le déploiement ou la gestion. L’un des principaux avantages d’une approche de microservices est que les équipes sont davantage orientées par les scénarios métier que par la technologie. De plus petites équipes développent un microservice en s’appuyant sur un scénario client et utilisent les technologies de leur choix.

En d’autres termes, l’organisation n’a pas besoin de standardiser la technologie pour maintenir les applications de microservices. Les équipes individuelles qui possèdent des services peuvent effectuer les actions les plus pertinentes pour elles en fonction de leur expertise ou de ce qui est le plus approprié pour le problème à résoudre. Dans la pratique, il est préférable de disposer d’un ensemble de technologies recommandées, par exemple d’un magasin NoSQL particulier ou d’une infrastructure d’application web.

L’inconvénient des microservices est qu’ils entraînent une gestion accrue d’entités distinctes et celle du contrôle des versions et des déploiements plus complexes. Le trafic réseau entre les microservices augmente, ainsi que les latences réseau correspondantes. Disposer d’un grand nombre de services granulaires et bavards risque de nuire gravement aux performances. Sans outils permettant d’afficher ces dépendances, il est difficile de « voir » le système dans son ensemble.

Les normes permettent à l’approche de microservices de fonctionner, en définissant la façon de communiquer et en tolérant uniquement les éléments qu’un service doit vous fournir, au lieu d’adopter des contrats rigides. Il est important de définir ces contrats en amont dans la conception, car les services sont mis à jour indépendamment les uns des autres. Une autre description a été formulée pour la conception avec une approche de microservices : « la SOA affinée ».

***Dans sa forme la plus simple, l’approche de conception de microservices fait référence à une fédération découplée des services, avec des modifications indépendantes pour chacun et des normes de communication définies.** _

Tandis qu’un nombre croissant d’applications cloud sont produites, les gens ont découvert que cette décomposition de l’application globale en services axés sur un scénario indépendant constitue une meilleure approche à long terme.

## <a name="comparison-between-application-development-approaches"></a>Comparaison entre les approches de développement des applications

![Développement d’applications pour la plateforme Service Fabric][Image1]

1) Une application monolithique contient des fonctionnalités propres à un domaine. Elle est normalement divisée en couches fonctionnelles telles que web, métier et données.

2) Vous mettez à l’échelle une application monolithique en la clonant sur plusieurs serveurs/machines virtuelles/conteneurs.

3) Une application de microservices sépare les fonctionnalités en services plus petits distincts.

4) L’approche des microservices augmente la taille des instances en déployant chaque service indépendamment, ce qui crée des instances de ces services sur les serveurs/machines virtuelles/conteneurs.

La conception avec une approche de microservices ne convient pas à tous les projets, mais cette vision se rapproche davantage des objectifs métier décrits précédemment. Commencer par une approche monolithique est acceptable si vous savez que vous n’aurez pas la possibilité de retravailler le code en une conception de microservices. Le scénario le plus courant est le suivant : vous débutez avec une application monolithique que vous fractionnez lentement en étapes, en commençant par les zones fonctionnelles qui doivent être plus évolutives ou agiles.

Lorsque vous utilisez une approche de microservices, vous composez votre application avec plusieurs petits services. Ces services sont exécutés dans des conteneurs déployés sur un cluster de machines. De plus petites équipes développent un service qui se concentre sur un scénario et testent, contrôlent les versions, déploient et mettent à l’échelle chaque service indépendamment, afin que l’intégralité de l’application puisse évoluer.

## <a name="what-is-a-microservice"></a>Que sont les microservices ?

Il existe plusieurs définitions du terme « microservice ». Cependant, la plupart des ces caractéristiques de microservices font l’objet d’un large consensus :

_ Ils encapsulent un scénario client ou d’entreprise. Quel problème résolvez-vous ?
* Ils sont développés par une petite équipe d’ingénierie.
* Ils peuvent être écrits dans n’importe quel langage de programmation, en utilisant n’importe quelle infrastructure.
* Ils se composent d’un code et éventuellement d’un état, gérés indépendamment en termes de contrôle des versions, de déploiement et de mise à l’échelle.
* Ils interagissent avec les autres microservices sur des interfaces et protocoles bien définis.
* Ils possèdent des noms uniques (URL) utilisés pour résoudre leur emplacement.
* Ils restent cohérents et disponibles en cas de défaillances.

En résumé :

***Les applications de microservice sont composées de services d’envergure modeste, évolutifs, dont les versions sont gérées indépendamment et qui communiquent entre eux via des protocoles standard avec des interfaces bien définies.** _

### <a name="written-in-any-programming-language-using-any-framework"></a>Ils peuvent être écrits dans n’importe quel langage de programmation, en utilisant n’importe quelle infrastructure.

En tant que développeurs, nous voulons être libres d’opter pour le langage ou l’infrastructure de notre choix, selon nos compétences ou les besoins du service que nous créons. Pour certains services, vous pouvez avant tout apprécier les avantages en termes de performances de C++. Pour d’autres, les facilités de développement de code managé que vous obtenez à partir de C# ou Java peuvent être plus importantes. Dans certains cas, vous devrez peut-être utiliser la bibliothèque d’un partenaire, une technologie de stockage de données ou une méthode pour exposer le service aux clients.

Après avoir choisi une technologie, vous devez prendre en considération l’étape de gestion du cycle de vie ou des opérations et de la mise à l’échelle du service.

### <a name="allows-code-and-state-to-be-independently-versioned-deployed-and-scaled"></a>Le code et l’état peuvent être indépendamment gérés en termes de version, déployés et mis à l’échelle

Toutefois, si vous écrivez vos microservices, le code et, éventuellement, l’état, doivent être déployés, mis à niveau et mis à l’échelle indépendamment. Ce problème est difficile à résoudre, car il dépend de vos choix en matière de technologie. Pour la mise à l’échelle, comprendre comment partitionner le code et l’état est difficile. Lorsque le code et l’état utilisent des technologies distinctes (ce qui est courant à l’heure actuelle), les scripts de déploiement pour votre microservice doivent pouvoir les mettre à l’échelle. Cette distinction concerne également l’agilité et la flexibilité afin que vous puissiez mettre à niveau certains des microservices sans avoir à les mettre à niveau tous en même temps.

Revenons un instant à notre comparaison entre les approches monolithiques et de microservices. Ce diagramme illustre les différences dans les approches pour l’état du stockage :

#### <a name="state-storage-for-the-two-approaches"></a>Stockage de l’état pour les deux approches

![Stockage de l’état de la plateforme Service Fabric][Image2]

_*_L’approche monolithique, située sur la gauche, possède une base de données unique et des niveaux de technologies spécifiques._*_

_*_L’approche des microservices, située sur la droite, comprend un graphique de microservices interconnectés, où l’état est généralement limité au microservice et aux différentes technologies utilisés._*_

Dans l’approche monolithique, l’application utilise généralement une base de données unique. L’avantage de l’utilisation d’une base de données est qu’il s’agit d’un seul emplacement, ce qui facilite le déploiement. Chaque composant peut avoir une seule table pour stocker son état. Les équipes doivent être rigoureuses en ce qui concerne la séparation de l’état, qui représente un véritable défi. Inévitablement, il est tentant d’ajouter une nouvelle colonne à une table cliente existante, de joindre les tables et de créer des dépendances au niveau de la couche de stockage. Une fois cela effectué, vous ne pouvez pas mettre à l’échelle les composants individuels.

Dans l’approche de microservices, chaque service gère et stocke son propre état. Chaque service est responsable de la mise à l’échelle du code et de l’état pour répondre aux demandes du service. Un inconvénient de cette approche apparaît lorsque vous devez créer des vues ou des requêtes relatives aux données de vos applications, car vous devez exécuter des requêtes dans plusieurs magasins d’état. Ce problème est généralement résolu en disposant d’un microservice distinct qui génère une vue sur une collection de microservices. Si vous devez exécuter plusieurs requêtes ad hoc sur les données, nous vous conseillons d’écrire chaque donnée de microservice vers un service d’entreposage de données pour l’analyse hors connexion.

Les microservices sont gérés. Vous pouvez exécuter différentes versions d’un microservice côte à côte. Il est possible qu’une version plus récente d’un microservice échoue au cours de la mise à niveau et doive être restaurée vers une version antérieure. La gestion des versions est également utile pour les tests A/B, où différents utilisateurs testent différentes versions du service. Par exemple, il est courant de mettre à niveau un microservice pour un ensemble spécifique de clients afin qu’ils testent les nouvelles fonctionnalités avant de le déployer à plus grande échelle.

### <a name="interacts-with-other-microservices-over-well-defined-interfaces-and-protocols"></a>Ils interagissent avec les autres microservices sur des interfaces et protocoles bien définis

Au cours des 10 dernières années, de nombreuses informations ont été publiées décrivant les modèles de communication dans les architectures orientées services. En règle générale, la communication de service utilise une approche REST avec les protocoles HTTP et TCP, et les formats de sérialisation XML ou JSON. Du point de vue de l’interface, il s’agit une approche de conception web. Toutefois, rien ne vous empêche d’utiliser des protocoles binaires ou vos propres formats de données. Ayez seulement à l’esprit que vos utilisateurs rencontreront davantage de difficultés à utiliser vos microservices si ces protocoles et formats ne sont pas ouvertement disponibles.

### <a name="has-a-unique-name-url-used-to-resolve-its-location"></a>Ils possèdent un nom unique (URL) utilisé pour résoudre leur emplacement

Votre microservice doit être adressable partout où il est exécuté. Si vous pensez à des machines et à celles qui exécutent un microservice spécifique, les choses peuvent rapidement mal tourner.

De la même façon que le DNS résout une URL spécifique pour une machine spécifique, votre microservice doit avoir un nom unique pour que le système puisse découvrir son emplacement actuel. Les microservices ont besoin de noms adressables indépendants de l’infrastructure sur laquelle ils sont exécutés. Bien sûr, cela implique l’existence d’une interaction entre le mode de déploiement de votre service et la méthode permettant de le découvrir, dans la mesure où un Registre de service doit être disponible. Lorsqu’une machine tombe en panne, le registre de services doit vous indiquer le nouvel emplacement d’exécution du service.

### <a name="remains-consistent-and-available-in-the-presence-of-failures"></a>Ils restent cohérents et disponibles en cas de défaillances

La gestion des défaillances inattendues est l’un des problèmes les plus difficiles à résoudre, en particulier dans un système distribué. Une grande partie du code que nous écrivons en tant que développeurs concerne la gestion des exceptions. Pendant le test, nous passons également la plupart du temps à gérer des exceptions. Ce processus est plus complexe que l’écriture de code pour gérer les défaillances. Que se passe-t-il en cas de défaillance de la machine sur laquelle s’exécute le microservice ? Vous devez détecter cette défaillance, ce qui est difficile. Mais vous devez également redémarrer votre microservice.

Pour rester disponible, un microservice doit résister aux défaillances et pouvoir redémarrer souvent sur une autre machine. En plus de ces exigences de résilience, les données ne doivent pas être perdues et elles doivent rester cohérentes.

La résilience est difficile à atteindre lorsque des défaillances surviennent pendant la mise à niveau d’une application. En association avec le système de déploiement, le microservice ne doit pas seulement récupérer. Il doit déterminer s’il faut continuer à avancer vers la version la plus récente ou effectuer une restauration vers une version précédente pour maintenir un état cohérent. Vous devez prendre en compte quelques questions : par exemple, y a-t-il suffisamment de machines disponibles pour continuer de progresser ? De quelle manière restaurer les versions précédentes du microservice ? Pour répondre à cela, le microservice doit émettre des informations d’intégrité.

### <a name="reports-health-and-diagnostics"></a>Rapports d’intégrité et diagnostics

Cela peut paraître évident, et c’est souvent négligé, mais un microservice doit générer des rapports sur son intégrité et ses diagnostics. Dans le cas contraire, les informations sur l’intégrité d’un point de vue opérationnel sont limitées. Corréler des événements de diagnostic sur un ensemble de services indépendants et gérer les variations d’horloges des machines afin de déterminer l’ordre des événements représente un défi. De la même manière que vous interagissez avec un microservice selon des protocoles et des formats de données convenus, vous devez standardiser la manière de journaliser les événements d’intégrité et de diagnostic qui finiront dans un magasin d’événements que l’on peut interroger et afficher. Dans une approche de microservices, les différentes équipes doivent s’accorder sur le choix d’un format de journalisation unique. Une approche cohérente doit être adoptée pour afficher les événements de diagnostic dans l’application comme un tout.

L’intégrité diffère des diagnostics. L’intégrité fait référence aux rapports du microservice sur son état actuel de sorte que des actions appropriées puissent être prises. Un bon exemple de cela est l’utilisation de mécanismes de mise à niveau et de déploiement pour garantir la disponibilité. Même si un service peut être défectueux en raison d’un blocage de processus ou d’un redémarrage de machine, il peut demeurer opérationnel. Dans ce genre de situation, il ne faut surtout pas effectuer de mise à niveau : cela empirerait les choses. La meilleure solution est de commencer par mener une enquête ou de laisser le temps au microservice de récupérer. Les événements d’intégrité d’un microservice nous aident à prendre des décisions avisées et nous aident effectivement à créer des services de réparation spontanée.

## <a name="guidance-for-designing-microservices-on-azure"></a>Conseils pour la conception de microservices sur Azure

Pour consulter notre guide sur la conception et le développement de microservices sur Azure, veuillez vous rendre sur notre [centre des architectures Azure](/azure/architecture/microservices/).

## <a name="service-fabric-as-a-microservices-platform"></a>Service Fabric en tant que plateforme de microservices

Azure Service Fabric est apparu lorsque Microsoft est passé de la fourniture de produits prêts à l’emploi, généralement monolithique, à la prestation de services. L’expérience de création et d’utilisation de services d’envergure tels qu’Azure SQL Database et Azure Cosmos DB a permis de façonner Service Fabric. La plateforme a évolué au fil du temps, à mesure de son adoption davantage de services. Service Fabric ne devait pas s’exécuter seulement dans Azure, mais également dans les déploiements Windows Server autonomes.

_*_L’objectif de Service Fabric est de résoudre les problèmes complexes de création et d’exécution d’un service et d’utiliser les ressources d’infrastructure efficacement, afin que les équipes puissent résoudre les problèmes métier avec une approche de microservices._*_

Cette courte vidéo présente Service Fabric et les microservices :
> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Service-Fabric/player]

Service Fabric vous aide à créer des applications avec une approche de microservices en fournissant ce qui suit :

_ Une plateforme qui fournit des services système pour déployer, mettre à niveau, détecter les services en panne, router les messages, ainsi que pour gérer l’état et surveiller l’intégrité.
* La possibilité de déployer des applications exécutées dans des conteneurs ou en tant que processus. Service Fabric est un conteneur et orchestrateur de processus.
* Des API de programmation productives qui vous permettent de créer des applications en tant que microservices : [ASP.NET Core, Reliable Actors et Reliable Services](service-fabric-choose-framework.md). Par exemple, vous pouvez obtenir des informations sur l’intégrité et les diagnostics, ou tirer parti de la haute disponibilité intégrée.

***Service Fabric est indifférent à la façon dont vous créez votre service. Vous pouvez utiliser la technologie de votre choix. Toutefois, il intègre des API de programmation qui facilitent la création de microservices.** _

### <a name="migrating-existing-applications-to-service-fabric"></a>Migrer des applications existantes vers Service Fabric

Service Fabric vous permet de réutiliser le code existant, puis de le moderniser à l’aide de nouveaux microservices. La modernisation d’une application comporte cinq étapes. Il est possible de commencer et d’arrêter à n’importe quelle étape. Les étapes sont :

1) Commencez par une application monolithique classique.  
2) Migrez. Utilisez des conteneurs ou des exécutables invités pour héberger le code existant dans Service Fabric.  
3) Modernisez. Ajoutez de nouveaux microservices en plus du code en conteneur existant.  
4) Innovez. Décomposez l’application monolithique en microservices, selon vos besoins.  
5) Transformez vos applications en microservices. Transformez vos applications monolithiques existantes ou créez des applications Greenfield.

![Migration vers les microservices][Image3]

N’oubliez pas, vous pouvez _vous arrêter et reprendre à chacune de ces étapes*. Vous n’avez pas à passer immédiatement à l’étape suivante. 

Penchons-nous sur des exemples de chacune de ces étapes.

**Migrer**  
De nombreuses entreprises déplacent des applications monolithiques existantes vers des conteneurs pour deux raisons :

* Une réduction des coûts liée à la consolidation et à la suppression de matériel existant ou à des applications en cours d’exécution à une densité plus élevée.
* Un contrat de déploiement cohérent entre le développement et les opérations.

Les réductions de coûts sont simples. Chez Microsoft, de nombreuses applications existantes sont placées en conteneur, ce qui conduit à des millions de dollars d’économies. Un déploiement cohérent est plus difficile à évaluer, mais tout aussi important. Cela signifie que les développeurs peuvent choisir les technologies qui leur conviennent, mais les opérations n’accepteront qu’une seule méthode pour déployer et gérer les applications. Ainsi, les opérations n’ont pas à gérer la complexité liée à la prise en charge de nombreuses technologies sans obliger les développeurs à ne choisir que certaines d’entre elles. En principe, chaque application est placée dans des images de déploiement autonomes.

De nombreuses organisations ne vont pas plus loin. Elles bénéficient déjà des avantages liés aux conteneurs et Service Fabric offre l’expérience de gestion complète du déploiement, notamment les mises à niveau, le contrôle de version, les restaurations, le contrôle d’intégrité, etc.

**Moderniser**  
La modernisation consiste à ajouter de nouveaux services en plus du code en conteneur existant. Si vous souhaitez écrire du code, nous vous conseillons de suivre quelques étapes liées aux microservices. Cela peut consister à ajouter un nouveau point de terminaison d’API REST ou une nouvelle logique métier. De cette manière, vous commencez à développer de nouveaux microservices et vous vous exercez à leur développement et déploiement.

**Innovation**  
Les microservices vous permettent de mieux vous adapter aux changements de vos besoins. À ce stade, vous devez décider si vous devez commencer à fractionner l’application monolithique en services ou opter pour l’innovation. Un exemple : une base de données utilisée comme file d’attente de flux de travail devient un goulot d’étranglement qui ralentit le traitement. L’augmentation des demandes de workflow requiert une distribution du travail pour la mise à l’échelle. Concernant cette partie de l’application qui n’est pas mise à l’échelle, ou qui doit être mise à jour plus fréquemment, nous vous conseillons de la fractionner en microservices et d’opter pour l’innovation.

**Transformer des applications en microservices**  
Votre application est entièrement composée de (ou divisée en) microservices. Pour atteindre ce stade, vous avez suivi la procédure liée aux microservices. Vous pouvez commencer ici. Toutefois, effectuer cette opération sans l’aide d’une plate-forme de microservices demande un investissement important.

### <a name="are-microservices-right-for-my-application"></a>Les microservices conviennent-ils à mon application ?

Peut-être. Chez Microsoft, tandis que de plus en plus d’équipes développent en gardant le cloud à l’esprit pour des raisons métier, nombre d’entre elles ont pris conscience des avantages d’une approche de microservices. Bing, par exemple, utilise des microservices depuis des années. Pour les autres équipes, l’approche des microservices était une nouveauté. Les équipes trouvaient qu’il existait des problèmes difficiles à résoudre en dehors de leurs principaux domaines d’expertise. C’est pourquoi Service Fabric est très populaire en tant que technologie pour créer des services.

L’objectif de Service Fabric consiste à réduire la complexité de la création d’applications de microservices afin de vous éviter des refontes coûteuses. Commencez petit, mettez à l’échelle lorsque c’est nécessaire, déconseillez les services inutiles, ajoutez-en de nouveaux, évoluez avec l’utilisation des clients. Nous savons également qu’il existe de nombreux autres problèmes à résoudre pour que les microservices soient plus abordables pour la plupart des développeurs. Les conteneurs et le modèle de programmation Actor sont des exemples de quelques étapes dans cette direction. Nous sommes convaincus que plusieurs innovations vont apparaître pour faciliter une approche de microservices.

## <a name="next-steps"></a>Étapes suivantes

* [Microservices : An application revolution powered by the cloud](https://azure.microsoft.com/blog/microservices-an-application-revolution-powered-by-the-cloud/).
* [Centre des architectures Azure : Création de microservices sur Azure](/azure/architecture/microservices/)
* [Bonnes pratiques relatives aux applications et aux clusters Azure Service Fabric](service-fabric-best-practices-overview.md)
* [Présentation de la terminologie Service Fabric](service-fabric-technical-overview.md)

[Image1]: media/service-fabric-overview-microservices/monolithic-vs-micro.png
[Image2]: media/service-fabric-overview-microservices/statemonolithic-vs-micro.png
[Image3]: media/service-fabric-overview-microservices/microservices-migration.png
