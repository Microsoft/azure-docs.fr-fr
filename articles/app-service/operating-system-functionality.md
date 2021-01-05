---
title: Fonctionnalités du système d’exploitation
description: Découvrez les fonctionnalités du système d’exploitation dans Azure App Service sur Windows. Découvrez les types de fichiers, de réseaux et d’accès au registre dont bénéficie votre application.
ms.assetid: 39d5514f-0139-453a-b52e-4a1c06d8d914
ms.topic: article
ms.date: 10/30/2018
ms.custom: seodec18
ms.openlocfilehash: 949e408544e25cb55622cf2a1b1d2dddb92350a6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001505"
---
# <a name="operating-system-functionality-on-azure-app-service"></a>Fonctionnalités de système d’exploitation sur Azure App Service
Cet article décrit les fonctionnalités de système d’exploitation communes de base accessibles à toutes les applications Windows exécutées sur [Azure App Service](./overview.md). Ces fonctionnalités englobent notamment l’accès aux fichiers, l’accès réseau et l’accès au registre, ainsi que les journaux d’activité et événements de diagnostic. 

> [!NOTE] 
> Les [applications Linux](overview.md#app-service-on-linux) dans App Service s’exécutent dans leurs propres conteneurs. Aucun accès au système d’exploitation hôte n’est autorisé, car vous n’avez pas d’accès racine au conteneur. De même, pour les [applications qui s’exécutent dans des conteneurs Windows](quickstart-custom-container.md?pivots=container-windows), vous disposez d’un accès administratif au conteneur mais pas d’un accès au système d’exploitation hôte. 
>

<a id="tiers"></a>

## <a name="app-service-plan-tiers"></a>Niveaux des plans App Service
App Service exécute les applications des clients dans un environnement d’hébergement mutualisé. Les applications déployées dans les niveaux **Gratuit** et **Partagé** sont exécutées dans des processus de travail sur des machines virtuelles partagées, alors que les applications déployées dans les niveaux **Standard** et **Premium** sont exécutées sur des machines virtuelles spécialement dédiées aux applications associées à un client individuel.

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

Dans la mesure où App Service offre une expérience de mise à l’échelle transparente entre différents niveaux, la configuration de sécurité appliquée aux applications App Service reste la même. Cela empêche les changements de comportement soudains des applications, qui peuvent se traduire par un échec imprévu quand un plan App Service passe d’un niveau à un autre.

<a id="developmentframeworks"></a>

## <a name="development-frameworks"></a>Infrastructures de développement
Les niveaux tarifaires d’App Service déterminent la quantité de ressources de calcul (processeur, stockage sur disque, mémoire et sortie réseau) accessible aux applications. Toutefois, l’étendue des fonctionnalités d’infrastructure accessibles aux applications reste la même, quel que soit le niveau de mise à l’échelle.

App Service prend en charge différentes infrastructures de développement, telles qu’ASP.NET, ASP classique, node.js, PHP et Python (toutes exécutées en tant qu’extensions sous IIS). Pour simplifier et normaliser la configuration de sécurité, les applications App Service exécutent généralement les différentes infrastructures de développement avec leurs paramètres par défaut. Une approche de la configuration des applications aurait pu consister à personnaliser les fonctionnalités et la surface d’exposition des API de chaque infrastructure de développement. L’approche retenue pour App Service est plus générale puisqu’elle offre une base commune de fonctionnalités de système d’exploitation, quelle que soit l’infrastructure de développement d’une application.

Les sections suivantes présentent brièvement les principaux types de fonctionnalités de système d’exploitation accessibles aux applications App Service.

<a id="FileAccess"></a>

## <a name="file-access"></a>Accès aux fichiers
Différents lecteurs sont disponibles dans App Service, notamment les lecteurs locaux et les lecteurs réseau.

<a id="LocalDrives"></a>

### <a name="local-drives"></a>Lecteurs locaux
App Service est un service qui s’exécute essentiellement au sommet de l’infrastructure PaaS (Platform as a Service) d’Azure. Par conséquent, les disques locaux « associés » à une machine virtuelle correspondent aux types de lecteurs accessibles à tous les rôles de travail exécutés sous Azure. notamment :

- Un lecteur de système d’exploitation (lecteur D:\)
- Un lecteur d’application contenant les fichiers cspkg du package Azure exclusivement utilisés par App Service (et non accessibles aux clients)
- Un lecteur « utilisateur » (lecteur C:\) dont la taille dépend de celle de la machine virtuelle 

Il est important de surveiller l’utilisation du disque à mesure du développement de votre application. Si le quota du disque est atteint, le fonctionnement de votre application peut s’en trouver modifié. Par exemple : 

- L’application peut lever une erreur indiquant que l’espace est insuffisant sur le disque.
- Vous pouvez voir des erreurs de disque quand vous accédez à la console Kudu.
- Le déploiement à partir d’Azure DevOps ou de Visual Studio peut échouer avec `ERROR_NOT_ENOUGH_DISK_SPACE: Web deployment task failed. (Web Deploy detected insufficient space on disk)`.
- Votre application peut subir un ralentissement des performances.

<a id="NetworkDrives"></a>

### <a name="network-drives-unc-shares"></a>Lecteurs réseau (partages UNC)
Pour simplifier le déploiement et la maintenance d’applications, App Service stocke tout le contenu utilisateur sur un ensemble de partages UNC. Ce modèle s’adapte bien au modèle commun de stockage de contenu utilisé par les environnements d’hébergement web locaux dotés de plusieurs serveurs à charge équilibrée. 

App Service intègre un certain nombre de partages UNC créés dans chaque centre de données. Un certain pourcentage du contenu de l'ensemble des utilisateurs de chaque centre de données est alloué à chacun des partages UNC. En outre, le contenu de tous les fichiers associés à l'abonnement d'un client est systématiquement placé sur le même partage UNC. 

En raison du mode de fonctionnement des services Azure, la machine virtuelle chargée d’héberger un partage UNC changera au fil du temps. Les partages UNC seront en effet associés à différentes machines virtuelles à mesure que celles-ci seront intégrées et écartées lors de l’évolution normale des opérations Azure. C'est la raison pour laquelle les applications ne doivent jamais partir du principe que les informations machine présentes dans un chemin de fichier UNC resteront stables au fil du temps. Elles doivent utiliser le *faux* chemin d’accès absolu **D:\home\site** fourni par App Service. Ce faux chemin d’accès absolu offre une méthode « portable » qui n’est pas propre à l’application ni à l’utilisateur pour faire référence à une application particulière. L’utilisation du chemin **D:\home\site** permet de transférer les fichiers partagés d’une application vers une autre sans avoir à configurer un nouveau chemin d’accès absolu à chaque transfert.

<a id="TypesOfFileAccess"></a>

### <a name="types-of-file-access-granted-to-an-app"></a>Types d'accès aux fichiers octroyés à une application
L’abonnement de chaque client comporte une structure de répertoires réservée sur un partage UNC spécifique dans un centre de données. Un client peut disposer de plusieurs applications créées dans un centre de données spécifique, ainsi tous les répertoires appartenant à un abonnement client sont créés sur le même partage UNC. Le partage peut inclure des répertoires tels que ceux destinés au contenu, aux journaux d’activité d’erreurs et de diagnostic et aux précédentes versions de l’application créées par le contrôle du code source. Les répertoires de l’application d’un client sont accessibles en lecture et en écriture lors de l’exécution par le code d’application de l’application.

Sur les lecteurs locaux associés à la machine virtuelle qui exécute une application, App Service réserve une partie de l’espace sur le lecteur C:\ au stockage local temporaire propre à l’application. Bien qu’une application dispose d’un accès en lecture/écriture complet à son propre stockage local temporaire, ce stockage n’est pas destiné à être directement utilisé par le code d’application. Il a pour but de fournir un stockage de fichiers temporaire à IIS et aux infrastructures d'applications Web. App Service limite également le stockage local temporaire accessible à chaque application pour empêcher que différentes applications n’utilisent une part excessive du stockage de fichiers local.

Deux exemples illustrent la façon dont App Service utilise le stockage local temporaire : le répertoire de fichiers temporaires ASP.NET et le répertoire de fichiers compressés IIS. Le système de compilation ASP.NET utilise le répertoire de fichiers temporaires « Temporary ASP.NET Files » comme emplacement de cache de compilation temporaire. IIS utilise le répertoire de fichiers compressés « IIS Temporary Compressed Files » pour stocker les réponses compressées. Ces deux types d’utilisation de fichiers (et d’autres) sont de nouveau mappés dans App Service sur un stockage local temporaire par application. Ce nouveau mappage préserve les fonctionnalités.

Chaque application dans App Service est exécutée sous une identité de processus de travail à faibles privilèges unique et aléatoire appelée « identité du pool d’applications », comme décrit plus en détail ici : [https://www.iis.net/learn/manage/configuring-security/application-pool-identities](https://www.iis.net/learn/manage/configuring-security/application-pool-identities). Le code d'application utilise cette identité pour l'accès en lecture seule de base au lecteur du système d'exploitation (lecteur D:\). Le code d'application peut ainsi dresser la liste les structures de répertoires communes et lire les fichiers communs sur le lecteur du système d'exploitation. Bien que ce niveau d'accès puisse sembler étendu, les mêmes répertoires et fichiers sont accessibles lorsque vous approvisionnez un rôle de travail dans le service hébergé Azure et lisez le contenu du lecteur. 

<a name="multipleinstances"></a>

### <a name="file-access-across-multiple-instances"></a>Accès aux fichiers sur plusieurs instances
Le répertoire de base comprend le contenu d’une application, et le code d’application peut y accéder en écriture. Si une application est exécutée sur plusieurs instances, le répertoire de base est partagé entre toutes les instances afin que celles-ci voient toutes le même répertoire. Ainsi, par exemple, si une application enregistre des fichiers chargés dans le répertoire de base, ces fichiers sont immédiatement accessibles à toutes les instances. 

<a id="NetworkAccess"></a>

## <a name="network-access"></a>Accès réseau
Le code d’application peut utiliser les protocoles TCP/IP et UDP pour établir des connexions réseau sortantes avec des points de terminaison accessibles par Internet qui exposent des services externes. Les applications peuvent utiliser ces mêmes protocoles pour se connecter aux services Azure, par exemple en établissant des connexions HTTPS à Azure SQL Database.

La capacité des applications à établir une connexion de bouclage locale et à écouter sur le socket de bouclage local correspondant est également limitée. Cette fonctionnalité a pour principal objectif d'activer les applications configurées pour écouter sur les sockets de bouclage locaux. Chaque application voit une connexion de bouclage « privée ». Une application « A » ne peut pas écouter sur un socket de bouclage local établi par une application « B ».

Les canaux nommés sont également pris en charge en tant que mécanisme de communication inter-processus (IPC) entre différents processus qui exécutent collectivement une application. Par exemple, le module FastCGI d'IIS utilise des canaux nommés pour coordonner les processus individuels qui exécutent les pages PHP.

<a id="Code"></a>

## <a name="code-execution-processes-and-memory"></a>Exécution de code, processus et mémoire
Comme mentionné précédemment, les applications sont exécutées dans des processus de travail à faibles privilèges qui utilisent une identité de pool d’applications aléatoire. Le code d'application a accès à l'espace mémoire associé au processus de travail ainsi qu'aux processus enfants qui peuvent être générés par les processus CGI ou par d'autres applications. Toutefois, une application n’a pas accès à la mémoire ou aux données d’une autre application, même si celles-ci se trouvent sur la même machine virtuelle.

Les applications peuvent exécuter des scripts ou des pages créés avec des infrastructures de développement Web prises en charge. App Service ne configure aucun paramètre d’infrastructure Web sur des modes plus restreints. Par exemple, les applications ASP.NET exécutées sur App Service sont exécutées avec une confiance totale par opposition à un mode de confiance plus limité. Les infrastructures Web, telles que ASP classique et ASP.NET, peuvent appeler des composants COM « in-process » (mais pas des composants COM hors processus), comme ADO (ActiveX Data Objects), qui sont enregistrés par défaut sur le système d'exploitation Windows.

Les applications peuvent générer et exécuter un code arbitraire. Une application est autorisée à effectuer des opérations telles que la génération d'une interface de commande ou l'exécution d'un script PowerShell. Toutefois, bien que du code et des processus arbitraires puissent être générés à partir d'une application, les programmes et scripts exécutables restent limités aux privilèges accordés au pool d'applications parent. Par exemple, une application peut générer un exécutable pour passer un appel HTTP sortant, mais ce même exécutable ne peut pas tenter de dissocier l’adresse IP d’une machine virtuelle de sa carte réseau. Les appels réseau sortants vers le code à faibles privilèges sont autorisés, mais toute tentative de reconfiguration des paramètres réseau sur une machine virtuelle nécessite des privilèges d'administration.

<a id="Diagnostics"></a>

## <a name="diagnostics-logs-and-events"></a>Journaux d’activité et événements de diagnostic
Certaines applications peuvent également tenter d'accéder aux informations de journalisation. Les types d’informations de journalisation accessibles au code exécuté dans App Service englobent les informations de diagnostic et de journalisation générées par une application et sont également facilement accessibles à une application. 

Par exemple, les journaux d’activité HTTP W3C générés par une application active sont disponibles dans un répertoire de journaux d’activité du partage réseau créé pour l’application ou dans un stockage d’objets blob si le client a configuré la journalisation W3C pour le stockage. Cette dernière option permet de collecter de gros volumes de journaux d’activité sans risquer de dépasser les limites de stockage de fichiers associées à un partage réseau.

De même, les informations de diagnostic en temps réel des applications .NET peuvent être consignées via l’infrastructure de suivi et de diagnostic .NET, avec possibilité de consigner les informations de suivi sur le partage réseau de l’application ou à un emplacement de stockage d’objets blob.

Les éléments de journalisation et de suivi des diagnostics non accessibles aux applications sont les événements ETW Windows et les journaux d’événements Windows communs (par exemple, les journaux d’événements système, d’applications et de sécurité). Dans la mesure où les informations de suivi ETW sont potentiellement visibles sur toute la machine (avec les ACL qui conviennent), l'accès en écriture aux événements ETW est bloqué. Les développeurs peuvent constater que les appels d’API à des événements ETW en lecture et écriture, ainsi qu’aux journaux des événements Windows communs semblent fonctionner, car App Service « simule » les appels pour donner l’impression qu’ils aboutissent. En réalité, le code d'application n'a pas accès à ces données d'événements.

<a id="RegistryAccess"></a>

## <a name="registry-access"></a>Accès au registre
Les applications disposent d'un accès en lecture seule à la majeure partie du registre (mais pas à la totalité de celui-ci) de la machine virtuelle sur laquelle elles sont exécutées. En pratique, cela signifie que les clés de registre qui permettent un accès en lecture seule au groupe Utilisateurs local sont accessibles aux applications. La ruche HKEY\_CURRENT\_USER est une zone du registre actuellement non prise en charge pour l’accès en lecture ou en écriture.

L'accès en écriture au registre est bloqué, y compris l'accès aux éventuelles clés de registre par utilisateur. Dans un environnement Azure, une application ne doit jamais compter sur l'accès en écriture au registre, car les applications peuvent être (et sont) migrées entre différentes machines virtuelles. Le seul stockage toujours accessible en écriture sur lequel une application peut compter est la structure des répertoires de contenu par application stockée dans les partages UNC App Service. 

## <a name="remote-desktop-access"></a>Accès au Bureau à distance

App Service ne fournit pas l’accès au Bureau à distance aux instances de machine virtuelle.

## <a name="more-information"></a>Informations complémentaires

[Bac à sable Azure App Service](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox) : informations les plus récentes sur l’environnement d’exécution d’App Service. L’équipe de développement d’App Service gère cette page directement.