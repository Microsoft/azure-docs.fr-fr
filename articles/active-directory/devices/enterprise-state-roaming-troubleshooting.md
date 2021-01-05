---
title: Résolution des problèmes rencontrés avec Enterprise State Roaming dans Azure Active Directory
description: Répond à certaines questions que les administrateurs informatiques peuvent se poser sur les paramètres et la synchronisation des données d’application.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 02/12/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: tanning
ms.collection: M365-identity-device-management
ms.openlocfilehash: df70891ef090d44769aadbc235273e3193bc780e
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94837207"
---
# <a name="troubleshooting-enterprise-state-roaming-settings-in-azure-active-directory"></a>Résolution des problèmes de paramètres Enterprise State Roaming dans Azure Active Directory

Cette rubrique fournit des informations sur la résolution et le diagnostic des problèmes liés Enterprise State Roaming, et fournit également une liste de problèmes connus.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!NOTE]
> Cet article s’applique au navigateur HTML hérité Microsoft Edge lancé avec Windows 10 en juillet 2015. L’article ne s’applique pas au nouveau navigateur Microsoft Edge basé sur Chromium publié le 15 janvier 2020. Pour plus d’informations sur le comportement de synchronisation pour le nouveau navigateur Microsoft Edge, consultez l’article [Synchronisation de Microsoft Edge](/deployedge/microsoft-edge-enterprise-sync).

## <a name="preliminary-steps-for-troubleshooting"></a>Étapes préliminaires pour la résolution des problèmes 

Avant de commencer à résoudre les problèmes, vérifiez que l’utilisateur et l’appareil ont été configurés correctement, et que toutes les exigences pour Enterprise State Roaming sont remplies par l’appareil et l’utilisateur. 

1. Windows 10 avec les dernières mises à jour et une version minimale 1511 (build du système d’exploitation 10586 ou ultérieur) sont installés sur l’appareil. 
1. L’appareil est joint à Azure AD ou à Azure AD hybride. Pour plus d’informations, consultez [Comment placer un appareil sous le contrôle d’Azure AD](overview.md).
1. Vérifiez que **Enterprise State Roaming** est activé pour le locataire dans Azure AD, comme décrit dans [Pour activer Enterprise State Roaming](enterprise-state-roaming-enable.md). Vous pouvez activer l’itinérance pour tous les utilisateurs, ou seulement pour un groupe sélectionné d’utilisateurs.
1. Une licence Azure Active Directory Premium est attribuée à l’utilisateur.  
1. L’appareil doit être redémarré et l’utilisateur doit se reconnecter pour accéder aux fonctionnalités Enterprise State Roaming.

## <a name="information-to-include-when-you-need-help"></a>Informations à inclure lorsque vous avez besoin d’aide
Si vous ne pouvez pas résoudre le problème avec les conseils ci-dessous, vous pouvez contacter nos ingénieurs de support. Si vous les contactez, spécifiez les informations suivantes :

* **Description générale de l’erreur**. L'utilisateur remarque-t-il des messages d'erreur ? Si aucun message d’erreur n’est apparu, décrivez en détail le comportement inattendu remarqué. Quelles fonctionnalités sont activées pour la synchronisation et quels éléments l’utilisateur prévoit-il de synchroniser ? Plusieurs fonctionnalités ne synchronisent-elles pas ou une est-elle isolée par rapport à une autre ?
* **Utilisateurs affectés** : la synchronisation fonctionne/échoue-t-elle pour un ou plusieurs utilisateurs ? Combien d’appareils sont concernés par utilisateur ? Tous les appareils ne synchronisent-ils pas ou certains d’entre eux synchronisent-ils et d’autres non ?
* **Informations sur l’utilisateur** : quelle identité l’utilisateur utilise-il pour se connecter à l’appareil ? Comment l’utilisateur se connecte-t-il à l’appareil ? Fait-il partie d’un groupe de sécurité sélectionné autorisé à synchroniser ? 
* **Informations sur l’appareil** : cet appareil est-il joint à Azure AD ou à un domaine ? Quelle build est installée sur l’appareil ? Quelles sont les dernières mises à jour ?
* **Date/heure/fuseau horaire** : quelles étaient la date et l’heure auxquelles l’erreur s’est produite (incluez le fuseau horaire) ?

Ces informations nous aident à résoudre votre problème aussi rapidement que possible.

## <a name="troubleshooting-and-diagnosing-issues"></a>Résolution et diagnostic des problèmes

Cette section propose des suggestions de résolution et de diagnostic des problèmes liés à Enterprise State Roaming.

## <a name="verify-sync-and-the-sync-your-settings-settings-page"></a>Vérifier la synchronisation et la page de paramètres « Synchroniser vos paramètres » 

1. Après avoir joint votre PC Windows 10 à un domaine configuré pour autoriser Enterprise State Roaming, connectez-vous avec votre compte professionnel. Accédez à **Paramètres** > **Comptes** > **Synchroniser vos paramètres** et vérifiez que la synchronisation et les paramètres sont activés, et que le haut de la page de paramètres indique que vous synchronisez avec votre compte professionnel. Vérifiez que le même compte est également utilisé comme compte de connexion dans **Paramètres** > **Comptes** > **Vos informations**. 
1. Vérifiez que la synchronisation fonctionne sur plusieurs ordinateurs en apportant des modifications sur l’ordinateur d’origine, par exemple en déplaçant la barre des tâches vers la droite ou le haut de l’écran. Après cinq minutes, vérifiez que la modification est appliquée sur le deuxième ordinateur. 

   * Le verrouillage et le déverrouillage de l’écran (Win + L) permettent de déclencher une synchronisation.
   * Comme Enterprise State Roaming est lié au compte de l’utilisateur et non pas au compte de la machine, vous devez vous connecter avec le même compte de connexion sur les deux ordinateurs pour que la synchronisation fonctionne.

**Problème potentiel** : si les contrôles de la page **Paramètres** ne sont pas disponibles , vous voyez le message « Certaines fonctionnalités de Windows ne sont disponibles que si vous utilisez un compte Microsoft ou un compte professionnel ». Ce problème peut se produire sur les appareils configurés pour être joints au domaine et inscrits auprès d’Azure AD, quand l’appareil n’a pas été authentifié correctement auprès d’Azure AD. Une cause possible est que la stratégie d’appareil doit être appliquée, mais cette application s’exécute de manière asynchrone et peut être retardée de quelques heures. 

### <a name="verify-the-device-registration-status"></a>Vérifier l’état de l’inscription d’appareils

Enterprise State Roaming exige que l’appareil soit inscrit auprès d’Azure AD. Bien que non spécifique à Enterprise State Roaming, le respect des instructions ci-dessous peut vous aider à vérifier si le client de Windows 10 est inscrit, et à vérifier l’empreinte, l’URL des paramètres Azure AD, l’état NGC et d’autres informations encore.

1. Ouvrez l’invite de commandes non élevée. Pour ce faire, dans Windows, ouvrez le lanceur Exécuter (touche Win + R) et tapez « cmd ».
1. Une fois l’invite de commandes ouverte, tapez « *dsregcmd.exe /status* ».
1. Pour le résultat attendu, la valeur du champ **AzureAdJoined** doit être « YES », la valeur du champ **WamDefaultSet** doit être « YES » et la valeur du champ **WamDefaultGUID** doit être un GUID se terminant par « (AzureAd) ».

**Problème potentiel** : la valeur des champs **WamDefaultSet** et **AzureAdJoined** est « NO », l’appareil était joint au domaine et inscrit auprès d’Azure AD, et l’appareil n’est pas synchronisé. Si ce problème apparaît, il est possible que l’appareil attente l’application de la stratégie ou que l’authentification de l’appareil a échoué lors de la connexion à Azure AD. Il est possible que l’utilisateur doive patienter quelques heures avant que la stratégie ne soit appliquée. D’autres étapes de résolution des problèmes peuvent inclure une nouvelle tentative d’inscription automatique en se déconnectant et en se reconnectant, ou en lançant la tâche dans le Planificateur de tâches. Dans certains cas, l’exécution de «*dsregcmd.exe /leave*» dans une fenêtre d’invite de commandes avec élévation de privilèges, un redémarrage et une nouvelle tentative d’inscription peuvent résoudre ce problème.

**Problème potentiel** : le champ **SettingsUrl** est vide et l’appareil n’est pas synchronisé. L’utilisateur a peut-être ouvert sa dernière session sur l’appareil avant l’activation d’Enterprise State Roaming dans le portail Azure Active Directory. Redémarrez l’appareil et connectez l’utilisateur. Éventuellement, dans le portail, demandez à l’administrateur informatique d’accéder à **Azure Active Directory** > **Appareils** > **Enterprise State Roaming** et de désactiver et réactiver **Les utilisateurs peuvent synchroniser les paramètres et les données d'application sur différents appareils**. Une fois l’option réactivée, redémarrez l’appareil et connectez l’utilisateur. Si cela ne résout pas le problème, **SettingsUrl** peut être vide si le certificat de l’appareil est erroné. Dans ce cas, l’exécution de «*dsregcmd.exe /leave*» dans une fenêtre d’invite de commandes avec élévation de privilèges, un redémarrage et une nouvelle tentative d’inscription peuvent résoudre ce problème.

## <a name="enterprise-state-roaming-and-multi-factor-authentication"></a>Enterprise State Roaming et authentification multifacteur 

Il peut arriver qu’Enterprise State Roaming ne parvienne pas à synchroniser les données si Azure AD Multi-Factor Authentication est configuré. Pour plus d’informations sur ces symptômes, consultez le document de support [KB3193683](https://support.microsoft.com/kb/3193683). 

**Problème potentiel** : si votre appareil est configuré pour requérir l’authentification multifacteur sur le portail Azure Active Directory, la synchronisation des paramètres lors de la connexion à un appareil Windows 10 à l’aide d’un mot de passe peut échouer. Ce type de configuration de l’authentification multifacteur a pour but de protéger un compte d’administrateur Windows Azure. Les utilisateurs administrateurs peuvent cependant continuer de synchroniser les paramètres en se connectant à leurs appareils Windows 10 à l’aide du code confidentiel Microsoft Passport for Work ou en effectuant une authentification multifacteur lors de l’accès à d’autres services Azure comme Microsoft 365.

**Problème potentiel** : la synchronisation peut échouer si l’administrateur configure la stratégie d’accès conditionnel Multi-Factor Authentication des services de fédération Active Directory (AD FS) et que le jeton d’accès de l’appareil arrive à expiration. Veillez à vous connecter et à vous déconnecter en utilisant le code confidentiel Microsoft Passport for Work ou à effectuer une authentification multifacteur lorsque vous accédez à d’autres services Azure comme Microsoft 365.

### <a name="event-viewer"></a>Observateur d'événements

Pour une résolution des problèmes avancée, l’Observateur d’événements peut être utilisé pour rechercher des erreurs spécifiques. Celles-ci sont décrites dans le tableau ci-dessous. Les événements se trouvent sous Observateur d’événements > **Journaux des applications et des services** > **Microsoft** > **Windows** > **SettingSync-Azure** et, pour les problèmes liés à l’identité, avec la synchronisation **Journaux des applications et des services** > **Microsoft** > **Windows** > **AAD**.

## <a name="known-issues"></a>Problèmes connus

### <a name="sync-does-not-work-on-devices-that-have-apps-side-loaded-using-mdm-software"></a>La synchronisation ne fonctionne pas sur les appareils contenant des applications chargées à l’aide de logiciels de gestion des appareils mobiles

Concerne les appareils exécutant la mise à jour anniversaire de Windows 10 (version 1607). Dans l’Observateur d’événements, dans les journaux d’activité SettingSync-Azure, l’ID d’événement 6013 avec l’erreur 80070259 apparaît fréquemment.

**Action recommandée**  
Assurez-vous que le client Windows 10 v1607 inclut la mise à jour cumulative du 23 août 2016 ([KB3176934](https://support.microsoft.com/kb/3176934) version du système d’exploitation 14393.82). 

---

### <a name="internet-explorer-favorites-do-not-sync"></a>Les favoris Internet Explorer ne sont pas synchronisés

Concerne les appareils exécutant la mise à jour de Windows 10 de novembre (version 1511).

**Action recommandée**  
Assurez-vous que le client Windows 10 v1511 inclut la mise à jour cumulative de juillet 2016 ([KB3172985](https://support.microsoft.com/kb/3172985) version du système d’exploitation 10586.494).

---

### <a name="theme-is-not-syncing-as-well-as-data-protected-with-windows-information-protection"></a>Le thème n’est pas synchronisé, ainsi que les données protégées avec Windows Information Protection 

Pour empêcher les fuites de données, les données protégées avec [Windows Information Protection](/windows/security/information-protection/windows-information-protection/protect-enterprise-data-using-wip) ne sont pas synchronisées via Enterprise State Roaming pour les appareils utilisant la mise à jour anniversaire de Windows 10.

**Action recommandée**  
Aucun. Les mises à jour futures de Windows résoudront peut-être ce problème.

---

### <a name="date-time-and-region-settings-do-not-sync-on-domain-joined-device"></a>Les paramètres de date, d’heure et région ne sont pas synchronisés sur un appareil joint au domaine 
  
Les appareils joints au domaine ne synchronisent pas les paramètres de date, d’heure et de région : l’heure est automatique. L’utilisation de l’heure automatique peut remplacer les autres paramètres de date, d’heure et de région, et entraîner la non-synchronisation de ces paramètres. 

**Action recommandée**  
Aucun. 

---

### <a name="uac-prompts-when-syncing-passwords"></a>Invites UAC lors de la synchronisation des mots de passe

Concerne les appareils exécutant la mise à jour de Windows 10 de novembre (version 1511) avec une carte réseau sans fil qui est configurée pour synchroniser les mots de passe.

**Action recommandée**  
Assurez-vous que le client Windows 10 v1511 inclut la mise à jour cumulative ([KB3140743](https://support.microsoft.com/kb/3140743) version du système d’exploitation 10586.494).

---

### <a name="sync-does-not-work-on-devices-that-use-smart-card-for-login"></a>La synchronisation ne fonctionne pas sur les appareils qui utilisent des cartes à puce pour la connexion

Si vous essayez de vous connecter à votre appareil Windows à l’aide d’une carte à puce physique ou virtuelle, la synchronisation des paramètres cessera de fonctionner.     

**Action recommandée**  
Aucun. Les mises à jour futures de Windows résoudront peut-être ce problème.

---

### <a name="domain-joined-device-is-not-syncing-after-leaving-corporate-network"></a>Un appareil joint au domaine n’est pas synchronisé après avoir quitté le réseau d’entreprise     

Les appareils joints au domaine inscrits auprès d’Azure AD peuvent rencontrer un échec de synchronisation si l’appareil est hors site pendant des périodes prolongées et l’authentification de domaine ne peut pas terminer.

**Action recommandée**  
Connectez l’appareil à un réseau d’entreprise afin que la synchronisation puisse reprendre.

---

### <a name="azure-ad-joined-device-is-not-syncing-and-the-user-has-a-mixed-case-user-principal-name"></a>L’appareil joint à Azure AD ne se synchronise pas et l’utilisateur possède un nom d’utilisateur principal à casse mixte.

Si l’utilisateur possède un nom d’utilisateur principal à casse mixte (par exemple, Nom d’Utilisateur au lieu de nom d’utilisateur) et qu’il figure sur un appareil joint à Azure AD qui a été mis à niveau de Windows 10 Build 10586 vers 14393, l’appareil de l’utilisateur risque de ne pas se synchroniser. 

**Action recommandée**  
L’utilisateur devra quitter l’annuaire puis reconnectez l’appareil au cloud. Pour ce faire, connectez-vous comme l’utilisateur Administrateur local et déconnectez l’appareil en choisissant **Paramètres** > **Système** > **A propos de**, puis en sélectionnant « Gérer ou se déconnecter de l’entreprise ou de l’école ». Nettoyez les fichiers ci-dessous, puis joignez de nouveau l’appareil à Azure AD en choisissant **Paramètres** > **Système** > **A propos de**, puis en sélectionnant « Connecter à l’entreprise ou l’école ». Continuez à joindre l’appareil à Azure Active Directory et terminez le flux.

Dans l’étape de nettoyage, nettoyez les fichiers suivants :
- Settings.dat dans `C:\Users\<Username>\AppData\Local\Packages\Microsoft.AAD.BrokerPlugin_cw5n1h2txyewy\Settings\`
- Tous les fichiers du dossier`C:\Users\<Username>\AppData\Local\Packages\Microsoft.AAD.BrokerPlugin_cw5n1h2txyewy\AC\TokenBroker\Account`

---

### <a name="event-id-6065-80070533-this-user-cant-sign-in-because-this-account-is-currently-disabled"></a>ID d’événement 6065 : 80070533 Cet utilisateur ne peut pas se connecter car ce compte est actuellement désactivé  

Dans l’Observateur d’événements, dans les journaux d’activité SettingSync/Debug, cette erreur peut apparaître lorsque les informations d’identification de l’utilisateur ont expiré. Par ailleurs, cette erreur peut apparaître lorsque le locataire n’a pas automatiquement configuré AzureRMS. 

**Action recommandée**  
Dans le premier cas, demandez à l’utilisateur de mettre à jour ses informations d’identification et de se connecter à l’appareil avec les nouvelles informations d’identification. Pour résoudre le problème AzureRMS, effectuez les étapes répertoriées dans [KB3193791](https://support.microsoft.com/kb/3193791). 

---

### <a name="event-id-1098-error-0xcaa5001c-token-broker-operation-failed"></a>ID d’événement 1098 : Erreur : Échec de l’opération de service Broker de jeton 0xCAA5001C  

Dans l’Observateur d’événements, dans les journaux d’activité AAD/Operational, cette erreur peut apparaître avec l’événement 1104 : le jeton Get de l’appel du plug-in AAD Cloud AP a renvoyé une erreur : 0xC000005F. Ce problème se produit si des autorisations ou des attributs de propriété sont manquants.  

**Action recommandée**  
Effectuez les étapes répertoriées dans [KB3196528](https://support.microsoft.com/kb/3196528).  

## <a name="next-steps"></a>Étapes suivantes

Pour une vue d’ensemble, consultez [Vue d’ensemble d’Enterprise State Roaming](enterprise-state-roaming-overview.md).