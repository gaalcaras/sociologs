---
layout: post
title:  "Une approche pratique des archives en ligne"
permalink: /approche-pratique-archives-en-ligne/
date:   2016-12-22 18:01:27 +0100
categories: jekyll update
thumbnail: images/archive.jpg
summary: >
  Les activités en ligne sont de plus en plus mobilisées comme matériaux d'enquête
  en sociologie. Mais leur volatilité et leur abondance en font des ressources difficiles à
  manipuler.
---

Les activités en ligne deviennent rapidement une source majeure de matériaux d'enquête pour la sociologie.
Des forums aux billets de blog, des échanges d'emails aux messages des réseaux sociaux, ces sources de données prolongent − et, parfois, renouvellent − les problématiques et les méthodes traditionnelles des sciences sociales[^beuscart].

Malheureusement, un problème concret s'est posé à moi avant même d'arriver à ces problématiques : comment gérer efficacement ce matériau ?

J'étudie notamment une liste de diffusion publique, la LKML[^lkml], où des milliers de personnes ont échangé plus de 2,5 millions d'emails entre 1995 et 2016.
Après avoir lu une centaine d'emails intéressants, et avec la perspective d'en analyser encore plusieurs centaines, j'ai rapidement compris que chaque email supplémentaire rendrait la tâche d'autant plus laborieuse si je me contentais de prendre des notes sur un carnet, soit-il numérique.

En continuant avec le même procédé, il me fallait choisir entre perdre un temps considérable ou être moins rigoureux dans le traitement de ces emails.
Étant donnée cette alternative insatisfaisante, j'ai commencé à réfléchir à une approche pratique qui serait attentive à trois aspects du problème :

1. Comment relever et commenter les emails pertinents pour mon sujet ?
2. Comment conserver un email, sachant que les archives en ligne ne sont pas immortelles ?
3. Comment les citer efficacement lors de la rédaction ?

C'est cette approche pratique que je vais détailler dans ce billet.
J'ai parlé de la LKML plus haut, mais j'utiliserai l'archive [MARC](http://marc.info) comme exemple dans ce billet, car elle est plus simple à implémenter.
Ce système peut être adapté pour toutes les activités en ligne publiques, accessibles avec une URL et dont le formatage est cohérent, comme un forum, des tweets, des pages de blog, etc.

# L'objectif : archiver un email en un clic

Avant d'entrer dans les détails, commençons par donner une vue d'ensemble de ce système.

<center>
![Le système](/assets/img/zotero-archives-numeriques/screencast.gif)

Notre solution en action
</center>

Pour archiver un email, il nous suffit d'aller sur la page correspondant à un email et de cliquer sur un bouton dans la barre de notre navigateur.
Et... c'est tout !
Les métadonnées (auteur, date d'envoi, titre du mail, etc.) sont automatiquement extraites de la page, une copie exacte de la page est téléchargée, la date de consultation de la page est enregistrée.
À partir de là, nous pouvons garder une trace précise des emails consultés et les citer automatiquement dans le logiciel de notre choix.

## Comment ça marche ?

Nous allons utiliser le logiciel [Zotero](https://www.zotero.org/), que vous utilisez peut-être déjà pour gérer vos bibliographies, en exploitant deux de ses caractéristiques.
D'une part, Zotero donne la possibilité de récupérer simplement des ressources bibliographiques, comme des articles scientifiques, directement depuis son navigateur.
D'autre part, il est [*open source*](https://opensource.org/osd) et peut être facilement modifié par chacun·e.

Pour mettre en place notre système, nous tirerons donc parti des fonctionnalités de Zotero en étendant ses capacités de récupération de ressources afin qu'il reconnaisse notre source de données.
Concrètement, le logiciel se fonde sur un système de *translators* (littéralement, « traducteurs ») : à chaque source bibliographique correspond un *translator*, qui n'est ni plus ni moins qu'un simple fichier.
Il est possible de consulter le *translator* pour [Cairn](https://github.com/zotero/translators/blob/master/Cairn.info.js), pour [Amazon](https://github.com/zotero/translators/blob/master/Amazon.js), pour [Wikipedia](https://github.com/zotero/translators/blob/master/Wikipedia.js) − pour [le millier de translators](https://github.com/zotero/translators) fournis d'office en réalité.

## L'ampleur de la tâche

Notre objectif se précise : nous devons écrire un *translator* pour Zotero, qui n'est ni plus ni moins qu'un fichier de quelques dizaines de lignes.

La quantité de travail à fournir est donc très faible.
En revanche, le coût d'entrée dans le sujet est relativement élevé.
Les *translators* sont des fichiers rédigés dans le langage de programmation [JavaScript](https://www.javascript.com/) et requièrent le plus souvent de rédiger des requêtes en syntaxe [XPath](https://en.wikipedia.org/wiki/XPath).
Enfin, être familier avec les [expressions régulières](https://fr.wikipedia.org/wiki/Expression_rationnelle) (ou Regex) peut s'avérer utile.

Si les termes peuvent paraître effrayants, les barrières à l'entrée sont moins hautes qu'il n'y paraît.
Il n'est pas nécessaire d'être autonome en JavaScript pour créer un *translator*, loin de là.
Quant aux requêtes XPath, il existe de [nombreuses](https://en.wikipedia.org/wiki/XPath) [ressources](https://openclassrooms.com/courses/structurez-vos-donnees-avec-xml/xpath-localiser-les-donnees) [en ligne](http://edutechwiki.unige.ch/fr/Tutoriel_XPath).
Grâce aux contributions d'[Alexandre Hobeika](http://data.hypotheses.org/author/ahobeika) sur le blog Data Sciences Sociales, il existe même [plusieurs](http://data.hypotheses.org/959) [introductions](http://data.hypotheses.org/1043) francophones aux Regex pour les néophytes en sciences sociales.
Enfin, la [documentation de Zotero](https://www.zotero.org/support/dev/translators) sur la création d'un *translator* vous donnera de bonnes bases.

Nous voilà donc confronté·e·s au fameux arbitrage utilitariste de l'automatisation : le gain de temps dû à l'automatisation de la tâche compensera-t-il le temps passé à la mettre en place ?

<center>
![Xkcd: automation](https://imgs.xkcd.com/comics/automation.png)

Automation @ [Xkcd](https://xkcd.com/1319/)
</center>

En l'occurrence, les questions suivantes peuvent s'avérer utiles pour prendre une décision.

+ Combien d'archives numériques avez-vous besoin de collecter ?
+ Est-ce un besoin ponctuel ou allez-vous archiver des ressources numériques régulièrement durant les années à venir ?
+ Pensez-vous collecter des données en ligne pour d'autres utilisations, par exemple quantitatives ?
  Dans ce cas, il y a de fortes chances pour qu'apprendre la syntaxe des requêtes XPath vous serve dans de futurs projets.
+ Existe-t-il déjà un *translator* pour votre source de données ?
  Vérifiez que Zotero ou un contributeur extérieur n'a pas déjà codé un *translator* auparavant (c'est le cas pour [Twitter](https://github.com/zotero/translators/blob/master/Twitter.js) par exemple) que vous pourriez reprendre tel quel, voire adapter légèrement à vos propres besoins.

Dans mon cas, je collecte des centaines d'emails de plusieurs sources différentes dans le cadre de mes recherches, et je connais plutôt bien JavaScript et XPath, j'ai donc décidé de me lancer.
Je vais essayer de détailler autant que possible mon approche, sans pour autant rédiger une introduction complète à JavaScript et XPath.

# Création d'un *translator* : exemple suivi

Avant d'entrer dans le vif du sujet, assurez-vous que vous avez installé [Firefox](https://firefox.com), [Zotero](https://www.zotero.org/) et [Scaffold](https://github.com/zotero/scaffold/releases/latest), une extension au format `.xpi` pour Firefox ([documentation](https://www.zotero.org/support/dev/translators/scaffold)), qui permet d'éditer, tester et débugger des *translators*.

## Analyser une page type

La première chose consiste à trouver une page d'où extraire les métadonnées.
Dans ce tutoriel, je prends [cet email](https://marc.info/?l=git&m=111490299628887&w=2) comme exemple.

![Notre email](/assets/img/zotero-archives-numeriques/email.png)

<center>Notre email de référence</center>

Commencez par repérer toutes les métadonnées utiles que vous pouvez extraire de la page :

| Champ | Valeur |
| ----- | ------ |
| Titre | **Re: Trying to use AUTHOR_DATE** |
| Publication | **git** - Mailing list ARChive |
| Auteur | **Linus Torvalds** |
| URL | **https://marc.info/?l=git&m=111490299628887&w=2**|
| Date | **2005-04-30** |
| Date de consultation | date du jour |
| Type | email |
| Langue | en |

<center>Valeurs à extraire de la page</center>

J'ai noté en gras les éléments qui proviennent directement de la page web.
Bonne nouvelle : nous n'avons finalement très peu d'éléments à extraire, à peine 5.

Nous pouvons maintenant lancer Scaffold (dans Firefox, menu Outils > Scaffold).

![L'interface scaffold](/assets/img/zotero-archives-numeriques/scaffold.png)

<center>L'interface de Scaffold v3.2.3</center>

Les icônes permettent un accès aux actions principales de gestion de fichiers (ouvrir et enregistrer un *translator*) et de debuggage.
Le panel de gauche sert à éditer le *translator* dans ses différentes composantes, déclinées en onglets.
Le panel de droite permet de tester et débugger son *translator*, ce qui est le principal atout de Scaffold[^neovim].
Je vous renvoie à la [documentation](https://www.zotero.org/support/dev/translators/scaffold) de Scaffold pour une explication, certes un peu datée, mais plus détaillée de cette interface.

Prenez soin de toujours conserver la page cible en onglet actif (URL dans le champ "Active Tab URL" de Scaffold) lors du développement, sous peine de rendre le débuggage très... disons laborieux.

## Paramétrer les métadonnées du *translator*

Commençons par le plus simple, le paramétrage des métadonnées du *translator*, qui fournissent des instructions à Zotero sur le contexte d'éxécution.
Voici les métadonnées à modifier *a minima* :

+ *Label* : le nom de votre *translator*.
+ *Target* : une Regex qui identifie les URLs concernées par ce *translator*.
  Dans mon cas, une expression très simple `https?://marc.info` suffira largement.
  Vous pouvez tester si votre Regex fonctionne avec le bouton "Test Regex" (devrait retourner `true`).
+ *Translator type* : choisissez "Web".

Enregistrez votre *translator* avant de passer à l'onglet Code.

## Le code du *translator*

L'ongle Code vous montre la partie du *translator* qui réalise l'essentiel du travail.
Un *translator* web doit comporter au moins deux fonctions : `detectWeb()` et `doWeb()`.

### Le rôle de `detectWeb()`

`detectWeb` est la première des deux fonctions à être exécutée.
Son rôle est de voir si la page cible est valide et correspond à la source de données appropriée.
Cette fonction doit retourner des valeurs spécifiques :

1. Si une seule source est détectée, le type de la source ("journalArticle" ou dans notre cas "email") est renvoyé.
2. Si plusieurs sources sont détectées, le type "multiple" est renvoyé.
3. Si aucune métadonnée ne peut être détectée, aucune valeur n'est renvoyée.

Pour l'instant, je vous propose d'aller au plus simple, et de coder une fonction qui retourne toujours "email".

```javascript
function detectWeb (doc, url) {
  return 'email'
}
```

Nous aurons le temps d'y revenir plus tard.
Vous pouvez vérifier que `detectWeb()` renvoie bien "email" systématiquement en utilisant le bouton "Run detectWeb".

### `doWeb()`, le nerf de la guerre

La deuxième fonction `doWeb` est celle qui procède à l'extraction des métadonnées et à leur enregistrement dans la base de données de Zotero.

Pour ce faire, cette fonction demande de créer une nouvelle instance de l'objet `item`, de le modifier avec les métadonnées extraites de la page et enfin de l'enregistrer dans la base.
Schématiquement, notre code sera donc organisé comme suit :

```javascript
function doWeb (doc, url) {
  /* Instanciation d'un nouvel item de type "email" */
  var email = new Zotero.item('email')

  /* Modification des propriétés de l'objet email */
  email.title = 'Titre'
  /* Et ainsi de suite... */

  /* Enfin, enregistrer l'objet */
  email.complete()
}
```

La procédure est donc très simple et linéaire.
Il ne reste plus qu'à extraire les données de la page.

#### Extraction des métadonnées

Dans notre exemple, les métadonnées que nous souhaitons extraire se situent toutes en haut de la page.

![Les métadonnées de la page](/assets/img/zotero-archives-numeriques/marc-header.png)

<center>Les métadonnées à extraire</center>

C'est une chance : dans d'autres cas, nous pourrions avoir besoin d'extraire les informations de plusieurs emplacements dans la page.


L'étape suivante consiste à analyser le code source des parties de la page qui nous intéresse.
Pour ce faire, dans Firefox, sélectionnez la partie en question, puis faites clic-droit et "Examiner l'élément".
Voici le code HTML de l'élément ci-dessus.

```html
<font size="+1">
List:       <a href="?l=git&amp;r=1&amp;w=2">git</a>
Subject:    <a href="?t=111481650200002&amp;r=1&amp;w=2">Re: Trying to use AUTHOR_DATE</a>
From:       <a href="?a=105701892400001&amp;r=1&amp;w=2">Linus Torvalds &lt;torvalds () osdl ! org&gt;</a>
Date:       <a href="https://marc.info/?l=git&amp;r=1&amp;b=200504&amp;w=2">2005-04-30 23:18:11</a>
Message-ID: <a href="?i=Pine.LNX.4.58.0504301607570.2296%20()%20ppc970%20!%20osdl%20!%20org">Pine.LNX.4.58.0504301607570.2296 () ppc970 ! osdl ! org</a>
</font>
```

Nous remarquons que ces données sont englobées dans le même [nœud](https://en.wikipedia.org/wiki/Node_\(computer_science\)) (ou balise, si vous préférez), `font`.
Ici encore, le travail est relativement simple, car chaque donnée est comprise dans un nœud `a`, c'est-à-dire un lien hypertexte en HTML.
Nous allons donc commencer par extraire le texte contenu dans ces nœuds dans diverses variables.

```javascript
var listTxt = ZU.xpathText(doc, '//font/a[1]')
var subTxt = ZU.xpathText(doc, '//font/a[2]')
var fromTxt = ZU.xpathText(doc, '//font/a[3]')
var dateTxt = ZU.xpathText(doc, '//font/a[4]')
```

`ZU` est un raccourci vers l'objet `Zotero.Utilities` ([documentation](https://www.zotero.org/support/dev/translators/functions)).
Ce dernier est automatiquement chargé lors de l'exécution d'un *translator*.

`xpathText(elt, req)` est l'une des méthodes mises à notre disposition par ces utilitaires.
C'est une fonction qui exécute une requête `req` au format XPath sur l'élément XML `elt`, et qui retourne le `textContent` de ce nœud (similaire dans à `/text()` en XPath).
Dans notre exemple :

  + la requête `//font/a[1]` signifie : « parmi tous les nœuds `font`, sélectionner le premier nœud enfant `a`».
  + l'élément xml est la variable `doc`, fournie en argument de la fonction `doWeb`, qui correspond au code html de la page.


Je recommande de tester régulièrement[^debug] que vos requêtes XPath fonctionnent, en particulier si vous débutez avec ce type de syntaxe.

Maintenant que le contenu texte des nœuds est extrait, il nous faut extraire les données pertinentes de ces chaînes de caractères.

#### Pour une poignée de Regex

Deux métadonnées ne nécessitent pas de traitements supplémentaires : le nom de la liste et le sujet de l'email.
Pour les autres, il nous faudra utiliser des expressions régulières pour isoler le nom de l'auteur de l'adresse email et pour isoler la date de l'heure.


```javascript
/* Name Extraction */
var namePattern = new RegExp(/(.*)\s</)
var authorName = namePattern.exec(fromTxt)[1]

/* Date Extraction */
var datePattern = new RegExp(/(\d{4}-\d{2}-\d{2})/)
var sentDate = datePattern.exec(dateTxt)[1]
```

Et voilà ! Nos variables `authorName` et `sentDate` contiennent les valeurs désirées.


#### Enregistrement des métadonnées dans Zotero

Il ne nous reste plus qu'à modifier les propriétés adéquates[^shorttitle] de l'objet `email`.

```javascript
email.title = subTxt
email.shortTitle = listTxt + ' - Mailing list ARChives'
email.libraryCatalog = email.shortTitle

email.creators.push(ZU.cleanAuthor(authorName))
email.date = sentDate

email.language = 'en'
email.accessDate = 'CURRENT_TIMESTAMP'
email.url = url
```

Nous utilisons à nouveau une fonction des *Zotero Utilities*, `cleanAuthor`, qui formate automatiquement une chaîne de caractères en nom de famille.
Je souligne l'importance de conserver la date et l'heure de consultation de l'email dans `accessDate` : c'est l'attribut qui est utilisé pour ajouter automatiquement « consulté le xx/xx/xxxx » à votre référence.

Il ne nous reste plus qu'à signaler à Zotero que nous avons terminé les modifications des propriétés de l'objet pour qu'il procède à son enregistrement :

```javascript
email.complete()
```

Nous avons à présent terminé le gros du travail dans `doWeb()`.

### Amélioration de `detectWeb()`

Revenons à la fonction `detectWeb()` que nous avions temporairement mise de côté quelques étapes plus tôt.
Pour l'instant, elle renvoie systématique "email" sans analyser la page d'aucune façon, ce qui n'est pas très utile.
Nous allons donc systématiquement vérifier que la page contient le texte "Message-ID: ", ce qui est un des signes distinctifs des pages cibles.

```javascript
var msgId = ZU.xpathText(doc, '//font/text()')
Zotero.debug(msgId)

if (msgId !== null && msgId.indexOf('Message-ID: ')) {
  return 'email'
}
```

## Code final du *translator*

Notre travail sur le code du *translator* est à présent terminé.
Le voici dans son ensemble :

```javascript
function detectWeb (doc, url) {
  var msgId = ZU.xpathText(doc, '//font/text()')
  Zotero.debug(msgId)

  if (msgId !== null && msgId.indexOf('Message-ID: ')) {
    return 'email'
  }
}

function doWeb (doc, url) {
  var email = new Zotero.Item('email')

  /* Elements extraction */
  var listTxt = ZU.xpathText(doc, '//font/a[1]')
  var subTxt = ZU.xpathText(doc, '//font/a[2]')
  var fromTxt = ZU.xpathText(doc, '//font/a[3]')
  var dateTxt = ZU.xpathText(doc, '//font/a[4]')

  /* Name Extraction */
  var namePattern = new RegExp(/(.*)\s</)
  var authorName = namePattern.exec(fromTxt)[1]

  /* Date Extraction */
  var datePattern = new RegExp(/(\d{4}-\d{2}-\d{2})/)
  var sentDate = datePattern.exec(dateTxt)[1]

  email.title = subTxt
  email.shortTitle = listTxt + ' - Mailing list ARChives'

  email.creators.push(ZU.cleanAuthor(authorName))
  email.date = sentDate

  email.language = 'en'
  email.accessDate = 'CURRENT_TIMESTAMP'
  email.libraryCatalog = email.shortTitle
  email.url = url

  email.complete()
}
```

Moins de 40 lignes de codes nous permettent donc d'archiver, de sauvegarder et de citer plusieurs millions d'emails parmi les centaines de listes de diffusion de l'archive MARC...
Une bonne affaire en somme !

# Conclusion

Il ne vous reste plus qu'à récolter les fruits de votre dur labeur.
Enregistrez votre *translator* [^dossier], fermez Scaffold et rechargez la page cible.
Zotero devrait à présent reconnaître la source de données :

<center>
![L'icône de Zotero signale la source de données](/assets/img/zotero-archives-numeriques/zotero-icon.png)

Zotero reconnaît maintenant la source de données !

</center>

Vous pouvez trouver l'ensemble de mes *translators* dans mon [dépôt Github](https://github.com/gaalcaras/translators).

Bien sûr, l'exemple présenté ici reste extrêmement simple.
Je vous invite à lire le code de *translators* plus complexes pour trouver des pistes d'amélioration, par exemple dans le [repo officiel](https://github.com/zotero/translators).
Dans un futur billet, j'aborderai la question des tests systématiques des *translators*, qui vous permettra de créer des traducteurs plus robustes.


[^beuscart]: Pour une introduction en français, voir les encadrés méthodologiques dans : Jean-Samuel Beuscart, Éric Dagiral, Sylvain Parasie, *Sociologie d'internet*, Armand Colin (2016),
  URL : http://www.armand-colin.com/sociologie-dinternet-9782200612429.
[^lkml]: Linux Kernel Mailing List, consacré au développement du noyau Linux.
  Une des archives disponibles : http://lkml.iu.edu/hypermail/linux/kernel/index.html.
[^neovim]: L'éditeur de code de Scaffold, plus que rudimentaire, rend très désagréable l'édition de *translators* longs ou complexes pour qui a l'habitude de travailler dans un éditeur performant.
  Je ne saurais trop vous recommander d'utiliser un éditeur de texte externe ([neovim](https://neovim.io) dans mon cas) une fois la phase d'apprentissage passée.
[^dossier]: Le fichier complet du *translator* est enregistré dans un dossier "translators" de votre répertoire Zotero (cf. [documentation](https://www.zotero.org/support/zotero_data)).
[^debug]:  Si vous avez l'habitude d'utiliser `console.log()` pour débugger votre JavaScript, vous pouvez utiliser `Zotero.debug()` dans Scaffold pour obtenir le même résultat.
  Par exemple, en ajoutant l'instruction `Zotero.debug(listTxt)` et en appuyant sur "Run doWeb", le mot "git" devrait apparaître dans le panel de droite.
[^shorttitle]: Pour des raisons de gestion de bibliographie dans LaTeX, je garde le nom de l'archive email à la fois dans `shortTitle` et `libraryCatalog`.
  C'est un bricolage personnel dont vous pouvez vous passer.

