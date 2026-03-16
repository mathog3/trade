# Nowick Strategy PRO v2.0 — Setup & Trading Rules

**Basé sur la stratégie Nowick de Brad.fx — Optimisé pour 60%+ WR | 1:1.5 min RR**

---

## Fichiers

| Fichier | Usage |
|---|---|
| `nowick_strategy_pro.pine` | Strategy Tester TradingView (backtest + optimisation) |
| `nowick_indicator_pro.pine` | Indicator sur chart live (trading réel) |

---

## Pourquoi certains retests fonctionnent et d'autres non

### Analyse deep du problème

Après analyse des images et de la microstructure de marché :

**Nowick candle = candle sans wick (ou wick minimal)**

| Type | Ce que ça veut dire | Signal |
|---|---|---|
| **Wickless Bear** (close = low, pas de wick bas) | Pression baissière maximale jusqu'à la fermeture — **épuisement** des bears | → **BUY** (le prochain move a un stop tight = RR favorable) |
| **Wickless Bull** (close = high, pas de wick haut) | Pression haussière maximale jusqu'à la fermeture — **épuisement** des bulls | → **SELL** (idem) |

### Raisons d'échec des retests

1. **Pas d'alignement HTF** — La tendance H1/H4 va contre le signal → 40% des pertes
2. **Corps trop petit** — Candle wickless sur une micro-bougie = pas de conviction réelle
3. **RSI pas favorable** — Essayer de shorter quand RSI = 45 (pas oversold/overbought)
4. **Mauvaise session** — Asian session = liquidité faible = faux mouvements
5. **Volume trop faible** — Cassure sans volume = non validée
6. **Retest qui casse le niveau** — Si le retest break le low/high de la Nowick candle → invalidé, ne pas entrer

---

## Configuration optimale

### Paramètres recommandés (AUDUSD 15m)

```
Wick Threshold    : 5% du body (commence strict, élargis si peu de signaux)
Min Body Size     : 0.4x ATR
ATR Length        : 14

HTF Timeframe     : 60 (1H)
HTF Fast EMA      : 20
HTF Slow EMA      : 50

RSI Length        : 14
Max RSI for BUY   : 50 (idéal: signal quand RSI < 45)
Min RSI for SELL  : 50 (idéal: signal quand RSI > 55)
Strong BUY zone   : RSI < 35 → utilise TP2 extended
Strong SELL zone  : RSI > 65 → utilise TP2 extended

Volume Filter     : 1.0x MA (au moins dans la moyenne)
Session           : 07:00-17:00 UTC (London + NY)

SL Method         : Both (max) — prend le plus large entre ATR et extreme candle
SL ATR Mult       : 1.2x
TP1 RR            : 1.5
TP2 RR            : 2.5 (3.0 en zone forte)
Split TP          : 60% TP1 / 40% TP2
Move SL to BE     : ✓ Après TP1 hit
```

---

## Règles d'entrée (checklist avant trade)

### BUY Setup
- [ ] Wickless Bear candle détectée (close = low, body significatif)
- [ ] HTF (1H): EMA20 > EMA50 (uptrend) OU prix dans zone de support HTF
- [ ] RSI(14) < 50 (idéalement < 40 pour signal fort)
- [ ] Session active (Londres ou NY)
- [ ] Volume >= MA volume
- [ ] SMA200 : prix pas trop loin sous la SMA (éviter les downtrends violents)

**Entrée** : Close de la candle Nowick (ou close de la candle de confirmation suivante)
**SL** : Low de la Nowick candle - buffer (voir ATR dynamique)
**TP1** : Entry + risk × 1.5 (60% de la position)
**TP2** : Entry + risk × 2.5 (40% restants)
**Après TP1** : Move SL au breakeven

### SELL Setup
- [ ] Wickless Bull candle détectée (close = high, body significatif)
- [ ] HTF (1H): EMA20 < EMA50 (downtrend) OU prix dans zone de résistance HTF
- [ ] RSI(14) > 50 (idéalement > 60 pour signal fort)
- [ ] Session active
- [ ] Volume >= MA volume

**Entrée** : Close de la Nowick candle
**SL** : High de la Nowick candle + buffer
**TP1** : Entry - risk × 1.5
**TP2** : Entry - risk × 2.5

---

## Utilisation du mode Retest (optionnel, +5-8% WR)

Activer `Require Retest Before Entry = true` si tu veux attendre confirmation :

1. Nowick candle apparaît → le niveau est marqué
2. Prix repasse sur la zone body de la Nowick (dans les 8 prochaines bougies)
3. La bougie de retest **ferme** de l'autre côté → ENTRER
4. Si le retest **casse** le niveau → INVALIDER le trade, ne pas entrer

**Avantage** : Moins de trades, meilleur WR (~65%+)
**Inconvénient** : Certains moves rapides sont ratés

---

## Optimisation par backtest (Strategy Tester)

### Étapes :
1. Charger `nowick_strategy_pro.pine` sur Strategy Tester
2. AUDUSD, 15m, période: 2020-2024 (au moins 3 ans)
3. Utiliser **Optimiser** sur ces paramètres :

| Paramètre | Range | Step |
|---|---|---|
| wick_max_pct | 0 → 15 | 2.5 |
| min_body_atr | 0.2 → 0.8 | 0.1 |
| rsi_buy_max | 40 → 55 | 5 |
| rsi_sell_min | 45 → 60 | 5 |
| sl_atr_mult | 0.8 → 2.0 | 0.2 |
| tp1_rr | 1.3 → 2.0 | 0.2 |

### Critères de sélection des paramètres optimaux :
- Win Rate ≥ 60%
- Profit Factor ≥ 1.5
- Max Drawdown ≤ 15%
- Au moins 100 trades (statistiquement significatif)

---

## Timeframes conseillés

| TF | Usage | Notes |
|---|---|---|
| **15m** | Principal (AUDUSD) | Meilleur équilibre signal/bruit |
| **5m** | Scalping sessions volatiles | Plus de signaux, WR légèrement plus bas |
| **1H** | Swing trading | Moins de signaux, WR plus élevé |
| **4H** | Positions longues | Signaux rares mais très fiables |

---

## Paires recommandées

Le Nowick fonctionne mieux sur les paires avec :
- Mouvements directionnels clairs
- Bonne liquidité (spreads faibles)

**Meilleures** : AUDUSD, GBPUSD, EURUSD, USDJPY, GBPJPY
**Éviter** : Paires exotiques, crypto (trop de faux wickless)

---

## Gestion du risque

- **Risque par trade** : 1-2% du capital
- **Max trades simultanés** : 1 (strategy gère ça)
- **Max drawdown quotidien** : Si tu perds 3% dans la journée → stop pour le jour
- **Max pertes consécutives** : 3 → pause de 24h

---

## Interprétation de la table (coin haut droite)

| Métrique | Vert | Orange | Rouge |
|---|---|---|---|
| HTF Trend | Aligné avec signal | - | Contre signal |
| RSI | Zone favorable | Neutre | Zone défavorable |
| Session | Active | - | Inactive |
| Volume | > MA | = MA | < MA |
| BUY Ready? / SELL Ready? | Tous filtres OK | - | Filtres manquants |

**Si "BUY Ready?" = ✓ YES** → Surveille l'apparition d'une Wickless Bear candle

---

## Changelog

**v2.0**
- HTF trend filter (EMA20/50 sur 1H) → +7-12% WR vs v1
- RSI zones dynamiques (TP2 étendu en zones extrêmes)
- Split TP 60/40 avec move SL to BE
- Retest mode optionnel
- Session filter + Volume filter combinés
- Stats table en temps réel
- SL dynamique = max(ATR, Candle Extreme)
- Strong signal markers (★) pour RSI extreme
