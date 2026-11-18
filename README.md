# contentdb

Journal des déclarations de la base des contenus de l'OFCE, du CEPII et de l'IFE.

Chaque contenu publié (document de travail, policy brief, blog, revue…) porte un **manifeste** qui le décrit. Pour qu'il entre dans la base des contenus, on le **déclare** ici : un petit fichier JSON qui donne l'adresse de son manifeste. La collecte lit ce journal, télécharge les manifestes et construit la base. Si la base est perdue, rejouer le journal la reconstruit à l'identique.

Le dépôt est public : une déclaration se lit sans jeton, par exemple
<https://raw.githubusercontent.com/ofceweb/contentdb/main/registry/2026/ofce-wp-2026-6-fr.json>.

La base elle-même (`contents.json`) n'est jamais committée ici : elle est publiée sur le serveur par la collecte.

## Une déclaration

Un fichier par contenu, rangé dans `registry/{annee}/{id}.json` :

```json
{
  "id": "ofce-wp-2026-6-fr",
  "manifest_url": "https://www.ofce.fr/wp/2026/6/v0/manifest.json",
  "pdf_url": "https://www.ofce.fr/wp/2026/6/v0/OFCEWP2026-6.pdf",
  "declared_at": "2026-06-15T14:32:00Z",
  "declared_by": "xtimbeau",
  "status": "published"
}
```

| Champ | Contenu |
|---|---|
| `id` | Identifiant du contenu, celui de son manifeste. C'est aussi le nom du fichier, sans `.json`. |
| `manifest_url` | Adresse `https://` du manifeste publié. Pour un contenu versionné, c'est l'adresse de la version publiée (`…/v0/manifest.json`) : une nouvelle version se déclare en changeant cette adresse. |
| `pdf_url` | Adresse `https://` du PDF, ou `null` s'il n'y a pas de PDF. Le champ est obligatoire, même à `null`. |
| `declared_at` | Date et heure de la déclaration, en UTC, au format `AAAA-MM-JJThh:mm:ssZ`. |
| `declared_by` | Votre login GitHub. |
| `status` | `published` (en ligne) ou `retired` (retiré). |

Aucun autre champ n'est admis. Le format complet est le schéma [`declaration-1.0.json`](https://raw.githubusercontent.com/ofceweb/ofceweb/main/inst/schema/declaration-1.0.json) du package `ofceweb`.

### Le rangement

- `{annee}` est l'année de **première publication** du contenu : celle de `date_published` dans son manifeste. Pour le blog (`ofce-blog`), c'est l'année de `date_published` de l'entrée du blog lui-même dans son manifeste collectif.
- L'année ne change plus ensuite : une nouvelle version publiée une autre année reste dans le même fichier.

## Déclarer un contenu

Deux façons, équivalentes. Dans les deux cas, le contrôle automatique s'exécute (onglet **Actions**).

**Par commit direct** sur `main` : ajouter le fichier `registry/{annee}/{id}.json`, committer, pousser. Rapide, mais le contrôle s'exécute après coup : si la déclaration n'est pas conforme, elle est déjà dans `main`. GitHub vous prévient alors par e-mail, et vous la corrigez par un nouveau commit.

**Par pull request** : ajouter le fichier sur une branche et ouvrir une pull request. Le contrôle s'exécute sur la pull request, et on ne la fusionne que s'il passe. C'est la façon sûre de déclarer quand on n'est pas certain du format.

## Retirer ou republier un contenu

On ne supprime jamais un fichier. Pour retirer un contenu, on **modifie** son fichier :

- `status` passe à `retired` ;
- `declared_at` prend la date et l'heure du retrait ;
- `declared_by` prend votre login.

Pour le republier, on refait la même chose avec `status: published`. L'historique Git du fichier garde toutes les déclarations successives (`git log -p registry/2026/ofce-wp-2026-6-fr.json`).

## Ce que vérifie le contrôle

À chaque push et à chaque pull request, le contrôle refuse :

- une déclaration qui ne respecte pas le format ci-dessus (champ manquant, valeur hors liste, adresse en `http://` ou relative, date sans `Z`, champ inconnu, fichier qui n'est pas du JSON) ;
- un fichier mal rangé : hors d'un dossier d'année à quatre chiffres, ou dont l'`id` n'est pas égal au nom du fichier ;
- un même `id` déclaré sous deux années ;
- la suppression d'un fichier de `registry/`, y compris son déplacement vers une autre année.

Il rapporte toutes les erreurs ensemble, avec le fichier, le champ et la valeur reçue.

Il ne vérifie **pas** que le manifeste répond ni qu'il est valide : c'est la collecte qui le signale dans son rapport. Pour contrôler un manifeste avant de le déclarer : `ofceweb::validate_manifest("https://…/manifest.json")`.

Pour lancer le contrôle en local, depuis un clone du dépôt :

```r
ofceweb::check_registry(".")
```
