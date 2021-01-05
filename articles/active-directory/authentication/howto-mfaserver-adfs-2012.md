---
title: Azure MFA Server avec AD FS dans Windows Server - Azure Active Directory
description: Cet article explique la prise en main d’Azure Multi-Factor Authentication et d’AD FS dans Windows Server 2012 R2 et 2016.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/11/2018
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6fd1bf4f9c463973d70f5289f7a82f372d0156cb
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742543"
---
# <a name="configure-azure-multi-factor-authentication-server-to-work-with-ad-fs-in-windows-server"></a>Configuration du serveur Azure Multi-Factor Authentication pour travailler avec AD FS dans Windows Server

Si vous utilisez Active Directory Federation Services (AD FS) et que vous souhaitez sécuriser des ressources cloud ou locales, vous pouvez configurer le serveur Azure Multi-Factor Authentication pour l’intégrer avec AD FS. Cette configuration active la vérification en deux étapes pour les points de terminaison de valeur élevée.

Cet article traite de l’utilisation du serveur Azure Multi-Factor Authentication avec AD FS dans Windows Server 2012 R2 ou Windows Server 2016. Pour plus d’informations, consultez l’article qui explique comment [sécuriser les ressources de cloud et locales à l’aide du serveur Azure Multi-Factor Authentication avec AD FS 2.0](howto-mfaserver-adfs-2.md).

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité Azure AD Multi-Factor Authentication basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisée avec Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Si vous utilisez une authentification multifacteur basée sur le cloud, consultez [Sécuriser les ressources cloud avec Azure AD Multi-Factor Authentication et AD FS](howto-mfa-adfs.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

## <a name="secure-windows-server-ad-fs-with-azure-multi-factor-authentication-server"></a>Sécuriser Windows Server AD FS avec le serveur Azure Multi-Factor Authentication

Lorsque vous installez le serveur Azure Multi-Factor Authentication, vous pouvez choisir l’une des options suivantes :

* Installer le serveur Azure Multi-Factor Authentication localement sur le même serveur qu’AD FS
* Installer l’adaptateur Azure Multi-Factor Authentication localement sur le serveur AD FS, puis installer le serveur Multi-Factor Authentication sur un autre ordinateur

Avant de commencer, tenez compte des informations suivantes :

* Vous n’êtes pas obligé d’installer le serveur Azure Multi-Factor Authentication sur votre serveur AD FS. Toutefois, vous devez installer l’adaptateur Multi-Factor Authentication pour AD FS sur un serveur Windows Server 2012 R2 ou Windows Server 2016 exécutant AD FS. Vous pouvez installer le serveur sur un autre ordinateur, si vous installez l’adaptateur AD FS séparément sur votre serveur de fédération AD FS. Consultez les procédures suivantes pour savoir comment installer l’adaptateur séparément.
* Si votre entreprise utilise les méthodes de vérification par messagerie texte ou application mobile, les chaînes définies dans les paramètres de la société contiennent un espace réservé <$*application_name*$>. Dans le serveur MFA v7.1, vous pouvez fournir un nom d’application qui remplace cet espace réservé. Dans les versions 7.0 ou antérieures, cet espace réservé n’est pas automatiquement remplacé lorsque vous utilisez l’adaptateur AD FS. Pour les versions antérieures, supprimez l’espace réservé dans les chaînes appropriées lorsque vous sécurisez AD FS.
* Le compte que vous utilisez pour vous connecter doit disposer des droits d’utilisateur pour créer des groupes de sécurité dans votre service Active Directory.
* L’assistant d’installation de l’adaptateur d’authentification multifacteur AD FS crée un groupe de sécurité appelé PhoneFactor Admins dans votre instance d’Active Directory, puis ajoute le compte de service AD FS de votre service de fédération à ce groupe. Vérifiez que le groupe PhoneFactor Admins a été créé sur votre contrôleur de domaine et que le compte de service AD FS est membre de ce groupe. Si nécessaire, ajoutez manuellement le compte de service AD FS au groupe PhoneFactor Admins sur votre contrôleur de domaine.
* Pour plus d’informations sur l’installation du Kit de développement logiciel (SDK) du service Web avec le portail de l’utilisateur, consultez l’article sur le [déploiement du portail de l’utilisateur pour le serveur Azure Multi-Factor Authentication](howto-mfaserver-deploy-userportal.md).

### <a name="install-azure-multi-factor-authentication-server-locally-on-the-ad-fs-server"></a>Installer le serveur Azure Multi-Factor Authentication localement sur le serveur AD FS

1. Téléchargez et installez le serveur Azure Multi-Factor Authentication sur votre serveur AD FS. Pour plus d’informations concernant l’installation, consultez l’article sur la [prise en main du serveur Azure Multi-Factor Authentication](howto-mfaserver-deploy.md).
2. Sur la console de gestion du serveur Azure Multi-Factor Authentication, cliquez sur l’icône **AD FS**. Sélectionnez les options **Autoriser l’inscription utilisateur** et **Autoriser les utilisateurs à sélectionner la méthode**.
3. Sélectionnez les autres options que vous souhaitez définir pour votre entreprise.
4. Cliquez sur **Installer adaptateur AD FS**.

   ![Installer l’adaptateur ADFS à partir de la console de MFA Server](./media/howto-mfaserver-adfs-2012/server.png)

5. Si la fenêtre Active Directory s’affiche, cela signifie deux choses. Votre ordinateur est joint à un domaine et la configuration d’Active Directory pour sécuriser la communication entre l’adaptateur AD FS et le service Azure Multi-Factor Authentication est incomplète. Cliquez sur **Suivant** pour terminer automatiquement cette configuration ou cochez la case **Ignorer la configuration automatique d’Active Directory et configurer les paramètres manuellement**. Cliquez sur **Suivant**.
6. Si les fenêtres Groupe local s’affichent, cela signifie deux choses. Votre ordinateur n’est pas joint à un domaine et la configuration du groupe local pour sécuriser la communication entre l’adaptateur AD FS et le service Azure Multi-Factor Authentication est incomplète. Cliquez sur **Suivant** pour terminer automatiquement cette configuration ou cochez la case **Ignorer la configuration automatique du groupe local et configurer les paramètres manuellement**. Cliquez sur **Suivant**.
7. Dans l’assistant d’installation, cliquez sur **Suivant**. Le serveur Azure Multi-Factor Authentication crée le groupe PhoneFactor Admins et ajoute le compte de service AD FS au groupe PhoneFactor Admins.
8. Sur la page **Lancer le programme d’installation**, cliquez sur **Suivant**.
9. Dans le programme d’installation de l’adaptateur d’authentification multifacteur AD FS, cliquez sur **Suivant**.
10. Une fois l’installation terminée, cliquez sur **Fermer** .
11. Lorsque l’adaptateur a été installé, vous devez l’enregistrer dans AD FS. Ouvrez Windows PowerShell et exécutez la commande suivante :

    `C:\Program Files\Multi-Factor Authentication Server\Register-MultiFactorAuthenticationAdfsAdapter.ps1`

12. Modifiez la stratégie d’authentification globale dans AD FS pour utiliser votre nouvel adaptateur enregistré. Dans la console de gestion AD FS, accédez au nœud **Stratégies d’authentification** . Dans la section **Authentification multifacteur**, cliquez sur le lien **Modifier** en regard de la section **Paramètres globaux**. Dans la fenêtre **Modifier la stratégie d’authentification globale**, sélectionnez **Authentification multifacteur** comme méthode d’authentification supplémentaire, puis cliquez sur **OK**. L'adaptateur est enregistré en tant que WindowsAzureMultiFactorAuthentication. Redémarrez le service AD FS pour que l’enregistrement soit pris en compte.

![Modifier la stratégie d’authentification globale](./media/howto-mfaserver-adfs-2012/global.png)

À ce stade, le serveur d’authentification multifacteur est configuré comme fournisseur d’authentification supplémentaire à utiliser avec AD FS.

## <a name="install-a-standalone-instance-of-the-ad-fs-adapter-by-using-the-web-service-sdk"></a>Installer une version autonome de l’adaptateur AD FS à l’aide du SDK du service Web

1. Installez le SDK du service Web sur le serveur qui exécute le serveur Multi-Factor Authentication.
2. Copiez les fichiers suivants à partir du répertoire \Program Files\Multi-Factor Authentication Server sur le serveur sur lequel vous prévoyez d’installer l’adaptateur AD FS :
   * MultiFactorAuthenticationAdfsAdapterSetup64.msi
   * Register-MultiFactorAuthenticationAdfsAdapter.ps1
   * Unregister-MultiFactorAuthenticationAdfsAdapter.ps1
   * MultiFactorAuthenticationAdfsAdapter.config
3. Exécutez le fichier d’installation MultiFactorAuthenticationAdfsAdapterSetup64.msi.
4. Dans le programme d’installation de l’adaptateur d’authentification multifacteur AD FS, cliquez sur **Suivant** pour lancer l’installation.
5. Une fois l’installation terminée, cliquez sur **Fermer** .

## <a name="edit-the-multifactorauthenticationadfsadapterconfig-file"></a>Modifier le fichier MultiFactorAuthenticationAdfsAdapter.config

Procédez comme suit pour modifier le fichier MultiFactorAuthenticationAdfsAdapter.config :

1. Définissez le nœud **UseWebServiceSdk** sur la valeur **true**.  
2. Définissez le champ **WebServiceSdkUrl** sur l’URL du Kit de développement logiciel (SDK) du service web Multi-Factor Authentication. Par exemple, *https:\/\/contoso.com/\<certificatename>/MultiFactorAuthWebServiceSdk/PfWsSdk.asmx*, où *\<certificatename>certificatename>* est le nom de votre certificat.  
3. Modifiez le script Register-MultiFactorAuthenticationAdfsAdapter.ps1 en ajoutant `-ConfigurationFilePath &lt;path&gt;` à la fin de la commande `Register-AdfsAuthenticationProvider`, où *&lt;chemin&gt;* correspond au chemin d’accès complet au fichier MultiFactorAuthenticationAdfsAdapter.config file.

### <a name="configure-the-web-service-sdk-with-a-username-and-password"></a>Configurer le Kit de développement logiciel (SDK) du service web avec un nom d’utilisateur et un mot de passe

Deux options s’offrent à vous pour configurer le Kit de développement logiciel (SDK) du service web : Soit avec un nom d’utilisateur et un mot de passe, soit avec un certificat client. Procédez comme suit pour la première option ou ignorez cette section si vous choisissez la deuxième option.  

1. Définissez le champ **WebServiceSdkUsername** sur un compte membre du groupe de sécurité PhoneFactor Admins. Utilisez le format &lt;domaine&gt;&#92;&lt;nom d’utilisateur&gt;.  
2. Définissez le champ **WebServiceSdkPassword** sur le mot de passe du compte approprié.

### <a name="configure-the-web-service-sdk-with-a-client-certificate"></a>Configurer le Kit de développement logiciel (SDK) du service web avec un certificat client

Si vous ne souhaitez pas utiliser de nom d’utilisateur et de mot de passe, procédez comme suit pour configurer le Kit de développement logiciel (SDK) du service web avec un certificat client.

1. Obtenez un certificat client auprès d’une autorité de certification pour le serveur qui exécute le Kit de développement logiciel (SDK) du service web. Découvrez comment [obtenir des certificats clients](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770328(v=ws.10)).  
2. Importez le certificat client dans le magasin de certificats personnels de l’ordinateur local sur le serveur qui exécute le Kit de développement logiciel (SDK) du service web. Assurez-vous que le certificat public de l’autorité de certification se trouve dans le magasin des certificats racines approuvés.  
3. Exportez les clés publiques et privées du certificat client vers un fichier .pfx.  
4. Exportez la clé publique au format Base64 vers un fichier .cer.  
5. Dans le Gestionnaire de serveur, vérifiez que la fonctionnalité Serveur Web (IIS)\Serveur Web\Sécurité\Authentification par mappage de certificat client IIS est installée. Si elle n’est pas installée, sélectionnez **Ajouter des rôles et fonctionnalités** pour ajouter cette fonctionnalité.  
6. Dans le Gestionnaire IIS, double-cliquez sur **Éditeur de configuration** dans le site web qui contient le répertoire virtuel du Kit de développement logiciel (SDK) du service web. Il est important de sélectionner le site web et non le répertoire virtuel.  
7. Accédez à la section **system.webServer/security/authentication/iisClientCertificateMappingAuthentication** .  
8. Définissez enabled sur **true**.  
9. Définissez oneToOneCertificateMappingsEnabled sur **true**.  
10. Cliquez sur le bouton **...** en regard de oneToOneMappings, puis cliquez sur le lien **Ajouter**.  
11. Ouvrez le fichier .cer au format Base64 que vous avez exporté précédemment. Supprimez *-----BEGIN CERTIFICATE-----* , *-----END CERTIFICATE-----* , ainsi que tous les sauts de ligne. Copiez la chaîne résultante.  
12. Définissez certificate sur la chaîne copiée à l’étape précédente.  
13. Définissez enabled sur **true**.  
14. Définissez userName sur un compte membre du groupe de sécurité PhoneFactor Admins. Utilisez le format &lt;domaine&gt;&#92;&lt;nom d’utilisateur&gt;.  
15. Définissez le mot de passe sur le mot de passe du compte approprié, puis fermez l’Éditeur de configuration.  
16. Cliquez sur le lien **Appliquer** .  
17. Dans le répertoire virtuel du Kit de développement logiciel (SDK) du service web, double-cliquez sur **Authentification**.  
18. Vérifiez que Emprunt d’identité ASP.NET et Authentification de base sont définis sur **Activé** et que tous les autres éléments sont définis sur **Désactivé**.  
19. Dans le répertoire virtuel du Kit de développement logiciel (SDK) du service web, double-cliquez sur **Paramètres SSL**.  
20. Définissez Certificats clients sur **Accepter**, puis cliquez sur **Appliquer**.  
21. Copiez le fichier .pfx exporté précédemment sur le serveur qui exécute l’adaptateur AD FS.  
22. Importez le fichier .pfx dans le magasin de certificats personnels de l’ordinateur local.  
23. Cliquez avec le bouton droit et sélectionnez **Gérer les clés privées**, puis accordez l’accès en lecture au compte que vous avez utilisé pour vous connecter au service AD FS.  
24. Ouvrez le certificat client et copiez l’empreinte à partir de l’onglet **Détails** .  
25. Dans le fichier MultiFactorAuthenticationAdfsAdapter.config, définissez le champ **WebServiceSdkCertificateThumbprint** sur la chaîne copiée à l’étape précédente.  

Enfin, pour enregistrer l’adaptateur, exécutez le script \Program Files\Multi-Factor Authentication Server\Register-MultiFactorAuthenticationAdfsAdapter.ps1 dans PowerShell. L'adaptateur est enregistré en tant que WindowsAzureMultiFactorAuthentication. Redémarrez le service AD FS pour que l’enregistrement soit pris en compte.

## <a name="secure-azure-ad-resources-using-ad-fs"></a>Sécurisation des ressources Azure AD à l’aide d’AD FS

Pour sécuriser vos ressources de cloud, configurez une règle de revendication afin que les services de fédération Active Directory émettent la revendication multipleauthn lorsqu’un utilisateur effectue la vérification en deux étapes avec succès. Cette revendication est transmise à Azure AD. Suivez cette procédure pour les différentes étapes :

1. Ouvrez Gestion AD FS.
2. Sur la gauche, sélectionnez **Approbations de partie de confiance**.
3. Cliquez avec le bouton droit sur **Plateforme d’identité Microsoft Office 365** et sélectionnez **Modifier les règles de revendication…**

   ![Modifier les règles de revendication dans la console ADFS](./media/howto-mfaserver-adfs-2012/trustedip1.png)

4. Sous Règles de transformation d’émission, cliquez sur **Ajouter une règle**.

   ![Modifier les règles de transformation dans la console ADFS](./media/howto-mfaserver-adfs-2012/trustedip2.png)

5. Dans l’Assistant Ajout de règle de revendication de transformation, sélectionnez **Passer ou filtrer une revendication entrante** dans la liste déroulante et cliquez sur **Suivant**.

   ![Assistant Ajout de règle de revendication et de transformation](./media/howto-mfaserver-adfs-2012/trustedip3.png)

6. Nommez votre règle.
7. Sélectionnez **Références des méthodes d’authentification** pour le type de revendication entrante.
8. Sélectionnez **Transférer toutes les valeurs de revendication**.

    ![Assistant Ajouter une règle de revendication de transformation](./media/howto-mfaserver-adfs-2012/configurewizard.png)

9. Cliquez sur **Terminer**. Fermez la console de gestion AD FS.

## <a name="troubleshooting-logs"></a>Journaux d’activité de résolution des problèmes

Pour vous aider à résoudre les problèmes avec l’adaptateur AD FS de serveur de l’authentification multifacteur, utilisez la procédure suivante pour activer la journalisation supplémentaire.

1. Dans l’interface du serveur de l’authentification multifacteur, ouvrez la section AD FS et cochez la case **activer la journalisation**.
2. Sur chaque serveur AD FS, utilisez **regedit.exe** pour créer la clé de Registre de valeur de chaîne `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Positive Networks\PhoneFactor\InstallPath` avec la valeur `C:\Program Files\Multi-Factor Authentication Server\` (ou un autre répertoire de votre choix).  **Notez que la barre oblique de fin est importante.**
3. Créez le répertoire `C:\Program Files\Multi-Factor Authentication Server\Logs` (ou un autre répertoire, tel que mentionné dans l’**étape 2**).
4. Accordez l’accès en modification au répertoire des journaux d’activité au compte de service AD FS.
5. Redémarrez le service AD FS.
6. Vérifiez que le fichier `MultiFactorAuthAdfsAdapter.log` a été créé dans le répertoire des journaux d’activité.

## <a name="related-topics"></a>Rubriques connexes

Pour la résolution des problèmes, consultez le [Forum Aux Questions d’Azure Multi-Factor Authentication](multi-factor-authentication-faq.md)