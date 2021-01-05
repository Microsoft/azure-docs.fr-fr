---
title: Améliorer la sécurité de la gestion à distance dans Azure | Microsoft Docs
description: Cet article décrit les étapes à suivre pour améliorer la sécurité de l’administration à distance dans le cadre de l’administration des environnements Microsoft Azure, notamment les services cloud, les machines virtuelles et les applications personnalisées.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: TomSh
ms.assetid: 2431feba-3364-4a63-8e66-858926061dd3
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/08/2020
ms.author: terrylan
ms.openlocfilehash: e9eabc73c244526f0ea15b9c72b5377545f662b2
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844861"
---
# <a name="security-management-in-azure"></a>Gestion de la sécurité dans Azure
Les abonnés Azure peuvent gérer leurs environnements cloud à partir de différents périphériques, comme les stations de travail de gestion, les ordinateurs de développement ou encore les périphériques d’utilisateurs finaux privilégiés, qui disposent d’autorisations spécifiques. Dans certains cas, les fonctions d’administration sont effectuées par le biais de consoles Web, comme le [portail Azure](https://azure.microsoft.com/features/azure-portal/). Des connexions directes peuvent aussi être établies avec Azure à partir de systèmes locaux sur des réseaux privés virtuels (VPN), Terminal Services, des protocoles d’application cliente ou l’API de gestion des services Azure (SMAPI) (par programmation). Par ailleurs, les points de terminaison de client peuvent être joints au domaine ou isolés et non gérés, comme les tablettes ou les smartphones.

Bien que les fonctions de gestion et d’accès offrent de nombreuses options, le déploiement de cloud peut se révéler plus délicat. La gestion, le suivi et l’audit peuvent être plus complexes pour les actions d’administration. Des menaces de sécurité peuvent également survenir avec l’accès non réglementé aux points de terminaison de client utilisés pour la gestion des services cloud. Les stations de travail personnelles ou communes destinées au développement et à la gestion de l’infrastructure introduisent des menaces imprévisibles, notamment avec la navigation Web (par exemple, lors d’attaques de point d’eau) ou la messagerie électronique (par exemple, ingénierie sociale et hameçonnage).

![Diagramme montrant les différentes façons dont une menace peut monter une attaque.](./media/management/typical-management-network-topology.png)

Les risques d’attaque sont plus importants dans ce type d’environnement : en effet, il est difficile de créer des stratégies et des mécanismes de sécurité pour gérer l’accès aux interfaces Azure (comme SMAPI) à partir de points de terminaison variés.

### <a name="remote-management-threats"></a>Menaces liées à la gestion à distance
Les personnes malveillantes tentent souvent d’obtenir un accès privilégié en compromettant des informations d’identification du compte (attaque par force brute, hameçonnage, collecte d’informations d’identification, etc.) ou en amenant les utilisateurs à exécuter un code malveillant (par exemple, à partir de téléchargements furtifs sur des sites web malveillants ou à partir de pièces jointes présentes dans des courriers électroniques malveillants). L’accès en tout temps et en tout lieu rend les environnements cloud gérés à distance plus vulnérables aux violations de compte.

Même avec un contrôle strict des comptes d’administrateur principal, des comptes d’utilisateur de niveau inférieur peuvent être utilisés pour exploiter les faiblesses de la stratégie de sécurité. En l’absence d’une formation de sécurité appropriée, des violations de compte peuvent découler de la divulgation accidentelle ou de l’exposition d’informations sur le compte.

Lorsqu’une station de travail utilisateur sert aux tâches d’administration, elle peut être compromise à de nombreux niveaux, que l’utilisateur navigue sur le web, qu’il se serve d’outils tiers et open source, ou qu’il ouvre un document contenant un cheval de Troie.

En règle générale, la plupart des attaques ciblées entraînant des violations de données sont liées à l’exploitation de vulnérabilités du navigateur, aux plug-ins (par exemple, Flash, PDF, Java) et au hameçonnage ciblé sur les ordinateurs de bureau. Ces ordinateurs disposent d’autorisations d’administration ou de service permettant d’accéder aux serveurs dynamiques ou aux périphériques réseau lors du développement ou de la gestion d’autres ressources.

### <a name="operational-security-fundamentals"></a>Principes de base pour la sécurité opérationnelle
Pour une gestion et un fonctionnement plus sécurisés, vous pouvez réduire la surface d’attaque d’un client en limitant le nombre de points d’entrée possibles. Cette opération peut être réalisée grâce aux principes de sécurité de séparation des tâches et de répartition des environnements.

L’isolement des fonctions sensibles évite l’effet domino. Exemples :

* Les tâches d’administration ne doivent pas être associées à des activités susceptibles d’entraîner une compromission (par exemple, un programme malveillant présent dans le courrier électronique de l’administrateur peut infecter un serveur d’infrastructure).
* La station de travail utilisée pour les opérations ultrasensibles ne doit pas être utilisée pour les activités à haut risque, comme la navigation sur Internet.

La limitation de la surface d’attaque du système par la suppression de logiciels inutiles. Exemple :

* Une station d’administration, de support ou de développement standard n’a pas besoin d’un client de messagerie ni d’autres applications de productivité si elle sert principalement à gérer les services cloud.

Les systèmes clients disposant d’un accès administrateur aux composants d’infrastructure doivent être soumis à une stratégie aussi stricte que possible afin de limiter les risques de sécurité. Exemples :

* Les stratégies de sécurité peuvent inclure des paramètres de stratégie de groupe refusant l’accès libre à Internet à partir de l’appareil, et l’utilisation d’une configuration de pare-feu restrictive.
* Utilisez des réseaux privés virtuels (VPN) de sécurité du protocole Internet (IPsec) si un accès direct est requis.
* Configurez des domaines Active Directory de gestion et de développement distincts.
* Isolez et filtrez du trafic réseau de la station de travail de gestion.
* Utilisez des logiciels anti-programme malveillant.
* Implémentez une authentification multifacteur afin de réduire le risque de vol d’informations d’identification.

La consolidation des ressources d’accès et la suppression des points de terminaison non gérés simplifient également les tâches de gestion.

### <a name="providing-security-for-azure-remote-management"></a>Mesures de sécurité pour la gestion à distance Azure
Azure propose divers mécanismes de sécurité aux administrateurs responsables de services cloud et de machines virtuelles Azure. Ces mécanismes sont les suivants :

* Authentification et [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md).
* Surveillance, journalisation et audit.
* Certificats et communications chiffrées.
* Portail de gestion Web.
* Filtrage des paquets réseau.

Avec la configuration de sécurité côté client et le déploiement du centre de données d’une passerelle de gestion, il est possible de restreindre et de surveiller l’accès administrateur aux applications cloud et aux données.

> [!NOTE]
> Certaines recommandations contenues dans cet article peuvent entraîner une augmentation des taux d’utilisation des données, des réseaux ou des ressources de calcul, débouchant sur des coûts de licence ou d’abonnement supplémentaires.
>
>

## <a name="hardened-workstation-for-management"></a>Station de travail renforcée pour la gestion
Le renforcement d’une station de travail permet de conserver uniquement les fonctions les plus essentielles afin de réduire autant que possible la surface d’attaque potentielle. Le renforcement du système inclut la diminution du nombre de services et d’applications, la restriction de l’exécution d’applications, la limitation de l’accès réseau et la mise à jour permanente du système. En outre, une station de travail renforcée pour la gestion permet de faire la distinction entre les outils et les activités d’administration d’une part, et les autres tâches incombant à l’utilisateur final d’autre part.

Dans un environnement d’entreprise local, vous pouvez réduire la surface d’attaque de votre infrastructure physique au travers de réseaux de gestion dédiés, de salles des serveurs avec accès à la carte, et de stations de travail s’exécutant sur des zones protégées du réseau. Dans un cloud ou un modèle informatique hybride, il est plus difficile d’assurer une gestion sécurisée en raison de l’absence d’accès physique aux ressources informatiques. L’implémentation de solutions de protection nécessite une configuration logicielle soignée, des processus axés sur la sécurité et des stratégies exhaustives.

Un encombrement logiciel réduit avec séparation des privilèges dans une station de travail verrouillée pour la gestion de cloud et le développement d’applications peut limiter les risques d’incident de sécurité grâce à la normalisation des environnements de développement et de gestion à distance. Une station de travail renforcée peut empêcher la compromission de comptes de gestion des ressources cloud principales en fermant les accès couramment utilisés par les logiciels malveillants et les attaques. Plus exactement, vous pouvez utiliser [Windows AppLocker](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd759117(v=ws.11)) et la technologie Hyper-V pour contrôler et isoler le comportement du système client et pour atténuer les menaces, y compris pour le courrier électronique ou la navigation sur Internet.

Sur une station de travail renforcée, l’administrateur exécute un compte d’utilisateur standard (ce qui bloque l’exécution au niveau administrateur), et les applications associées sont contrôlées au moyen d’une liste verte. Voici les éléments de base d’une station de travail renforcée :

* Analyse et mise à jour corrective actives. Déployez des logiciels anti-programme malveillant, effectuez des analyses régulières de vulnérabilité et mettez à jour toutes les stations de travail en procédant à la dernière mise à jour de sécurité dans les temps.
* Fonctionnalités limitées. Désinstallez toutes les applications qui ne sont pas nécessaires et désactivez les services inutiles (démarrage).
* Renforcement du réseau. Utilisez les règles de pare-feu Windows pour autoriser uniquement les adresses IP, les ports et les URL de gestion Azure valides. Assurez-vous que les connexions à distance vers la station de travail sont également bloquées.
* Restriction d’exécution. Autorisez uniquement un ensemble de fichiers exécutables prédéfinis qui sont nécessaires à la gestion de l’exécution (refus par défaut). Par défaut, les utilisateurs ne doivent pas être autorisés à exécuter n’importe quel programme, sauf mention spécifique dans la liste verte.
* Séparation des privilèges. Les utilisateurs de la station de travail de gestion ne doivent pas disposer de privilèges d’administration sur l’ordinateur local lui-même. Ainsi, ils sont dans l’impossibilité de modifier la configuration du système et les fichiers système, que ce soit de manière intentionnelle ou non.

Vous pouvez mettre en œuvre toutes ces règles à l’aide [d’objets de stratégie de groupe](../../active-directory-domain-services/manage-group-policy.md) dans les services de domaine Active Directory (AD DS) et les appliquer à tous les comptes de gestion par le biais de votre domaine de gestion (local).

### <a name="managing-services-applications-and-data"></a>Gestion des services, des applications et des données
La configuration des services cloud Azure s’effectue sur le portail Azure ou SMAPI grâce à l’interface de ligne de commande Windows PowerShell ou à une application personnalisée tirant parti de ces interfaces RESTful. Azure Active Directory (Azure AD), Azure Storage, les sites Web Azure et Azure Virtual Network font partie des services qui utilisent ces mécanismes.

Machine virtuelle : les applications déployées fournissent leurs propres outils et interfaces clients en fonction des besoins, comme la console Microsoft Management Console (MMC), une console de gestion d’entreprise (par exemple, Microsoft System Center ou Windows Intune) ou une autre application de gestion, telle que Microsoft SQL Server Management Studio. Ces outils se trouvent généralement dans un environnement d’entreprise ou sur un réseau client. Ils peuvent dépendre de protocoles réseau spécifiques, comme le protocole RDP (Remote Desktop Protocol), qui nécessitent des connexions directes avec état. Certains peuvent avoir des interfaces Web qui ne doivent pas être publiées ni accessibles librement sur Internet.

Vous pouvez limiter l’accès à la gestion des services de plateforme et d’infrastructure dans Azure grâce à [l’authentification multifacteur](../../active-directory/authentication/concept-mfa-howitworks.md), aux [certificats de gestion X.509](/archive/blogs/azuresecurity/certificate-management-in-azure-dos-and-donts) et aux règles de pare-feu. Le portail Azure et SMAPI nécessitent un protocole TLS (Transport Layer Security). Cependant, les services et applications déployés dans Azure vous obligent à prendre des mesures de protection adaptées à l’application. Ces opérations sont souvent plus faciles avec une station de travail renforcée normalisée.

### <a name="management-gateway"></a>Passerelle de gestion
Pour centraliser tous les accès d’administration et simplifier l’analyse ainsi que la journalisation, vous pouvez déployer un serveur [Passerelle des services Bureau à distance](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd560672(v=ws.10)) dédié sur votre réseau local, qui sera connecté à votre environnement Azure.

Une passerelle des services Bureau à distance désigne un service de proxy RDP basé sur une stratégie et qui met en application des exigences de sécurité. L’implémentation d’une passerelle des services Bureau à distance avec la protection d’accès réseau (NAP) Windows Server garantit que seuls les clients répondant aux critères d’intégrité de la sécurité établis par les objets de stratégie de groupe des services de domaine Active Directory (AD DS) peuvent se connecter. Informations supplémentaires :

* Configurez un [certificat de gestion Azure](/previous-versions/azure/gg551722(v=azure.100)) sur la passerelle des services Bureau à distance afin qu’il soit le seul hôte autorisé à accéder au portail Azure.
* Associez la passerelle des services Bureau à distance au même [domaine de gestion](/previous-versions/windows/it-pro/windows-2000-server/bb727085(v=technet.10)) que les stations de travail d’administration. Cette opération est indispensable lorsque vous utilisez une connexion VPN IPsec de site à site ou ExpressRoute au sein d’un domaine offrant une approbation à sens unique vers Azure AD, ou lorsque vous fédérez des informations d’identification entre votre instance AD DS locale et Azure AD.
* Configurez une [stratégie d’autorisation de connexion client](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753324(v=ws.11)) pour permettre à la passerelle des services Bureau à distance de vérifier que le nom de l’ordinateur client est valide (joint au domaine) et que l’ordinateur est autorisé à accéder au portail Azure.
* Utilisez IPsec pour [Azure VPN](https://azure.microsoft.com/documentation/services/vpn-gateway/) afin de protéger le trafic de gestion contre les écoutes clandestines et les vols de jeton, ou vérifiez un lien Internet isolé par le biais d’[Azure ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/).
* Activez l'authentification multifacteur (via [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md)) ou l'authentification par carte à puce pour les administrateurs qui se connectent au moyen de la passerelle des services Bureau à distance.
* Configurez les [restrictions d’adresse IP](https://azure.microsoft.com/blog/2013/08/27/confirming-dynamic-ip-address-restrictions-in-windows-azure-web-sites/) sources ou les [groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md) dans Azure afin de limiter le nombre de points de terminaison de gestion autorisés.

## <a name="security-guidelines"></a>Conseils de sécurité
En règle générale, la sécurisation des stations de travail d’administration pour le cloud s’apparente aux pratiques utilisées pour les stations de travail locales, comme la minimisation de la build et les autorisations restrictives. Certains aspects uniques de la gestion du cloud relèvent plutôt de la gestion d’entreprise à distance ou hors bande. Ces derniers incluent l’utilisation et l’audit des informations d’identification, de l’accès à distance sécurisé, ainsi que de la détection des menaces et des interventions associées.

### <a name="authentication"></a>Authentification
Vous pouvez vous servir des restrictions d’ouverture de session Azure dans l’optique de limiter les adresses IP sources pouvant accéder aux outils d’administration et aux demandes d’accès à l’audit. Pour permettre à Azure d’identifier les clients de gestion (stations de travail et/ou applications), vous pouvez configurer SMAPI (avec des outils clients tels que les applets de commande Windows PowerShell) et le portail Azure en vue d’exiger l’installation de certificats de gestion côté client en plus des certificats TLS/SSL. Il est vivement recommandé d’exiger l’application de l’authentification multifacteur pour tous les accès administrateur.

Certaines applications ou certains services déployés dans Azure peuvent avoir leurs propres mécanismes d’authentification pour l’accès utilisateur ou administrateur, tandis que d’autres tirent pleinement avantage d’Azure AD. Selon que vous fédérez des informations d’identification par le biais des services de fédération Active Directory (AD FS), que vous utilisiez la synchronisation de répertoires ou que vous conserviez uniquement les comptes d’utilisateur dans le cloud, [Microsoft Identity Manager](/microsoft-identity-manager/) (qui fait partie d’Azure AD Premium) vous aide à gérer les cycles de vie d’identité entre les ressources.

### <a name="connectivity"></a>Connectivité
Plusieurs mécanismes permettent de sécuriser les connexions client vers vos réseaux virtuels Azure. Deux de ces mécanismes (connexion [VPN de site à site](https://channel9.msdn.com/series/Azure-Site-to-Site-VPN) (S2S) et connexion [VPN de point à site](../../vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal.md) (P2S)) acceptent la norme IPsec standard (S2S) ou le protocole [Secure Socket Tunneling Protocol](/previous-versions/technet-magazine/cc162322(v=msdn.10)) (SSTP) (P2S) pour le chiffrement et le tunneling. Lorsque Azure se connecte à un système de gestion des services Azure public, comme le portail Azure, un protocole HTTPS (Hypertext Transfer Protocol Secure) est requis.

Une station de travail renforcée autonome qui ne se connecte pas à Azure par le biais d’une passerelle des services Bureau à distance doit utiliser une connexion VPN de point à site basée sur le protocole SSTP pour mettre en place la connexion initiale vers le réseau virtuel Azure, puis établir la connexion RDP avec des machines virtuelles individuelles dans le tunnel VPN.

### <a name="management-auditing-vs-policy-enforcement"></a>Audit de gestion et application de stratégies
deux approches courantes permettent de sécuriser les processus de gestion : l’audit et l’application de stratégies. Ces deux approches offrent un contrôle complet, mais il n’est pas toujours possible de les utiliser. De plus, chaque approche affiche différents niveaux de risque, de coût et d’effort concernant la gestion de la sécurité, surtout parce qu’il est aussi question du niveau de confiance accordé aux individus et aux architectures système.

La surveillance, la journalisation et l’audit aident à suivre et à comprendre les activités d’administration, mais il n’est pas toujours possible d’auditer toutes les actions en détail compte tenu de la quantité de données générées. Il est toutefois recommandé d’auditer l’efficacité des stratégies de gestion.

L’application de stratégies incluant des contrôles d’accès stricts permet d’instaurer des mécanismes de programmation afin de régir les actions d’administration, et elle garantit l’utilisation de toutes les mesures de protection disponibles. La journalisation fournit une preuve de la mise en application des stratégies et consigne également le nom des personnes impliquées, les lieux choisis et les moments concernés. Avec la journalisation, vous pouvez aussi auditer et recouper les informations liées au respect des stratégies par les administrateurs, et obtenir une preuve d’activité.

## <a name="client-configuration"></a>Configuration de client
Nous recommandons trois configurations principales pour une station de travail renforcée. Les principales différences entre ces configurations concernent les coûts, la facilité d’utilisation et l’accessibilité dans le cadre d’un profil de sécurité similaire pour toutes les options. Le tableau suivant fournit une brève analyse des avantages et des risques liés à chaque configuration. (L’expression « PC d’entreprise » fait référence à une configuration d’ordinateur de bureau standard déployée pour tous les utilisateurs du domaine, quel que soit leur rôle).

| Configuration | Avantages | Inconvénients |
| --- | --- | --- |
| Station de travail renforcée autonome |Station de travail étroitement contrôlée |Coût plus élevé pour les stations de travail dédiées |
| - | Réduction des risques d’exploitation de l’application |Efforts de gestion accrus |
| - | Séparation nette des responsabilités | - |
| PC d’entreprise servant de machine virtuelle |Réduction des coûts matériels | - |
| - | Séparation des rôles et des applications | - |

La station de travail renforcée doit être l’hôte et non l’invité, et rien ne doit se trouver entre le système d’exploitation de l’hôte et le matériel. Le « principe de source propre » (ou « origine sécurisée ») signifie que l’ordinateur hôte doit être renforcé de manière optimale. Autrement, la station de travail renforcée (invité) est vulnérable aux attaques portant sur le système d’hébergement.

Vous pouvez isoler des fonctions d’administration au moyen d’images système dédiées pour chaque station de travail renforcée disposant uniquement des outils et des autorisations nécessaires à la gestion des applications cloud et Azure sélectionnées, avec les objets de stratégie de groupe AD DS locaux associés aux tâches requises.

Pour les environnements informatiques sans infrastructure locale (par exemple, sans accès à une instance AD DS locale pour les objets de stratégie de groupe en raison de la présence de tous les serveurs sur le cloud), un service comme [Microsoft Intune](/mem/intune/) peut simplifier le déploiement et la maintenance de la configuration des stations de travail.

### <a name="stand-alone-hardened-workstation-for-management"></a>Station de travail renforcée autonome pour la gestion
Avec une station de travail renforcée autonome, les administrateurs disposent d’un ordinateur de bureau ou d’un ordinateur portable pour les tâches d’administration, et d’un autre ordinateur de bureau ou ordinateur portable pour les tâches restantes. Une station de travail dédiée à la gestion de vos services Azure ne nécessite pas d’autres applications. En outre, l’utilisation de stations de travail compatibles avec le [Module de plateforme sécurisée](/previous-versions/windows/it-pro/windows-vista/cc766159(v=ws.10)) (TPM) ou une technologie de chiffrement matériel similaire permet d’authentifier les appareils et empêche certaines attaques. Le Module de plateforme sécurisée prend également en charge la protection du volume complet du lecteur système avec le [Chiffrement de lecteur BitLocker](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc732774(v=ws.11)).

Dans une station de travail renforcée autonome (voir ci-dessous), l’instance locale du pare-feu Windows (ou un pare-feu client tiers) est configurée de manière à bloquer les connexions entrantes, comme le protocole RDP. L’administrateur peut ouvrir une session sur la station de travail renforcée et lancer une session RDP qui se connecte à Azure après l’établissement d’une connexion VPN avec Azure Virtual Network, mais il ne peut pas utiliser un PC d’entreprise et le protocole RDP pour se connecter à la station de travail renforcée elle-même.

![Diagramme montrant le scénario de station de travail renforcée autonome.](./media/management/stand-alone-hardened-workstation-topology.png)

### <a name="corporate-pc-as-virtual-machine"></a>PC d’entreprise servant de machine virtuelle
Si l’utilisation d’une station de travail renforcée autonome distincte coûte trop cher ou n’est pas pratique, la station de travail renforcée peut héberger une machine virtuelle pour les tâches non administratives.

![Diagramme montrant la station de travail renforcée hébergeant une machine virtuelle afin d’effectuer des tâches non administratives.](./media/management/hardened-workstation-enabled-with-hyper-v.png)

Dans l’optique de contourner plusieurs risques de sécurité liés à l’utilisation d’une station de travail pour la gestion des systèmes et d’autres tâches quotidiennes, vous pouvez déployer une machine virtuelle Windows Hyper-V vers la station de travail renforcée. Cette machine virtuelle sert ainsi de PC d’entreprise. L’environnement des PC d’entreprise peut rester isolé de l’hôte, ce qui réduit la surface d’attaque et supprime la coexistence entre activités quotidiennes de l’utilisateur (par exemple, courrier électronique) et tâches d’administration sensibles.

La machine virtuelle du PC d’entreprise s’exécute dans un espace protégé et fournit des applications utilisateur. L’hôte reste une « source propre » et applique des stratégies de réseau strictes dans le système d’exploitation racine (par exemple, blocage de l’accès RDP à partir de la machine virtuelle).

## <a name="best-practices"></a>Meilleures pratiques
Tenez compte des recommandations suivantes pour la gestion des applications et des données dans Azure.

### <a name="dos-and-donts"></a>Choses à faire et à ne pas faire
Même si une station de travail est verrouillée, les autres exigences de sécurité courantes doivent être respectées. Le risque est plus important en raison des niveaux d’accès élevés dont disposent généralement les comptes d’administrateur. Le tableau ci-dessous présente des exemples de risques et d’autres pratiques de sécurité.

| À ne pas faire | À faire |
| --- | --- |
| N’envoyez pas d’informations d’identification administrateur ou d’autres informations sensibles (par exemple, certificats TLS/SSL ou certificats de gestion) par courrier électronique. |Préservez la confidentialité des données en fournissant oralement les noms et les mots de passe de compte (sans les stocker dans la messagerie vocale), installez les certificats client/serveur à distance (par le biais d’une session chiffrée), effectuez des téléchargements à partir d’un partage réseau protégé ou procédez à une distribution manuelle au moyen de supports amovibles. |
| - | Gérez de manière proactive les cycles de vie de votre certificat de gestion. |
| Ne stockez pas de mots de passe non chiffrés ou non hachés dans le système de stockage de l’application (feuilles de calcul, sites SharePoint, partages de fichiers, etc.). |Établissez des principes de gestion de sécurité et des stratégies de renforcement de système, et appliquez-les à votre environnement de développement. |
| - | Utilisez les règles d’épinglage de certificat [Enhanced Mitigation Experience Toolkit 5.5](https://technet.microsoft.com/security/jj653751) pour garantir un accès approprié aux sites Azure SSL/TLS. |
| Ne partagez pas de comptes ni de mots de passe entre plusieurs administrateurs, et ne réutilisez pas les mots de passe sur plusieurs comptes d’utilisateur ou services, notamment pour les médias sociaux ou d’autres activités non administratives. |Créez un compte Microsoft dédié pour gérer votre abonnement Azure, que vous n’utiliserez pas pour le courrier électronique personnel. |
| N’envoyez pas de fichiers de configuration par courrier électronique. |Les profils et les fichiers de configuration doivent être installés à partir d’une source approuvée (par exemple, lecteur flash USB chiffré), et non à partir d’un mécanisme qui peut être facilement compromis, comme le courrier électronique. |
| N’utilisez pas de mots de passe d’ouverture de session trop simples ou trop faibles. |Appliquez des stratégies de mot de passe fort, des cycles d’expiration (modification à la première utilisation), des délais d’expiration de la console et l’automatisation des verrouillages de compte. Utilisez un système de gestion de mot de passe client compatible avec l’authentification multifacteur pour l’accès par mot de passe. |
| N’exposez pas les ports de gestion sur Internet. |Verrouillez les ports Azure et les adresses IP afin de limiter l’accès à la gestion. Pour plus d’informations, consultez le livre blanc sur la [sécurité du réseau Azure](https://download.microsoft.com/download/4/3/9/43902EC9-410E-4875-8800-0788BE146A3D/Windows%20Azure%20Network%20Security%20Whitepaper%20-%20FINAL.docx). |
| - | Utilisez des pare-feu, des réseaux privés virtuels et une protection d’accès réseau pour toutes les connexions de gestion. |

## <a name="azure-operations"></a>Opérations Azure
Dans le cadre du fonctionnement de la technologie Azure, les ingénieurs d’exploitation et le support technique qui peuvent accéder aux systèmes de production Azure utilisent [des stations de travail PC renforcées avec des machines virtuelles](#stand-alone-hardened-workstation-for-management) configurées pour autoriser un accès au réseau d’entreprise interne et aux applications (courrier électronique, intranet, etc.). Toutes les stations de travail de gestion sont dotées de Modules de plateforme sécurisée ; le lecteur de démarrage hôte est chiffré avec BitLocker ; et ces composants sont joints à une unité d’organisation spéciale dans le domaine d’entreprise principal de Microsoft.

Le renforcement du système est mis en application au travers de la stratégie de groupe, avec la centralisation des mises à jour logicielles. Pour l’audit et l’analyse, les journaux des événements (par exemple, sur la sécurité et AppLocker) sont collectés auprès de stations de travail de gestion et enregistrés dans un emplacement central.

En outre, des serveurs jump dédiés sur le réseau Microsoft et nécessitant une authentification à deux facteurs permettent de se connecter au réseau de production Azure.

## <a name="azure-security-checklist"></a>Liste de contrôle de sécurité Azure
La limitation du nombre de tâches réalisables par les administrateurs sur une station de travail renforcée réduit la surface d’attaque de votre environnement de gestion et de développement. Utilisez les technologies suivantes pour protéger votre station de travail renforcée :

* Renforcement d’Internet Explorer. Le navigateur Internet Explorer (comme tout autre navigateur web) est un point d’entrée clé pour le code malveillant en raison de ses nombreuses interactions avec les serveurs externes. Passez en revue vos stratégies de clients et optez pour un mode protégé, la désactivation des modules complémentaires, la désactivation des téléchargements de fichiers et le filtrage [Microsoft SmartScreen](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj618329(v=ws.11)). Assurez-vous que les avertissements de sécurité s’affichent correctement. Utilisez les zones Internet et créez une liste de sites de confiance pour lesquels vous avez configuré un renforcement raisonnable. Bloquez tous les autres sites et le code intégré du navigateur, comme les contrôles ActiveX et Java.
* Utilisateur standard. Le statut d’utilisateur standard offre un certain nombre d’avantages : il complique notamment le vol d’informations d’identification de l’administrateur par le biais de programmes malveillants. En outre, un compte d’utilisateur standard ne dispose pas de privilèges élevés sur le système d’exploitation racine, tandis que plusieurs options de configuration et API sont verrouillées par défaut.
* AppLocker. Vous pouvez utiliser [AppLocker](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee619725(v=ws.10)) pour limiter les programmes et les scripts exécutables. Vous pouvez utiliser AppLocker en mode d’audit ou d’application. Par défaut, AppLocker comporte une règle d’autorisation qui permet aux utilisateurs disposant d’un jeton Admin d’exécuter tout le code sur le client. Cette règle empêche les administrateurs de procéder eux-mêmes au verrouillage, et elle s’applique uniquement aux jetons correspondant à des privilèges élevés. Consultez également l’intégrité du code dans le cadre de la [sécurité de base](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd348705(v=ws.10)) Windows Server.
* Signature de code. La signature de code pour tous les scripts et les outils utilisés par les administrateurs offre un moyen simple de déployer des stratégies de verrouillage d’application. Les hachages ne s’adaptent pas à de rapides modifications sur le code, et les chemins d’accès ne fournissent pas un niveau élevé de sécurité. Vous devez combiner les règles AppLocker avec une [stratégie d’exécution](/previous-versions/windows/it-pro/windows-powershell-1.0/ee176961(v=technet.10)) PowerShell qui autorise uniquement [l’exécution](/powershell/module/microsoft.powershell.security/set-executionpolicy) du code et des scripts signés.
* Stratégie de groupe. Créez une stratégie d’administration globale qui s’applique à toute station de travail de domaine utilisée pour la gestion (et bloquez l’accès de toutes les autres), ainsi qu’aux comptes d’utilisateur authentifiés sur ces stations de travail.
* Approvisionnement plus sécurisé. Protégez l’image de votre poste de travail renforcé de référence afin de vous protéger contre la falsification. Utilisez différentes mesures de sécurité, comme le chiffrement et l’isolement, pour stocker des images, des machines virtuelles et des scripts, et pour restreindre l’accès (en vous servant par exemple d’un processus d’archivage/extraction vérifiable).
* Mise à jour corrective. Conservez une build cohérente (ou disposez d’images distinctes pour le développement, les opérations et les autres tâches d’administration), analysez les modifications et les logiciels malveillants régulièrement, mettez la build à jour et activez uniquement les ordinateurs lorsque nécessaire.
* Chiffrement. Assurez-vous que les stations de travail de gestion disposent d’un Module de plateforme sécurisée pour sécuriser le [système de fichiers EFS](/previous-versions/tn-archive/cc700811(v=technet.10)) (Encrypting File System) et BitLocker.
* Gouvernance. Utilisez les objets de stratégie de groupe AD DS pour contrôler toutes les interfaces Windows de l’administrateur, comme le partage de fichiers. Incluez les stations de travail de gestion dans l’audit, la surveillance et la journalisation. Effectuez le suivi des accès et de l’utilisation au niveau administrateur et développeur.

## <a name="summary"></a>Résumé
Une station de travail renforcée pour l’administration de vos services cloud Azure, de vos machines virtuelles et de vos applications peut vous aider à lutter contre de nombreux risques et des menaces découlant de la gestion à distance de l’infrastructure informatique critique. Azure et Windows proposent des méthodes que vous pouvez employer pour protéger et contrôler les communications, l’authentification et le comportement client.

## <a name="next-steps"></a>Étapes suivantes
Les ressources suivantes fournissent des informations générales sur les services Azure et Microsoft associés, ainsi que sur les éléments évoqués dans cet article :

* [Sécurisation de l’accès privilégié](/windows-server/identity/securing-privileged-access/securing-privileged-access) : obtenez les détails techniques de conception et de création d’une station de travail d’administration sécurisée pour la gestion Azure
* [Centre de gestion de la confidentialité Microsoft](https://microsoft.com/en-us/trustcenter/cloudservices/azure) : approfondissez vos connaissances sur les fonctionnalités de la plateforme Azure qui protègent la structure Azure et les charges de travail s’exécutant sur Azure
* [Centre de réponse aux problèmes de sécurité Microsoft](https://www.microsoft.com/msrc) : les vulnérabilités de sécurité Microsoft, y compris les problèmes avec Azure, peuvent être rapportées ou signalées par courrier électronique à l’adresse [secure@microsoft.com](mailto:secure@microsoft.com)