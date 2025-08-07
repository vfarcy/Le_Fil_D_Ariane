# Le fil d'Ariane - Workflow d'Automatisation (GitHub & MailerLite)

![Statut du Workflow](https://github.com/vfarcy/Le_Fil_D_Ariane/actions/workflows/newsletter-mailerlite.yml/badge.svg)

> « Le sentier se crée en marchant. »
> — Francisco Varela

## 1. Objectif du Projet

Ce dépôt contient un système d'automatisation complet pour la newsletter **"Le fil d'Ariane"**. L'objectif est de créer un flux de travail "Push-to-Publish" où l'auteur se concentre exclusivement sur la rédaction d'articles en **Markdown**. Une fois l'article poussé (`git push`), une **GitHub Action** gère la création de la campagne, l'envoi d'un test, attend une **approbation manuelle**, puis distribue la newsletter et met à jour le site d'archives.

---

## 2. ⚠️ Avertissement : Prérequis Commercial

Ce workflow utilise l'API de MailerLite pour injecter du contenu HTML directement dans une campagne. Cette fonctionnalité est considérée comme une fonction "avancée" par MailerLite.

**Pour que ce système fonctionne, vous devez disposer d'un forfait MailerLite "Advanced" ou supérieur.**

Le forfait gratuit ou "Growing Business" ne permet pas cette automatisation et provoquera une erreur.

---

## 3. Fonctionnement Détaillé du Workflow (avec Validation)

Le processus est entièrement automatisé, à l'exception d'une étape de décision cruciale de votre part.

#### **Étape 1 : Le Déclenchement (Votre Action)**
Tout commence lorsque vous poussez (`git push`) un nouveau commit contenant un fichier `.md` dans le dossier `/_posts`.

#### **Étape 2 : Préparation et Test (L'Automate Travaille)**
La première tâche du workflow (`prepare_and_test_email`) s'exécute immédiatement :
-   Elle détecte le nouveau fichier que vous avez envoyé.
-   Elle crée une campagne **brouillon** sur votre compte MailerLite, en y injectant le contenu de votre article.
-   Elle envoie automatiquement un **e-mail de test** à l'adresse que vous avez définie dans le secret `MAILERLITE_TEST_EMAIL`.

#### **Étape 3 : La Pause pour Validation (Votre Décision)**
C'est ici que vous intervenez. Le workflow se met en **PAUSE**. La tâche `send_final_email` est mise en attente.
-   Vous recevez l'e-mail de test dans votre boîte de réception. Vous l'examinez attentivement (contenu, liens, mise en page).
-   Vous vous rendez sur l'onglet **`Actions`** de votre dépôt GitHub. Vous verrez que le workflow est "En attente d'approbation".
-   Vous cliquez sur **`Review deployments`**. Deux choix s'offrent à vous :

#### **Étape 4a : Scénario "Approuvé" ✅**
1.  Vous cliquez sur **`Approve and deploy`**.
2.  L'approbation débloque la tâche `send_final_email`, qui s'exécute.
3.  Le script envoie la campagne finale à votre groupe d'abonnés via MailerLite.
4.  Une fois l'envoi réussi, les tâches `build_site` et `deploy_site` se lancent séquentiellement pour publier votre article sur le site d'archives.
5.  **Résultat :** Votre newsletter est envoyée et l'archive est à jour.

#### **Étape 4b : Scénario "Rejeté" ❌**
1.  Vous cliquez sur **`Reject`**.
2.  La tâche `send_final_email` est immédiatement marquée comme **"Annulée" (Cancelled)**.
3.  Toutes les tâches qui en dépendaient (`build_site` et `deploy_site`) **ne se lanceront jamais**.
4.  **Résultat :** Rien n'est envoyé à vos abonnés, le site n'est pas mis à jour. Le brouillon reste sur MailerLite si vous souhaitez l'inspecter. Vous pouvez alors corriger votre fichier `.md` et relancer tout le processus avec un nouveau `push`.

---

## 4. Installation et Configuration

#### Étape A : Configuration de MailerLite
1.  Souscrire au forfait **"Advanced"** ou supérieur.
2.  Créer un **Groupe** d'abonnés et noter son **ID**.
3.  Générer une **Clé API** (`Integrations` > `API`).
4.  Vérifier votre **domaine d'envoi** (SPF/DKIM).

#### Étape B : Configuration du Dépôt GitHub
1.  **Créez l'Environnement `Production`** (`Settings > Environments`) et ajoutez-vous comme réviseur (`Required reviewers`).
2.  **Configurez les 5 Secrets** (`Settings > Secrets and variables > Actions`) :
    1.  `MAILERLITE_API_TOKEN`
    2.  `MAILERLITE_GROUP_ID`
    3.  `MAILERLITE_FROM_NAME`
    4.  `MAILERLITE_REPLY_TO_EMAIL`
    5.  `MAILERLITE_TEST_EMAIL`
3.  **Ajoutez le fichier de workflow** `.github/workflows/newsletter-mailerlite.yml`.

---

## 5. Utilisation Quotidienne

1.  **Rédiger** : Créez un nouveau fichier au format `AAAA-MM-JJ-titre.md` dans le dossier `/_posts`.
2.  **Déclencher** : Faites un `git push` pour envoyer votre article.
3.  **Valider** : Recevez le test, puis allez sur l'onglet **`Actions`** de GitHub pour **Approuver** ou **Rejeter** l'envoi.
