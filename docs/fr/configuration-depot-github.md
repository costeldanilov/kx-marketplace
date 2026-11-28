# Configurer un dépôt GitHub et son Project pour les plugins kx

État : 2026-11-28. Réglages appliqués sur le dépôt de l'exemple, puis relus par l'API. Exemple réel : le dépôt du marketplace, https://github.com/costeldanilov/kx-marketplace. Il applique la **dérogation solo** (spécification §7.1) en attendant une identité dédiée aux agents ou un second relecteur humain.

Chaque réglage se fait dans l'interface de GitHub, par le propriétaire du dépôt. Aucun agent ne modifie ces réglages : ce sont eux qui protègent le dépôt contre les agents. Un agent peut les **lire** par l'API pour vérifier le guide.

Pour chaque réglage, le guide donne la valeur et la raison. Une raison renvoie souvent à une règle de base :

- **R1** : l'auteur, le relecteur et l'approbateur sont trois rôles distincts ;
- **R2** : seul un humain approuve une PR, passe un ticket à Ready, à Test ou à Done, pose un tag, livre et déploie ;
- **R3** : un contrôle déterministe passe avant le jugement d'un modèle ; il ne se contourne pas ;
- **R4** : toute modification du dépôt passe par une PR, une relecture et `validate`.

## 0. La dérogation solo

| Élément | Valeur | Pourquoi |
|---|---|---|
| Motif | Un seul humain sur le projet ; les PR des agents sont créées sous son identité | GitHub ne permet pas d'approuver sa propre PR. Sans dérogation, aucune PR ne peut être fusionnée |
| Date | 2026-11-27 | Une dérogation est datée, pour être revue |
| Date de revue | 2027-02-27, ou dès qu'une identité d'agents (GitHub App) ou un second relecteur existe | La dérogation lève R1 : elle doit rester courte |
| Ce qui est levé | L'approbation par une autre personne | Seulement cela. R2, R3 et R4 restent en vigueur |
| Contrôles compensatoires | 1. Attendre la **fin** de la relecture Copilot du dernier push, puis traiter chacun de ses fils. 2. `validate` réussi. 3. Lire tout le diff, en particulier `.github/workflows/`. 4. Aucun contournement du ruleset | Ils remplacent le regard d'un second humain. Le ruleset ne bloque pas une fusion pendant que Copilot travaille : l'attente est un geste humain |
| Ce qui ne change pas | Aucun agent ne fusionne, ne pose de tag, ne passe un ticket à Ready, à Test ou à Done | R2 |

La dérogation sera aussi écrite dans le profil du dépôt (`.knowledge/factory/profile.yml`) quand `kx-core` existera.

**Fin de la dérogation** : remettre **Required approvals** à 1 ; cocher **Require review from Code Owners** et **Require approval of the most recent reviewable push** (§2).

## 1. Réglages généraux

**Settings → General**

| Réglage | Valeur | Pourquoi |
|---|---|---|
| Allow merge commits | Coché, **seule méthode** | Le commit de fusion garde les commits de la PR, avec leur ligne `Signed-off-by` (DCO) et leur auteur. Un squash ou un rebase réécrit les commits |
| Allow squash merging, Allow rebase merging | Décochés | Idem |
| Allow auto-merge | **Décoché** | Une fusion automatique part dès que les contrôles passent, sans geste humain (R2) |
| Automatically delete head branches | Coché | Les branches fusionnées disparaissent ; les branches restantes sont le travail en cours |
| Issues | Coché | Les tickets sont des issues (spécification §2.13) |
| Projects | Coché | Le Project porte les états des tickets (§6) |
| Wikis | **Décoché** | La documentation vit dans `docs/`, où elle passe par R4. Un wiki se modifie sans PR, sans relecture et sans `validate` ; sur un dépôt public, par n'importe quel compte si son édition n'est pas restreinte |

## 2. Ruleset de la branche par défaut

**Settings → Rules → Rulesets → New ruleset → New branch ruleset.** Sur l'exemple : ruleset `Default branch`.

| Réglage | Valeur | Pourquoi |
|---|---|---|
| Enforcement status | Active | Un ruleset en mode *Evaluate* ne bloque rien |
| Bypass list | **Vide** | Un acteur qui contourne peut fusionner une PR bloquée. Même l'administrateur passe par les contrôles (R3) ; une urgence passe aussi par une PR |
| Target branches | Default branch | La branche `main` est celle que l'on livre |
| Restrict deletions | Coché | `main` ne peut pas être supprimée |
| Block force pushes | Coché | L'historique de `main` ne peut pas être réécrit : la trace des fusions reste |
| Require a pull request before merging | Coché | R4 : personne ne pousse directement sur `main` |
| → Required approvals | **0** pendant la dérogation ; 1 ensuite | L'auteur ne peut pas approuver sa PR. Les contrôles compensatoires du §0 tiennent lieu d'approbation |
| → Dismiss stale pull request approvals when new commits are pushed | Coché | Une approbation porte sur un état du code. Un nouveau push l'annule |
| → Require review from Code Owners | **Décoché** pendant la dérogation ; coché ensuite | Le propriétaire serait aussi l'auteur : la PR resterait bloquée. `CODEOWNERS` est prêt pour la fin de la dérogation (§7) |
| → Require approval of the most recent reviewable push | Décoché pendant la dérogation ; coché ensuite | Ensuite, celui qui pousse le dernier commit ne peut pas être celui qui approuve (R1) |
| → Require conversation resolution before merging | Coché | Chaque fil de relecture, de Copilot ou d'un humain, est traité avant la fusion |
| → Allowed merge methods | Merge | Même raison qu'au §1 |
| Require status checks to pass | Coché | R3 : un contrôle en échec bloque la fusion |
| → Require branches to be up to date before merging | Coché | `validate` porte sur le code qui sera vraiment fusionné. Après chaque fusion, les autres PR passent par **Update branch**, ce qui relance `validate` |
| → Status checks | `validate`, source **GitHub Actions** | Le contrôle du marketplace : plugins valides, aucun plugin commercial, aucun secret. La source empêche un autre outil de publier un faux succès sous ce nom |
| Automatically request Copilot code review | Coché, à chaque push | Une relecture par un second regard, même sans second humain (§0) |

Pour ajouter le contrôle : *Add checks*, taper `validate`, puis choisir la source GitHub Actions. GitHub ne le propose qu'après une première exécution du workflow.

**Option visible seulement par l'API.** `require_extra_approval_for_unattributed_changes` est active sur l'exemple. Elle ne bloque pas les PR de l'exemple. **[NV]** Son effet exact. Si une PR reste bloquée malgré 0 approbation exigée, vérifiez-la d'abord.

**Relecture Copilot manquante.** La demande automatique peut ne pas partir après un push. C'est arrivé sur l'exemple après une modification du ruleset. Avant de fusionner, vérifiez que la dernière relecture de Copilot porte sur le dernier commit. Sinon, demandez-la : **Reviewers → Copilot**.

**Limite de `validate`.** Le workflow est déclenché par `pull_request` : il s'exécute dans la version de la branche de la PR. Une PR peut donc modifier ce workflow et publier un succès sous le même nom. Tant qu'aucune validation de confiance n'existe (spécification §2.13), lisez toujours les changements de `.github/workflows/` avant de fusionner.

## 3. Ruleset des tags

**Settings → Rules → Rulesets → New ruleset → New tag ruleset.** Sur l'exemple : ruleset `Tags`.

| Réglage | Valeur | Pourquoi |
|---|---|---|
| Enforcement status | Active | Idem §2 |
| Bypass list | **Repository admin** | Les mainteneurs humains posent les tags (R2). Aucune identité d'agent dans cette liste |
| Target tags | *Include all tags* | Aucun tag n'échappe à la règle, quelle que soit sa forme. Un motif comme `*--v*` laisserait un agent créer un autre tag |
| Restrict creations | Coché | Seuls les acteurs de la liste de contournement créent un tag |
| Restrict updates | Coché | Un tag publié ne bouge plus : il désigne toujours le même commit |
| Restrict deletions | Coché | Un tag publié reste : les dépendances entre plugins s'y réfèrent |
| Block force pushes | Coché | Idem |

Les tags de version ont la forme `<plugin>--v<version>`. Ils servent à résoudre les dépendances entre plugins (D96).

**Limite pendant la dérogation.** Les agents agissent avec votre jeton, qui est administrateur : ce ruleset ne les arrête pas. L'interdit « aucun agent ne pose de tag » repose sur l'expert et sur votre discipline, jusqu'à ce qu'une identité d'agents existe (spécification §7.1).

## 4. Sécurité

**Settings → Advanced Security** (ou **Code security**, selon l'interface)

| Réglage | Valeur | Pourquoi |
|---|---|---|
| Private vulnerability reporting | Enable | Une faille se signale en privé, pas dans une issue publique. `SECURITY.md` et le formulaire d'issue y renvoient |
| Dependency graph | Enable | Base des alertes de dépendances |
| Dependabot alerts | Enable | Signale une dépendance qui a une faille connue |
| Dependabot security updates | Enable | Propose la correction par une PR, qui passe par R4 |
| Secret Protection, push protection | Enable | Bloque un push qui contient un secret, avant qu'il soit public |
| Code scanning : CodeQL, *Default setup* | Facultatif ; actif sur l'exemple | Analyse les workflows (langage `actions`), qui sont le point le plus exposé du dépôt |

## 5. GitHub Actions

**Settings → Actions → General**

| Réglage | Valeur | Pourquoi |
|---|---|---|
| Workflow permissions | **Read repository contents and packages permissions** | Le jeton d'un workflow ne peut pas écrire dans le dépôt par défaut. Un workflow qui doit écrire le déclare lui-même |
| Allow GitHub Actions to create and approve pull requests | **Décoché** | Une approbation par Actions compterait comme celle d'un relecteur (R1, R2) |
| Fork pull request workflows | Approbation requise pour les contributeurs extérieurs | Le code d'un inconnu ne s'exécute pas sans qu'un humain l'ait regardé |

`doctor` ne lit pas encore ces réglages : vérifiez-les à la main.

## 6. Le GitHub Project : le tableau des tickets

Le Project porte l'état des tickets. Les états reprennent les rôles de la spécification (§2.4), dans le même ordre.

### 6.1 Créer le Project

| Étape | Valeur | Pourquoi |
|---|---|---|
| Propriétaire | **En dérogation solo** : le compte du propriétaire (**Projects → New project**). **Dès qu'une GitHub App d'agents existe** : une **organisation** | L'API REST des Projects d'un compte personnel n'accepte pas les jetons d'une GitHub App, ni les jetons fins. En dérogation solo, l'expert agit avec le jeton du développeur. Une organisation gratuite suffit ; le dépôt peut y être transféré, et GitHub redirige l'ancienne adresse |
| Modèle | **Kanban** | Il crée Status, Priority, Size et la vue Board, que le guide adapte |
| Nom | Le nom du dépôt (`kx-marketplace`) | Un seul Project par dépôt. Supprimez un ancien Project du même nom : on se trompe de Project sinon |
| Visibilité (**⋯ → Settings → Manage access**) | **Privé** au début | Les issues sont publiques avec le dépôt ; le Project privé cache seulement les champs de pilotage. Il pourra devenir public, par exemple pour publier la feuille de route |
| Lien avec le dépôt | Depuis le dépôt : **Projects → Link a project** | Le Project apparaît dans le dépôt, et le workflow « Auto-add » peut viser le dépôt |

### 6.2 Les états : champ `Status`

**⋯ → Settings → Fields → Status.** Le modèle Kanban crée Backlog, Ready, In progress, In review et Done. Ajoutez Refinement, Blocked, Test et Verified, puis mettez les options **dans cet ordre** :

| Option | Rôle (spec §2.4) | Qui y met un ticket | Signification |
|---|---|---|---|
| Backlog | `backlog` | Tous | Idée ou besoin, pas encore travaillé |
| Refinement | `refinement` | Tous ; `ticket specify` y laisse le ticket | En cours de spécification avec l'agent |
| Ready | `ready` | **Humain seulement** (`spec_approvers`) | Critères et `Sensitive` figés ; ticket prêt à être réalisé |
| In progress | `doing` | Développeur ou agent | En réalisation |
| Blocked | `blocked` | Tous | Bloqué ; l'état d'origine est noté en commentaire |
| In review | `review` | Développeur, agent, ou le workflow « Pull request linked to issue » | Une PR liée est ouverte |
| Test | `test` | **Humain seulement** | PR fusionnée, à vérifier |
| Verified | `verified` | Testeur ou agent | Chaque critère a un résultat |
| Done | `done` | **Humain seulement** | Livré ; l'issue est fermée « completed » |

Pourquoi cet ordre : c'est l'ordre du cycle. Les vues et les scripts lisent l'ordre des options. Pourquoi ces noms : ce sont les noms réels que le profil associe aux rôles (spécification §2.3).

La couleur est libre. Conseil : gris pour Backlog et Refinement, bleu pour Ready à In review, orange pour Blocked, vert pour Test à Done.

### 6.3 Champs du ticket

| Champ | Où | Options | À faire | Pourquoi |
|---|---|---|---|---|
| Priority | Champ du Project, créé par le modèle (P0 à P2) | P0, P1, P2, P3, P4 ; P0 est le plus urgent | Ajouter P3 et P4, dans cet ordre | Des codes plutôt que des mots : `critical` et `high` désignent déjà la gravité d'une faille (spécification §2.3). L'ordre des options dit quelle priorité est la plus haute |
| Size | Champ du Project, créé par le modèle (XS à XL) | S, M, L, XL | Retirer XS | Il porte la classe du ticket, **calculée par le script `weigh`** (spécification §3.6), qui ne produit pas XS. Ce n'est pas une estimation. Le développeur ou le lead dev peut monter ou baisser la classe, avec une raison notée dans le dossier du ticket, jamais sous un plancher. La description d'un champ du modèle ne se modifie pas : la règle est donc écrite ici |
| Sensitive | **Issue field**, choix simple, porté par l'issue | Yes, No | Créer le champ avec les descriptions ci-dessous, puis l'**épingler** | Il suit le ticket dans tout Project, et chaque changement est tracé avec son auteur et ses valeurs. `criteria-locked` le lit pour bloquer un passage de Yes à No après Ready (spécification §2.4, §2.13) |

**Descriptions de Sensitive**

- Champ : `Le ticket touche une zone sensible (sensitive-zones.yml) : sécurité, auth, réglages de l'usine. Fixé par un humain avant Ready, puis figé : repasser à No exige un nouveau Ready.`
- Yes : `Zone sensible touchée : relecture de sécurité et poids minimum M.`
- No : `Aucune zone sensible. La PR échoue si elle en touche une.`

Pourquoi : la description rappelle la règle à celui qui choisit la valeur.

**Afficher Sensitive.** Un Issue field ne s'affiche dans l'issue que s'il est **épinglé** : roue dentée de **Fields**, dans la barre latérale de l'issue, puis cocher **Sensitive**. Sans cela, l'issue montre « No fields added », même si une valeur existe. Une valeur posée par l'API peut demander de recharger la page. Le champ se modifie dans les réglages du propriétaire (lien *project owner's settings* dans les réglages du champ).

Les champs `Estimate`, `Start date` et `Target date` du modèle sont sans effet sur l'usine. Vous pouvez les garder.

Les critères d'acceptation et les décisions restent dans le corps de l'issue, par le formulaire d'issue (§7).

### 6.4 Automatisations du Project

**⋯ → Workflows.** Cliquez sur un workflow dans la liste de gauche ; l'interrupteur **On/Off** est en haut à droite du panneau. Une modification se valide par **Save** ou **Save and turn on**.

| Workflow | Réglage | Pourquoi |
|---|---|---|
| Item added to project | **On** : Status = Backlog | Tout ticket entre par le début du cycle |
| Auto-add to project | **On** : dépôt `kx-marketplace`, filtre `is:issue` | Chaque nouvelle issue arrive dans le tableau. Le modèle ne le fournit pas actif : il est à créer |
| Auto-add sub-issues to project | On | Les sous-issues suivent leur ticket |
| Pull request linked to issue | **On** : Status = **In review** | Une PR liée ouverte, c'est l'état `review`. Attention : les PR de l'usine portent `Refs #<n>`, qui ne crée pas de lien. Ce workflow ne se déclenche donc pas pour elles : l'expert passe lui-même le ticket à In review (spécification §2.13, `change.open`). Sans expert, faites-le à la main |
| Item closed | **Off** | Il traite aussi une fermeture « not planned » comme une livraison, alors que Done exige une release faite |
| Pull request merged | **Off** | Une fusion mène à Test, pas à Done. Actif, il passerait le ticket à Done à la fusion (R2) |
| Auto-close issue | **On** | Le passage humain à Done ferme l'issue « completed » : un seul geste |
| Auto-archive items | Facultatif : `is:closed updated:<@today-30d` | Garde le tableau lisible |

**Clore un ticket**

1. **Livré** : après la release, passez Status à Done. « Auto-close issue » ferme l'issue « completed ».
2. **Abandonné** : fermez l'issue avec **Close as not planned**, sans passer Status à Done. Pourquoi : Done veut dire livré.

### 6.5 Vues

| Vue | Réglage | Pourquoi |
|---|---|---|
| Board (vue du modèle) | Regroupée par Status | Le cycle en un coup d'œil |
| `Attente humaine` | Nouvelle vue **Table**, filtre `status:Refinement,Test,Verified` | Les tickets qui attendent un geste humain : passage à Ready, vérification, Done |
| `Bloqués` | Nouvelle vue **Table**, filtre `status:Blocked` | Un blocage se traite vite ou se signale |

### 6.6 Limite à connaître

GitHub ne permet pas d'interdire un changement de Status selon la personne. Le geste reste possible ; le contrôle se fait **après coup**. La chronologie de l'issue donne, pour chaque changement de Status ou de Sensitive, l'auteur, la date et les valeurs. L'expert `kx-github` refusera les transitions interdites à une identité d'agent ; un contrôle planifié lira la chronologie et signalera un passage à Ready, Test ou Done fait par un agent (spécification §2.13, D103). **Pendant la dérogation solo, ce contrôle est aveugle** : les agents agissent avec votre jeton, et la chronologie vous désigne comme auteur de chaque geste. La règle repose alors sur l'expert et sur votre discipline (spécification §7.1).

## 7. Fichiers du dépôt

Ils complètent les réglages. Ils sont dans la zone `factory` : chaque modification passe par une PR lue en entier.

| Fichier | Contenu | Pourquoi |
|---|---|---|
| `.github/CODEOWNERS` | `* @<propriétaire>` : des humains seulement | Prêt pour la fin de la dérogation, quand la revue des propriétaires sera exigée. Jamais une identité d'agent : son approbation compterait (R1) |
| `.github/ISSUE_TEMPLATE/story.yml` | Formulaire « Ticket » : *Need*, *Acceptance criteria* (`AC-1`, `AC-2`…, dans la langue de travail du profil, D105), *Decisions* | L'expert `kx-github` lit ces titres : ne les renommez pas. *Sensitive* n'y est pas : c'est un Issue field (§6.3). Une seule source par donnée |
| `.github/ISSUE_TEMPLATE/config.yml` | Issues vides désactivées ; lien vers le signalement privé | Chaque issue a la forme d'un ticket ; une faille ne passe pas par une issue publique |

## 8. Ordre conseillé

| Étape | Pourquoi dans cet ordre |
|---|---|
| 1. §1 et §2 | Le ruleset de `main` débloque les PR (0 approbation) et impose `validate` avant toute autre modification |
| 2. §3 à §5 | Tags, sécurité et Actions : aucun ne dépend des autres |
| 3. §6 | Le Project : les tickets peuvent ensuite suivre le cycle |
| 4. §7 | Les fichiers, par une PR liée à un ticket : la PR suit déjà les règles des §2 et §6 |

## 9. Vérifier

Un agent peut relire ces réglages par l'API, sans les modifier : `gh api` pour le dépôt, les rulesets et la sécurité ; GraphQL pour le Project. Deux pièges constatés sur l'exemple :

- les options d'un Issue field ne se lisent pas par l'API des Projects (liste vide), mais par `Repository.issueFields` ;
- l'API dit si un workflow du Project est actif, pas la valeur qu'il pose : le réglage de « Pull request linked to issue » se vérifie dans l'interface.
