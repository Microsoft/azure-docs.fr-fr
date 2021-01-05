---
title: Supervision d’intégrité de fichier dans Azure Security Center
description: Découvrez comment comparer des lignes de base avec le Monitoring d’intégrité de fichier dans Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: c8a2a589-b737-46c1-b508-7ea52e301e8f
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/22/2020
ms.author: memildin
ms.openlocfilehash: 89230b6aca9e225c28a1efd0b99476b35f4d8db0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91439553"
---
# <a name="compare-baselines-using-file-integrity-monitoring-fim"></a>Comparer des lignes de base avec le Monitoring d'intégrité de fichier (FIM)

Le Monitoring d'intégrité de fichier vous informe en cas de modifications apportées à des zones sensibles de vos ressources pour vous permettre d'examiner et de corriger toute activité non autorisée. Le Monitoring d'intégrité de fichier surveille les fichiers et les registres Windows, ainsi que les fichiers Linux.

Cette rubrique explique comment activer le Monitoring d'intégrité de fichier sur les fichiers et les registres. Pour plus d’informations sur le Monitoring d'intégrité de fichier, consultez [Monitoring d’intégrité de fichier dans Azure Security Center](security-center-file-integrity-monitoring.md).

## <a name="why-use-fim"></a>Pourquoi l'utiliser le Monitoring d'intégrité de fichier ?

Le système d’exploitation, les applications et les configurations connexes contrôlent le comportement et l'état de sécurité de vos ressources. Dès lors, les attaquants ciblent les fichiers qui contrôlent vos ressources, afin de s'en prendre au système d’exploitation d'une ressource et/ou d’exécuter des activités sans être détectés.

En fait, de nombreuses normes de conformité réglementaires telles que PCI-DSS et ISO 17799 exigent la mise en œuvre de contrôles FIM.  

## <a name="enable-built-in-recursive-registry-checks"></a>Activer les vérifications intégrées et récursives de registre

Par défaut, la ruche du registre du Monitoring d'intégrité de fichier offre un moyen pratique de surveiller les modifications récursives dans des zones de sécurité courante.  Par exemple, un pirate peut configurer un script de manière à ce qu'il s'exécute dans le contexte LOCAL_SYSTEM en configurant une exécution au démarrage ou l'arrêt.  Pour surveiller ce type de modifications, activez la vérification intégrée.  

![Registre](./media/security-center-file-integrity-monitoring-baselines/baselines-registry.png)

>[!NOTE]
> Les vérifications récursives s’appliquent uniquement aux ruches de sécurité recommandées, et pas aux chemins d’accès personnalisés au registre.  

## <a name="add-a-custom-registry-check"></a>Ajouter une vérification de registre personnalisée

Dans un premier temps, le Monitoring d'intégrité de fichier identifie les caractéristiques d’un état valide connu du système d’exploitation.  Pour cet exemple, nous allons nous concentrer sur les configurations de stratégie de mot de passe pour Windows Server 2008 ou version ultérieure.


|Nom de la stratégie                 | Paramètre de registre|
|---------------------------------------|-------------|
|Contrôleur de domaine : Refuser les modifications de mot de passe de compte de machine| MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\RefusePasswordChange|
|Membre de domaine : Chiffrer ou signer numériquement les données des canaux sécurisés (toujours)|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\RequireSignOrSeal|
|Membre de domaine : Chiffrer numériquement les données des canaux sécurisés (si possible)|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\SealSecureChannel|
|Membre de domaine : Signer numériquement les données des canaux sécurisés (si possible)|MACHINE\System\CurrentControlSet\Services   \Netlogon\Parameters\SignSecureChannel|
|Membre de domaine : Désactiver les modifications de mot de passe du compte de machine|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\DisablePasswordChange|
|Membre de domaine : Durée maximale du mot de passe du compte de machine|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\MaximumPasswordAge|
|Membre de domaine : Exiger une clé de session forte (Windows 2000 ou version ultérieure)|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\RequireStrongKey|
|Sécurité réseau : Restreindre NTLM :  Authentification NTLM dans ce domaine|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\RestrictNTLMInDomain|
|Sécurité réseau : Restreindre NTLM : Ajouter des exceptions de serveur dans ce domaine|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\DCAllowedNTLMServers|
|Sécurité réseau : Restreindre NTLM : Auditer l’authentification NTLM dans ce domaine|MACHINE\System\CurrentControlSet\Services  \Netlogon\Parameters\AuditNTLMInDomain|

> [!NOTE]
> Pour en savoir plus sur les paramètres de registre pris en charge par les différentes versions du système d’exploitation, consultez la [feuille de calcul de référence Paramètres de stratégie de groupe](https://www.microsoft.com/download/confirmation.aspx?id=25250).

*Pour configurer le Monitoring intégré de fichier pour surveiller les lignes de base de registre :*

1. Dans la fenêtre **Ajouter le Registre Windows pour Change Tracking**, dans la zone de texte **Clé de Registre Windows**, entrez la clé de Registre.

    <code>

    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters
    </code>

      ![Activer le Monitoring intégré de fichier sur un registre](./media/security-center-file-integrity-monitoring-baselines/baselines-add-registry.png)

## <a name="track-changes-to-windows-files"></a>Suivre les modifications apportées aux fichiers Windows

1. Dans la fenêtre **Ajouter le fichier Windows pour Change Tracking**, dans la zone de texte **Entrer le chemin d'accès**, entrez le dossier contenant les fichiers que vous souhaitez suivre. Dans l’exemple de la figure suivante, **Contoso Web App** se trouve sous le lecteur D:\ dans la structure de dossiers **ContosWebApp**.  
1. Créez une entrée de fichier Windows personnalisée en entrant un nom pour la classe de paramètres, en activant la récursion et en spécifiant le dossier supérieur avec un suffixe de caractère générique (*).

    ![Activer FIM sur un fichier](./media/security-center-file-integrity-monitoring-baselines/baselines-add-file.png)

## <a name="retrieve-change-data"></a>Récupérer les données modifiées

Les données FIM se trouvent dans la table Azure Log Analytics / ConfigurationChange définie.  

 1. Définissez un intervalle de temps pour récupérer un résumé des modifications par ressource.
Dans l’exemple suivant, nous récupérons toutes les modifications apportées aux catégories du registre et aux fichiers au cours des 14 derniers jours :

    <code>

    > ConfigurationChange

    > | where TimeGenerated > ago(14d)

    > | where ConfigChangeType in ('Registry', 'Files')

    > | summarize count() by Computer, ConfigChangeType

    </code>

1. Pour afficher les détails des modifications apportées au registre :

    1. Supprimez **Files** de la clause **where**, 
    1. Supprimez la ligne de résumé et remplacez-la par une clause de classement :

    <code>

    > ConfigurationChange

    > | where TimeGenerated > ago(14d)

    > | where ConfigChangeType in ('Registry')

    > | order by Computer, RegistryKey

    </code>

Les rapports peuvent être exportés au format CSV à des fins d'archivage et/ou dirigés dans un rapport Power BI.  

![Données FIM](./media/security-center-file-integrity-monitoring-baselines/baselines-data.png)