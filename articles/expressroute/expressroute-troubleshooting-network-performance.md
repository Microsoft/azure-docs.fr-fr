---
title: 'Détecter un problème de performance de la liaison réseau : Azure'
description: Cette page fournit une méthode standardisée pour tester le niveau de performance de la liaison réseau Azure.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 12/20/2017
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: a021d658711e77c3e3be0df722223cefe506abba
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92204586"
---
# <a name="troubleshooting-network-performance"></a>Résolution des problèmes de performances réseau
## <a name="overview"></a>Vue d’ensemble
Azure met à votre disposition des moyens stables et rapides pour vous connecter à Azure à partir de votre réseau local. Les méthodes telles que le VPN de site à site et ExpressRoute sont couramment utilisées par les clients de grande et petite tailles pour exécuter leurs activités dans Azure. Mais que se passe-t-il quand les performances ne sont pas à la hauteur de vos attentes ou expériences passées ? Ce document peut vous aider à standardiser la façon de tester et de planifier votre environnement.

Ce document vous montre comment tester la latence réseau et la bande passante entre deux hôtes de façon aisée et cohérente. Il fournit également des conseils sur la façon d’examiner le réseau Azure et d’isoler les points problématiques. Le script PowerShell et les outils présentés nécessitent deux hôtes sur le réseau (à chaque extrémité de la liaison testée). L’un des hôtes doit être doté de Windows Server ou Windows Desktop, l’autre peut être doté de Windows ou de Linux. 

>[!NOTE]
>L’approche de résolution des problèmes, les outils et les méthodes utilisés sont des préférences personnelles. Ce document décrit les outils et l’approche dont je me sers souvent. Il est probable que votre approche soit différente ; il est en effet normal d’envisager des approches différentes pour résoudre des problèmes. Toutefois, si vous n’avez pas d’approche établie, ce document peut vous initier à l’élaboration de vos propres méthodes, outils et préférences en vue de résoudre les problèmes liés au réseau.
>
>

## <a name="network-components"></a>Composants réseau
Avant de nous plonger dans la résolution des problèmes, examinons certains composants et termes courants. Par le biais de cette discussion, nous nous penchons sur chaque composant de la chaîne de bout en bout qui assure la connectivité dans Azure.
![1][1]

Sur le plan général, je décris trois domaines de routage réseau principaux :

- Le réseau Azure (cloud bleu à droite)
- Internet ou WAN (cloud vert au centre)
- Le réseau d’entreprise (cloud pêche à gauche)

Observons brièvement chaque composant du diagramme, de droite à gauche :
 - **Machine virtuelle** : le serveur peut avoir plusieurs cartes réseau ; assurez-vous que les itinéraires statiques, les itinéraires par défaut et les paramètres de système d’exploitation envoient et reçoivent le trafic comme prévu. En outre, chaque référence de machine virtuelle a une restriction de bande passante. Si vous utilisez une référence de machine virtuelle plus petite, le trafic est limité par la bande passante disponible pour la carte réseau. En général, j’utilise un DS5v2 à des fins de test (que je supprime une fois les tests effectués pour économiser de l’argent) pour garantir une bande passante suffisante à la machine virtuelle.
 - **Carte réseau** : vérifiez que vous connaissez l’adresse IP privée qui est assignée à la carte réseau en question.
 - **Groupe de sécurité réseau (NSG) de la carte réseau** : plusieurs NSG spécifiques peuvent être appliqués au niveau de la carte réseau ; vérifiez que l’ensemble de règles NSG est approprié pour le trafic à faire passer. Par exemple, assurez-vous que les ports 5201 pour iPerf, 3389 pour RDP ou 22 pour SSH sont ouverts afin de permettre le passage du trafic test.
 - **Sous-réseau de réseau virtuel** : la carte réseau est assignée à un sous-réseau spécifique ; vérifiez que vous le connaissez, ainsi que les règles qui lui sont associées.
 - **Groupe de sécurité réseau de sous-réseau** : tout comme la carte réseau, des groupes de sécurité réseau peuvent être appliqués à ce sous-réseau. Assurez-vous que l’ensemble de règles NSG est approprié pour le trafic à faire passer. (Pour le trafic qui entre dans la carte réseau, le NSG du sous-réseau s’applique en premier, puis le NSG de la carte réseau ; à l’inverse, pour le trafic qui sort de la machine virtuelle, le NSG de la carte réseau s’applique en premier, puis le NSG du sous-réseau entre en jeu).
 - **UDR de sous-réseau**  : les routages définis par l’utilisateur peuvent diriger le trafic vers un tronçon intermédiaire (par exemple, un pare-feu ou un équilibreur de charge). Vérifiez s’il existe un UDR pour votre trafic et, si tel est le cas, déterminez sa position et la façon dont le tronçon suivant doit traiter le trafic. (Par exemple, un pare-feu peut laisser passer un trafic et en refuser un autre entre deux mêmes hôtes.)
 - **Sous-réseau de passerelle / groupe de sécurité réseau / UDR** : tout comme le sous-réseau de la machine virtuelle, le sous-réseau de passerelle peut avoir des groupes de sécurité réseau et des UDR. Vérifiez leur présence éventuelle et leur impact sur le trafic.
 - **Passerelle de réseau virtuel (ExpressRoute)** : une fois le peering (ExpressRoute) ou le VPN activé, peu de paramètres peuvent affecter le routage du trafic ou l’existence de ce routage. Si vous avez plusieurs circuits ExpressRoute ou tunnels VPN connectés à la même passerelle de réseau virtuel, vous devez connaître le paramétrage du poids de la connexion, car il affecte les préférences de connexion et le chemin emprunté par le trafic.
 - **Filtre de routage** (non affiché) : un filtre de routage s’applique seulement au peering Microsoft sur ExpressRoute, mais il est essentiel de vérifier si vous ne voyez pas les itinéraires attendus sur le peering Microsoft. 

À ce stade, vous êtes sur la partie WAN de la liaison. Ce domaine de routage peut être votre fournisseur de services, votre réseau étendu d’entreprise ou Internet. Le nombre élevé de tronçons, de technologies et de sociétés concernés par ces liens peut rendre difficile la résolution des problèmes. Souvent, vous vous attachez d’abord à écarter Azure et vos réseaux d’entreprise avant de vous pencher sur cette multitude de sociétés et de tronçons.

Dans le diagramme précédent, à l’extrême gauche se trouve votre réseau d’entreprise. Selon la taille de votre entreprise, ce domaine de routage peut regrouper quelques périphériques réseau entre vous et le réseau étendu ou plusieurs couches de périphériques sur un réseau de campus ou d’entreprise.

Étant donné la complexité de ces trois environnements réseau généraux différents, il est souvent optimal de démarrer à la périphérie et d’essayer de déterminer où les performances sont bonnes et où elles se dégradent. Cette approche peut vous aider à identifier lequel des trois domaines de routage est problématique, puis à concentrer la résolution des problèmes sur cet environnement.

## <a name="tools"></a>Outils
La plupart des problèmes réseau peuvent être analysés et isolés à l’aide d’outils de base tels que ping et traceroute. Il est rare que vous deviez pousser jusqu’à effectuer une analyse de paquets à l’aide d’un outil tel que Wireshark. Pour vous aider à résoudre les problèmes, la boîte à outils de connectivité Azure (AzureCT) a été développée pour placer une partie de ces outils dans un package simple. Pour le test de performance, je préfère utiliser iPerf et PSPing. iPerf est un outil couramment utilisé et s’exécute sur la plupart des systèmes d’exploitation. iPerf convient pour les tests de performances de base et est relativement facile à utiliser. PSPing est un outil de test Ping développé par SysInternals. PSPing est un moyen simple pour effectuer des tests Ping ICMP et TCP au moyen d’une seule commande également facile à utiliser. Ces deux outils sont légers et vous pouvez les « installer » simplement en copiant les fichiers dans un répertoire sur l’hôte.

J’ai placé tous ces outils et méthodes dans un module PowerShell (AzureCT) que vous pouvez installer et utiliser.

### <a name="azurect---the-azure-connectivity-toolkit"></a>AzureCT : la boîte à outils de connectivité Azure
Le module PowerShell AzureCT comporte deux composants : [test de la disponibilité][Availability Doc] et [test de performance][Performance Doc]. Ce document ne concernant que le test de performance, concentrons-nous sur les deux commandes de test de la performance de la liaison incluses dans ce module PowerShell.

L’utilisation de cette boîte à outils pour tester les performances comprend trois étapes de base. (1) Installer le module PowerShell, (2) installer les applications de prise en charge iPerf et PSPing et (3) exécuter le test de performance.

1. Installer le module PowerShell

    ```powershell
    (new-object Net.WebClient).DownloadString("https://aka.ms/AzureCT") | Invoke-Expression
    
    ```

    Cette commande télécharge le module PowerShell et l’installe localement.

2. Installer les applications de prise en charge
    ```powershell
    Install-LinkPerformance
    ```
    Cette commande AzureCT installe iPerf et PSPing dans un nouveau répertoire « C:\ACTTools » et ouvre également les ports du pare-feu Windows pour autoriser le trafic ICMP et via le port 5201 (iPerf).

3. Exécuter le test de performance

    Sur l’hôte distant, vous devez d’abord installer et exécuter iPerf en mode serveur. Vérifiez également que l’hôte distant est à l’écoute sur le port 3389 (RDP pour Windows) ou 22 (SSH pour Linux), et qu’il autorise le trafic sur le port 5201 pour iPerf. Si l’hôte distant est doté de Windows, installez la boîte à outils AzureCT et exécutez la commande Install-LinkPerformance pour configurer iPerf et les règles de pare-feu nécessaires pour démarrer iPerf en mode serveur correctement. 
    
    Une fois que l’ordinateur distant est prêt, ouvrez PowerShell sur l’ordinateur local et démarrez le test :
    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 10
    ```

    Cette commande exécute une série de tests de charge et de latence simultanés pour aider à estimer la latence et la capacité de la bande passante de votre liaison réseau.

4. Examiner la sortie des tests

    Le format de sortie PowerShell est similaire à ceci :

    ![4][4]

    Les résultats détaillés de tous les tests PSPing et iPerf se trouvent dans des fichiers texte individuels dans le répertoire des outils AzureCT à l’emplacement « C:\ACTTools ».

## <a name="troubleshooting"></a>Dépannage
Si le test de performance ne fournit pas les résultats attendus, une approche étape par étape progressive peut s’avérer pertinente. Étant donné le nombre de composants jalonnant le chemin, une approche systématique permet généralement de résoudre les problèmes plus rapidement qu’une approche empirique au cours de laquelle un même test risque d’être inutilement effectué plusieurs fois.

>[!NOTE]
>Dans notre scénario, nous devons résoudre un problème de performance, pas un problème de connectivité. La procédure serait différente si le trafic était complètement bloqué.
>
>

Tout d’abord, reconsidérez vos hypothèses. Vos attentes sont-elles raisonnables ? Par exemple, si vous avez un circuit ExpressRoute de 1 Gbit/s et une latence de 100 ms, il n’est pas raisonnable d’espérer le trafic maximal de 1 Gbit/s étant donné les caractéristiques des performances du protocole TCP sur les liaisons à latence élevée. Consultez la [section Références](#references) pour plus d’informations sur les hypothèses de performance.

Ensuite, je vous recommande de commencer à la périphérie des domaines de routage et d’essayer d’isoler le problème sur un seul domaine de routage principal : le réseau d’entreprise, le réseau étendu ou le réseau Azure. Il est courant de blâmer la « boîte noire » sur le chemin, mais cette approche facile risque de retarder considérablement la résolution, en particulier si le problème se trouve effectivement dans une zone où vous pouvez apporter des modifications. Veillez à procéder aux vérifications nécessaires avant de contacter votre fournisseur de services ou fournisseur de services Internet.

Une fois que vous avez identifié le domaine de routage principal qui contient le problème, vous devez créer un diagramme de la zone en question. Que ce soit sur un tableau blanc, dans un bloc-notes ou dans Visio, un diagramme fournit un « plan de bataille » concret pour isoler le problème de façon méthodique. Vous pouvez planifier des points de tests et mettre à jour le plan à mesure que vous écartez des zones ou approfondir des analyses à mesure que le test progresse.

Une fois votre diagramme prêt, commencez par diviser le réseau en segments afin de cerner le problème petit à petit. Déterminez où il fonctionne et où il ne fonctionne pas. Continuez à déplacer vos points de tests pour isoler le composant responsable.

En outre, n’oubliez pas d’examiner d’autres couches du modèle OSI. Il est facile de se concentrer sur le réseau et les couches 1 à 3 (couches Physique, Liaison de données et Réseau), mais les problèmes peuvent également se situer au niveau de la couche 7 (Application). Conservez une certaine ouverture d’esprit et vérifiez les hypothèses.

## <a name="advanced-expressroute-troubleshooting"></a>Résolution avancée des problèmes quand ExpressRoute est utilisé
Si vous ignorez le périmètre réel du cloud, l’isolation des composants Azure peut être difficile. Quand ExpressRoute est utilisé, le périmètre est un composant réseau appelé MSEE (Microsoft Enterprise Edge). **Quand vous utilisez ExpressRoute**, MSEE est le premier point de contact dans le réseau de Microsoft et le dernier tronçon en sortie de ce dernier. Quand vous créez un objet de connexion entre votre passerelle de réseau virtuel et le circuit ExpressRoute, vous établissez en fait une connexion à MSEE. Reconnaître MSEE comme premier ou dernier tronçon (en fonction de la direction dans laquelle vous allez) est crucial pour isoler un problème de réseau Azure afin d’établir qu’il se trouve dans Azure ou plus en aval dans le réseau étendu ou le réseau d’entreprise. 

![2][2]

>[!NOTE]
> Notez que MSEE ne se trouve pas dans le cloud Azure. ExpressRoute est en fait à la périphérie du réseau Microsoft, pas véritablement dans Azure. Une fois que vous êtes connecté avec ExpressRoute à MSEE, vous êtes connecté au réseau de Microsoft, à partir duquel vous pouvez accéder à n’importe quel service cloud, tel que Microsoft 365 (avec le Peering Microsoft) ou Azure (avec le Peering privé et/ou Microsoft).
>
>

Si deux réseaux virtuels (réseaux virtuels A et B dans le diagramme) sont connectés au **même** circuit ExpressRoute, vous pouvez effectuer une série de tests pour isoler le problème dans Azure (ou établir qu’il n’est pas dans Azure).
 
### <a name="test-plan"></a>Plan de test
1. Exécutez le test Get-LinkPerformance entre VM1 et VM2. Ce test indique si le problème est local ou non. Si ce test génère des résultats acceptables concernant la latence et la bande passante, vous pouvez marquer le réseau virtuel local comme étant correct.
2. En supposant que le trafic du réseau virtuel local est correct, exécutez le test Get-LinkPerformance entre VM1 et VM3. Ce test teste la connexion via le réseau Microsoft jusqu’au MSEE, puis jusqu’à Azure. Si ce test génère des résultats acceptables concernant la latence et la bande passante, vous pouvez marquer le réseau Azure comme étant correct.
3. Si Azure est écarté, vous pouvez effectuer une séquence similaire de tests sur votre réseau d’entreprise. Si ces tests sont également concluants, il est temps de collaborer avec votre fournisseur de services ou fournisseur de services Internet pour diagnostiquer votre connexion WAN. Exemple : Exécuter ce test entre deux succursales, ou entre votre bureau et un serveur de centre de données. Selon ce que vous testez, recherchez des points de terminaison (serveurs, PC, etc.) qui peuvent constituer ce chemin.

>[!IMPORTANT]
> Il est essentiel que pour chaque test vous notiez l’heure du jour à laquelle vous l’exécutez et que vous centralisiez l’enregistrement des résultats (j’ai un faible pour OneNote ou Excel). Chaque série de tests doit avoir une sortie identique afin que vous puissiez comparer les données résultantes entre les séries de tests et que vous n’ayez pas de « trous » dans les données. L’importance de la cohérence entre les différents tests est la raison principale qui m’amène à utiliser la boîte à outils AzureCT pour la résolution des problèmes. Ce ne sont pas les scénarios de charge exacts que j’exécute qui importent, mais le fait que j’obtiens une *sortie de données* et *de test cohérente* à l’issue de chaque test. Le fait d’enregistrer l’heure et d’avoir systématiquement des données cohérentes s’avère particulièrement utile si vous êtes amené à constater que le problème est sporadique. Faites preuve de diligence avec la collecte de données en amont ; vous devriez ainsi éviter de passer des heures à tester les mêmes scénarios plusieurs fois (je l’ai appris à mes dépens il y a de nombreuses années).
>
>

## <a name="the-problem-is-isolated-now-what"></a>Le problème étant isolé, que devons-nous faire ?
Plus vous pouvez isoler le problème, plus il est facile à résoudre ; cependant, arrive souvent un moment où vous ne pouvez plus approfondir ou étendre la résolution des problèmes. Exemple : Vous constatez que la liaison via votre fournisseur de services emprunte des tronçons en Europe, alors que la totalité du chemin est censée se trouver en Asie. Dans ce cas de figure, nous vous conseillons de demander de l’aide. La personne à laquelle vous vous adressez dépend du domaine de routage dans lequel vous avez isolé le problème, ou mieux encore, du composant que vous pouvez éventuellement identifier comme étant affecté.

Pour les problèmes de réseau d’entreprise, votre service informatique interne ou votre fournisseur de services assurant la prise en charge de votre réseau (qui peut être le fabricant du matériel) peut être en mesure de vous aider à configurer les appareils ou à réparer le matériel.

Dans le cas du réseau étendu, partager vos résultats de tests avec votre fournisseur de services ou fournisseur de services Internet peut les aider à prendre les choses en main et leur éviter d’effectuer de nouveau une partie des tests déjà réalisés par vos soins. Toutefois, ne soyez pas contrarié s’ils souhaitent vérifier vos résultats eux-mêmes. « Faire confiance, mais vérifier » est une bonne devise quand il s’agit de résoudre des problèmes à partir des résultats fournis par d’autres personnes.

Avec Azure, une fois que vous avez isolé le problème aussi précisément que possible, vous pouvez examiner la [documentation sur le réseau Azure][Network Docs] et, si nécessaire, [ouvrir un ticket de support][Ticket Link].

## <a name="references"></a>References
### <a name="latencybandwidth-expectations"></a>Attentes en termes de latence et de bande passante
>[!TIP]
> La latence géographique (miles ou kilomètres) entre les points de terminaison que vous testez est de loin le composant de la latence le plus important. Même s’il existe une latence liée aux équipements (composants physiques et virtuels, le nombre de tronçons, etc.), il s’avère que le critère géographique est le composant le plus important de la latence globale concernant les connexions WAN. Il est également important de noter que la distance est la distance en fibre optique, pas la distance à vol d’oiseau ou routière. Cette distance est extrêmement difficile à obtenir avec précision. Ainsi, j’utilise généralement un outil de calcul de distance entre villes sur Internet. Cette méthode est certes approximative, mais elle suffit pour définir une attente générale.
>
>

J’ai une installation ExpressRoute à Seattle, Washington aux États-Unis. Le tableau suivant présente la latence et la bande passante que j’ai constatées en effectuant des tests à différents emplacements Azure. J’ai estimé la distance géographique entre chaque fin du test.

Configuration des tests :
 - Un serveur physique exécutant Windows Server 2016 doté d’une carte réseau 10 Gbits/s, connecté à un circuit ExpressRoute.
 - Un circuit ExpressRoute Premium 10 Gbits/s à l’emplacement identifié avec le peering privé activé.
 - Un réseau virtuel Azure doté d’une passerelle UltraPerformance dans la région spécifiée.
 - Une machine virtuelle DS5v2 exécutant Windows Server 2016 sur le réseau virtuel. La machine virtuelle n’a pas été jointe à un domaine et a été générée à partir de l’image Azure par défaut (sans optimisation ou personnalisation) à l’aide de la boîte à outils AzureCT installée.
 - Tous les tests ont été effectués à l’aide de la commande Get-LinkPerformance de la boîte à outils AzureCT, à raison d’un test de charge de 5 minutes pour chacune des six séries de tests. Par exemple :

    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 300
    ```
 - Pour chaque test, la charge du flux de données allait du serveur physique local (client iPerf à Seattle) à la machine virtuelle Azure (serveur iPerf dans la région Azure répertoriée).
 - Les données de la colonne « Latence » sont issues du test avec absence de charge (test de latence TCP sans exécution d’iPerf).
 - Les données de la colonne « Bande passante maximale » sont issues du test de charge de 16 flux TCP avec une taille de fenêtre de 1 Mo.

![3][3]

### <a name="latencybandwidth-results"></a>Résultats de latence et de bande passante
>[!IMPORTANT]
> Les données numériques ci-dessous ne sont indiquées qu’à titre de référence générale. De nombreux facteurs affectent la latence et, bien que ces valeurs soient cohérentes sur la durée, les conditions dans Azure ou les réseaux des fournisseurs de services peuvent envoyer du trafic via différents chemins à tout moment, ce qui peut impacter la latence et la bande passante. En règle générale, ces modifications n’entraînent pas de différences significatives.
>
>

| ExpressRoute<br/>Emplacement|Azure<br/>Région | Distance<br/>estimée (km) | Latence|1 Session<br/>Bande passante | Maximale<br/>Bande passante |
| ------------------------------------------ | --------------------------- |  - | - | - | - |
| Seattle | USA Ouest 2        |    191 km |   5 ms | 262,0 Mbits/s |  3,74 Gbits/s |
| Seattle | USA Ouest          |  1\.094 km |  18 ms |  82,3 Mbits/s |  3,70 Gbits/s |
| Seattle | USA Centre       |  2\.357 km |  40 ms |  38,8 Mbits/s |  2,55 Gbits/s |
| Seattle | États-Unis - partie centrale méridionale |  2\.877 km |  51 ms |  30,6 Mbits/s |  2,49 Gbits/s |
| Seattle | Centre-Nord des États-Unis |  2\.792 km |  55 ms |  27,7 Mbits/s |  2,19 Gbits/s |
| Seattle | USA Est 2        |  3\.769 km |  73 ms |  21,3 Mbits/s |  1,79 Gbit/s |
| Seattle | USA Est          |  3\.699 km |  74 ms |  21,1 Mbits/s |  1,78 Gbit/s |
| Seattle | Japon Est       |  7\.705 km | 106 ms |  14,6 Mbits/s |  1,22 Gbit/s |
| Seattle | Sud du Royaume-Uni         |  7\.708 km | 146 ms |  10,6 Mbits/s |   896 Mbits/s |
| Seattle | Europe Ouest      |  7\.834 km | 153 ms |  10,2 Mbits/s |   761 Mbits/s |
| Seattle | Australie Est   | 12.484 km | 165 ms |   9,4 Mbits/s |   794 Mbits/s |
| Seattle | Asie Sud-Est   | 12.989 km | 170 ms |   9,2 Mbits/s |   756 Mbits/s |
| Seattle | Brésil Sud *   | 10.930 km | 189 ms |   8,2 Mbits/s |   699 Mbits/s |
| Seattle | Inde Sud      | 12.918 km | 202 ms |   7,7 Mbits/s |   634 Mbits/s |

\* La latence au Brésil est un bon exemple où la distance à vol d’oiseau diffère considérablement de la distance en fibre optique. Alors que la latence devrait normalement se situer aux alentours de 160 ms, elle s’élève à 189 ms. Cette différence par rapport à mes attentes pourrait indiquer un problème réseau quelque part, mais très probablement que la fibre optique en direction du Brésil n’emprunte pas une ligne droite, ce qui se traduit par un supplément de 1 000 km environ sur l’itinéraire entre Seattle et le Brésil.

## <a name="next-steps"></a>Étapes suivantes
1. Télécharger la boîte à outils de connectivité Azure à partir de GitHub à l’adresse [https://aka.ms/AzCT][ACT]
2. Suivre les instructions pour [tester le niveau de performance des liaisons][Performance Doc]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-network-performance/network-components.png "Composants réseau Azure"
[2]: ./media/expressroute-troubleshooting-network-performance/expressroute-troubleshooting.png "Détecter un problème ExpressRoute"
[3]: ./media/expressroute-troubleshooting-network-performance/test-diagram.png "Environnement du test de performances"
[4]: ./media/expressroute-troubleshooting-network-performance/powershell-output.png "Sortie PowerShell"

<!--Link References-->
[Performance Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/PerformanceTesting.md
[Availability Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/AvailabilityTesting.md
[Network Docs]: ../index.yml
[Ticket Link]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview
[ACT]: https://aka.ms/AzCT