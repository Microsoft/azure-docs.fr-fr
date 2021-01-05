---
title: Connecter des données de renseignement sur les menaces à Azure Sentinel | Microsoft Docs
description: Découvrez comment connecter des données de veille contre les menaces à Azure Sentinel.
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/22/2019
ms.author: yelevin
ms.openlocfilehash: 205cc6eea5d1ac3be2d0e266621067dc8e20d2f9
ms.sourcegitcommit: b8a175b6391cddd5a2c92575c311cc3e8c820018
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96121745"
---
# <a name="connect-data-from-threat-intelligence-providers"></a>Connecter des données issues de fournisseurs de veille contre les menaces

> [!IMPORTANT]
> Les connecteurs de données Threat Intelligence dans Azure Sentinel sont actuellement en préversion publique.
> Cette fonctionnalité est fournie sans contrat de niveau de service et est déconseillée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Sentinel vous permet d’importer les indicateurs de menace que votre organisation utilise, ce qui peut améliorer la capacité de vos analystes de sécurité à détecter et hiérarchiser les menaces connues. Plusieurs fonctionnalités d’Azure Sentinel deviennent ensuite disponibles ou sont améliorées :

- L’**analytique** comprend un ensemble de modèles de règles planifiées que vous pouvez activer pour générer des alertes et des incidents basés sur des correspondances d’événements de journal de vos indicateurs de menace.

- Les **classeurs** fournissent des informations résumées sur les indicateurs de menace importés dans Azure Sentinel et toutes les alertes générées à partir des règles d’analytique correspondant à vos indicateurs de menace.

- Les requêtes de **chasse** permettent aux investigateurs de sécurité d’utiliser des indicateurs de menace dans le cadre de scénarios de chasse courants.

- Les **notebooks** peuvent utiliser des indicateurs de menace lorsque vous enquêtez sur des anomalies et recherchez des comportements malveillants.

Vous pouvez transmettre en continu des indicateurs de menace à Azure Sentinel en utilisant l’un des produits de la plateforme Threat Intelligence intégrée listés dans la section suivante, en le connectant à des serveurs TAXII ou en utilisant une intégration directe avec l’[API Microsoft Graph Security tiIndicators](/graph/api/resources/tiindicator).

## <a name="integrated-threat-intelligence-platform-products"></a>Produits de la plateforme Threat Intelligence intégrée

- [Plateforme Threat Intelligence open source MISP](https://www.misp-project.org/)
    
    Pour obtenir un exemple de script qui fournit aux clients des instances MISP pour migrer les indicateurs de menace vers l’API Microsoft Graph Security, voir [MISP pour le script Microsoft Graph Security](https://github.com/microsoftgraph/security-api-solutions/tree/master/Samples/MISP).

- [ThreatStream d’Anomali](https://www.anomali.com/products/threatstream)

    Pour télécharger l’intégrateur et les extensions ThreatStream, ainsi que les instructions pour la connexion de l’intelligence ThreatStream à l’API Microsoft Graph Security, consultez la page des [téléchargements ThreatStream](https://ui.threatstream.com/downloads).

- [Palo Alto Networks MineMeld](https://www.paloaltonetworks.com/products/secure-the-network/subscriptions/minemeld)
    
    Pour obtenir des instructions pas à pas, voir [Envoi d’IOCs à l’API Microsoft Graph Security à l’aide de MineMeld.](https://live.paloaltonetworks.com/t5/MineMeld-Articles/Sending-IOCs-to-the-Microsoft-Graph-Security-API-using-MineMeld/ta-p/258540)

- [Plateforme ThreatConnect](https://threatconnect.com/solution/)

    Pour plus d’informations, consultez [ThreatConnect Integrations](https://threatconnect.com/integrations/) et recherchez Microsoft Graph Security API dans la page.

- [Plateforme EclecticIQ](https://www.eclecticiq.com/solutions)

- [Plateforme Threat Intelligence ThreatQ](https://www.threatq.com/)

    Pour obtenir des informations et des instructions détaillées, consultez [Connecteur Microsoft Sentinel pour l’intégration de ThreatQ](https://appsource.microsoft.com/product/web-apps/threatquotientinc1595345895602.microsoft-sentinel-connector-threatq?src=health&tab=Overview).

## <a name="connect-azure-sentinel-to-your-threat-intelligence-platform"></a>Connecter Azure Sentinel à votre plateforme Threat Intelligence

### <a name="prerequisites"></a>Prérequis  

- Rôle Azure AD d’administrateur général ou d’administrateur de la sécurité pour accorder des autorisations à votre produit ou application personnalisée de plateforme Threat Intelligence qui utilise une intégration directe avec l’API Microsoft Graph Security tiIndicators.

- Autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel pour stocker vos indicateurs de menace.

### <a name="instructions"></a>Instructions

1. [Inscrivez une application](/graph/auth-v2-service#1-register-your-app) dans Azure Active Directory pour recevoir un ID d’application, une clé secrète d’application et un ID de locataire Azure Active Directory. Vous avez besoin de ces valeurs au moment de configurer votre produit ou application intégrés de la plateforme Threat Intelligence qui utilise l’intégration directe avec l’API Microsoft Graph Security tiIndicators.

2. [Configurez des autorisations d’API](/graph/auth-v2-service#2-configure-permissions-for-microsoft-graph) pour l’application inscrite : Ajoutez l’autorisation d’Application Microsoft Graph **ThreatIndicators.ReadWrite.OwnedBy** à votre application inscrite.

3. Demandez à votre administrateur client Azure Active Directory d’accorder un consentement administrateur à l’application inscrite pour votre organisation. À partir du portail Azure : **Azure Active Directory** > **Inscriptions des applications** >  **\<_app name_>**  > **Afficher les autorisations de l’API** > **Accorder un consentement administrateur pour \<_tenant name_>** .

4. Configurez votre produit ou application de la plateforme Threat Intelligence qui utilise une intégration directe avec l’API Microsoft Graph Security tiIndicators pour envoyer des indicateurs à Azure Sentinel en spécifiant les éléments suivants :
    
    a. Valeurs pour l’ID, le secret et l’ID de locataire de l’application inscrite.
    
    b. Pour le produit cible, spécifiez Azure Sentinel.
    
    c. Pour l’action, spécifiez alerte.

5. Dans le portail Azure, accédez à **Azure Sentinel** > **Connecteurs de données**, puis sélectionnez le connecteur **Plateformes Threat Intelligence (préversion)** .

6. Sélectionnez **Ouvrir la page du connecteur**, puis **Connecter**.

7. Pour afficher les indicateurs de menace importés dans Azure Sentinel, accédez à **Azure Sentinel - Logs** > **SecurityInsights**, puis développez **ThreatIntelligenceIndicator**.

## <a name="connect-azure-sentinel-to-taxii-servers"></a>Connecter Azure Sentinel à des serveurs TAXII

### <a name="prerequisites"></a>Prérequis

- Autorisations en lecture et en écriture dans l’espace de travail Azure Sentinel pour stocker vos indicateurs de menace.

- URI du serveur TAXII 2.0 et ID de collection.

### <a name="instructions"></a>Instructions

1. Sur le portail Azure, accédez à **Azure Sentinel** > **Connecteurs de données**, puis sélectionnez le connecteur **Threat Intelligence - TAXII (Préversion)** .

2. Sélectionnez **Ouvrir la page du connecteur**.

3. Spécifiez les informations obligatoires et facultatives dans les zones de texte.

4. Sélectionnez **Ajouter** pour permettre la connexion au serveur TAXII 2.0.

5. Si vous disposez de serveurs TAXII 2.0 supplémentaires : Répétez les étapes 3 et 4.

6. Pour afficher les indicateurs de menace importés dans Azure Sentinel, accédez à **Azure Sentinel - Logs** > **SecurityInsights**, puis développez **ThreatIntelligenceIndicator**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à connecter votre fournisseur d’intelligence des menaces à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, consultez les articles suivants.

- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](./tutorial-detect-threats-built-in.md).