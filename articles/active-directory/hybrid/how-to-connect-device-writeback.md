---
title: 'Azure AD Connect : Activation de la réécriture d’appareil | Microsoft Docs'
description: Ce document explique comment activer l’écriture différée des appareils à l’aide d’Azure AD Connect
services: active-directory
documentationcenter: ''
author: billmath
manager: femila
editor: curtand
ms.assetid: c0ff679c-7ed5-4d6e-ac6c-b2b6392e7892
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/08/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: d526394ac89e2d29b2002004736e8480bb15b954
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95973420"
---
# <a name="azure-ad-connect-enabling-device-writeback"></a>Azure AD Connect : Activation de la réécriture d’appareil
> [!NOTE]
> L’écriture différée sur appareil nécessite un abonnement Azure AD Premium.
> 
> 

La documentation suivante fournit des informations sur l’activation de la fonctionnalité d’écriture différée des appareils dans Azure AD Connect. L’écriture différée des appareils est utilisée dans les scénarios suivants :

* Activer [Windows Hello Entreprise à l’aide d’un déploiement hybride de certificats de confiance](/windows/security/identity-protection/hello-for-business/hello-hybrid-cert-trust-prereqs#device-registration)
* Activer l’accès conditionnel basé sur les appareils pour les applications protégées ADFS (2012 R2 ou version ultérieure) (approbations de la partie de confiance)

Cela fournit une sécurité supplémentaire et l’assurance que l’accès aux applications est accordé uniquement aux appareils de confiance. Pour plus d’informations sur l’accès conditionnel, consultez [Gestion des risques avec accès conditionnel](../conditional-access/overview.md) et [Configuration d’un accès conditionnel en local à l’aide d’Azure Active Directory Device Registration](../devices/overview.md).

> [!IMPORTANT]
> <li>Les appareils doivent se trouver dans la même forêt que les utilisateurs. Étant donné que les appareils doivent être réécrits dans une seule forêt, cette fonctionnalité ne prend pas en charge un déploiement à plusieurs forêts d’utilisateurs pour l’instant.</li>
> <li>Vous ne pouvez ajouter qu’un seul objet de configuration d’enregistrement d’appareil à la forêt Active Directory locale. Cette fonctionnalité n’est pas compatible avec une topologie dans laquelle le domaine Active Directory local est synchronisé à plusieurs annuaires Azure AD.</li>

## <a name="part-1-install-azure-ad-connect"></a>Première partie : Installer Azure AD Connect
Installez Azure AD Connect à l’aide de paramètres personnalisés ou Express. Microsoft recommande de commencer par synchroniser correctement tous les utilisateurs et groupes avant d’activer l’écriture différée des appareils.

## <a name="part-2-enable-device-writeback-in-azure-ad-connect"></a>Deuxième partie : Activer la réécriture d’appareil dans Azure AD Connect
1. Réexécutez l’Assistant d’installation. Sur la page Tâches supplémentaires, sélectionnez **Configurer les options de l’appareil**, puis cliquez sur **Suivant**. 

    ![Tâche Configurer les options de l’appareil](./media/how-to-connect-device-writeback/deviceoptions.png)

    >[!NOTE]
    > La nouvelle tâche Configurer les options de l’appareil est uniquement disponible dans la version 1.1.819.0 et dans les versions ultérieures.

2. Sur la page Options de l’appareil, sélectionnez **Configurer la réécriture d’appareil**. L’option **Désactiver la réécriture d’appareil** n’est accessible que si la réécriture d’appareil a été activée. Cliquez sur **Suivant** pour passer à la page suivante de l’Assistant.
    ![Choix de l’opération d’appareil](./media/how-to-connect-device-writeback/configuredevicewriteback1.png)

3. Dans la page de l’écriture différée, vous verrez le domaine fourni en tant que forêt d’écriture différée d’appareil par défaut.
   ![Installation personnalisée - Forêt cible de l’écriture différée des appareils](./media/how-to-connect-device-writeback/writebackforest.png)

4. La page **Conteneur d’appareil** offre la possibilité de préparer Active Directory à l’aide de l’une des deux options disponibles :

    a. **Fournir des informations d’identification d’administrateur d’entreprise** : Si les informations d’identification d’administrateur d’entreprise sont fournies pour la forêt dans laquelle les appareils doivent être réécrits, Azure AD Connect prépare automatiquement la forêt lors de la configuration de la réécriture d’appareil.

    b. **Télécharger un script PowerShell** : Azure AD Connect génère automatiquement un script PowerShell qui peut préparer Active Directory à la réécriture d’appareil. Si les informations d’identification de l’administrateur d’entreprise ne peuvent pas être fournies dans Azure AD Connect, il est recommandé de télécharger le script PowerShell. Fournissez le script PowerShell téléchargé **CreateDeviceContainer.ps1** à l’administrateur d’entreprise de la forêt dans laquelle les appareils seront réécrits.
    ![Préparer la forêt Active Directory](./media/how-to-connect-device-writeback/devicecontainercreds.png)
    
    Les opérations effectuées dans le cadre de la préparation de la forêt Active Directory sont les suivantes :
    * Si elles n’existent pas, des conteneurs et des objets sont créés et configurés sous CN=Device Registration Configuration,CN=Services,CN=Configuration,[forest-dn].
    * Si elles existent, des conteneurs et des objets sont créés et configurés sous CN=RegisteredDevices,[domain-dn]. Les objets d’appareil seront créés dans ce conteneur.
    * Définit les autorisations nécessaires sur le compte Azure AD Connector pour gérer des appareils sur votre Active Directory.
    * Ne doit s’exécuter que sur une seule forêt, même si Azure AD Connect est installé sur plusieurs forêts.

## <a name="verify-devices-are-synchronized-to-active-directory"></a>Vérifier la synchronisation des appareils avec Active Directory
L’écriture différée des appareils doit désormais fonctionner correctement. Sachez que la réécriture des objets d’appareil dans AD peut prendre jusqu’à 3 heures.  Pour vérifier que vos appareils sont correctement synchronisés, procédez comme suit après la fin des règles de synchronisation :

1. Lancez le Centre d’administration Active Directory.
2. Développez RegisteredDevices au sein du domaine en cours de fédération.

   ![Active Directory - Appareils inscrits au Centre d’administration](./media/how-to-connect-device-writeback/devicewriteback5.png)

3. Les appareils enregistrés actuels sont répertoriés à cet emplacement.

   ![Active Directory - Liste des appareils inscrits au Centre d’administration](./media/how-to-connect-device-writeback/devicewriteback6.png)

## <a name="enable-conditional-access"></a>Activer l’accès conditionnel
Des instructions détaillées pour activer ce scénario sont disponibles dans [Configuration d’un accès conditionnel en local à l’aide du service d’inscription d’appareils Azure Active Directory](../devices/overview.md)

## <a name="troubleshooting"></a>Dépannage
### <a name="the-writeback-checkbox-is-still-disabled"></a>La case à cocher de l'écriture différée est toujours désactivée.
Si la case à cocher pour l'écriture différée des appareils n'est pas activée alors que vous avez suivi les étapes ci-dessus, la procédure suivante vous guidera dans ce que l'Assistant d'installation vérifie avant l'activation de la case.

Commençons par le début :

* Le schéma de forêt de la forêt où figurent les appareils doit être mis à jour au niveau Windows 2012 R2 pour que l’objet d’appareil et les attributs associés soient présents.
* Si l'Assistant d'installation est déjà en cours d'exécution, les modifications ne seront pas détectées. Dans ce cas, terminez l'Assistant installation et exécutez-le à nouveau.
* Assurez-vous que le compte que vous fournissez dans le script d'initialisation est bien l'utilisateur utilisé par le connecteur Active Directory. Pour ce faire, procédez comme suit :
  * Dans le menu Démarrer, ouvrez **Service de synchronisation**.
  * Ouvrez l’onglet **Connecteurs** .
  * Trouvez le connecteur de type services de domaine Active Directory et sélectionnez-le.
  * Sous **Actions**, sélectionnez **Propriétés**.
  * Accédez à **Se connecter à la forêt Active Directory**. Vérifiez que le nom de domaine et le nom d’utilisateur spécifiés sur cet écran correspondent au compte fourni pour le script.
    ![Compte de connecteur dans Sync Service Manager](./media/how-to-connect-device-writeback/connectoraccount.png)

Vérifiez la configuration dans Active Directory :

* Vérifiez que le Service d’inscription de l’appareil se trouve à l’emplacement ci-dessous (CN = DeviceRegistrationService, CN = Services d’inscription de périphérique, CN = Inscription de l’appareil, CN = Services, CN = Configuration) dans le contexte d’appellation de configuration.

![Résoudre les problèmes, DeviceRegistrationService dans l’espace de noms de configuration](./media/how-to-connect-device-writeback/troubleshoot1.png)

* Vérifiez qu'il n'y a qu'un seul objet de configuration en recherchant l'espace de noms de configuration. S'il en existe plusieurs, supprimez le doublon.

![Résoudre les problèmes, rechercher les objets en double](./media/how-to-connect-device-writeback/troubleshoot2.png)

* Sur l'objet Service d'inscription de l'appareil, assurez-vous que l'attribut msDS-DeviceLocation est présent et a une valeur. Recherchez cet emplacement et assurez-vous qu'il est présent avec l'attribut objectType msDS-DeviceContainer.

![Résoudre les problèmes, msDS-DeviceLocation](./media/how-to-connect-device-writeback/troubleshoot3.png)

![Résoudre les problèmes, classe d’objet RegisteredDevices](./media/how-to-connect-device-writeback/troubleshoot4.png)

* Vérifiez que le compte utilisé par le connecteur Active Directory dispose des autorisations requise sur le conteneur d’appareils inscrits trouvé à l’étape précédente. Voici les autorisations attendues sur ce conteneur :

![Résoudre les problèmes, vérifier les autorisations du conteneur](./media/how-to-connect-device-writeback/troubleshoot5.png)

* Vérifiez que le compte Active Directory dispose des autorisations sur CN = Inscription de l’appareil, CN = Services, CN = Objet de Configuration.

![Résoudre les problèmes, vérifier les autorisations de la configuration de l’inscription des appareils](./media/how-to-connect-device-writeback/troubleshoot6.png)

## <a name="additional-information"></a>Informations supplémentaires
* [Gestion des risques avec accès conditionnel](../conditional-access/overview.md)
* [Configuration d’un accès conditionnel en local à l’aide du service d’inscription d’appareils Azure Active Directory](../devices/overview.md)

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).