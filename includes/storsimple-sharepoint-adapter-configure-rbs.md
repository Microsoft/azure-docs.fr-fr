---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 45b2ac73ffcb4b777572f9ad62d8aae0e8277e1c
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95560946"
---
> [!NOTE]
> Lorsque vous modifiez la configuration RBS de l’adaptateur StorSimple pour SharePoint, vous devez être connecté avec un compte utilisateur appartenant au groupe Domain Admins. En outre, vous devez accéder à la page de configuration à partir d’un navigateur s’exécutant sur le même hôte que l’Administration centrale.
> 
> 

#### <a name="to-configure-rbs"></a>Configuration de RBS
1. Ouvrez la page Administration centrale de SharePoint et accédez à **Paramètres du système**. 
2. Dans la section **Azure StorSimple**, cliquez sur **Configuration de l’adaptateur StorSimple**.
   
    ![Configuration de l’adaptateur StorSimple](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS1-include.png) 
3. Sur la page **Configuration de l’adaptateur StorSimple** :
   
   1. Assurez-vous que la case à cocher **Activer la modification du chemin d’accès** est activée.
   2. Dans la zone de texte, tapez le chemin d’accès UNC (Universal Naming Convention) du magasin d’objets BLOB.
      
      > [!NOTE]
      > Le volume de stockage d’objets BLOB doit être hébergé sur un volume iSCSI configuré sur l’appareil StorSimple.

   3. Cliquez sur le bouton **Activer** sous chaque base de données de contenu que vous souhaitez configurer pour le stockage à distance.
      
      > [!NOTE]
      > Le magasin d’objets BLOB doit être partagé par tous les serveurs web frontaux (WFE) et le compte d’utilisateur qui est configuré pour la batterie de serveurs SharePoint doit avoir accès au partage.
      
      ![Activation du fournisseur RBS](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS2-include.png)
      
      Lorsque vous activez ou désactivez RBS, vous verrez également le message suivant.
      
      ![Désactivation activation Configuration de l’adaptateur StorSimple](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_ConfigureStorSimpleAdapterEnableDisableMessage-include.png)

   4. Cliquez sur le bouton **Mettre à jour** pour appliquer la configuration. Quand vous cliquez sur le bouton **Mettre à jour** , l’état de configuration de RBS est actualisé sur tous les serveurs web frontaux (WFE) et la batterie entière est activée pour RBS. Le message suivant s’affiche.
      
      ![Message de configuration de l’adaptateur](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS3-include.png)
      
      > [!NOTE]
      > Si vous configurez RBS pour une batterie de serveurs SharePoint avec un très grand nombre de bases de données (plus de 200), la page web de l’Administration centrale de SharePoint peut expirer. Si cela se produit, actualisez la page. Cela n’affecte pas le processus de configuration.

4. Vérifiez la configuration :
   
   1. Connectez-vous au site web de l’Administration centrale de SharePoint, puis accédez à la page **Configuration de l’adaptateur StorSimple**.
   2. Vérifiez les détails de configuration pour vous assurer qu’ils correspondent aux paramètres que vous avez saisis. 
5. Vérifiez que RBS fonctionne correctement :
   
   1. Chargez un document dans SharePoint. 
   2. Recherchez le chemin d’accès UNC que vous avez configuré. Assurez-vous que la structure de répertoire RBS a été créée et qu’elle contient l’objet téléchargé.
6. (Facultatif) Vous pouvez utiliser l’applet de commande Microsoft RBS `Migrate()` PowerShell inclus avec SharePoint pour migrer le contenu des objets BLOB existants vers l’appareil StorSimple. Pour plus d’informations, consultez [Migration du contenu vers ou à partir de RBS dans SharePoint 2013][6] ou [Migration du contenu vers ou à partir de RBS (SharePoint Foundation 2010)][7].
7. (Facultatif) Sur les installations de test, vous pouvez vérifier que les objets BLOB ont été déplacés hors de la base de données de contenu comme suit : 
   
   1. Démarrez SQL Management Studio.
   2. Exécutez la requête ListBlobsInDB_2010.sql ou ListBlobsInDB_2013.sql, comme suit.
      
      ```
      **ListBlobsInDB_2013.sql**
      
        USE WSS_Content
        GO
      
        SELECT DocStreams.DocId,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               DocStreams.RbsId,
               TimeLastModified
      
        FROM DocStreams
      
             INNER JOIN AllDocs ON DocStreams.DocId = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      
      **ListBlobsInDB_2010.sql**
      
        USE WSS_Content
        GO
      
        SELECT AllDocStreams.Id,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               RbsId,
               TimeLastModified
        FROM AllDocStreams
      
             INNER JOIN AllDocs ON AllDocStreams.Id = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      ```
      
      Si RBS est correctement configuré, une valeur NULL doit apparaître dans la colonne SizeOfContentInDB pour un objet qui a été téléchargé et effectivement externalisé avec RBS.
8. (Facultatif) Une fois que vous configurez RBS et déplacez tout le contenu des objets BLOB vers l’appareil StorSimple, vous pouvez déplacer la base de données de contenu sur l’appareil. Si vous décidez de déplacer la base de données de contenu, nous vous recommandons de configurer le stockage de base de données de contenu sur l’appareil en tant que volume principal. Ensuite, utilisez les meilleures pratiques de SQL Server pour faire migrer la base de données de contenu vers l’appareil StorSimple. 
   
   > [!NOTE]
   > Le déplacement de la base de données de contenu sur l’appareil est uniquement pris en charge pour la série StorSimple 8000 (aucune prise en charge pour les séries 5000 ou 7000).
   
   Si vous stockez des objets BLOB et la base de données contenu dans des volumes distincts sur l’appareil StorSimple, nous vous recommandons de les configurer dans le même conteneur de volume. Cela garantit qu’ils seront sauvegardés ensemble.
   
   > [!WARNING]
   > Si vous n’avez pas activé RBS, il est déconseillé de déplacer la base de données de contenu sur l’appareil StorSimple. Il s'agit d'une configuration non testée.
   
9. Passez à l’étape suivante : [Configurer le nettoyage de la mémoire](#configure-garbage-collection).

[6]: /SharePoint/administration/migrate-content-into-or-out-of-rbs
[7]: /previous-versions/office/sharepoint-foundation-2010/ff628255(v=office.14)