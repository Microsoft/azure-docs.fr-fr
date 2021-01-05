---
title: Lecture d’enregistrements – Azure
description: Vous pouvez utiliser Live Video Analytics sur IoT Edge pour l’enregistrement vidéo continu, qui permet d’enregistrer des vidéos dans le cloud pendant des semaines ou des mois. Vous pouvez également limiter votre enregistrement aux clips qui présentent un intérêt, par le biais de l’enregistrement basé sur les événements. Cet article explique comment lire des enregistrements.
ms.topic: how-to
ms.date: 04/27/2020
ms.openlocfilehash: 6222d2c05b2fe05945d4bcbef6dbb0d64bd4726a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84260382"
---
# <a name="playback-of-recordings"></a>Lecture d’enregistrements 

## <a name="pre-read"></a>Lecture préalable  

* [Lecture de vidéo](video-playback-concept.md)
* [Enregistrement de vidéo continu](continuous-video-recording-concept.md)
* [Enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md)

## <a name="background"></a>Arrière-plan  

Vous pouvez utiliser Live Video Analytics sur IoT Edge pour l’[enregistrement de vidéo continu](continuous-video-recording-concept.md) (CVR), qui permet d’enregistrer des vidéos dans le cloud pendant des semaines ou des mois. Vous pouvez également limiter votre enregistrement aux clips qui présentent un intérêt, par le biais de l’[enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md) (EVR). 

Votre compte Media Services est lié à un compte Stockage Azure et, lorsque vous enregistrez une vidéo dans le cloud, le contenu est écrit sur une [ressource Media Services](terminology.md#asset). Media Services vous permet de [diffuser ce contenu en continu](terminology.md#streaming), soit une fois l’enregistrement terminé, soit pendant que l’enregistrement est en cours. Media Services vous fournit les capacités nécessaires pour diffuser des flux via les protocoles HLS ou MPEG-DASH aux appareils de lecture (clients). Pour plus d’informations, consultez l’article sur [la lecture vidéo](video-playback-concept.md). Vous pouvez utiliser les API Media Services pour générer les URL de diffusion en continu nécessaires, utilisées par les clients pour lire la vidéo et l’audio. Pour construire l’URL de diffusion en continu pour un élément multimédia, consultez [Créer un localisateur de streaming et générer des URL](../latest/create-streaming-locator-build-url.md). Une fois que l’URL de streaming a été créée pour un élément multimédia, vous pouvez et devez stocker l’association de l’URL avec la source vidéo (c’est-à-dire, la caméra) dans votre système de gestion de contenu.

Par exemple, lors de la diffusion en continu via HLS, l’URL de streaming ressemble à `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl).m3u8`. Pour MPEG-DASH, elle ressemble à `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=mpd-time-cmaf).mpd`.

Cela retourne un fichier manifeste, qui, entre autres choses, décrit la durée globale du flux qui est remis et indique s’il s’agit d’un contenu préenregistré ou d’un flux en direct.

### <a name="live-vs-vod"></a>En direct vs VoD  

Les protocoles de diffusion en continu, comme HLS et MPEG-DASH, ont été créés pour gérer des scénarios tels que la diffusion en continu de vidéos en direct, ainsi que la diffusion en continu de contenus à la demande/préenregistrés, comme des émissions TV et des films. Pour les vidéos en direct, les clients HLS et MPEG-DASH sont conçus pour commencer à lire à partir du moment « le plus récent ». Avec les films, cependant, les spectateurs s’attendent à pouvoir commencer au début de la vidéo et avancer/reculer s’ils le souhaitent. Les manifestes HLS et MPEG-DASH ont des indicateurs qui indiquent aux clients si la vidéo représente un stream en direct ou s’il s’agit d’un contenu préenregistré.
Ce concept s’applique également aux flux HLS et MPEG-DASH des ressources qui contiennent des vidéos enregistrées à l’aide de Live Video Analytics sur IoT Edge.

## <a name="browsing-and-selective-playback-of-recordings"></a>Exploration et lecture sélective des enregistrements  

Imaginez le scénario suivant : vous avez utilisé Live Video Analytics sur IoT Edge pour enregistrer des vidéos uniquement de 8 h à 10 h les jours d’école, pour l’ensemble de l’année scolaire. Ou peut-être que vous enregistrez des vidéos uniquement de 7 h à 19 h les jours fériés. Dans l’un ou l’autre de ces deux scénarios, lorsque vous tentez de parcourir et de lire votre enregistrement vidéo, vous avez besoin des éléments suivants :

* Un moyen de déterminer les dates/heures de vidéo pour lesquelles vous avez un enregistrement.
* Un moyen de sélectionner une partie (par exemple, de 9 h à 11 h le jour de l’an) de cet enregistrement pour la lire.

Media Services vous fournit une API de requête (availableMedia) pour résoudre le premier problème, et des filtres de plage horaire (startTime, endTime) pour traiter le second.

## <a name="query-api"></a>API de requête 

Lors de l’utilisation de CVR, les appareils de lecture (clients) ne peuvent pas demander de manifeste couvrant la lecture de l’intégralité de l’enregistrement.  Un enregistrement sur plusieurs années produirait un fichier manifeste trop volumineux pour la lecture. Il serait difficile de l’analyser en portions utilisables côté client.  Le client doit savoir quelles plages DateHeure comportent des données dans l’enregistrement afin de pouvoir effectuer des requêtes valides sans trop de conjectures. C’est là qu’intervient la nouvelle API de requête : les clients peuvent désormais utiliser cette API côté serveur pour découvrir du contenu.

La valeur de précision peut être l’une des suivantes : année, mois, jour ou complète (comme indiqué ci-dessous). 

|Precision|year|month|day|complet|
|---|---|---|---|---|
|Requête|`/availableMedia?precision=year&startTime=2018&endTime=2019`|`/availableMedia?precision=month& startTime=2018-01& endTime=2019-02`|`/availableMedia?precision=day& startTime=2018-01-15& endTime=2019-02-02`|`/availableMedia?precision=full& startTime=2018-01-15T10:08:11.123& endTime=2019-01-015T12:00:01.123`|
|response|`{  "timeRanges":[{ "start":"2018", "end":"2019" }]}`|`{  "timeRanges":[{ "start":"2018-03", "end":"2019-01" }]}`|`{  "timeRanges":[    { "start":"2018-03-01", "end":"2018-03-07" },    { "start":"2018-03-09", "end":"2018-03-31" }  ]}`|Réponse complète. S’il n’y avait aucune interruption, le début serait startTime, et la fin serait endTime.|
|Contraintes|&#x2022;startTime <= endTime<br/>&#x2022;Les deux doivent être au format AAAA, sinon cela retourne une erreur.<br/>&#x2022;Les valeurs peuvent être séparées par un nombre quelconque d’années.<br/>&#x2022;Les valeurs sont inclusives.|&#x2022;startTime <= endTime<br/>&#x2022;Les deux doivent être au format AAAA-MM, sinon cela retourne une erreur.<br/>&#x2022;Les valeurs peuvent être séparées d’un maximum de 12 mois.<br/>&#x2022;Les valeurs sont inclusives.|&#x2022;startTime <= endTime<br/>&#x2022;Les deux doivent être au format AAAA-MM-JJ, sinon cela retourne une erreur.<br/>&#x2022;Les valeurs peuvent être séparées d’un maximum de 31 jours.<br/>Les valeurs sont inclusives.|&#x2022;startTime < endTime<br/>&#x2022;Les valeurs peuvent être séparées d’un maximum de 25 heures.<br/>&#x2022;Les valeurs sont inclusives.|

#### <a name="additional-request-format-considerations"></a>Considérations supplémentaires relatives au format des requêtes

* Toutes les heures sont exprimées en UTC.
* Le paramètre de chaîne de requête de précision doit être spécifié.  
* Les paramètres de chaîne de requête startTime et endTime sont requis pour les valeurs de précision mois, jour et complète.  
* Les paramètres de chaîne de requête startTime et endTime sont facultatifs pour la valeur de précision année (aucun, l’un ou l’autre ou les deux sont pris en charge).  

    * Si startTime est omise, elle est supposée être la première année de l’enregistrement.
    * Si endTime est omise, elle est supposée être la dernière année de l’enregistrement.
    * Par exemple, si votre enregistrement a commencé en 2011 et s’est poursuivi jusqu’en 2020, alors :

        * /availableMedia?precision=l’année est la même que /availableMedia?precision=year&startTime=2011&endTime=2020
        * /availableMedia?precision=year&startTime=2015 est la même que /availableMedia?precision=year&startTime=2015&endTime=2020
        * /availableMedia?precision=year&endTime=2018 est la même que /availableMedia?precision=year&startTime=2011&endTime=2018

Si aucune donnée multimédia n’est disponible pour les intervalles de temps, une liste vide est retournée.

Si la ressource ne contient pas d’enregistrement d’un graphe multimédia, une réponse HTTP 400 est renvoyée avec un message d’erreur expliquant que cette fonctionnalité n’est disponible que pour le contenu enregistré via un graphe multimédia.

Si la durée spécifiée par les paramètres est supérieure à celle autorisée par l’intervalle de temps maximal pour un type de requête donné, un message d’erreur HTTP 400 est renvoyé avec un message d’erreur expliquant l’intervalle de temps maximal autorisé pour le type de requête demandé.

En supposant que l’intervalle de temps est valide pour les paramètres donnés, aucune erreur n’est renvoyée pour les requêtes qui sont en dehors de la fenêtre temporelle des données dans l’enregistrement.  Cela signifie que si l’enregistrement a démarré il y a 7 heures, mais que le client demande un média disponible de {UtcNow – 24 hours} à UtcNow, nous ne retournons que les heures de données {UtcNow – 7}.

### <a name="response-format"></a>Format de la réponse  

L’appel availableMedia retourne un jeu de valeurs de temps.

La réponse sera un corps JSON, les valeurs timeRanges étant un tableau de dates UTC à la norme ISO 8601 pour l’année (au format AAAA), le mois (au format AAAA-MM) ou le jour (AAAA-MM-JJ) selon la précision demandée.  Le timeRanges complet contiendra à la fois une valeur de début et de fin au format de date UTC ISO 8601 (au format AAAA-MM-JJThh:mm:ss.sssZ).

Pour la requête availableMedia, l’API se basera sur la chronologie de la vidéo. Toutes les discontinuités dans la chronologie s’affichent sous la forme d’intervalles dans la réponse.

#### <a name="response-example-for-availablemedia"></a>Exemple de réponse pour availableMedia

##### <a name="example-1-live-recording-with-no-gaps"></a>Exemple 1 : Enregistrement en direct sans intervalle

Voici une réponse qui indique tous les médias disponibles pour le 21 décembre 2019, où l’enregistrement est complet à 100 %. La réponse n’a qu’une seule paire de début/fin.

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
   {
         "start":"2019-12-21T00:00:00.000Z", 
         "end":"2019-12-22T00:00:00.000Z"
    }
   ]
}
```

##### <a name="example-2-live-recording-with-a-2-second-gap"></a>Exemple 2 : Enregistrement en direct avec un intervalle de 2 secondes

Supposons, pour une raison quelconque, que la caméra n’a pas pu envoyer de vidéo valide pendant un intervalle de 2 secondes sur une journée. Voici une réponse qui indique tous les médias disponibles pour le 21 décembre 2019 :

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:01:08Z"
    },
    {
         "start":"2019-12-21T04:01:10Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

##### <a name="example-3-live-recording-with-an-8-hour-gap"></a>Exemple 3 : Enregistrement en direct avec un intervalle de 8 heures

Supposons que la caméra ou l’installation locale soit hors tension pendant une période de 8 heures pendant la journée, de 4 h UTC à 12 h UTC, et il n’y avait aucune source d’alimentation de secours. Voici une réponse qui indique tous les médias disponibles pour une telle journée :

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:00:00Z"
    },
    {
         "start":"2019-12-21T12:00:00Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

#### <a name="example-4-live-recording-where-no-video-is-recorded-over-summer-holidays"></a>Exemple 4 : Enregistrement en direct où aucune vidéo n’est enregistrée pendant les vacances d’été

Supposons que vous avez enregistré la vidéo uniquement pendant la période scolaire et que l’enregistrement a été arrêté du 17 juin au 6 septembre. Une requête pour les mois disponibles se présente comme suit :

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=month&startTime=2019-01&endTime=2019-12
{
   "timeRanges":[
    {
         "start":"2019-01", 
         "end":"2019-06"
    },
    {
         "start":"2019-09", 
         "end":"2019-12"
    }
   ]
}
```

Si vous avez ensuite demandé les jours disponibles en juin, vous pouvez voir :

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=day&startTime=2019-06-01&endTime=2019-06-30
{
   "timeRanges":[
    {
         "start":"2019-06-01", 
         "end":"2019-06-17"
    }
   ]
}
```

##### <a name="example-5-live-recording-where-no-video-is-recorded-over-weekends-or-holidays"></a>Exemple 5 : Enregistrement en direct où aucune vidéo n’est enregistrée pendant les week-ends ou les vacances

Supposons que vous avez enregistré la vidéo uniquement pendant les heures ouvrables. Une requête pour les jours disponibles se présente comme suit :

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=day&startTime=2020-02-01&endTime=2020-02-29
{
   "timeRanges":[
    {
         "start":"2020-02-03", 
         "end":"2020-02-07"
    },
    {
         "start":"2020-02-10", 
         "end":"2020-02-14"
    },
    {
         "start":"2020-02-18", // Monday Feb 17th was a holiday
         "end":"2020-02-21"
    },
    {
         "start":"2020-02-24", 
         "end":"2020-02-28"
    }
   ]
}
```

## <a name="time-range-filters"></a>Filtre d’intervalle de temps

Comme indiqué ci-dessus, ces filtres vous aident à sélectionner des parties de votre enregistrement (par exemple, de 9 h à 11 h le jour de l’an) pour la lecture. Lors de la diffusion en continu via HLS, l’URL de streaming ressemble à `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl).m3u8`. Pour sélectionner une partie de votre enregistrement, vous devez ajouter un paramètre startTime et un paramètre endTime, par exemple : `https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T08:00:00Z,endTime=2019-12-21T10:00:00Z).m3u8`. Par conséquent, les filtres d’intervalle de temps sont des modificateurs d’URL utilisés pour décrire la partie de la chronologie de l’enregistrement qui est incluse dans le manifeste de diffusion en continu :

* `starttime` est un tampon DateHeure ISO 8601 qui décrit l’heure de début souhaitée de la chronologie vidéo retournée dans le manifeste.
* `endtime` est un tampon DateHeure ISO 8601 qui décrit l’heure de fin souhaitée de la chronologie vidéo retournée dans le manifeste.

La longueur maximale (dans le temps) d’un tel manifeste ne peut pas dépasser 24 heures.

Voici les contraintes sur les filtres.

|startTime| endTime |Résultats|
|---|---|---|
|Absent |Absent |Retourne la partie la plus récente de la vidéo dans la ressource, jusqu’à une durée maximale de 4 heures.<br/><br/>Si la ressource n’a pas été écrite (aucune donnée vidéo entrante) récemment, un manifeste à la demande (VoD) est retourné. Sinon, un manifeste en direct est retourné.|
|Présent|Absent|    Retourne un manifeste avec toute vidéo disponible qui est plus récente que startTime, si ce manifeste est inférieur à 24 heures.<br/>Si la longueur du manifeste dépasse 24 heures, il retourne une erreur.<br/>Si la ressource n’a pas été écrite (aucune donnée vidéo entrante) récemment, un manifeste à la demande (VoD) est retourné. Sinon, un manifeste en direct est retourné.
|Absent|Présent |Erreur : si startTime est présent, endTime est obligatoire.|
|Présent|Présent|Retourne un manifeste VoD avec toute vidéo disponible entre startTime et endTime.<br/>L’intervalle (startTime, endTime) ne peut pas dépasser 24 heures.|

### <a name="response-examples"></a>Exemples de réponse  

#### <a name="example-1-live-recording-with-no-gaps"></a>Exemple 1 : Enregistrement en direct sans intervalle

Voici une réponse qui indique tous les médias disponibles pour le 21 décembre 2019, où l’enregistrement est complet à 100 %. La réponse n’a qu’une seule paire de début/fin.

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
   {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

Lorsque l’enregistrement est continu, vous pouvez demander des manifestes HLS ou DASH pour n’importe quelle partie de temps au sein de cette paire de début/fin, à condition que l’intervalle soit inférieur ou égal à 24 heures. Un exemple de demande de manifeste HLS de 4 heures pour ce qui précède serait :

`GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T14:00:00.000Z,endTime=2019-12-21T18:00:00.000Z).m3u8`

#### <a name="example-2-live-recording-with-a-2-second-gap"></a>Exemple 2 : Enregistrement en direct avec un intervalle de 2 secondes

Supposons, pour une raison quelconque, que la caméra n’a pas pu envoyer de vidéo valide pendant un intervalle de 2 secondes sur une journée. Voici une réponse qui indique les médias disponibles pour le 21 décembre 2019 :

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:01:08Z"
    },
    {
         "start":"2019-12-21T04:01:10Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

Avec un enregistrement tel que celui ci-dessus, vous pouvez à nouveau demander des manifestes HLS et DASH avec n’importe quelle paire startTime/endTime, à condition que l’intervalle soit inférieur à 24 heures. Si ces valeurs chevauchaient l’intervalle à 04:01:08AM UTC, le manifeste retourné signalerait la discontinuité dans la chronologie du média, conformément aux spécifications appropriées pour ces protocoles.

#### <a name="example-3-live-recording-with-an-8-hour-gap"></a>Exemple 3 : Enregistrement en direct avec un intervalle de 8 heures

Supposons que la caméra ou l’installation locale soit hors tension pendant une période de 8 heures pendant la journée, de 4 h UTC à 12 h UTC, et il n’y avait aucune source d’alimentation de secours. Voici une réponse qui indique tous les médias disponibles pour une telle journée :

```
GET https://hostname/locatorId/content.ism/availableMedia?precision=full&startTime=2019-12-21T00:00:00Z&endTime=2019-12-22T00:00:00Z
{
   "timeRanges":[
    {
         "start":"2019-12-21T00:00:00Z", 
         "end":"2019-12-21T04:00:00Z"
    },
    {
         "start":"2019-12-21T12:00:00Z", 
         "end":"2019-12-22T00:00:00Z"
    }
   ]
}
```

Avec un tel enregistrement :

* Si vous demandez un manifeste où les deux filtres d’intervalle startTime/endTime se trouvent dans les parties « disponibles » de la chronologie, à savoir de 0 h à 4 h ou de midi à minuit, le service retourne un manifeste qui couvre la période entre startTime et endTime, par exemple :

    `GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T14:01:00.000Z,endTime=2019-12-21T03:00:00.000Z).m3u8`
* Si vous demandez un manifeste où les startTime et endTime se trouvent dans le « trou » au milieu, disons de 8 h à 10 h UTC, alors le service se comporte de la même façon que si un filtre de ressource aboutissait à un résultat vide.

    [Il s’agit d’une requête qui obtient une réponse vide] `GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T08:00:00.000Z,endTime=2019-12-21T10:00:00.000Z).m3u8`
* Si vous demandez un manifeste dans lequel un seul des paramètres startTime ou endTime se trouve dans le « trou », le manifeste retourné n’inclut qu’une partie de cette période. La valeur startTime ou endTime est ainsi ramenée à la limite valide la plus proche. Par exemple, si vous avez demandé un flux de 3 heures de 10 h à 13 h, la réponse contiendra 1 heure de média pour la période de 12 h à 13 h.

    `GET https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2019-12-21T10:00:00.000Z,endTime=2019-12-21T13:00:00.000Z).m3u8`
    
    Le manifeste retourné commencerait à 2019-12-21T12:00:00.000Z, même si la demande demandait de commencer à 10 h. Lorsque vous créez un système de gestion vidéo avec un plug-in de lecteur, vous devez veiller à le signaler au spectateur. Un spectateur non averti ne comprendrait pas pourquoi la chronologie de lecture commence à midi et non à 10 h comme demandé.

## <a name="recording-and-playback-latencies"></a>Latences d’enregistrement et de lecture

Lorsque vous utilisez Live Video Analytics sur IoT Edge pour l’enregistrement dans une ressource, vous devez spécifier une propriété segmentLength qui indique au module d’agréger une durée minimale de vidéo (en secondes) avant qu’elle ne soit enregistrée dans le cloud. Par exemple, si segmentLength est défini sur 300, le module accumule 5 minutes de vidéo avant de charger un « bloc » de 5 minutes, puis passe en mode accumulation pendant les 5 minutes suivantes, puis le charge à nouveau. L’augmentation de la segmentLength présente l’avantage de réduire les coûts de transaction de Stockage Azure, car le nombre de lectures et d’écritures n’est pas plus fréquent qu’une fois toutes les secondes segmentLength.

Par conséquent, la diffusion en continu de la vidéo à partir de Media Services sera retardée d’au moins autant de temps. 

Un autre facteur qui détermine la latence de lecture (retard entre le moment où un événement se produit devant la caméra et le moment où il peut être lu sur un périphérique de lecture) est la durée du groupe d’images [GOP](https://en.wikipedia.org/wiki/Group_of_pictures). Comme l’explique [la réduction du retard des streams en direct par l’utilisation de trois techniques simples](https://medium.com/vrt-digital-studio/reducing-the-delay-of-live-streams-by-using-3-simple-techniques-e8e028b0a641), plus la durée du GOP est longue, plus la latence est importante. Il est courant d’utiliser des caméras IP dans des scénarios de surveillance et de sécurité qui soient configurées pour utiliser des GOP de plus de 30 secondes. Cela a un impact important sur la latence globale.

## <a name="next-steps"></a>Étapes suivantes

[Didacticiel sur l’enregistrement de vidéo en continu](continuous-video-recording-tutorial.md)
