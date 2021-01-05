---
title: 'Tutoriel : Configurer la réécriture Workday dans Azure Active Directory | Microsoft Docs'
description: Découvrir comment configurer la réécriture des attributs d’Azure AD à Workday
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 10/14/2020
ms.author: chmutali
ms.openlocfilehash: c65fddcc90b25f70759fb038a72dad0facfa99a9
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359729"
---
# <a name="tutorial-configure-attribute-writeback-from-azure-ad-to-workday"></a>Tutoriel : Configurer la réécriture des attributs d’Azure AD à Workday
L’objectif de ce tutoriel est de vous montrer les étapes à suivre pour réécrire des attributs d’Azure AD à Workday. L’application de provisionnement Workday Writeback prend en charge l’affectation de valeurs sur les attributs Workday suivants :
* Adresse e-mail professionnelle 
* Nom d’utilisateur Workday
* Numéro de téléphone fixe professionnel (y compris l’indicatif du pays, l’indicatif régional, le numéro et l’extension)
* Indicateur principal du numéro de téléphone fixe professionnel
* Numéro de téléphone mobile professionnel (y compris l’indicatif du pays, l’indicatif régional, le numéro)
* Indicateur principal du téléphone mobile

## <a name="overview"></a>Vue d’ensemble

Après avoir configuré l’intégration du provisionnement entrant à l’aide de l’application de provisionnement [Workday vers AD local](workday-inbound-tutorial.md) ou [Workday vers Azure AD](workday-inbound-cloud-only-tutorial.md), vous pouvez le cas échéant configurer l’application Workday Writeback pour réécrire des informations de contact professionnelles, telles que l’adresse e-mail et le numéro de téléphone, sur Workday. 

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>À qui cette solution d’attribution d’utilisateurs convient-elle le mieux ?

Cette solution de provisionnement d’utilisateurs Workday Writeback est idéale pour :

* Les organisations qui utilisent Microsoft 365 et qui souhaitent réécrire sur Workday des attributs faisant autorité, gérés par le service informatique (par exemple, l’adresse e-mail, le nom d’utilisateur et le numéro de téléphone)

## <a name="configure-integration-system-user-in-workday"></a>Configurer un utilisateur du système d’intégration dans Workday

Reportez-vous à la section [Configurer un utilisateur du système d’intégration](workday-inbound-tutorial.md#configure-integration-system-user-in-workday) pour la création d’un compte d’utilisateur du système d’intégration Workday, doté des autorisations permettant de récupérer des données d’employés. 

## <a name="configuring-azure-ad-attribute-writeback-to-workday"></a>Configuration de la reprise de l’attribut Azure AD sur Workday

Suivez ces instructions pour configurer l’écriture différée des adresses e-mail et des noms d’utilisateurs d’Azure Active Directory vers Workday.

* [Ajouter l'application de connecteur de réécriture et établir la connexion avec Workday](#part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday)
* [Configurer les mappages d'attributs de réécriture](#part-2-configure-writeback-attribute-mappings)
* [Activer et lancer l'approvisionnement des utilisateurs](#enable-and-launch-user-provisioning)

### <a name="part-1-adding-the-writeback-connector-app-and-creating-the-connection-to-workday"></a>Première partie : Ajouter l'application de connecteur de réécriture et établir la connexion avec Workday

**Pour configurer le connecteur de réécriture Workday :**

1. Atteindre <https://portal.azure.com>.

2. Dans le Portail Azure, recherchez et sélectionnez **Azure Active Directory**.

3. Cliquez sur **Applications d’entreprise** , puis sur **Toutes les applications**.

4. Sélectionnez **Ajouter une application** , puis sélectionnez la catégorie **Tous**.

5. Recherchez **Écriture différée Workday** et ajoutez cette application à partir de la galerie.

6. Une fois l’application ajoutée et l’écran de détails de l’application affiché, sélectionnez **Provisionnement**.

7. Définissez le **mode** de **provisionnement** sur **Automatique**

8. Fermez la section **Informations d’identification de l’administrateur** , comme suit :

   * **Nom d’utilisateur de l’administrateur** : entrez le nom d’utilisateur du compte de système d’intégration Workday, avec le nom du domaine client ajouté. Le résultat doit ressembler à : *nom d’utilisateur\@contoso4*

   * **Mot de passe administrateur :** entrez le mot de passe du compte du système d'intégration Workday.

   * **URL du client :**  entrez l’URL du point de terminaison des services web Workday pour votre client. Cette URL doit être semblable à : `https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources`, où *contoso4* est remplacé par le nom de votre locataire et *wd3-impl* par la chaîne d'environnement qui convient (si nécessaire).

   * **E-mail de notification :** entrez votre adresse e-mail et activez la case à cocher « Envoyer un e-mail en cas de défaillance ».

   * Cliquez sur le bouton **Tester la connexion**. Si le test de connexion réussit, cliquez sur le bouton **Enregistrer** en haut. En cas d’échec, vérifiez que l’URL et les informations d’identification Workday sont valides dans Workday.

### <a name="part-2-configure-writeback-attribute-mappings"></a>Deuxième partie : Configurer les mappages d'attributs de réécriture

Dans cette section, vous allez configurer le transfert des attributs de réécriture d'Azure AD vers Workday. 

1. Sous l’onglet Provisionnement, sous **Mappages** , cliquez sur le nom du mappage.

2. Dans le champ **Portée de l'objet source** , vous avez la possibilité de filtrer les ensembles d’utilisateurs d’Azure Active Directory qui doivent faire partie de la réécriture. La portée par défaut est « tous les utilisateurs dans Azure AD ».

3. Dans la section **Mappages d’attributs** , mettez à jour l’ID correspondant pour indiquer l’attribut dans Azure Active Directory dans lequel est stocké l’ID d’employé Workday. Une méthode populaire de correspondance consiste à synchroniser l’ID de l’employé Workday avec extensionAttribute1-15 dans Azure AD puis, à utiliser cet attribut dans Azure AD pour faire à nouveau correspondre les utilisateurs dans Workday.

4. Généralement, vous mappez l’attribut Azure AD *userPrincipalName* vers l’attribut *UserID* de Workday et mappez l’attribut *mail* d’Azure AD vers l’attribut *EmailAddress* de Workday. 

     >[!div class="mx-imgBorder"]
     >![Azure portal](./media/workday-inbound-tutorial/workday-writeback-mapping.png)

5. Utilisez les indications partagées ci-dessous pour mapper les valeurs d’attribut de numéro de téléphone d’Azure AD à Workday. 

     | Attribut de téléphone Workday | Valeur attendue | Indication de mappage |
     |-------------------------|----------------|------------------|
     | WorkphoneLandlineIsPrimary | true/false | Mappage de constante ou d’expression dont la sortie est une valeur de chaîne « true » ou « false ». |
     | WorkphoneLandlineCountryCodeName | [Code du pays ISO 3166-1 à trois lettres](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) | Mappage de constante ou d’expression dont la sortie est un code de pays à trois lettres. |
     | WorkphoneLandlineCountryCodeNumber | [Indicatif téléphonique international](https://en.wikipedia.org/wiki/List_of_country_calling_codes) | Mappage de constante ou d’expression dont la sortie est un indicatif de pays valide (sans le signe +). |
     | WorkphoneLandlineNumber | Numéro de téléphone complet incluant l’indicatif régional | Mapper à l’attribut *telephoneNumber*. Utiliser regex pour supprimer les espaces, crochets et indicatif de pays. Voir l’exemple ci-dessous. |
     | WorkphoneLandlineExtension | Numéro d’extension | Si *telephoneNumber* contient une extension, utilisez regex pour extraire la valeur. |
     | WorkphoneMobileIsPrimary | true/false | Mappage de constante ou d’expression dont la sortie est une valeur de chaîne « true » ou « false ». |
     | WorkphoneMobileCountryCodeName | [Code du pays ISO 3166-1 à trois lettres](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) | Mappage de constante ou d’expression dont la sortie est un code de pays à trois lettres. |
     | WorkphoneMobileCountryCodeNumber | [Indicatif téléphonique international](https://en.wikipedia.org/wiki/List_of_country_calling_codes) | Mappage de constante ou d’expression dont la sortie est un indicatif de pays valide (sans le signe +). |
     | WorkphoneMobileNumber | Numéro de téléphone complet incluant l’indicatif régional | Mapper à l’attribut *mobile*. Utiliser regex pour supprimer les espaces, crochets et indicatif de pays. Voir l’exemple ci-dessous. |

     > [!NOTE]
     > Lors de l’appel du service web Workday Change_Work_Contact, Azure AD envoie les valeurs constantes suivantes : <br>
     > * La valeur de **Communication_Usage_Type_ID** est définie sur la chaîne constante « WORK ». <br>
     > * La valeur de **Phone_Device_Type_ID** est définie sur la chaîne constante « Mobile » pour les numéros de téléphone mobile, et sur « Landline » pour les numéros de téléphone fixe. <br>
     > 
     > Des échecs de réécriture se produiront si votre locataire Workday utilise différents Type_ID. Pour éviter de telles situations, vous pouvez utiliser la tâche Workday **Maintain Reference IDs** (Gérer les ID de référence) et mettre à jour les différents Type_ID pour qu’ils correspondent aux valeurs utilisées par Azure AD. <br>
     >  

     **Expressions regex de référence – Exemple 1**

     Utilisez l’expression régulière ci-dessous, si le numéro de téléphone dans Azure AD est défini à l’aide du format exigé pour la réinitialisation de mot de passe en libre-service (SSPR). <br>
     Exemple : si la valeur du numéro de téléphone est +1 1112223333 ->, l’expression regex génère 1112223333

     ```C#
     Replace([telephoneNumber], , "\\+(?<isdCode>\\d* )(?<phoneNumber>\\d{10})", , "${phoneNumber}", , )
     ```

     **Expressions regex de référence – Exemple 2**

     Utilisez l’expression régulière ci-dessous, si le numéro de téléphone dans Azure AD est défini à l’aide du format (XXX) XXX-XXXX. <br>
     Exemple : si la valeur du numéro de téléphone est (111) 222-3333 ->, l’expression regex génère 1112223333

     ```C#
     Replace([mobile], , "[()\\s-]+", , "", , )
     ```

6. Pour enregistrer vos mappages, cliquez sur **Enregistrer** en haut de la section Mappage d’attributs.

## <a name="enable-and-launch-user-provisioning"></a>Activer et lancer l'approvisionnement des utilisateurs

Une fois les configurations d'application d'approvisionnement Workday effectuées, vous pouvez activer le service d'approvisionnement sur le portail Azure.

> [!TIP]
> Par défaut, lorsque vous activez le service d'approvisionnement, il lance les opérations d'approvisionnement pour tous les utilisateurs concernés. En cas d'erreur de mappage ou de problème lié aux données Workday, le travail d'approvisionnement peut échouer et être mis en quarantaine. Pour éviter ce genre de problème, nous vous recommandons de configurer le filtre **Portée de l'objet source** et de tester vos mappages d'attributs sur quelques utilisateurs test avant de lancer la synchronisation complète de tous les utilisateurs. Après avoir vérifié que les mappages fonctionnent et qu'ils vous donnent les résultats souhaités, vous pouvez supprimer le filtre ou l'étendre progressivement pour inclure d'autres utilisateurs.

1. Dans l’onglet **Approvisionnement** , définissez **État d’approvisionnement** sur **Activé**.

1. Dans la liste déroulante **Étendue** , sélectionnez **Synchroniser tous les utilisateurs et groupes**. Grâce à cette option, l’application Writeback écrira en différé les attributs mappés de tous les utilisateurs d’Azure AD dans Workday, selon les règles d’étendue définies sous **Mappages** -> **Portée de l’objet source**. 

   > [!div class="mx-imgBorder"]
   > ![Select Writeback scope](./media/sap-successfactors-inbound-provisioning/select-writeback-scope.png)

   > [!NOTE]
   > L’application d’approvisionnement Workday Writeback ne prend pas en charge l’option **Synchroniser uniquement les utilisateurs et les groupes attribués**.
 

2. Cliquez sur **Enregistrer**.

3. Cette opération permet de lancer la synchronisation initiale dont la durée dépendra du nombre d’utilisateurs contenus dans le répertoire source. Vous pouvez consulter la barre de progression pour suivre la progression du cycle de synchronisation. 

4. À tout moment, consultez l’onglet **Journaux d’approvisionnement** dans le portail Azure pour connaître les actions effectuées par le service d’approvisionnement. Les journaux d’audit listent tous les événements de synchronisation individuels effectués par le service de provisionnement, par exemple quels utilisateurs sont importés à partir de la source et lesquels sont exportés vers l’application cible.  

5. Au terme de la synchronisation initiale, un rapport de synthèse est créé dans l’onglet **Provisionnement** , comme illustré ci-dessous.

     > [!div class="mx-imgBorder"]
     > ![Barre de progression de provisionnement](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="known-issues-and-limitations"></a>Limitations et problèmes connus

* L’application Writeback utilise une valeur prédéfinie pour les paramètres **Communication_Usage_Type_ID** et **Phone_Device_Type_ID**. Si votre locataire Workday utilise une valeur différente pour ces attributs, l’opération d’écriture différée échouera. Comme solution de contournement, nous vous suggérons de mettre à jour les paramètres Type_ID dans Workday. 
* Lorsque l’application Writeback est configurée pour mettre à jour les numéros de téléphone secondaires, elle ne remplace pas le numéro de téléphone secondaire existant dans Workday. Elle ajoute un autre numéro de téléphone secondaire à l’enregistrement du Worker. Il n’existe aucune solution de contournement à ce comportement. 


## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
* [Découvrir comment configurer l’authentification unique entre Workday et Azure Active Directory](workday-tutorial.md)
* [Découvrir comment intégrer d’autres applications SaaS à Azure Active Directory](tutorial-list.md)
* [Découvrez comment exporter et importer vos configurations de provisionnement](../app-provisioning/export-import-provisioning-configuration.md)

