---
title: FAQ de la réinitialisation de mot de passe libre-service - Azure Active Directory
description: Forum aux questions concernant la réinitialisation du mot de passe en libre-service Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/20/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0874f6aeb74b028e92e10d972757cc5971735e39
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744310"
---
# <a name="self-service-password-reset-frequently-asked-questions"></a>Foire aux questions concernant la réinitialisation du mot de passe en libre-service

Voici quelques questions fréquemment posées (FAQ) sur tout ce qui concerne la réinitialisation de mot de passe en libre-service.

Si vous avez une question d’ordre général sur Azure Active Directory (Azure AD) et la réinitialisation du mot de passe en libre-service qui ne trouve pas de réponse ici, vous pouvez demander de l’aide à la communauté sur la [Page de questions Microsoft Q&A pour Azure Active Directory](/answers/topics/azure-active-directory.html). Les membres de la communauté comprennent des ingénieurs, des chefs de produit, MVP et autres professionnels de l’informatique.

Ce Forum Aux Questions est organisé de la manière suivante :

* [Questions relatives à l’inscription à la réinitialisation du mot de passe](#password-reset-registration)
* [Questions relatives à la réinitialisation du mot de passe](#password-reset)
* [Questions relatives au changement du mot de passe](#password-change)
* [Questions relatives aux rapports de gestion des mots de passe](#password-management-reports)
* [Questions relatives à la réécriture du mot de passe](#password-writeback)

## <a name="password-reset-registration"></a>Inscription à la réinitialisation de mot de passe

* **Q :  Mes utilisateurs peuvent-ils inscrire leurs propres données de réinitialisation du mot de passe ?**

  > **R :** Oui. Tant que la réinitialisation de mot de passe est activée et qu’ils disposent d’une licence, vos utilisateurs peuvent accéder au portail d’inscription à la réinitialisation de mot de passe (https://aka.ms/ssprsetup) pour inscrire leurs informations d’authentification. Ils peuvent également s’inscrire par le biais du volet d’accès (https://myapps.microsoft.com). Pour s’inscrire par le biais du volet d’accès, ils doivent sélectionner l’image de leur profil, sélectionner **Profil**, puis sélectionner l’option **Réinitialiser mon mot de passe**.
  >
  > Si vous activez l'[inscription combinée](concept-registration-mfa-sspr-combined.md), les utilisateurs peuvent s'inscrire en même temps à SSPR et Azure AD Multi-Factor Authentication.
* **Q :  Si j’active la réinitialisation du mot de passe pour un groupe, puis décide de l’activer pour tout le monde, mes utilisateurs doivent-ils se réenregistrer ?**

  > **R :** Non. Les utilisateurs possédant des données d’authentification renseignées ne doivent pas s’enregistrer de nouveau.
  >
  >
* **Q :  Puis-je définir des données de réinitialisation de mot de passe pour le compte de mes utilisateurs ?**

  > **R :** Oui, vous pouvez le faire avec Azure AD Connect, PowerShell, le [portail Azure](https://portal.azure.com) ou le [Centre d’administration Microsoft 365](https://admin.microsoft.com). Pour plus d’informations, consultez [Données utilisées par la réinitialisation du mot de passe en libre-service Azure AD](howto-sspr-authenticationdata.md).
  >
  >
* **Q :  Puis-je synchroniser localement des données pour des questions de sécurité ?**

  > **R :** Non, ce n’est actuellement pas possible.
  >
  >
* **Q :  Mes utilisateurs peuvent-ils inscrire des données de manière à ce que les autres utilisateurs ne puissent pas les voir ?**

  > **R :** Oui. Quand les utilisateurs inscrivent des données à l’aide du portail d’inscription à la réinitialisation de mot de passe, les données sont enregistrées dans des champs d’authentification privés uniquement visibles par les administrateurs généraux et l’utilisateur.
  >
  >
* **Q :  Mes utilisateurs doivent-ils être inscrits avant de pouvoir utiliser la réinitialisation de mot de passe ?**

  > **R :** Non. Si vous définissez suffisamment d’informations d’authentification en leur nom, les utilisateurs n’ont pas besoin de s’inscrire. La réinitialisation de mot de passe fonctionne tant que vous disposez de données correctement formatées stockées dans les champs appropriés de l’annuaire.
  >
  >
* **Q :  Puis-je synchroniser ou définir les champs Téléphone d’authentification, E-mail d’authentification ou Autre téléphone d’authentification pour le compte de mes utilisateurs ?**

  > **R :** Les champs qui peuvent être définis par un administrateur global sont décrits dans l’article [Spécifications des données SSPR](howto-sspr-authenticationdata.md).
  >
  >
* **Q :  Comment le portail d’inscription détermine-t-il quelles options proposer à mes utilisateurs ?**

  > **R :** Le portail d’inscription de réinitialisation de mot de passe affiche uniquement les options que vous avez activées pour vos utilisateurs. Ces options se trouvent sous la section **Stratégie de réinitialisation de mot de passe utilisateur** de l'onglet **Configurer** de votre annuaire. Par exemple, si vous n’activez pas les questions de sécurité, les utilisateurs ne peuvent pas s’inscrire pour cette option.
  >
  >
* **Q :  Quand un utilisateur est-il considéré comme inscrit ?**

  > **R :** Un utilisateur est considéré comme inscrit pour la réinitialisation de mot de passe en libre-service quand il a enregistré au moins le **Nombre de méthodes requises pour la réinitialisation** d’un mot de passe, que vous avez défini dans le [portail Azure](https://portal.azure.com).
  >
  >

## <a name="password-reset"></a>Réinitialisation du mot de passe

* **Q :  Empêchez-vous les utilisateurs d’effectuer plusieurs tentatives de réinitialisation de mot de passe dans un bref laps de temps ?**

  > **R :** Oui, il existe des fonctionnalités de sécurité intégrées à la réinitialisation de mot de passe pour empêcher toute utilisation malveillante. 
  >
  > Les utilisateurs peuvent faire cinq tentatives d’entrée du mot de passe seulement dans une période de 24 heures, avant qu’elle ne soit verrouillée pendant 24 heures. 
  >
  > Les utilisateurs peuvent tenter de valider un numéro de téléphone, envoyer un SMS ou valider des questions et réponses de sécurité cinq fois seulement dans un délai d’une heure avant que la validation ne soit verrouillée pendant 24 heures. 
  >
  > Les utilisateurs peuvent envoyer un e-mail 10 fois maximum dans un délai de 10 minutes avant que l’envoi ne soit verrouillé pendant 24 heures.
  >
  > Les compteurs sont réinitialisés dès que l’utilisateur réinitialise son mot de passe.
  >
  >
* **Q :  Q : Combien de temps dois-je attendre avant de recevoir un message électronique, un SMS ou un appel téléphonique de la réinitialisation du mot de passe ?**

  > **R :** Les e-mails, messages SMS et appels téléphoniques doivent arriver en moins d’une minute. Le cas normal est de 5 à 20 secondes.
  > Si vous ne recevez pas la notification dans ce laps de temps :
  > * Vérifiez votre dossier de courrier indésirable.
  > * Vérifiez que le numéro ou que l’e-mail contacté est celui que vous attendez.
  > * Vérifiez que les données d’authentification dans l’annuaire sont mises en forme correctement, par exemple +1 4255551234 ou *user\@contoso.com*. 
* **Q :  Quelles langues sont prises en charge pour la réinitialisation de mot de passe ?**

  > **R :** L’interface utilisateur pour la réinitialisation du mot de passe, les SMS et les appels vocaux sont localisés dans les mêmes langues que celles prises en charge dans Microsoft 365.
  >
  >
* **Q :  Quelles parties de l'expérience de réinitialisation de mot de passe sont personnalisées lorsque je configure des éléments de personnalisation de l'organisation sous l'onglet Configurer de mon annuaire ?**

  > **R :** Le portail de réinitialisation de mot de passe affiche le logo de votre organisation et vous permet également de configurer le lien « Contactez votre administrateur » pour qu’il pointe vers une URL ou une adresse e-mail personnalisée. Tout e-mail envoyé par la fonctionnalité de réinitialisation de mot de passe inclut le logo, les couleurs et le nom de votre organisation dans le corps de l'e-mail, et il est personnalisé à partir des paramètres de ce nom spécifique.
  >
  >
* **Q :  Comment puis-je informer mes utilisateurs des liens sur lesquels ils peuvent cliquer pour réinitialiser leur mot de passe ?**

  > **R :** Essayez certaines des suggestions de notre [article sur le déploiement de la réinitialisation de mot de passe](howto-sspr-deployment.md#plan-communications).
  >
  >
* **Q :  Puis-je utiliser cette page à partir d’un appareil mobile ?**

  > **R :** Oui, cette page fonctionne sur les appareils mobiles.
  >
  >
* **Q :  Prenez-vous en charge le déverrouillage des comptes Active Directory locaux quand les utilisateurs réinitialisent leur mot de passe ?**

  > **R :** Oui. Si la réécriture du mot de passe a été déployée par le biais d'Azure AD Connect, le compte d'utilisateur est automatiquement déverrouillé au moment où l'utilisateur réinitialise son mot de passe.
  >
  >
* **Q :  Comment puis-je intégrer la réinitialisation de mot de passe directement dans l'expérience de connexion de Bureau de mes utilisateurs ?**

  > **R :** Si vous êtes client Azure AD Premium, vous pouvez installer Microsoft Identity Manager sans coût supplémentaire et déployer la solution locale de réinitialisation de mot de passe.
  >
  >
* **Q :  Puis-je définir des questions de sécurité différentes pour différents paramètres régionaux ?**

  > **R :** Non, ce n’est actuellement pas possible.
  >
  >
* **Q :  Combien de questions puis-je configurer pour l’option d’authentification Questions de sécurité ?**

  > **R :** Vous pouvez configurer jusqu’à 20 questions de sécurité personnalisées dans le [portail Azure](https://portal.azure.com).
  >
  >
* **Q :  Quelle peut être la longueur des questions de sécurité ?**

  > **R :** Les questions de sécurité peuvent comporter entre 3 et 200 caractères.
  >
  >
* **Q :  Quelle peut être la longueur des réponses aux questions de sécurité ?**

  > **R :** Les réponses peuvent comprendre entre 3 et 40 caractères.
  >
  >
* **Q :  Les réponses dupliquées aux questions de sécurité sont-elles rejetées ?**

  > **R :** Oui, nous rejetons les réponses dupliquées aux questions de sécurité.
  >
  >
* **Q :  Un utilisateur peut-il inscrire plusieurs fois la même question de sécurité ?**

  > **R :** Non. Une fois qu’un utilisateur a inscrit une question particulière, il ne peut plus inscrire cette même question une deuxième fois.
  >
  >
* **Q :  Est-il possible de définir un nombre minimal de questions de sécurité pour l’inscription et la réinitialisation ?**

  > **R :** Oui, vous pouvez définir une limite pour l’inscription et une autre pour la réinitialisation. De trois à cinq questions de sécurité peuvent être requises pour l’inscription, et de trois à cinq questions peuvent être requises pour la réinitialisation.
  >
  >
* **Q :  J’ai configuré ma stratégie de sorte qu’elle demande aux utilisateurs d’utiliser les questions de sécurité à des fins de réinitialisation, mais les administrateurs Azure semblent configurés différemment.**

  > **R :** C’est le comportement normal. Microsoft applique par défaut une stratégie de réinitialisation de mot de passe fort à deux verrous pour tous les rôles d’administrateur Azure. Cela empêche les administrateurs d’utiliser les questions de sécurité. Vous trouverez plus d’informations sur cette stratégie dans l’article [Stratégies de mot de passe et restrictions dans Azure Active Directory](concept-sspr-policy.md).
  >
  >
* **Q :  Si un utilisateur a inscrit plus de questions que le nombre maximal demandé pour la réinitialisation, comment les questions de sécurité sont-elles sélectionnées lors de la réinitialisation ?**

  > **R :** *N* questions de sécurité sont sélectionnées au hasard parmi le nombre total de questions qu’un utilisateur a inscrit, *N* correspondant à l’option **Nombre de questions requises pour la réinitialisation** définie. Par exemple, si un utilisateur a inscrit cinq questions de sécurité, mais que seules trois sont requises pour réinitialiser un mot de passe, trois des cinq questions sont sélectionnées au hasard et sont présentées lors de la réinitialisation. Pour éviter de poser toujours les mêmes questions, si l’utilisateur ne répond pas correctement aux questions, le processus de sélection recommence.
  >
  >
* **Q :  Quelle est la durée de validité des codes secrets à usage unique pour les e-mails et les SMS ?**

  > **R :** La durée de vie de session pour la réinitialisation du mot de passe est de 15 minutes. À compter du début de l’opération de réinitialisation du mot de passe, l’utilisateur dispose de 15 minutes pour le réinitialiser. Le code secret à usage unique pour les e-mails et les SMS est valide pendant 5 minutes lors de la session de réinitialisation du mot de passe.
  >
  >
* **Q :  Puis-je empêcher des utilisateurs de réinitialiser leur mot de passe ?**

  > **R :** Oui, si vous utilisez un groupe pour activer la réinitialisation du mot de passe en libre-service, vous pouvez supprimer un utilisateur de ce groupe qui permet aux utilisateurs de réinitialiser leur mot de passe. Si l’utilisateur est administrateur général, il conserve la possibilité de réinitialiser son mot de passe, et cette fonctionnalité ne peut pas être désactivée.
  >
  >

## <a name="password-change"></a>Modification de mot de passe

* **Q :  Où mes utilisateurs doivent-ils aller pour modifier leur mot de passe ?**

  > **R :** Les utilisateurs peuvent changer leur mot de passe partout où ils voient leur image ou leur icône de profil, par exemple en haut à droite de leur portail [Office 365](https://portal.office.com) ou dans le [Volet d’accès](https://myapps.microsoft.com). Les utilisateurs peuvent changer leurs mots de passe à partir de la [page Profil du Panneau d’accès](https://account.activedirectory.windowsazure.com/r#/profile). Ils peuvent également être invités à changer automatiquement leurs mots de passe dans la page de connexion Azure AD si leurs mots de passe ont expiré. Pour finir, les utilisateurs peuvent accéder directement au [portail de changement de mot de passe Azure AD](https://account.activedirectory.windowsazure.com/ChangePassword.aspx) s’ils souhaitent changer leur mot de passe.
  >
  >
* **Q :  Mes utilisateurs peuvent-ils être informés de l’expiration de leur mot de passe local dans le portail Office ?**

  > **R :** Oui, c’est possible aujourd’hui si vous utilisez les services de fédération Active Directory (AD FS). Si vous utilisez les services AD FS, suivez les instructions de l’article [Sending password policy claims with AD FS (Envoi de revendications de stratégie de mot de passe avec les services AD FS)](/windows-server/identity/ad-fs/operations/configure-ad-fs-to-send-password-expiry-claims?f=255&MSPPError=-2147217396). Si vous utilisez la synchronisation de hachage de mot de passe, ceci n’est pas possible actuellement. Comme nous ne synchronisons pas les stratégies de mot de passe à partir des annuaires locaux, nous pouvons pas publier de notifications d’expiration dans les expériences cloud. Dans les deux cas, il est également possible de [notifier les utilisateurs dont les mots de passe sont sur le point d’expirer par le biais de PowerShell](https://social.technet.microsoft.com/wiki/contents/articles/23313.notify-active-directory-users-about-password-expiry-using-powershell.aspx).
  >
  >
* **Q :  Puis-je empêcher des utilisateurs de modifier leur mot de passe ?**

  > **R :** Pour les utilisateurs du cloud uniquement, les modifications de mot de passe ne peuvent pas être bloquées. Pour les utilisateurs locaux, vous pouvez activer l’option **L’utilisateur ne peut pas changer de mot de passe**. Les utilisateurs sélectionnés ne peuvent pas modifier leur mot de passe.
  >
  >

## <a name="password-management-reports"></a>Rapports sur la gestion des mots de passe

* **Q :  Combien de temps faut-il pour que les données soient affichées dans les rapports de gestion des mots de passe ?**

  > **R :** Les données doivent apparaître dans les rapports de gestion des mots de passe dans les cinq à dix minutes. Dans certains cas, il faut attendre une heure avant qu’elles n’apparaissent.
  >
  >
* **Q :  Comment faire pour filtrer les rapports de gestion des mots de passe ?**

  > **R :** Pour filtrer les rapports de gestion des mots de passe, sélectionnez la petite loupe qui se trouve à l’extrême droite des étiquettes de colonnes, pratiquement en haut du rapport. Pour effectuer un filtrage plus précis, vous pouvez télécharger le rapport dans Excel et créer un tableau croisé dynamique.
  >
  >
* **Q : Quel est le nombre maximal d’événements stockés dans les rapports de gestion de mot de passe ?**

  > **R :** Jusqu’à 75 000 événements de réinitialisation de mot de passe ou d’inscription à la réinitialisation de mot de passe sont stockés dans les rapports de gestion de mot de passe, pour une sauvegarde d’une durée de 30 jours. Nous travaillons à l’augmentation de ce nombre pour inclure davantage d’événements.
  >
  >
* **Q :  Jusqu’à quand remontent les rapports de gestion des mots de passe ?**

  > **R :** Les rapports de gestion des mots de passe affichent les opérations qui ont eu lieu durant les 30 derniers jours. Pour le moment, si vous devez archiver ces données, vous pouvez télécharger les rapports régulièrement et les enregistrer dans un emplacement distinct.
  >
  >
* **Q :  Existe-t-il une limite maximale au nombre de lignes qui peuvent apparaître dans les rapports de gestion des mots de passe ?**

  > **R :** Oui. 75 000 lignes au maximum peuvent apparaître sur un rapport de gestion des mots de passe, qu’elles soient téléchargées ou affichées dans l’interface utilisateur.
  >
  >
* **Q :  Existe-t-il une API pour accéder aux données des rapports sur la réinitialisation de mot de passe ou sur l’inscription à la réinitialisation de mot de passe ?**

  > **R :** Oui. Pour savoir comment accéder aux données des rapports sur la réinitialisation de mot de passe, consultez [Informations de référence sur l’API REST Azure Log Analytics](/rest/api/loganalytics/).
  >
  >

## <a name="password-writeback"></a>Réécriture du mot de passe

* **Q :  Comment la réécriture du mot de passe fonctionne-t-elle en arrière-plan ?**

  > **R :** Consultez l’article [Fonctionnement de la réécriture du mot de passe](./tutorial-enable-sspr-writeback.md) pour obtenir une explication de ce qui se passe quand vous activez la réécriture du mot de passe et pour comprendre la manière dont les données circulent entre le système et votre environnement local.
  >
  >
* **Q :  Combien de temps faut-il pour que la réécriture du mot de passe fonctionne ? Y a-t-il un délai de synchronisation comme avec la synchronisation du hachage de mot de passe ?**

  > **R :** La réécriture du mot de passe est instantanée. Il s’agit d’un pipeline synchrone qui fonctionne différemment de la synchronisation du hachage de mot de passe. L’écriture différée du mot de passe permet aux utilisateurs d’obtenir des commentaires en temps réel quant à la réussite de la modification ou de la réinitialisation de leur mot de passe. L’écriture différée réussie d’un mot de passe dure en moyenne moins de 500 ms.
  >
  >
* **Q :  Si mon compte local est désactivé, comment mon accès au cloud et mon compte cloud sont-ils affectés ?**

  > **R :** Si votre ID local est désactivé, votre ID cloud et votre accès au cloud seront également désactivés lors du prochain intervalle de synchronisation par le biais d’Azure AD Connect. Par défaut, cette synchronisation a lieu toutes les 30 minutes.
  >
  >
* **Q :  Si mon compte local est contraint par une stratégie de mot de passe Active Directory locale, la réinitialisation de mot de passe en libre-service respecte-t-elle cette stratégie quand je modifie mon mot de passe ?**

  > **R :** Oui, la réinitialisation de mot de passe en libre-service s’appuie sur la stratégie de mot de passe Active Directory locale et la respecte. Cette stratégie inclut la stratégie de mot de passe de domaine Active Directory par défaut, ainsi que les stratégies de mot de passe affinées et définies qui ciblent un utilisateur.
  >
  >
* **Q :  Pour quels types de comptes la réécriture du mot de passe fonctionne-t-elle ?**

  > **R :** La réécriture du mot de passe fonctionne pour les comptes d’utilisateur synchronisés entre Active Directory en local et Azure AD, y compris les utilisateurs fédérés, synchronisés avec hachage de mot de passe et à authentification directe.
  >
  >
* **Q :  La réécriture du mot de passe applique-t-elle les stratégies de mot de passe de mon domaine ?**

  > **R :** Oui. La réécriture du mot de passe applique l’âge du mot de passe, l’historique, la complexité, les filtres et toute autre restriction que vous pouvez mettre en place sur les mots de passe dans votre domaine local.
  >
  >
* **Q :  La réécriture du mot de passe est-elle sécurisée ?  Comment puis-je être sûr que je ne me ferai pas pirater ?**

  > **R :** Oui, la réécriture du mot de passe est sécurisée. Pour en savoir plus sur les multiples couches de sécurité implémentées par le service de réécriture du mot de passe, consultez la section [Sécurité de réécriture du mot de passe](concept-sspr-writeback.md#password-writeback-security) dans l’article [Vue d’ensemble de la réécriture du mot de passe](./tutorial-enable-sspr-writeback.md).
  >
  >

## <a name="next-steps"></a>Étapes suivantes

* [Comment réussir le lancement de la réinitialisation de mot de passe en libre-service ?](howto-sspr-deployment.md)
* [Réinitialiser ou modifier votre mot de passe](../user-help/active-directory-passwords-update-your-own-password.md)
* [S’inscrire pour la réinitialisation du mot de passe en libre-service](../user-help/active-directory-passwords-reset-register.md)
* [Vous avez une question relative à la licence ?](concept-sspr-licensing.md)
* [Quelles données sont utilisées par la réinitialisation de mot de passe en libre-service et quelles données vous devez renseigner pour vos utilisateurs ?](howto-sspr-authenticationdata.md)
* [Quelles méthodes d'authentification sont accessibles aux utilisateurs ?](concept-sspr-howitworks.md#authentication-methods)
* [Quelles sont les options de stratégie disponibles avec la réinitialisation de mot de passe en libre-service ?](concept-sspr-policy.md)
* [Quelle est l’écriture différée de mot de passe et pourquoi dois-je m’y intéresser ?](./tutorial-enable-sspr-writeback.md)
* [Comment puis-je générer des rapports sur l’activité dans la réinitialisation de mot de passe en libre-service ?](howto-sspr-reporting.md)
* [Quelles sont toutes les options disponibles dans la réinitialisation de mot de passe en libre-service et que signifient-elles ?](concept-sspr-howitworks.md)
* [Je pense qu’il y a une panne quelque part. Comment puis-je résoudre les problèmes de la réinitialisation de mot de passe en libre-service ?](./troubleshoot-sspr.md)