---
title: Configurer une interface utilisateur hébergée pour le service Recherche personnalisée Bing | Microsoft Docs
titleSuffix: Azure Cognitive Services
description: Cet article vous explique comment configurer et intégrer une interface utilisateur hébergée pour le service Recherche personnalisée Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: conceptual
ms.date: 02/12/2019
ms.author: aahi
ms.custom: devx-track-js
ms.openlocfilehash: ca3f9da681c60608b5b196b17191f0b4d549f305
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338365"
---
# <a name="configure-your-hosted-ui-experience"></a>Configurer votre expérience d’interface utilisateur hébergée

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Le service Recherche personnalisée Bing fournit une interface utilisateur hébergée que vous pouvez facilement intégrer à vos applications et pages web en tant qu’extrait de code JavaScript. À l’aide du portail Recherche personnalisée Bing, vous pouvez configurer la mise en page, la couleur et les options de recherche de l’interface utilisateur.



## <a name="configure-the-custom-hosted-ui"></a>Configuration de l’interface utilisateur hébergé personnalisée

Pour configurer une interface utilisateur hébergée pour vos applications web, procédez comme suit. Lorsque vous apportez des modifications, le volet de droite vous fournira un aperçu de votre interface utilisateur. Les résultats de recherche qui sont affichés ne sont pas les résultats réels de votre instance.

1. Connectez-vous au [portail](https://customsearch.ai) de la Recherche personnalisée Bing.  
  
2. Sélectionnez votre instance de Recherche personnalisée Bing.

3. Cliquez sur l’onglet **Hosted UI** (IU hébergée).  
  
4. Sélectionnez une disposition.

    - Barre de recherche et résultats (par défaut) : Affiche une zone de recherche avec les résultats de la recherche en dessous.
    - Résultats uniquement : Affiche uniquement les résultats de la recherche, sans zone de recherche. Lorsque vous utilisez cette mise en page, vous devez fournir la requête de recherche (`&q=<query string>`). Ajoutez le paramètre de requête à l’URL de requête dans l’extrait de code JavaScript, ou le lien vers le point de terminaison HTML.
    - Fenêtre superposée glissante : Fournit une zone de recherche et affiche les résultats de recherche dans une fenêtre glissante qui vient se superposer à la fenêtre actuelle.

5. Sélectionnez un thème de couleur. Vous pouvez personnaliser les couleurs en fonction de votre application en cliquant sur **Customize theme** (Personnaliser le thème). Pour modifier une couleur, entrez la valeur hexadécimale de couleur RVB (par exemple, `#366eb8`), ou cliquez sur l’aperçu de la couleur.

   Vous pouvez consulter un aperçu de vos modifications sur le côté droit du portail. En cliquant sur **Reset to default** (Rétablir les valeurs par défaut), vous rétablissez les couleurs par défaut du thème sélectionné.

   > [!NOTE]
   > Lors du choix des couleurs, pensez à l’accessibilité.

6. Sous **Additional Configurations** (Configurations supplémentaires), fournissez des valeurs en fonction de votre application. Ces paramètres sont facultatifs. Pour voir l’effet de leur application ou de leur suppression, regardez le volet de visualisation situé à droite. Voici les options de configuration disponibles :  

7. Entrez la clé d’abonnement de la recherche, ou sélectionnez-en une dans la liste déroulante. La liste déroulante est remplie avec les clés des abonnements de votre compte Azure. Consultez [Compte d'API Cognitive Services](../cognitive-services-apis-create-account.md).  

8. Si vous avez activé la suggestion automatique, entrez la clé d’abonnement de la suggestion automatique, ou sélectionnez-en une dans la liste déroulante. La liste déroulante est remplie avec les clés des abonnements de votre compte Azure. La Suggestion automatique personnalisée nécessite un niveau d’abonnement spécifique. Pour plus d’informations, consultez les [Tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/).

[!INCLUDE [publish or revert](./includes/publish-revert.md)]

## <a name="consume-custom-ui"></a>Consommer l’interface utilisateur personnalisée

Pour consommer l’IU hébergée, soit vous : 

- Incluez le script dans votre page web  
  
  ```html
  <html>
      <body>
          <script type="text/javascript" 
              id="bcs_js_snippet"
              src="https://ui.customsearch.ai /api/ux/rendering-js?customConfig=<YOUR-CUSTOM-CONFIG-ID>&market=en-US&safeSearch=Moderate&version=latest&q=">
          </script>
      </body>    
  </html>
  ```

- Vous pouvez également utiliser l’URL suivante dans un navigateur web.   
  
  `https://ui.customsearch.ai/hosted?customConfig=YOUR-CUSTOM-CONFIG-ID`  
  
  > [!NOTE]
  > Ajoutez les paramètres de requête suivants à l’URL selon vos besoins. Pour plus d’informations sur ces paramètres, consultez la référence [API Recherche personnalisée](/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference#query-parameters).
  >
  > - q
  > - mkt
  > - safesearch
  > - setlang

  > [!IMPORTANT]
  > La page ne peut pas afficher votre déclaration de confidentialité, ou d’autres mentions et conditions. L’adéquation à votre utilisation peut varier.  

Pour obtenir plus d’informations, y compris votre ID de configuration personnalisée, accédez à **Endpoints** (Points de terminaison) sous l’onglet **Production** (Production).

## <a name="configuration-options"></a>Options de configuration

Vous pouvez configurer le comportement de votre interface utilisateur hébergée en cliquant sur **Configurations supplémentaires** et en fournissant les valeurs de votre choix. Ces paramètres sont facultatifs. Pour voir l’effet de leur application ou de leur suppression, regardez le volet de visualisation situé à droite. 

### <a name="web-search-configurations"></a>Configurations de la recherche web :

- Résultats web activés : Détermine si la recherche sur le Web est activée (si c’est le cas, vous voyez l’onglet Web en haut de la page).
- Activer la suggestion automatique : Détermine si la suggestion automatique personnalisée est activée (voir les [Tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/) pour connaître les frais supplémentaires).
- Résultats web par page : Nombre de résultats à afficher pour une recherche sur le Web (le maximum est de 50 résultats par page).
- Légende d’image : Détermine si des images sont affichées avec les résultats de la recherche

Les configurations suivantes sont affichées si vous cliquez sur **Show advanced configurations** (Afficher les configurations avancées).

- Mettre les mots en surbrillance : Détermine si les résultats sont affichés avec les termes de la recherche en gras.
- Cible du lien : Détermine si la page web s’ouvre dans un nouvel onglet de navigateur (vide) ou dans le même onglet de navigateur lorsque l’utilisateur clique sur un résultat de recherche.

### <a name="image-search-configurations"></a>Configurations de la recherche d’images

- Résultats d’images activés : Détermine si la recherche d’images est activée (si c’est le cas, vous voyez l’onglet Images en haut de la page).
- Résultats d’images par page : Nombre de résultats à afficher pour une recherche d’images (le maximum est de 150 résultats par page).

La configuration suivante est affichée si vous cliquez sur **Show advanced configurations** (Afficher les configurations avancées).  
  
- Activer les filtres : Ajoute des filtres dont l’utilisateur peut se servir pour filtrer les images retournées par Bing. Par exemple, l’utilisateur peut filtrer les résultats pour n’afficher que des fichiers GIF animés.

### <a name="video-search-configurations"></a>Configurations pour la recherche de vidéos

- Résultats de vidéos activés : Détermine si la recherche de vidéos est activée (si c’est le cas, vous voyez l’onglet Vidéos en haut de la page).
- Résultats de vidéos par page : Nombre de résultats à afficher pour une recherche de vidéo (le maximum est de 150 résultats par page).

La configuration suivante est affichée si vous cliquez sur **Show advanced configurations** (Afficher les configurations avancées).  
  
- Activer les filtres : Ajoute des filtres dont l’utilisateur peut se servir pour filtrer les vidéos retournées par Bing. Par exemple, l’utilisateur peut filtrer les résultats pour n’afficher que les vidéos ayant une résolution spécifique ou les vidéos découvertes au cours des dernières 24 heures.

### <a name="miscellaneous-configurations"></a>Configurations diverses

- Titre de la page : Texte affiché dans la zone de titre de la page des résultats de recherche (ne s’applique pas à la disposition de fenêtre superposée glissante).
- Thème de la barre d’outils : Détermine la couleur d’arrière-plan de la zone de titre de la page des résultats de recherche.

Les configurations suivantes sont affichées si vous cliquez sur **Show advanced configurations** (Afficher les configurations avancées).  

|Colonne1  |Colonne2  |
|---------|---------|
|Search box text placeholder (Espace réservé au texte de la zone de recherche)   | Texte affiché dans la zone de recherche avant l’entrée de texte.        |
|Title link url (URL du lien de titre)    |Cible du lien vers le titre.         |
|URL du logo     | Image affichée à côté du titre.         |
|Favicon (Icône favorite)    | Icône affichée dans la barre de titre du navigateur.          |

Les configurations suivantes s’appliquent uniquement si vous utilisez l’interface utilisateur hébergée via le point de terminaison HTML (elles ne s’appliquent pas si vous utilisez l’extrait de code JavaScript).

- Titre de la page
- Thème de la barre d’outils
- URL du lien de titre
- URL du logo
- URL d’icône favorite  

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser des marqueurs d’ornement pour mettre en surbrillance du texte](../bing-web-search/hit-highlighting.md)
- [Paginer des pages web](../bing-web-search/paging-search-results.md)