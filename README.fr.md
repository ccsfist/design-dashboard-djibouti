# Tableau de Bord de Conception

Ce dépôt contient le **Tableau de Bord de Conception**, un outil de prévision climatique et d'évaluation des risques conçu pour soutenir les décisions d'aide humanitaire et la planification agricole.

## Aperçu

Le Tableau de Bord de Conception fournit :
- **Prévisions climatiques** avec analyse de probabilité de dépassement
- **Évaluation des risques** pour la prise de décision d'aide humanitaire
- **Validation historique** de la performance des prévisions
- **Support multi-pays** pour les régions d'Afrique et au-delà
- **Visualisation interactive des données** avec cartes, graphiques et tableaux
- **Support des fichiers de formes locales** pour les limites géographiques sans connectivité de base de données

## Guide de Démarrage Rapide

Ce guide vous accompagnera dans la configuration du Tableau de Bord de Conception depuis le début, incluant la préparation des données, la configuration de l'environnement et la configuration.

### Prérequis

- **Python 3.9+** et **Conda** installés
- **Git** pour cloner le dépôt
- **Connaissances de base** en Python et opérations en ligne de commande

### Étape 1 : Cloner et Configurer le Dépôt

```bash
# Cloner le dépôt
git clone https://github.com/nitinmagima/python-maproom-djibouti.git
cd python-maproom-djibouti

# Naviguer vers le répertoire fbfmaproom
cd fbfmaproom
```

### Étape 2 : Installer l'Environnement Conda

**Choisissez le fichier de verrouillage approprié pour votre système d'exploitation :**

- **Linux (64-bit) :** `conda-linux-64-design-dashboard.lock`
- **macOS (Intel/64-bit) :** `conda-osx-64-design-dashboard.lock`
- **macOS (Apple Silicon/ARM64) :** `conda-osx-arm64-design-dashboard.lock`
- **Windows (64-bit) :** `conda-win-64-design-dashboard.lock`

```bash
# Créer et activer l'environnement conda en utilisant le fichier de verrouillage approprié

# Pour Linux :
conda create -n designdashboard --file conda-linux-64-design-dashboard.lock

# Pour macOS Intel :
conda create -n designdashboard --file conda-osx-64-design-dashboard.lock

# Pour macOS Apple Silicon :
conda create -n designdashboard --file conda-osx-arm64-design-dashboard.lock

# Pour Windows :
conda create -n designdashboard --file conda-win-64-design-dashboard.lock

# Activer l'environnement
conda activate designdashboard
```

### Étape 3 : Préparation des Données

L'application nécessite trois types de fichiers de données :

**Note :** Si vous souhaitez apprendre à générer vous-même les données de prévision, consultez le [Guide Utilisateur des Prévisions Saisonnières PyCPT 2.5](https://iri-pycpt.github.io/PyCPT2-Seasonal-Forecast-User-Guide/intro.html). Toutes les prévisions pour Djibouti se trouvent dans le dossier du sous-module `python_maproom_djibouti`.

#### 3.1 Données de Prévision (Format Zarr)

**Source :** Fichiers de prévision NetCDF dans `data/original-data/djibouti/prcp-jas-v4/`

**Processus :** Convertir les fichiers NetCDF au format Zarr pour un accès efficace

```bash
# Naviguer vers les scripts de conversion de données
cd data-conversion-scripts

# Convertir les données de prévision au format Zarr
# Format : python zarrify-forecast.py [pays]/[nom-dataset]
python zarrify-forecast.py djibouti/prcp-jas-v4

# Ce script va :
# - Lire les fichiers NetCDF depuis data/original-data/djibouti/prcp-jas-v4/
# - Les convertir au format Zarr
# - Sauvegarder dans data/djibouti/prcp-jas-v4.zarr/
```

**Sortie Attendue :** Répertoire `data/djibouti/prcp-jas-v4.zarr/` avec les données de prévision au format Zarr

#### 3.2 Données des Mauvaises Années (Format Zarr)

**Source :** Données de classification des mauvaises années (fichiers CSV)

**Processus :** Convertir les fichiers CSV au format Zarr pour la cohérence

```bash
# Convertir les données des mauvaises années au format Zarr
# Format : python zarrify-bad-years.py [pays]/[nom-fichier] [mois_centre_cible]
python zarrify-bad-years.py djibouti/bad-years-v2-jas 7.5
```

**Notes Importantes :**
- **N'ajoutez pas `.csv`** au nom du fichier lors de l'exécution du script
- **Le mois centre cible** (7.5) représente le centre de la saison cible (trouvé dans la configuration YAML)
- **Emplacement du fichier :** Le script s'attend à ce que les fichiers CSV soient dans le répertoire du pays approprié

**Sortie Attendue :** Répertoire `data/djibouti/bad-years-v2-jas.zarr/` avec les données des mauvaises années au format Zarr

**Directives de Configuration des Datasets :**

L'application prend en charge deux types de données de mauvaises années avec différentes interprétations :

1. **Classification Catégorielle (format : bad) :**
   ```yaml
   bad-years:
     label: Classification des Mauvaises Années
     format: bad  # Classification catégorielle oui/non
     # Aucune déclaration d'unités nécessaire
   ```

2. **Classement Numérique (format : number0 + units : rank) :**
   ```yaml
   bad-years-rank-v2:
     label: Classement des Mauvaises Années
     format: number0  # Entier avec zéro décimal
     units: rank      # Classement ordinal (1er, 2ème, 3ème, etc.)
     lower_is_worse: yes  # Nombre plus élevé = rang plus bas (pire)
   ```

**Différences Clés :**
- **format : bad** = Classification catégorielle (Mauvaise Année vs Pas Mauvaise Année)
- **format : number0 + units : rank** = Classement numérique (1 = pire, 2 = deuxième pire, etc.)
- **lower_is_worse : yes** = Utilisé avec les rangs où des nombres plus élevés indiquent des conditions pires

#### 3.3 Limites Géographiques (Fichiers de Formes)

**Source :** Fichiers de formes GADM pour les limites administratives de Djibouti

**Emplacement :** `data/djibouti/dji_adm_gadm_2022_shp/`

**Fichiers Inclus :**
- `dji_admbnda_gadm_adm0_2022.shp` - Limites nationales
- `dji_admbnda_gadm_adm1_2022.shp` - Limites régionales  
- `dji_admbnda_gadm_adm2_2022.shp` - Limites de districts

**Note :** Ces fichiers de formes sont déjà inclus dans le dépôt et ne nécessitent pas de conversion.

### Étape 4 : Vérifier la Structure des Données

Assurez-vous que la structure de votre répertoire de données ressemble à ceci :

```
fbfmaproom/
├── data/
│   └── djibouti/
│       ├── prcp-jas-v4.zarr/          # Données de prévision
│       ├── bad-years-v2-jas.zarr/     # Données des mauvaises années
│       └── dji_adm_gadm_2022_shp/     # Fichiers de formes
│           ├── dji_admbnda_gadm_adm0_2022.shp
│           ├── dji_admbnda_gadm_adm1_2022.shp
│           └── dji_admbnda_gadm_adm2_2022.shp
```

### Étape 5 : Configuration

#### 5.1 Mettre à Jour le Fichier de Configuration

Éditez `fbfmaproom-sample.yaml` pour correspondre à vos chemins de données :

```yaml
# Sections de configuration clés à vérifier :

data_root: ./data  # Assurez-vous que cela pointe vers votre répertoire de données

shapes:
  - name: National
    local_file: dji_admbnda_gadm_adm0_2022.shp
    key_field: ADM0_PCODE
    label_field: ADM0_EN
    # ... SQL de secours de base de données ...

  - name: Regional  
    local_file: dji_admbnda_gadm_adm1_2022.shp
    key_field: ADM1_PCODE
    label_field: ADM1_EN
    # ... SQL de secours de base de données ...

  - name: District
    local_file: dji_admbnda_gadm_adm2_2022.shp
    key_field: ADM2_PCODE
    label_field: ADM2_EN
    # ... SQL de secours de base de données ...

datasets:
  forecasts:
    pnep-v4:
      path: djibouti/prcp-jas-v4.zarr  # Vérifiez ce chemin
      # ... autre configuration ...

  observations:
    bad-years-rank-v2:
      path: djibouti/bad-years-v2-jas.zarr  # Vérifiez ce chemin
      # ... autre configuration ...
```

### Étape 6 : Exécuter l'Application

#### 6.1 Démarrer l'Application

```bash
# Exécuter l'application avec la configuration et le chemin Python
PYTHONPATH=$PYTHONPATH:$(pwd) CONFIG=fbfmaproom-sample.yaml python fbfmaproom.py
```

#### 6.2 Accéder à l'Application

L'application démarrera sur un port par défaut (généralement 8050). Vérifiez la sortie du terminal pour l'URL exacte.

Ouvrez votre navigateur web et naviguez vers :
- **Application Principale :** `http://localhost:8050/fbfmaproom/djibouti` (ou le port affiché dans le terminal)
- **API des Régions :** `http://localhost:8050/fbfmaproom/regions?country=djibouti&level=0` (ou le port affiché dans le terminal)

**Note :** Si le port 8050 est déjà utilisé, l'application utilisera automatiquement le port suivant disponible. Vérifiez la sortie du terminal pour l'URL réelle.

### Étape 7 : Vérifier l'Installation

- Naviguez vers `http://localhost:8050/fbfmaproom/djibouti`
- Vérifiez que la carte se charge sans erreurs
- Testez en cliquant sur différentes régions
- Vérifiez que les données de prévision s'affichent correctement

### Support

Contactez Nitin Magima (nm2996@columbia.edu) ou l'Équipe du Secteur des Instruments Financiers, NCDP (fist@iri.columbia.edu) pour le support. 