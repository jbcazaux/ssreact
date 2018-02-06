# server side react
This is a WORK IN PROGRESS

### Du client au serveur

L'idee est de partir d'une application classique et simple, qui ne fait le rendu que cote front, pour l ameliorer au fur et a mesure.
Les differentes etapes d amelioration sont:
- faire un rendu cote serveur
- mettre en place du routing avec react-router
- ajouter redux a la stack et transferer le store du back au front
- initialiser le store de facon asynchrone cote serveur
- creer une redirection

### Application de base

Dans la version client-side only, l application est un simple composant, qui affiche le nombre de fois que l'on clique sur le bouton.

(gist counter)
C'est un composant classique qui gere son compteur de clics dans son state.
(gist webpack)
Rien d'extraordinaire non plus dans le fichier de conf webpack, on retrouve simplement la generation du fichier index.html, l appel a babel pour transpiler l es6 et le jsx en es5. Le dev-server est utilise pour le rechargement a chaud de l'appli lors de modification du code.
On remarque simplement que pour le front le point d'entrée est le fichier client/index.js.

### Server side rendering

Afin de produire un rendu cote back il faut configurer et lancer un serveur web dans un process node.
La premiere etape est donc de creer un fichier pour lancer express, afin qu il ecoute les requetes http, et serve le rendu html de l application.
(code server.js)
Rien de particulier a part l'import de la librairie serve-favicon qui permet de servir le fichier public/fav.ico en tant que facvicon.

La partie importante est l'utilisation de la methode renderToString de react-dom qui permet de generer dans une string, le code html du composant passe en parametre, en general le composant racine de l application.
Une fois le code html de l'application genere, il faut l'inclure dans une page html, avec les traditionnelles balises <html>, <head>, <body>, etc... 
(code server.js)
Et bien sur une balise pour charger le code JS de notre application, afin qu'elle soit presente sur le navigateur pour assurer un rendu dynamique (revenir au cas classique du rendering cote navigateur.)
(code html.js)

Il faut ensuite configurer webpack pour qu'il génère également un fichier des sources pour le backend. Le point d'entrée est alors server/server.js. On notera que l'on peut exclure du bundle les différentes librairies utilisées, étant donnée qu'elles sont dans le répertoire /node_modules.
Le cleanWebpackPlugin permet de supprimer le répertoire dist avant chaque nouveau build.
Les sources sont transpilées de la même façon que pour le front, d'où l'extraction de certaines propriétés dans un objet 'common'.
(code webpack.config.js)

Pour tester l'application, il suffit de lancer 'npm run s' ou 'yarn s' et ouvrir un navigateur sur localhost:3000. En affichant la source de la page, on constate biem que le html genere cote serveur est bien present. \o/
Pour le mode client-side seulement, on peut lancer 'npm run start:dev' ou 'yarn start:dev'.
### Routing

Comme on a pu le voir, générer le rendu html des composants coté serveur est plutôt simple. La difficulté vient en fait de la gestion du routing (urls) et des chargements asynchrones des données. Dans ce paragraphe nous allons nous voir le routing.

Dans l'application nous allons donc ajouter un menu avec 3 liens, chacun des liens va charger un des composants. L'url doit refléter le composant actuellement afficher.
Bien sûr, quand on demandera le rendu coté serveur, il faudra que le bon composant soit généré en html afin que le navigateur l'affiche, le temps que l'application react soit chargée. 
Nous utilisons react-router de façon simpliste, avec 2 nouveaux composants ne faisant rien d'extraordinaire.
(code app.js)
Si le code source des composants restera le même pour le front et pour le back, il y a cependant une différence au niveau du router à utiliser.
Pour le front il faudra utiliser le BrowserRouter, et pour le back le StaticRouter.
Nous allons profiter que pour le front ce soit le fichier client/index.js qui charge l'application et pour le front le fichier server/server.js (voir la conf webpack) pour encapsuler dans chacun de ces fichiers notre <App> qui elle est universelle (front et back).
(code webpack.config.js)
(code index.js)
C'est donc surtout dans la génération de l'application coté back que nous allons avoir du travail. Tout d'abord il faut passer 2 paramètres au 'StaticRouter': l'url courante (location) et un objet qui permet de transporter des informations (context).
Dans un premier temps, nous n'utiliserons pas le contexte, un objet vide suffira. L'url courante est simplement récupérée dans la requête envoyée à express.
(code server.js)

On peut relancer l'application afin de controler que tout fonctionne.

### Redux

Beaucoup d'applications utilisent redux afin de gérer les données de l'application. Afin que le rendu s'opère correctement coté back, et surtout garder exactement le même code de composants pour le front et le back, il faut créer le store également coté back. Pas de difficulté particulière car la méthode à appeler (createStore) est la même.
En revanche on peut optimiser... passer le store back au front...