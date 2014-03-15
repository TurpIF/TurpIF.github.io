---
layout: post
title: Jekyll en premier
image: "/assets/premier-article-icone.png"
categories: web jekyll github
---

Voici mon premier article sur ce nouveau site que je viens de faire rapidos.
Avant de passer au vif de l'article, je vais donc tout d'abord me présenter
comme il se doit :  
Je m'appelle Pierre Turpin, j'ai dans les vingts ans (21 pour être exact) et je
suis en étude d'ingénieur en informatique en ce moment. J'aime les nouvelles
technologies et en général pas mal de trucs qui touche à l'informatique allant
du développement de programmes systèmes avec des forks de partout jusqu'au
petit jeu vidéo en navigateur en passant par les memes et les chats sur le net
(et aussi le Python <3 ).

Avec cette description, on pourrait penser que je reste constamment enfermé
dans une salle sombre devant un écran avec plein de lignes de code. Et non, il
m'arrive d'ouvrir de temps en temps ma fenêtre et d'apercevoir le soleil. Des
fois même, je sors dehors. Plus sérieusement, j'aime également voir des films,
écouter de la musique (plutôt rock / rock alternatif mais je reste assez
ouvert), bien manger et faire du sport.

Voici ma présentation finie, nous allons donc pouvoir attaquer le sujet :
[Jekyll][].

## Docteur Jekyll et mister Hide

Je viens de découvrir cet outil que très récemment et c'est donc qu'une
présentation assez succincte que je vais faire et pas un mémoire.

Déjà pourquoi en parler ? J'en parle car cet outil m'a permis de faire
rapidement ce site et permet de le maintenir très facilement (pour ce que j'ai
vu en ce moment). En effet, il m'a fallu que quelques heures seulement pour
découvrir, manipuler et utiliser Jekyll pour créer ce site web.  
Attention je ne dis pas qu'en quelques heures, on devient incollable sur le
sujet. Juste qu'en peu de temps on peut faire pas mal de chose et ça c'est
cool.

### Un peu de liquide...

Jekyll est écrit en **Ruby** et est un générateur de sites statiques. Rien
n'est dynamique. Cela rend son utilisation assez délicate avec des gros sites
(voir impossible je pense) mais pour des sites simples c'est tout à fait
satisfaisant.

Jekyll utilise le langage de template [Liquid][] pour pouvoir générer les
toutes vues statiques nécessaires. Les blocs de contrôle classique sont
disponibles comme les conditions et les boucles. Voici un exemple de base qui
montre ce qu'on peut faire avec :

```html
<ul>
{ % for post in site.posts %}
  <li>
    <h1>{ { post.title }}</h1>
    { % if post.image %}
      <img src="{ { post.image }}" />
    { % endif %}
    <p>{ { post.description | truncate: 200 }}</p>
  </li>
{ % endfor %}
</ul>
```

*Pour éviter que Jekyll ne parse ce code, j'ai du placé un espace après
chaque crochet ouvrant. Normalement il n'y en a pas.*

La boucle `for` s'utilise comme un `foreach` des autres langages. Il itère sur
le conteneur et passe une référence nommée dans son contexte d'exécution. Le
`if` test une expression booléenne comme dans les autres langages. Dans
l'exemple, `post.image` n'est pas un booléen mais une conversion implicite doit
avoir lieu. Enfin, l'affichage du contenu d'une variable se fait simplement et
il est possible d'utiliser des **tags** de Liquid afin de traiter la données (voir
le `truncate` dans l'exemple). Ces tags ressemblent aux pipes sous Linux et
peuvent s'enchainer.

L'avantage de tout cela repose dans la simplicité d'écriture et de
compréhension. De plus, je trouve que les tags sont très puissants et
permettent de transformer comme on veut nos données. D'autant plus, qu'il est
possible d'en créer soit même pour augmenter la gamme proposée par défaut. Ce
n'est que du **Ruby** à placer dans un dossier \_plugins. Je ne connais
malheureusement pas assez bien le Ruby, je ne peux donc pas vous montrer
maintenant comment réaliser un tag personnalisé. Mais cela pourra bien être le
sujet d'un futur article.

### ... et des pas mal de gadgets

En plus de Liquid, Jekyll intègre pas mal d'outils permettant de rédiger et
construire plus facilement son site.

#### La pagination

Pour automatiser la pagination des articles, il suffit d'ajouter les options
`paginate` et `paginate_path` dans le fichier *_config.yml* de Jekyll. Ces deux
options indiquent respectivement le nombre d'articles par pages et le chemin
généré pour accéder aux différentes pages :

```yaml
paginate: 5
paginate_path: "/mon/chemin/favori/page:num"
```

L'intégration dans le squelette se fait assez facilement avec la variable
`paginator` automatiquement inclue dans Liquid. Cette variable est un
objet qui contient toutes les informations utiles.

Voici une liste non-exhaustive des attributs de l'objet `paginator` :

- `paginator.page` : Numéro de la page courante
- `paginator.posts` : Liste des articles pour la page courante
- `paginator.previous_page_path` : URL vers la page précédente
  (`paginator.next_page_path` existe aussi)
- `paginator.previous_page` : Numéro de page de la précédente
  (`paginator.next_page` existe aussi)

Je vous laisse voir la [documentation sur la pagination][DocPagination] de
Jekyll pour plus de détail.

#### Des articles en Markdown ou Textile

Jekyll propose d'écrire nos articles avec le langage de notre choix parmi le
**HTML**, le **Mardown** ou le **Textile**. Dans le cas des deux derniers,
ceux-ci sont d'abord généré en HTML puis inclue dans le code source du site.

Cela permet d'avoir beaucoup de confort d'écriture en tout en ayant assez de
liberté pour la mise en page et la structuration de l'article.

De plus, les articles ne sont pas stockés dans une base de donnée mais
simplement dans un dossier *./_post*. Chaque fichier à l'intérieur de ce
dossier est considéré comme un article. Pour les petits sites (qui est la cible
de Jekyll), cela rend la maintenance assez aisée et simple. En effet, AMHA, la
gestion et l'utilisation est plus facile qu'avec une base de données.

### Intégration avec GitHub

J'utilise énormément [GitHub][] et ce qui m'a fait choisir Jekyll c'est avant
tout son intégration dans les pages GitHub. GitHub permet d'héberger des pages
sur le compte de chaque utilisateur. Cependant, ce n'est pas un hébergeur. Donc
les services proposés sont très simple : page statique uniquement. Donc seul
les langages HTML/CSS/JS sont autorisés.

Faire un site de une seule page est réalisable mais créer un site un peu plus
compliqué avec même que quelques pages devient alors très rapidement compliqué.
Il faut un fichier HTML par page avec à chaque fois le header et le footer qui
est recopié dans chaque fichier.  Cela signifie qu'à la moindre modification,
il faut la faire dans tous les fichiers. À partir de 3-4 pages différentes ça
devient très rapidement ingérables.

C'est la que Jekyll et GitHub entre en jeu. Jekyll permet de générer un site
statique qui est donc compatible avec GitHub. Il est alors possible de faire la
génération et d'envoyer ça sur son répo distant. Mais encore mieux, GitHub se
charge automatiquement de faire ça. Il suffit d'envoyer les fichiers de notre
projet Jekyll, et GitHub possède de son côté Jekyll qui va construire notre
site web.

#### Quelques limites

Cette fusion a l'air génial mais reste toutefois limité. En effet, une des
puissances de Jekyll vient de sa facilité à rajouter des extensions et créer de
nouveaux tags par exemple. Ces extensions sont écrites en Ruby et doivent donc
être exécuté avec Jekyll pour la génération du site. Tout cela est désactivé
sur GitHub par raison de sécurité. C'est donc un très gros frein dès qu'il y a
besoin de quelques personnalisations.

Heureusement il y a quand même moyen de contourner cette difficulté en
utilisant deux répertoires GitHub : un qui contient la source Jekyll avec les
extensions et un autre qui contient le site généré. Il faut alors exécuter soit
même la construction du site et pousser celui-ci sur le repo distant. On perd
malheureusement l'intégration de Jekyll dans GitHub.

#### Aller loin avec GitHub

GitHub permet de forker des repo très facilement. Cela est très utilisé pour
contribuer sur un projet. On peut alors penser que ce système peut être
reproduit sur des sites web. Il est tout à fait possible de contribuer à
l'écriture des articles, au design, ... Cela permet de reproduire l'effet wiki
(tout le monde peut être contributeur).

Ce dernier point reste toutefois assez utopique et n'est valable que pour les
très gros site (qui ne seront pas héberger sur GitHub alors).

[GitHub]: http://github.com/
[DocPagination]: http://jekyllrb.com/docs/pagination/
[Bootstrap]: http://getbootstrap.com/
[Jekyll]: http://jekyllrb.com/
[Liquid]: http://liquidmarkup.org/
