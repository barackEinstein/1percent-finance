 1Percent - L'Argent Devrait Arriver en Entier

https://img.shields.io/badge/License-MIT-green.svg
https://img.shields.io/badge/Website-1Percent-blue

💸 Le Problème : Pourquoi Payer 3.5% Quand 1% Suffit ?

PayPal prend 3.5%, Stripe 2.9% + 0.30€, Western Union 8-12%. 
La diaspora africaine perd des milliards chaque année en frais abusifs.

🎯 Notre Solution : 1% de Frais, Point Final

1Percent est un système de paiement intelligent qui utilise Nexus Core, notre algorithme open-source, pour trouver systématiquement le chemin le plus rapide et le moins cher.

📊 Comparaison Brutale (sur 100€)

Service Frais Reçu Économie avec 1Percent
PayPal 3.50€ 96.50€ +2.50€
Stripe 3.20€ 96.80€ +2.20€
1Percent 1.00€ 99.00€ REFERENCE

🧠 Comment Nexus Core Fonctionne

```
1. Écoute : Montant, source, destination
2. Analyse : Scan 20+ corridors en temps réel
3. Décide : Trouve la route optimale (coût × vitesse × fiabilité)
4. Exécute : Transfert à 1% de frais maximum
```

🌍 Conçu Depuis l'Afrique, Pour le Monde

· Focus corridors : Europe → Afrique (RDC, Sénégal, Côte d'Ivoire...)
· Intégration native : MPesa, Airtel Money, Orange Money
· Transparence radicale : Vous voyez tout avant de payer

🚀 Démo Immédiate

👉 Tester notre comparateur de frais en direct

🛠️ Stack Technique

· Frontend : HTML5, CSS3, JavaScript (Vanilla)
· Backend : Node.js + Express (en développement)
· Algorithme : Nexus Core (routage intelligent)
· Hébergement : GitHub Pages + GoDaddy

1percent-finance/
├── prototypes/
│   ├── calculator/          # Ton comparateur
│   └── routing-simulator/   # Simulateur d'algorithme
├── docs/
│   ├── ARCHITECTURE.md      # Comment Nexus Core marche
│   └── API_SPEC.md         # Spécifications API
├── research/
│   └── fee-analysis/       # Tes recherches sur les frais
└── assets/                 # Images, logos

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>1Percent - Comparateur de Frais</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: Arial, sans-serif; }
        body { background: #0f172a; color: white; padding: 20px; }
        .container { max-width: 1000px; margin: auto; }
        .header { text-align: center; padding: 3rem 0; }
        h1 { color: #10b981; font-size: 2.5rem; margin-bottom: 1rem; }
        .comparator { background: #1e293b; padding: 2rem; border-radius: 15px; }
        .cards { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; margin: 2rem 0; }
        .card { background: #334155; padding: 1.5rem; border-radius: 10px; text-align: center; }
        .paypal { border-top: 4px solid #003087; }
        .stripe { border-top: 4px solid #635bff; }
        .us { border-top: 4px solid #10b981; background: rgba(16, 185, 129, 0.1); }
        .fee { font-size: 2rem; font-weight: bold; margin: 1rem 0; }
        .paypal .fee { color: #003087; }
        .stripe .fee { color: #635bff; }
        .us .fee { color: #10b981; }
        .cta { text-align: center; margin-top: 2rem; }
        button { background: #10b981; color: white; border: none; padding: 1rem 2rem; border-radius: 8px; font-size: 1.1rem; cursor: pointer; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>1PERCENT COMPARATEUR</h1>
            <p>Comparez ce que vous payez vraiment</p>
        </div>
        
        <div class="comparator">
            <div class="cards">
                <div class="card paypal">
                    <h3>PAYPAL</h3>
                    <div class="fee">3.50€</div>
                    <p>Frais sur 100€</p>
                    <small>3.5% de frais</small>
                </div>
                
                <div class="card stripe">
                    <h3>STRIPE</h3>
                    <div class="fee">3.20€</div>
                    <p>Frais sur 100€</p>
                    <small>2.9% + 0.30€</small>
                </div>
                
                <div class="card us">
                    <h3>1PERCENT</h3>
                    <div class="fee">1.00€</div>
                    <p>Frais sur 100€</p>
                    <small><strong>1% SEULEMENT</strong></small>
                </div>
            </div>
            
            <div class="cta">
                <button onclick="window.open('https://site-tbnf2ydge.godaddysites.com', '_blank')">
                    REJOINDRE LA BÊTA
                </button>
                <p style="margin-top: 1rem; color: #94a3b8; font-size: 0.9rem;">
                    Économisez 2.20€ à 2.50€ par transfert de 100€
                </p>
            </div>
        </div>
    </div>
</body>
</html>
# 🧮 Comparateur de Frais 1Percent

Démo live de notre comparateur qui montre l'économie réelle entre PayPal, Stripe et 1Percent.

## Fonctionnalités
- Comparaison visuelle des frais
- Calcul en temps réel
- Design responsive

## Comment l'utiliser
Ouvrir `index.html` dans un navigateur ou visiter notre [site web](https://site-tbnf2ydge.godaddysites.com).
/**
 * Nexus Core - Algorithme de routage intelligent
 * Version prototype simplifiée
 */

class NexusCore {
    constructor() {
        this.corridors = [
            { name: 'Stripe Direct', cost: 0.029, fixed: 0.30, speed: '1-2 jours' },
            { name: 'PayPal Commerce', cost: 0.035, fixed: 0, speed: '3-5 jours' },
            { name: 'Mobile Money Direct', cost: 0.015, fixed: 0.10, speed: '15 minutes' },
            { name: 'Bank Transfer EU', cost: 0.02, fixed: 1.50, speed: '1-3 jours' }
        ];
    }

    /**
     * Trouve la route optimale pour un transfert
     */
    findOptimalRoute(amount, fromCountry, toCountry) {
        console.log(`🔍 Recherche route optimale pour ${amount}€ de ${fromCountry} à ${toCountry}`);
        
        // 1. Filtrer les corridors disponibles
        const available = this.filterAvailableCorridors(fromCountry, toCountry);
        
        // 2. Calculer le coût total pour chaque
        const routes = available.map(corridor => ({
            name: corridor.name,
            totalCost: (amount * corridor.cost) + corridor.fixed,
            percentage: ((amount * corridor.cost) + corridor.fixed) / amount * 100,
            speed: corridor.speed,
            amountReceived: amount - ((amount * corridor.cost) + corridor.fixed)
        }));
        
        // 3. Trier par coût total (priorité) puis vitesse
        routes.sort((a, b) => {
            if (a.totalCost === b.totalCost) {
                return a.speed.includes('minutes') ? -1 : 1;
            }
            return a.totalCost - b.totalCost;
        });
        
        // 4. Retourner la meilleure
        const bestRoute = routes[0];
        console.log(`✅ Route optimale trouvée : ${bestRoute.name}`);
        console.log(`   Coût : ${bestRoute.totalCost.toFixed(2)}€ (${bestRoute.percentage.toFixed(2)}%)`);
        console.log(`   Vitesse : ${bestRoute.speed}`);
        console.log(`   Reçu : ${bestRoute.amountReceived.toFixed(2)}€`);
        
        return bestRoute;
    }
    
    /**
     * Filtre les corridors disponibles entre deux pays
     */
    filterAvailableCorridors(from, to) {
        // Logique simplifiée - à compléter avec vraie logique
        if (to.includes('CD') || to.includes('SN') || to.includes('CI')) {
            // Pays africains → inclure Mobile Money
            return this.corridors;
        } else {
            // Autres pays → exclure Mobile Money
            return this.corridors.filter(c => !c.name.includes('Mobile Money'));
        }
    }
    
    /**
     * Calcule l'économie vs PayPal
     */
    calculateSavings(amount) {
        const paypalCost = amount * 0.035;
        const ourRoute = this.findOptimalRoute(amount, 'FR', 'CD');
        const savings = paypalCost - ourRoute.totalCost;
        
        return {
            paypalCost: paypalCost.toFixed(2),
            ourCost: ourRoute.totalCost.toFixed(2),
            savings: savings.toFixed(2),
            percentageSaved: ((savings / paypalCost) * 100).toFixed(1)
        };
    }
}

// Exemple d'utilisation
if (typeof module !== 'undefined' && module.exports) {
    module.exports = NexusCore;
} else {
    window.NexusCore = NexusCore;
}
<!DOCTYPE html>
<html>
<head>
    <title>Nexus Core Demo</title>
    <script src="nexus-core.js"></script>
    <style>
        body { font-family: monospace; padding: 20px; background: #0f172a; color: white; }
        .demo { max-width: 600px; margin: auto; }
        input, button { padding: 10px; margin: 5px; }
        .result { background: #1e293b; padding: 15px; border-radius: 10px; margin-top: 20px; }
    </style>
</head>
<body>
    <div class="demo">
        <h2>🧠 Nexus Core - Simulation</h2>
        <input type="number" id="amount" value="100" placeholder="Montant en €">
        <button onclick="runDemo()">Trouver la route optimale</button>
        
        <div id="result" class="result" style="display:none;">
            <h3>📊 Résultat</h3>
            <p id="routeInfo"></p>
            <p id="savingsInfo"></p>
        </div>
    </div>
    
    <script>
        const nexus = new NexusCore();
        
        function runDemo() {
            const amount = parseFloat(document.getElementById('amount').value);
            const result = nexus.findOptimalRoute(amount, 'FR', 'CD');
            const savings = nexus.calculateSavings(amount);
            
            document.getElementById('routeInfo').innerHTML = `
                <strong>Route choisie :</strong> ${result.name}<br>
                <strong>Frais :</strong> ${result.totalCost.toFixed(2)}€ (${result.percentage.toFixed(2)}%)<br>
                <strong>Vitesse :</strong> ${result.speed}<br>
                <strong>Montant reçu :</strong> ${result.amountReceived.toFixed(2)}€
            `;
            
            document.getElementById('savingsInfo').innerHTML = `
                <strong>Économie vs PayPal :</strong> ${savings.savings}€<br>
                <strong>Soit ${savings.percentageSaved}% d'économisé</strong>
            `;
            
            document.getElementById('result').style.display = 'block';
        }
        
        // Démarrer une démo automatique
        setTimeout(runDemo, 1000);
    </script>
</body>
</html>
# 🏗️ Architecture de Nexus Core

## Vue d'ensemble

## Composants

### 1. API Gateway
- Gestion des requêtes HTTP
- Authentification & Autorisation
- Rate limiting

### 2. Nexus Core Engine
- **Crawler** : Récupération des données en temps réel
- **Optimizer** : Calcul de la route optimale
- **Executor** : Exécution des transactions

### 3. Data Layer
- Base de données des corridors
- Cache Redis pour les taux
- Logs des transactions

### 4. Providers Interface
- Abstraction des fournisseurs (PayPal, Stripe, Mobile Money)
- Gestion des erreurs et retry

## Flux de données
1. Requête utilisateur avec montant, source, destination
2. Nexus Core scanne tous les corridors disponibles
3. Calcul du coût total pour chaque option
4. Tri par (coût × vitesse × fiabilité)
5. Exécution via le fournisseur sélectionné
6. Tracking et notification
# 📡 Spécification API 1Percent

## Base URL
`https://api.1percent.com/v1`

## Endpoints

### POST /quote
Calcule le coût d'un transfert

```json
Request:
{
  "amount": 100.00,
  "currency": "EUR",
  "from_country": "FR",
  "to_country": "CD",
  "receiver_type": "mobile_money" // mobile_money, bank_account, cash_pickup
}

Response:
{
  "quote_id": "quo_abc123",
  "amount": 100.00,
  "fees": 1.00,
  "exchange_rate": 2200.00,
  "amount_received": 219000.00,
  "currency_received": "CDF",
  "estimated_delivery": "15 minutes",
  "routes": [
    {
      "provider": "mobile_money_direct",
      "fees": 1.00,
      "speed": "15 minutes",
      "reliability": 99.7
    }
  ]
}

---

## 🔍 **4. RECHERCHE (research/)**

**Fichier : `fee-analysis.md`**
```markdown
# 📊 Analyse des Frais du Marché

## Données collectées (Mars 2026)

### PayPal
- Transferts entre particuliers : 3.5% + 0€ fixe
- Ventes en ligne : 3.4% + 0.35€
- Taux de change : +2.5% à 4% sur le taux interbancaire

### Stripe
- Standard : 2.9% + 0.30€
- Cartes internationales : +1.5%
- Currency conversion : +1% si différente de la devise du compte

### Western Union (France → RDC)
- 100€ → 8.00€ à 10.50€ de frais (8-10.5%)
- Livraison Mobile Money : 15 minutes à 24h
- Taux de change : ~5% de marge

### MoneyGram
- Similaire à Western Union
- Frais légèrement inférieurs (7.5-9.5%)

### TransferWise/Wise
- Frais transparents : ~0.5% + taux réel
- Mais limitations sur l'Afrique

## Opportunité identifiée
Le corridor France → RDC présente les frais les plus élevés (8-12%) 
pour un service qui pourrait techniquement coûter <1%.
# 🚀 1Percent - L'argent devrait arriver en entier

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

## 💸 Le Problème
PayPal prend 3.5%. Stripe prend 2.9% + 0.30€. Western Union prend 8-12%. 
**Pourquoi envoyer de l'argent coûte-t-il 10x plus que le nécessaire ?**

## 🎯 Notre Solution
**1Percent** - Un système de paiement intelligent qui trouve systématiquement le chemin le moins cher via notre algorithme **Nexus Core**.

### Comparaison des Frais (sur 100€)
| Service | Frais | Reçu | Pourcentage |
|---------|-------|------|------------|
| PayPal | 3.50€ | 96.50€ | 3.5% |
| Stripe | 3.20€ | 96.80€ | 2.9% + 0.30€ |
| **1Percent** | **1.00€** | **99.00€** | **1%** |

## 🔧 Comment ça marche

## 🌍 Focus Afrique
Nous construisons spécifiquement pour les corridors **Europe → Afrique** avec intégration native des réseaux Mobile Money.

## 🚀 Démo Live
👉 **[Tester le comparateur de frais](https://site-tbnf2ydge.godaddysites.com)**

## 👨💻 L'Équipe
**Barack Ndenga** - Fondateur & Développeur  
Développeur congolais de Mont-Ngafula, Kinshasa.

## 📄 Licence
MIT License - Voir [LICENSE](LICENSE) pour les détails.

---
*"Si la technologie peut commander un taxi à l'autre bout du monde en 30 secondes, elle peut aussi y envoyer de l'argent pour le prix d'une baguette."*
MIT License

Copyright (c) 2026 1Percent

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.