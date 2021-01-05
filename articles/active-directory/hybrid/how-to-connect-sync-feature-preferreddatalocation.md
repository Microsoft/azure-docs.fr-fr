---
title: 'Azure AD Connect : Configurer un emplacement de données par défaut pour les ressources Microsoft 365'
description: Explique comment rapprocher vos ressources utilisateur Microsoft 365 de l’utilisateur avec la synchronisation Azure Active Directory Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/11/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e4c456e7788280b7ca5328342e1cd848ba3a583a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95972757"
---
# <a name="azure-active-directory-connect-sync-configure-preferred-data-location-for-microsoft-365-resources"></a>Synchronisation Azure Active Directory Connect : Configurer un emplacement de données par défaut pour les ressources Microsoft 365
L’objectif de cette rubrique est de vous expliquer comment configurer l’attribut d’emplacement des données préféré dans la synchronisation Azure Active Directory (Azure AD) Connect. Lorsqu’une personne utilise les capacités multigéographiques dans Microsoft 365, vous utilisez cet attribut pour désigner l’emplacement géographique des données Microsoft 365 de l’utilisateur. (Les termes *région* et *zone géographique* sont utilisés de manière interchangeable.)

## <a name="enable-synchronization-of-preferred-data-location"></a>Activer la synchronisation de l’emplacement des données préféré
Par défaut, les ressources Microsoft 365 des utilisateurs se trouvent dans la même zone géographique que le locataire Azure AD. Par exemple, si votre locataire est situé en Amérique du Nord, les boîtes aux lettres Exchange des utilisateurs sont également situées en Amérique du Nord. Pour une organisation multinationale, cela n’est peut-être pas optimal.

L’attribut **preferredDataLocation** vous permet de définir la zone géographique d’un utilisateur. Il est possible de mettre les ressources Microsoft 365 de l’utilisateur, par exemple la boîte aux lettres et OneDrive, dans la même zone géographique que l’utilisateur, tout en conservant un seul locataire pour l’ensemble de votre organisation.

> [!IMPORTANT]
> Les fonctionnalités multigéographiques sont actuellement disponibles pour les clients dotés d’un Contrat Entreprise actif avec au moins 250 abonnements aux services Microsoft 365. Pour plus d’informations, adressez-vous à votre représentant Microsoft.
>
>

Vous trouverez la liste de toutes les zones géographiques pour Microsoft 365 dans la section [Où se trouvent vos données ?](/microsoft-365/enterprise/o365-data-locations)

Voici les zones géographiques Microsoft 365 disponibles pour les fonctionnalités multigéographiques :

| Zone géographique | Valeur de PreferredDataLocation |
| --- | --- |
| Asie-Pacifique | APC |
| Australie | AUS |
| Canada | CAN |
| Union européenne | EUR |
| France | FRA |
| Inde | IND |
| Japon | JPN |
| Corée du Sud | KOR |
| Afrique du Sud | ZAF |
| Suisse | CHE |
| Émirats Arabes Unis | ARE |
| Royaume-Uni | GBR |
| États-Unis | NAM |

* Si une zone géographique, par exemple l’Amérique du Sud, n’apparaît pas dans ce tableau, c’est qu’elle n’est pas utilisable pour les fonctionnalités multigéographiques.

* Toutes les charges de travail Microsoft 365 ne prennent pas en charge l’utilisation du paramètre de zone géographique des utilisateurs.

### <a name="azure-ad-connect-support-for-synchronization"></a>Prise en charge d’Azure AD Connect pour la synchronisation

Azure AD Connect prend en charge la synchronisation de l’attribut **preferredDataLocation** pour les objets **Utilisateur** dans la version 1.1.524.0 et versions ultérieures. Plus précisément :

* Le schéma du type d’objet **Utilisateur** dans le connecteur Azure AD est étendu pour inclure l’attribut **preferredDataLocation**. L’attribut est de type chaîne à une seule valeur.
* Le schéma du type d’objet **Personne** dans le métaverse est étendu pour inclure l’attribut **preferredDataLocation**. L’attribut est de type chaîne à une seule valeur.

Par défaut, l’attribut **preferredDataLocation** n’est pas activé pour la synchronisation. Cette fonctionnalité est destinée aux grandes organisations. Le schéma Active Directory dans Windows Server 2019 a un attribut **msDS-preferredDataLocation** que vous devez utiliser à cet effet. Si vous n’avez pas mis à jour le schéma Active Directory et que vous ne pouvez pas le faire, vous devez identifier un attribut destiné à contenir la zone géographique Microsoft 365 pour vos utilisateurs. Cet attribut varie d’une organisation à l’autre.

> [!IMPORTANT]
> Azure AD permet que l’attribut **preferredDataLocation** sur des **objets utilisateur cloud** soit directement configuré à l’aide d’Azure AD PowerShell. Pour configurer cet attribut sur des **objets utilisateur synchronisés**, vous devez utiliser Azure AD Connect.

Avant d’activer la synchronisation :

* Si vous n’avez pas mis à niveau le schéma d’Active Directory vers 2019, choisissez l’attribut Active Directory local à utiliser comme attribut source. Il doit être de type **chaîne à valeur unique**.
* Si vous avez déjà configuré l’attribut **preferredDataLocation** sur des **objets utilisateur synchronisés** existants dans Azure AD à l’aide d’Azure AD PowerShell, vous devez rétroporter les valeurs d’attribut vers les objets **Utilisateur** correspondants dans l’Active Directory local.

    > [!IMPORTANT]
    > Si vous ne rétroportez pas ces valeurs, Azure AD Connect supprime les valeurs d’attribut existantes dans Azure AD lorsque la synchronisation de l’attribut **preferredDataLocation** est activée.

* Configurez maintenant l’attribut source sur au moins deux objets Utilisateur de l’Active Directory local. Vous pouvez l’utiliser pour une vérification ultérieure.

Les étapes suivantes fournissent les étapes d’activation de la synchronisation de l’attribut **preferredDataLocation**.

> [!NOTE]
> Les étapes sont décrites dans le cadre d’un déploiement d’Azure AD avec une topologie de forêt unique et sans règles de synchronisation personnalisées. Si vous avez une topologie à forêts multiples, des règles de synchronisation personnalisées configurées ou un serveur intermédiaire, vous devez ajuster les étapes en conséquence.

## <a name="step-1-disable-sync-scheduler-and-verify-there-is-no-synchronization-in-progress"></a>Étape 1 : Désactiver le planificateur de synchronisation et vérifier qu'aucune synchronisation n’est en cours
Pour éviter l’exportation de modifications indésirables vers Azure AD, veillez à ce qu’aucune synchronisation ne se produise pendant la mise à jour des règles de synchronisation. Pour désactiver le planificateur de synchronisation intégré :

1. Lancez une session PowerShell sur le serveur Azure AD Connect.
2. Désactivez la synchronisation planifiée en exécutant la cmdlet suivante : `Set-ADSyncScheduler -SyncCycleEnabled $false`.
3. Lancez **Synchronization Service Manager** en accédant au menu **DÉMARRER** > **Service de synchronisation**.
4. Sélectionnez l’onglet **Opérations** et confirmez qu’aucune opération n’est à l’état *En cours*.

![Capture d’écran de Synchronization Service Manager](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step1.png)

## <a name="step-2-refresh-the-schema-for-active-directory"></a>Étape 2 : Actualiser le schéma pour Active Directory
Si vous avez mis à jour le schéma Active Directory vers 2019 et que Connect a été installé avant l’extension de schéma, le schéma de connexion Connect ne dispose pas du schéma mis à jour. Vous devez alors actualiser le schéma à partir de l’Assistant pour qu’il s’affiche dans l’interface utilisateur.

1. Démarrez l’Assistant Azure AD Connect à partir du bureau.
2. Sélectionnez l’option **Actualiser le schéma de l’annuaire**, puis cliquez sur **Suivant**.
3. Entrez vos informations d’identification Azure AD, puis cliquez sur **Suivant**.
4. Dans la page **Actualiser le schéma de l’annuaire**, vérifiez que toutes les forêts sont sélectionnées, puis cliquez sur **Suivant**.
5. Quand vous avez terminé, fermez l’Assistant.

![Capture d’écran de l’option Actualiser le schéma de l’annuaire dans l’Assistant Connect](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-refreshschema.png)

## <a name="step-3-add-the-source-attribute-to-the-on-premises-active-directory-connector-schema"></a>Étape 3 : Ajouter l’attribut source au schéma du connecteur Active Directory local
**Cette étape est nécessaire seulement si vous exécutez Connect version 1.3.21 ou ultérieure. Si vous utilisez la version 1.4.18 ou une version plus récente, passez à l’étape 5.**  
Tous les attributs Azure AD ne sont pas importés dans l’espace de connecteur Active Directory local. Si vous avez choisi d’utiliser un attribut qui n’est pas synchronisé par défaut, vous devez l’importer. Pour ajouter l’attribut source à la liste des attributs importés :

1. Sélectionnez l’onglet **Connecteurs** dans Synchronization Service Manager.
2. Cliquez avec le bouton droit sur le connecteur Active Directory local, puis sélectionnez **Propriétés**.
3. Dans la boîte de dialogue contextuelle, accédez à l’onglet **Sélectionner les attributs**.
4. Assurez-vous que l’attribut source que vous avez sélectionné est activé dans la liste d’attributs. Si vous ne voyez pas votre attribut, sélectionnez la case à cocher **Afficher tout**.
5. Pour enregistrer, sélectionnez **OK**.

![Capture d’écran montrant Synchronization Service Manager et la boîte de dialogue Propriétés avec la liste « Attributs » en surbrillance.](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step2.png)

## <a name="step-4-add-preferreddatalocation-to-the-azure-ad-connector-schema"></a>Étape 4 : Ajouter **preferredDataLocation** au schéma du connecteur Azure AD
**Cette étape est nécessaire seulement si vous exécutez Connect version 1.3.21 ou ultérieure. Si vous utilisez la version 1.4.18 ou une version plus récente, passez à l’étape 5.**  
Par défaut, l’attribut **preferredDataLocation** n’est pas importé dans l’espace du connecteur Azure AD. Pour l’ajouter à la liste des attributs importés :

1. Sélectionnez l’onglet **Connecteurs** dans Synchronization Service Manager.
2. Cliquez avec le bouton droit sur le connecteur Azure AD, puis sélectionnez **Propriétés**.
3. Dans la boîte de dialogue contextuelle, accédez à l’onglet **Sélectionner les attributs**.
4. Sélectionnez l’attribut **preferredDataLocation** dans la liste.
5. Pour enregistrer, sélectionnez **OK**.

![Capture d’écran de Synchronization Service Manager et de la boîte de dialogue Propriétés](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step3.png)

## <a name="step-5-create-an-inbound-synchronization-rule"></a>Étape 5 : Créer une règle de synchronisation de trafic entrant
La règle de synchronisation du trafic entrant permet de transmettre la valeur de l’attribut au métaverse à partir de l’attribut source de l’Active Directory local.

1. Lancez **Synchronization Rules Editor** dans le menu **DÉMARRER** > **Éditeur de règles de synchronisation**.
2. Définissez le filtre de recherche **Direction** sur **Entrant**.
3. Pour créer une règle de trafic entrant, sélectionnez **Ajouter une nouvelle règle**.
4. Sous l’onglet **Description**, définissez la configuration suivante :

    | Attribut | Valeur | Détails |
    | --- | --- | --- |
    | Nom | *Donnez-lui un nom* | Par exemple, « Entrant depuis AD – Utilisateur preferredDataLocation » |
    | Description | *Fournissez une description personnalisée* |  |
    | Système connecté | *Sélectionnez le connecteur Active Directory local* |  |
    | Type d’objet système connecté | **Utilisateur** |  |
    | Type d’objet métaverse | **Person** |  |
    | Type de lien | **Join** |  |
    | Priorité | *Choisissez une valeur comprise entre 1 et 99* | Les valeurs comprises entre 1 et 99 sont réservées aux règles de synchronisation personnalisées. Ne sélectionnez pas de valeur utilisée par une autre règle de synchronisation. |

5. Conservez le **filtre d’étendue** vide pour inclure tous les objets. Vous devrez peut-être adapter le filtre d’étendue à votre déploiement Azure AD Connect.
6. Accédez à l’onglet **Transformation**, puis implémentez la règle de transformation suivante :

    | Type de flux | Attribut cible | Source | Appliquer une seule fois | Type de fusion |
    | --- | --- | --- | --- | --- |
    |Direct | preferredDataLocation | Sélectionnez l’attribut source | Désactivé | Update |

7. Pour créer la règle de trafic entrant, sélectionnez **Ajouter**.

![Capture d’écran de la Créer une règle de synchronisation de trafic entrant](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step4.png)

## <a name="step-6-create-an-outbound-synchronization-rule"></a>Étape 6 : Créer une règle de synchronisation de trafic sortant
La règle de synchronisation du trafic sortant permet de transmettre la valeur de l’attribut à l’attribut **preferredDataLocation** dans Azure AD à partir du métaverse :

1. Accédez à l’**Éditeur de règles de synchronisation**.
2. Définissez le filtre de recherche **Direction** sur **Sortante**.
3. Sélectionnez **Ajouter une nouvelle règle**.
4. Sous l’onglet **Description**, définissez la configuration suivante :

    | Attribut | Valeur | Détails |
    | ----- | ------ | --- |
    | Nom | *Donnez-lui un nom* | Par exemple, « Sortant vers Azure AD – Utilisateur preferredDataLocation » |
    | Description | *Fournissez une description* ||
    | Système connecté | *Sélectionnez le connecteur Azure AD* ||
    | Type d’objet système connecté | **Utilisateur** ||
    | Type d’objet métaverse | **Person** ||
    | Type de lien | **Join** ||
    | Priorité | *Choisissez une valeur comprise entre 1 et 99* | Les valeurs comprises entre 1 et 99 sont réservées aux règles de synchronisation personnalisées. Ne sélectionnez pas de valeur utilisée par une autre règle de synchronisation. |

5. Accédez à l’onglet **Filtre d’étendue**, puis ajoutez un groupe de filtres à étendue unique avec deux clauses :

    | Attribut | Opérateur | Valeur |
    | --- | --- | --- |
    | sourceObjectType | EQUAL | Utilisateur |
    | cloudMastered | NOTEQUAL | True |

    Le filtre d’étendue détermine les objets Active Directory auxquels cette règle de synchronisation de trafic sortant s’applique. Dans cet exemple, nous utilisons le filtre d’étendue de la règle de synchronisation prête à l’emploi « Sortant vers Azure AD – Identité de l’utilisateur ». Il empêche l’application de la règle de synchronisation aux objets **Utilisateur** non synchronisés à partir de l’Active Directory local. Vous devrez peut-être adapter le filtre d’étendue à votre déploiement Azure AD Connect.

6. Accédez à l’onglet **Transformation**, puis implémentez la règle de transformation suivante :

    | Type de flux | Attribut cible | Source | Appliquer une seule fois | Type de fusion |
    | --- | --- | --- | --- | --- |
    | Direct | preferredDataLocation | preferredDataLocation | Désactivé | Update |

7. Fermez **Ajouter** pour créer la règle de trafic sortant.

![Capture d’écran de la Créer une règle de synchronisation de trafic sortant](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-step5.png)

## <a name="step-7-run-full-synchronization-cycle"></a>Étape 7 : Exécuter un cycle de synchronisation complète
En règle générale, un cycle de synchronisation complet est nécessaire. Ceci est dû au fait que vous avez ajouté de nouveaux attributs aux schémas Active Directory et du connecteur Azure AD, et introduit des règles de synchronisation personnalisées. Vérifiez les modifications avant de les exporter vers Azure AD. Vous pouvez procéder comme suit pour vérifier les modifications tandis que vous exécutez manuellement les étapes d’un cycle de synchronisation complet.

1. Exécutez une **Importation intégrale** sur le connecteur Active Directory local :

   1. Accédez à l’onglet **Opérations** dans Synchronization Service Manager.
   2. Cliquez avec le bouton droit sur le **connecteur Active Directory local**, puis sélectionnez **Exécuter**.
   3. Dans la boîte de dialogue, sélectionnez **Importation intégrale**, puis **OK**.
   4. Attendez que l’opération se termine.

      > [!NOTE]
      > Vous pouvez ignorer l’importation intégrale sur le connecteur Azure Directory local si l’attribut source est déjà inclus dans la liste des attributs importés. En d’autres termes, vous n’avez apporté aucune modification à l’étape 2 plus haut dans cet article.

2. Exécutez une **Importation intégrale** sur le Connecteur Azure AD :

   1. Cliquez avec le bouton droit sur le **connecteur Azure AD**, puis sélectionnez **Exécuter**.
   2. Dans la boîte de dialogue, sélectionnez **Importation intégrale**, puis **OK**.
   3. Attendez que l’opération se termine.

3. Vérifiez le changement de la règle de synchronisation sur un objet **Utilisateur** existant.

   L’attribut source de l’Active Directory local et l’attribut **preferredDataLocation** d’Azure AD ont été importés dans chaque espace de connecteur respectif. Avant de passer à l’étape de synchronisation complète, affichez un aperçu d’un objet **Utilisateur** existant dans l’espace du connecteur Azure Directory local. L’attribut source de l’objet que vous avez choisi doit avoir une valeur définie. Un bon aperçu avec l’attribut **preferredDataLocation** renseigné dans le métaverse est un bon indicateur que vous avez correctement configuré les règles de synchronisation. Pour plus d’informations sur la façon de générer un aperçu, consultez la section [Vérifier la modification](how-to-connect-sync-change-the-configuration.md#verify-the-change).

4. Exécutez une **Synchronisation complète** sur le connecteur Active Directory local :

   1. Cliquez avec le bouton droit sur le **connecteur Active Directory local**, puis sélectionnez **Exécuter**.
   2. Dans la boîte de dialogue, sélectionnez **Synchronisation complète**, puis **OK**.
   3. Attendez que l’opération se termine.

5. Vérifiez les **Exportations en attente** vers Azure AD :

   1. Cliquez avec le bouton droit sur le **connecteur Azure AD** et sélectionnez **Rechercher dans l’espace connecteur**.
   2. Dans la boîte de dialogue **Rechercher dans l’espace connecteur** :

        a. Définissez l’**Étendue** sur **En attente d’exportation**.<br>
        b. Cochez les trois cases : **Ajouter, Modifier et Supprimer**.<br>
        c. Pour obtenir la liste d’objets contenant des modifications à exporter, sélectionnez **Rechercher**. Pour examiner les modifications apportées à un objet donné, double-cliquez sur celui-ci.<br>
        d. Vérifiez que les modifications sont correctes.

6. Exécutez une **Exportation** sur le **connecteur Azure AD**

   1. Cliquez avec le bouton droit sur le **connecteur Azure AD**, puis sélectionnez **Exécuter**.
   2. Dans la boîte de dialogue **Exécuter le connecteur**, sélectionnez **Exporter**, puis **OK**.
   3. Attendez que l’opération se termine.

> [!NOTE]
> Vous pouvez remarquer que les étapes n’incluent pas la synchronisation complète sur le connecteur Azure AD ou l’étape d’exportation sur le connecteur Active Directory. Les étapes ne sont pas obligatoires car les valeurs d’attributs sont transmises uniquement de l’Active Directory local à Azure AD.

## <a name="step-8-re-enable-sync-scheduler"></a>Étape 8 : Réactiver le planificateur de synchronisation
Réactivez le planificateur de synchronisation intégré :

1. Démarrez une session PowerShell.
2. Réactivez la synchronisation planifiée en exécutant la cmdlet suivante : `Set-ADSyncScheduler -SyncCycleEnabled $true`

## <a name="step-9-verify-the-result"></a>Étape 9 : Vérifier le résultat
Il est maintenant temps de vérifier la configuration et de l’activer pour vos utilisateurs.

1. Ajoutez la zone géographique à l’attribut sélectionné sur un utilisateur. Vous trouverez la liste des zones géographiques disponibles dans ce tableau.  
![Capture d'écran de l'attribut AD ajouté à un utilisateur](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-adattribute.png)
2. Attendez que l’attribut soit synchronisé avec Azure AD.
3. À l’aide d’Exchange Online PowerShell, vérifiez que la région de boîte aux lettres a été correctement définie.  
![Capture d’écran d’Exchange Online PowerShell](./media/how-to-connect-sync-feature-preferreddatalocation/preferreddatalocation-mailboxregion.png)  
Si votre locataire a été marqué comme étant en mesure d’utiliser cette fonctionnalité, la boîte aux lettres est déplacée dans la bonne zone géographique. Vous pouvez le vérifier en examinant le nom du serveur où se trouve la boîte aux lettres.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les fonctionnalités multigéographiques dans Microsoft 365 :

* [Sessions à plusieurs zones géographiques dans Ignite](https://aka.ms/MultiGeoIgnite)
* [Plusieurs zones géographiques dans OneDrive](/microsoft-365/enterprise/multi-geo-capabilities-in-onedrive-and-sharepoint-online-in-microsoft-365)
* [Plusieurs zones géographiques dans SharePoint Online](/microsoft-365/enterprise/multi-geo-capabilities-in-onedrive-and-sharepoint-online-in-microsoft-365)

En savoir plus sur le modèle de configuration dans le moteur de synchronisation :

* En savoir plus sur le modèle de configuration dans [Comprendre l’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning.md).
* En savoir plus sur le langage d’expression dans [Comprendre les expressions d’approvisionnement déclaratif](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md).

Rubriques de présentation :

* [Synchronisation Azure AD Connect : Comprendre et personnaliser la synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)