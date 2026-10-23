# CLAUDE.md — Appli « Menu de la semaine famille + courses + Dukan »

Ce fichier donne à Claude Code le contexte pour reprendre le projet. Il est écrit en français, comme tout le projet. Lis-le en entier avant de modifier quoi que ce soit.

## 1. Objectif

Une appli web mobile, utilisée sur téléphone, pour Cyril (technicien de maintenance en horaires 2x8), sa compagne Aurore et leurs filles Laura (la grande) et Camille (la petite). Elle sert à :

1. Composer le **menu de la semaine** (midi et soir) selon qui est à la maison, le poste de travail de Cyril et les règles de la famille.
2. Gérer en plus les **repas Dukan** de Cyril (et plus tard d'Aurore), selon la phase choisie jour par jour.
3. Générer une **liste de courses** (Super U de Pontcharra) rangée dans l'ordre réel des rayons du magasin, modifiable, avec produits ménagers et de base.
4. Produire un **PDF imprimable** du menu qui ressemble à leurs feuilles de menu papier (7 cases Lundi à Dimanche, « Midi » et « Soir », « Bon appétit ! »).

Une appli Dukan séparée (`dukan.html`) existe pour le **suivi** (poids, graphique, craquages, activité). Décision de Cyril : à terme elle ne garde que le suivi ; les plats et repas Dukan par phase vivent dans l'appli principale.

## 2. Fichiers et liens

| Fichier | Rôle | Lien publié (artifact claude.ai) |
|---|---|---|
| `menu-famille.html` | **Appli principale** (menu famille + Dukan + courses + plats) | https://claude.ai/artifact/3sjFK2HpxAnNA2JenmsJ2d |
| `dukan.html` | Appli Dukan V2 (menus Dukan, craquages, activité, poids avec courbe et barres, courses) | https://claude.ai/artifact/4DXqH7XbLmrgtbg3e35ssV |

Les deux sont des fichiers HTML autonomes (CSS et JS dans le même fichier). Il n'y a ni build, ni serveur, ni dépendance npm.

## 3. Règles de la famille (le cœur métier)

### Qui mange quand
- **Laura** mange à la maison tous les midis.
- **Camille** est à la cantine le midi, sauf le mercredi. Le mercredi, Laura, Camille et Aurore mangent ensemble.
- **Aurore** travaille le vendredi (5h–13h30), le samedi et le dimanche (5h–17h). Ces jours-là, elle ne mange que le soir. Du lundi au jeudi, elle mange à midi (plats différents de ceux de Laura, sauf le mercredi) et le soir avec les filles.
- **Le soir**, tout le monde mange ensemble, tous les jours.
- **Jeudi soir où Cyril est d'après-midi** : repas simple et rapide (les filles ont tir à l'arc).
- **Week-end midi** : plat cuisiné à la maison ou chez Papy / en famille.
- **Case « Vacances »** sur chaque jour : tout le monde est à la maison (midi famille + soir, plus de plat rapide, pas de poste pour Cyril).

### Le planning de Cyril (calendrier)
Fonction `shift(d)` dans le JS. Cycle copié de son calendrier de travail HTML :
- On part de la semaine du lundi 29/12/2025 (numéro 0).
- Semaine paire : **matin** (5h–13h) du lundi au jeudi, repos vendredi à dimanche.
- Semaine impaire : **après-midi** (13h–21h) du lundi au vendredi, repos samedi et dimanche.
- Le 1er janvier et le 31 décembre sont toujours en repos.
- Les congés posés dans le calendrier d'origine ne sont **pas** importés (voir « reste à faire »).

### Midi de Laura (slot `L`)
- **Semaine du matin** : restes de la veille (le nom du plat du soir précédent s'affiche) ou plat cuisiné de supermarché facile à réchauffer.
- **Semaine d'après-midi** : 50 % de plats prêts de supermarché, 50 % de plats que Cyril prépare pour lui avant de partir. Les 50/50 sont équilibrés sur les jours d'après-midi de la semaine affichée (`pickL`).
- Si Cyril est au régime Dukan ce jour-là (case cochée + phase), le plat « préparé par Cyril » devient **son plat Dukan du midi** (`dkLink`), avec ses ingrédients ajoutés aux courses sous la mention « (Laura) ». Si Dukan est décoché, le plat standard revient.
- Jour de repos (ex. vendredi d'une semaine du matin) : plats prêts ou plats « L ».

### Dukan
- 5 phases (`PH`) : Attaque (`at`, protéines pures), Croisière PP (`pp`), Croisière PL (`pl`), Consolidation (`co`), Stabilisation (`st`). Les phases `at` et `pp` excluent les plats avec légumes.
- Chaque jour : skyr ou yaourt 0 % avec des cuillères à soupe de son d'avoine (1,5 / 2 / 2 / 2,5 / 3 selon la phase) et 1 fruit (pomme ou poire) en Consolidation et Stabilisation.
- Cyril choisit **jour par jour** si son repas Dukan s'ajoute ou remplace le repas famille. Midi **et** soir.
- Cocher Dukan pour une personne la **retire du menu standard** ce jour-là (étiquettes « sans Cyril », « sans Aurore », ligne midi d'Aurore supprimée, ses ingrédients retirés des courses).
- Contraintes de temps pour les plats Dukan de Cyril : matin 15 min, après-midi midi 30 min et soir 10 min, repos illimité. Les plats « batch » sont toujours autorisés.
- **Aurore** (clé `'a'+date`) : mêmes plats que Cyril si la phase est la même, avec possibilité de modifier (un drapeau `o` par repas ; le bouton « = » revient au plat de Cyril). Midi seulement du lundi au jeudi et en vacances, soir seul le vendredi, samedi, dimanche.

## 4. Fonctionnalités déjà faites (appli principale)

- Onglets : **Menu**, **Courses**, **Plats**.
- Semaine navigable ; le dimanche, l'appli s'ouvre sur la semaine suivante (Cyril prépare le menu le dimanche).
- Génération automatique (« Remplir la semaine »), 🎲 au hasard et ✏️ choix manuel pour chaque repas.
- Cases Vacances et Dukan (Cyril et Aurore) par jour, phase mémorisée (`lp` et `lpa`).
- 88 plats famille avec **recette détaillée** chacun (bouton « Recette »), dont 14 plats prêts de supermarché pour le midi de Laura.
- 28 plats Dukan avec recette, grammes et portions.
- Onglet **Plats** : ajouter, modifier, supprimer un plat ; choisir ses moments ; ajouter les ingrédients à la main **ou** les choisir dans la liste des ingrédients déjà enregistrés (cases multiples + recherche).
- **Liste de courses** en trois modes : « Ma liste » (articles choisis, avec cases « pris »), « Choisir les articles » (ingrédients du menu cochés d'office, produits de base à cocher), « Parcours du magasin » (apprentissage de l'ordre des rayons en cochant les articles un à un dans le magasin).
- Ingrédients cités dans les recettes mais absents des plats : ajoutés automatiquement (`V`, `recX`, `recXD`). Frais cochés d'office, produits de placard listés non cochés.
- Ajout de produits à la main (dont petit-déjeuner par personne), produits ménagers et de base (31 articles dans `BASE`).
- Courses Dukan avec la mention « (Cyril) », « (Aurore) » ou « (Laura) », quantités en grammes ou en nombre (œufs, skyr, fruits).
- Export de la liste en texte (téléchargement ou copie) avec ☐ / ☑.
- PDF : bouton « Imprimer / PDF » (feuille A4 paysage, 7 cases, midi de Laura et d'Aurore séparés, mention « sans Cyril / sans Aurore »).
- Thème clair et sombre, zones sûres (encoche), mise en page mobile d'abord.

## 5. Choix techniques

- **Un seul fichier HTML**, JavaScript vanilla, aucune bibliothèque, aucune requête réseau. Tout est en dur dans le fichier.
- **Hébergement** : publié comme artifact claude.ai (page privée, partageable). Contrainte de cet environnement : pas de requêtes vers d'autres sites, pas d'API `window.storage`. Seuls `localStorage` et la capacité `downloads` sont utilisables.
- **Stockage** : `localStorage`, clé `menufam1` (objet `S` sérialisé), toujours dans un `try/catch`. Les données restent **dans le navigateur de chaque appareil** (pas de synchronisation).
- **Rendu** : `render()` reconstruit `#app` en `innerHTML` à chaque changement ; les gestionnaires sont des `onclick` / `onchange` inline qui appellent des fonctions globales.
- **Impression** : CSS `@media print` + une zone `.po.sheet` (`sheetH`) affichée seulement à l'impression ; les contrôles ont la classe `np` (masqués à l'impression).
- **Téléchargement** : `claude.use('downloads')` puis `save({filename, data})`, avec repli sur la copie dans le presse-papiers puis sur une zone de texte à copier à la main. **Jamais testé dans l'environnement publié réel.**

### Carte du fichier `menu-famille.html`
1. `<style>` : variables de thème (clair/sombre), composants, règles d'impression.
2. Données :
   - `R` : 12 rayons dans l'ordre de base.
   - `DEF` : plats famille. Une ligne par plat : `nom|tags|ingrédient:rayon,ingrédient:rayon`. L'identifiant est `'d'+index` : **ne jamais réordonner ni insérer au milieu, seulement ajouter à la fin.**
   - Tags : `S` soir, `Q` rapide, `W` midi mercredi et week-end, `E` midi week-end seul, `C` midi d'Aurore, `L` midi de Laura, `R` plat prêt.
   - `FREC` : recettes, **alignées par index sur `DEF`** (étapes séparées par `;`, conseil après `#`).
   - `BASE`, `DK` (plats Dukan : `nom|minutes|batch|aliment,grammes,portion;...`), `DKREC`, `PH`, `VG`, `DR`, `V`.
3. Utilitaires : `iso`, `add`, `fr`, `esc`, `sow`, `shift`.
4. État `S` + `save()` + migration des plats (`S.dn`).
5. Menu : `slots0/slots`, `tag`, `opts`, `pick`, `pickL`, `fill`, `rnd`, `setD`, `dayH`, `menuV`, `labF`, `exc`, `vac`, `sheetH`.
6. Courses : `items`, `selOf`, `rank`, `catalog`, `chosen`, `shopV`, `parcV`, `saveO`, `txt`, `dlT`, `cpL`.
7. Plats : `platsV`, `pkH` (sélecteur d'ingrédients existants), `saveF`, `delD`.
8. Dukan : `dkBlock`, `dkEns`, `dkPick`, `dkItems`, `dkLink`, `dkSl`, `dkTog`, `dkPh`, `dkRnd`, `dkSet`, `dkSame`.
9. `render()` et appel final.

### Contenu de l'état `S` (`localStorage`)
`m` (menus : date → { slot → id de plat }), `v` (semaines validées), `x` (articles ajoutés par semaine), `c` (articles « pris »), `sl` (cochage « à acheter » forcé), `rm` (articles retirés), `ord` (ordre des rayons appris : nom → rang), `D` (copie personnalisée des plats ou `null`), `dn` (nombre de plats `DEF` déjà fusionnés), `vac`, `dk` et `dm` (Dukan : clé = date pour Cyril, `'a'+date` pour Aurore), `lp` et `lpa` (dernière phase de Cyril et d'Aurore).

## 6. Problèmes connus

1. **Détection des ingrédients dans les recettes** : elle repose sur une liste de mots (`V`). Un ingrédient inhabituel cité dans une recette mais absent du plat n'est pas ajouté aux courses.
2. **Clés des articles « pris »** : elles contiennent le nom, donc les grammes pour les plats Dukan. Si le menu change après avoir coché des articles, ceux dont la quantité change repartent décochés.
3. **Ordre du magasin** : tant que le parcours n'est pas fait, l'ordre est celui des rayons par défaut. Un parcours partiel place les articles non cochés à la fin. Pas de refonte rayon par rayon.
4. **Aurore** utilise la liste d'aliments de Cyril (pas de questionnaire pour elle) et n'a aucune contrainte de temps de préparation.
5. **Consolidation et Stabilisation** : seuls le fruit et le son d'avoine sont gérés. Pain complet, fromage, féculents, repas de fête ne sont pas modélisés.
6. **Code mort à nettoyer** : `shopOld`, `txtOld`, `dkItemsOld`, `delI`, `addX` (anciennes versions gardées lors des modifications).
7. **PDF** : approximation des feuilles papier, jamais vérifiée à l'impression réelle (police « Segoe Print » avec repli).
8. **Mercredi midi** (slot `M`) : non concerné par les règles de Laura (c'est voulu, à confirmer).
9. **Recettes des 38 plats ajoutés** (tartiflette, curry, etc.) : plus courtes que celles des plats d'origine.
10. **Téléchargement du fichier texte** : jamais testé en conditions réelles (voir plus haut).
11. **Pas de sauvegarde** de l'état dans l'appli principale : un changement de téléphone ou un effacement du navigateur efface tout.

## 7. Ce qu'il reste à faire

Dans l'ordre de priorité demandé par Cyril :

1. **Export JSON Dukan** : un bouton « Export Cyril » et un bouton « Export Aurore » dans l'onglet Menu, visibles seulement si au moins un jour de la semaine a la case Dukan cochée avec une phase pour cette personne. Contenu : **uniquement** les infos Dukan (semaine, phase, midi et soir avec ingrédients en grammes et portions, skyr/son d'avoine, fruit). Rien de la famille, ni des courses, ni de l'autre personne.
2. **Appli Dukan secondaire réduite au suivi** : profil Cyril/Aurore, import du JSON ci-dessus, suivi graphique (courbe de poids + barres de grammes perdus ou repris), craquages (chocolat noir Nestlé Dessert, compote d'abricot Andros), activité (pas, vélo km et D+, muscu en minutes). Plus de plats dans cette appli.
3. **Sauvegarde / restauration** de l'état de l'appli principale (copier-coller de JSON), y compris l'ordre des rayons et les plats personnalisés.
4. **Tripler les plats famille** : on est à 88 plats pour 40 d'origine. Cible environ 120. Chaque nouveau plat doit avoir une **recette détaillée**. Étoffer aussi les recettes courtes.
5. **Questionnaire alimentaire d'Aurore** pour lui donner sa propre liste d'aliments Dukan (elle suivra Dukan plus tard).
6. **Lien avec le drive Super U** (site Courses U) : mémoriser une adresse produit par article, ou un lien de recherche, plus éventuellement un mode « Drive » (liste par catégories du site) et le remplissage du panier par Claude dans Chrome. L'appli ne peut pas appeler le site (restriction de l'environnement).
7. **Import du calendrier de congés** de Cyril (export JSON de son calendrier de travail) pour pré-cocher « Vacances » et gérer les congés.
8. **Idées proposées, pas encore retenues** : copier la semaine passée, éviter les mêmes plats sur 3 semaines, rappel de cuisiner en double quand « Restes » est prévu le lendemain, export du menu en texte pour message.
9. Nettoyer le code mort (voir problèmes connus).

## 8. Tester sans navigateur

La logique peut être testée avec Node en simulant le DOM. Méthode utilisée jusqu'ici :

1. Extraire le contenu de `<script>...</script>` du fichier HTML.
2. Ajouter au début des faux `document`, `window`, `localStorage` et `navigator` (objets avec des fonctions vides).
3. Ajouter à la fin un scénario (par exemple cocher Dukan, choisir une phase, appeler `dayH(...)`, `items()`, `sheetH(...)` et afficher des résultats).
4. Lancer avec `node`.

À vérifier à chaque modification : `DEF.length === FREC.length` (recettes alignées), les règles de Laura (semaine du matin et d'après-midi), le retrait d'une personne du menu standard quand Dukan est coché, les articles de courses issus des plats Dukan et des recettes.

## 9. Conseils pour modifier le code

- Ajouter un plat famille : ajouter une ligne **à la fin** de `DEF` et la recette correspondante **à la fin** de `FREC` (même ordre). Utiliser les mêmes noms d'ingrédients que les plats existants pour que les quantités se fusionnent (`×2`).
- Après un ajout de plats, la migration `S.dn` ajoute automatiquement les nouveaux plats aux copies personnalisées (`S.D`). Ne pas casser ce mécanisme.
- Un plat prêt de supermarché porte le tag `R` (ex. `LR`). Un plat rapide porte `Q`.
- Toute nouvelle donnée à garder doit passer par `S` puis `save()`.
- Garder les chaînes en français correct (accents, apostrophes `’` dans les textes lus).
- Garder la compatibilité mobile (une main, grands boutons) et le mode sombre.

## 10. Préférences de travail de Cyril

- **Langue** : français, toujours.
- **Explications simples**, sans jargon, **étape par étape**.
- Réponses courtes, directes, lisibles sur un téléphone.
- Il aime être **questionné précisément** (questions courtes à choix) avant de construire, et que les réponses soient mémorisées.
- Quand il dit « ne le fais pas maintenant » ou « dis-moi comment tu verrais la chose », **ne rien construire** : décrire d'abord, puis attendre son accord.
- Ne pas inventer de règles : demander quand une règle familiale n'est pas claire.
- Pour la nutrition et la marche, ton strict ; pour le vélo, souple selon l'emploi du temps familial.
- Vérifier chaque changement (tests, exemples concrets) avant de dire que c'est fait, et signaler franchement les limites.
