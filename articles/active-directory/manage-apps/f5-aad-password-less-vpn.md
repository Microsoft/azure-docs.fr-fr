---
title: Accès hybride sécurisé à Azure AD avec F5 VPN | Microsoft Docs
description: Tutoriel pour l’intégration de l’authentification unique Azure Active Directory à F5 BIG-IP pour VPN sans mot de passe
services: active-directory
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 10/12/2020
ms.author: gasinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2961f3f01f6ea4398fab6144b34fcb4409cdd96f
ms.sourcegitcommit: e5f9126c1b04ffe55a2e0eb04b043e2c9e895e48
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96317593"
---
# <a name="tutorial-for-azure-active-directory-single-sign-on-integration-with-f5-big-ip-for-password-less-vpn"></a>Tutoriel pour l’intégration de l’authentification unique Azure Active Directory à F5 BIG-IP pour VPN sans mot de passe

Dans ce tutoriel, vous allez apprendre à intégrer la solution SSL-VPN basée sur F5 BIG-IP avec Azure Active Directory (AD) pour un accès hybride sécurisé (SHA).

L’intégration d’un service SSL-VPN BIG-IP avec Azure AD offre [de nombreux avantages clés](f5-aad-integration.md), notamment :

- Gouvernance à confiance zéro améliorée via [la pré-authentification et l’autorisation Azure AD](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization)

- [Authentification sans mot de passe pour le service VPN](https://www.microsoft.com/security/business/identity/passwordless)

- Gestion des identités et de l’accès à partir d’un plan de contrôle unique – le [portail Azure](https://portal.azure.com/#home)

Malgré ces remarquables valeurs ajoutées, le réseau VPN classique reste tributaire de la notion d’un périmètre de réseau, qui délimite la confiance à l’intérieur et l’absence de confiance à l’extérieur. Ce modèle ne permet plus d’atteindre une véritable posture à confiance zéro, car les ressources d’entreprise ne sont plus limitées aux murs d’un centre de données d’entreprise, mais couvrent plutôt des environnements multiclouds sans limites fixes. Pour cette raison, nous encourageons nos clients à passer à une approche plus basée sur les identités pour la gestion de l’[accès application par application](https://docs.microsoft.com/azure/active-directory/fundamentals/five-steps-to-full-application-integration-with-azure-ad).

## <a name="scenario-description"></a>Description du scénario

Dans ce scénario, l’instance BIG-IP APM du service SSL-VPN est configurée en tant que fournisseur de services SAML et Azure AD devient le fournisseur d’identité SAML approuvé, fournissant une pré-authentification. L’authentification unique (SSO) à partir d’Azure AD est ensuite fournie via une authentification par revendications auprès de BIG-IP APM, offrant ainsi une expérience d’accès VPN transparente.

![Image montrant l’architecture SSL-VPN](media/f5-sso-vpn/ssl-vpn-architecture.png)

>[!NOTE]
>Tous les exemples de chaînes et de valeurs référencés dans ce guide doivent être remplacés par ceux de votre environnement réel.

## <a name="prerequisites"></a>Prérequis

Aucune expérience ni connaissance préalable de F5 BIG-IP n’est requise. Toutefois, vous avez besoin des éléments suivants :

- Un [abonnement gratuit](https://azure.microsoft.com/trial/get-started-active-directory/) Azure AD ou plus

- Les identités des utilisateurs doivent être [synchronisées à partir de l’annuaire local](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sync-whatis) à Azure AD.

- Un compte disposant des [autorisations](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles#application-administrator) d’administrateur d’application Azure AD

- Une infrastructure BIG-IP existante avec routage du trafic client vers et depuis le système BIG-IP, ou [déployez BIG-IP Virtual Edition dans Azure](f5-bigip-deployment-guide.md).

- Un enregistrement pour le service VPN publié BIG-IP doit exister dans le système DNS public, ou dans le fichier localhost d’un client de test pendant le test

- Le système BIG-IP doit être provisionné avec les certificats SSL nécessaires pour la publication de services via HTTPS.

Vous familiariser avec la [terminologie de F5 BIG-IP](https://www.f5.com/services/resources/glossary) vous aidera également à comprendre les divers composants référencés dans ce tutoriel.

>[!NOTE]
>Azure évolue constamment et vous trouverez peut-être de légères différences entre les instructions de ce guide et ce que vous voyez dans le portail Azure. Les captures d’écran proviennent de BIG-IP v15, mais elles restent relativement similaires à celles de la version 13.1.

## <a name="add-f5-big-ip-from-the-azure-ad-gallery"></a>Ajouter F5 BIG-IP depuis la galerie Azure AD

La configuration d’une approbation de fédération SAML entre BIG-IP permet au système Azure AD BIG-IP de transmettre la pré-authentification et l’[accès conditionnel](https://docs.microsoft.com/azure/active-directory/conditional-access/overview) à Azure AD, avant d’accorder l’accès au service VPN publié.

1. Connectez-vous au portail Azure AD à l’aide d’un compte disposant des droits d’administrateur d’application.

2. Dans le volet de navigation gauche, sélectionnez le **service Azure Active Directory**.

3. Accédez à **Applications d’entreprise**, puis, dans le ruban supérieur, sélectionnez **Nouvelle application**.

4. Recherchez F5 dans la Galerie, puis sélectionnez **Intégration d’Azure AD avec F5 BIG-IP APM**.

5. Fournissez un nom pour l’application, puis sélectionnez **Ajouter/Créer** pour l’ajouter à votre locataire. L’utilisateur peut voir le nom sous la forme d’une icône dans les portails d’application Azure et Office 365. Le nom doit refléter ce service spécifique. Par exemple, VPN.

## <a name="configure-azure-ad-sso"></a>Configurer l’authentification unique Azure AD

1. Dans la vue de vos nouvelles propriétés d’application F5, accédez à **Manage** (Gérer) > **Single sign-on** (Authentification unique).

2. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**. Ignorez l’invite pour enregistrer les paramètres d’authentification unique en sélectionnant **No, I’ll save later** (Non, j’enregistrerai ultérieurement).

3. Dans le menu **Setup single sign-on with SAML** (Configurer l’authentification unique avec SAML), sélectionnez l’icône de stylet pour **Basic SAML Configuration** (Configuration SAML de base) pour spécifier les détails suivants :

   - Remplacez l’**URL d’identificateur** prédéfinie par l’URL de votre service publié BIG-IP. Par exemple : `https://ssl-vpn.contoso.com`

   - Procédez de même avec la zone de texte **Reply URL** (URL de réponse), y compris le chemin d’accès du point de terminaison SAML. Par exemple : `https://ssl-vpn.contoso.com/saml/sp/profile/post/acs`

   - Dans cette seule configuration, l’application fonctionne dans un mode initié par le fournisseur d’identité, où Azure AD fournit à l’utilisateur une assertion SAML avant de le rediriger vers le service BIG-IP SAML. Pour les applications qui ne prennent pas en charge le mode initié par le fournisseur d’identité, spécifiez l’**URL de connexion** pour le service BIG-IP SAML. Par exemple : `https://ssl-vpn.contoso.com`.

   - Pour l’URL de déconnexion, entrez le point de terminaison de déconnexion unique (SLO) BIG-IP APM précédé par l’en-tête d’hôte du service en cours de publication. Par exemple : `https://ssl-vpn.contoso.com/saml/sp/profile/redirect/slr`

   La fourniture d’une URL SLO garantit qu’une session utilisateur est terminée aux deux extrémités, côté BIG-IP et côté Azure AD, une fois que l’utilisateur s’est déconnecté. BIG-IP APM fournit également une [option](https://support.f5.com/csp/article/K12056) pour mettre fin à toutes les sessions lors de l’appel d’une URL d’application spécifique.

![Image montrant une configuration SAML de base](media/f5-sso-vpn/basic-saml-configuration.png).

>[!NOTE]
>Depuis TMOS v16, le point de terminaison de déconnexion unique SAML a été remplacé par /saml/sp/profile/redirect/slo.

4. Sélectionnez **Save** (Enregistrer) avant de quitter le menu de configuration SAML et ignorez l’invite de test SSO.

Observez les propriétés de la section **User Attributes & Claims** (Attributs utilisateur et revendications), car Azure AD les fournira aux utilisateurs pour l’authentification BIG-IP APM.

![Image montrant les attributs utilisateur et les revendications](media/f5-sso-vpn/user-attributes-claims.png)

N’hésitez pas à ajouter d’autres revendications spécifiques que votre service publié BIG-IP peut attendre, tout en notant que les revendications définies en plus de l’ensemble par défaut sont émises seulement si elles existent dans Azure AD, en tant qu’attributs renseignés. De la même façon, les appartenances à un [groupe ou rôle](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-fed-group-claims) d’annuaire doivent également être définies par rapport à un objet utilisateur dans Azure AD avant de pouvoir être émises en tant que revendication.

![Image montrant un lien de téléchargement des métadonnées de fédération](media/f5-sso-vpn/saml-signing-certificate.png)

Les certificats de signature SAML créés par Azure AD ont une durée de vie de trois ans et auront besoin d’être gérés selon les conseils publiés pour Azure AD.

### <a name="azure-ad-authorization"></a>Autorisation Azure AD

Par défaut, Azure AD émet uniquement des jetons pour les utilisateurs auxquels l’accès à un service a été accordé.

1. Toujours dans la vue de la configuration de l’application, sélectionnez **Utilisateurs et groupes**.

2. Sélectionnez **+ Ajouter un utilisateur**, puis **Utilisateurs et groupes** dans le menu Ajouter une attribution.

3. Dans la boîte de dialogue **Utilisateurs et groupes**, ajoutez les groupes d’utilisateurs qui sont autorisés à accéder au VPN, puis choisissez **Sélectionner** > **Attribuer**.

![Image montrant l’ajout d’un lien utilisateur ](media/f5-sso-vpn/add-user-link.png)

4. Cela termine la partie Azure AD de l’approbation de fédération SAML. BIG-IP APM peut maintenant être configuré pour publier le service SSL-VPN et configuré avec un ensemble de propriétés correspondant pour terminer l’approbation, pour la pré-authentification SAML.

## <a name="big-ip-apm-configuration"></a>Configuration de BIG-IP APM

### <a name="saml-federation"></a>Fédération SAML

La section suivante crée le fournisseur de services BIG-IP SAML et les objets de fournisseur d’identité SAML correspondants requis pour effectuer la fédération du service VPN avec Azure AD.

1. Accédez à **Access** (Accéder) > **Federation** (Fédération) > **SAML Service Provider** (Fournisseur de services SAML) > **Local SP Services** (Services SP locaux) et sélectionnez **Create** (Créer).

![Image montrant une configuration BIG-IP SAML](media/f5-sso-vpn/bigip-saml-configuration.png)

2. Entrez un **nom** et la même valeur d’**ID d’entité** que vous avez définie précédemment dans Azure AD, ainsi que le nom de domaine complet d’hôte qui sera utilisé pour la connexion à l’application.

![Image montrant la création d’un nouveau service SP SAML](media/f5-sso-vpn/create-new-saml-sp.png)

   Les paramètres de nom de fournisseur de services (**SP Name Settings**) sont requis uniquement si l’ID d’entité ne correspond pas exactement à la partie nom d’hôte de l’URL publiée, ou si elle n’est pas dans un format d’URL standard basé sur un nom d’hôte. Spécifiez le schéma externe et le nom d’hôte de l’application publiée si l’ID d’entité est `urn:ssl-vpn:contosoonline`.

3. Faites défiler l’affichage vers le bas pour sélectionner le nouvel **objet SP SAML** et sélectionnez **Bind/UnBind IDP Connectors** (Lier/dissocier les connecteurs IDP).

![Image montrant la création de la fédération avec le service SP local](media/f5-sso-vpn/federation-local-sp-service.png)

4. Sélectionnez **Create New IDP Connector** (Créer un connecteur IDP), puis, dans le menu déroulant, sélectionnez **From Metadata** (À partir des métadonnées).

![Image montrant la création d’un nouveau connecteur IDP](media/f5-sso-vpn/create-new-idp-connector.png)

5. Accédez au fichier XML de métadonnées de fédération que vous avez téléchargé précédemment et spécifiez un paramètre **Identity Provider Name** (Nom de fournisseur d’identité) pour l’objet APM qui représentera le fournisseur d’identité SAML externe.

6. Sélectionnez **Add New Row** (Ajouter une nouvelle ligne) pour sélectionner le nouveau connecteur IDP externe Azure AD.

![Image montrant le connecteur IDP externe](media/f5-sso-vpn/external-idp-connector.png)

7. Sélectionnez **Update** (Mettre à jour) pour lier l’objet SP SAML à l’objet IDP SAML, puis sélectionnez **OK**.

![Image montrant le fournisseur d’identité SAML utilisant le fournisseur de services](media/f5-sso-vpn/saml-idp-using-sp.png)

### <a name="webtop-configuration"></a>Configuration des webtops

Les étapes suivantes permettent de proposer le service SSL-VPN aux utilisateurs via le portail web propriétaire de BIG-IP.

1. Accédez à **Access** (Accéder) > **Webtops** > **Webtop Lists** (Listes des webtops), puis sélectionnez **Create** (Créer).

2. Donnez un nom au portail et définissez le type sur **Full** (Complet). Par exemple : `Contoso_webtop`.

3. Ajustez les préférences restantes, puis sélectionnez **Finished** (Terminé).

![Image montrant la configuration des webtops](media/f5-sso-vpn/webtop-configuration.png)

### <a name="vpn-configuration"></a>Configuration VPN

La fonctionnalité VPN est constituée de plusieurs éléments, chacun contrôlant un aspect différent du service global.

1. Accédez à **Access** (Accéder) > **Connectivity/VPN** (Connectivité/VPN) > **Network Access (VPN)** (Accès réseau (VPN)) > **IPV4 Lease Pools** (Pools de baux IPV4) et sélectionnez **Create** (Créer).

2. Spécifiez un nom pour le pool d’adresses IP allouées aux clients VPN. Par exemple, Contoso_vpn_pool.

3. Définissez le type sur **IP Address Range** (Plage d’adresses IP) et spécifiez une adresse IP de début et de fin, puis sélectionnez **Add** (Ajouter) et **Finished** (Terminé).

![Image montrant la configuration du VPN](media/f5-sso-vpn/vpn-configuration.png)

Une liste d’accès réseau provisionne le service avec les paramètres IP et DNS du pool VPN, les autorisations de routage des utilisateurs et peut également lancer des applications, si nécessaire.

1. Accédez à **Access** (Accéder) > **Connectivity/VPN: Network Access (VPN)** (Connectivité/VPN : Accès réseau (VPN)) > **Network Access Lists** (Listes d’accès réseau) et sélectionnez **Create** (Créer).

2. Donnez un nom pour la liste d’accès VPN et une légende, par exemple, Contoso-VPN, puis sélectionnez **Finished** (Terminé).

![Image montrant la configuration VPN dans une liste d’accès réseau](media/f5-sso-vpn/vpn-configuration-network-access-list.png)

3. Dans le ruban supérieur, sélectionnez **Network Settings** (Paramètres réseau) et ajoutez les paramètres ci-dessous :

   • **Supported IP version** (Version IP prise en charge) : IPV4

   • **IPV4 Lease Pool** (Pool de baux IPV4) : Sélectionnez le pool VPN créé précédemment, par exemple, Contoso_vpn_pool.

![Image montrant le pool VPN de Contoso](media/f5-sso-vpn/contoso-vpn-pool.png)

   Les options Client Settings (Paramètres client) peuvent être utilisées pour appliquer des restrictions sur la façon dont le trafic client est routé quand un VPN est établi.

4. Sélectionnez **Finished** (Terminé) et accédez à l’onglet DNS/Hosts (DNS/Hôtes) pour ajouter les paramètres :

   • **IPV4 Primary Name Server** (Serveur de noms principaux IPV4) : adresse IP du serveur DNS de votre environnement

   • **DNS Default Domain Suffix** (Suffixe de domaine par défaut DNS) : suffixe de domaine pour cette connexion VPN spécifique. Par exemple, contoso.com

![Image montrant le suffixe de domaine par défaut](media/f5-sso-vpn/domain-suffix.png)

Cet [article F5](https://techdocs.f5.com/kb/en-us/products/big-ip_apm/manuals/product/apm-network-access-11-5-0/2.html) fournit des détails sur l’ajustement des paramètres restants en fonction de vos préférences.

Un profil de connexion BIG-IP est désormais requis pour configurer les paramètres de chacun des types de client VPN que le service VPN doit prendre en charge. Par exemple, Windows, OSX et Android.

1. Accédez à **Access** (Accéder) > **Connectivity/VPN** (Connectivité/VPN) > **Connectivity** (Connectivité) > **Profiles** (Profils) et sélectionnez **Add** (Ajouter).

2. Spécifiez un nom de profil et définissez le profil parent sur **/Common/connectivity**, par exemple, Contoso_VPN_Profile.

![Image montrant la création d’un nouveau profil de connectivité](media/f5-sso-vpn/create-connectivity-profile.png)

La [documentation](https://techdocs.f5.com/kb/en-us/bigip-edge-apps.html) de F5 fournit plus de détails sur la prise en charge des clients.

## <a name="access-profile-configuration"></a>Configuration du profil d’accès

Une fois les objets VPN configurés, une stratégie d’accès est requise pour activer le service pour l’authentification SAML.

1. Accédez à **Access** (Accéder) > **Profiles/Policies** (Profils/Stratégies) > **Access Profiles (Per-Session Policies)** (Profils d’accès (stratégies par session)) et sélectionnez **Create** (Créer).

2. Spécifiez un nom de profil et, pour le type de profil, sélectionnez **All** (Tous), par exemple, Contoso_network_access.

3. Faites défiler l’affichage vers le bas pour ajouter au moins une langue dans **Accepted Languages** (Langues acceptées) et sélectionnez **Finished** (Terminé).

![Image montrant les propriétés générales](media/f5-sso-vpn/general-properties.png)

4. Sélectionnez **Edit** (Modifier) dans le champ Per-Session Policy (Stratégie par session) du nouveau profil d’accès pour que l’éditeur de stratégie visuel démarre dans un onglet de navigateur distinct.

![Image montrant une stratégie par session](media/f5-sso-vpn/per-session-policy.png)

5. Sélectionnez le signe **+** et, dans le menu contextuel, sélectionnez **Authentication** (Authentification) > **SAML Auth** (Authentification SAML) > **Add Item** (Ajouter un élément).

6. Dans la configuration du fournisseur de services d’authentification SAML, sélectionnez l’objet SP SAML VPN que vous avez créé précédemment, puis sélectionnez **Enregistrer**.

![Image montrant l’authentification SAML](media/f5-sso-vpn/saml-authentication.png)

7. Sélectionnez **+** dans la branche Successful (Réussite) de l’authentification SAML.

8. Dans l’onglet Assignment (Affectation), sélectionnez **Advanced Resource Assign** (Affectation avancée de ressources), puis sélectionnez **Add Item** (Ajouter un élément).

![Image montrant l’affectation avancée de ressources](media/f5-sso-vpn/advance-resource-assign.png)

9. Dans la fenêtre contextuelle, sélectionnez **New Entry** (Nouvelle entrée) puis **Add/Delete** (Ajouter/Supprimer).

10. Dans la fenêtre enfant, sélectionnez **Network Access** (Accès réseau), puis sélectionnez le profil d’accès réseau créé précédemment.

![Image montrant l’ajout d’une nouvelle entrée d’accès réseau](media/f5-sso-vpn/add-new-entry.png)

11. Basculez vers l’onglet **Webtop** et ajoutez l’objet Webtop créé précédemment.

![Image montrant l’ajout d’un objet Webtop](media/f5-sso-vpn/add-webtop-object.png)

12. Sélectionnez **Update** (Mettre à jour), puis **Save** (Enregistrer).

13. Sélectionnez le lien dans la zone Deny (Refuser) du haut pour modifier la branche Successful (Réussite) en sélectionnant **Allow** (Autoriser), puis sélectionnez **Save** (Enregistrer).

![Image montrant un nouvel éditeur de stratégie visuelle](media/f5-sso-vpn/vizual-policy-editor.png)

14. Validez ces paramètres en sélectionnant **Apply Access Policy** (Appliquer la stratégie d’accès) et fermez l’onglet de l’éditeur de stratégie visuelle.

![Image montrant un nouveau gestionnaire de stratégie d’accès](media/f5-sso-vpn/access-policy-manager.png)

## <a name="publish-the-vpn-service"></a>Publier le service VPN

Avec tous les paramètres en place, le service APM nécessite désormais un serveur virtuel frontal pour écouter les clients qui se connectent au VPN.

1. Sélectionnez **Local Traffic** (Trafic local) > **Virtual Servers** (Serveurs virtuels) > **Virtual Server List** (Liste des serveurs virtuels) et sélectionnez **Create** (Créer).

2. Fournissez un **nom** pour le serveur virtuel VPN, par exemple, **VPN_Listener**.

3. Fournissez au serveur virtuel une **adresse de destination IP** inutilisée dont le routage est en place pour recevoir le trafic client.

4. Définissez le port de service sur **443 HTTPS** et veillez à ce que l’état indique **Enabled** (Activé).

![Image montrant un nouveau serveur virtuel](media/f5-sso-vpn/new-virtual-server.png)

5. Définissez le **profil HTTP** sur http et ajoutez le profil SSL (client) pour le certificat SSL public que vous avez provisionné dans le cadre des prérequis.

![Image montrant le profil SSL](media/f5-sso-vpn/ssl-profile.png)

6. Sous Access Policy (Stratégie d’accès), définissez le **profil d’accès** et le **profil de connectivité** pour utiliser les objets VPN créés.

![Image montrant la stratégie d’accès](media/f5-sso-vpn/access-policy.png)

7. Sélectionnez **Finished** (Terminé) une fois que vous avez terminé.

8.  Votre service SSL-VPN est désormais publié et accessible via SHA, soit directement via son URL, soit via les portails d’application de Microsoft.

## <a name="additional-resources"></a>Ressources supplémentaires

- [Fin des mots de passe, adoptez l’authentification sans mot de passe](https://www.microsoft.com/security/business/identity/passwordless)

- [Qu’est-ce que l’accès conditionnel ?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Microsoft Zero Trust framework to enable remote work](https://www.microsoft.com/security/blog/2020/04/02/announcing-microsoft-zero-trust-assessment-tool/)

- [Cinq étapes pour l’intégration de toutes vos applications à Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/five-steps-to-full-application-integration-with-azure-ad)

## <a name="next-steps"></a>Étapes suivantes

Ouvrez un navigateur sur un client Windows distant et accédez à l’URL du **service VPN BIG-IP**. Après vous être authentifié auprès d’Azure AD, vous verrez le portail Webtop de BIG-IP et le lanceur VPN.

![Image montrant le lanceur VPN](media/f5-sso-vpn/vpn-launcher.png)

La sélection de la vignette VPN installe le client BIG-IP Edge et établit une connexion VPN configurée pour SHA.
L’application F5 VPN doit également être visible en tant que ressource cible dans l’accès conditionnel Azure AD. Consultez notre [guide](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-policies) pour créer des stratégies d’accès conditionnel et permettre aux utilisateurs de bénéficier d’une [authentification sans mot de passe](https://www.microsoft.com/security/business/identity/passwordless) Azure AD.