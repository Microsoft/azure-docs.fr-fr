---
title: Sécurité de la gamme StorSimple 8000 | Microsoft Docs
description: Décrit les fonctionnalités de sécurité et de confidentialité qui protègent votre service, votre appareil et vos données StorSimple en local et dans le cloud.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 05/18/2018
ms.author: alkohli
ms.openlocfilehash: 72bf252488878c8b6fabb2ea2d61658a2cd87e14
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966205"
---
# <a name="storsimple-security-and-data-protection"></a>Sécurité et protection des données StorSimple

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

La sécurité est une préoccupation majeure pour toute personne adoptant une nouvelle technologie, surtout lorsque cette technologie est utilisée avec des données confidentielles ou propriétaires. Lorsque vous évaluez différentes technologies, vous devez tenir compte des risques et des coûts relatifs à la protection des données. Microsoft Azure StorSimple fournit une solution de sécurité et de confidentialité pour la protection des données, afin de garantir :

* **La confidentialité** : seules les entités autorisées peuvent consulter vos données.
* **L’intégrité** : seules les entités autorisées peuvent modifier ou supprimer vos données.

La solution Microsoft Azure StorSimple repose sur l’interaction de quatre composants :

* **Le service StorSimple Device Manager hébergé dans Microsoft Azure** : service de gestion vous permettant de configurer et d’approvisionner l’appareil StorSimple.
* **L’appareil StorSimple** : appareil physique installé dans votre centre de données. Tous les hôtes et les clients qui génèrent des données sont connectés à l’appareil StorSimple, qui gère les données et les déplace vers le cloud Azure selon vos besoins.
* **Les clients/hôtes connectés à l’appareil** : clients de votre infrastructure se connectant à l’appareil StorSimple et générant des données devant être protégées.
* **Le stockage cloud** : emplacement dans le cloud Azure où les données sont stockées.

Les sections suivantes décrivent les fonctionnalités de sécurité StorSimple de chacun de ces composants et les données qu’ils protègent. Elles comprennent également une liste de questions que vous pouvez vous poser sur la sécurité de Microsoft Azure StorSimple et des réponses à celles-ci.

## <a name="storsimple-device-manager-service-protection"></a>Protection du service StorSimple Device Manager

Le service StorSimple Device Manager est un service de gestion hébergé dans Microsoft Azure, qui permet de gérer tous les appareils StorSimple de votre organisation. Vous pouvez accéder au service StorSimple Device Manager en vous connectant au portail Azure via un navigateur web à l’aide de vos informations d’identification professionnelles.

L’accès au service StorSimple Device Manager requiert que votre organisation dispose d’un abonnement Azure incluant StorSimple. Votre abonnement régit les fonctionnalités auxquelles vous pouvez accéder dans le portail Azure. Si votre organisation ne dispose pas d’un abonnement Azure et que vous souhaitez en savoir plus à ce sujet, consultez [Inscription à Azure en tant qu’organisation](../active-directory/fundamentals/sign-up-organization.md).

Étant donné que le service StorSimple Device Manager est hébergé dans Azure, il est protégé par les fonctionnalités de sécurité Azure. Pour plus d’informations sur les fonctionnalités de sécurité fournies par Microsoft Azure, accédez au [Centre de confidentialité Microsoft Azure](https://azure.microsoft.com/support/trust-center/security/).

## <a name="storsimple-device-protection"></a>Protection de l’appareil StorSimple

L’appareil StorSimple est un appareil de stockage hybride local qui contient des disques SSD et des disques durs, ainsi que des contrôleurs redondants et des fonctionnalités de basculement automatique. Les contrôleurs gèrent la hiérarchisation du stockage en plaçant les données actuellement (ou souvent) utilisées sur le stockage local (sur l’appareil StorSimple ou les serveurs locaux), tout en déplaçant les données moins fréquemment utilisées sur le cloud.

Seuls les appareils autorisés par StorSimple peuvent rejoindre le service StorSimple Device Manager que vous avez créé dans votre abonnement Azure. Pour autoriser un appareil, vous devez l’inscrire auprès du service StorSimple Device Manager en fournissant la clé d’inscription. La clé d’inscription de service est une clé aléatoire de 128 bits, générée dans le portail Azure.

![Clé d'inscription du service](./media/storsimple-security/ServiceRegistrationKey.png)

Pour savoir comment obtenir une clé d’inscription, passez à l’ [Étape 2 : Obtenir la clé d’inscription du service](storsimple-8000-deployment-walkthrough-u2.md#step-2-get-the-service-registration-key).

La clé d’inscription est une clé longue, contenant plus de 100 caractères. Vous pouvez copier la clé et l’enregistrer dans un fichier texte dans un emplacement sécurisé, afin de l’utiliser pour autoriser des appareils supplémentaires si nécessaire. Si vous égarez la clé d’inscription après avoir enregistré votre premier appareil, vous pouvez générer une nouvelle clé à partir du service StorSimple Device Manager. Cela n’a pas d’incidence sur le fonctionnement des appareils existants.

Une fois qu’un appareil est inscrit, il utilise des jetons pour communiquer avec Microsoft Azure. La clé d’inscription au service n’est pas utilisée après l’inscription de l’appareil.

> [!NOTE]
> Il est recommandé de régénérer la clé d'inscription au service après chaque utilisation.


## <a name="protect-your-storsimple-solution-via-passwords"></a>Protection de votre solution StorSimple par des mots de passe

Les mots de passe sont un aspect important de la sécurité informatique et sont très utilisés dans la solution StorSimple, afin de garantir que seuls les utilisateurs autorisés ont accès à vos données. StorSimple vous permet de configurer les mots de passe suivants :

* Mot de passe Administrateur d’appareil StorSimple
* Mots de passe initiateur et cible de protocole CHAP (Challenge Handshake Authentication Protocol)
* Mot de passe de gestionnaire d’instantanés StorSimple

### <a name="windows-powershell-for-storsimple-and-the-storsimple-device-administrator-password"></a>Mot de passe Administrateur Windows PowerShell pour StorSimple et appareil StorSimple

Windows PowerShell pour StorSimple est une interface de ligne de commande que vous pouvez utiliser pour gérer l’appareil StorSimple. Windows PowerShell pour StorSimple dispose de fonctionnalités qui vous permettent d’inscrire votre appareil, de configurer l’interface réseau sur votre appareil, d’installer certains types de mises à jour, de résoudre les problèmes de votre appareil en accédant à la session de support et de modifier l’état de l’appareil. Vous pouvez accéder à Windows PowerShell pour StorSimple en vous connectant à la console série sur l’appareil ou à l’aide de l’accès distant Windows PowerShell.

L’accès distant PowerShell peut être effectué via HTTPS ou HTTP. Si la gestion à distance via HTTPS est activée, vous devez télécharger le certificat de gestion à distance à partir du périphérique et l’installer sur le client distant. Pour plus d'informations sur la communication à distance PowerShell, consultez la page [Connexion à distance à votre appareil StorSimple](storsimple-8000-remote-connect.md).

Après avoir utilisé Windows PowerShell pour StorSimple pour vous connecter à l’appareil, vous devez fournir le mot de passe Administrateur de ce dernier pour vous y connecter.

![Mot de passe d'administrateur de l'appareil](./media/storsimple-security/DeviceAdminPW.png)

Gardez à l'esprit les meilleures pratiques suivantes :

* La gestion à distance est désactivée par défaut. Vous pouvez utiliser le service StorSimple Device Manager pour l’activer. Pour des raisons de sécurité, l’accès à distance doit être activé uniquement pendant la période durant laquelle il est réellement nécessaire.
* Si vous modifiez le mot de passe, veillez à informer tous les utilisateurs de l’accès à distance afin qu’ils ne connaissent pas de perte de connectivité inattendue.
* Le service StorSimple Device Manager ne peut pas récupérer les mots de passe existants : il ne peut que les réinitialiser. Nous vous recommandons de stocker tous les mots de passe dans un endroit sûr, afin de ne pas être obligé de réinitialiser un mot de passe en cas d’oubli. Si vous devez réinitialiser un mot de passe, veillez à informer tous les utilisateurs au préalable.

Vous pouvez accéder à l’interface Windows PowerShell à l’aide d’une connexion série à l’appareil. Vous pouvez également y accéder à distance à l’aide de HTTP ou HTTPS, qui offre une sécurité supplémentaire. Le protocole HTTPS assure un niveau de sécurité supérieur à une connexion série ou HTTP. Toutefois, pour utiliser le protocole HTTPS, vous devez d’abord installer un certificat sur l’ordinateur client qui accède à l’appareil. Vous pouvez télécharger le certificat d’accès à distance à partir de la page de configuration de l’appareil du service StorSimple Device Manager. En cas de perte du certificat pour l’accès à distance, vous devez en télécharger un nouveau et le transmettre à tous les clients autorisés à utiliser la gestion à distance.

### <a name="challenge-handshake-authentication-protocol-chap-initiator-and-target-passwords"></a>Mots de passe initiateur et cible de protocole CHAP (Challenge Handshake Authentication Protocol)

CHAP est un schéma d’authentification utilisé par l’appareil StorSimple pour valider l’identité des clients distants. La vérification repose sur un mot de passe partagé. Le protocole CHAP peut être à sens unique (unidirectionnel) ou mutuel (bidirectionnel). Dans le cas de l’authentification CHAP unidirectionnelle, la cible (l’appareil StorSimple) authentifie un initiateur (l’hôte). L’authentification CHAP mutuelle ou inverse nécessite que la cible authentifie l’initiateur, puis que l’initiateur authentifie la cible. Votre appareil StorSimple peut être configuré pour utiliser l’une ou l’autre de ces méthodes.

N'oubliez pas les éléments suivants lorsque vous configurez le protocole CHAP :

* Le nom d’utilisateur CHAP doit contenir moins de 233 caractères.
* Le mot de passe CHAP doit comprendre entre 12 et 16 caractères. L’utilisation d’un nom d’utilisateur ou d’un mot de passe plus long entraîne un échec d’authentification sur l’hôte Windows.
* Vous ne pouvez pas utiliser le même mot de passe pour l’initiateur CHAP et la cible CHAP.
* Une fois le mot de passe défini, il peut être modifié, mais il ne peut pas être récupéré. En cas de modification du mot de passe, veillez à informer tous les utilisateurs de l’accès à distance afin qu’ils puissent se connecter à l’appareil StorSimple.

Pour plus d'informations sur CHAP et comment le configurer pour votre solution StorSimple, consultez la page [Configuration de CHAP pour votre appareil StorSimple](storsimple-8000-configure-chap.md).

### <a name="storsimple-snapshot-manager-password"></a>Mot de passe de gestionnaire d’instantanés StorSimple

Le gestionnaire d’instantanés StorSimple est un composant logiciel enfichable MMC (Microsoft Management Console) qui utilise des groupes de volumes et Windows Volume Shadow Copy Service pour générer des sauvegardes cohérentes avec les applications. En outre, vous pouvez utiliser le gestionnaire d’instantanés StorSimple pour créer des planifications de sauvegarde et cloner ou restaurer des volumes.

Lorsque vous configurez un appareil pour utiliser le gestionnaire d'instantanés StorSimple, vous devez fournir le mot de passe du gestionnaire d'instantanés StorSimple. Ce mot de passe est d'abord défini dans Windows PowerShell pour StorSimple lors de l'inscription. Le mot de passe peut également être défini et modifié à partir du service StorSimple Device Manager. Ce mot de passe authentifie l’appareil auprès du gestionnaire d’instantanés StorSimple.

![Mot de passe de gestionnaire d’instantanés StorSimple](./media/storsimple-security/SnapshotMgrPassword.png)

Le mot de passe du Gestionnaire d'instantanés StorSimple doit comporter 14 ou 15 caractères et contenir une combinaison d'au moins 3 caractères en majuscules, minuscules, numériques et spéciaux. Une fois le mot de passe du gestionnaire d’instantanés Storsimple défini, il peut être modifié, mais il ne peut pas être récupéré. Si vous modifiez le mot de passe, veillez à informer tous les utilisateurs distants.

Pour plus d'informations sur le Gestionnaire d'instantanés StorSimple, consultez [Qu'est-ce que le Gestionnaire d'instantanés StorSimple ?](storsimple-what-is-snapshot-manager.md)

### <a name="password-best-practices"></a>Meilleures pratiques relatives aux mots de passe

Nous vous recommandons d'utiliser les instructions suivantes afin de garantir que les mots de passe StorSimple sont forts et bien protégés :

* Modifiez votre mot de passe tous les trois mois. Vous devez modifier les mots de passe tous les ans.
* Utilisez des mots de passe forts. Pour plus d’informations, consultez [Conseils pour créer un mot de passe plus fort et le protéger](https://cloudblogs.microsoft.com/microsoftsecure/2014/08/25/create-stronger-passwords-and-protect-them/).
* Utilisez toujours des mots de passe différents pour les différents mécanismes d’accès : chacun des mots de passe que vous spécifiez doit être unique.
* Ne partagez pas les mots de passe avec les personnes non autorisées à accéder à l’appareil StorSimple.
* Ne parlez pas d’un mot de passe devant d’autres personnes et ne faites pas allusion à son format.
* Si vous pensez qu’un compte ou un mot de passe a été compromis, signalez l’incident à votre service de sécurité.
* Traitez tous les mots de passe comme des informations sensibles et confidentielles. 

## <a name="storsimple-data-protection"></a>Protection des données StorSimple

Cette section décrit les fonctionnalités de sécurité StorSimple visant à protéger les données transmises et stockées.

Comme mentionné dans les sections précédentes, les mots de passe sont utilisés pour autoriser et authentifier les utilisateurs et leur permettre d'accéder à votre solution StorSimple. La protection des données contre les utilisateurs non autorisés pendant leur transfert entre les systèmes de stockage et pendant leur stockage est également primordiale. Les sections suivantes décrivent les fonctionnalités de protection des données fournies par StorSimple.

> [!NOTE]
> La déduplication offre une protection supplémentaire pour les données stockées sur l’appareil StorSimple et dans le stockage Microsoft Azure. Lorsque les données sont dédupliquées, les objets de données sont stockés séparément des métadonnées utilisées pour les mapper et y accéder : aucun contexte au niveau du stockage n’est disponible pour reconstruire les données en fonction de la structure d’un volume, du système de fichiers ou du nom de fichier.


## <a name="protect-data-flowing-through-the-service"></a>Protection des données transitant par le service

L’objectif principal du service StorSimple Device Manager est de gérer et de configurer l’appareil StorSimple. Le service StorSimple Device Manager s’exécute dans Microsoft Azure. Le portail Azure vous permet d’entrer des données de configuration d’appareil, puis Microsoft Azure utilise le service StorSimple Device Manager pour envoyer ces données vers l’appareil. StorSimple utilise un système de paires de clés asymétriques pour garantir que la compromission du service Azure ne compromette pas les informations stockées.

![Chiffrement des données à la volée](./media/storsimple-security/DataEncryption.png)

Le système de clés asymétriques permet de protéger les données qui transitent par le service de la manière suivante :

1. Un certificat de chiffrement de données qui utilise une paire de clés publiques et privées asymétrique est généré sur l’appareil, puis il est utilisé pour protéger les données. Les clés sont générées lors de l’inscription du premier appareil.
2. Les clés du certificat de chiffrement de données sont exportées dans un fichier d’échange d’informations personnelles (.pfx), protégé par la clé de chiffrement des données de service, qui est une clé forte de 128 bits, générée de façon aléatoire par le premier appareil pendant l’inscription.
3. La clé publique du certificat est accessible en toute sécurité par le service StorSimple Device Manager et la clé privée est conservée dans l’appareil.
4. Les données entrant dans le service sont chiffrées à l’aide de la clé publique et déchiffrées à l’aide de la clé privée stockée sur l’appareil, afin de garantir que le service Azure ne peut pas déchiffrer les données envoyées vers l’appareil.

La clé de chiffrement des données du service est générée uniquement sur le premier appareil inscrit auprès du service. Tous les autres appareils inscrits auprès du service doivent utiliser la même clé de chiffrement.

> [!IMPORTANT]
> Il est très important de faire une copie de la clé de chiffrement des données du service et de l'enregistrer dans un emplacement sécurisé. La méthode de stockage de la copie de la clé de chiffrement des données de service doit permettre à toute personne autorisée d’y accéder et de la communiquer facilement à l’administrateur de l’appareil.
> 
> Si la clé de chiffrement est perdue, une personne du support technique Microsoft peut vous aider à la récupérer, sous réserve qu’au moins un appareil soit en ligne. Nous vous recommandons de modifier la clé de chiffrement des données de service après sa récupération.

Pour modifier la clé de chiffrement des données de service, et le certificat de chiffrement des données correspondant, suivez les étapes dans [Modifier la clé de chiffrement des données de service pour votre service Gestionnaire de périphériques StorSimple](storsimple-8000-manage-service.md#change-the-service-data-encryption-key). La modification des clés de chiffrement requiert que tous les appareils soient mis à jour avec la nouvelle clé. Par conséquent, nous vous recommandons de modifier la clé lorsque tous les appareils sont en ligne. Si des appareils sont hors connexion, leurs clés peuvent être modifiées plus tard. Les appareils avec des clés obsolètes seront toujours en mesure d’exécuter des sauvegardes, mais ils ne seront pas en mesure de restaurer les données, et ce tant que leur clé n’aura pas été mise à jour.

La clé de chiffrement des données du service et le certificat de chiffrement de données n’arrivent pas à expiration. Toutefois, nous vous recommandons de modifier la clé de chiffrement des données du service tous les ans pour des raisons de sécurité.

## <a name="protect-data-at-rest"></a>Protection des données au repos

L’appareil StorSimple gère les données en les stockant dans des couches locales et dans le cloud, en fonction de leur fréquence d’utilisation. Tous les ordinateurs hôtes connectés à l’appareil envoient des données à l’appareil, qui les déplace ensuite vers le cloud, si nécessaire. Les données sont transférées de l'appareil vers le cloud de façon sécurisée via Internet. Chaque périphérique possède une cible iSCSI qui couvre tous les volumes partagés sur cet appareil. Toutes les données sont chiffrées avant d’être envoyées vers le stockage cloud. 

![Clé de chiffrement de stockage cloud](./media/storsimple-security/CloudStorageEncryption.png)

Pour garantir la sécurité et l'intégrité des données déplacées vers le cloud, StorSimple vous permet de définir des clés de chiffrement de stockage cloud comme suit :

* Vous spécifiez la clé de chiffrement de stockage cloud lorsque vous créez un conteneur de volume. La clé ne peut pas être modifiée ou ajoutée ultérieurement.
* Tous les volumes d’un conteneur de volumes partagent la même clé de chiffrement. Si vous souhaitez utiliser une autre forme de cryptage pour un volume spécifique, nous vous recommandons de créer un nouveau conteneur de volumes pour héberger ce volume.
* Lorsque vous entrez la clé de chiffrement de stockage cloud dans le service StorSimple Device Manager, la clé est chiffrée à l’aide de la partie publique de la clé de chiffrement des données de service, puis envoyée à l’appareil.
* La clé de chiffrement de stockage cloud n’est pas stockée dans le service et est connue uniquement de l’appareil.
* La spécification d’une clé de chiffrement de stockage cloud est facultative. Vous pouvez envoyer des données chiffrées à l’hôte et à l’appareil.

### <a name="additional-security-best-practices"></a>Meilleures pratiques supplémentaires en matière de sécurité

* Fractionner le trafic : isolez votre SAN iSCSI du trafic utilisateur sur un réseau LAN d'entreprise en déployant un réseau totalement séparé et en utilisant des réseaux locaux virtuels où l'isolation physique n'est pas une option. Un réseau dédié pour le stockage iSCSI garantit la sécurité et les performances de vos données critiques. Le fait de mélanger le stockage et le trafic utilisateur sur un réseau LAN d'entreprise n'est pas recommandé et peut augmenter la latence, ainsi que provoquer des défaillances du réseau.
* Pour une sécurité réseau côté hôte, utilisez des interfaces réseau qui prennent en charge le moteur de déchargement TCP/IP (TOE). TOE réduit la charge de l'UC grâce au traitement de TCP sur la carte réseau.

## <a name="protect-data-via-storage-accounts"></a>Protection des données par les comptes de stockage

Chaque abonnement Microsoft Azure peut créer un ou plusieurs comptes de stockage. (Un compte de stockage fournit un espace de noms unique pour utiliser les données stockées dans le cloud Azure.) L’accès à un compte de stockage est contrôlé par l’abonnement et les clés d’accès associées à ce compte de stockage.

Lorsque vous créez un compte de stockage, Microsoft Azure génère deux clés d’accès de stockage 512 bits : l’une d’entre elles est utilisée pour l’authentification lorsque l’appareil StorSimple accède au compte de stockage. Notez que seule une des clés est utilisée. L’autre clé est gardée en réserve, ce qui permet une rotation régulière des clés. Pour ce faire, activez la clé secondaire, puis supprimez la clé primaire. Vous pouvez ensuite créer une nouvelle clé, à utiliser lors de la rotation suivante. (De nombreux centres de données ont recours à la rotation des clés pour des raisons de sécurité).

Nous vous recommandons de suivre ces méthodes recommandées pour la rotation des clés :

* Utilisez la rotation des clés de compte de stockage régulièrement pour vous assurer que votre compte de stockage n’est pas accessible aux utilisateurs non autorisés.
* Votre administrateur Azure doit changer ou régénérer régulièrement la clé primaire ou secondaire à l’aide de la section Stockage du portail Azure pour accéder directement au compte de stockage.

## <a name="protect-data-via-encryption"></a>Protection des données par chiffrement

StorSimple utilise les algorithmes de chiffrement suivants pour protéger les données stockées sur les composants ou en transit dans votre solution StorSimple.

| Algorithm | Longueur de clé | Protocoles/applications/commentaires |
| --- | --- | --- |
| RSA |2 048 |RSA PKCS 1 v1.5 est utilisé par le portail Azure pour chiffrer les données de configuration envoyées à l’appareil : par exemple, les informations d’identification du compte de stockage, la configuration de l’appareil StorSimple et les clés de chiffrement du stockage cloud. |
| AES |256 |AES avec CBC est utilisé pour chiffrer la partie publique de la clé de chiffrement des données du service avant son envoi au portail Azure à partir de l’appareil StorSimple. Il est également utilisé par l’appareil StorSimple pour chiffrer les données avant qu’elles ne soient envoyées au compte de stockage cloud. |

## <a name="storsimple-cloud-appliance-security"></a>Sécurité StorSimple Cloud Appliance

[!INCLUDE [storsimple Cloud Appliance security](../../includes/storsimple-virtual-device-security.md)]

## <a name="managing-personal-information"></a>Gestion des informations personnelles

StorSimple Device Manager pour les séries physiques et virtuelles collecte des informations personnelles principalement dans les cas suivants :

- Paramètres utilisateur d’alerte où l’adresse e-mail des utilisateurs est configurée. L’administrateur a la possibilité d’afficher et d’effacer ces données. Cela s’applique aux appareils StorSimple série 8000 et à StorSimple Virtual Array.
  * Pour afficher et effacer les paramètres de StorSimple série 8000, suivez les étapes de la page [Afficher et gérer les alertes StorSimple](storsimple-8000-manage-alerts.md#configure-alert-settings).
  * Pour afficher et effacer les paramètres de StorSimple Virtual Array, suivez les étapes de la page [Afficher et gérer les alertes StorSimple](storsimple-virtual-array-manage-alerts.md#configure-alert-settings).
- Utilisateurs qui ont accès aux données qui se trouvent sur les partages. La liste de ces utilisateurs s’affiche et peut être consultée. Elle est par ailleurs supprimée à la suppression des partages. Cela s’applique uniquement à StorSimple Virtual Arrays.
  * Pour afficher la liste des utilisateurs qui y ont accès ou supprimer un partage, suivez les étapes de la page [Gérer les partages sur StorSimple Virtual Array](storsimple-virtual-array-manage-shares.md).

Pour plus d’informations, consultez la [Politique de confidentialité Microsoft sur le Centre de gestion de la confidentialité](https://www.microsoft.com/trustcenter).

## <a name="frequently-asked-questions-faq"></a>Forum Aux Questions (FAQ)

Voici quelques questions et réponses relatives à la sécurité et à Microsoft Azure StorSimple.

**Q :** Mon service est compromis. Quelles doivent être mes étapes suivantes ?

**R :** Vous devez immédiatement modifier la clé de chiffrement des données du service et les clés du compte de stockage utilisé pour hiérarchiser les données. Pour obtenir des instructions, consultez :

* [Modification de la clé de chiffrement des données du service](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)
* [Rotation des clés de comptes de stockage](storsimple-8000-manage-storage-accounts.md#key-rotation-of-storage-accounts)

**Q :** J’ai un nouvel appareil StorSimple qui demande la clé d’inscription du service. Comment la récupérer ?

**R :** Cette clé a été créée quand vous avez créé le service StorSimple Device Manager. Si vous utilisez le service StorSimple Device Manager pour vous connecter à l’appareil, vous pouvez utiliser la page de démarrage rapide du service pour afficher ou régénérer la clé d’inscription. La génération d’une nouvelle clé d’inscription n’affecte pas les appareils déjà inscrits. Pour obtenir des instructions, consultez :

* [Affichage ou régénération de la clé d’inscription au service](storsimple-8000-manage-service.md#regenerate-the-service-registration-key)

**Q :** J’ai perdu ma clé de chiffrement des données du service. Que faire ?

**R :** Contactez le Support Microsoft. Il peut ouvrir une session de support sur votre appareil et vous aider à récupérer la clé (à condition qu’au moins un appareil soit connecté) Dès que vous avez obtenu la clé de chiffrement des données du service, vous devez la modifier pour vous assurer que vous seul la connaissez. Pour obtenir des instructions, consultez :

* [Modification de la clé de chiffrement des données du service](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)

**Q :**  J’ai autorisé un appareil pour un changement de clé de chiffrement des données du service, mais n’ai pas démarré le processus de changement de clé. Que dois-je faire ?

**R :** Si le délai d’expiration est dépassé, vous devez ré-autoriser l’appareil pour le changement de clé de chiffrement des données du service et redémarrer le processus.

**Q :**  J’ai changé la clé de chiffrement des données du service, mais je n’ai pas pu mettre à jour les autres appareils dans un délai de quatre heures. Dois-je recommencer ?

**R :** Le délai de quatre heures s’applique uniquement au lancement du changement. Une fois que vous avez démarré la mise à jour sur l’appareil StorSimple autorisé, l’autorisation est valide jusqu’à ce que tous les appareils soient mis à jour.

**Q :** Notre administrateur StorSimple a quitté l’entreprise. Que dois-je faire ?

**R :** Changez et réinitialisez les mots de passe qui permettent l’accès à l’appareil StorSimple, puis changez la clé de chiffrement des données du service pour vous assurer que les nouvelles informations ne seront pas accessibles au personnel non autorisé. Pour obtenir des instructions, consultez :

* [Utilisation du service StorSimple Device Manager pour modifier vos mots de passe StorSimple](storsimple-8000-change-passwords.md)
* [Modification de la clé de chiffrement des données du service](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)
* [Configuration de CHAP pour votre appareil StorSimple](storsimple-8000-configure-chap.md)

**Q :** Je souhaite fournir le mot de passe StorSimple Snapshot Manager à un hôte qui se connecte à l’appareil StorSimple, mais le mot de passe n’est pas disponible. Que puis-je faire ?

**R :** Si vous avez oublié le mot de passe, vous devez en créer un nouveau. Veillez ensuite à informer tous les utilisateurs existants que le mot de passe a été modifié et qu’ils doivent mettre à jour leur client pour utiliser le nouveau mot de passe. Pour obtenir des instructions, consultez :

* [Modification du mot de passe du Gestionnaire d’instantanés StorSimple](storsimple-8000-change-passwords.md#set-the-storsimple-snapshot-manager-password)
* [Authentification d’un appareil](storsimple-snapshot-manager-manage-devices.md#authenticate-a-device)

**Q :** Le certificat pour l’accès à distance à Windows PowerShell pour StorSimple a été changé sur l’appareil. Comment mettre à jour mes clients d’accès à distance ?

**R :** Vous pouvez télécharger le nouveau certificat à partir du service StorSimple Device Manager, puis l’installer dans le magasin de certificats de vos clients d’accès à distance. Pour obtenir des instructions, consultez :

* [Applet de commande Import-Certificate](/powershell/module/pkiclient/import-certificate)

**Q :** Mes données sont-elles protégées si le service StorSimple Device Manager est compromis ?

**R :** Les données de configuration du service sont toujours chiffrées avec votre clé publique quand vous les affichez dans un navigateur web. Comme le service n’a pas accès à la clé privée, il n’est pas en mesure d’accéder aux données. Si le service StorSimple Device Manager est compromis, il n’y a aucun impact, car aucune clé n’est stockée dans ce service.

**Q :** Si quelqu’un a accès au certificat de chiffrement des données, mes données seront-elles compromises ?

**R :** Microsoft Azure stocke la clé de chiffrement des données du client (fichier .pfx) dans un format chiffré. Comme le fichier .pfx est chiffré et que le service StorSimple ne dispose pas de la clé de chiffrement des données du service nécessaire pour déchiffrer le fichier .pfx, l’accès au fichier .pfx ne met pas vos données en danger.

**Q :** Que se passe-t-il si une entité gouvernementale demande mes données à Microsoft ?

**R :** Comme toutes les données sont chiffrées dans le service et que la clé privée est conservée sur l’appareil, l’entité gouvernementale doit demander les données au client.



## <a name="next-steps"></a>Étapes suivantes

[Déploiement de votre appareil StorSimple](storsimple-8000-deployment-walkthrough-u2.md).