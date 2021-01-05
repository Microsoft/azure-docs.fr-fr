---
title: Prise en main du Serveur Azure MFA – Azure Active Directory
description: Prise en main pas à pas du serveur Azure Multi-Factor Authentication local
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: d7bd9913cccbe077a4deed9a7c5bfdc601f3dd5e
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742338"
---
# <a name="getting-started-with-the-azure-multi-factor-authentication-server"></a>Prise en main du serveur Azure Multi-Factor Authentication

<center>

![Bien démarrer avec MFA Server local](./media/howto-mfaserver-deploy/server2.png)</center>

Cette page vous indique comment installer le serveur et le configurer avec votre Active Directory local. Si le serveur MFA est déjà installé et que vous souhaitez effectuer une mise à niveau, voir [Mise à niveau vers le serveur Azure Multi-Factor Authentication le plus récent](howto-mfaserver-deploy-upgrade.md). Pour plus d’informations sur l’installation du service web, voir [Déploiement du service web de l’application mobile pour le serveur Azure Multi-Factor Authentication](howto-mfaserver-deploy-mobileapp.md).

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisés avec l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

## <a name="plan-your-deployment"></a>Planifier votre déploiement

Avant de télécharger le serveur Microsoft Azure Multi-Factor Authentication, pensez à vos exigences en termes de charge et de haute disponibilité. Prenez en compte ces informations pour décider de quelle manière et à quel endroit effectuer le déploiement.

Le nombre d’utilisateurs que vous souhaitez authentifier, sur une base régulière, est une bonne ligne directrice pour avoir une idée de la quantité de mémoire nécessaire.

| Utilisateurs | RAM |
| ----- | --- |
| 1-10,000 | 4 Go |
| 10,001-50,000 | 8 Go |
| 50,001-100,000 | 12 Go |
| 100,000-200,001 | 16 Go |
| 200,001+ | 32 Go |

Vous devez configurer plusieurs serveurs pour l’équilibrage de charge ou la haute disponibilité ? Il existe plusieurs façons de définir cette configuration avec le serveur Azure MFA. Lorsque vous installez votre premier serveur Azure MFA, il devient le maître. Tous les serveurs supplémentaires deviennent ses subordonnés. Ils synchronisent automatiquement les utilisateurs ainsi que la configuration avec le serveur maître. Par la suite, vous pouvez configurer un serveur principal et conserver le reste des serveurs en tant que serveurs de sauvegarde. Vous pouvez aussi configurer l’équilibrage de charge parmi l’ensemble des serveurs.

Lorsqu’un serveur Azure MFA maître est hors connexion, les serveurs subordonnés peuvent continuer à traiter les requêtes de vérification en deux étapes. Toutefois, vous ne pouvez pas ajouter de nouveaux utilisateurs. Quant aux utilisateurs existants, ils ne peuvent mettre à jour leurs paramètres que lorsqu’un serveur maître est à nouveau en ligne ou qu’un serveur subordonné est promu.

### <a name="prepare-your-environment"></a>Préparation de votre environnement

Assurez-vous que le serveur que vous utilisez pour l’authentification multifacteur Azure répond aux exigences suivantes :

| Configuration requise du serveur Azure Multi-Factor Authentication | Description |
|:--- |:--- |
| Matériel |<li>200 Mo d’espace disque</li><li>Processeur compatible x32 ou x64</li><li>1 Go de RAM ou plus</li> |
| Logiciel |<li>Windows Server 2016</li><li>Windows Server 2012 R2</li><li>Windows Server 2012</li><li>Windows Server 2008/R2 (avec [ESU](/lifecycle/faq/extended-security-updates) uniquement)</li><li>Windows 10</li><li>Windows 8.1, toutes les éditions</li><li>Windows 8, toutes les éditions</li><li>Windows 7, toutes les éditions (avec [ESU](/lifecycle/faq/extended-security-updates) uniquement)</li><li>Microsoft .NET 4.0 Framework</li><li>IIS 7.0 ou version ultérieure si vous installez le kit de développement logiciel du portail de l’utilisateur ou du service web</li> |
| Autorisations | Compte d’administrateur de domaine ou d’administrateur d’entreprise pour l’inscription auprès d’Active Directory |

### <a name="azure-mfa-server-components"></a>Composants du serveur Azure MFA

Le serveur Azure MFA est constitué de trois composants Web :

* Kit de développement logiciel (SDK) Web Services : il assure la communication avec les autres composants et est installé sur le serveur d’applications Azure MFA
* Portail utilisateur : un site web IIS qui permet aux utilisateurs de s’inscrire dans Azure Multi-Factor Authentication (MFA) et de mettre à jour leurs comptes.
* Service Web de l’application mobile : il permet l’utilisation d’une application mobile comme l’application Microsoft Authenticator, pour une vérification en deux étapes.

Ces trois composants peuvent être installés sur le même serveur s’il est accessible sur Internet. Si vous séparez les composants, le kit de développement logiciel (SDK) Web Service est installé sur le serveur d’applications Azure MFA. Le portail utilisateur et le service Web de l’application mobile sont quant à eux installés sur un serveur accessible sur Internet.

### <a name="azure-multi-factor-authentication-server-firewall-requirements"></a>Configuration requise du serveur Azure Multi-Factor Authentication

Chaque serveur MFA doit pouvoir communiquer sur les éléments sortants du port 443, vers les adresses suivantes :

* https://pfd.phonefactor.net
* https://pfd2.phonefactor.net
* https://css.phonefactor.net

Si les pare-feu sortants sont limités sur le port 443, ouvrez les plages d'adresses IP suivantes :

| Sous-réseau IP | Masque réseau | Plage d’adresses IP |
|:---: |:---: |:---: |
| 134.170.116.0/25 |255.255.255.128 |134.170.116.1 – 134.170.116.126 |
| 134.170.165.0/25 |255.255.255.128 |134.170.165.1 – 134.170.165.126 |
| 70.37.154.128/25 |255.255.255.128 |70.37.154.129 – 70.37.154.254   |
| 52.251.8.48/28   | 255.255.255.240 | 52.251.8.48 - 52.251.8.63     |
| 52.247.73.160/28 | 255.255.255.240 | 52.247.73.160 - 52.247.73.175 |
| 52.159.5.240/28  | 255.255.255.240 | 52.159.5.240 - 52.159.5.255   |
| 52.159.7.16/28   | 255.255.255.240 | 52.159.7.16 - 52.159.7.31     |
| 52.250.84.176/28 | 255.255.255.240 | 52.250.84.176 - 52.250.84.191 |
| 52.250.85.96/28  | 255.255.255.240 | 52.250.85.96 - 52.250.85.111  |

Si vous n’utilisez pas la fonctionnalité de confirmation d’événement et que les utilisateurs ne se servent pas d’applications mobiles pour effectuer des vérifications à partir des appareils sur le réseau de l’entreprise, vous avez uniquement besoin des plages suivantes :

| Sous-réseau IP | Masque réseau | Plage d’adresses IP |
|:---: |:---: |:---: |
| 134.170.116.72/29 |255.255.255.248 |134.170.116.72 – 134.170.116.79|
| 134.170.165.72/29 |255.255.255.248 |134.170.165.72 – 134.170.165.79|
| 70.37.154.200/29 |255.255.255.248  |70.37.154.201 – 70.37.154.206  |
| 52.251.8.48/28   | 255.255.255.240 | 52.251.8.48 - 52.251.8.63     |
| 52.247.73.160/28 | 255.255.255.240 | 52.247.73.160 - 52.247.73.175 |
| 52.159.5.240/28  | 255.255.255.240 | 52.159.5.240 - 52.159.5.255   |
| 52.159.7.16/28   | 255.255.255.240 | 52.159.7.16 - 52.159.7.31     |
| 52.250.84.176/28 | 255.255.255.240 | 52.250.84.176 - 52.250.84.191 |
| 52.250.85.96/28  | 255.255.255.240 | 52.250.85.96 - 52.250.85.111  |

## <a name="download-the-mfa-server"></a>Télécharger le serveur MFA

Procédez comme suit pour télécharger le serveur Azure Multi-Factor Authentication (MFA) à partir du portail Azure :

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur à leurs utilisateurs doivent utiliser la fonctionnalité Azure AD Multi-Factor Authentication basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisée avec Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude. Les étapes suivantes fonctionnent uniquement si vous êtes un client du Serveur Azure MFA.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.
2. Recherchez et sélectionnez *Azure Active Directory*. Sélectionnez **Sécurité** > **MFA**.
3. Sous **Serveur MFA du gestionnaire**, sélectionnez **Paramètres du serveur**.
4. Cliquez sur **Télécharger** et suivez les instructions de la page de téléchargement pour enregistrer le programme d’installation. 

   ![Télécharger MFA Server à partir du portail Azure](./media/howto-mfaserver-deploy/downloadportal.png)

5. Ne fermez pas cette page, car nous y faisons mention après avoir exécuté le programme d’installation.

## <a name="install-and-configure-the-mfa-server"></a>Installer et configurer le serveur MFA

Maintenant que vous avez téléchargé le serveur, vous pouvez l’installer et le configurer. Assurez-vous que le serveur que vous installez répond aux exigences regroupées sur la liste dans la section de planification.

1. Double-cliquez sur le fichier exécutable.
2. Sur l'écran Sélectionner le dossier d’installation, assurez-vous que le dossier est correct, puis cliquez sur **Suivant**.
3. Une fois l'installation terminée, cliquez sur **Terminer**. L'Assistant de configuration démarre.
4. Sur l’écran d’accueil de l’Assistant Configuration, cochez l’option **Ignorer l’Assistant Configuration de l’authentification**, puis cliquez sur **Suivant**. La fermeture de l’Assistant intervient et le serveur démarre.

   ![Ignorer l’Assistant Configuration de l’authentification](./media/howto-mfaserver-deploy/skip2.png)

5. Revenez à la page à partir de laquelle vous avez téléchargé le serveur, puis cliquez sur le bouton **Générer des informations d’identification d’activation**. Copiez ces informations dans les zones appropriées du serveur Azure MFA, puis cliquez sur **Activer**.

> [!NOTE]
> Seuls des administrateurs généraux peuvent générer des informations d’identification d’activation dans le portail Azure.

## <a name="send-users-an-email"></a>Envoi d’un e-mail aux utilisateurs

Pour faciliter le lancement, autorisez le serveur MFA à communiquer avec vos utilisateurs. Le serveur MFA peut envoyer un e-mail les informant qu’ils ont été inscrits pour la vérification en deux étapes.

L’e-mail que vous envoyez doit être déterminé par la façon dont vous configurez vos utilisateurs pour la vérification en deux étapes. Par exemple, si vous pouvez importer les numéros de téléphone à partir du répertoire de l’entreprise, l’e-mail doit inclure les numéros de téléphone par défaut afin que les utilisateurs sachent à quoi s’attendre. Si vous ne pouvez pas importer les numéros de téléphone ou si les utilisateurs doivent recourir à l’application mobile, envoyez-leur un e-mail leur demandant d’effectuer l’inscription de leur compte. en fournissant un lien hypertexte permettant d’accéder au portail utilisateur Azure Multi-Factor Authentication.

Le contenu de l’e-mail varie en fonction de la méthode de vérification définie pour l'utilisateur (appel téléphonique, SMS ou application mobile). Par exemple, si l'utilisateur doit saisir un code confidentiel pour s'authentifier, l’e-mail lui indique le code confidentiel initial qui a été défini. Les utilisateurs doivent modifier leur code PIN lors de leur première vérification.

### <a name="configure-email-and-email-templates"></a>Configuration du courrier électronique et des modèles de courrier électronique

Cliquez sur l'icône de courrier électronique sur la gauche pour configurer les paramètres d'envoi de ces e-mails. Cette page vous permet de saisir les informations SMTP de votre serveur de messagerie et d’envoyer des e-mails, en cochant la case **Envoyer des e-mails aux utilisateurs**.

![Configuration de la messagerie du serveur MFA](./media/howto-mfaserver-deploy/email1.png)

L'onglet Contenu des e-mails présente tous les modèles d’e-mail disponibles. Selon la façon dont les utilisateurs ont été configurés pour utiliser la vérification en deux étapes, vous pouvez choisir le modèle qui correspond le mieux à vos besoins.

![Modèles d’e-mail de MFA Server dans la console](./media/howto-mfaserver-deploy/email2.png)

## <a name="import-users-from-active-directory"></a>Importation des utilisateurs à partir d'Active Directory

Maintenant que le serveur est installé, vous pouvez ajouter des utilisateurs. Vous pouvez les créer manuellement, importer des utilisateurs à partir d’Active Directory ou configurer la synchronisation automatique avec Active Directory.

### <a name="manual-import-from-active-directory"></a>Importation manuelle à partir d’Active Directory

1. Sur la gauche du serveur Azure MFA, sélectionnez **Utilisateurs**.
2. En bas de la page, sélectionnez **Importer à partir d’Active Directory**.
3. Vous pouvez désormais rechercher des utilisateurs individuels ou bien le répertoire Active des unités d'organisation, composé lui-même d’utilisateurs. Dans ce cas, nous spécifions l'unité d'organisation des utilisateurs.
4. Mettez en surbrillance tous les utilisateurs mentionnés sur la droite, puis cliquez sur **Importer**. Vous devriez recevoir un message vous indiquant que l’opération a été réalisée avec succès. Fermez la fenêtre d'importation.

   ![Importation des utilisateurs de MFA Server à partir d’Active Directory](./media/howto-mfaserver-deploy/import2.png)

### <a name="automated-synchronization-with-active-directory"></a>Synchronisation automatique avec Active Directory

1. Sur la gauche du serveur Azure MFA, sélectionnez **Intégration d’annuaire**.
2. Accédez à l’onglet **Synchronisation**.
3. En bas, choisissez **Ajouter**
4. Dans la zone **Ajouter objet à synchroniser** qui s’affiche, choisissez le domaine, l’unité d’organisation **ou** le groupe de sécurité, les paramètres, la méthode et la langue par défaut de cette tâche de synchronisation, puis cliquez sur **Ajouter**.
5. Vérifiez la zone intitulée **Activer la synchronisation avec Active Directory** et choisissez un **Intervalle de synchronisation** compris entre une minute et 24 heures.

## <a name="how-the-azure-multi-factor-authentication-server-handles-user-data"></a>Comment le serveur Azure Multi-Factor Authentication gère les données utilisateur

Lorsque vous utilisez le serveur Multi-Factor Authentication (MFA) localement, les données utilisateur sont stockées sur les serveurs locaux. Aucune donnée utilisateur persistante n'est stockée dans le cloud. Lorsque l'utilisateur effectue une vérification en deux étapes, le serveur MFA envoie des données au service cloud Azure MFA pour effectuer la vérification. Lorsque ces demandes d’authentification sont envoyées au service cloud, les champs suivants sont envoyés dans la demande et les journaux d’activité afin qu’ils soient disponibles dans les rapports d’utilisation/d’authentification du client. Certains champs étant facultatifs, ils peuvent être activés ou désactivés sur le serveur Multi-Factor Authentication. La communication du serveur MFA au service cloud MFA utilise SSL/TLS sur le port 443 sortant. Ces champs sont les suivants :

* ID unique : nom d'utilisateur ou ID du serveur MFA interne
* Prénom et nom (facultatif)
* Adresse de messagerie (facultatif)
* Numéro de téléphone : en cas d'authentification par appel vocal ou SMS
* Jeton du périphérique : en cas d'authentification par application mobile
* Mode d'authentification
* Résultat de l'authentification
* Nom du serveur MFA
* Adresse IP du serveur MFA
* Adresse IP du client (si disponible)

En plus des champs ci-dessus, le résultat de la vérification (réussite/échec) et le motif de refus sont également stockés avec les données d'authentification et disponibles dans les rapports d'utilisation/d'authentification.

> [!IMPORTANT]
> Depuis mars 2019, les options d’appel téléphonique ne sont plus disponibles pour les utilisateurs de MFA Server dans des locataires Azure AD gratuits ou à l’essai. Cette modification n’affecte pas les messages SMS. Les appels téléphoniques continueront à être disponibles pour les utilisateurs de locataires Azure AD payants. Ce changement affecte uniquement les locataires Azure AD gratuits ou à l’essai.

## <a name="back-up-and-restore-azure-mfa-server"></a>Sauvegarder et restaurer le serveur Azure MFA

Être sûr de disposer d’une sauvegarde valide est une étape importante dans l’utilisation de n’importe quel système.

Pour sauvegarder le serveur Azure MFA, veillez à disposer d’une copie du dossier **C:\Program Files\Multi-Factor Authentication Server\Data**, avec le fichier **PhoneFactor.pfdata**. 

Si une restauration est nécessaire, procédez comme suit :

1. Réinstallez le serveur Azure MFA sur un nouveau serveur.
2. Activez le nouveau serveur Azure MFA.
3. Arrêtez le service **MultiFactorAuth**.
4. Remplacez le fichier **PhoneFactor.pfdata** par la copie sauvegardée.
5. Démarrez le service **MultiFactorAuth**.

Le nouveau serveur est maintenant en place, avec les données d’utilisateur et de configuration d’origine que vous aviez sauvegardées.

## <a name="managing-the-tlsssl-protocols-and-cipher-suites"></a>Gestion des protocoles TLS/SSL et des suites de chiffrement

Une fois que vous avez installé la version 8.x ou ultérieure du serveur MFA ou que vous avez effectué une mise à niveau vers cette dernière, il est recommandé que les suites de chiffrement plus anciennes et plus faible soient désactivées ou supprimées, à moins qu’elles ne soient requises par votre organisation. Vous trouverez plus d’informations sur la manière d’effectuer cette tâche dans l’article [Gestion des protocoles SSL/TLS et des suites de chiffrement pour AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs)

## <a name="next-steps"></a>Étapes suivantes

- Installez et configurez le [portail utilisateur](howto-mfaserver-deploy-userportal.md) pour une utilisation en libre-service.
- Installez et configurez le serveur Azure MFA avec [Active Directory Federation Service](multi-factor-authentication-get-started-adfs.md), [l’authentification RADIUS](howto-mfaserver-dir-radius.md) ou [l’authentification LDAP](howto-mfaserver-dir-ldap.md).
- Installez et configurez la [passerelle des services Bureau à distance et Azure Multi-Factor Authentication Server avec RADIUS](howto-mfaserver-nps-rdg.md).
- [Déployez le service web de l’application mobile du serveur Azure Multi-Factor Authentication](howto-mfaserver-deploy-mobileapp.md).
- [Scénarios avancés avec Azure Multi-Factor Authentication et des VPN tiers](howto-mfaserver-nps-vpn.md).