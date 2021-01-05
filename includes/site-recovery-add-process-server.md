---
author: rayne-wiselman
ms.service: site-recovery
ms.topic: include
ms.date: 10/26/2018
ms.author: raynew
ms.openlocfilehash: 31e61069c95be9bd1c7a684bb83ebcd93bcb14be
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019133"
---
1. Lancer le fichier UnifiedSetup.exe d’Azure Site Recovery
2. Dans la zone **Avant de commencer**, sélectionnez **l’ajout de serveurs de traitement supplémentaires pour effectuer un déploiement avec scale-out**.

   ![Ajouter un serveur de traitement](./media/site-recovery-add-process-server/ps-page-1.png)

3. Dans **Détails du serveur de configuration**, spécifiez l’adresse IP du serveur de configuration et la phrase secrète.

   ![Ajouter un serveur de traitement 2](./media/site-recovery-add-process-server/ps-page-2.png)
4. Dans **Paramètres Internet**, indiquez de quelle manière le fournisseur qui s’exécute sur le serveur de configuration doit se connecter à Azure Site Recovery par le biais d’Internet.

   ![Ajouter un serveur de traitement 3](./media/site-recovery-add-process-server/ps-page-3.png)

   * Si vous voulez vous connecter avec le proxy actuellement configuré sur la machine, sélectionnez **Se connecter avec les paramètres de proxy existants**.
   * Si vous voulez que le fournisseur se connecte directement, sélectionnez **Se connecter directement sans proxy**.
   * Si le proxy existant nécessite une authentification, ou si vous voulez utiliser un proxy personnalisé pour la connexion au fournisseur, sélectionnez **Se connecter avec des paramètres de proxy personnalisés**.

     * Si vous utilisez un proxy personnalisé, vous devez spécifier l’adresse, le port et les données d’identification.
     * Si vous utilisez un proxy, vous devez avoir déjà autorisé l’accès aux URL du service.

5. Dans **Vérification de la configuration requise**, le programme d’installation procède à une vérification afin de garantir le bon déroulement de l’installation. Si un avertissement s’affiche à propos de la **vérification de la synchronisation globale**, vérifiez que l’heure de l’horloge système (paramètres **Date et heure**) est identique à celle du fuseau horaire.

     ![Ajouter un serveur de traitement 4](./media/site-recovery-add-process-server/ps-page-4.png)

6. Dans **Détails de l’environnement**, indiquez si vous voulez répliquer des machines virtuelles VMware. Si c’est le cas, le programme d’installation vérifie que PowerCLI 6.0 est installé.

     ![Ajouter un serveur de traitement 5](./media/site-recovery-add-process-server/ps-page-5.png)

7. Dans **Emplacement d’installation**, sélectionnez l’emplacement où vous voulez installer les fichiers binaires et stocker le cache. Le lecteur sélectionné doit présenter au moins 5 Go d’espace disque disponible. Toutefois, nous vous recommandons d’utiliser un lecteur de cache présentant au moins 600 Go d’espace disponible.
     ![Capture d’écran montrant où sont installés les fichiers binaires et le stockage du cache](./media/site-recovery-add-process-server/ps-page-6.png)

8. Dans **Sélection du réseau**, spécifiez l’écouteur (carte réseau et port SSL) à partir duquel le serveur de configuration envoie et reçoit les données de réplication. Le port 9443 est le port utilisé par défaut pour envoyer et recevoir le trafic de réplication, mais vous pouvez le modifier en fonction des exigences de votre environnement. Outre le port 9443, nous ouvrons également le port 443, qui est utilisé par un serveur web pour orchestrer les opérations de réplication. N’utilisez pas le port 443 pour envoyer ou recevoir le trafic de réplication.

     ![Ajouter un serveur de traitement 6](./media/site-recovery-add-process-server/ps-page-7.png)
9. Dans **Résumé**, passez en revue les informations, puis cliquez sur **Installer**. Une fois l’installation terminée, une phrase secrète est générée. Vous en aurez besoin lorsque vous activerez la réplication. Par conséquent, copiez-la et conservez-la en lieu sûr.

     ![Ajouter un serveur de traitement 7](./media/site-recovery-add-process-server/ps-page-8.png)
