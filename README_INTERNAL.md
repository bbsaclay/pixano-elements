**ATTENTION :** ce document est interne et ne doit en aucun cas être publié sur le github !  
La documentation publique se trouver dans le [README.md](./README.md).

Contenu de ce document :  
[[_TOC_]]

# organisation/gestion des dépôts
## version opensource publiée sur github
- [dépôt](https://github.com/pixano/pixano-elements) opensource sous licence [CeCILL-C](./LICENSE.txt)
- contient tous les modules et codes issus de l'EdA
- étiquettes de versions : vX.Y.Z
- **on ne devrait jamais pousser directement sur ce dépôt !** Il faut toujours passer par les pull-request pour sécuriser et générer du trafic (voir [procédure](#procedure-de-publication)).

## version interne publiée sur notre gitlab
### branche remote github
- cette branche est une copie locale du dépôt github **forké**. Elle n'est utilisée que pour préparer et effectuer les publications (ou plus rarement pour récupérer du code du github s'il est en avance sur master du gitlab).
- pour chaque nouvelle version à publier sur le github :
	- on la prépare sur cette branche
	- on la teste et valide sur cette branche
	- on l'étiquette sur cette branche
	- on la pousse sur le github forké avant de faire une pull-request sur le dépôt original (voir [procédure détaillée](#procedure-de-publication))
### branche master
- branche par défaut récupérée par tout nouveau contributeur
- version la plus à jour, elle contient, en plus des fonctionnalités présentes sur le dépôt github, toutes les fonctionnalités intelligentes utilisables en interne labo et avec nos partenaires
- regroupe "régulièrement" les avancées projets (sauf les code projets qui restent propriétaires d'une entreprise en particulier)
- étiquettes de versions : viX.Y.Z
### autres branches
- une branche par projet (industriel, thèse, stage)
- chacune de ces branche dérive de master
- toutes les fonctionnalités ou corrections suffisamment matures sont poussées "régulièrement" sur le master (merge ou cherry-pick selon le cas)
- en fin de projet, toute la branche est fusionnée avec master
- étiquettes de versions (le cas échéant) : vipX.Y.Z ou étiquette spécifique à l'industriel (par exemple : vipX.Y.Z_arcure, vipX.Y.Z_valeo, etc)

### schématiquement :
```
github                                      gitlab  
------                     ---------------------------------------  
                                                    <--merge--> p1  
master  <------push------  github <--merge-- master <--merge--> p2  
                                                    <--merge--> p2  
                                                    ...  
```

## cas particulier d'un dépôt projet devant utiliser tuleap
- les développements liés au projet sont effectués sur le tuleap par les partenaires et par les membres CEA
- push "régulier" des avancées tuleap vers la branche du projet sur le gitlab
- remontée des avancées gitlab vers le tuleap en fonction du besoin, procédure :
	1. push tuleap vers gitlab (projet)
	2. merge des branches gitlab master et projet
	3. push gitlab (projet) vers tuleap

*On entend par "régulièrement" : tous les 6 mois environs, idéalement lors de deux campagnes à la rentrée de septembre et celle de janvier. Libre à chacun évidement de faire ces merges plus régulièrement ou au fil de l'eau.*




# Open-source publication procedure
## prerequisite
### on github
- create a [github](https://github.com) $MYACCOUNT account 
- create a fork of the [original repository](https://github.com/pixano/pixano-elements)
### initialize the repository remote branch
	git remote add upstream git@github.com:$MYACCOUNT/pixano-elements.git
	git fetch upstream
	git checkout -b github upstream/master
### avoir cloné le site web
	cd .. ; git clone git@github.com:pixano/pixano.github.io.git ; cd -

## mettre à jour son fork
- sur son fork github, appuyer sur "Fetch upstream", puis "Fetch and merge"

## préparer la publication
	# être sûr d'avoir les dépôts à jour
	git fetch
	git fetch upstream
	git checkout github
	# intégrer nos modifs à la branche github
	## git merge master github
	# procéder par cherry-pick pour plus de sécurité (ne pas inclure les commits purement internes)
	# git cherry-pick b4cb0b18^..d5e075f2 (all commits from x to y) or git cherry-pick d5e075f2
	git cherry-pick commitSHA

Durant le merge / avant le commit, **ne pas inclure / supprimer les fichiers et codes internes/propriétaires** :  

- ne pas inclure le présent fichier [README_INTERNE.md](./README_INTERNE.md), ni le .gitlab-ci.yml
- ne pas inclure le dossier [doc_interne](./doc_interne)
- ne pas inclure les fichiers équipés d'une balise "propriétaire"
- ne pas inclure les blocs de code entourés de d'une balise "propriétaire"

## vérification et validation du code
### vérifications "à la main"
	npm run build
	npm run build:umd
	npx serve demos/cuboid/
	npx serve demos/polygon/
	npx serve demos/graph/
	npx serve demos/segmentation/
	npx serve demos/segmentation-interactive/
	npx serve demos/rectangle/
	npx serve demos/smart-rectangle/
	npx serve demos/tracking/

### finalisation
- revue de code
- clean code: `npm run tslint`

## publier
### 1. push
	VERSION=0.5.14
	#maj de la version de publication
	node changeversion.js $VERSION
	git commit -m "release $VERSION"
	git tag -m "v$VERSION" "v$VERSION"
	# pousser les modifs sur le fork
	git push upstream github:master --follow-tags
### 2. pull-request
Le reste se passe sur [github](https://github.com) :

	- sur le fork $MONCOMPTE : onglet "Pull requests" => "New pull request" => "Create pull request" => "Create pull request"
	- des vérifications automatiques sont effectuées par github
	- sur le compte pixano : aller dans "Merge pull request" => "Confirm merge"
### 3. release
Transformer le tag en release github (permet de rendre le dernier tag plus visible) :

<!--	NE FONCTIONNE PAS (encore?) car les tags ne sont pas exportés lors des pull request-->
<!--	- aller sur la page des [tags](https://github.com/pixano/pixano-elements/tags)-->
<!--	- sélectionner le dernier tag-->
<!--	- "Edit release"-->
<!--	- dans "Release title", remettre la version vX.Y.Z-->
<!--	- "Update release"-->
	
	- aller sur la page des [release](https://github.com/pixano/pixano-elements/releases)
	- bouton "Draft a new release"
	- "Tag version" vX.Y.Z
	- dans "Release title", remettre la version vX.Y.Z
	- "Publish release"
	
	
### 4. publication npm
	# if not logged already
	# contact pixano@cea.fr for more information
	npm login
	npm publish packages/core
	npm publish packages/ai
	npm publish packages/graphics-2d
	npm publish packages/graphics-3d

### 5. publication of the documentation
	npm run docs
	rm -r ../pixano.github.io/docs ; cp -r docs ../pixano.github.io/docs
	cd ../pixano.github.io/
	git commit -a -m "release $VERSION"
	git tag -m "v$VERSION" "v$VERSION"
	git push --follow-tags

# règles de développement
- les fonctionalités propriétaires (à **ne pas** reporter sur le github) doivent être bien identifiées : fichier séparé à chaque fois que possible, bloc identifié par des balises autrement
- la documentation interne (à **ne pas** reporter sur le github) est séparée de la documentation publique : elle est intégralement stockée dans le dossier [doc_interne](./doc_interne) et le fichier [README_INTERNE.md](./README_INTERNE.md)
- la documentation publique est entièrement rédigée à destination des utilisateurs github et ne doit faire aucune référence à notre fonctionnement interne

## exemple de balise à placer en début de fichier
	#####################################
	###### INTERNAL, do not publish #####
	#####################################





