# Instructions Cursor - Projet d'Analyse de Corrélations Crypto

## 🎯 OBJECTIF DU PROJET

Développer une stratégie de trading basée sur l'analyse scientifique des corrélations entre 6 paires de cryptomonnaies, en utilisant des méthodes d'économétrie financière avancées, avec pour but final la création d'indicateurs rentables et leur implémentation sur TradingView.

---

## 📊 PAIRES ANALYSÉES

- BTCUSDT
- ETHUSDT
- XRPUSDT
- SOLUSDT
- DOGEUSDT
- BNBUSDT

---

## 🗂️ STRUCTURE DU PROJET

```
project_root/
├── data/                          # Fichiers Excel (timeframe 1H, 3 mois, avec CVD)
│   ├── BTCUSDT.xlsx
│   ├── ETHUSDT.xlsx
│   ├── XRPUSDT.xlsx
│   ├── SOLUSDT.xlsx
│   ├── DOGEUSDT.xlsx
│   └── BNBUSDT.xlsx
├── notebooks/                     # Jupyter notebooks (optionnels pour visualisation)
│   └── quick_viz.ipynb           # Visualisation rapide des résultats
├── src/                           # Code source Python
│   ├── data_loader.py            # Chargement et preprocessing des données
│   ├── correlation_analysis.py   # Analyses de corrélations croisées avec tests stat
│   ├── var_models.py             # Modèles VAR avec validation rigoureuse
│   ├── granger_tests.py          # Tests de causalité de Granger
│   ├── strategy_builder.py       # Construction de stratégies basées sur corrélations
│   ├── backtesting.py            # Backtesting avec walk-forward
│   ├── kpi_calculator.py         # Calcul des KPIs avec IC
│   ├── scientific_validation.py  # Tests de robustesse et validation
│   └── utils.py                  # Fonctions utilitaires
├── scripts/                       # Scripts d'exécution automatisés
│   ├── run_phase1.py             # MASTER SCRIPT - Lance toute la Phase 1
│   ├── run_phase2.py             # MASTER SCRIPT - Lance la Phase 2
│   └── config.py                 # Configuration centralisée
├── pine_script/                   # Code Pine Script pour TradingView
│   └── correlation_strategy.pine
├── results/                       # Résultats et visualisations
│   ├── correlation_matrices/
│   ├── lag_analysis/
│   ├── var_results/
│   ├── granger_tests/
│   ├── backtest_results/
│   ├── kpi_reports/
│   ├── scientific_validation/
│   └── final_report.pdf
├── requirements.txt               # Dépendances Python
├── README.md                      # Documentation du projet
└── METHODOLOGY.md                 # Documentation méthodologique scientifique
```

---

## 📚 CONTEXTE MATHÉMATIQUE ET MÉTHODOLOGIES

### 1. Branche Mathématique
**Économétrie financière** - Séries temporelles multivariées

### 2. Méthodes à Implémenter

#### A. Cross-Correlation Lagged (CCF)
- **But** : Identifier les décalages temporels entre paires
- **Implémentation** : Calculer la corrélation de Pearson entre une série et une autre décalée de k périodes (lags)
- **Librairies** : `pandas`, `numpy`, `statsmodels.tsa.stattools.ccf`
- **Output attendu** : 
  - Matrices de corrélation pour différents lags (de -50 à +50 périodes)
  - Graphiques CCF pour chaque paire de cryptos
  - Identification des lags significatifs (corrélation > 0.7 ou < -0.7)

#### B. Modèles VAR (Vector AutoRegressive)
- **But** : Capturer les dynamiques multivariées et interdépendances
- **Implémentation** : `statsmodels.tsa.api.VAR`
- **Étapes** :
  1. Test de stationnarité (ADF test)
  2. Différenciation si nécessaire
  3. Sélection de l'ordre optimal (AIC, BIC)
  4. Estimation du modèle VAR
  5. Analyse des coefficients et IRF (Impulse Response Functions)
- **Output attendu** :
  - Ordre optimal du modèle
  - Coefficients significatifs
  - Prévisions à court terme

#### C. Tests de Causalité de Granger
- **But** : Déterminer si une série aide à prédire une autre
- **Implémentation** : `statsmodels.tsa.stattools.grangercausalitytests`
- **Output attendu** :
  - Matrice de causalité (qui influence qui)
  - P-values pour différents lags
  - Graphe de causalité dirigé

---

## 🔬 PHASE 1 : ANALYSE EN PYTHON

### Objectif
Trouver des patterns de corrélation exploitables commercialement

### Étapes de Développement

#### 1. Data Loading & Preprocessing
```python
# Fichier: src/data_loader.py
# - Charger tous les fichiers Excel du dossier ./data/
# - Colonnes: time, open, high, low, close, CVD(Ouverture), CVD(Haut), CVD(Bas), CVD(Fermer)
# - Vérifier l'intégrité des données (NaN, outliers)
# - Synchroniser les timestamps entre toutes les paires
# 
# TRAITEMENT DES 4 COLONNES CVD:
# Option 1: Utiliser CVD(Fermer) comme référence principale (recommandé)
# Option 2: Calculer la moyenne des 4 CVD pour lisser le signal
# Option 3: Calculer le delta intra-bougie: CVD(Fermer) - CVD(Ouverture)
# Option 4: Utiliser les 4 CVD séparément pour capturer la dynamique intra-bougie
#
# Recommandation initiale: CVD(Fermer) + delta intra-bougie
#
# - Créer un DataFrame multivarié avec colonnes: 
#   [timestamp, BTC_close, BTC_cvd, BTC_cvd_delta, ETH_close, ETH_cvd, ETH_cvd_delta, ...]
# - Calculer les returns logarithmiques pour chaque paire
# - Normaliser les CVD (z-score) pour comparabilité entre paires
```

**Colonnes présentes dans les Excel** :
- time (timestamp)
- open
- high
- low
- close
- CVD (Ouverture) - CVD à l'ouverture de la bougie
- CVD (Haut) - CVD au plus haut de la bougie
- CVD (Bas) - CVD au plus bas de la bougie
- CVD (Fermer) - CVD à la fermeture de la bougie

**Note** : Les 4 colonnes CVD fournissent une granularité intra-bougie du volume delta

#### 2. Analyse Exploratoire
```python
# Fichier: notebooks/01_data_exploration.ipynb
# - Statistiques descriptives de chaque paire
# - Visualisation des prix et volumes
# - Analyse de la distribution des returns
# - Détection des périodes de haute/basse volatilité
# - Corrélation simple (statique) entre toutes les paires
```

#### 3. Cross-Correlation Analysis
```python
# Fichier: src/correlation_analysis.py
# Pour chaque paire (i, j):
#   - Calculer CCF pour lags de -50 à +50 heures
#   - Identifier les lags avec |corrélation| > seuil (0.6, 0.7, 0.8)
#   - Créer des heatmaps de corrélation par lag
#   - Extraire les "leading pairs" (qui précèdent les mouvements)
```

#### 4. VAR Modeling
```python
# Fichier: src/var_models.py
# - Test de stationnarité (Augmented Dickey-Fuller)
# - Différenciation des séries si nécessaire
# - Sélection de l'ordre optimal (max_lags = 24h)
# - Estimation du modèle VAR
# - Analyse des résidus
# - Forecast à horizon 1h, 4h, 12h
```

#### 5. Granger Causality
```python
# Fichier: src/granger_tests.py
# - Matrice de causalité 6x6
# - Tests pour max_lag = 1, 4, 8, 12, 24 heures
# - Identifier les paires "causales" (p-value < 0.05)
# - Créer un graphe de causalité NetworkX
```

#### 6. Strategy Development
```python
# Fichier: src/strategy_builder.py
# Basé sur les résultats précédents:
# - Identifier les signaux de trading:
#   * Si paire A monte avec lag de 2h → paire B devrait suivre
#   * Si causalité forte A→B → utiliser A comme signal pour B
#   * Si corrélation négative forte → stratégie de pairs trading
# - Définir les règles d'entrée/sortie
# - Paramètres: seuils de corrélation, horizons temporels, taille des positions
```

#### 7. Backtesting
```python
# Fichier: src/backtesting.py
# - Implémentation vectorisée du backtesting
# - Walk-forward analysis
# - Gestion des frais de transaction (0.1% par trade)
# - Gestion du risque (stop-loss, take-profit)
# - Calcul des métriques de performance
```

#### 8. KPI Calculation
```python
# Fichier: src/kpi_calculator.py
# KPIs à calculer (OBLIGATOIRES):

# === PERFORMANCE ABSOLUE ===
# - Total Return (%)
# - Annualized Return (%)
# - Profitabilité sur Capital Fictif (ex: 10,000 USDT de départ)
#   * Capital Final
#   * Profit Net en USDT
#   * ROI (%)

# === RATIOS DE PERFORMANCE ===
# - Sharpe Ratio
# - Sortino Ratio
# - Calmar Ratio
# - Return over Maximum Drawdown (RoMaD)

# === GESTION DU RISQUE ===
# - Maximum Drawdown (%)
# - Average Drawdown (%)
# - Drawdown Duration (nombre de périodes)
# - Value at Risk (VaR) 95% et 99%

# === STATISTIQUES DES TRADES ===
# - Win Rate (%) - OBLIGATOIRE
# - Nombre total de trades
# - Nombre de trades gagnants
# - Nombre de trades perdants
# - Profit Factor (gains totaux / pertes totales)
# - Average Win / Average Loss
# - Ratio Win/Loss
# - Largest Win / Largest Loss
# - Average Trade Duration (en heures)
# - Expectancy par trade

# === FRÉQUENCE DE TRADING ===
# - Trades per jour/semaine/mois
# - Temps moyen entre deux trades

# === STABILITÉ ===
# - Correlation des returns de la stratégie avec chaque paire
# - Rolling Sharpe Ratio (fenêtre de 30 jours)
# - Consistency Score (% de mois positifs)
```

### Critères de Succès Phase 1
Une stratégie est considérée **profitable** si TOUS les critères suivants sont atteints :

**Performance Absolue :**
- Rentabilité annualisée > 20%
- Profitabilité sur capital fictif (10,000 USDT) > 2,000 USDT net (20%+)

**Ratios de Risque/Rendement :**
- Sharpe Ratio > 1.5
- Sortino Ratio > 2.0
- Maximum Drawdown < 20%
- Calmar Ratio > 1.0

**Statistiques de Trading :**
- Win Rate > 55% (OBLIGATOIRE)
- Profit Factor > 1.5
- Average Trade > 0 après frais
- Expectancy positive

**Robustesse :**
- Performance cohérente en walk-forward (écart < 30% entre training et testing)
- Pas de période de drawdown > 30 jours
- Au moins 30+ trades sur la période d'analyse

Si UN SEUL critère échoue → **NO-GO pour Phase 2**

---

## 🎨 PHASE 2 : IMPLÉMENTATION PINE SCRIPT

### Déclenchement
Cette phase ne commence QUE si la Phase 1 démontre une stratégie profitable selon les critères ci-dessus.

### Objectif
Créer un indicateur/stratégie TradingView pour backtesting visuel et déploiement potentiel.

### Structure Pine Script
```pine
// Fichier: pine_script/correlation_strategy.pine
//@version=5
strategy("Crypto Correlation Strategy", overlay=true)

// INPUTS
// - Définir les paramètres de corrélation trouvés en Python
// - Lags optimaux
// - Seuils de corrélation
// - Paramètres de gestion du risque

// DATA FETCHING
// - Récupérer les données des 6 paires via request.security()
// - Calculer les returns

// CORRELATION CALCULATION
// - Implémenter les calculs de corrélation lagged
// - Utiliser ta.correlation() avec des séries décalées

// SIGNAL GENERATION
// - Reproduire la logique de la stratégie Python
// - Générer les signaux d'achat/vente

// STRATEGY LOGIC
// - Entry conditions
// - Exit conditions
// - Stop-loss / Take-profit

// VISUALIZATION
// - Afficher les signaux sur le graphique
// - Tables avec les métriques de performance
// - Indicateurs de corrélation en temps réel
```

### Limitations Pine Script à Anticiper
- Pas de calcul matriciel complexe (simplifier les VAR)
- Limitation à ~5000 barres de données
- Pas de multi-timeframe complexe
- Focus sur les signaux CCF et causalité simple

---

## 🛠️ STACK TECHNIQUE

### Python
**Librairies requises** (requirements.txt) :
```
pandas>=2.0.0
numpy>=1.24.0
scipy>=1.10.0
statsmodels>=0.14.0
matplotlib>=3.7.0
seaborn>=0.12.0
plotly>=5.14.0
openpyxl>=3.1.0  # Pour lire Excel
networkx>=3.1    # Pour graphes de causalité
jupyter>=1.0.0
scikit-learn>=1.2.0
```

### Environnement
- Python 3.10+
- Jupyter Notebook/Lab
- Git pour versioning

---

## 📋 BONNES PRATIQUES DE DÉVELOPPEMENT

### Code Quality
1. **Documentation** : Docstrings pour toutes les fonctions
2. **Type Hints** : Utiliser les annotations de type Python
3. **Modularité** : Fonctions réutilisables et testables
4. **Logging** : Utiliser le module `logging` pour tracer les analyses
5. **Config** : Centraliser les paramètres dans un fichier de config

### Gestion de Données
1. Sauvegarder les résultats intermédiaires (pickle, CSV)
2. Versionner les datasets avec hash MD5
3. Créer des checkpoints pour les analyses longues

### Visualisations
1. Graphiques interactifs avec Plotly pour exploration
2. Graphiques statiques avec Matplotlib/Seaborn pour rapports
3. Sauvegarder toutes les figures importantes

### Performance
1. Utiliser des opérations vectorisées (NumPy/Pandas)
2. Éviter les boucles Python quand possible
3. Profiler le code avec `cProfile` si nécessaire

---

## 📊 LIVRABLES ATTENDUS

### Phase 1 (Python)
1. **Notebooks d'analyse** : Exploration complète des données
2. **Code source modulaire** : Bibliothèque réutilisable
3. **Rapport de corrélations** : 
   - Matrices de corrélation par lag
   - Graphes CCF
   - Résultats VAR
   - Matrice de causalité Granger
4. **Documentation de stratégie** : Description détaillée de la logique
5. **Résultats de backtesting** :
   - Equity curve
   - Drawdown chart
   - Distribution des returns
   - KPIs complets
6. **Recommandation GO/NO-GO** pour Phase 2

### Phase 2 (Pine Script)
1. **Code Pine Script** : Indicateur/Stratégie fonctionnel
2. **Backtest TradingView** : Résultats visuels sur historique long
3. **Guide d'utilisation** : Comment paramétrer l'indicateur
4. **Comparaison Python vs Pine** : Validation de la concordance

---

## 🚀 WORKFLOW DE DÉVELOPPEMENT - SPRINT INTENSIF 2 JOURS

### 🔴 JOUR 1 : PHASE 1 - ANALYSE PYTHON (8h de travail)

**Heure 0-1 : Setup & Data Loading**
- [ ] Créer la structure du projet complète
- [ ] Installer toutes les dépendances
- [ ] Développer `data_loader.py` : chargement automatisé des 6 fichiers Excel
- [ ] Synchronisation des timestamps
- [ ] Calcul des returns logarithmiques
- [ ] Validation de l'intégrité des données (NaN check)

**Heure 1-3 : Cross-Correlation Analysis (CCF)**
- [ ] Implémenter `correlation_analysis.py` avec calcul CCF automatisé
- [ ] Calcul matriciel pour lags -50 à +50 heures pour TOUTES les paires (15 combinaisons)
- [ ] Génération automatique des heatmaps de corrélation
- [ ] Extraction automatique des lags significatifs (|corr| > 0.7)
- [ ] Export des résultats en CSV et PNG

**Heure 3-5 : Modèles VAR & Tests de Granger**
- [ ] `var_models.py` : Tests ADF de stationnarité sur toutes les séries
- [ ] Différenciation automatique si nécessaire
- [ ] Estimation VAR avec sélection automatique d'ordre (AIC/BIC)
- [ ] `granger_tests.py` : Matrice complète de causalité 6x6
- [ ] Tests pour lags 1, 4, 8, 12, 24h
- [ ] Génération du graphe de causalité NetworkX
- [ ] Export des p-values et résultats significatifs

**Heure 5-7 : Développement & Backtesting de Stratégie**
- [ ] `strategy_builder.py` : Développement de 3-5 stratégies candidates basées sur :
  - Signaux de lead-lag des CCF
  - Relations de causalité Granger
  - Combinaisons multi-paires
- [ ] `backtesting.py` : Engine de backtesting vectorisé ultra-rapide
- [ ] Test des stratégies sur la période complète
- [ ] Walk-forward validation (70% training, 30% testing)
- [ ] Intégration des frais de transaction (0.1% par trade)

**Heure 7-8 : KPIs, Validation & Décision GO/NO-GO**
- [ ] `kpi_calculator.py` : Calcul automatisé de TOUS les KPIs
- [ ] Génération des graphiques de performance (equity curve, drawdown)
- [ ] Rapport automatique de décision GO/NO-GO
- [ ] Export complet des résultats
- [ ] **DÉCISION FINALE** : Si critères atteints → Phase 2, sinon STOP

---

### 🟢 JOUR 2 : PHASE 2 - PINE SCRIPT (6-8h de travail)

**Uniquement si Phase 1 = GO**

**Heure 0-2 : Traduction Stratégie → Pine Script**
- [ ] Analyser la stratégie retenue
- [ ] Simplifier les calculs pour Pine Script
- [ ] Coder la structure de base de l'indicateur
- [ ] Implémenter les calculs de corrélation lagged

**Heure 2-4 : Logique de Trading & Signaux**
- [ ] Coder les conditions d'entrée/sortie
- [ ] Implémenter stop-loss / take-profit
- [ ] Ajouter la gestion de position
- [ ] Tests unitaires sur données historiques

**Heure 4-6 : Backtesting TradingView & Optimisation**
- [ ] Déployer sur TradingView
- [ ] Backtesting sur historique long (6-12 mois si possible)
- [ ] Ajustement des paramètres visuels
- [ ] Comparaison des résultats avec Python

**Heure 6-8 : Validation & Documentation**
- [ ] Vérification de la concordance Python ↔ Pine
- [ ] Documentation de l'indicateur
- [ ] Guide d'utilisation pour TradingView
- [ ] Rapport final complet

---

## ⏱️ CONTRAINTES TEMPORELLES - AUTOMATISATION MAXIMALE

Pour tenir le délai de 2 jours, le code doit être :

1. **Entièrement automatisé** : Zéro intervention manuelle entre les étapes
2. **Parallélisé** : Utiliser `multiprocessing` pour calculs lourds
3. **Script d'exécution master** : `run_phase1.py` et `run_phase2.py` qui lancent tout
4. **Minimal mais complet** : Pas de notebook exploratoire, code production direct
5. **Résultats auto-sauvegardés** : Chaque étape génère et sauvegarde ses outputs

---

## 🤖 SCRIPTS D'EXÉCUTION AUTOMATISÉS

### Script Master Phase 1 : `scripts/run_phase1.py`

```python
"""
MASTER SCRIPT - PHASE 1
Lance automatiquement toute l'analyse en ~6-8h
"""

import logging
import time
from datetime import datetime
from pathlib import Path
import multiprocessing as mp

# Configuration du logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler(f'phase1_{datetime.now().strftime("%Y%m%d_%H%M%S")}.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

def main():
    """Exécution séquentielle de toutes les étapes de Phase 1"""
    
    start_time = time.time()
    logger.info("="*80)
    logger.info("DÉBUT PHASE 1 - ANALYSE DE CORRÉLATIONS CRYPTO")
    logger.info("="*80)
    
    # ========== ÉTAPE 1: CHARGEMENT DES DONNÉES ==========
    logger.info("\n[1/8] Chargement et préparation des données...")
    from src.data_loader import load_and_prepare_data
    data, metadata = load_and_prepare_data('./data/')
    logger.info(f"✓ Données chargées: {len(data)} observations, {len(data.columns)} colonnes")
    logger.info(f"✓ Période: {data.index[0]} → {data.index[-1]}")
    
    # ========== ÉTAPE 2: CORRÉLATIONS CROISÉES ==========
    logger.info("\n[2/8] Analyse des corrélations croisées (CCF)...")
    from src.correlation_analysis import run_ccf_analysis
    ccf_results = run_ccf_analysis(data, max_lag=50, n_jobs=-1)
    logger.info(f"✓ {ccf_results['n_significant']} corrélations significatives trouvées")
    logger.info(f"✓ Seuil alpha ajusté: {ccf_results['alpha_corrected']:.2e}")
    
    # ========== ÉTAPE 3: TESTS DE STATIONNARITÉ ==========
    logger.info("\n[3/8] Tests de stationnarité (ADF)...")
    from src.var_models import test_all_stationarity
    stationarity_results = test_all_stationarity(data)
    logger.info(f"✓ Séries stationnaires: {stationarity_results['n_stationary']}/{stationarity_results['n_total']}")
    
    # ========== ÉTAPE 4: MODÈLES VAR ==========
    logger.info("\n[4/8] Estimation des modèles VAR...")
    from src.var_models import estimate_var_strict
    var_results = estimate_var_strict(data, max_lags=24)
    logger.info(f"✓ Ordre optimal VAR: {var_results['optimal_lag']}")
    logger.info(f"✓ Modèle valide: {var_results['validation']['model_valid']}")
    
    # ========== ÉTAPE 5: TESTS DE GRANGER ==========
    logger.info("\n[5/8] Tests de causalité de Granger...")
    from src.granger_tests import granger_causality_matrix_strict
    granger_results, alpha_corrected = granger_causality_matrix_strict(data, max_lag=12)
    n_causal = sum([1 for v in granger_results.values() 
                    for r in v.values() if r.get('is_causal', False)])
    logger.info(f"✓ Relations causales significatives: {n_causal}")
    logger.info(f"✓ Alpha corrigé: {alpha_corrected:.2e}")
    
    # ========== ÉTAPE 6: DÉVELOPPEMENT STRATÉGIES ==========
    logger.info("\n[6/8] Développement des stratégies de trading...")
    from src.strategy_builder import build_strategies
    strategies = build_strategies(
        data=data,
        ccf_results=ccf_results,
        granger_results=granger_results,
        var_results=var_results
    )
    logger.info(f"✓ {len(strategies)} stratégies candidates générées")
    
    # ========== ÉTAPE 7: BACKTESTING ==========
    logger.info("\n[7/8] Backtesting des stratégies...")
    from src.backtesting import backtest_strategies
    backtest_results = backtest_strategies(
        strategies=strategies,
        data=data,
        initial_capital=10000,
        transaction_cost=0.001,
        walk_forward=True
    )
    logger.info(f"✓ Stratégies testées avec walk-forward validation")
    
    # ========== ÉTAPE 8: CALCUL DES KPIs ==========
    logger.info("\n[8/8] Calcul des KPIs et validation scientifique...")
    from src.kpi_calculator import calculate_all_kpis
    from src.scientific_validation import validate_strategy
    
    best_strategy = None
    best_sharpe = -float('inf')
    
    for strategy_name, bt_result in backtest_results.items():
        kpis = calculate_all_kpis(
            returns=bt_result['returns'],
            trades=bt_result['trades'],
            initial_capital=10000
        )
        
        validation = validate_strategy(
            strategy=strategy_name,
            backtest_result=bt_result,
            kpis=kpis,
            data=data
        )
        
        logger.info(f"\n--- {strategy_name} ---")
        logger.info(f"Sharpe Ratio: {kpis['sharpe_ratio']:.2f}")
        logger.info(f"Win Rate: {kpis['win_rate']:.1f}%")
        logger.info(f"Max Drawdown: {kpis['max_drawdown']:.1f}%")
        logger.info(f"Total Return: {kpis['total_return']:.1f}%")
        logger.info(f"Capital Final: ${kpis['final_capital']:,.2f}")
        logger.info(f"Validation: {'PASS ✓' if validation['passed'] else 'FAIL ✗'}")
        
        if validation['passed'] and kpis['sharpe_ratio'] > best_sharpe:
            best_strategy = strategy_name
            best_sharpe = kpis['sharpe_ratio']
    
    # ========== DÉCISION FINALE ==========
    logger.info("\n" + "="*80)
    logger.info("DÉCISION FINALE PHASE 1")
    logger.info("="*80)
    
    if best_strategy:
        logger.info(f"✓ STRATÉGIE PROFITABLE TROUVÉE: {best_strategy}")
        logger.info(f"✓ Sharpe Ratio: {best_sharpe:.2f}")
        logger.info("\n>>> GO POUR PHASE 2 - PINE SCRIPT <<<")
        decision = "GO"
    else:
        logger.info("✗ AUCUNE STRATÉGIE NE SATISFAIT LES CRITÈRES")
        logger.info("\n>>> NO-GO - ARRÊT DU PROJET <<<")
        decision = "NO-GO"
    
    # Sauvegarde des résultats
    logger.info("\nSauvegarde des résultats...")
    from src.utils import save_results
    save_results({
        'data_metadata': metadata,
        'ccf_results': ccf_results,
        'stationarity_results': stationarity_results,
        'var_results': var_results,
        'granger_results': granger_results,
        'backtest_results': backtest_results,
        'decision': decision,
        'best_strategy': best_strategy
    }, output_dir='./results/')
    
    elapsed = time.time() - start_time
    logger.info(f"\n✓ Phase 1 terminée en {elapsed/3600:.2f}h")
    logger.info("="*80)
    
    return decision, best_strategy

if __name__ == "__main__":
    decision, best_strategy = main()
```

### Script Master Phase 2 : `scripts/run_phase2.py`

```python
"""
MASTER SCRIPT - PHASE 2
Génération du code Pine Script et validation TradingView
Ne s'exécute QUE si Phase 1 = GO
"""

import logging
import json
from pathlib import Path

logger = logging.getLogger(__name__)

def main():
    """Exécution Phase 2 - Pine Script"""
    
    # Vérifier la décision de Phase 1
    results_file = Path('./results/phase1_results.json')
    if not results_file.exists():
        logger.error("Phase 1 non complétée. Exécutez run_phase1.py d'abord.")
        return
    
    with open(results_file, 'r') as f:
        phase1_results = json.load(f)
    
    if phase1_results['decision'] != 'GO':
        logger.error("Phase 1 = NO-GO. Phase 2 annulée.")
        return
    
    logger.info("="*80)
    logger.info("DÉBUT PHASE 2 - PINE SCRIPT")
    logger.info("="*80)
    
    best_strategy = phase1_results['best_strategy']
    logger.info(f"Stratégie à implémenter: {best_strategy}")
    
    # ========== ÉTAPE 1: GÉNÉRATION PINE SCRIPT ==========
    logger.info("\n[1/4] Génération du code Pine Script...")
    from src.pine_generator import generate_pine_script
    pine_code = generate_pine_script(
        strategy_name=best_strategy,
        phase1_results=phase1_results
    )
    
    pine_file = Path('./pine_script/correlation_strategy.pine')
    pine_file.write_text(pine_code)
    logger.info(f"✓ Code Pine Script généré: {pine_file}")
    
    # ========== ÉTAPE 2: INSTRUCTIONS TRADINGVIEW ==========
    logger.info("\n[2/4] Génération des instructions TradingView...")
    instructions = f"""
    INSTRUCTIONS POUR TRADINGVIEW:
    
    1. Ouvrir TradingView
    2. Créer un nouvel indicateur Pine Script
    3. Copier-coller le contenu de: {pine_file}
    4. Sauvegarder l'indicateur
    5. Appliquer sur le graphique BTCUSDT 1H
    6. Lancer le backtesting Strategy Tester
    7. Vérifier les résultats sur 6-12 mois
    
    Paramètres recommandés:
    {json.dumps(phase1_results['best_strategy_params'], indent=2)}
    
    KPIs attendus (Python):
    - Sharpe Ratio: {phase1_results['best_kpis']['sharpe_ratio']:.2f}
    - Win Rate: {phase1_results['best_kpis']['win_rate']:.1f}%
    - Max DD: {phase1_results['best_kpis']['max_drawdown']:.1f}%
    """
    
    Path('./pine_script/INSTRUCTIONS.txt').write_text(instructions)
    logger.info("✓ Instructions sauvegardées")
    
    # ========== ÉTAPE 3: COMPARAISON ==========
    logger.info("\n[3/4] Préparer la comparaison Python vs Pine...")
    logger.info("→ Copier les résultats TradingView manuellement")
    logger.info("→ Utiliser src/pine_validator.py pour comparer")
    
    # ========== ÉTAPE 4: DOCUMENTATION FINALE ==========
    logger.info("\n[4/4] Génération de la documentation finale...")
    from src.utils import generate_final_report
    generate_final_report(phase1_results, pine_file)
    logger.info("✓ Rapport final généré: ./results/final_report.pdf")
    
    logger.info("\n" + "="*80)
    logger.info("PHASE 2 TERMINÉE")
    logger.info("="*80)
    logger.info("\nPROCHAINES ÉTAPES:")
    logger.info("1. Tester le code Pine sur TradingView")
    logger.info("2. Comparer les résultats avec Python")
    logger.info("3. Ajuster si nécessaire")
    logger.info("4. Déployer si validé")

if __name__ == "__main__":
    main()
```

---

## 📦 CONFIGURATION CENTRALISÉE

### Fichier `scripts/config.py`

```python
"""
Configuration centralisée du projet
"""

from pathlib import Path

# Chemins
PROJECT_ROOT = Path(__file__).parent.parent
DATA_DIR = PROJECT_ROOT / 'data'
RESULTS_DIR = PROJECT_ROOT / 'results'
SRC_DIR = PROJECT_ROOT / 'src'

# Paires de cryptos
PAIRS = ['BTCUSDT', 'ETHUSDT', 'XRPUSDT', 'SOLUSDT', 'DOGEUSDT', 'BNBUSDT']

# Paramètres d'analyse
MAX_LAG_CCF = 50  # Heures
MAX_LAG_VAR = 24  # Heures
MAX_LAG_GRANGER = 12  # Heures

# Paramètres statistiques
ALPHA = 0.05  # Niveau de significativité
CONFIDENCE_LEVEL = 0.95

# Paramètres de backtesting
INITIAL_CAPITAL = 10000  # USDT
TRANSACTION_COST = 0.001  # 0.1%
SLIPPAGE = 0.0005  # 0.05%

# Critères de succès (Phase 1)
CRITERIA = {
    'min_sharpe_ratio': 1.5,
    'min_sortino_ratio': 2.0,
    'max_drawdown': 0.20,  # 20%
    'min_win_rate': 0.55,  # 55%
    'min_profit_factor': 1.5,
    'min_annual_return': 0.20,  # 20%
    'min_calmar_ratio': 1.0,
    'min_trades': 30
}

# Paramètres de validation
TRAIN_SIZE = 0.6
VAL_SIZE = 0.2
TEST_SIZE = 0.2

WALK_FORWARD_STEP = 24  # Heures
MIN_TEST_SIZE = 168  # 1 semaine

# Parallélisation
N_JOBS = -1  # Utiliser tous les CPU

# Random seed pour reproductibilité
RANDOM_SEED = 42
```

---

## 🔬 RIGUEUR MÉTHODOLOGIQUE SCIENTIFIQUE - PROTOCOLE STRICT

### ⚠️ RÈGLES NON-NÉGOCIABLES

Cette section définit les **standards scientifiques stricts** à respecter pour garantir la validité statistique des analyses. **AUCUN raccourci n'est acceptable.**

---

### 1. Tests de Significativité Statistique

#### A. Corrélations (CCF)
```python
# Pour CHAQUE corrélation calculée:
# 1. Calculer la p-value associée
# 2. Appliquer la correction de Bonferroni pour tests multiples:
#    α_adjusted = α / n_tests
#    Exemple: α = 0.05, avec 15 paires × 101 lags = 1515 tests
#    → α_adjusted = 0.05 / 1515 = 0.000033
# 3. NE RETENIR que les corrélations avec p-value < α_adjusted
# 4. Calculer les intervalles de confiance à 95%

from scipy.stats import pearsonr

def calculate_ccf_with_significance(series1, series2, max_lag=50):
    """
    Calcule CCF avec tests de significativité rigoureux
    """
    results = []
    for lag in range(-max_lag, max_lag + 1):
        if lag < 0:
            s1 = series1[:lag]
            s2 = series2[-lag:]
        elif lag > 0:
            s1 = series1[lag:]
            s2 = series2[:-lag]
        else:
            s1 = series1
            s2 = series2
        
        corr, p_value = pearsonr(s1, s2)
        results.append({
            'lag': lag,
            'correlation': corr,
            'p_value': p_value,
            'n_obs': len(s1)
        })
    
    # Correction de Bonferroni
    n_tests = len(results) * 15  # 15 paires de cryptos
    alpha_corrected = 0.05 / n_tests
    
    # Filtrer les résultats significatifs
    significant = [r for r in results if r['p_value'] < alpha_corrected]
    
    return results, significant, alpha_corrected
```

**Obligation** : Documenter dans les résultats :
- Nombre de tests effectués
- Seuil de significativité ajusté
- Nombre de corrélations significatives vs totales
- Taux de faux positifs attendu

---

#### B. Tests de Stationnarité (Augmented Dickey-Fuller)
```python
from statsmodels.tsa.stattools import adfuller

def test_stationarity_strict(series, alpha=0.05):
    """
    Test ADF strict avec interprétation rigoureuse
    """
    result = adfuller(series, autolag='AIC')
    
    is_stationary = result[1] < alpha  # p-value < 0.05
    
    output = {
        'adf_statistic': result[0],
        'p_value': result[1],
        'critical_values': result[4],
        'is_stationary': is_stationary,
        'lags_used': result[2],
        'n_obs': result[3]
    }
    
    # OBLIGATION: Si non-stationnaire, différencier
    if not is_stationary:
        series_diff = series.diff().dropna()
        # Re-tester la série différenciée
        result_diff = adfuller(series_diff, autolag='AIC')
        output['diff_needed'] = True
        output['diff_p_value'] = result_diff[1]
        output['diff_is_stationary'] = result_diff[1] < alpha
    
    return output

# RÈGLE: Toutes les séries doivent être stationnaires avant VAR
# Si 2 différenciations ne suffisent pas → ABANDON de cette série
```

**Obligation** : 
- Tester TOUTES les séries (prix ET CVD)
- Rapporter les p-values
- Ne JAMAIS utiliser de séries non-stationnaires dans VAR

---

#### C. Modèles VAR - Sélection Rigoureuse

```python
from statsmodels.tsa.api import VAR

def estimate_var_strict(data, max_lags=24):
    """
    Estimation VAR avec validation rigoureuse
    """
    model = VAR(data)
    
    # 1. Sélection de l'ordre par critères d'information
    lag_order_results = model.select_order(maxlags=max_lags)
    
    # Utiliser le consensus des critères (AIC, BIC, HQIC)
    selected_lags = {
        'aic': lag_order_results.aic,
        'bic': lag_order_results.bic,
        'hqic': lag_order_results.hqic
    }
    
    # RÈGLE: Privilégier BIC (plus conservateur)
    optimal_lag = selected_lags['bic']
    
    # 2. Estimation du modèle
    fitted_model = model.fit(optimal_lag)
    
    # 3. VALIDATION DES RÉSIDUS (OBLIGATOIRE)
    residuals = fitted_model.resid
    
    # Test de normalité des résidus (Jarque-Bera)
    from scipy.stats import jarque_bera
    normality_tests = {}
    for col in residuals.columns:
        jb_stat, jb_pval = jarque_bera(residuals[col])
        normality_tests[col] = {
            'statistic': jb_stat,
            'p_value': jb_pval,
            'is_normal': jb_pval > 0.05
        }
    
    # Test d'autocorrélation des résidus (Ljung-Box)
    from statsmodels.stats.diagnostic import acorr_ljungbox
    autocorr_tests = {}
    for col in residuals.columns:
        lb_result = acorr_ljungbox(residuals[col], lags=10, return_df=True)
        autocorr_tests[col] = {
            'p_values': lb_result['lb_pvalue'].tolist(),
            'no_autocorr': all(lb_result['lb_pvalue'] > 0.05)
        }
    
    # Test d'hétéroscédasticité (White test)
    # ... (implementation détaillée)
    
    validation = {
        'lag_selection': selected_lags,
        'optimal_lag': optimal_lag,
        'normality': normality_tests,
        'autocorrelation': autocorr_tests,
        'model_valid': all([
            # Tous les résidus doivent être non-autocorrélés
            all([v['no_autocorr'] for v in autocorr_tests.values()])
        ])
    }
    
    return fitted_model, validation

# RÈGLE CRITIQUE: Si validation échoue → le modèle VAR n'est PAS fiable
# Ne PAS utiliser les résultats d'un modèle VAR invalide
```

**Obligations** :
- Toujours valider les résidus
- Rapporter les résultats des tests
- Ne jamais utiliser un modèle dont les résidus sont autocorrélés
- Documenter les limites du modèle

---

#### D. Tests de Causalité de Granger - Rigueur Maximale

```python
from statsmodels.tsa.stattools import grangercausalitytests

def granger_causality_matrix_strict(data, max_lag=12, alpha=0.05):
    """
    Matrice de causalité avec correction pour tests multiples
    """
    variables = data.columns
    n_vars = len(variables)
    n_tests = n_vars * (n_vars - 1)  # Chaque paire dans les 2 sens
    
    # Correction de Bonferroni
    alpha_corrected = alpha / n_tests
    
    causality_matrix = {}
    
    for caused in variables:
        causality_matrix[caused] = {}
        for causing in variables:
            if caused == causing:
                continue
            
            # Test de Granger : causing → caused
            test_data = data[[caused, causing]]
            
            try:
                test_result = grangercausalitytests(
                    test_data, 
                    maxlag=max_lag, 
                    verbose=False
                )
                
                # Extraire les p-values pour chaque lag
                p_values = {}
                for lag in range(1, max_lag + 1):
                    # Utiliser le test F
                    p_values[lag] = test_result[lag][0]['ssr_ftest'][1]
                
                # Déterminer le lag optimal (plus petite p-value)
                best_lag = min(p_values, key=p_values.get)
                best_p_value = p_values[best_lag]
                
                causality_matrix[caused][causing] = {
                    'p_values': p_values,
                    'best_lag': best_lag,
                    'best_p_value': best_p_value,
                    'is_causal': best_p_value < alpha_corrected,
                    'alpha_corrected': alpha_corrected
                }
                
            except Exception as e:
                causality_matrix[caused][causing] = {
                    'error': str(e),
                    'is_causal': False
                }
    
    return causality_matrix, alpha_corrected

# RÈGLE: NE RETENIR que les causalités avec p-value < α_corrected
# Documenter le nombre de tests et le seuil ajusté
```

**Obligations** :
- Correction pour tests multiples (Bonferroni ou FDR)
- Tester plusieurs lags et rapporter le lag optimal
- Ne jamais affirmer une causalité si p-value > α_corrected
- Distinguer "causalité statistique" et "causalité économique"

---

### 2. Validation Croisée et Robustesse

#### A. Walk-Forward Analysis (OBLIGATOIRE)
```python
def walk_forward_validation(data, strategy_func, 
                            train_size=0.7, 
                            step_size=24,  # 1 jour
                            min_test_size=168):  # 1 semaine
    """
    Walk-forward analysis pour éviter l'overfitting
    """
    results = []
    n = len(data)
    train_end = int(n * train_size)
    
    while train_end + min_test_size <= n:
        # Période d'entraînement
        train_data = data[:train_end]
        
        # Période de test
        test_start = train_end
        test_end = min(test_start + min_test_size, n)
        test_data = data[test_start:test_end]
        
        # Entraîner la stratégie
        strategy = strategy_func(train_data)
        
        # Tester sur données out-of-sample
        test_performance = strategy.backtest(test_data)
        
        results.append({
            'train_start': 0,
            'train_end': train_end,
            'test_start': test_start,
            'test_end': test_end,
            'performance': test_performance
        })
        
        # Avancer la fenêtre
        train_end += step_size
    
    return results

# RÈGLE: La stratégie doit performer de manière STABLE sur tous les walks
# Si écart-type de performance > 50% → stratégie INSTABLE → REJET
```

#### B. Bootstrapping pour Intervalles de Confiance
```python
from scipy.stats import bootstrap

def bootstrap_confidence_intervals(returns, n_bootstrap=10000):
    """
    Calcul des IC à 95% pour les métriques de performance
    """
    rng = np.random.RandomState(42)
    
    def sharpe(data):
        return np.mean(data) / np.std(data) * np.sqrt(252 * 24)  # Annualisé
    
    result = bootstrap(
        (returns,), 
        sharpe, 
        n_resamples=n_bootstrap,
        random_state=rng
    )
    
    ci_low = result.confidence_interval.low
    ci_high = result.confidence_interval.high
    
    return ci_low, ci_high

# RÈGLE: Toujours rapporter les IC, pas seulement les moyennes
```

---

### 3. Gestion du Data Snooping et Overfitting

#### A. Hold-Out Set STRICT
```python
# RÈGLE ABSOLUE:
# 1. Diviser les données en 3:
#    - Training: 60% (pour développer la stratégie)
#    - Validation: 20% (pour ajuster les paramètres)
#    - Test: 20% (NE JAMAIS TOUCHER jusqu'à la fin)
#
# 2. Le test set ne doit JAMAIS influencer les décisions
# 3. Les résultats finaux sont ceux du test set UNIQUEMENT

def create_strict_data_splits(data):
    n = len(data)
    train_end = int(n * 0.6)
    val_end = int(n * 0.8)
    
    return {
        'train': data[:train_end],
        'validation': data[train_end:val_end],
        'test': data[val_end:]  # SACRED - DO NOT TOUCH
    }
```

#### B. Limite sur l'Optimisation de Paramètres
```python
# RÈGLE: Maximum 5 paramètres à optimiser
# Plus de paramètres = plus de risque d'overfitting

# RÈGLE: Grid search limité
# Pas plus de 10 valeurs par paramètre
# Total de combinaisons: < 10,000

# RÈGLE: Penalty pour complexité
# Utiliser AIC/BIC pour pénaliser les modèles complexes
```

---

### 4. Tests de Robustesse POST-Développement

Avant de valider la stratégie finale, effectuer:

#### A. Sensitivity Analysis
```python
# Tester la stratégie avec:
# - Paramètres +/- 10%
# - Paramètres +/- 20%
# 
# Si performance s'effondre avec petit changement → stratégie FRAGILE → REJET
```

#### B. Monte Carlo Simulation
```python
# 1000 simulations avec:
# - Ordre des trades aléatoire
# - Dates de départ aléatoires
# - Bootstrapping des returns
#
# Calculer la distribution de la performance
# Si P(Sharpe < 1.0) > 25% → stratégie PAS ROBUSTE
```

#### C. Régime de Marché
```python
# Tester séparément sur:
# - Périodes haussières (Bull)
# - Périodes baissières (Bear)
# - Périodes latérales (Sideways)
#
# La stratégie doit être profitable dans AU MOINS 2 régimes sur 3
```

---

### 5. Documentation et Reproductibilité

#### Obligations Finales:

1. **Code Versionné** : Git avec commits atomiques
2. **Seed Fixe** : `np.random.seed(42)` partout
3. **Requirements.txt** : Versions exactes des librairies
4. **Rapport Méthodologique** : Document séparé détaillant:
   - Chaque test statistique effectué
   - Chaque hypothèse faite
   - Chaque limitation identifiée
   - Chaque décision méthodologique justifiée
5. **Résultats Bruts** : Sauvegarder TOUS les résultats intermédiaires
6. **Checklist de Validation** : Cocher chaque étape méthodologique

---

## 🚨 CHECKLIST DE VALIDATION SCIENTIFIQUE

Avant de passer en Phase 2, VÉRIFIER:

- [ ] Toutes les corrélations ont été testées pour significativité
- [ ] Correction de Bonferroni appliquée
- [ ] Tests de stationnarité effectués sur toutes les séries
- [ ] Résidus du VAR validés (normalité, non-autocorrélation)
- [ ] Tests de Granger avec correction pour tests multiples
- [ ] Walk-forward validation effectuée (min 3 walks)
- [ ] Hold-out test set jamais touché pendant le développement
- [ ] Intervalles de confiance calculés pour tous les KPIs
- [ ] Tests de robustesse (sensitivity, Monte Carlo) passés
- [ ] Performance stable sur différents régimes de marché
- [ ] Ratio signal/bruit > 2 (performance / écart-type)
- [ ] Pas de data snooping détecté
- [ ] Code reproductible à 100% (seed fixe)
- [ ] Documentation méthodologique complète
- [ ] Peer review du code et de la méthode (si possible)

**Si UN SEUL item échoue → STOP et correction avant Phase 2**

---

### Pièges Statistiques
1. **Spurious Correlations** : Vérifier que les corrélations ne sont pas dues au hasard
2. **Overfitting** : Ne pas sur-optimiser sur les données historiques
3. **Non-stationnarité** : Différencier les séries si nécessaire
4. **Look-ahead bias** : Ne jamais utiliser de données futures dans les calculs

### Réalisme du Trading
1. **Transaction Costs** : Toujours inclure les frais (0.1% par trade minimum)
2. **Slippage** : Considérer 0.05-0.1% de slippage
3. **Market Impact** : Pour gros volumes, impact sur les prix
4. **Liquidité** : Vérifier que les volumes permettent l'exécution

### Validation Croisée
1. **Out-of-sample testing** : Garder 20-30% des données pour validation finale
2. **Walk-forward analysis** : Tester sur des périodes roulantes
3. **Différents régimes de marché** : Bull, bear, sideways

---

## 🎯 OBJECTIFS SMART DU PROJET

- **Spécifique** : Développer une stratégie de trading basée sur les corrélations entre 6 paires crypto avec rigueur scientifique stricte
- **Mesurable** : Atteindre Sharpe > 1.5, Win Rate > 55%, DD < 20%, ROI > 20% sur capital fictif de 10,000 USDT
- **Atteignable** : Utiliser des méthodes éprouvées d'économétrie avec validation statistique rigoureuse
- **Réaliste** : Focus sur timeframe 1H et 3 mois de données (environ 2,160 observations)
- **Temporel** : 
  - **Jour 1 (8h)** : Phase 1 complète - Analyse Python et décision GO/NO-GO
  - **Jour 2 (6-8h)** : Phase 2 si GO - Pine Script et validation TradingView

**Contrainte critique** : Automatisation totale via scripts master pour respecter le timing serré

---

## 📞 SUPPORT & RESSOURCES

### Documentation Clés
- Statsmodels : https://www.statsmodels.org/stable/index.html
- Pine Script : https://www.tradingview.com/pine-script-docs/
- Pandas Time Series : https://pandas.pydata.org/docs/user_guide/timeseries.html

### Références Académiques
- "Analysis of Financial Time Series" - Ruey S. Tsay
- "Time Series Analysis" - James D. Hamilton
- Articles sur cross-correlation dans le trading crypto

---

## ✅ CHECKLIST DE VALIDATION FINALE

Avant de considérer le projet terminé :

- [ ] Toutes les analyses sont documentées
- [ ] Le code est commenté et testé
- [ ] Les résultats sont reproductibles
- [ ] Les KPIs sont calculés correctement
- [ ] Le backtesting inclut les coûts réalistes
- [ ] La stratégie est validée out-of-sample
- [ ] Le rapport final est rédigé
- [ ] (Si Phase 2) Le Pine Script fonctionne sur TradingView
- [ ] (Si Phase 2) Les résultats Python et Pine concordent

---

**Version** : 1.0  
**Dernière mise à jour** : 31 octobre 2025  
**Status** : Prêt pour développement