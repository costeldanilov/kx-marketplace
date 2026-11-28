# Configurer un dépôt GitHub et son Project pour les plugins kx

État : 2026-11-28, réglages appliqués et relus par l'API sur le dépôt de l'exemple. Exemple réel : le dépôt du marketplace, https://github.com/costeldanilov/kx-marketplace, qui applique la **dérogation solo** (spécification §7.1) en attendant une identité dédiée aux agents ou un second relecteur humain.

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
   - cocher seulement **Allow merge commits** ;
   - cocher **Automatically delete head branches** ;
   - laisser **Allow auto-merge** décoché.
2. *Features* : garder **Issues** et **Projects** cochés. Décocher **Wikis** : la documentation vit dans `docs/`, où elle passe par une PR, une relecture et `validate`. Un wiki se modifie sans aucun de ces contrôles, et, sur un dépôt public, par n'importe quel compte si son édition n'est pas restreinte.

## 2. Ruleset de la branche par défaut

**Settings → Rules → Rulesets → New ruleset → New branch ruleset** (sur l'exemple : ruleset `Default branch`)

| Réglage | Valeur |
|---|---|
| Enforcement status | Active |
| Bypass list | Vide |
| Target branches | Default branch |
| Restrict deletions | Coché |
| Block force pushes | Coché |
| Require a pull request before merging | Coché |
| → Required approvals | **0** (dérogation solo) |
| → Dismiss stale pull request approvals when new commits are pushed | Coché |
| → Require review from Code Owners | **Décoché** pendant la dérogation : le propriétaire serait aussi l'auteur, la PR resterait bloquée |
| → Require approval of the most recent reviewable push | Décoché pendant la dérogation |
| → Require conversation resolution before merging | Coché |
| → Allowed merge methods | Merge |
| Require status checks to pass | Coché |
| → Require branches to be up to date before merging | Coché : après chaque fusion, les autres PR passent par **Update branch**, ce qui relance `validate` |
| → Status checks | `validate`, source **GitHub Actions** |
| Automatically request Copilot code review | Coché, à chaque push |

Pour ajouter le contrôle : *Add checks*, taper `validate`, puis choisir la source GitHub Actions. GitHub ne le propose qu'après une première exécution du workflow.

L'API montre aussi l'option `require_extra_approval_for_unattributed_changes`, active sur l'exemple, qui n'apparaît pas sous ce nom dans la liste ci-dessus. Elle ne bloque pas les PR de l'exemple. **[NV]** Son effet exact ; si une PR reste bloquée malgré 0 approbation exigée, vérifiez-la d'abord.

**Relecture Copilot.** La demande automatique peut ne pas partir après un push (constaté sur l'exemple après une modification du ruleset). Avant de fusionner, vérifiez que la dernière relecture de Copilot porte sur le dernier commit. Sinon, demandez-la à la main : **Reviewers → Copilot**.

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

| Réglage | Valeur |
|---|---|
| Private vulnerability reporting | Enable : `SECURITY.md` et le formulaire d'issue y renvoient |
| Dependency graph | Enable |
| Dependabot alerts | Enable |
| Dependabot security updates | Enable |
| Secret Protection, push protection | Enable |
| Code scanning : CodeQL, *Default setup* | Facultatif. Analyse les workflows (langage `actions`) |

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

Dans le Project : **⋯ → Settings → Fields → Status**. Le modèle Kanban crée déjà Backlog, Ready, In progress, In review et Done. Ajoutez Refinement, Blocked, Test et Verified, puis mettez les options dans cet ordre :

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

La couleur est libre ; conseil : gris pour Backlog et Refinement, bleu pour Ready à In review, orange pour Blocked, vert pour Test à Done.

### 6.3 Champs du ticket

**⋯ → Settings → Fields**

| Champ | Origine | Options | À faire |
|---|---|---|---|
| Priority | Modèle Kanban (P0, P1, P2) | P0, P1, P2, P3, P4 ; P0 est le plus urgent | Ajouter P3 et P4, dans cet ordre |
| Size | Modèle Kanban (XS à XL) | S, M, L, XL : poids du ticket (spécification §3.6) | Retirer XS |
| Sensitive | **New field**, single select | Yes, No | Créer le champ, avec les descriptions ci-dessous |

- **Priority** : des codes plutôt que des mots, car `critical` et `high` désignent déjà la gravité d'une faille (spécification §2.3).
- **Size** porte la classe du ticket : elle est **calculée par le script `weigh`**, ce n'est pas une estimation. Ne la modifiez pas à la main. La description d'un champ du modèle ne se modifie pas : la règle est donc écrite ici. `Estimate` reste libre pour une estimation humaine.
- **Sensitive**, descriptions à saisir :
  - champ : `Le ticket touche une zone sensible (sensitive-zones.yml) : sécurité, auth, réglages de l'usine. Fixé par un humain avant Ready.` ;
  - Yes : `Zone sensible touchée : relecture de sécurité et poids minimum M.` ;
  - No : `Aucune zone sensible. La PR échoue si elle en touche une.`
- Les champs `Estimate`, `Start date` et `Target date` du modèle sont sans effet sur l'usine.

Les critères d'acceptation et les décisions restent dans le corps de l'issue, par le formulaire d'issue ajouté au dépôt (§7).

### 6.4 Automatisations du Project

**⋯ → Workflows**

| Workflow | Réglage | Pourquoi |
|---|---|---|
| Item added to project | **Activé** : Status = Backlog | Tout ticket entre par le début |
| Item closed | **Désactivé** | Il traite aussi une fermeture « not planned » comme une livraison, alors que Done exige une release faite |
| Pull request merged | **Désactivé** | Une fusion mène à Test, pas à Done ; Done est un geste humain |
| Pull request linked to issue | Activé : Status = **In review** | Une PR liée ouverte, c'est l'état `review` |
| Auto-close issue | **Activé** | Le passage à Done ferme l'issue « completed » |
| Auto-add sub-issues to project | Activé | Les sous-issues suivent leur ticket |
| Auto-add to project | **Activé** : dépôt `kx-marketplace`, filtre `is:issue` | Chaque nouvelle issue arrive dans le tableau. Ce workflow est à créer : le modèle ne le fournit pas actif |
| Auto-archive items | Facultatif : `is:closed updated:<@today-30d` | Garde le tableau lisible |

Clore un ticket, à la main :

1. **Livré** : après la release, passez Status à Done. « Auto-close issue » ferme l'issue « completed ».
2. **Abandonné** : fermez l'issue avec **Close as not planned**, sans passer Status à Done.

### 6.5 Vues

1. **Board** (vue du modèle) : regroupée par Status.
2. Nouvelle vue **Table**, nom `Attente humaine`, filtre : `status:Refinement,Test,Verified`. Ce sont les tickets qui attendent votre décision.
3. Nouvelle vue **Table**, nom `Bloqués`, filtre : `status:Blocked`.

### 6.6 Limite à connaître

GitHub ne permet pas d'interdire un changement de Status selon la personne. Sur Azure DevOps, une règle de processus bloque le geste ; ici, il reste possible. Pendant la dérogation solo, vous êtes le seul humain : la règle « seul un humain passe à Ready, Test ou Done » repose sur la discipline et sur l'expert `kx-github`, qui refusera ces transitions à une identité d'agent. Un contrôle déterministe viendra avec `kx-github` (spec §2.13, D103).

## 7. Fichiers du dépôt qui complètent ces réglages

- `.github/CODEOWNERS` : les propriétaires du code, humains seulement. Sans effet tant que la revue des propriétaires est décochée ; prêt pour la fin de la dérogation ;
- `.github/ISSUE_TEMPLATE/story.yml` : formulaire de ticket avec les sections *Need*, *Acceptance criteria* (`AC-1`, `AC-2`…, dans la langue de travail du profil, D105) et *Decisions*. *Sensitive* n'y est pas : c'est un champ du Project (§6.3), pour garder une seule source ;
- `.github/ISSUE_TEMPLATE/config.yml` : renvoi des failles vers le signalement privé.

## 8. Ordre conseillé

1. §2 : ruleset de la branche par défaut.
2. §3 à §5 : ruleset des tags, sécurité, Actions.
3. §6 : le Project.
4. §7 : les fichiers du dépôt, par une PR.
