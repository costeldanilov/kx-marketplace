# kx — usine logicielle agentique pour Claude Code

> Les agents exécutent. L'humain décide.

`kx` est une marketplace de plugins Claude Code. Elle outille le cycle de vie d'un ticket, de sa création jusqu'à la release et au déploiement, avec des agents, des portes déterministes et une approbation humaine à chaque étape qui compte.

*English summary: `kx` is a Claude Code plugin marketplace for an agentic software factory. Agents do the work; humans approve every merge, release and deployment.*

> **État : 0.1.0, en construction.** Les plugins sont des squelettes. Ne les utilisez pas encore sur un dépôt de production.

## La règle de base

- L'auteur, le relecteur et l'approbateur sont toujours trois rôles distincts.
- Seul un humain approuve une PR, passe un ticket à Ready ou à Done, lance la release et le déploiement.
- Aucun agent ne fusionne, ne pose de tag, ne déploie au-delà de `dev`, ni n'écrit en production.

## Les plugins de ce dépôt

| Plugin | Rôle | Licence |
| --- | --- | --- |
| `kx-core` | Socle : profil du dépôt, bundle OKF, journal du ticket, scripts déterministes | Apache-2.0 |
| `kx-factory` | Flux d'étapes gardées : spécifier, planifier, coder, relire, vérifier | Apache-2.0 |
| `kx-kit-netfx` | Kit de stack .NET Framework | Apache-2.0 |
| `kx-kit-netfx-nuget` | Kit pour les bibliothèques NuGet .NET Framework | Apache-2.0 |
| `kx-kit-netfx-microservice` | Kit pour les microservices .NET Framework | Apache-2.0 |

Les experts de forge (Azure DevOps, puis GitHub) arrivent ensuite. Les modules `kx-compliance`, `kx-guild` et `kx-observability` sont des offres commerciales. Ils ne sont pas dans ce dépôt.

## Installer

```bash
claude plugin marketplace add costeldanilov/kx-marketplace
claude plugin install kx-core@kx
claude plugin install kx-factory@kx
```

Le plugin ne contient rien de propre à un projet. Tout le contexte du dépôt cible vient de son bundle de connaissances et de la configuration du plugin.

## Contribuer

Lisez [CONTRIBUTING.md](CONTRIBUTING.md). Chaque commit porte une ligne `Signed-off-by` (DCO).

## Sécurité

Ne signalez pas une faille dans une issue publique. Lisez [SECURITY.md](SECURITY.md).

## Licence et marques

Le code est sous licence [Apache-2.0](LICENSE). Voir aussi [NOTICE](NOTICE).

La licence ne donne aucun droit sur les noms « kx » et « KymexIA ». Lisez [TRADEMARKS.md](TRADEMARKS.md).

Claude et Claude Code sont des marques d'Anthropic. Ce projet n'est ni affilié à Anthropic, ni approuvé par Anthropic.
