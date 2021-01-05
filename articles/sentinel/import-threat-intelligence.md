---
title: Importer le renseignement sur les menaces dans Azure Sentinel | Microsoft Docs
description: Utilisez des flux de renseignement sur les menaces dans Azure Sentinel pour analyser des données, détecter des menaces, et générer des alertes et des incidents. Visualisez les informations concernant le renseignement sur les menaces avec des classeurs.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/26/2020
ms.author: yelevin
ms.openlocfilehash: bde11c8e06891025be96810acf6d87952a3d8d2f
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660778"
---
# <a name="import-threat-intelligence-into-azure-sentinel"></a>Importer le renseignement sur les menaces dans Azure Sentinel

## <a name="introduction-to-threat-intelligence"></a>Introduction au renseignement sur les menaces

Le renseignement sur les cybermenaces (CTI, Cyber Threat Intelligence) est constitué d’informations qui décrivent des menaces connues existantes ou potentielles pour les systèmes et les utilisateurs. Ce type d’informations prend de nombreuses formes, depuis des rapports écrits détaillant les motivations, l’infrastructure et les techniques d’un acteur de menace particulier jusqu’à des observations spécifiques d’adresses IP, de domaines et de hachages de fichier associés à des cybermenaces. Les organisations utilisent le renseignement sur les cybermenaces pour fournir un contexte essentiel à une activité inhabituelle, afin que le personnel de sécurité puisse prendre rapidement des mesures pour protéger leurs employés et leurs ressources. Le renseignement sur les cybermenaces peut provenir de plusieurs emplacements, comme des communautés de partage de renseignements sur les menaces, des flux de renseignements commerciaux et des renseignements recueillis au niveau local, le tout collecté au cours d’enquêtes de sécurité au sein d’une organisation.

Dans une solution SIEM (Informations de sécurité et gestion d’événements) comme Azure Sentinel, les indicateurs de menace, souvent appelés « indicateurs de compromission » ou « IoC », constituent la forme de renseignement sur les cybermenaces la plus utilisée. Les indicateurs de menace sont des données qui associent des observations comme des URL, des hachages de fichier ou des adresses IP, à une activité de menace connue comme le hameçonnage, les botnets ou les programmes malveillants. Cette forme de renseignement sur les menaces est souvent appelée « renseignement tactique sur les menaces », car elle peut être appliquée à des produits de sécurité et à l’automatisation à grande échelle. Elle permet également de détecter les menaces potentielles auxquelles une organisation est exposée et la protéger contre celles-ci. Dans Azure Sentinel, vous pouvez utiliser des indicateurs de menace pour détecter les activités malveillantes observées dans votre environnement et fournir un contexte aux enquêteurs sur la sécurité afin d’éclairer leurs décisions.

Vous pouvez intégrer le renseignement sur les menaces à Azure Sentinel par le biais des activités suivantes :

- Utilisez des **Connecteurs de données** sur différentes plateformes Threat Intelligence pour [importer le renseignement sur les menaces](./connect-threat-intelligence.md) dans Azure Sentinel.
- Affichez et gérez le renseignement sur les menaces importé dans des **Journaux** et dans la nouvelle zone **Threat Intelligence** d’Azure Sentinel.
- Utilisez les modèles de règle d’**analytique** intégrés pour générer des alertes et des incidents de sécurité à l’aide de votre renseignement sur les menaces importé.
- Visualisez les informations essentielles concernant votre renseignement sur les menaces dans Azure Sentinel à l’aide du **classeur Threat Intelligence**.

Le renseignement sur les menaces fournit également un contexte utile dans d’autres expériences Azure Sentinel, comme la **Chasse** et les **Notebooks**, et bien qu’elles ne soient pas abordées dans cet article, ces expériences sont traitées dans [ce billet de blog très intéressant d’Ian Hellen sur les notebooks Jupyter dans Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/using-threat-intelligence-in-your-jupyter-notebooks/ba-p/860239), qui traite de l’utilisation du renseignement sur les cybermenaces dans les notebooks.

## <a name="azure-sentinel-data-connectors-for-threat-intelligence"></a>Connecteurs de données Azure Sentinel pour le renseignement sur les menaces

Comme toutes les autres données d’événement dans Azure Sentinel, les indicateurs de menace sont importés à l’aide de connecteurs de données. Il existe deux connecteurs de données dans Azure Sentinel, fournis spécifiquement pour les indicateurs de menace : **Threat Intelligence - TAXII** et **Plateformes Threat Intelligence**. Vous pouvez utiliser soit un connecteur de données seul, soit les deux connecteurs ensemble, en fonction de l’endroit d’où proviennent les indicateurs de menace utilisés par votre organisation. Examinons chacun des connecteurs de données.

### <a name="adding-threat-indicators-to-azure-sentinel-with-the-threat-intelligence-platforms-data-connector"></a>Ajout d’indicateurs de menace à Azure Sentinel avec le connecteur de données Plateformes Threat Intelligence

De nombreuses organisations utilisent des solutions de plateforme Threat Intelligence (TIP) pour agréger les flux d’indicateurs de menace provenant de diverses sources, organiser les données au sein de la plateforme, puis choisir les indicateurs de menace à appliquer à différentes solutions de sécurité, comme les appareils réseau, les solutions de protection avancée contre les menaces ou les solutions SIEM comme Azure Sentinel. Si votre organisation utilise une solution TIP intégrée, comme MISP, Anomali ThreatStream, ThreatConnect, EclecticIQ Platform, la plateforme de renseignement sur les menaces ThreatQ ou MineMeld de Palo Alto Networks, le **connecteur de données Plateformes Threat Intelligence** vous permet d’utiliser votre TIP pour importer des indicateurs de menace dans Azure Sentinel. Étant donné que le connecteur fonctionne avec l’[API Microsoft Graph Security tiIndicators](/graph/api/resources/tiindicator) pour effectuer cette opération, le connecteur peut également être utilisé par n’importe quelle plateforme de renseignement sur les menaces personnalisée afin de tirer parti de l’API tiIndicators pour envoyer des indicateurs à Azure Sentinel (et à d’autres solutions de sécurité Microsoft comme Defender ATP).

:::image type="content" source="media/import-threat-intelligence/threat-intel-import-path.png" alt-text="Chemin d’importation du renseignement sur les menaces":::

Effectuez les étapes suivantes pour importer des indicateurs de menace dans Azure Sentinel à partir de votre solution de plateforme Threat Intelligence (TIP) intégrée ou de renseignement sur les menaces personnalisée :

1. Obtenir un ID d’application et un secret client à partir de votre instance Azure Active Directory

1. Entrer ces informations dans votre solution de plateforme Threat Intelligence (TIP) ou votre application personnalisée

1. Activer le connecteur de données Plateformes Threat Intelligence dans Azure Sentinel

Voici un examen détaillé de chacune de ces étapes.

#### <a name="obtain-an-application-id-and-client-secret-from-your-azure-active-directory"></a>Obtenir un ID d’application et un secret client à partir de votre instance Azure Active Directory

Que vous travailliez avec une solution de plateforme Threat Intelligence (TIP) ou avec une solution personnalisée, l’API tiIndicators nécessite certaines informations de base pour se connecter et envoyer des indicateurs de menace. Les trois informations que vous devez obtenir sont les suivantes :
- ID d’application (client)
- ID de l’annuaire (locataire)
- Clé secrète client

Ces informations proviennent toujours de votre instance Azure Active Directory par le biais d’un processus appelé **Inscription d’application**, qui inclut les trois étapes suivantes :
- Inscrire une application dans Azure Active Directory
- Spécifier les autorisations exigées par l’application pour se connecter à l’API tiIndicators Microsoft Graph et envoyer des indicateurs de menace
- Obtenir le consentement de votre organisation pour accorder ces autorisations à cette application  

**Inscrire une application auprès d’Azure Active Directory**

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Active Directory**.

1. Sélectionnez **Inscriptions d’applications** dans le menu, puis sélectionnez **Nouvelle inscription**.

1. Choisissez un nom pour l’inscription de votre application, sélectionnez la case d’option **Locataire unique**, puis sélectionnez **Inscrire**.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-register-application.png" alt-text="Inscrire une application":::

1. Dans l’écran qui s’affiche, copiez les valeurs **ID d’application (client)** et **ID de l’annuaire (locataire)** . Ce sont les deux premières informations dont vous aurez besoin ultérieurement pour configurer votre solution de plateforme Threat Intelligence (TIP) ou votre solution personnalisée afin d’envoyer des indicateurs de menace à Azure Sentinel.

**Spécifier les autorisations exigées par l’application**

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Active Directory**.

1. Sélectionnez **Inscriptions d’applications** dans le menu, puis sélectionnez votre application nouvellement inscrite.

1. Sélectionnez **Autorisations de l’API** dans le menu, puis cliquez sur le bouton **Ajouter une autorisation**.

1. Dans la page **Sélectionner une API**, sélectionnez **Microsoft Graph** et faites votre choix dans la liste d’autorisations Microsoft Graph.

1. Quand vous êtes invité à répondre à la question **Quel type d’autorisation votre application nécessite-t-elle ?** , sélectionnez **Autorisations d’application**. Il s’agit du type d’autorisation utilisé par les applications qui s’authentifient avec un ID d’application et des secrets d’application (clés API).

1. Sélectionnez **ThreatIndicators.ReadWrite.OwnedBy**, puis **Ajouter des autorisations** pour ajouter cette autorisation à la liste d’autorisations de votre application.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-api-permissions-1.png" alt-text="Spécifier des autorisations":::

**Obtenir le consentement de votre organisation pour accorder ces autorisations**

1. Pour accorder un consentement, vous devez disposer d’un administrateur général Azure Active Directory qui sélectionne le bouton **Accorder un consentement d’administrateur pour votre locataire** dans la page des autorisations d’API de votre application. Si vous n’avez pas le rôle Administrateur général sur votre compte, ce bouton n’est pas disponible et vous devez demander à un Administrateur général de votre organisation d’effectuer cette étape.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-api-permissions-2.png" alt-text="Accorder le consentement":::

1. Une fois le consentement accordé à votre application, vous devez voir une coche verte sous **Statut**.
 
Maintenant que votre application a été inscrite et que des autorisations ont été accordées, vous pouvez obtenir le dernier élément figurant sur votre liste : un secret client pour votre application.

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Active Directory**.

1. Sélectionnez **Inscriptions d’applications** dans le menu, puis sélectionnez votre application nouvellement inscrite.

1. Sélectionnez **Certificats et secrets** dans le menu, puis cliquez sur le bouton **Nouveau secret client** pour obtenir un secret (clé API) pour votre application.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-client-secret.png" alt-text="Obtenir un secret client":::

1. Cliquez sur le bouton **Ajouter** et **veillez à copier le secret client**, car vous ne pouvez pas récupérer ce secret si vous quittez cette page. Vous aurez besoin de cette valeur quand vous configurerez votre solution de plateforme Threat Intelligence (TIP) ou votre solution personnalisée.

#### <a name="input-this-information-into-your-tip-solution-or-custom-application"></a>Entrer ces informations dans votre solution de plateforme Threat Intelligence (TIP) ou votre application personnalisée

Vous avez maintenant les trois informations dont vous avez besoin pour configurer votre solution de plateforme Threat Intelligence (TIP) ou votre solution personnalisée afin d’envoyer des indicateurs de menace à Azure Sentinel. 
- ID d’application (client)
- ID de l’annuaire (locataire)
- Clé secrète client

Entrez ces valeurs dans la configuration de votre solution de plateforme Threat Intelligence ou votre solution personnalisée, si nécessaire. Les indicateurs de menace sont alors envoyés par le biais de l’API Microsoft Graph tiIndicators ciblée sur Azure Sentinel.

#### <a name="enable-the-threat-intelligence-platforms-data-connector-in-azure-sentinel"></a>Activer le connecteur de données Plateformes Threat Intelligence dans Azure Sentinel

La dernière étape du processus d’intégration consiste à activer le connecteur de données **Plateformes Threat Intelligence** dans Azure Sentinel. Il s’agit de l’étape qui importe les indicateurs de menace envoyés à partir de votre solution de plateforme Threat Intelligence (TIP) ou de votre solution personnalisée dans Azure Sentinel par le biais de l’API Microsoft Graph tiIndicators. Ces indicateurs sont disponibles pour tous les espaces de travail Azure Sentinel de votre organisation. Effectuez les étapes suivantes pour activer le connecteur de données Plateformes Threat Intelligence pour chaque espace de travail :

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Sentinel**.

1. Choisissez l’**espace de travail** dans lequel vous voulez importer les indicateurs de menace envoyés à partir de votre solution de plateforme Threat Intelligence (TIP) ou votre solution personnalisée.

1. Sélectionnez **Connecteurs de données** dans le menu, sélectionnez **Plateformes Threat Intelligence** dans la galerie des connecteurs, puis cliquez sur le bouton **Ouvrir la page du connecteur**.

1. Comme vous avez déjà effectué l’inscription de l’application et configuré votre solution de plateforme Threat Intelligence (TIP) ou votre solution personnalisée pour envoyer des indicateurs de menace, il ne vous reste plus qu’à cliquer sur le bouton **Se connecter**.

Après quelques minutes, les indicateurs de menace doivent commencer à circuler dans cet espace de travail Azure Sentinel.

### <a name="adding-threat-indicators-to-azure-sentinel-with-the-threat-intelligence---taxii-data-connector"></a>Ajout d’indicateurs de menace à Azure Sentinel avec le connecteur de données Threat Intelligence - TAXII

La norme du secteur la plus largement adoptée pour la transmission du renseignement sur les menaces est une [combinaison du format de données STIX et du protocole TAXII](https://oasis-open.github.io/cti-documentation/). Si votre organisation obtient des indicateurs de menace à partir de solutions qui prennent en charge la version actuelle de STIX/TAXII (2.0 ou 2.1), vous pouvez utiliser le connecteur de données **Threat Intelligence - TAXII** pour apporter vos indicateurs de menace dans Azure Sentinel. Le connecteur de données Threat Intelligence - TAXII permet à un client TAXII intégré d’Azure Sentinel d’importer le renseignement sur les menaces à partir de serveurs TAXII 2.x.

:::image type="content" source="media/import-threat-intelligence/threat-intel-taxii-import-path.png" alt-text="Chemin d’importation de TAXII":::
 
Effectuez les étapes suivantes pour importer des indicateurs de menace au format STIX dans Azure Sentinel à partir d’un serveur TAXII :

1. Obtenir la racine d’API et l’ID de collection du serveur TAXII

1. Activer le connecteur de données Threat Intelligence - TAXII dans Azure Sentinel

Examinons maintenant attentivement chacune de ces étapes.

#### <a name="obtain-the-taxii-server-api-root-and-collection-id"></a>Obtenir la racine d’API et l’ID de collection du serveur TAXII

Les serveurs TAXII 2.x publient les racines d’API, qui sont des URL hébergeant des collections de renseignement sur les menaces. La plupart du temps, la racine d’API peut être obtenue par le biais de la page de documentation du fournisseur de renseignement sur les menaces qui héberge le serveur TAXII. Toutefois, la seule information publiée est parfois une URL appelée « point de terminaison de découverte ». Si c’est le cas, il est facile de trouver la racine d’API à l’aide du point de terminaison de découverte. Si vous connaissez déjà la racine d’API du serveur TAXII et les ID de collection que vous voulez utiliser, n’hésitez pas à passer à la section suivante, **Activer le connecteur de données Threat Intelligence - TAXII dans Azure Sentinel**.

Examinons un exemple concret d’utilisation d’un utilitaire en ligne de commande simple appelé [cURL](https://en.wikipedia.org/wiki/CURL), qui est fourni dans Windows et la plupart des distributions Linux, pour découvrir la racine d’API et parcourir les collections d’un serveur TAXII démarrant uniquement à partir du point de terminaison de découverte. Pour cet exemple, nous allons utiliser le point de terminaison de découverte du serveur ThreatStream TAXII 2.0 [Anomali Limo](https://www.anomali.com/community/limo).

1.  Dans un navigateur, accédez au [point de terminaison de découverte du serveur ThreatStream TAXII 2.0](https://limo.anomali.com/taxii) pour récupérer la racine d’API. Authentifiez-vous avec l’utilisateur et le mot de passe `guest`.

    ```json
    {
        "api_roots":
        [
            "https://limo.anomali.com/api/v1/taxii2/feeds/",
            "https://limo.anomali.com/api/v1/taxii2/trusted_circles/",
            "https://limo.anomali.com/api/v1/taxii2/search_filters/"
        ],
        "contact": "info@anomali.com",
        "default": "https://limo.anomali.com/api/v1/taxii2/feeds/",
        "description": "TAXII 2.0 Server (guest)",
        "title": "ThreatStream Taxii 2.0 Server"
    }
    ```

2.  Utilisez l’utilitaire cURL et la racine d’API (https://limo.anomali.com/api/v1/taxii2/feeds/) de l’étape précédente pour parcourir la liste des ID de collection hébergés sur la racine d’API.

    ```json
    curl -u guest https://limo.anomali.com/api/v1/taxii2/feeds/collections/
    {
        "collections":
        [
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "107",
                "title": "Phish Tank"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "135",
                "title": "Abuse.ch Ransomware IPs"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "136",
                "title": "Abuse.ch Ransomware Domains"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "150",
                "title": "DShield Scanning IPs"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "200",
                "title": "Malware Domain List - Hotlist"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "209",
                "title": "Blutmagie TOR Nodes"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "31",
                "title": "Emerging Threats C&C Server"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "33",
                "title": "Lehigh Malwaredomains"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "41",
                "title": "CyberCrime"
            },
            {
                "can_read": true,
                "can_write": false,
                "description": "",
                "id": "68",
                "title": "Emerging Threats - Compromised"
            }
        ]
    }
    ```

Vous disposez maintenant de toutes les informations dont vous avez besoin pour connecter Azure Sentinel à une ou plusieurs collections de serveurs TAXII fournies par Anomali Limo.

| **Racine d’API** (https://limo.anomali.com/api/v1/taxii2/feeds/) | ID de la collection |
| ------------------------------------------------------------ | ------------: |
| **Phish Tank**                                               | 107           |
| **Adresses IP de ransomware Abuse.ch**                                  | 135           |
| **Domaines de ransomware Abuse.ch**                              | 136           |
| **Adresses IP d’analyse DShield**                                     | 150           |
| **Liste de domaines de programmes malveillants - Liste critique**                            | 200           |
| **Nœuds TOR de Blutmagie**                                      | 209           |
| **Menaces émergentes - Serveur de commande et contrôle**                              |  31           |
| **Domaines de programmes malveillants Lehigh**                                    |  33           |
| **Cybercrime**                                               |  41           |
| **Menaces émergentes - Compromis**                           |  68           |
|

#### <a name="enable-the-threat-intelligence---taxii-data-connector-in-azure-sentinel"></a>Activer le connecteur de données Threat Intelligence - TAXII dans Azure Sentinel

Pour importer des indicateurs de menace dans Azure Sentinel à partir d’un serveur TAXII, effectuez les étapes suivantes :

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Sentinel**.

1. Choisissez l’**espace de travail** dans lequel vous voulez importer des indicateurs de menace à partir du serveur TAXII.

1. Sélectionnez **Connecteurs de données** dans le menu, sélectionnez **Threat Intelligence - TAXII** dans la galerie des connecteurs, puis cliquez sur le bouton **Ouvrir la page du connecteur**.

1. Tapez un **nom** pour cette collection de serveurs TAXII, une **URL de la racine d’API**, un **ID de collection**, un **Nom d’utilisateur** (si nécessaire) et un **Mot de passe** (si nécessaire), puis cliquez sur le bouton **Ajouter**.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-configure-taxii-servers.png" alt-text="Configurer des serveurs TAXII":::
 
Vous devez recevoir la confirmation qu’une connexion au serveur TAXII a été établie avec succès, et vous pouvez répéter l’étape (4) ci-dessus autant de fois que vous le souhaitez pour vous connecter à plusieurs collections à partir du même serveur TAXII ou de serveurs différents.

## <a name="view-your-threat-indicators-in-azure-sentinel"></a>Consulter vos indicateurs de menace dans Azure Sentinel

Maintenant que vous avez correctement importé des indicateurs de menace dans Azure Sentinel à l’aide du connecteur de données **Plateformes Threat Intelligence** et/ou du connecteur de données **Threat Intelligence - TAXII**, vous pouvez les voir dans la table **ThreatIntelligenceIndicator** des **Journaux**, où sont stockées toutes vos données d’événement Azure Sentinel. Cette table est la base des requêtes effectuées par d’autres fonctionnalités Azure Sentinel, comme l’Analytique et les Classeurs. Voici comment rechercher et consulter vos indicateurs de menace dans la table **ThreatIntelligenceIndicator**.

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Sentinel**.

1. Choisissez l’**espace de travail** dans lequel vous avez importé des indicateurs de menace à l’aide de l’un ou l’autre des connecteurs de données du renseignement sur les menaces.

1. Sélectionnez **Journaux** dans la section **Général** du menu Azure Sentinel.

1. La table **ThreatIntelligenceIndicator** se trouve sous le groupe **Azure Sentinel**.

1. Sélectionnez l’icône **Aperçu des données** (représentant un œil) en regard du nom de la table, puis sélectionnez le bouton **Afficher dans l’éditeur de requête** pour exécuter une requête qui affiche les enregistrements de cette table.

Vos résultats doivent ressembler à l’exemple d’indicateur de menace illustré ci-dessous :

:::image type="content" source="media/import-threat-intelligence/threat-intel-sample-query.png" alt-text="Exemple de données de requête":::
 
## <a name="manage-your-threat-indicators-in-the-new-threat-intelligence-area-of-azure-sentinel"></a>Gérer vos indicateurs de menace dans la nouvelle zone Threat Intelligence d’Azure Sentinel

Avec la nouvelle zone **Threat Intelligence**, accessible à partir du menu Azure Sentinel, vous pouvez également afficher, trier, filtrer et rechercher vos indicateurs de menace importés sans même écrire de requête de **journaux**. Cette nouvelle zone vous permet également de créer des indicateurs de menace directement dans l’interface Azure Sentinel, ainsi que d’effectuer des tâches d’administration courantes concernant le renseignement sur les menaces, comme l’étiquetage des indicateurs et la création d’indicateurs liés aux enquêtes de sécurité.
Examinons deux des tâches les plus courantes : la création d’indicateurs de menace et l’étiquetage des indicateurs pour faciliter leur regroupement et leur référence.

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Sentinel**.

1. Choisissez l’**espace de travail** dans lequel vous avez importé des indicateurs de menace à l’aide de l’un ou l’autre des connecteurs de données du renseignement sur les menaces.

1. Sélectionnez **Threat Intelligence** dans la section Gestion des menaces du menu Azure Sentinel.

1. Sélectionnez le bouton **Ajouter nouveau** dans le menu en haut de la page.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-add-new-indicator.png" alt-text="Ajouter un nouvel indicateur de menace" lightbox="media/import-threat-intelligence/threat-intel-add-new-indicator.png":::

1. Choisissez le type d’indicateur, puis renseignez les champs obligatoires marqués d’un astérisque rouge (*) dans le panneau **Nouvel indicateur**.

1. Sélectionnez **Appliquer**. L’indicateur est ajouté à la grille Indicateurs, et est également envoyé à la table ThreatIntelligenceIndicator dans les **Journaux**.

L’étiquetage des indicateurs de menace est un moyen simple de les regrouper pour faciliter leur recherche. En général, vous pouvez appliquer une étiquette à des indicateurs liés à un incident particulier ou à des indicateurs représentant les menaces provenant d’un acteur connu particulier ou d’une campagne d’attaque bien connue. Vous pouvez étiqueter les indicateurs de menace individuellement, ou sélectionner plusieurs indicateurs et les étiqueter tous en même temps. L’exemple ci-dessous montre l’étiquetage de plusieurs indicateurs avec un ID d’incident. Étant donné que l’étiquetage est de forme libre, il est recommandé de créer des conventions de nommage standard pour les étiquettes d’indicateur de menace. Vous pouvez appliquer plusieurs étiquettes à chaque indicateur.

:::image type="content" source="media/import-threat-intelligence/threat-intel-tagging-indicators.png" alt-text="Appliquer des étiquettes à des indicateurs de menace" lightbox="media/import-threat-intelligence/threat-intel-tagging-indicators.png":::

## <a name="analytics-puts-your-threat-indicators-to-work-detecting-potential-threats"></a>L’analytique exploite vos indicateurs de menace pour détecter des menaces potentielles

Vos indicateurs de menace sont injectés dans Azure Sentinel. Vous avez vu comment les voir et les gérer. Voyons maintenant ce qu’ils peuvent faire pour vous. Le cas d’usage le plus important pour les indicateurs de menace dans les solutions SIEM comme Azure Sentinel consiste à fournir des règles d’analytique.  Ces règles basées sur les indicateurs comparent les événements bruts provenant de vos sources de données avec vos indicateurs de menace pour détecter les menaces de sécurité qui pèsent sur votre organisation. Dans l’option **Analytique** d’Azure Sentinel, vous créez des règles d’analytique qui s’exécutent selon une planification et qui génèrent des alertes de sécurité. Les règles sont pilotées par des requêtes, ainsi que par des configurations qui déterminent la fréquence à laquelle la règle doit s’exécuter, le type de résultats de requête qui doivent générer des alertes de sécurité et toutes les réponses automatisées à déclencher quand des alertes sont générées.

Alors que vous pouvez toujours créer des règles d’analytique à partir de zéro, Azure Sentinel fournit un ensemble de modèles de règle intégrés, créés par des ingénieurs Sécurité Microsoft, que vous pouvez utiliser tels quels ou modifier pour répondre à vos besoins. Vous pouvez facilement identifier les modèles de règle qui utilisent des indicateurs de menace, car ils ont tous un nom qui commence par « **TI map**... » (Carte de renseignement sur les menaces...). Tous ces modèles de règle fonctionnent de la même manière, la seule différence étant le type d’indicateurs de menace utilisés (domaine, e-mail, hachage de fichier, adresse IP ou URL) et le type d’événements à mettre en correspondance. Chaque modèle liste les sources de données nécessaires pour que la règle fonctionne, de sorte que vous pouvez voir d’un coup d’œil si les événements nécessaires sont déjà importés dans Azure Sentinel.

Examinons l’un de ces modèles de règle et découvrons comment activer et configurer la règle pour générer des alertes de sécurité à l’aide des indicateurs de menace que vous avez importés dans Azure Sentinel. Pour cet exemple, nous allons utiliser le modèle de règle appelé **TI map IP entity to AzureActivity** (Carte de renseignement sur les menaces - Entité IP sur AzureActivity). Cette règle fait correspondre n’importe quel indicateur de menace de type adresse IP à tous vos événements d’activité Azure. Quand une correspondance est trouvée, une **alerte** est générée, ainsi qu’un **incident** correspondant en vue d’une enquête par votre équipe en charge des opérations de sécurité. Cette règle d’analytique ne fonctionnera correctement que si vous avez activé l’un des connecteurs de données **Threat Intelligence** ou les deux (pour importer des indicateurs de menace), et le connecteur de données **Activité Azure** (pour importer vos événements au niveau de l’abonnement Azure).

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Sentinel**.

1. Choisissez l’**espace de travail** dans lequel vous avez importé des indicateurs de menace à l’aide des connecteurs de données **Threat Intelligence** et les données d’activité Azure à l’aide du connecteur de données **Azure Activity**.

1. Sélectionnez **Analytics** dans la section **Configuration** du menu Azure Sentinel.

1. Sélectionnez l’onglet **Modèles de règle** pour voir la liste des modèles de règle d’analytique disponibles.

1. Accédez à la règle intitulée **TI map IP entity to AzureActivity** (Carte de renseignement sur les menaces - Entité IP sur AzureActivity), puis vérifiez que vous avez connecté toutes les sources de données nécessaires, comme indiqué ci-dessous.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-required-data-sources.png" alt-text="Sources de données nécessaires":::

1. Sélectionnez cette règle, puis sélectionnez le bouton **Créer une règle**. Cette opération ouvre un Assistant permettant de configurer la règle. Renseignez les paramètres affichés ici, puis sélectionnez le bouton **Suivant : Définir la logique de la règle >** .

    :::image type="content" source="media/import-threat-intelligence/threat-intel-create-analytics-rule.png" alt-text="Créer une règle d’analytique":::

1. La partie logique de règle de l’Assistant contient les éléments suivants :
    - La requête qui sera utilisée dans la règle.

    - Les mappages d’entité, qui indiquent à Azure Sentinel comment reconnaître des entités comme des comptes, des adresses IP et des URL, afin que les **incidents** et les **enquêtes** comprennent comment utiliser les données se trouvant dans toute alerte de sécurité générée par cette règle.

    - La planification de l’exécution de cette règle.

    - Le nombre de résultats de requête nécessaires avant qu’une alerte de sécurité soit générée.

    Les paramètres par défaut du modèle sont les suivants :
    - Exécuter une fois par heure.

    - Faire correspondre tous les indicateurs de menace d’adresse IP de la table **ThreatIntelligenceIndicator** avec toute adresse IP trouvée au cours de la dernière heure d’événements de la table **AzureActivity**.

    - Générer une alerte de sécurité si les résultats de requête sont supérieurs à zéro, c’est-à-dire si des correspondances sont trouvées.

Vous pouvez conserver les paramètres par défaut ou changer ceux nécessaires pour répondre à vos besoins. Quand vous avez terminé, sélectionnez le bouton **Suivant : Réponse automatisée >** .

1. Cette étape de l’Assistant vous permet de configurer une automatisation que vous souhaitez déclencher quand une alerte de sécurité est générée à partir de cette règle d’analytique. Dans Azure Sentinel, l’automatisation est effectuée à l’aide de **playbooks** fournis par Azure Logic Apps. Pour en savoir plus, consultez ce [Tutoriel : Configurer des réponses automatisées aux menaces dans Azure Sentinel](./tutorial-respond-threats-playbook.md). Pour cet exemple, nous allons simplement sélectionner le bouton **Suivant : Réviser >** pour continuer.

1. Cette dernière étape valide les paramètres définis dans votre règle. Quand vous êtes prêt à activer la règle, sélectionnez le bouton **Créer**. Vous avez terminé.

Maintenant que vous avez activé votre règle d’analytique, vous pouvez trouver votre règle activée sous l’onglet **Règles actives** de la section **Analytique** d’Azure Sentinel. Vous pouvez modifier, activer, désactiver, dupliquer ou supprimer la règle active à partir de cet emplacement. La nouvelle règle s’exécute immédiatement lors de l’activation, et à compter de ce moment, s’exécute selon la planification définie pour elle.

Selon les paramètres par défaut, chaque fois que la règle s’exécute selon sa planification, tous les résultats trouvés génèrent une alerte de sécurité. Dans Azure Sentinel, les alertes de sécurité peuvent être consultées dans la section **Journaux** d’Azure Sentinel, dans la table **SecurityAlert** située sous le groupe **Azure Sentinel**.

Dans Azure Sentinel, les alertes générées à partir des règles d’analytique génèrent également des incidents de sécurité qui se trouvent dans **Incidents** sous **Gestion des menaces**  dans le menu Azure Sentinel. Les incidents sont ce que vos équipes chargées des opérations de sécurité trient et examinent pour déterminer les actions de réponse appropriées. Vous trouverez des informations détaillées dans ce [Tutoriel : Examiner les incidents avec Azure Sentinel](./tutorial-investigate-cases.md).

## <a name="workbooks-provide-insights-about-your-threat-intelligence"></a>Les classeurs fournissent des insights concernant le renseignement sur les menaces

Enfin, vous pouvez utiliser un classeur Azure Sentinel pour visualiser des informations essentielles concernant votre renseignement sur les menaces dans Azure Sentinel, et vous pouvez facilement personnaliser les classeurs en fonction des besoins de votre entreprise.
Examinons comment trouver le classeur Threat Intelligence fourni dans Azure Sentinel et comment apporter des modifications à ce classeur pour le personnaliser.

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez au service **Azure Sentinel**.

1. Choisissez l’**espace de travail** dans lequel vous avez importé des indicateurs de menace à l’aide de l’un ou l’autre des connecteurs de données du renseignement sur les menaces.

1. Sélectionnez **Classeurs** dans la section **Gestion des menaces** du menu Azure Sentinel.

1. Accédez au classeur intitulé **Threat Intelligence**, puis vérifiez que vous avez des données dans la table **ThreatIntelligenceIndicator**, comme indiqué ci-dessous.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-verify-data.png" alt-text="Vérifier les données":::
 
1. Sélectionnez le bouton **Enregistrer**, puis choisissez un emplacement Azure pour stocker le classeur. Cette étape est nécessaire si vous envisagez de modifier le classeur de quelque façon que ce soit et d’enregistrer les modifications apportées.

1. Sélectionnez maintenant le bouton **Afficher le workbook enregistré** pour ouvrir le classeur afin de le consulter et de le modifier.

1. Vous devez maintenant voir les graphiques par défaut fournis par le modèle. Apportons à présent des changements à l’un des graphiques. Sélectionnez le bouton **Modifier** en haut de la page pour passer en mode édition pour le classeur.

1. Ajoutons un nouveau graphique des indicateurs de menace par type de menace. Faites défiler l’écran vers le bas de la page, puis sélectionnez Ajouter une requête.

1. Ajoutez le texte suivant dans la zone de texte **Journal des requêtes de l’espace de travail Log Analytics** :
    ```kusto
    ThreatIntelligenceIndicator
    | summarize count() by ThreatType
    ```

1. Dans la liste déroulante **Visualisation**, sélectionnez **Graphique à barres**.

1. Sélectionnez le bouton **Modification terminée**. Vous avez créé un graphique pour votre classeur.

    :::image type="content" source="media/import-threat-intelligence/threat-intel-bar-chart.png" alt-text="Graphique à barres":::

Les classeurs fournissent des tableaux de bord interactifs puissants qui vous donnent des insights de tous les aspects d’Azure Sentinel. Il y a beaucoup de choses que vous pouvez faire avec des classeurs, et même si les modèles fournis constituent un excellent point de départ, vous voudrez probablement aller plus loin en personnalisant ces modèles, ou en créant des tableaux de bord qui combinent de nombreuses sources de données différentes pour vous permettre de visualiser vos données de manière unique. Étant donné que les classeurs Azure Sentinel sont basés sur des classeurs Azure Monitor, une documentation complète et de nombreux autres modèles sont déjà disponibles. Cet article est idéal pour découvrir comment [Créer des rapports interactifs avec les classeurs Azure Monitor](../azure-monitor/platform/workbooks-overview.md). 

Il existe également une vaste communauté de [classeurs Azure Monitor sur GitHub](https://github.com/microsoft/Application-Insights-Workbooks) où vous pouvez télécharger des modèles supplémentaires et partager vos propres modèles.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez découvert les fonctionnalités du renseignement sur les menaces d’Azure Sentinel, ainsi que le nouveau panneau Threat Intelligence. Pour obtenir des conseils pratiques sur l’utilisation des fonctionnalités du renseignement sur les menaces d’Azure Sentinel, consultez les articles suivants :

- [Connecter des données de renseignement sur les menaces](./connect-threat-intelligence.md) à Azure Sentinel.
- Créer des alertes [intégrées](./tutorial-detect-threats-built-in.md) ou [personnalisées](./tutorial-detect-threats-custom.md), et [examiner](./tutorial-investigate-cases.md) des incidents dans Azure Sentinel.