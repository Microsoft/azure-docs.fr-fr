---
title: Questions et réponses sur l’application Microsoft Authenticator – Azure AD
description: Forum aux questions (FAQ) sur l’application Microsoft Authenticator et la vérification à deux facteurs.
services: active-directory
author: curtand
manager: daveba
ms.assetid: f04d5bce-e99e-4f75-82d1-ef6369be3402
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 12/09/2020
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: a1307aa19c66e3f6fcb71f677e6c67551d2a01f9
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928322"
---
# <a name="frequently-asked-questions-faq-about-the-microsoft-authenticator-app"></a>Forum aux questions (FAQ) sur l’application Microsoft Authenticator

Cet article répond aux questions courantes sur l’application Microsoft Authenticator. Si vous ne trouvez pas la réponse à votre question, consultez le [Forum de l’application Microsoft Authenticator](https://social.technet.microsoft.com/Forums/en-US/home?forum=MicrosoftAuthenticatorApp).

L’application Microsoft Authenticator a remplacé l’application Azure Authenticator. Elle est désormais l’application recommandée lorsque vous utilisez Azure AD Multi-Factor Authentication. L’application Microsoft Authenticator est disponible pour [Android](https://app.adjust.com/e3rxkc_7lfdtm?fallback=https%3A%2F%2Fplay.google.com%2Fstore%2Fapps%2Fdetails%3Fid%3Dcom.azure.authenticator) et [iOS](https://app.adjust.com/e3rxkc_7lfdtm?fallback=https%3A%2F%2Fitunes.apple.com%2Fus%2Fapp%2Fmicrosoft-authenticator%2Fid983156458).

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="registering-a-device"></a>Inscription d’un appareil

**Q** : Le fait d’inscrire un appareil signifie-t-il que l’entreprise ou le service obtiennent l’accès à cet appareil ?

**R** : L’inscription d’un appareil permet à celui-ci d’accéder aux services de votre organisation, mais ne permet pas à votre organisation d’accéder à l’appareil.

## <a name="too-many-app-permissions"></a>Trop d’autorisations d’application

**Q** : Pourquoi l’application demande-t-elle autant d’autorisations ?

**R** : Voici la liste complète des autorisations qui peuvent être demandées ainsi que la façon dont elles sont utilisées par l’application. Les autorisations présentées dépendent du type de téléphone que vous utilisez.

- **Emplacement**. Parfois, votre organisation souhaite connaître votre emplacement avant de vous permettre d’accéder à certaines ressources. L’application demande cette autorisation uniquement si votre organisation a mis en place une stratégie nécessitant un emplacement. 

### <a name="legacy-apns-support-deprecated"></a>Prise en charge héritée d’APNs déconseillée

**Q** : Étant donné que l’interface binaire héritée pour Apple Push Notification Service est déconseillée en novembre 2020, comment puis-je continuer à utiliser Microsoft Authenticator/PhoneFactor pour la connexion ?

**R** : [Apple a annoncé la dépréciation](https://developer.apple.com/news/?id=11042019a) de notifications Push qui utilisent son interface binaire pour les appareils iOS, tels que celles utilisées par PhoneFactor. Pour continuer à recevoir des notifications Push, nous recommandons aux utilisateurs de mettre à jour leur application Authenticator vers sa version la plus récente. En attendant, vous pouvez contourner ce problème en recherchant manuellement les notifications dans l’application Authenticator.

### <a name="app-lock-feature"></a>Fonctionnalité Verrou d’application

**Q** : Qu’est-ce que le verrou d’application et comment puis-je l’utiliser pour mieux sécuriser le stockage ?

**R** : La fonctionnalité Verrou d’application sécurise les codes de vérification à usage unique, les informations et les paramètres d’application. Si vous activez le Verrou d’application, vous êtes invité à vous authentifier à l’aide du code PIN ou verrou biométrique de votre machine chaque fois que vous ouvrez l’Authenticator. Le Verrou d’application permet également de s’assurer que vous êtes le seul habilité à approuver les notifications en demandant votre code confidentiel ou biométrique à chaque fois que vous approuvez une notification de connexion. Vous pouvez activer ou désactiver le Verrou d’application sur la page Paramètres de l’Authentificateur. Par défaut, le Verrou d’application est activé lorsque vous configurez un code PIN ou biométrique sur votre appareil.<br><br>Malheureusement, il n’est nullement garanti que la fonctionnalité Verrou d’application empêche une personne d’accéder à l’application Authenticator. Cela est dû au fait que l’inscription de l’appareil peut se produire dans d’autres emplacements en dehors de l’application Authenticator, par exemple dans l’application Portail d’entreprise ou dans les paramètres de compte Android.

### <a name="windows-mobile-retired"></a>Windows Mobile mis hors service

**Q** : J’ai un appareil Windows Mobile et l’application Microsoft Authenticator sur Windows Mobile est déconseillée. Puis-je continuer à utiliser l’application pour l’authentification ?

**R** : Les authentifications à l’aide de l’application Microsoft Authenticator sur Windows Mobile ne seront plus possibles après le 15 juillet 2020. Nous vous recommandons vivement d’utiliser une autre méthode d’authentification afin d’éviter de ne plus pouvoir accéder à vos comptes.<br>Les autres options pour les utilisateurs en entreprise sont les suivantes :<br><ul><li>Configuration de l’application Microsoft Authenticator pour [Android](https://play.google.com/store/apps/details?id=com.azure.authenticator) ou [iOS](https://apps.apple.com/app/microsoft-authenticator/id983156458).</li><li>Configuration de la réception des codes de vérification [par SMS](multi-factor-authentication-setup-phone-number.md).</li><li>Configuration d’un numéro de téléphone pour recevoir des [appels téléphoniques afin de vérifier l’identité](multi-factor-authentication-setup-office-phone.md).</li></ul><br>Les autres options pour les utilisateurs de compte Microsoft personnel sont les suivantes :<br><ul><li>Configuration de l’application Microsoft Authenticator pour [Android](https://play.google.com/store/apps/details?id=com.azure.authenticator) ou [iOS](https://apps.apple.com/app/microsoft-authenticator/id983156458).</li><li>Configuration d’une autre méthode de connexion (SMS ou e-mail) en mettant à jour vos informations de sécurité via la [page de sécurité du compte Microsoft](https://account.microsoft.com/security/).</li></ul>

### <a name="android-screenshots"></a>Captures d’écran Android

**Q** : Puis-je prendre des captures d’écran de mes codes de mot de passe à usage unique (OTP) sur l’authentificateur Android ?

**R** : Depuis la version 6.2003.1704 de l’authentificateur Android, par défaut, tous les codes OTP sont masqués lorsque vous prenez une capture d’écran. Si vous souhaitez que vos codes OTP soient visibles sur les captures d’écran ou permettre à d’autres applications de capturer l’écran de l’authentificateur, c’est possible. Activez simplement le paramètre **Capture d’écran** dans l’authentificateur et redémarrez l’application.

### <a name="delete-stored-data"></a>Supprimer les données stockées

**Q** : Quelles données Authenticator stocke-t-il en mon nom et comment puis-je les supprimer ?

**R** : L’application Authenticator collecte trois types d’informations :<ul><li>Les informations de compte que vous fournissez quand vous ajoutez votre compte. Vous pouvez supprimer ces données en supprimant votre compte.</li><li>Les données des journaux de diagnostic qui demeurent uniquement dans l’application jusqu’à ce que vous sélectionniez **Envoyer les journaux** dans le menu **Aide** de l’application pour envoyer les journaux à Microsoft. Ces journaux peuvent contenir des données personnelles, telles que des adresses e-mail, des adresses de serveur ou des adresses IP. Elles peuvent également contenir des données relatives à l’appareil, telles que le nom de celui-ci et la version du système d’exploitation. Toutes les données personnelles collectées sont limitées aux informations nécessaires pour résoudre les problèmes liés aux applications. Vous pouvez parcourir ces fichiers journaux dans l’application à tout moment pour voir les informations collectées. Si vous envoyez vos fichiers journaux aux ingénieurs de l’application d’authentification, ceux-ci les utiliseront uniquement pour résoudre les problèmes signalés.</li><li>Les données d’utilisation non personnellement identifiables, telles que « flux d’ajout de compte démarré/compte correctement ajouté » ou « notification approuvée ». Ces données sont indispensables pour nos prises de décisions en matière d’ingénierie. Votre utilisation nous aide à déterminer les améliorations que nous pouvons apporter aux applications. Une notification relative à cette collecte de données s’affiche lorsque vous utilisez l’application pour la première fois. Elle vous informe que vous pouvez la désactiver via la page **Paramètres** de l’application. Vous pouvez activer ou désactiver ce paramètre à tout moment.</li></ul>

### <a name="codes-in-the-app"></a>Codes dans l’application

**Q** : À quoi servent les codes de l’application ?

**R** : Lorsque vous ouvrez l’application Authenticator, les comptes que vous avez ajoutés s’affichent sous forme de vignettes. Vos comptes professionnels ou scolaires, ainsi que vos comptes Microsoft personnels comportent six ou huit chiffres visibles dans l’affichage plein écran du compte (accessible en appuyant sur la vignette du compte). Pour les autres comptes, un numéro de six ou huit chiffres s’affiche dans la page **Comptes** de l’application.<br>Vous pouvez utiliser ces codes comme mot de passe à usage unique pour vérifier que vous êtes bien celui que vous prétendez être. Une fois que vous vous êtes connecté avec votre nom d’utilisateur et votre mot de passe, vous allez saisir le code de vérification associé à ce compte. Par exemple, si vous êtes Katy en train de vous connecter à votre compte Contoso, vous devez appuyer sur la vignette du compte, puis utiliser le code de vérification 895823. Pour le compte Outlook, vous devez procéder de la même manière.<br>Appuyez sur la vignette du compte Contoso.<br>![Vignettes de comptes dans l’application Authenticator](media/user-help-auth-app-faq/katy-signin.png)<br>Une fois que vous avez appuyé sur la vignette du compte Contoso, le code de vérification est visible en mode plein écran.<br>![Code de vérification dans la vignette du compte dans l’application Authenticator](media/user-help-auth-app-faq/verification-code.png)

### <a name="countdown-timer"></a>Minuteur de compte à rebours

**Q** : Pourquoi le nombre en regard du code ne cesse de diminuer ?

**R** : Il est possible que vous voyiez un compte à rebours de 30 secondes qui défile en regard de votre code de vérification actif. Ce compte à rebours est là pour vous empêcher de vous connecter en utilisant deux fois le même code. Contrairement à un mot de passe, vous n’êtes pas supposé retenir ce nombre. L’idée est que seules les personnes ayant accès à votre téléphone connaissent votre code.

### <a name="grayed-account-tile"></a>Vignette de compte grisée

**Q** : Pourquoi la mosaïque de mon compte apparaît-elle en gris ?

**R** : Certaines organisations imposent d’utiliser l’application Authenticator avec une authentification unique afin de protéger leurs ressources. Dans ce cas, le compte n’est pas utilisé pour la vérification en deux étapes et s’affiche en gris ou comme inactif. C’est ce que l’on appelle généralement un compte « broker ».

### <a name="device-registration"></a>Enregistrement de l’appareil

**Q** : En quoi consiste l’inscription d’un appareil ?
**R** : Votre organisation peut exiger que vous inscriviez l’appareil pour effectuer le suivi de l’accès à certaines ressources sécurisées, telles que des fichiers et des applications. Elle peut également activer l’accès conditionnel pour réduire le risque d’accès indésirable à ces ressources. Vous pouvez annuler l’inscription de votre appareil à partir du menu **Paramètres**, mais vous risquez de perdre l’accès à vos e-mails dans Outlook ainsi qu’à vos fichiers stockés dans OneDrive. De plus, vous risquez de ne plus pouvoir utiliser la connexion par téléphone.

### <a name="verification-codes-when-connected"></a>Codes de vérification lors de la connexion

**Q** : Dois-je être connecté à Internet ou à mon réseau pour obtenir et utiliser les codes de vérification ?

**R** : Les codes peuvent être utilisés sans que vous ayez accès à Internet ou que vous soyez connecté aux données. Autrement dit, vous n’avez pas besoin d’un service téléphonique pour vous connecter. Et comme l’application s’arrête dès que vous le fermez, vous économisez de la batterie.

### <a name="no-notifications-when-app-is-closed"></a>Aucune notification quand l’application est fermée

**Q** : Pourquoi reçois-je des notifications uniquement lorsque l’application est ouverte ? Lorsque l’application est fermée, je ne reçois pas de notifications.

**R** : Si vous obtenez des notifications, mais pas d’alerte, même lorsque votre sonnerie est activée, vous devez vérifier les paramètres de votre application. Vérifiez que l’application est paramétrer pour signaler les notifications par un signal sonore ou en mode vibreur. Si vous ne recevez pas de notifications, vérifiez les conditions suivantes :<ul><li>Votre téléphone est-il en mode silencieux ou Ne pas déranger ? Ces modes peuvent empêcher les applications d’envoyer des notifications.</li><li>Vous recevez des notifications d’autres applications ? Si ce n’est pas le cas, il peut y avoir un problème au niveau des connexions réseau sur votre téléphone ou avec le canal de notifications d’Android ou Apple. Vous pouvez essayer de résoudre vos problèmes de connexion réseau via les paramètres de votre téléphone. Il se peut que vous deviez contacter votre fournisseur de services pour obtenir de l’aide en lien avec le canal de notifications Android ou Apple.</li><li>Vous pouvez recevoir des notifications de certains comptes sur l’application, mais pas d’autres ? Si oui, supprimez de votre application le compte qui pose problème et ajoutez-le à nouveau pour voir si le problème est résolu.</li></ul>Si le problème persiste alors que vous avez tenté toutes ces étapes, nous vous recommandons d’envoyer vos fichiers journaux de diagnostics. Ouvrez l’application, accédez au menu **Aide**, puis sélectionnez **Envoyer des journaux d’activité**. Ensuite, accédez au [Forum de l’application Microsoft Authenticator](https://social.technet.microsoft.com/Forums/en-US/home?forum=MicrosoftAuthenticatorApp) pour nous faire part du problème que vous rencontrez et des étapes que vous avez tentées.

### <a name="switch-to-push-notifications"></a>Basculer vers les notifications Push

**Q** : J’utilise les codes de vérification dans l’application, mais comment faire pour activer les notifications Push ?

**R** : Vous pouvez configurer des notifications pour votre compte professionnel ou scolaire (si votre administrateur l’autorise), ainsi que pour votre compte Microsoft personnel. Les notifications ne fonctionnent pas pour des comptes tiers, tels que Google ou Facebook.<br>Pour activer les notifications pour votre compte personnel, vous devrez réinscrire votre appareil avec le compte. Accédez à **Ajouter un compte**, sélectionnez **Compte Microsoft personnel**, puis connectez-vous à l'aide de votre nom d'utilisateur et de votre mot de passe.<br>Pour votre compte professionnel ou scolaire, votre organisation décide d’autoriser ou non les notifications en un clic.

### <a name="notifications-for-other-accounts"></a>Notifications pour d’autres comptes

**Q** : Les notifications fonctionnent-elles pour les comptes non Microsoft ?

**R** : Non, les notifications fonctionnent uniquement avec les comptes Microsoft et Azure Active Directory. Si votre entreprise ou établissement scolaire utilise des comptes Azure AD, ils peuvent désactiver cette fonctionnalité.

### <a name="backup-and-recovery"></a>Sauvegarde et récupération

**Q** : J’ai un nouvel appareil ou bien j’ai restauré mon appareil à partir d’une sauvegarde. Comment faire pour reconfigurer mes comptes dans l’application Authenticator ?

**R** : Si vous avez activé l’option **Sauvegarde cloud** sur votre ancien appareil, vous pouvez utiliser votre ancienne sauvegarde pour récupérer les informations d’identification de votre compte sur votre nouvel appareil iOS ou Android. Pour en savoir plus, consultez l’article [Sauvegarder et récupérer des informations d’identification de compte avec l’application Authenticator](user-help-auth-app-backup-recovery.md).

### <a name="lost-device"></a>Appareil perdu

**Q** : J’ai perdu mon appareil ou j’utilise un nouveau périphérique. Comment m’assurer que les notifications ne sont plus envoyées sur mon ancien périphérique ?

**R** : L’ajout de l’application Authenticator à votre nouvel appareil n’a pas pour effet de la supprimer automatiquement de l’ancien. Même la suppression de l’application de votre ancien périphérique ne suffit pas. Vous devez supprimer l’application de votre ancien appareil ET demander à Microsoft ou à votre organisation d’oublier celui-ci et d’annuler son inscription.<ul><li>**Pour supprimer l’application d’un appareil à l’aide d’un compte Microsoft personnel :** Accédez à la zone de vérification en deux étapes de votre page [Sécurité du compte](https://account.microsoft.com/security)  et choisissez de désactiver la vérification de votre ancien périphérique.</li><li>**Pour supprimer l’application d’un appareil à l’aide d’un compte Microsoft scolaire ou professionnel :** Pour désactiver la vérification de votre ancien appareil, accédez à la zone de vérification en deux étapes de votre [page MyApps](https://myapps.microsoft.com/) ou du portail personnalisé de votre organisation.</li></ul>

### <a name="remove-account-from-app"></a>Supprimer le compte de l’application

**Q** : Comment supprimer un compte de l’application ?

**R** : Appuyez sur la vignette du compte que vous souhaitez supprimer de l’application pour afficher le compte en mode plein écran. Appuyez sur l’option **Supprimer le compte** pour supprimer le compte de l’application.<br>Si vous utilisez un appareil inscrit auprès de votre organisation, il se peut que vous deviez effectuer une opération supplémentaire pour supprimer votre compte. Sur ces appareils, l’application Authenticator est automatiquement inscrite en tant qu’administrateur d’appareil. Si vous souhaitez désinstaller complètement l’application, vous devez tout d’abord annuler l’inscription de l’application dans les paramètres d’application.

### <a name="too-many-permissions"></a>Trop d’autorisations

**Q** : Pourquoi l’application demande-t-elle autant d’autorisations ?

**R** : Voici la liste complète des autorisations qui peuvent être demandées ainsi que la façon dont elles sont utilisées par l’application. Les autorisations présentées dépendent du type de téléphone que vous utilisez.<ul><li>**Utiliser du matériel biométrique.** Certains comptes professionnels et scolaires demandent un code PIN supplémentaire chaque fois que vous vérifiez votre identité. L’application requiert votre consentement pour utiliser une reconnaissance biométrique ou faciale au lieu d’entrer le code PIN.</li><li>**Appareil photo.** Permet d’analyser les codes QR lorsque vous ajoutez un compte professionnel, scolaire ou tiers.</li><li>**Contacts et téléphone.** L’application nécessite cette autorisation pour rechercher des comptes Microsoft professionnels ou scolaires sur votre téléphone et les ajouter à l’application pour vous.</li><li>**SMS.** Utilisé pour s’assurer que votre numéro de téléphone correspond au numéro enregistré lorsque vous vous connectez avec votre compte Microsoft personnel pour la première fois. Nous envoyons au téléphone sur lequel vous avez installé l’application un SMS contenant un code de vérification de 6 à 8 chiffres. Vous n’avez pas besoin de chercher ce code et de l’entrer parce que l’application Authenticator le trouve automatiquement dans le SMS.</li><li>**Se superposer aux autres applications.** La notification que vous recevez pour la vérification de votre identité s’affiche également dans les autres applications en cours d’exécution.</li><li>**Recevoir des données à partir d’Internet.** Cette autorisation est requise pour l’envoi de notifications.</li><li>**Empêcher la mise en veille du téléphone.** Si vous enregistrez votre appareil auprès de votre entreprise, elle peut modifier cette stratégie sur votre téléphone.</li><li>**Contrôler les vibrations.** Vous pouvez choisir si vous souhaitez que votre téléphone vibre chaque fois que vous recevez une notification visant à vérifier votre identité.</li><li>**Utiliser du matériel de détection de l’empreinte digitale.** Certains comptes professionnels et scolaires demandent un code PIN supplémentaire chaque fois que vous vérifiez votre identité. Pour faciliter le processus, nous vous autorisons à utiliser vos empreintes digitales au lieu d’avoir à saisir votre code PIN.</li><li> **Afficher les connexions réseau.** Lorsque vous ajoutez un compte Microsoft, l’application exige une connexion réseau/Internet.</li><li>**Lire le contenu de votre stockage.** Cette autorisation est utilisée uniquement lorsque vous signalez un problème technique via les paramètres de l’application. Certaines informations sont collectées à partir de votre espace de stockage afin de diagnostiquer le problème.</li><li>**Accès complet au réseau.** Cette autorisation est requise pour l’envoi de notifications visant à vérifier votre identité.</li><li>**Exécuter au démarrage.** Si vous redémarrez votre téléphone, cette autorisation vous permet de continuer de recevoir des notifications visant à vérifier votre identité.</li></ul>

### <a name="approve-requests-without-unlocking"></a>Approuver les demandes sans déverrouillage

**Q** : Pourquoi l’application Authenticator permet-elle d’approuver une demande sans déverrouiller l’appareil ?

**R** : Vous n’êtes pas obligé de déverrouiller votre appareil pour approuver les requêtes de vérification, car vous devez simplement prouver que vous avez votre téléphone avec vous. La vérification en deux étapes fait appel à deux éléments : un que vous connaissez, et un autre que vous possédez. L’élément que vous connaissez est votre mot de passe. L’élément que vous possédez est votre téléphone (configuré avec l’application Authenticator et inscrit en tant que preuve pour l’authentification multifacteur). Par conséquent, le fait d’être en possession du téléphone et d’approuver la demande remplit les critères du second facteur d’authentification.

### <a name="activity-notifications"></a>Notifications d’activité

**Q** : Pourquoi est-ce que je reçois des notifications sur l’activité de mon compte ?

**R** : Des notifications d’activité sont envoyées immédiatement à l’application Authenticator chaque fois qu’une modification est apportée à vos comptes Microsoft personnels, ce qui vous permet de rester plus sécurisé. Auparavant, nous envoyions ces notifications uniquement par e-mail et SMS. Pour plus d’informations sur ces notifications d’activité, voir [Que se passe-t-il en cas de connexion inhabituelle à votre compte](https://support.microsoft.com/help/13967/microsoft-account-unusual-sign-in). Pour changer l’endroit où vous recevez vos notifications, connectez-vous à la page [Où pouvons-nous vous contacter pour les alertes de compte non critiques](https://account.live.com/SecurityNotifications/Update) de votre compte.

### <a name="one-time-passcodes"></a>Codes secrets à usage unique

**Q** : Mes codes secrets à usage unique ne fonctionnent pas. Que dois-je faire ?

**R** : Vérifiez que la date et l’heure de votre appareil sont correctes et qu’elles sont synchronisées automatiquement. Si la date et l’heure sont incorrectes, ou ne sont pas synchronisées, le code ne fonctionne pas.

### <a name="windows-10-mobile"></a>Windows 10 Mobile

**Q** : Le système d'exploitation Windows 10 Mobile est déconseillé depuis décembre 2019. L'application Microsoft Authenticator sera-t-elle également déconseillée sur les systèmes d'exploitation Windows Mobile ?

**R** : Depuis le 28 février 2020, l’application Authenticator n’est plus prise en charge sur les systèmes d’exploitation Windows Mobile. Passé cette date, les utilisateurs ne recevront plus aucune nouvelle mise à jour de l'application. Après le 28 février 2020, les services Microsoft qui prennent actuellement en charge les authentifications à l'aide de Microsoft Authenticator sur tous les systèmes d'exploitation Windows Mobile cesseront progressivement cette prise en charge. Pour s'authentifier auprès des services Microsoft, nous encourageons vivement tous nos utilisateurs à passer à un autre mécanisme d'authentification avant cette date.

### <a name="default-mail-app"></a>Application de messagerie par défaut

**Q** : Lorsque je me connecte à mon compte professionnel ou scolaire à l’aide de l’application de messagerie par défaut fournie avec iOS, l’application Authenticator m’invite à fournir mes informations de vérification de sécurité. Une fois que je saisis ces informations et que je retourne sur l’application de messagerie, je reçois un message d’erreur. Que puis-je faire ?

**R** : Cela est probablement dû au fait que votre connexion et votre application de messagerie se font sur deux applications différentes, provoquant ainsi l’arrêt, puis l’échec du processus initial de connexion en arrière-plan. Pour tenter de résoudre ce problème, nous vous recommandons de sélectionner l’icône **Safari** située en bas à droite de l’écran lorsque vous vous connectez à l’application de messagerie. En utilisant Safari, l’ensemble du processus de connexion se produit dans une application unique, ce qui vous permet de vous connecter à l’application avec succès.

### <a name="apple-watch-watchos-7"></a>Apple Watch watchOS 7

**Q** : Pourquoi j’ai des problèmes avec Apple Watch sur watchOS 7 ?

**R** : Un problème perturbe l’approbation des notifications sur watchOS 7 et nous travaillons avec Apple pour le résoudre. En attendant, toutes les notifications qui nécessitent l’application Microsoft Authenticator watchOS doivent être approuvées sur votre téléphone à la place.

### <a name="apple-watch-doesnt-show-accounts"></a>Apple Watch n’affiche pas les comptes

**Q** : Pourquoi tous mes comptes ne s’affichent-ils pas lorsque j’ouvre l’application Authenticator sur mon Apple Watch ?

**R** : L’application Authenticator prend uniquement en charge les comptes Microsoft personnels ou les comptes scolaires ou professionnels paramétrés avec l’envoi de notifications Push sur l’application complémentaire Apple Watch. Pour vos autres comptes, par exemple Google ou Facebook, vous devez ouvrir l’application Authenticator sur votre téléphone pour afficher vos codes de vérification.

### <a name="apple-watch-notifications"></a>Notifications Apple Watch

**Q** : Pourquoi je ne peux pas approuver ou refuser les notifications sur mon Apple Watch ?

**R** : Commencez par vérifier que vous avez bien effectué la mise à niveau de votre application Authenticator vers la version 6.0.0 ou une version ultérieure sur votre iPhone. Après cela, ouvrez l’application complémentaire Microsoft Authenticator sur votre Apple Watch et recherchez tous les comptes sous lesquels figure un bouton **Configurer**. Suivez le processus de configuration pour approuver les notifications pour ces comptes.

### <a name="apple-watch-communication-error"></a>Erreur de communication Apple Watch

**Q** : J’obtiens une erreur de communication entre l’Apple Watch et mon téléphone. Que puis-je faire pour résoudre les problèmes ?

**R** : Cette erreur se produit lorsque l’écran de l’Apple Watch se met en veille avant qu’elle n’ait fini de communiquer avec votre téléphone.<br><b>Si l’erreur se produit pendant l’installation :</b><br>Essayez de réexécuter le programme d’installation, en veillant à garder votre Apple Watch active jusqu’à ce que le processus soit terminé. En même temps, ouvrez l’application sur votre téléphone et répondez à toutes les invites qui s’affichent.<br>Si votre téléphone et votre montre ne parviennent toujours pas à communiquer, vous pouvez essayer l’une des actions suivantes :<ol><li>Forcez l’arrêt de l’application de téléphone Microsoft Authenticator et ouvrez-la à nouveau sur votre iPhone.</li><li>Forcez l’arrêt de l’application complémentaire sur votre Apple Watch.<ol><li> Ouvrez l’application complémentaire Microsoft Authenticator sur votre montre.</li><li>Maintenez le bouton latéral enfoncé jusqu’à ce que l’écran **Arrêt** s’affiche.</li><li>Relâchez le bouton latéral et maintenez la Digital Crown enfoncée pour forcer l’arrêt de l’application.</li></ol></li><li>Désactivez le Bluetooth et le Wi-Fi pour votre téléphone et votre montre, puis réactivez-les.</li><li>Redémarrez votre iPhone et votre Watch.</li></ol><b>Si l’erreur se produit lorsque vous tentez d’approuver une notification :</b><br>La prochaine fois que vous tentez d’approuver une notification sur votre Apple Watch, maintenez l’écran allumé jusqu’à ce que la demande soit terminée et que vous entendiez le son qui indique sa réussite.

### <a name="apple-watch-companion-app-not-syncing"></a>Application compagnon Apple Watch non synchronisée

**Q** : Pourquoi l’application complémentaire Microsoft Authenticator pour Apple Watch ne se synchronise pas ou n’apparaît pas sur mon Apple Watch ?

**R** : Si l’application n’apparaît pas sur votre Apple Watch, tentez les actions suivantes : <ol><li>Assurez-vous que votre Apple Watch fonctionne sous watchOS 4.0 ou une version ultérieure.</li><li>Synchronisez de nouveau votre Apple Watch.</li></ol>

### <a name="apple-watch-companion-app-crashed"></a>L’application compagnon Apple Watch s’est bloquée

**Q** : Mon application complémentaire Apple Watch a planté. Puis-je envoyer vous mes journaux d’activité d’incident afin que vous puissiez l’examiner ?

**R** : Vous devez d’abord vous assurer que vous avez choisi de partager votre analytique avec nous. Si vous êtes un utilisateur de TestFlight, vous êtes déjà inscrit. Sinon, accédez à **Paramètres > Confidentialité > Analytique** et sélectionnez les options **Partager l’analyse de l’iPhone et de Watch** et **Partager avec les développeurs d’applications**.<br>Une fois inscrit, vous pouvez essayer de reproduire l’incident pour que vos journaux d’activité d’incident nous soient automatiquement envoyés pour examen. Toutefois, si vous ne pouvez pas reproduire l’incident, vous pouvez copier manuellement vos fichiers journaux et nous les envoyer.<ol><li>Ouvrez l’application Watch sur votre téléphone, accédez à **Paramètres > Général**, puis cliquez sur **Copy Watch Analytics** (Copier l’analyse de Watch).</li><li>Recherchez l’incident correspondant sous **Paramètres > Confidentialité > Analyse > Données d’analyse**, puis copiez manuellement l’intégralité du texte.</li><li>Ouvrez l’application Authenticator sur votre téléphone et collez le texte copié dans la zone de texte **Partage avec les développeurs d’applications** sur la page **Envoyer des journaux d’activité**.</li></ol>

## <a name="autofill-for-consumers"></a>Remplissage automatique pour les consommateurs

**Q** : Qu’est-ce que le remplissage automatique dans Authenticator ?

**R** : L’application Authenticator permet désormais de stocker et de remplir automatiquement et en toute sécurité les mots de passe des applications et des sites web que vous consultez sur votre téléphone. Vous pouvez utiliser la fonction de remplissage automatique pour synchroniser et remplir automatiquement vos mots de passe sur vos appareils iOS et Android. Une fois que vous avez configuré l’application Authenticator comme fournisseur de remplissage automatique sur votre téléphone, elle propose de sauvegarder vos mots de passe lorsque vous les entrez sur un site ou une page de connexion à l’application. Les mots de passe sont enregistrés dans le cadre de [votre compte Microsoft](https://account.microsoft.com/account) et sont également disponibles lorsque vous vous connectez à Microsoft Edge avec votre compte Microsoft.

**Q** : Quelles sont les informations qu’Authenticator peut remplir automatiquement pour moi ?

**R** : Authenticator peut remplir automatiquement les noms d’utilisateur et les mots de passe sur les sites et les applications que vous consultez sur votre téléphone.

**Q** : Comment activer le remplissage automatique des mots de passe dans Authenticator sur mon téléphone ?

**R** : Procédez comme suit :

1. Ouvrez l’application Authenticator.
1. Dans **Paramètres** sous **Bêta**, activez le **remplissage automatique**.
1. Dans l’onglet **Mots de passe** d’Authenticator, sélectionnez **Se connecter avec Microsoft** et connectez-vous à l’aide de [votre compte Microsoft](https://account.microsoft.com/account). Actuellement, cette fonctionnalité prend uniquement en charge les comptes Microsoft ; elle ne prend pas encore en charge les comptes professionnels ou scolaires.

**Q** : Comment faire d’Authenticator le fournisseur de remplissage automatique par défaut sur mon téléphone ?

**R** : Procédez comme suit :

1. Ouvrez les **paramètres** d’Authenticator et, sous **Bêta**, activez le **remplissage automatique**.
1. Dans l’onglet **Mots de passe** de l’application, connectez-vous à l’aide de [votre compte Microsoft](https://account.microsoft.com/account).
1. Effectuez l’une des opérations suivantes :

   - Sur iOS, sous **Paramètres**, sélectionnez **Comment activer la fonction de remplissage automatique** dans la section Paramètres de remplissage automatique pour savoir comment définir Authenticator comme fournisseur de remplissage automatique par défaut.
   - Sur Android, sous **Paramètres**, sélectionnez **Définir comme fournisseur de remplissage automatique** dans la section Paramètres de remplissage automatique pour définir Authenticator comme fournisseur de remplissage automatique par défaut.

**Q** : Que se passe-t-il si le bouton bascule du **remplissage automatique** est grisé dans mes paramètres ?

**R** : Le remplissage automatique est actuellement en version bêta et n’a pas encore été activé pour toutes les organisations ni pour tous les types de comptes. Si le bouton bascule du **remplissage automatique** dans **Paramètres** est grisé, cela est probablement dû au fait que vous utilisez l’application Authenticator avec votre compte professionnel. Vous pouvez utiliser cette fonctionnalité sur un appareil sur lequel votre compte professionnel n’est pas ajouté. Si votre organisation travaille avec Microsoft, le bouton bascule du **remplissage automatique** est activé même lorsqu’un compte professionnel est ajouté à Authenticator.

**Q** : Comment arrêter la synchronisation des mots de passe ?

**R** : Pour arrêter la synchronisation des mots de passe dans l’application Authenticator, ouvrez **Paramètres** > **Paramètres de remplissage automatique** > **Synchroniser le compte**. Dans l’écran suivant, vous pouvez sélectionner l’option **Arrêter la synchronisation et supprimer toutes les données de remplissage automatique**. Cela supprimera les mots de passe et autres données de remplissage automatique de l’appareil. La suppression des données de remplissage automatique n’a pas d’impact sur l’authentification multifacteur.

**Q** : Comment mes mots de passe sont-ils protégés par l’application Authenticator ?

**R** : L’application Authenticator offre déjà un niveau de sécurité élevé pour l’authentification multifacteur et la gestion des comptes, et la même exigence élevée en matière de sécurité est appliquée à la gestion de vos mots de passe.

- **Une authentification forte est nécessaire pour l’application Authenticator** : La connexion à Authenticator requiert un deuxième facteur. Cela signifie que vos mots de passe dans l’application Authenticator ne sont pas accessibles même si un tiers connaît le mot de passe de votre compte Microsoft.
- **Les données de remplissage automatique sont protégées par des données biométriques et un code secret** : Avant de pouvoir remplir automatiquement le mot de passe sur une application ou un site, Authenticator requiert un code secret par données biométriques ou un code secret d’appareil. Cela garantit que, même si une autre personne a accès à votre appareil, elle ne peut ni remplir ni voir votre mot de passe, car elle serait incapable de fournir vos données biométriques ou le code confidentiel de l’appareil. En outre, un utilisateur ne peut pas ouvrir la page Mots de passe, sauf s’il fournit des données biométries ou un code confidentiel et ce, même s’il désactive la fonctionnalité Verrou d’application dans les paramètres de l’application.
- **Mots de passe chiffrés sur l’appareil** : Les mots de passe sur l’appareil sont chiffrés, et les clés de chiffrement/déchiffrement ne sont jamais stockées et sont toujours générées à la volée. Les mots de passe ne sont déchiffrés que lorsque l’utilisateur le souhaite, c’est-à-dire lors du remplissage automatique ou lorsque l’utilisateur souhaite voir le mot de passe, les deux cas nécessitant des données biométriques ou un code confidentiel.
- **Sécurité du cloud et du réseau** : Vos mots de passe sur le cloud sont chiffrés et déchiffrés uniquement lorsqu’ils atteignent votre appareil. Les mots de passe sont synchronisés via une connexion HTTPS protégée par SSL, ce qui garantit qu’aucun attaquant ne peut espionner les données sensibles lorsqu’elles sont synchronisées. Nous veillons également à vérifier l’intégrité des données synchronisées sur le réseau en utilisant des fonctions hachées de chiffrement (en particulier, un code d’authentification des messages basé sur le hachage).

## <a name="autofill-for-it-admins"></a>Remplissage automatique pour les administrateurs informatiques

**Q** : Mes employés ou étudiants pourront-ils utiliser le remplissage automatique des mots de passe dans l’application Authenticator ?

**R** : Non. La fonctionnalité de remplissage automatique est actuellement en version bêta et n’a pas encore été activée pour toutes les organisations ni pour tous les types de comptes. Si votre employé ou étudiant a ajouté son compte professionnel ou scolaire à l’application Microsoft Authenticator, le remplissage automatique des mots de passe ne lui est pas accessible. La seule exception à cette restriction est lorsque votre employé ou étudiant ajoute son compte professionnel ou scolaire dans l’authentification multifacteur basée sur le cloud Microsoft en tant que [compte externe ou tiers](user-help-auth-app-add-non-ms-account.md).

**Q** : Puis-je rendre la fonctionnalité de remplissage automatique disponible pour mes employés (ou étudiants) ?

**R** : Oui. Pour permettre à vos employés ou étudiants d’y accéder, votre entreprise ou votre école peut être ajoutée à une liste d’autorisation. Contactez votre support technique ou votre contact Microsoft pour être ajouté à la liste d’autorisation. En outre, si vous êtes un administrateur informatique de votre organisation, vous pouvez également remplir un formulaire pour exprimer votre intérêt à rejoindre le programme sur la page [Ajouter une entreprise à une liste d’autorisation pour le remplissage automatique dans Authenticator](https://aka.ms/RequestAutofillInAuthenticator).

**Q** : Le mot de passe du compte professionnel ou scolaire de mes utilisateurs sera-t-il automatiquement synchronisé ?

**R** : Non. Le remplissage automatique des mots de passe ne synchronise pas le mot de passe du compte professionnel ou scolaire de vos utilisateurs. Lorsque les utilisateurs accèdent à un site ou à une application, Authenticator propose d’enregistrer le mot de passe de ce site ou de cette application, et le mot de passe est enregistré uniquement quand l’utilisateur le souhaite.
  
**Q** : Puis-je ajouter à une liste d’autorisation uniquement certains utilisateurs de mon organisation pour le remplissage automatique ?

**R** : Non. Pour le moment, les entreprises peuvent uniquement activer le remplissage automatique des mots de passe pour tous leurs employés ou aucun d’entre eux. Nous développerons progressivement ces contrôles.

**Q** : Que se passe-t-il si mon employé ou étudiant a plusieurs comptes professionnels ou scolaires ? Par exemple, mon employé a des comptes provenant de plusieurs entreprises ou écoles enregistrés dans son application Microsoft Authenticator.

**R** : Toutes les entreprises ou écoles ajoutées à l’application Authenticator doivent être ajoutées à une liste d’autorisation pour le remplissage automatique dans Authenticator afin que le propriétaire de l’application puisse l’utiliser. La seule exception à cette restriction est lorsque votre employé ou étudiant ajoute son compte professionnel ou scolaire dans l’authentification multifacteur basée sur le cloud Microsoft en tant que [compte externe ou tiers](user-help-auth-app-add-non-ms-account.md).

## <a name="next-steps"></a>Étapes suivantes

- Si vous ne parvenez pas à obtenir le code de vérification de votre compte Microsoft personnel, consultez la section **Résolution des problèmes de code de vérification** de l’article [Informations de sécurité et codes de vérification des comptes Microsoft](https://support.microsoft.com/help/12428/microsoft-account-security-info-verification-codes).

- Pour plus d’informations sur la vérification en deux étapes, consultez [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md)

- Pour en savoir plus sur les informations de sécurité, consultez [Vue d’ensemble des informations de sécurité (préversion)](./security-info-setup-signin.md).

- Si vous n’avez pas trouvé la réponse à votre question ici, rendez-vous sur le [Forum de l’application Microsoft Authenticator](https://social.technet.microsoft.com/Forums/en-us/home?forum=MicrosoftAuthenticatorApp) pour publier votre question et obtenir de l’aide de la part de la communauté ou laissez un commentaire sur cette page.
