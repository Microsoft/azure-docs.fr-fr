---
title: Détection avancée des attaques multiphases dans Azure Sentinel
description: Utilisez la technologie Fusion d’Azure Sentinel pour réduire l’épuisement des alertes et créer des incidents actionnables basés sur la détection avancée des attaques multiphases.
services: sentinel
documentationcenter: na
author: yelevin
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/30/2020
ms.author: yelevin
ms.openlocfilehash: ba872f221f3bde29f0bb48b04dc2259d3ab4938a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90906280"
---
# <a name="advanced-multistage-attack-detection-in-azure-sentinel"></a>Détection avancée des attaques multiphases dans Azure Sentinel


> [!IMPORTANT]
> Certaines fonctionnalités de Fusion dans Azure Sentinel sont actuellement en **préversion publique**.
> Ces fonctionnalités étant fournies sans contrat de niveau de service, elles sont déconseillées pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

La technologie Fusion basée sur l’apprentissage automatique permet à Azure Sentinel de détecter automatiquement les attaques multiphases en identifiant des combinaisons de comportements anormaux et d’activités suspectes observés à différentes phases de la chaîne de destruction. Sur la base de ces découvertes, Azure Sentinel génère des incidents qui seraient autrement difficiles à intercepter. Ces incidents comprennent au moins deux alertes ou activités. Le système est conçu de façon à ce que ces incidents soient peu volumineux, soient détectés avec une haute fidélité, et présentent un niveau de gravité élevé.

Personnalisée pour votre environnement, cette technologie de détection, en plus de réduire la fréquence de faux positifs, peut détecter des attaques même si les informations sont limitées ou si certaines informations sont manquantes.

## <a name="configuration-for-advanced-multistage-attack-detection"></a>Configuration de la détection avancée des attaques multiphases

Cette détection est activée par défaut dans Azure Sentinel. Pour vérifier l’état de la détection ou pour la désactiver si vous utilisez une autre solution pour créer des incidents sur la base d’alertes multiples, suivez les instructions suivantes :

1. Si ce n’est pas déjà fait, connectez-vous au [portail Azure](https://portal.azure.com).

1. Accédez à **Azure Sentinel** > **Configuration** > **Analytique**.

1. Sélectionnez **Règles actives**, puis, dans la colonne **Nom**, cherchez **Détection avancée des attaques multiphases** en filtrant la liste pour le type de règle **Fusion**. Consultez la colonne **État** pour vérifier si cette détection est activée ou désactivée.

    :::image type="content" source="./media/fusion/selecting-fusion-rule-type.png" alt-text="{alt-text}":::

1. Pour changer l’état, sélectionnez cette entrée, puis, dans le panneau **Détection avancée des attaques multiphases**, sélectionnez **Modifier**.

1. Le changement d’état est sélectionné automatiquement dans le panneau de l’**Assistant de création de règles**. Sélectionnez **Suivant : Réviser**, puis sélectionnez **Enregistrer**. 

 Étant donné que le type de règle **Fusion** ne contient qu’une seule règle qui ne peut pas être modifiée, les modèles de règles ne s’appliquent pas pour ce type de règle.

> [!NOTE]
> Azure Sentinel utilise actuellement 30 jours de données historiques pour former les systèmes de Machine Learning. Ces données sont toujours chiffrées à l’aide des clés de Microsoft à mesure qu’elles passent par le pipeline de Machine Learning. Cependant, les données d’apprentissage ne sont pas chiffrées à l’aide de [clés gérées par le client (CMK)](customer-managed-keys.md) si vous avez activé CMK dans votre espace de travail Azure Sentinel. Pour désactiver la technologie Fusion, accédez à **Azure Sentinel** \> **Configuration** \> **Analytics \> Règles actives \>Détection avancée des attaques multiphases**, puis, dans la colonne **État**, sélectionnez **Désactiver**.

## <a name="attack-detection-scenarios"></a>Scénarios de détection des attaques

La section suivante répertorie les types de scénarios de corrélation, regroupés par classification des menaces, qu’Azure Sentinel recherche à l’aide de la technologie Fusion.

Comme mentionné plus haut, étant donné que la technologie Fusion met en corrélation plusieurs alertes de sécurité provenant de différents produits pour détecter les attaques multiphases avancées, les détections de Fusion réussies sont présentées en tant qu’**Incidents de Fusion** sur la page **Incidents** d’Azure Sentinel, non en tant qu’**Alertes** dans la table **Alertes de sécurité** figurant dans les **Journaux**.

Pour activer ces scénarios de détection des attaques optimisés par Fusion, toutes les sources de données répertoriées doivent être ingérées à l’aide des connecteurs de données Azure Sentinel associés.

> [!NOTE]
> Certains de ces scénarios sont en **préversion publique**. Cela est spécifié.

## <a name="compute-resource-abuse"></a>Abus de ressource de calcul

### <a name="multiple-vm-creation-activities-following-suspicious-azure-active-directory-sign-in"></a>Plusieurs activités de création de machine virtuelle suite à une connexion suspecte à Azure Active Directory
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, impact 

**Techniques MITRE ATT&CK :** Compte valide (T1078), Détournement de ressource (T1496)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal de machines virtuelles ont été créées au cours d’une seule session suite à une connexion suspecte à un compte Azure AD. Ce type d’alerte hautement fiable signifie que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour créer des machines virtuelles à des fins non autorisées, telles que l’exécution d’opérations d’extraction de chiffrement. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte d’activités multiples de création de machines virtuelles sont les suivantes :

- **Déplacement impossible vers un emplacement atypique, à l’origine d’activités multiples de création de machines virtuelles**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine d’activités multiples de création de machines virtuelles**

- **Événement de connexion à partir d’un appareil infecté, à l’origine d’activités multiples de création de machines virtuelles**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine d’activités multiples de création de machines virtuelles**

- **Événement de connexion d’un utilisateur à l’aide d’informations d’identification ayant fuité, à l’origine d’activités multiples de création de machines virtuelles**

## <a name="data-exfiltration"></a>Exfiltration de données

### <a name="office-365-mailbox-exfiltration-following-a-suspicious-azure-ad-sign-in"></a>Exfiltration de boîte aux lettres Office 365 suite à une connexion suspecte à Azure AD

**Tactiques MITRE ATT&CK :** Accès initial, Exfiltration, Collection

**Techniques MITRE ATT&CK :** Compte valide (T1078), Collection d’e-mails (T1114), Exfiltration automatisée (T1020)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type détectés par Fusion indiquent qu’une règle de transfert de boîte de réception a été définie pour la boîte de réception d’un utilisateur suite à une connexion suspecte à un compte Azure AD. Cette indication hautement fiable signifie que le compte de l’utilisateur (indiqué dans la description de l’incident fournie par Fusion) a été compromis et utilisé pour exfiltrer des données du réseau de votre organisation en activant une règle de transfert de boîtes aux lettres à l’insu de l’utilisateur véritable. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte d’exfiltration de boîte aux lettres Office 365 sont les suivantes :

- **Déplacement impossible vers un emplacement atypique, à l’origine de l’exfiltration d’une boîte aux lettres Office 365**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine de l’exfiltration d’une boîte aux lettres Office 365**

- **Événement de connexion à partir d’un appareil infecté à l’origine de l’exfiltration d’une boîte aux lettres Office 365**

- **Événement de connexion à partir d’une adresse IP anonyme à l’origine de l’exfiltration d’une boîte aux lettres Office 365**

- **Événement de connexion d’un utilisateur dont les informations d’identification ont fuité à l’origine de l’exfiltration d’une boîte aux lettres Office 365**

### <a name="mass-file-download-following-suspicious-azure-ad-sign-in"></a>Téléchargement de fichiers en masse suite à une connexion suspecte à Azure AD

**Tactiques MITRE ATT&CK :** Accès initial, Exfiltration

**Techniques MITRE ATT&CK :** Compte valide (T1078)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal de fichiers ont été téléchargés par un utilisateur suite à une connexion suspecte à un compte Azure AD. Cette indication hautement fiable signifie que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour exfiltrer des données du réseau de votre organisation. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte de téléchargement de fichiers en masse sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’un téléchargement de fichiers en masse**

- **Événement de connexion à partir d’un lieu inconnu à l’origine d’un téléchargement de fichiers en masse**

- **Événement de connexion à partir d’un appareil infecté à l’origine d’un téléchargement de fichiers en masse**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine d’un téléchargement de fichiers en masse**

- **Événement de connexion d’un utilisateur à l’aide d’informations d’identification fuitées, à l’origine d’un téléchargement de fichiers en masse**

### <a name="mass-file-sharing-following-suspicious-azure-ad-sign-in"></a>Partage de fichiers en masse suite à une connexion suspecte à Azure AD

**Tactiques MITRE ATT&CK :** Accès initial, Exfiltration

**Techniques MITRE ATT&CK :** Compte valide (T1078), Exfiltration sur service web (T1567)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un certain nombre de fichiers au-dessus d’un seuil particulier ont été partagés avec d’autres personnes suite à une connexion suspecte à un compte Azure AD. Cette indication hautement fiable signifie que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour exfiltrer des données du réseau de votre organisation en partageant des fichiers tels que des documents, des feuilles de calcul, et autres, avec des utilisateurs non autorisés, à des fins malveillantes. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte de partage de fichiers en masse sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’un partage de fichiers en masse**

- **Événement de connexion à partir d’un lieu inconnu à l’origine d’un partage de fichiers en masse**

- **Événement de connexion à partir d’un appareil infecté à l’origine d’un partage de fichiers en masse**

- **Événement de connexion à partir d’une adresse IP anonyme à l’origine d’un partage de fichiers en masse**

- **Événement de connexion d’un utilisateur à l’aide d’informations d’identification fuitées, à l’origine d’un partage de fichiers en masse**

### <a name="suspicious-inbox-manipulation-rules-set-following-suspicious-azure-ad-sign-in"></a>Règles de manipulation de boîte de réception définies suite à une connexion suspecte à Azure AD
Ce scénario appartient à deux classifications de menaces dans cette liste : l’**exfiltration de données** et le **déplacement latéral**. Par souci de clarté, il apparaît dans les deux sections.

Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, Déplacement latéral, Exfiltration

**Techniques MITRE ATT&CK :** Compte valide (T1078), Harponnage interne (T1534)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type détectés par Fusion indiquent que des règles anormales ont été définies pour la boîte de réception d’un utilisateur suite à une connexion suspecte à un compte Azure AD. Cela constitue une indication hautement fiable que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour manipuler les règles de la boîte de réception de l’utilisateur à des fins malveillantes. Il peut s’agir d’une tentative d’un attaquant d’exfiltrer des données à partir du réseau de l’organisation. L’attaquant pourrait également tenter de générer des e-mails de hameçonnage à partir de l’intérieur de l’organisation (en contournant les mécanismes de détection de hameçonnage ciblant le courrier en provenance de sources externes) à des fins de déplacement latéral en accédant à d’autres comptes utilisateur et/ou privilégiés. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte des règles de manipulation suspecte de boîte de réception sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion à partir d’un appareil infecté, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion d’utilisateur à l’aide d’informations d’identification fuitées**

### <a name="multiple-power-bi-report-sharing-activities-following-suspicious-azure-ad-sign-in"></a>Activités multiples de partage de rapport Power BI suite à une connexion suspecte à Azure AD 
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, Exfiltration 

**Techniques MITRE ATT&CK :** Compte valide (T1078), Exfiltration sur service web (T1567)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal rapports Power BI ont été partagés au cours d’une seule session suite à une connexion suspecte à un compte Azure AD. Cette indication hautement fiable signifie que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour exfiltrer des données du réseau de votre organisation en partageant des rapports Power BI avec des utilisateurs non autorisés, à des fins malveillantes. Les permutations d’alertes de connexion suspecte à Azure AD avec les multiples activités de partage de rapport Power BI sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’activités multiples de partage de rapport Power BI**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine d’activités multiples de partage de rapport Power BI**

- **Événement de connexion à partir d’une appareil infecté, à l’origine d’activités multiples de partage de rapport Power BI**

- **Événement de connexion d’une adresse IP anonyme, à l’origine d’activités multiples de partage de rapport Power BI**

- **Événement de connexion d’utilisateur avec des informations d’identification fuitées, à l’origine d’activités multiples de partage de rapport Power BI**

### <a name="suspicious-power-bi-report-sharing-following-suspicious-azure-ad-sign-in"></a>Partage de rapport Power BI suite à une connexion suspecte à Azure AD
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, Exfiltration 

**Techniques MITRE ATT&CK :** Compte valide (T1078), Exfiltration sur service web (T1567)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type détectés par Fusion indiquent qu’une activité suspecte de partage de rapport Power BI s’est produite suite à une connexion suspecte à un compte Azure AD. L’activité de partage a été identifiée comme suspecte parce que le rapport Power BI contenait des informations sensibles identifiées à l’aide d’un traitement en langage naturel, et a été partagé avec une adresse e-mail externe publiée sur le web ou remis en tant que capture instantanée à une adresse e-mail abonnée en externe. Cette alerte hautement fiable signifie que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour exfiltrer des données sensibles de votre organisation en partageant des rapports Power BI avec des utilisateurs non autorisés à des fins malveillantes. Les permutations d’alertes de connexion suspecte à Azure AD avec le partage suspect de rapport Power BI sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’un partage suspect de rapport Power BI**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine d’un partage suspect de rapport Power BI**

- **Événement de connexion à partir d’un appareil infecté, à l’origine d’un partage suspect de rapport Power BI**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine d’un partage suspect de rapport Power BI**

- **Événement de connexion d’utilisateur avec des informations d’identification fuitées, à l’origine d’un partage suspect de rapport Power BI**

## <a name="data-destruction"></a>Destruction de données

### <a name="mass-file-deletion-following-suspicious-azure-ad-sign-in"></a>Suppression de fichiers en masse suite à une connexion suspecte à Azure AD

**Tactiques MITRE ATT&CK :** Accès initial, impact

**Techniques MITRE ATT&CK :** Compte valide (T1078), Destruction de données (T1485)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal de fichiers uniques ont été supprimés suite à une connexion suspecte à un compte Azure AD. Cela indique que le compte signalé dans la description de l’incident par Fusion a peut-être été compromis et a été utilisé pour détruire des données à des fins malveillantes. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte de suppression de fichiers en masse sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’une suppression de fichiers en masse**

- **Événement de connexion à partir d’un lieu inconnu à l’origine d’une suppression de fichiers en masse**

- **Événement de connexion à partir d’un appareil infecté à l’origine d’une suppression de fichiers en masse**

- **Événement de connexion à partir d’une adresse IP anonyme à l’origine d’une suppression de fichiers en masse**

- **Événement de connexion d’un utilisateur avec des informations d’identification fuitées, à l’origine d’une suppression de fichiers en masse**

### <a name="suspicious-email-deletion-activity-following-suspicious-azure-ad-sign-in"></a>Activité suspecte de suppression de courrier suite à une connexion suspecte à Azure AD
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, impact 

**Techniques MITRE ATT&CK :** Compte valide (T1078), Destruction de données (T1485)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal d’e-mails ont été supprimés au cours d’une seule session suite à une connexion suspecte à un compte Azure AD. Cela indique que le compte signalé dans la description de l’incident par Fusion a peut-être été compromis et a été utilisé pour détruire des données à des fins malveillantes, telles que nuire à l’organisation ou masquer une activité de courrier indésirable. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte d’activité suspecte de suppression de courrier sont les suivantes :   

- **Déplacement impossible vers un emplacement atypique, à l’origine d’une activité suspecte de suppression de courrier**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine d’une activité suspecte de suppression de courrier**

- **Événement de connexion à partir d’un appareil infecté, à l’origine d’une activité suspecte de suppression de courrier**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine d’une activité suspecte de suppression de courrier**

- **Événement de connexion d’utilisateur avec des informations d’identification fuitées, à l’origine d’une activité suspecte de suppression de courrier**

## <a name="denial-of-service"></a>Denial of service (déni de service)

### <a name="multiple-vm-delete-activities-following-suspicious-azure-ad-sign-in"></a>Activités multiples de suppression de machines virtuelles suite à une connexion suspecte à Azure AD
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, impact

**Techniques MITRE ATT&CK :** Compte valide (T1078), Déni de service de point de terminaison (T1499)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal de machines virtuelles ont été supprimées au cours d’une seule session suite à une connexion suspecte à un compte Azure AD. Cette indication hautement fiable signifie que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour tenter d’interrompre ou de détruire l’environnement cloud de l’organisation. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte d’activités multiples de suppression de machines virtuelles sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’activités multiples de suppression de machines virtuelles**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine d’activités multiples de suppression de machines virtuelles**

- **Événement de connexion à partir d’un appareil infecté, à l’origine d’activités multiples de suppression de machines virtuelles**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine d’activités multiples de suppression de machines virtuelles**

- **Événement de connexion d’un utilisateur à l’aide d’informations d’identification ayant fuité, à l’origine d’activités multiples de suppression de machines virtuelles**

## <a name="lateral-movement"></a>Mouvement latéral

### <a name="office-365-impersonation-following-suspicious-azure-ad-sign-in"></a>Emprunt d’identité d’Office 365 suite à une connexion suspecte à Azure AD

**Tactiques MITRE ATT&CK :** Accès initial, Déplacement latéral

**Techniques MITRE ATT&CK :** Compte valide (T1078), Harponnage interne (T1534)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal d’actions d’emprunt d’identité ont eu lieu suite à une connexion suspecte à partit d’un compte Azure AD. Dans certains logiciels, il existe des options permettant aux utilisateurs d’emprunter l’identité d’autres utilisateurs. Par exemple, les services de courrier permettent aux utilisateurs d’autoriser d’autres utilisateurs à envoyer des e-mails en leur nom. Cette alerte indique avec une degré de fiabilité élevé que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour conduire des activités d’emprunt d’identité à des fins malveillantes, telles que l’envoi de messages de hameçonnage pour la distribution de logiciels malveillants ou un déplacement latéral. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte d’emprunt d’identité Office 365 sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’un emprunt d’identité Office 365**

- **Événement de connexion à partir d’un lieu inconnu à l’origine d’un emprunt d’identité Office 365**

- **Événement de connexion à partir d’un appareil infecté à l’origine d’un emprunt d’identité Office 365**

- **Événement de connexion à partir d’une adresse IP anonyme à l’origine d’un emprunt d’identité Office 365**

- **Événement de connexion d’un utilisateur dont les informations d’identification ont fuité à l’origine d’un emprunt d’identité Office 365**
 
### <a name="suspicious-inbox-manipulation-rules-set-following-suspicious-azure-ad-sign-in"></a>Règles de manipulation de boîte de réception définies suite à une connexion suspecte à Azure AD
Ce scénario appartient à deux classifications de menaces dans cette liste : le **déplacement latéral** et l’**exfiltration de données**. Par souci de clarté, il apparaît dans les deux sections.

Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, Déplacement latéral, Exfiltration

**Techniques MITRE ATT&CK :** Compte valide (T1078), Harponnage interne (T1534), Exfiltration automatisée (T1020)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type détectés par Fusion indiquent que des règles anormales ont été définies pour la boîte de réception d’un utilisateur suite à une connexion suspecte à un compte Azure AD. Cette indication hautement fiable signifie que le compte noté dans la description de l’incident par Fusion a été compromis et utilisé pour manipuler les règles de la boîte de réception de l’utilisateur à des fins malveillantes. Il peut s’agir d’une tentative d’un attaquant d’exfiltrer des données à partir du réseau de l’organisation. L’attaquant pourrait également tenter de générer des e-mails de hameçonnage à partir de l’intérieur de l’organisation (en contournant les mécanismes de détection de hameçonnage ciblant le courrier en provenance de sources externes) à des fins de déplacement latéral en accédant à d’autres comptes utilisateur et/ou privilégiés. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte des règles de manipulation suspecte de boîte de réception sont les suivantes :

- **Déplacement impossible vers un emplacement atypique, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion à partir d’un emplacement inhabituel, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion à partir d’un appareil infecté, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine d’une règle de manipulation suspecte de boîte de réception**

- **Événement de connexion d’utilisateur à l’aide d’informations d’identification fuitées**

## <a name="malicious-administrative-activity"></a>Activité administrative malveillante

### <a name="suspicious-cloud-app-administrative-activity-following-suspicious-azure-ad-sign-in"></a>Activité administrative d’application cloud suspecte suite à une connexion suspecte à Azure AD

**Tactiques MITRE ATT&CK :** Accès initial, Persistance, Contournement de défense, Déplacement latéral, Collecte, Exfiltration et Impact

**Techniques MITRE ATT&CK :** N/A

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’un nombre anormal d’activités administratives ont été effectuées au cours d’une seule session suite à une connexion suspecte à Azure AD à partir du même compte. Cela indique que le compte signalé dans la description de l’incident par Fusion a peut-être été compromis et a été utilisé effectuer un nombre quelconque d’actions administratives non autorisées à des fins malveillantes. Cela indique également qu’un compte avec des privilèges administratifs a peut-être été compromis. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte d’activité administrative d’application cloud suspecte sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine d’une activité administrative suspecte d’application cloud**

- **Événement de connexion à partir d’un lieu inconnu à l’origine d’une activité d’administration suspecte d’application cloud**

- **Événement de connexion à partir d’un appareil infecté à l’origine d’une activité d’administration suspecte d’application cloud**

- **Événement de connexion à partir d’une adresse IP anonyme à l’origine d’une activité d’administration suspecte d’application cloud**

- **Événement de connexion d’un utilisateur dont les informations d’identification ont fuité à l’origine d’une activité d’administration suspecte d’application cloud**

## <a name="malicious-execution-with-legitimate-process"></a>Exécution malveillante avec un processus légitime

### <a name="powershell-made-a-suspicious-network-connection-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>PowerShell a établi une connexion réseau suspecte, suivie d’un trafic anormal signalé par le pare-feu Palo Alto Networks.
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Exécution

**Techniques MITRE ATT&CK :** Interpréteur de commandes et de scripts (T1059)

**Sources de connecteur de données :** Microsoft Defender for Endpoint (anciennement Microsoft Defender Advanced Threat Protection, ou MDATP), Palo Alto Networks 

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’une demande de connexion sortante a été effectuée à l’aide d’une commande PowerShell, puis qu’une activité d’entrée anormale a été détectée par le pare-feu Palo Alto Networks. Cela indique qu’un attaquant a probablement obtenu l’accès à votre réseau et tente d’effectuer des actions malveillantes. Les tentatives de connexion par PowerShell qui suivent ce modèle peuvent indiquer une activité de contrôle et de commande de programme malveillant, des demandes de téléchargement de programme malveillant supplémentaires, ou l’établissement d’un accès interactif à distance par une personne malveillante. Comme toutes les attaques « vivant de la terre », cette activité pourrait être une utilisation légitime de PowerShell. Toutefois, l’exécution de la commande PowerShell suivie d’une activité de pare-feu entrante suspecte renforce la suspicion que PowerShell est utilisé de manière malveillante et doit être examiné de plus près. Dans les journaux Palo Alto, Azure Sentinel se concentre sur les [journaux des menaces](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs), et le trafic est considéré comme suspect lorsque des menaces sont autorisées (données, fichiers, saturations, paquets, analyses, logiciels espions, URL, virus, vulnérabilités, virus d’incendie, incendies suspects). Référencez également le journal des menaces Palo Alto correspondant au [type de menace/contenu](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) indiqué dans la description de l’incident par Fusion pour obtenir des détails supplémentaires sur l’alerte.

### <a name="suspicious-remote-wmi-execution-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Exécution suspecte de WMI à distance, suivie d’un trafic anormal signalé par le pare-feu Palo Alto Networks
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Exécution, Détection

**Techniques MITRE ATT&CK :** Windows Management Instrumentation (T1047)

**Sources de connecteur de données :** Microsoft Defender for Endpoint (anciennement MDATP), Palo Alto Networks 

**Description :** Les incidents de ce type signalés par Fusion indiquent que des commandes Windows Management Instrumentation (WMI) ont été exécutées à distance sur un système, et qu’une activité entrante suspecte a été détectée par le pare-feu Palo Alto Networks. Cela indique qu’un attaquant a peut-être eu accès à votre réseau et tente de se déplacer latéralement, d’élever les privilèges et/ou d’exécuter des charges utiles malveillantes. Comme toutes les attaques « vivant de la terre », cette activité pourrait être une utilisation légitime de WMI. Toutefois, l’exécution de la commande WMI suivie d’une activité de pare-feu entrante suspecte renforce la suspicion que WMI est utilisé de manière malveillante et doit être examiné de plus près. Dans les journaux Palo Alto, Azure Sentinel se concentre sur les [journaux des menaces](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs), et le trafic est considéré comme suspect lorsque des menaces sont autorisées (données, fichiers, saturations, paquets, analyses, logiciels espions, URL, virus, vulnérabilités, virus d’incendie, incendies suspects). Référencez également le journal des menaces Palo Alto correspondant au [type de menace/contenu](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) indiqué dans la description de l’incident par Fusion pour obtenir des détails supplémentaires sur l’alerte.

## <a name="malware-c2-or-download"></a>Logiciel malveillant C2 ou téléchargement

### <a name="network-request-to-tor-anonymization-service-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Demande réseau au service d’anonymisation TOR, suivie d’un trafic anormal signalé par le pare-feu de Palo Alto Networks.
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Commande et contrôle

**Techniques MITRE ATT&CK :** Canal chiffré (T1573), Proxy (T1090)

**Sources de connecteur de données :** Microsoft Defender for Endpoint (anciennement MDATP), Palo Alto Networks 

**Description :** Les incidents de ce type signalés par Fusion indiquent qu’une demande de connexion sortante a été adressée au service d’anonymisation TOR, puis qu’une activité d’entrée anormale a été détectée par le pare-feu de Palo Alto Networks. Cela indique qu’un attaquant a probablement obtenu l’accès à votre réseau et tente de dissimuler ses actions et son intention. Les connexions au réseau TOR qui suivent ce modèle peuvent indiquer une activité de contrôle et de commande de programme malveillant, des demandes de téléchargement de programme malveillant supplémentaires, ou l’établissement d’un accès interactif à distance par une personne malveillante. Dans les journaux Palo Alto, Azure Sentinel se concentre sur les [journaux des menaces](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs), et le trafic est considéré comme suspect lorsque des menaces sont autorisées (données, fichiers, saturations, paquets, analyses, logiciels espions, URL, virus, vulnérabilités, virus d’incendie, incendies suspects). Référencez également le journal des menaces Palo Alto correspondant au [type de menace/contenu](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) indiqué dans la description de l’incident par Fusion pour obtenir des détails supplémentaires sur l’alerte.

### <a name="outbound-connection-to-ip-with-a-history-of-unauthorized-access-attempts-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Connexion sortante à l’adresse IP avec un historique des tentatives d’accès non autorisées suivie d’un trafic anormal signalé par le pare-feu Palo Alto Networks
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Commande et contrôle

**Techniques MITRE ATT&CK :** Non applicable

**Sources de connecteur de données :** Microsoft Defender for Endpoint (anciennement MDATP), Palo Alto Networks 

**Description :** Les incidents de fusion de ce type indiquent qu’une connexion sortante à une adresse IP avec un historique de tentatives d’accès non autorisées a été établie, puis qu’une activité anormale a été détectée par le pare-feu Palo Alto Networks. Cela indique qu’un attaquant a probablement obtenu l’accès à votre réseau. Les tentatives de connexion qui suivent ce modèle peuvent indiquer une activité de contrôle et de commande de programme malveillant, des demandes de téléchargement de programme malveillant supplémentaires, ou l’établissement d’un accès interactif à distance par une personne malveillante. Dans les journaux Palo Alto, Azure Sentinel se concentre sur les [journaux des menaces](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs), et le trafic est considéré comme suspect lorsque des menaces sont autorisées (données, fichiers, saturations, paquets, analyses, logiciels espions, URL, virus, vulnérabilités, virus d’incendie, incendies suspects). Référencez également le journal des menaces Palo Alto correspondant au [type de menace/contenu](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) indiqué dans la description de l’incident par Fusion pour obtenir des détails supplémentaires sur l’alerte.

## <a name="ransomware"></a>Ransomware

### <a name="ransomware-execution-following-suspicious-azure-ad-sign-in"></a>Exécution de rançongiciel suite à une connexion suspecte à Azure AD

**Tactiques MITRE ATT&CK :** Accès initial, impact

**Techniques MITRE ATT&CK :** Compte valide (T1078), Données chiffrées pour l’impact (T1486)

**Sources de connecteur de données :** Microsoft Cloud App Security, Azure Active Directory Identity Protection

**Description :** Les incidents de ce type détectés par Fusion signifient qu’un comportement anormal d’un utilisateur indiquant une attaque de rançongiciel a été détecté suite à une connexion suspecte à un compte Azure AD. Cette indication hautement fiable signifie que le compte signalé dans la description de l’incident par Fusion a été compromis et utilisé pour chiffrer les données à des fins d’extorsion à l’encontre du propriétaire des données ou de refus de l’accès du propriétaire des données à ses données. Les permutations d’alertes de connexion suspecte à Azure AD avec l’alerte d’exécution de rançongiciel sont les suivantes :  

- **Déplacement impossible vers un emplacement atypique, à l’origine du déploiement d’un rançongiciel dans une application cloud**

- **Événement de connexion à partir d’un lieu inconnu à l’origine du déploiement d’un rançongiciel dans une application cloud**

- **Événement de connexion à partir d’un appareil infecté à l’origine du déploiement d’un rançongiciel dans une application cloud**

- **Événement de connexion à partir d’une adresse IP anonyme, à l’origine du déploiement d’un rançongiciel dans une application cloud**

- **Événement de connexion d’un utilisateur avec des informations d’identification fuitées, à l’origine du déploiement d’un rançongiciel dans une application cloud**

## <a name="remote-exploitation"></a>Exploitation à distance

### <a name="suspected-use-of-attack-framework-followed-by-anomalous-traffic-flagged-by-palo-alto-networks-firewall"></a>Utilisation présumée d’infrastructure d’attaque, suivie d’un trafic anormal signalé par le pare-feu Palo Alto Networks
Ce scénario est actuellement en **préversion publique**.

**Tactiques MITRE ATT&CK :** Accès initial, Exécution, Déplacement latéral, Élévation des privilèges

**Techniques MITRE ATT&CK :** Exploitation d’application publique (T1190), Exploitation pour l’exécution de client (T1203), Exploitation de services distants (T1210), Exploitation d’élévation des privilèges (T1068)

**Sources de connecteur de données :** Microsoft Defender for Endpoint (anciennement MDATP), Palo Alto Networks 

**Description :** Les incidents de ce type signalés par Fusion indiquent que des utilisations non standard de protocoles, ressemblant à l’utilisation d’infrastructures d’attaque telles que Metasploit, ont été détectées, puis qu’une activité entrante suspecte a été détectée par le pare-feu Palo Alto Networks. Il peut s’agir d’une indication initiale qu’un attaquant a exploité un service pour accéder à vos ressources réseau, ou a déjà obtenu l’accès à celui-ci et tente d’exploiter davantage les systèmes/services disponibles pour déplacer latéralement ou élever des privilèges. Dans les journaux Palo Alto, Azure Sentinel se concentre sur les [journaux des menaces](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/view-and-manage-logs/log-types-and-severity-levels/threat-logs), et le trafic est considéré comme suspect lorsque des menaces sont autorisées (données, fichiers, saturations, paquets, analyses, logiciels espions, URL, virus, vulnérabilités, virus d’incendie, incendies suspects). Référencez également le journal des menaces Palo Alto correspondant au [type de menace/contenu](https://docs.paloaltonetworks.com/pan-os/8-1/pan-os-admin/monitoring/use-syslog-for-monitoring/syslog-field-descriptions/threat-log-fields.html) indiqué dans la description de l’incident par Fusion pour obtenir des détails supplémentaires sur l’alerte.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous en savez plus sur la détection avancée des attaques multiphases, il peut vous être utile de suivre le guide de démarrage suivant pour savoir comment bénéficier d’une visibilité sur vos données et sur les menaces potentielles : [Bien démarrer avec Azure Sentinel](quickstart-get-visibility.md).

Si vous êtes prêt à examiner les incidents créés pour vous, consultez le tutoriel suivant : [Examiner les incidents avec Azure Sentinel](tutorial-investigate-cases.md).

