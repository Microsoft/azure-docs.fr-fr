---
title: Diagnostic des problèmes de performances graphiques dans le Bureau à distance - Azure
description: Cet article décrit comment utiliser les compteurs graphiques RemoteFX dans les sessions de protocole Bureau à distance pour diagnostiquer les problèmes de performances liés aux graphiques dans Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 05/23/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: dc5c6499fa47a1e32a517032d5cc2a97b3f2677f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88007164"
---
# <a name="diagnose-graphics-performance-issues-in-remote-desktop"></a>Diagnostic des problèmes de performances graphiques dans le Bureau à distance

Pour diagnostiquer les problèmes de qualité de l’expérience dans vos sessions à distance, des compteurs sont disponibles sous la section Vidéo RemoteFX de l’Analyseur de performances. Cet article vous montre comment utiliser ces compteurs pour localiser et résoudre les goulots d’étranglement liés aux performances graphiques au cours de sessions RDP (Remote Desktop Protocol).

## <a name="find-your-remote-session-name"></a>Rechercher le nom de la session à distance

Vous aurez besoin du nom de la session à distance pour identifier les compteurs de performances graphiques. Suivez les instructions de cette section pour identifier votre instance de chaque compteur.

1. Ouvrez l’invite de commandes Windows à partir de votre session à distance.
2. Exécutez la commande **qwinsta** et localisez le nom de votre session.
    - Si votre session est hébergée dans une machine virtuelle multisession : L’instance de chaque compteur est suivie du même nombre que celui terminant le nom de votre session (par exemple : « rdp-tcp 37 »).
    - Si votre session est hébergée dans une machine virtuelle prenant en charge des processeurs virtuels (vGPU) : Votre instance de chaque compteur est stockée sur le serveur, et non sur votre machine virtuelle. Vos instances de compteur incluent le nom de la machine virtuelle, et non le nombre figurant dans le nom de la session (par exemple : « Win8 Enterprise VM »).

>[!NOTE]
> Tandis que les compteurs comportent le terme « RemoteFX » dans leur nom, ils incluent également des graphiques de bureau à distance dans les scénarios vGPU.

## <a name="access-performance-counters"></a>Accéder aux compteurs de performances

Une fois que vous avez déterminé le nom de votre session à distance, suivez ces instructions pour collecter les compteurs de performances RemoteFX Graphics de votre session à distance.

1. Sélectionnez **Démarrer** > **Outils d’administration** > **Analyseur de performances**.
2. Dans la boîte de dialogue **Analyseur de performances**, développez **Outils d’analyse**, sélectionnez **Analyseur de performances**, puis **Ajouter**.
3. Dans la liste **Compteurs disponibles** de la boîte de dialogue **Ajouter des compteurs**, développez la section Vidéo RemoteFX.
4. Sélectionnez les compteurs à analyser.
5. Dans la liste **Instances de l’objet sélectionné**, sélectionnez les instances spécifiques à superviser pour les compteurs sélectionnés, puis sélectionnez **Ajouter**. Pour sélectionner toutes les instances de compteur disponibles, sélectionnez **Toutes les instances**.
6. Après avoir ajouté les compteurs, sélectionnez **OK**.

Les compteurs de performances sélectionnés s’affichent sur l’écran de l’Analyseur de performances.

>[!NOTE]
>Chaque session active sur un ordinateur hôte possède sa propre instance pour chaque compteur de performances.

## <a name="diagnose-issues"></a>Diagnostiquer les problèmes

Les problèmes de performances graphiques se répartissent généralement en quatre catégories :

- Faible fréquence d’images
- Interruptions aléatoires
- Latence d’entrée élevée
- Qualité d’image médiocre

### <a name="addressing-low-frame-rate-random-stalls-and-high-input-latency"></a>Résolution des problèmes liés à la faible fréquence d’images, aux interruptions aléatoires et à la latence d’entrée élevée

Vérifiez tout d’abord le compteur Images de sortie/seconde. celui-ci mesure le nombre d’images accordées au client. Si cette valeur est inférieure au compteur Images d’entrée/seconde, les images sont ignorées. Pour identifier le goulot d’étranglement, utilisez les compteurs Images ignorées/seconde.

Il existe trois types de compteurs Images ignorées/seconde :

- Images ignorées/seconde (ressources serveur insuffisantes)
- Images ignorées/seconde (ressources réseau insuffisantes)
- Images ignorées/seconde (ressources client insuffisantes)

La valeur élevée d’un compteur Images ignorées/seconde signifie que le problème est lié à la ressource analysée par le compteur. Par exemple, si le client ne décode pas et ne présente pas les images à la vitesse à laquelle celles-ci sont fournies au serveur, le compteur Images ignorées/seconde (ressources client insuffisantes) sera élevé.

Si le compteur Images de sortie/seconde correspond au compteur Images d’entrée/seconde, bien qu’un temps de latence ou une interruption inhabituelle subsiste, le problème peut être lié à la durée moyenne de codage. Le codage est un processus synchrone qui se produit sur le serveur dans un scénario de session unique (vGPU) et sur la machine virtuelle dans un scénario multisession. La durée moyenne de codage doit être inférieure à 33 ms. Si tel est le cas et que des problèmes de performances subsistent, le problème peut être lié à l’application ou au système d’exploitation que vous utilisez.

Pour plus d’informations sur le diagnostic des problèmes liés aux applications, consultez [Compteurs de performances de retard d’entrée utilisateur](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/).

Étant donné que le protocole RDP (Remote Desktop Protocol) prend en charge une durée moyenne d’encodage de 33 ms, il prend en charge une fréquence d’images d’entrée pouvant atteindre 30 images/seconde. Notez que 33 ms est la fréquence d’images maximale prise en charge. Dans de nombreux cas, la fréquence d’images dont bénéficie l’utilisateur est inférieure, selon la fréquence à laquelle l’image est fournie au protocole RDP par la source. Par exemple, les tâches telles que la lecture d’une vidéo nécessitent une fréquence d’images d’entrée complète de 30 images/seconde. En revanche, les tâches moins gourmandes en ressources, comme la modification ponctuelle d’un document, nécessitent une fréquence d’images d’entrée par seconde nettement inférieure, qui ne dégrade aucunement l’expérience pour l’utilisateur.

### <a name="addressing-poor-frame-quality"></a>Résolution des problèmes de qualité d’image

Utilisez le compteur de qualité des images pour diagnostiquer les problèmes de qualité d’image. Ce compteur exprime la qualité de l’image de sortie sous forme de pourcentage de qualité de l’image source. La perte de qualité peut être due à RemoteFX, ou peut être inhérente à la source graphique. Si RemoteFX a provoqué une perte de qualité, cela peut être dû au fait que les ressources réseau ou serveur ne sont pas parvenues à envoyer un contenu de plus haute fidélité.

## <a name="mitigation"></a>Limitation des risques

Si les ressources serveur sont à l’origine du goulot d’étranglement, essayez l’une des approches suivantes pour améliorer les performances :

- Réduisez le nombre de sessions par hôte.
- Augmentez la mémoire et les ressources de calcul sur le serveur.
- Supprimez la résolution de la connexion.

Si les ressources réseau sont à l’origine du goulot d’étranglement, essayez l’une des approches suivantes pour améliorer la disponibilité du réseau par session :

- Réduisez le nombre de sessions par hôte.
- Utilisez un réseau à bande passante supérieure.
- Supprimez la résolution de la connexion.

Si les ressources client sont à l’origine du goulot d’étranglement, essayez l’une des approches suivantes pour améliorer les performances :

- Installez le client Bureau à distance le plus récent.
- Augmentez la mémoire et les ressources de calcul sur la machine cliente.

> [!NOTE]
> Le compteur Images sources/seconde n’est pas pris en charge. Pour l’instant, le compteur Images source/seconde affiche toujours 0.

## <a name="next-steps"></a>Étapes suivantes

- Pour créer une machine virtuelle Azure optimisée pour le GPU, consultez [Configurer l’accélération du GPU pour l’environnement Windows Virtual Desktop](configure-vm-gpu.md).
- Pour découvrir une vue d’ensemble de la résolution des problèmes et des procédures d’escalade, consultez l’article [Vue d’ensemble du dépannage, commentaires et support](troubleshoot-set-up-overview.md).
- Pour plus d’informations sur le service, consultez [Environnement Windows Virtual Desktop](environment-setup.md).
