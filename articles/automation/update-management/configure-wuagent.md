---
title: Configurer les paramètres de Windows Update pour Azure Automation Update Management
description: Cet article explique comment configurer les paramètres de Windows Update pour qu’ils fonctionnent avec Azure Automation Update Management.
services: automation
ms.subservice: update-management
ms.date: 05/04/2020
ms.topic: conceptual
ms.openlocfilehash: a1f95ca856223628974a9519b7c4811bde43965e
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221540"
---
# <a name="configure-windows-update-settings-for-azure-automation-update-management"></a>Configurer les paramètres de Windows Update pour Azure Automation Update Management

Azure Automation Update Management s’appuie sur le [client Windows Update](/windows/deployment/update/windows-update-overview) pour télécharger et installer les mises à jour de Windows. Des paramètres spécifiques sont utilisés par le client Windows Update lors de la connexion à Windows Server Update Services (WSUS) ou à Windows Update. Vous pouvez gérer une grande partie de ces paramètres par les moyens suivants :

- Éditeur de stratégie de groupe locale
- Stratégie de groupe
- PowerShell
- Modification directe du Registre

Update Management respecte un grand nombre des paramètres spécifiés pour contrôler le client Windows Update. Si vous utilisez des paramètres pour activer les mises à jour non-Windows, Update Management gérera également ces mises à jour. Si vous souhaitez activer le téléchargement des mises à jour avant le déploiement de mises à jour, le déploiement des mises à jour peut être plus rapide et moins susceptible de dépasser la fenêtre de maintenance.

Pour obtenir des recommandations supplémentaires sur la configuration de WSUS dans votre abonnement Azure, et conserver vos machines virtuelles Windows à jour de manière sécurisée, consultez [Planifier votre déploiement pour la mise à jour des machines virtuelles Windows dans Azure à l’aide de WSUS](/azure/architecture/example-scenario/wsus/).

## <a name="pre-download-updates"></a>Pré-télécharger des mises à jour

Pour configurer le téléchargement automatique des mises à jour sans leur installation automatiquement, vous pouvez utiliser une stratégie de groupe pour définir le [paramètre Configurer les mises à jour automatiques](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates##configure-automatic-updates) sur 3. Ce paramètre permet de télécharger les mises à jour nécessaires en arrière-plan, puis de vous avertir que les mises à jour sont prêtes à être installées. Cela permet à Update Management de garder le contrôle des planifications, mais autorise le téléchargement des mises à jour en dehors de la fenêtre de maintenance d’Update Management. Ce comportement empêche les erreurs de type `Maintenance window exceeded` dans Update Management.

Vous pouvez activer ce paramètre dans PowerShell :

```powershell
$WUSettings = (New-Object -com "Microsoft.Update.AutoUpdate").Settings
$WUSettings.NotificationLevel = 3
$WUSettings.Save()
```

## <a name="configure-reboot-settings"></a>Configurer les paramètres de redémarrage

Les clés de Registre répertoriées sous [Configuration des mises à jour automatiques en modifiant le registre](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry) et [Clés de Registre utilisées pour gérer le redémarrage](/windows/deployment/update/waas-restart#registry-keys-used-to-manage-restart) peuvent entraîner le redémarrage de vos machines, même si vous spécifiez **Ne jamais redémarrer** dans les paramètres **Déploiement des mises à jour**. Configurez ces clés de Registre pour qu’elles s’adaptent au mieux à votre environnement.

## <a name="enable-updates-for-other-microsoft-products"></a>Activer les mises à jour pour d’autres produits Microsoft

Par défaut, le client Windows Update est configuré pour fournir des mises à jour uniquement pour Windows. Si vous activez le paramètre **Me communiquer les mises à jour d’autres produits Microsoft lorsque je mets à jour Windows**, les mises à jour des autres produits vous sont également fournies, y compris les correctifs de sécurité pour Microsoft SQL Server ou d’autres logiciels Microsoft. Vous pouvez configurer cette option si vous avez téléchargé et copié les derniers [fichiers de modèles d’administration](https://support.microsoft.com/help/3087759/how-to-create-and-manage-the-central-store-for-group-policy-administra) disponibles pour Windows 2016 et versions ultérieures.

Si vous avez des machines exécutant Windows Server 2012 R2, vous ne pouvez pas configurer ce paramètre par le biais d’une stratégie de groupe. Exécutez la commande PowerShell suivante sur ces machines :

```powershell
$ServiceManager = (New-Object -com "Microsoft.Update.ServiceManager")
$ServiceManager.Services
$ServiceID = "7971f918-a847-4430-9279-4a52d1efe18d"
$ServiceManager.AddService2($ServiceId,7,"")
```

## <a name="make-wsus-configuration-settings"></a>Définir des paramètres de configuration WSUS

Update Management prend en charge les paramètres WSUS. Vous pouvez spécifier les sources à utiliser pour l’analyse et le téléchargement des mises à jour en suivant les instructions de la rubrique [Spécifier l’emplacement intranet du service de mise à jour Microsoft](/windows/deployment/update/waas-wu-settings#specify-intranet-microsoft-update-service-location). Par défaut, le client Windows Update est configuré pour télécharger les mises à jour à partir de Windows Update. Quand vous spécifiez un serveur WSUS comme source pour vos machines, si les mises à jour ne sont pas approuvées dans WSUS, leur déploiement échoue. 

Pour limiter les machines au service de mise à jour interne, activez l’option [Ne pas se connecter à des emplacements Internet Windows Update](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates#do-not-connect-to-any-windows-update-internet-locations).

## <a name="next-steps"></a>Étapes suivantes

Planifiez un déploiement des mises à jour en suivant les instructions dans [Gérer les mises à jour et les correctifs pour vos machines virtuelles](manage-updates-for-vm.md).
