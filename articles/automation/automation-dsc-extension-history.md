---
title: Utiliser l’historique des versions de l’extension Azure Desired State Configuration
description: Cet article explique comment utiliser l’historique des versions de l’extension Desired State Configuration (DSC) dans Azure.
ms.date: 07/22/2020
keywords: dsc, powershell, azure, extension
services: automation
ms.subservice: dsc
ms.topic: conceptual
ms.openlocfilehash: b45512faf09cfe745023d29d32f89a4432cc3b2b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87079755"
---
# <a name="work-with-azure-desired-state-configuration-extension-version-history"></a>Utiliser l’historique des versions de l’extension Azure Desired State Configuration

L’extension de machine virtuelle Configuration d’état souhaité Azure est mise à jour en fonction des besoins pour prendre en charge les améliorations et les nouvelles fonctionnalités fournies par Azure, par Windows Server et par Windows Management Framework qui inclut Windows PowerShell.

Cet article fournit des informations sur chaque version de l’extension de machine virtuelle Configuration d’état souhaité Azure et sur les environnements qu’elle prend en charge. Il contient aussi des commentaires et des remarques sur les nouvelles fonctionnalités ou sur les changements.

## <a name="latest-version"></a>Version la plus récente

### <a name="version-280"></a>Version 2.80

- **Date de publication :**
  - 26 septembre, Sep-2019 (Azure) | 6 juillet 2020 (Azure China 21Vianet) | 20 juillet 2020 (Azure Government)
- **Prise en charge du système d’exploitation :**
  - Windows Server 2019
  - Windows Server 2016
  - Windows Server 2012 R2
  - Windows Server 2012
  - Windows Server 2008 R2 SP1
  - Windows Client 7/8.1/10
  - Nano Server
- **Prise en charge de WMF :**
  - WMF 5.1
  - WMF 5.0 RTM
  - WMF 4.0 Update
  - WMF 4.0
- **Environment :**
  - Azure
  - Azure Chine Vianet 21
  - Azure Government
- **Remarques :** Aucune nouvelle fonctionnalité n’est incluse dans cette version.

### <a name="version-276"></a>Version 2.76

- **Date de publication :**
  - 9 mai 2018 (Azure) | 21 juin 2018 (Azure Chine Vianet 21, Azure Government)
- **Prise en charge du système d’exploitation :**
  - Windows Server 2016
  - Windows Server 2012 R2
  - Windows Server 2012
  - Windows Server 2008 R2 SP1
  - Windows Client 7/8.1/10
  - Nano Server
- **Prise en charge de WMF :**
  - WMF 5.1
  - WMF 5.0 RTM
  - WMF 4.0 Update
  - WMF 4.0
- **Environment :**
  - Azure
  - Azure Chine Vianet 21
  - Azure Government
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - Amélioration des métadonnées d’extension pour le sous-état et autres correctifs de bogues mineurs.

## <a name="supported-versions"></a>Versions prises en charge

> [!WARNING]
> Les versions 2.4 à 2.13 utilisent WMF 5.0 Public Preview dont les certificats de signature ont expiré en août
> 2016. Pour plus d’informations sur ce problème, consultez le [billet de blog](https://devblogs.microsoft.com/powershell/azure-dsc-extension-versions-2-4-up-to-2-13-will-retire-in-august/).

### <a name="version-275"></a>Version 2.75

- **Date de publication :** 5 mars 2018
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10, Nano Server
- **Prise en charge de WMF :** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - Depuis le récent passage de GitHub à TLS 1.2, vous ne pouvez pas intégrer une machine virtuelle dans Azure Automation DSC en utilisant des modèles Resource Manager personnalisés disponibles sur la Place de marché Azure, ni utiliser l’extension DSC pour obtenir des configurations hébergées sur GitHub. Quand vous déployez l’extension, vous voyez une erreur similaire à celle-ci :

    ```json
    {
        "code": "DeploymentFailed",
        "message": "At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.",
        "details": [{
            "code": "Conflict",
            "message": "{
                \"status\": \"Failed\",
                \"error\": {
                    \"code\": \"ResourceDeploymentFailure\",
                    \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.\",
                    \"details\": [ {
                        \"code\": \"VMExtensionProvisioningError\",
                        \"message\": \"VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'.
                        Error message: \\\"The DSC Extension failed to execute: Error downloading
                        https://github.com/Azure/azure-quickstart-templates/raw/master/dsc-extension-azure-automation-pullserver/UpdateLCMforAAPull.zip
                        after 29 attempts: The request was aborted: Could not create SSL/TLS secure channel..\\nMore information about the failure can
                        be found in the logs located under 'C:\\\\WindowsAzure\\\\Logs\\\\Plugins\\\\Microsoft.Powershell.DSC\\\\2.74.0.0' on the VM.\\\".\"
                    } ]
                }
            }"
        }]
    }
    ```

  - Dans la nouvelle version de l’extension, TLS 1.2 est maintenant imposé. Lors du déploiement de l’extension, si vous aviez déjà AutoUpgradeMinorVersion = true dans le modèle Resource Manager, l’extension est automatiquement mise à niveau vers 2.75. Pour les mises à jour manuelles, spécifiez `TypeHandlerVersion = 2.75` dans votre modèle Resource Manager.

### <a name="version-270---272"></a>Version 2.70 - 2.72

- **Date de publication :** 13 novembre 2017
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10, Nano Server
- **Prise en charge de WMF :** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - Correctifs de bogues et améliorations qui simplifient l’utilisation de DSC Azure Automation via l’interface utilisateur du portail et du modèle Resource Manager. Pour plus d’informations, consultez [Script de configuration par défaut](../virtual-machines/extensions/dsc-overview.md) dans la documentation de l’extension DSC.

### <a name="version-226"></a>Version 2.26

- **Date de publication :** 9 juin 2017
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10, Nano Server
- **Prise en charge de WMF :** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - Améliorations de la télémétrie.

### <a name="version-225"></a>Version 2.25

- **Date de publication :** 2 juin 2017
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1/10, Nano Server
- **Prise en charge de WMF :** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - Plusieurs correctifs de bogues et d’autres améliorations mineures ont été ajoutés.

### <a name="version-224"></a>Version 2.24

- **Date de publication :** 13 avril 2017
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Prise en charge de WMF :** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - Expose l’UUID de la machine virtuelle et l’ID de l’agent DSC sous forme de métadonnées de l’extension. D’autres améliorations mineures ont été ajoutées.

### <a name="version-223"></a>Version 2.23

- **Date de publication :** 15 mars 2017
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Prise en charge de WMF :** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - De nombreux correctifs de bogues et d’autres améliorations ont été ajoutés.

### <a name="version-222"></a>Version 2.22

- **Date de publication :** 8 février 2017
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Prise en charge de WMF :** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - L’extension DSC prend désormais en charge WMF 5.1.
  - D’autres améliorations mineures ont été ajoutées.

### <a name="version-221"></a>Version 2.21

- **Date de publication :** 2 décembre 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Nano Server
- **Prise en charge de WMF :** WMF 5.1 Preview, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage). Pour Nano Server, le rôle DSC est installé sur la machine virtuelle.
- **Nouvelles fonctionnalités :**
  - L’extension DSC est désormais disponible sur Nano Server. Cette version contient principalement des modifications du code pour l’exécution de l’extension sur Nano Server.
  - D’autres améliorations mineures ont été ajoutées.

### <a name="version-220"></a>Version 2.20

- **Date de publication :** 2 août 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Prise en charge de WMF :** WMF 5.1 Preview, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016 Technical Preview. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage).
- **Nouvelles fonctionnalités :**
  - Prise en charge de WMF 5.1 Preview. Lors de la première publication, cette version était une mise à niveau facultative, et vous deviez spécifier Wmfversion = '5.1PP' dans les modèles Resource Manager pour installer WMF 5.1 Preview.
    Wmfversion = 'latest' installe toujours [WMF 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/).
    Pour plus d’informations sur WMF 5.1 Preview, consultez [ce blog](https://devblogs.microsoft.com/powershell/announcing-windows-management-framework-wmf-5-1-preview/).
  - D’autres correctifs et améliorations mineures ont été ajoutés.

### <a name="version--219"></a>Version 2.19

- **Date de publication :** 3 juin 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Prise en charge de WMF :** WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure, Azure Chine Vianet 21, Azure Government
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016 Technical Preview. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage).
- **Nouvelles fonctionnalités :**
  - L’extension DSC est désormais intégrée à Azure Chine Vianet 21. Cette version contient principalement des correctifs pour l’exécution de l’extension sur Azure Chine Vianet 21.

### <a name="version-218"></a>Version 2.18

- **Date de publication :** 3 juin 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Prise en charge de WMF :** WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016 Technical Preview. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage).
- **Nouvelles fonctionnalités :**
  - La télémétrie devient non bloquante quand une erreur se produit pendant le téléchargement du correctif de la télémétrie (problème connu d’Azure DNS) ou lors de l’installation.
  - Correctif du problème intermittent où l’extension arrêtait de traiter la configuration après un redémarrage.
    Ceci avait pour effet que l’extension DSC restait dans l’état « en transition ».
  - D’autres correctifs et améliorations mineures ont été ajoutés.

### <a name="version-217"></a>Version 2.17

- **Date de publication :** 26 avril 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Prise en charge de WMF :** WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016 Technical Preview. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage).
- **Nouvelles fonctionnalités :**
  - Prise en charge de WMF 4.0 Update. Pour plus d’informations sur WMF 4.0 Update, consultez [ce blog](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-4-0-update-now-available-for-windows-server-2012-windows-server-2008-r2-sp1-and-windows-7-sp1/).
  - Logique de nouvelle tentative sur les erreurs qui se produisent pendant l’installation de l’extension DSC ou lors de l’application d’une installation de l’extension postérieure à la configuration DSC. Avec cette modification, l’extension réessaye l’installation en cas d’échec d’une installation précédente ou reprend une configuration DSC qui a précédemment échoué, trois fois au maximum jusqu’à atteindre un état d’achèvement (réussite/erreur), ou si une nouvelle demande arrive. Si l’extension échoue en raison d’une entrée utilisateur ou de paramètres utilisateur non valides, elle n’effectue pas de nouvelle tentative. Dans ce cas, l’extension doit être rappelée avec une nouvelle demande et des paramètres utilisateur corrects. Remarque : l’extension DSC dépend de l’agent de machine virtuelle Azure pour les nouvelles tentatives. L’agent de machine virtuelle Azure appelle l’extension avec la dernière demande en échec jusqu’à atteindre un état de réussite ou d’erreur.

### <a name="version-216"></a>Version 2.16

- **Date de publication :** 21 avril 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Prise en charge de WMF :** WMF 5.0 RTM, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016 Technical Preview. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage).
- **Nouvelles fonctionnalités :**
  - Amélioration de la gestion des erreurs et autres correctifs de bogues mineurs.
  - Nouvelle propriété dans les paramètres de l’extension DSC. « ForcePullAndApply » a été ajouté dans AdvancedOptions pour permettre à l’extension DSC de traiter des configurations DSC quand le mode d’actualisation est par tirage (pull), au contraire du mode par envoi (push) par défaut. Pour plus d’informations, reportez-vous à [ce blog](https://devblogs.microsoft.com/powershell/arm-dsc-extension-settings/), qui donne des informations sur les paramètres de l’extension DSC.

### <a name="version-215"></a>Version 2.15

- **Date de publication :** 14 mars 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Prise en charge de WMF :** WMF 5.0 RTM, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016 Technical Preview. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage).
- **Nouvelles fonctionnalités :**
  - Dans l’extension version 2.14, des modifications ont été apportées pour l’installation de WMF RTM. Lors de la mise à niveau depuis l’extension version 2.13.2.0 vers 2.14.0.0, certaines applets de commande DSC peuvent échouer, ou votre configuration peut échouer avec une erreur : « Aucune instance trouvée avec les valeurs de propriété données ». Pour plus d’informations, consultez les [notes de publication de DSC](/powershell/scripting/wmf/known-issues/known-issues-dsc). Les solutions de contournement pour ces problèmes ont été ajoutées dans la version 2.15.
  - Malheureusement, si vous avez déjà installé la version 2.14 et que vous rencontrez un des deux problèmes mentionnés ci-dessus, vous devez effectuer ces étapes manuellement. Ouvrez une session PowerShell avec élévation de privilèges :
    - `Remove-Item -Path $env:SystemRoot\system32\Configuration\DSCEngineCache.mof`
    - `mofcomp $env:windir\system32\wbem\DscCoreConfProv.mof`

### <a name="version-214"></a>Version 2.14

- **Date de publication :** 25 février 2016
- **Prise en charge du système d’exploitation :** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1
- **Prise en charge de WMF :** WMF 5.0 RTM, WMF 4.0
- **Environment :** Azure
- **Remarques :** Cette version utilise DSC telle qu’elle est incluse dans Windows Server 2016 Technical Preview. Pour les autres systèmes d’exploitation Windows, elle installe [Windows Management Framework 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (l’installation de WMF nécessite un redémarrage).
- **Nouvelles fonctionnalités :**
  - Utilise WMF RTM.
  - Active la collecte de données afin d’améliorer la qualité de l’extension DSC. Pour plus d’informations, consultez [le blog](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/).
  - Fournit un format de paramètres mis à jour pour l’extension dans un modèle Resource Manager. Pour plus d’informations, consultez [le blog](https://devblogs.microsoft.com/powershell/arm-dsc-extension-settings/).
  - Correctifs de bogues et autres améliorations.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus informations sur DSC PowerShell, consultez le [Centre de documentation PowerShell](/powershell/scripting/dsc/overview/overview).
- Examinez le [modèle Resource Manager pour l’extension DSC](../virtual-machines/extensions/dsc-template.md).
- Pour plus de fonctionnalités et de ressources que vous pouvez gérer avec DSC PowerShell, parcourez le référentiel [PowerShell Gallery](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0).
- Pour plus d’informations sur le passage de paramètres sensibles dans des configurations, consultez [Gérer les informations d’identification de façon sécurisée avec le gestionnaire d’extensions DSC](../virtual-machines/extensions/dsc-credentials.md).
