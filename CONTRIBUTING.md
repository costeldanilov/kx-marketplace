# Contribuer à kx

Merci de votre intérêt. Voici les règles.

## Avant de commencer

- Ouvrez une issue pour toute évolution qui n'est pas un petit correctif.
- Un plugin ne contient rien de propre à un projet ou à une entreprise : ni nom de produit, ni règle métier, ni règle d'un client.
- N'apportez pas de code, de prompt ou d'exemple que vous n'avez pas le droit de publier, par exemple du travail fait pour un employeur.
- N'apportez pas de code, de prompt ou d'exemple copié d'un autre projet, même open source. Un travail de la communauté se cite, avec un lien pour aller le découvrir ; il ne se recopie pas ici.

## Rédaction

- Français, phrases courtes.
- Les critères d'acceptation sont écrits dans la langue de travail du profil du dépôt, en rédaction contrôlée (`ste-en` ou `ste-fr`).
- Un `SKILL.md` fait moins de 500 lignes. Sa `description` fait 1 024 caractères au plus.

## Signature des commits (DCO)

Chaque commit porte une ligne `Signed-off-by`. Elle certifie que vous avez le droit de soumettre ce code sous la licence du projet ([Developer Certificate of Origin](https://developercertificate.org/)).

```bash
git commit -s -m "Votre message"
```

## Pull requests

- Une PR traite un seul sujet.
- La validation `claude plugin validate . --strict` passe.
- Une personne autre que l'auteur approuve la PR. Aucun agent n'approuve ni ne fusionne.
