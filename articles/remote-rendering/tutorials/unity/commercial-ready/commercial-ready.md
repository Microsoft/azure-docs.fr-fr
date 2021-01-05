---
title: Création d’une application Azure Remote Rendering prête à la commercialisation
description: Stratégies et considérations relatives à la création d’une application Azure Remote Rendering prête à la commercialisation
author: FlorianBorn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.openlocfilehash: 7258b37c9d92f25227eac678dde77963e644e64b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483315"
---
# <a name="tutorial-creating-a-commercial-ready-azure-remote-rendering-application"></a>Tutoriel : Création d’une application Azure Remote Rendering prête à la commercialisation

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Gestion des sessions pour les applications commerciales
> * Suivi des sessions pour la facturation
> * Optimisation de l’expérience utilisateur par rapport au temps de chargement des sessions
> * Considérations relatives à la latence du réseau

## <a name="prerequisites"></a>Prérequis

* Ce tutoriel s’appuie sur le [Tutoriel : Sécurisation d’Azure Remote Rendering et du stockage de modèles](../security/security.md).

## <a name="introduction-to-commercial-readiness"></a>Présentation de l’état de préparation à la commercialisation

Azure Remote Rendering développe le champ des possibles de la réalité mixte. Une fois les principes de base intégrés à votre solution, un certain nombre de considérations supplémentaires sont à prendre en compte pour que vous puissiez garantir une solution sécurisée, évolutive et prête à fournir de la valeur.

Ce module présente quelques fonctionnalités supplémentaires dont vous devrez peut-être tenir compte pour votre application commerciale.

Afin d’obtenir une vue d’ensemble étendue des bonnes pratiques en matière d’architecture système, consultez :

* [Centre des architectures Azure](/azure/architecture/)
* [Guide de démarrage pour les développeurs Azure](../../../../guides/developer/azure-developer-guide.md)

## <a name="analytics"></a>Analytics

L’intégration d’outils d’analytique peut faciliter la gestion, le suivi et améliorer votre solution.

Pour obtenir la liste complète des ressources d’analytique disponibles, visitez le site :

* [Services d’analytique Azure](https://azure.microsoft.com/product-categories/analytics/)

### <a name="tracking-usage-for-billing"></a>Suivi de l’utilisation pour la facturation

Le suivi de la consommation d’Azure Remote Rendering par différentes équipes internes ou clients externes se révèle un élément primordial, surtout dans des situations à plusieurs locataires.

Pour ce faire, Azure propose un service nommé « étiquetage des ressources », qui impute la consommation du service Azure Remote Rendering à chaque client.

Pour plus d’informations sur le nommage et l’étiquetage des ressources, il convient de commencer par :

* [Guides de décision concernant le nommage et l’étiquetage des ressources](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=%2fazure%2fazure-resource-manager%2fmanagement%2ftoc.json)

### <a name="diagnostics"></a>Diagnostics

De puissants outils, tels que le suivi d’événements pour Windows (ETW) et la journalisation du suivi des événements (ETL) facilitent la génération d’événements de trace au sein même de votre application, et peuvent aider dans le diagnostic des problèmes de réseau, d’ingestion de contenu, de session, d’application et d’autres problèmes pouvant survenir lors du déploiement d’une solution commerciale.

Pour plus d’informations, consultez :

* [Créer des traces de performances côté client](../../../how-tos/performance-tracing.md)
* [Guide pratique permettant de collecter les données du suivi d’événements pour Windows (ETW)](/visualstudio/profiling/how-to-collect-event-tracing-for-windows-etw-data)
* [Utilisation du portail d’appareil Windows : Journalisation](/windows/mixed-reality/using-the-windows-device-portal)

### <a name="usage-analysis"></a>Analyse de l’utilisation

Azure Application Insights vous permet de comprendre comment les utilisateurs se servent de votre application Azure Remote Rendering. Chaque fois que vous mettez à jour votre application, vous pouvez évaluer son bon fonctionnement pour les utilisateurs, et ainsi améliorer votre solution en conséquence. Grâce à ces informations, vous pouvez prendre des décisions pilotées par des données quant à vos prochains cycles de développement.

Pour plus d’informations, consultez :

* [Analyse de l’utilisation avec Application Insights](../../../../azure-monitor/app/usage-overview.md)

## <a name="fast-startup-time-strategies"></a>Stratégies de temps de démarrage rapide

Votre cas d’usage peut exiger un démarrage rapide, entre le lancement de l’application et l’affichage du modèle 3D. Par exemple, pendant une réunion importante pour laquelle il est essentiel que tout soit prêt et opérationnel à l’avance. Un autre exemple s’illustre dans l’étude d’un modèle 3D de CAO, où une itération de conception prompte entre une application CAO et une réalité mixte est décisive en termes de performances.

Azure Remote Rendering demande des modèles 3D prétraités, alors qu’Azure prend actuellement quelques minutes pour créer une session et charger un modèle destiné au rendu. Rendre ce processus aussi transparent et rapide que possible nécessite la préparation anticipée des données du modèle 3D et de la session ARR (Azure Remote Rendering).

Les suggestions partagées ici ne font pour l’instant pas partie d’Azure Remote Rendering standard, mais vous pouvez les implémenter vous-même pour accélérer vos propres démarrages.

### <a name="initiate-early"></a>Initialisation préalable

Pour réduire le temps de démarrage, la solution la plus simple consiste à déplacer la création et l’initialisation de la session le plus tôt possible dans le workflow de l’utilisateur. Une stratégie s’attache à initialiser la session dès qu’il est su qu’une session ARR sera nécessaire. C’est souvent le cas lorsque l’utilisateur commence à charger un modèle 3D dans le stockage Blob Azure pour l’utiliser avec Azure Remote Rendering. Dans ce cas, la création et l’initialisation de session peuvent être lancées en même temps que le chargement du modèle 3D, afin que les deux flux de travail s’exécutent en parallèle.

Ce processus peut être simplifié davantage en veillant à ce que les conteneurs d’entrée et de sortie du stockage Blob Azure choisis se trouvent dans le même centre de données régional que la session Azure Remote Rendering.

### <a name="scheduling"></a>Planification

Si vous savez que vous aurez besoin d’Azure Remote Rendering ultérieurement, vous pouvez planifier une date et une heure précises pour démarrer la session Azure Remote Rendering.

Il est possible de proposer cette option par le biais d’un portail web dans lequel les utilisateurs peuvent à la fois télécharger un modèle 3D et planifier un moment auquel l’afficher par la suite. C’est également un bon endroit pour demander la précision d’autres préférences, telles que le rendu [*Standard*](../../../reference/vm-sizes.md) ou [*Premium*](../../../reference/vm-sizes.md). Le rendu *Premium* peut convenir s’il est préférable d’afficher une combinaison de ressources dans laquelle la taille idéale est plus difficile à déterminer automatiquement, ou s’il faut s’assurer que la région Azure dispose de machines virtuelles disponibles à ce moment-là.

### <a name="session-pooling"></a>Regroupement de sessions

Dans les situations les plus exigeantes, une autre option se présente avec le regroupement de sessions, au sein duquel une ou plusieurs sessions sont créées et initialisées à tout moment. La création d’un pool de sessions est destinée à l’utilisation immédiate par un utilisateur demandeur. L’inconvénient de cette approche réside dans la facturation du service qui démarre aussitôt la machine virtuelle initialisée. Il n’est pas toujours rentable de garder en permanence un pool de sessions en cours d’exécution, mais en s’appuyant sur l’analytique, il peut être possible de prévoir les pics de charge, ou d’associer ce pool à la stratégie de planification ci-dessus pour prédire le moment où des sessions seront nécessaires et ajuster ainsi le pool de sessions en conséquence.

Cette stratégie permet également d’optimiser le choix entre les sessions *Standard* et *Premium* d’une manière plus dynamique, car il peut s’avérer beaucoup plus rapide de basculer entre les deux types au sein d’une même session utilisateur, par exemple lorsqu’un modèle de complexité *Premium* est affiché en premier, suivi d’un modèle pouvant fonctionner en session *Standard*. Si ces sessions utilisateur sont relativement longues, d’importantes économies peuvent en résulter.

Pour plus d’informations sur les sessions Azure Remote Rendering, consultez :

* [Sessions Remote Rendering](../../../concepts/sessions.md)

## <a name="standard-vs-premium-server-size-routing-strategies"></a>Stratégies de routage sur une machine virtuelle Standard ou Stratégies de routage de taille de serveur Premium

La nécessité de décider de la création d’une taille de serveur *Standard* ou *Premium* représente un défi au niveau de la conception de votre expérience utilisateur et de votre système de bout en bout. Bien que l’utilisation de sessions *Premium* uniquement soit une option, les sessions *Standard* utilisent beaucoup moins de ressources de calcul Azure, et sont moins onéreuses que la solution *Premium*. Voici donc une excellente motivation pour utiliser des sessions *Standard* dans la mesure du possible, et utiliser seulement la solution *Premium* en cas de nécessité.

Nous partageons ici plusieurs options, de la moins aboutie à la plus complète, pour répondre au désir de gérer des choix de session.

### <a name="use-only-standard-or-premium"></a>Utiliser uniquement Standard ou Premium

Si vous êtes certain que vos besoins seront *toujours* inférieurs au seuil instauré entre les niveaux *Standard* et *Premium*, votre décision s’en trouvera grandement simplifiée. Utilisez simplement *Standard*. Gardez toutefois à l’esprit que l’impact sur l’expérience utilisateur est conséquente si la somme totale de la complexité des ressources chargées est rejetée comme étant trop complexe pour une session *Standard*.

De même, si vous prévoyez qu’une grande partie des utilisations dépassera le seuil entre les niveaux *Standard* et *Premium*, ou si le coût n’est pas un facteur déterminant dans votre cas d’usage, le choix systématique de la solution *Premium* demeure également une option qui vous permet de simplifier les choses.

### <a name="ask-the-user"></a>Demander à l’utilisateur

Si vous souhaitez vraiment prendre en charge les deux solutions *Standard* et *Premium*, le moyen le plus simple pour déterminer le type de session de machine virtuelle à instancier consiste à le demander à l’utilisateur quand il sélectionne les ressources 3D à afficher. Le difficulté de cette approche vient de ce qu’elle implique la compréhension par l’utilisateur de la notion de complexité de la ressource 3D, ou même des quelques ressources qui seront affichées. En général, c’est la raison pour laquelle elle n’est pas recommandée. Si l’utilisateur sélectionne le mauvais choix en optant pour *Standard*, l’expérience utilisateur obtenue peut être compromise à un moment inopportun.

### <a name="analyze-the-3d-model"></a>Analyser le modèle 3D

Une autre approche relativement simple consiste à analyser la complexité des ressources 3D sélectionnées. Si la complexité du modèle est inférieure au seuil pour *Standard*, lancez une session *Standard*, sinon initiez une session *Premium*. Ici, le défi réside dans le fait qu’une seule session peut finalement être utilisée pour afficher plusieurs modèles, parmi lesquels certains peuvent dépasser le seuil de complexité d’une session *Standard*, entraînant l’incapacité à utiliser la même session de façon fluide pour une séquence de ressources 3D différentes.

### <a name="automatic-switching"></a>Basculement automatique

Le basculement automatique entre les sessions *Standard* et *Premium* peut s’avérer très judicieux dans une conception de système qui comprend également un regroupement de sessions. Cette stratégie permet une optimisation plus poussée de l’exploitation des ressources. À mesure que l’utilisateur charge des modèles pour l’affichage, la complexité est déterminée et la taille de session correcte est demandée auprès du service de regroupement de sessions.

## <a name="working-with-networks"></a>Utilisation des réseaux

### <a name="diagnostics"></a>Diagnostics

Azure Remote Rendering nécessite une connexion Internet rapide, à faible latence. La qualité du réseau de l’utilisateur peut avoir un impact considérable sur la qualité de l’expérience. Étant donné que vos clients sont susceptibles d’avoir des configurations réseau différentes, et que la latence du réseau est rarement médiocre, les outils de diagnostic s’avèrent donc essentiels.  

Afin que vous puissiez garantir une expérience de qualité supérieure de façon uniforme, nous vous recommandons d’intégrer à vos applications Azure Remote Rendering des outils d’analytique côté serveur et côté client. Vous serez ainsi pourvus d’informations qui vous seront utiles pour diagnostiquer et atténuer les problèmes de réseau auxquels vos clients peuvent être confrontés.

### <a name="client-network-configurations"></a>Configurations du réseau client

L’un des plus grands défis existant dans le développement de solutions de collaboration robustes, déployées dans un large éventail d’environnements d’entreprise, est actuellement en préparation pour les différentes topologies réseau et configurations de pare-feu d’entreprise que vos clients peuvent utiliser.

Nombreuses sont les entreprises qui bloquent l’intégralité du trafic pair à pair dans un réseau local. Il est donc compliqué de tirer parti de la simplicité et de l’expérience utilisateur rationalisée présentes dans la découverte automatique de réseaux locaux pour établir une session partagée locale entre toutes les instances découvertes de votre application de réalité mixte.

D’autres points de défaillance potentiels sont les routeurs configurés pour limiter intentionnellement la bande passante et les pare-feu qui bloquent la plupart des ports TCP/IP.

Chaque fois que vous envisagez d’utiliser Azure Remote Rendering sur un réseau inconnu, nous vous conseillons de suivre les recommandations suivantes :

* Fournir une liste de contrôle de réunion préparatoire pour évaluer la disponibilité du réseau.
* Vous assurer que le centre de données régional approprié peut traiter la demande.
* Laisser suffisamment de temps pour diagnostiquer des problèmes.
* Apporter un point d’accès sans fil mobile avec un plan de données à bande passante élevée comme dispositif supplémentaire.

### <a name="end-to-end-bandwidth"></a>Bande passante de bout en bout

Il est important d’évaluer les capacités de la bande passante sur chaque tronçon du réseau pouvant exister entre la machine virtuelle Azure Remote Rendering et le client final. Gardez à l’esprit que le segment réseau qui part du centre de données Azure pour rejoindre le fournisseur de services Internet du client peut se révéler un facteur de limitation plus important que celui qui relie le fournisseur de services Internet au client. Le test de vitesse de téléchargement d’objets blob peut servir pour diagnostiquer de tels problèmes.

### <a name="bandwidth-competition"></a>Compétition pour la bande passante

Lorsque vous concevez votre application de réalité mixte, n’oubliez pas que certaines fonctionnalités de l’application peuvent entrer en concurrence avec Azure Remote Rendering pour la bande passante. L’exemple probablement le plus inattendu survient lorsqu’un nombre important de participants réunis dans une même pièce s’attendent à tous utiliser ARR en même temps pour afficher une ressource 3D. Chaque tronçon du flux de données réseau devra donc disposer de la capacité de transporter la somme totale de tous les flux ARR combinés.

À citer comme autres exemples : une vidéo envoyée en streaming, des téléchargements simultanés en arrière-plan de contenus connexes autres et la conversation vocale, surtout en présence d’un nombre important de participants alors que le système utilise une approche distribuée pair à pair, contrairement à un serveur de mixage audio dans l’approche intermédiaire.

Pour plus d’informations sur l’analytique réseau, consultez :

* [Test de vitesse de téléchargement du stockage Blob Azure](https://www.azurespeed.com/Azure/Download)
* [Statistiques de latence aller-retour réseau Azure](../../../../networking/azure-network-latency.md)
* [Traces de performances côté serveur](../../../overview/features/performance-queries.md)
* [Traces de performances côté client](../../../how-tos/performance-tracing.md)

## <a name="collaboration-considerations"></a>Considérations sur la collaboration

Certaines utilisations parmi les plus intéressantes d’Azure Remote Rendering impliquent la collaboration entre plusieurs participants qui visualisent simultanément la même expérience 3D. Dans ces sessions partagées, il est important de reconnaître que chaque participant aura besoin d’une session Azure Remote Rendering unique, que ces participants se trouvent ou non dans un même lieu, sur le même réseau.

Cela est avéré, car chaque participant voit en fait la même expérience à partir de points d’observation différents, ce qui demande que les mêmes ressources 3D soient rendues à partir de chacune de ces perspectives simultanément.

### <a name="multiple-azure-remote-rendering-sessions"></a>Plusieurs sessions Azure Remote Rendering

Si vous envisagez de prendre en charge des expériences partagées avec Azure Remote Rendering, les systèmes que vous mettez en place pour créer et gérer des sessions ARR devront être préparés afin de lancer plusieurs sessions. Ces sessions devront peut-être être initialisées dans différents centres de données Azure si les participants sont dispersés géographiquement.

Votre système doit également gérer la possibilité qu’un ou plusieurs participants puissent se trouver dans une région géographique qui n’est pas actuellement prise en charge par Azure Remote Rendering, ou qui ne dispose pour le moment d’aucune instance de machine virtuelle Azure Remote Rendering disponible.

Cette gestion de plusieurs sessions simultanées peut être rationalisée davantage lorsqu’elle est associée à un regroupement de sessions et à d’autres stratégies présentées dans ce document.

### <a name="azure-blob-storage-considerations"></a>Considérations relatives au stockage

Toutes les sessions ARR simultanées peuvent référencer le même URI SAS pour le modèle converti à afficher. Il est ainsi rendu possible de charger et de convertir les éléments 3D souhaités une seule fois, puis de les partager dans toutes les sessions. Cela est particulièrement vrai lorsque les participants sont colocalisés et qu’ils utilisent le même centre de données, où n’existe aucun souci de performance lié au fait que le stockage Blob Azure soit situé dans un autre centre de données que celui du serveur Azure Remote Rendering et de l’utilisateur.

Si les ressources 3D sont généralement chargées pour une seule session d’affichage, puis ignorées, par exemple dans une session de révision de conception, la région géographique du stockage Blob Azure par rapport au serveur Azure Remote Rendering est également moins critique.

Toutefois, pour les composants 3D qui seront utilisés à plusieurs reprises, par exemple dans un cas d’usage de formation, nous vous recommandons de conserver les ressources 3D prêtes à l’emploi dans le stockage d’objets blob de chaque centre de données régional dans lequel vous envisagez d’utiliser Azure Remote Rendering. Cette opération peut être automatisée à l’aide de la redondance du stockage Azure. Le CDN est souvent utilisé à cette fin, mais ce n’est pas encore une option pour Azure Remote Rendering.

Pour plus d'informations :

* [Expériences partagées dans la réalité mixte](/windows/mixed-reality/shared-experiences-in-mixed-reality)
* [Redondance de Stockage Azure](../../../../storage/common/storage-redundancy.md)

## <a name="managing-model-access"></a>Gestion de l’accès aux modèles

Tirer pleinement parti d’Azure Remote Rendering nécessite d’apporter une attention particulière à l’infrastructure de bout en bout pour la gestion des modèles 3D.

L’un des avantages de l’utilisation d’Azure Remote Rendering réside dans les ressources 3D volumineuses qui n’ont jamais besoin d’être transmises directement à l’appareil de réalité mixte avant d’être affichées.  De plus, dès lors qu’une ressource 3D a été téléchargée et convertie en vue de son utilisation avec Azure Remote Rendering, n’importe quel nombre d’utilisateurs peut partager cette instance unique du modèle 3D.

### <a name="considerations-for-3d-model-access"></a>Considérations relatives à l’accès du modèle 3D

Voici quelques considérations primordiales à prendre en compte lorsque vous décidez de votre stratégie d’accès au modèle.

En fonction du cas d’usage anticipé, déterminez le meilleur emplacement ou la combinaison d’emplacements idéale qui permet à un utilisateur de sélectionner les ressources 3D à afficher. Voici quelques options courantes :

* Directement dans l’expérience de réalité mixte
* Via un portail web associé
* Dans une application mobile ou de bureau associée

Si votre cas d’usage présente des modèles d’utilisation dans lesquels la même ressource 3D peut être chargée plusieurs fois, le serveur principal effectuera le suivi des modèles qui sont déjà convertis pour une utilisation avec ARR, afin qu’un modèle ne soit pré-traité qu’une seule fois pour plusieurs sélections ultérieures. Un exemple de révision de conception représente l’endroit où une équipe accède à une ressource 3D d’origine, commune. Chaque membre de l’équipe est censé passer en revue le modèle à l’aide d’ARR, à un moment donné dans son flux de travail. Ainsi, seul le premier affichage déclenchera l’étape de pré-traitement. Les affichages suivants rechercheront le fichier post-traitement associé, dans le conteneur de sortie SAS.

Selon le cas d’usage, vous souhaiterez probablement déterminer la taille appropriée de serveur Azure Remote Rendering, *Standard* ou *Premium*, et la rendre potentiellement persistante, pour chaque ressource ou groupe de ressources 3D qui seront affichés ensemble au cours de la même session.  

### <a name="on-device-model-selection-list"></a>Liste de sélection de modèles sur l’appareil

Dans de nombreux cas d’usage, comme une formation, un guide de tâches ou une application marketing, l’ensemble des éléments 3D à afficher fréquemment dans Azure Remote Rendering peut être relativement statique. Dans ces situations, un ensemble organisé de ressources 3D peut être pré-converti et mis à disposition via une base de données qui contient les informations nécessaires pour remplir une liste de sélection de ressources organisées. Ces données peuvent ensuite être récupérées à partir de l’application de réalité mixte pour remplir un menu de sélection.

Vous pouvez aller encore un peu plus loin dans votre démarche en proposant également un moyen de charger des ressources 3D privées, propres à chaque individu ou groupe. Cette liste de ressources privées peut ensuite être combinée à la liste des ressources communes et organisées dans l’expérience utilisateur pour le choix des ressources 3D à afficher.

### <a name="on-device-onedrive-access"></a>Accès OneDrive sur l’appareil

Étant donné qu’un sélecteur de fichiers OneDrive est généré en mode natif sur les appareils de réalité mixte de Microsoft, la sélection de ressources 3D sur l’appareil à partir de OneDrive devient intéressante, surtout dans les cas d’usage où il est courant de charger des modèles 3D différents ou modifiés. Dans ce scénario, l’utilisateur sera amené à sélectionner une ou plusieurs ressources 3D via le sélecteur de fichiers OneDrive de votre application de réalité mixte. Les ressources 3D seront ensuite migrées vers un conteneur d’entrée SAS, puis converties vers un conteneur de sortie SAS et attachées à la session ARR. Dans l’idéal, l’application de réalité mixte appellera un processus basé sur le cloud pour effectuer ces étapes, au lieu de déplacer tous les bits, de OneDrive vers l’appareil, pour les retourner ensuite dans le stockage Blob Azure.

Cette approche peut être poussée plus loin en conservant une association entre des ressources 3D qui ont été précédemment consultées de telle sorte que, lorsque le même modèle est de nouveau choisi dans OneDrive, l’application peut contourner le processus de conversion et charger directement la ressource 3D convertie associée via son URI SAS.

Pour plus d'informations :

* [Modèle Microsoft Power Automate pour OneDrive destiné à la réplication dans le stockage Azure](https://flow.microsoft.com/galleries/public/templates/2f90b5d3-029b-4e2e-ad37-1c0fe6d187fe/when-a-file-is-uploaded-to-onedrive-copy-it-to-azure-storage-container/)
* [Vue d’ensemble de l’API de stockage de fichiers OneDrive](/graph/onedrive-concept-overview)

### <a name="direct-cad-access"></a>Accès direct CAO

Les révisions de conception de travaux CAO en cours constituent un cas d’usage intéressant pour la réalité mixte. Dans ce scénario, un temps de chargement extrêmement rapide entre l’ordinateur de bureau et la réalité mixte est essentiel. Une solution idéale pourrait impliquer le développement de plug-ins pour des applications CAO spécifiques. Ces plug-ins géreraient directement chaque aspect des processus de chargement, de conversion et d’affichage :

* Fournir une expérience utilisateur pour :
  * Coupler l’application CAO à un appareil de réalité mixte spécifique (une seule fois).
  * Demander que la géométrie sélectionnée soit affichée sur cet appareil de réalité mixte.
* Si elle n’est pas déjà en cours d’exécution, lancer la session Azure Remote Rendering afin qu’elle puisse assurer le traitement en parallèle pendant le chargement et la conversion du fichier de CAO
* Normaliser les données géométriques de CAO dans un des formats pris en charge par Azure Remote Rendering
* Transmettre directement les données normalisées au conteneur d’entrée du stockage Blob Azure
* Initialiser le processus de conversion du modèle
* Lier l’URI SAS du conteneur de sortie du modèle à la session Azure Remote Rendering
* Informer l’application de réalité mixte associée que le modèle est disponible et prêt à être affiché, et fournir l’URI SAS du conteneur de sortie afin que l’application puisse l’attacher à la session.

Une approche beaucoup plus simple mais un peu moins rationalisée pourrait automatiser le processus d’enregistrement du modèle 3D sur un disque dur local, puis lancer un processus de transmission de ce fichier enregistré au conteneur d’entrée SAS.

### <a name="azure-marketplace"></a>Place de marché Azure

Nombreux sont les clients d’entreprise qui exigent que votre système Azure Stack soit déployé sous leurs propres comptes et informations d’identification Azure pour des raisons de sécurité. Pour ce faire, vous pouvez envisager de mettre en package votre application managée Azure, afin qu’elle puisse être publiée sur la Place de marché Azure en tant qu’offre d’application Azure.

Pour plus d'informations :

* [Azure Marketplace](https://azure.microsoft.com/marketplace/)
* [Tutoriel : Publier des applications managées Azure sur la Place de marché](../../../../marketplace/create-new-azure-apps-offer.md)

### <a name="security"></a>Sécurité

Il est vital de créer votre solution Azure Remote Rendering de bout en bout, qui plus est lorsqu’il est question de sécurité. De nombreux aspects de la sécurité sont à prendre en compte dans la conception de votre solution de bout en bout, notamment :

* Stratégies d’authentification
* Gestion des accès : groupes, stratégies et autorisations
* Multilocation
* Stockage des données et chiffrement de transfert
* Jetons d’utilisation temporaire
* Attaques par déni de service distribué (DDoS)
* Détection de menaces
* VPN et réseaux sécurisés
* Pare-feux
* Gestion des certificats et des clés secrètes
* Attaques et vulnérabilité des applications

Pour l’authentification, il est avisé de déplacer autant que possible la gestion des sessions et l’authentification ARR sur un service web Azure. Une solution mieux gérée et plus sécurisée en résultera.

Pour plus d'informations :

* [Authentification auprès du service Azure AD](../../../../spatial-anchors/concepts/authentication.md?tabs=csharp#azure-ad-service-authentication)
* [Renforcer votre sécurité avec Azure](https://azure.microsoft.com/overview/security/)
* [Sécurité cloud](https://azure.microsoft.com/product-categories/security/)