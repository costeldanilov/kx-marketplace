# Configurer un dépôt GitHub et son Project pour les plugins kx

État : 2026-11-27. Exemple réel : le dépôt du marketplace, https://github.com/costeldanilov/kx-marketplace, qui applique la **dérogation solo** (spécification §7.1) en attendant une identité dédiée aux agents ou un second relecteur humain.

Chaque étape se fait dans l'interface de GitHub, par le propriétaire du dépôt. Aucun agent ne modifie ces réglages : ce sont eux qui protègent le dépôt contre les agents.

## 0. La dérogation solo, en clair

| Élément | Valeur |
|---|---|
| Motif | Un seul humain sur le projet. Les PR des agents sont créées sous son identité ; GitHub ne permet pas d'approuver sa propre PR |
| Date | 2026-11-27 |
| Date de revue | 2027-02-27, ou dès qu'une identité d'agents (GitHub App) ou un second relecteur existe |
| Ce qui est levé | L'approbation par une autre personne |
| Contrôles compensatoires | Attendre la **fin** de la relecture Copilot du dernier push, puis traiter chacun de ses fils, avant de fusionner (le ruleset ne bloque pas une fusion pendant que Copilot travaille) ; contrôle `validate` obligatoire ; lecture complète du diff par l'humain, en particulier de tout changement sous `.github/workflows/` ; aucun contournement du ruleset |
| Ce qui ne change pas | Aucun agent ne fusionne, ne pose de tag, ne passe un ticket à Ready ou à Done |

La dérogation sera aussi écrite dans le profil du dépôt (`.knowledge/factory/profile.yml`) quand `kx-core` existera.

## 1. Réglages généraux

**Settings → General**

1. *Pull Requests* :
   - cocher seulement **Allow merge commits** (déjà fait) ;
   - cocher **Automatically delete head branches** ;
   - laisser **Allow auto-merge** décoché.
2. *Features* : garder **Issues** et **Projects** cochés. **Wikis** peut être décoché : la documentation vit dans `docs/`.

## 2. Ruleset de la branche par défaut

**Settings → Rules → Rulesets → « Default branch protected and auto review by Copilot » → Edit**

| Réglage | Valeur | État actuel |
|---|---|---|
| Enforcement status | Active | ✅ déjà |
| Bypass list | Vide | ✅ déjà |
| Target branches | Default branch | ✅ déjà |
| Restrict deletions | Coché | ✅ déjà |
| Block force pushes | Coché | ✅ déjà |
| Require a pull request before merging | Coché | ✅ déjà |
| → Required approvals | **0** (dérogation solo) | ❌ à changer (1 aujourd'hui) |
| → Dismiss stale pull request approvals when new commits are pushed | Coché | ✅ déjà |
| → Require review from Code Owners | **Décoché** pendant la dérogation | ❌ à changer : le propriétaire serait aussi l'auteur, la PR resterait bloquée |
| → Require approval of the most recent reviewable push | Décoché pendant la dérogation | ✅ déjà |
| → Require conversation resolution before merging | Coché | ✅ déjà |
| → Allowed merge methods | Merge | ✅ déjà |
| Require status checks to pass | **Coché** | ❌ à ajouter |
| → Require branches to be up to date before merging | Coché | à ajouter |
| → Status checks | `validate`, source **GitHub Actions** | à ajouter |
| Automatically request Copilot code review | Coché, à chaque push | ✅ déjà |

Pour ajouter le contrôle : *Add checks*, taper `validate`, puis choisir la source GitHub Actions. Le contrôle a déjà tourné sur `main` : GitHub le propose dans la liste.

**Limite à connaître.** Le contrôle `validate` vient d'un workflow déclenché par `pull_request` : il s'exécute dans la version de la branche de la PR. Une PR peut donc modifier ce workflow et publier un succès sous le même nom. Tant qu'aucune validation de confiance n'existe (spécification §2.13), lisez toujours les changements de `.github/workflows/` avant de fusionner.

Fin de la dérogation : remettre **Required approvals** à 1, cocher **Require review from Code Owners** et **Require approval of the most recent reviewable push**.

## 3. Ruleset des tags

**Settings → Rules → Rulesets → New ruleset → New tag ruleset**

| Réglage | Valeur |
|---|---|
| Ruleset name | `Tags` |
| Enforcement status | Active |
| Bypass list | **Repository admin** : les mainteneurs humains, qui posent les tags |
| Target tags | *Add target → Include all tags* : aucun tag n'échappe à la règle, quelle que soit sa forme |
| Restrict creations | Coché |
| Restrict updates | Coché |
| Restrict deletions | Coché |
| Block force pushes | Coché |

Aucun agent ne crée de tag, quel qu'il soit. Les tags de version ont la forme `<plugin>--v<version>` et servent à résoudre les dépendances entre plugins. Seul un humain les pose (D96).

## 4. Sécurité

**Settings → Advanced Security** (ou **Code security** selon l'interface)

| Réglage | Valeur | État actuel |
|---|---|---|
| Private vulnerability reporting | Enable | ❌ désactivé, alors que `SECURITY.md` y renvoie |
| Dependency graph | Enable | à vérifier |
| Dependabot alerts | Enable | à activer |
| Dependabot security updates | Enable | ❌ désactivé |
| Secret Protection, push protection | Enable | ✅ déjà |

## 5. GitHub Actions

**Settings → Actions → General**

1. *Workflow permissions* : **Read repository contents and packages permissions**.
2. Décocher **Allow GitHub Actions to create and approve pull requests**.
3. *Fork pull request workflows* : garder l'approbation requise pour les contributeurs extérieurs.

Vérifiez ces réglages à la main : `doctor` ne les lit pas encore.

## 6. Le GitHub Project : le tableau des tickets

Le Project joue le rôle du tableau Azure Boards. Les états reprennent les rôles de la spécification (§2.4), dans le même ordre.

### 6.1 Créer le Project

1. Où créer le Project :
   - **en dérogation solo**, sur le profil du propriétaire (onglet **Projects → New project**) : l'expert agit avec le jeton du développeur ;
   - **dès qu'une GitHub App d'agents existe**, dans une **organisation** : l'API REST des Projects d'un compte personnel n'accepte pas les jetons d'une GitHub App, ni les jetons fins. Une organisation gratuite suffit ; le dépôt peut y être transféré (GitHub redirige l'ancienne adresse).
2. Choisir le modèle **Kanban**, nom : `kx-marketplace`.
3. Dans le Project : **⋯ → Settings → Manage access** : garder le Project **privé** au début. Il pourra devenir public plus tard.
4. Lier le dépôt : depuis le dépôt, onglet **Projects → Link a project → kx-marketplace**.

### 6.2 Les états : champ `Status`

Dans le Project : **⋯ → Settings → Fields → Status**. Remplacer les options du modèle par celles-ci, dans cet ordre :

| Option | Rôle (spec §2.4) | Qui y met un ticket | Signification |
|---|---|---|---|
| Backlog | `backlog` | Tous | Idée ou besoin, pas encore travaillé |
| Refinement | `refinement` | Tous | En cours de spécification avec l'agent |
| Ready | `ready` | **Humain seulement** (`spec_approvers`) | Critères figés, ticket prêt à être réalisé |
| In progress | `doing` | Développeur ou agent | En réalisation |
| Blocked | `blocked` | Tous | Bloqué ; l'état d'origine est noté en commentaire |
| In review | `review` | Développeur ou agent | Une PR liée est ouverte |
| Test | `test` | **Humain seulement** | PR approuvée et fusionnée |
| Verified | `verified` | Testeur ou agent | Chaque critère a un résultat |
| Done | `done` | **Humain seulement** | Livré ; l'issue est fermée « completed » |

Le modèle Kanban crée « Todo » : renommez-le en `Backlog`. La couleur est libre ; conseil : gris pour Backlog et Refinement, bleu pour Ready à In review, orange pour Blocked, vert pour Test à Done.

### 6.3 Champs en plus

**⋯ → Settings → Fields → New field**

| Champ | Type | Options |
|---|---|---|
| Priority | Single select | P1, P2, P3, P4 |
| Sensitive | Single select | Yes, No |
| Class | Single select | S, M, L, XL (poids du ticket, §3.6) |

Les critères d'acceptation et les décisions restent dans le corps de l'issue, par le formulaire d'issue ajouté au dépôt (§7).

### 6.4 Automatisations du Project

**⋯ → Workflows**

| Workflow | Réglage | Pourquoi |
|---|---|---|
| Item added to project | **Activé** : Status = Backlog | Tout ticket entre par le début |
| Item closed | **Désactivé** | Il traite aussi une fermeture « not planned » comme une livraison, alors que Done exige une release faite |
| Pull request merged | **Désactivé** | Une fusion mène à Test, pas à Done ; Done est un geste humain |
| Auto-add to project | **Activé** : dépôt `kx-marketplace`, filtre `is:issue` | Chaque nouvelle issue arrive dans le tableau |
| Auto-archive items | Facultatif : `is:closed updated:<@today-30d` | Garde le tableau lisible |

Clore un ticket, à la main :

1. **Livré** : après la release, passez Status à Done, puis fermez l'issue avec **Close as completed**. Déplacer la carte ne ferme pas l'issue.
2. **Abandonné** : fermez l'issue avec **Close as not planned**, sans passer Status à Done.

### 6.5 Vues

1. **Board** (vue du modèle) : regroupée par Status.
2. Nouvelle vue **Table**, nom `Attente humaine`, filtre : `status:Refinement,Test,Verified`. Ce sont les tickets qui attendent votre décision.
3. Nouvelle vue **Table**, nom `Bloqués`, filtre : `status:Blocked`.

### 6.6 Limite à connaître

GitHub ne permet pas d'interdire un changement de Status selon la personne. Sur Azure DevOps, une règle de processus bloque le geste ; ici, il reste possible. Pendant la dérogation solo, vous êtes le seul humain : la règle « seul un humain passe à Ready, Test ou Done » repose sur la discipline et sur l'expert `kx-github`, qui refusera ces transitions à une identité d'agent. Un contrôle déterministe viendra avec `kx-github` (spec §2.13, D103).

## 7. Fichiers du dépôt qui complètent ces réglages

- `.github/CODEOWNERS` : les propriétaires du code, humains seulement. Sans effet tant que la revue des propriétaires est décochée ; prêt pour la fin de la dérogation ;
- `.github/ISSUE_TEMPLATE/story.yml` : formulaire de ticket avec les sections *Need*, *Acceptance criteria* (`AC-1`, `AC-2`… en ASD-STE100), *Decisions*, *Sensitive* ;
- `.github/ISSUE_TEMPLATE/config.yml` : renvoi des failles vers le signalement privé.

## 8. Ordre conseillé

1. §2 : ruleset de la branche par défaut.
2. §3 à §5 : ruleset des tags, sécurité, Actions.
3. §6 : le Project.
4. §7 : les fichiers du dépôt, par une PR.
