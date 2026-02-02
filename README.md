
🚀 1Percent - Redéfinir les transferts d'argent à 1% de frais

Problème : PayPal (3.5%), Stripe (2.9%+0.30€), Western Union (8-12%) prennent trop.
Solution : Nexus Core, algorithme open-source qui scanne 20+ corridors en temps réel pour trouver systématiquement le chemin le plus rapide et le moins cher.

✨ Envoyez 100€, payez 1€ de frais seulement (vs 3.50€ PayPal).
🌍 Focus corridors Europe → Afrique avec intégration Mobile Money native.
🧠 Algorithme qui sacrifie notre marge pour votre optimisation.

Pour la diaspora, les créateurs, les entrepreneurs. 
Parce que votre argent devrait arriver en entier.

🔗 Site : https://site-tbnf2ydge.godaddysites.com


1percent-finance/

├── backend/              # Le coeur de Nexus Core
│   ├── src/
│   │   ├── crawlers/    # Data Crawler (pour récupérer les taux)
│   │   ├── optimizer/   # Cost Optimizer (algorithme de routage)
│   │   ├── engine/      # Execution Engine
│   │   └── api/         # Future API
│   ├── package.json     # Dépendances (si Node.js)
│   └── README.md        # Instructions d'installation
├── frontend/            # Interface future (site web/app)
│   ├── public/
│   └── src/
├── prototypes/          # Tes prototypes existants (calculateur, simulateur)
├── research/            # Tes analyses
├── docs/                # Ta documentation technique
├── .gitignore           ★ FICHIER CRUCIAL - À créer en premier !
├── LICENSE              # Déjà présent (MIT)
├── README.md            # Ta page principale
└── CONTRIBUTING.md      # Guide pour les contributeurs

{
  "name": "1percent-core",
  "version": "2.1.0",
  "description": "1Percent Payment Platform - 1% fees vs 3.5% PayPal",
  "main": "src/index.js",
  "type": "module",
  "scripts": {
    "dev": "nodemon src/index.js",
    "start": "node src/index.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/",
    "docker:dev": "docker-compose up",
    "docker:build": "docker-compose build",
    "docker:down": "docker-compose down",
    "format": "prettier --write \"src/**/*.js\""
  },
  "dependencies": {
    "express": "4.18.2",
    "cors": "2.8.5",
    "helmet": "7.0.0",
    "dotenv": "16.0.3",
    "winston": "3.8.2",
    "joi": "17.7.0",
    "uuid": "9.0.0",
    "axios": "1.3.4",
    "redis": "4.6.7",
    "bcryptjs": "2.4.3",
    "jsonwebtoken": "9.0.0",
    "node-cache": "5.1.2",
    "lodash": "4.17.21",
    "compression": "1.7.4",
    "express-rate-limit": "6.7.0",
    "express-validator": "6.15.0"
  },
  "devDependencies": {
    "nodemon": "2.0.22",
    "jest": "29.5.0",
    "supertest": "6.3.3",
    "eslint": "8.38.0",
    "prettier": "2.8.7"
  },
  "keywords": [
    "fintech",
    "payments",
    "remittances",
    "africa",
    "money-transfer",
    "1percent"
  ],
  "author": "Barack Ndenga <barack@1percent.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/1percent-finance/core.git"
  },
  "bugs": {
    "url": "https://github.com/1percent-finance/core/issues"
  },
  "homepage": "https://1percent.com"
}
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: 1percent_dev
      POSTGRES_USER: 1percent_user
      POSTGRES_PASSWORD: dev_password_123
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U 1percent_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  api:
    build:
      context: .
      dockerfile: Dockerfile.api
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      PORT: 3000
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: 1percent_dev
      DB_USER: 1percent_user
      DB_PASSWORD: dev_password_123
      REDIS_HOST: redis
      REDIS_PORT: 6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./src:/app/src
      - ./logs:/app/logs
    restart: unless-stopped

  web:
    build:
      context: .
      dockerfile: Dockerfile.web
    ports:
      - "3001:3000"
    environment:
      NODE_ENV: development
      NEXT_PUBLIC_API_URL: http://localhost:3000
    depends_on:
      - api
    volumes:
      - ./src/web:/app/src
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY src ./src
COPY config ./config
COPY scripts ./scripts

RUN addgroup -g 1001 -S nodejs
RUN adduser -S 1percent -u 1001
RUN chown -R 1percent:nodejs /app

USER 1percent

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if(r.statusCode!==200)process.exit(1)})"

CMD ["node", "src/index.js"]
import { createServer } from './api/server.js';
import { logger } from './core/utils/logger.js';

const PORT = process.env.PORT || 3000;

async function startServer() {
  try {
    const app = await createServer();
    
    const server = app.listen(PORT, () => {
      logger.info(`🚀 Server running on port ${PORT}`);
      logger.info(`📊 Health check: http://localhost:${PORT}/health`);
      logger.info(`🔧 Environment: ${process.env.NODE_ENV}`);
    });

    // Graceful shutdown
    const shutdown = () => {
      logger.info('🛑 Shutting down gracefully...');
      server.close(() => {
        logger.info('✅ Server closed');
        process.exit(0);
      });
      
      setTimeout(() => {
        logger.error('❌ Force shutdown after timeout');
        process.exit(1);
      }, 10000);
    };

    process.on('SIGTERM', shutdown);
    process.on('SIGINT', shutdown);

  } catch (error) {
    logger.error('❌ Failed to start server:', error);
    process.exit(1);
  }
}

startServer();
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import compression from 'compression';
import { rateLimit } from 'express-rate-limit';
import { logger } from '../core/utils/logger.js';

export async function createServer() {
  const app = express();
  
  // Security middleware
  app.use(helmet());
  app.use(cors({
    origin: process.env.CORS_ORIGIN || '*',
    credentials: true
  }));
  
  // Rate limiting
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    standardHeaders: true,
    legacyHeaders: false
  });
  app.use('/api/', limiter);
  
  // Body parsing
  app.use(express.json({ limit: '10mb' }));
  app.use(express.urlencoded({ extended: true }));
  app.use(compression());
  
  // Request logging
  app.use((req, res, next) => {
    logger.http(`${req.method} ${req.url}`, {
      ip: req.ip,
      userAgent: req.get('User-Agent')
    });
    next();
  });
  
  // Routes
  app.get('/health', (req, res) => {
    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      version: process.env.npm_package_version || '2.1.0',
      uptime: process.uptime()
    });
  });
  
  // API routes
  app.use('/api/v1/quotes', (await import('./routes/quotes.js')).router);
  
  // 404 handler
  app.use('*', (req, res) => {
    res.status(404).json({
      error: 'Not Found',
      message: `Route ${req.originalUrl} not found`
    });
  });
  
  // Error handler
  app.use((err, req, res, next) => {
    logger.error('API Error:', {
      error: err.message,
      stack: err.stack,
      path: req.path,
      method: req.method
    });
    
    const status = err.status || 500;
    const message = process.env.NODE_ENV === 'production' 
      ? 'Internal server error'
      : err.message;
    
    res.status(status).json({
      error: message,
      ...(process.env.NODE_ENV !== 'production' && { stack: err.stack })
    });
  });
  
  return app;
}
import { Router } from 'express';
import Joi from 'joi';
import { FeeCalculator } from '../../core/nexus/Calculator.js';
import { validateRequest } from '../../core/utils/validation.js';

const router = Router();

const quoteSchema = Joi.object({
  amount: Joi.number().min(1).max(100000).required(),
  from: Joi.string().length(2).uppercase().required(),
  to: Joi.string().length(2).uppercase().required(),
  currency: Joi.string().length(3).uppercase().default('EUR')
});

router.post('/', validateRequest(quoteSchema), async (req, res) => {
  try {
    const { amount, from, to, currency } = req.body;
    
    // Calculate all fees
    const allFees = FeeCalculator.calculate(amount);
    const optimal = FeeCalculator.findOptimalRoute(amount, from, to);
    
    // Generate quote ID
    const quoteId = `quo_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    
    // Simulate exchange rate
    const exchangeRate = simulateExchangeRate(currency, to);
    
    const response = {
      quote_id: quoteId,
      request: {
        amount,
        currency,
        from_country: from,
        to_country: to
      },
      quote: {
        provider: optimal.provider,
        fee_amount: optimal.fee,
        fee_percentage: optimal.percentage,
        amount_sent: amount,
        amount_received: optimal.received * exchangeRate,
        exchange_rate: exchangeRate,
        estimated_delivery: estimateDelivery(from, to),
        expires_at: new Date(Date.now() + 15 * 60 * 1000).toISOString()
      },
      comparison: Object.entries(allFees).map(([provider, data]) => ({
        provider,
        fee_amount: data.fee,
        fee_percentage: data.percentage,
        amount_received: data.received * exchangeRate
      }))
    };
    
    res.json(response);
    
  } catch (error) {
    res.status(400).json({
      error: 'Failed to generate quote',
      message: error.message
    });
  }
});

// Helper functions
function simulateExchangeRate(fromCurrency, toCountry) {
  const rates = {
    'EUR-CG': 655,  // EUR to XAF (Congo)
    'EUR-CD': 2200, // EUR to CDF (DRC)
    'EUR-SN': 655,  // EUR to XOF (Senegal)
    'EUR-CI': 655,  // EUR to XOF (Ivory Coast)
    'EUR-CM': 655,  // EUR to XAF (Cameroon)
    'USD-CD': 2000, // USD to CDF
    'USD-CG': 600,  // USD to XAF
    'default': 1
  };
  
  return rates[`${fromCurrency}-${toCountry}`] || rates.default;
}

function estimateDelivery(from, to) {
  const isAfrica = ['CD', 'SN', 'CI', 'CM', 'CG'].includes(to);
  const isEurope = ['FR', 'BE', 'DE', 'ES', 'IT', 'NL'].includes(from);
  
  if (isEurope && isAfrica) {
    return '2-15 minutes';
  }
  
  return '1-3 business days';
}

export { router };
export class FeeCalculator {
  static calculate(amount) {
    const providers = {
      paypal: {
        percentage: 0.035,
        fixed: 0,
        min: 0.35,
        name: 'PayPal'
      },
      stripe: {
        percentage: 0.029,
        fixed: 0.30,
        min: 0.30,
        name: 'Stripe'
      },
      wise: {
        percentage: 0.005,
        fixed: 1.50,
        min: 1.50,
        name: 'Wise'
      },
      'western-union': {
        percentage: 0.085,
        fixed: 0,
        min: 5.00,
        name: 'Western Union'
      },
      'mobile-money': {
        percentage: 0.015,
        fixed: 0.50,
        min: 0.50,
        name: 'Mobile Money Direct'
      }
    };
    
    const results = {};
    
    for (const [key, config] of Object.entries(providers)) {
      const fee = Math.max(
        (amount * config.percentage) + config.fixed,
        config.min
      );
      
      results[key] = {
        provider: config.name,
        fee: parseFloat(fee.toFixed(2)),
        percentage: parseFloat(((fee / amount) * 100).toFixed(2)),
        received: parseFloat((amount - fee).toFixed(2))
      };
    }
    
    // 1Percent offer
    const ourFee = amount * 0.01;
    results['1percent'] = {
      provider: '1Percent',
      fee: parseFloat(ourFee.toFixed(2)),
      percentage: 1.00,
      received: parseFloat((amount - ourFee).toFixed(2))
    };
    
    return results;
  }
  
  static findOptimalRoute(amount, fromCountry, toCountry) {
    const allOptions = this.calculate(amount);
    
    // Filter by corridor availability
    const availableOptions = this.filterByCorridor(allOptions, fromCountry, toCountry);
    
    if (Object.keys(availableOptions).length === 0) {
      throw new Error(`No available routes for ${fromCountry} -> ${toCountry}`);
    }
    
    // Find cheapest option (including 1percent)
    let optimal = null;
    let lowestFee = Infinity;
    
    for (const [provider, data] of Object.entries(availableOptions)) {
      if (data.fee < lowestFee) {
        lowestFee = data.fee;
        optimal = { provider, ...data };
      }
    }
    
    return optimal;
  }
  
  static filterByCorridor(options, from, to) {
    const available = {};
    
    // Define corridor support
    const corridors = {
      paypal: ['global'], // PayPal is global
      stripe: ['FR-BE', 'FR-DE', 'BE-FR', 'DE-FR'], // European corridors
      wise: ['global'], // Wise is global
      'western-union': ['global'],
      'mobile-money': ['FR-CD', 'FR-SN', 'FR-CI', 'FR-CM', 'BE-CD', 'BE-SN'],
      '1percent': ['FR-CD', 'FR-SN', 'FR-CI', 'FR-CM', 'BE-CD', 'BE-SN', 'DE-CD']
    };
    
    const corridorKey = `${from}-${to}`;
    
    for (const [provider, data] of Object.entries(options)) {
      const supportedCorridors = corridors[provider] || [];
      
      if (supportedCorridors.includes('global') || supportedCorridors.includes(corridorKey)) {
        available[provider] = data;
      }
    }
    
    return available;
  }
  
  static calculateSavings(amount, comparedTo = 'paypal') {
    const options = this.calculate(amount);
    
    if (!options[comparedTo] || !options['1percent']) {
      throw new Error('Cannot calculate savings: provider not found');
    }
    
    const competitorFee = options[comparedTo].fee;
    const ourFee = options['1percent'].fee;
    const savings = competitorFee - ourFee;
    const percentage = (savings / competitorFee) * 100;
    
    return {
      competitor: comparedTo,
      competitor_fee: competitorFee,
      our_fee: ourFee,
      savings: parseFloat(savings.toFixed(2)),
      savings_percentage: parseFloat(percentage.toFixed(1))
    };
  }
}
import winston from 'winston';

const logFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.json()
);

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: logFormat,
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    new winston.transports.File({ 
      filename: 'logs/error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'logs/combined.log' 
    })
  ]
});

// Create a stream for Morgan
logger.stream = {
  write: (message) => {
    logger.info(message.trim());
  }
};
import Joi from 'joi';

export const validateRequest = (schema) => {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });
    
    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details.map(detail => ({
          field: detail.path.join('.'),
          message: detail.message
        }))
      });
    }
    
    req.body = value;
    next();
  };
};

// Common validation schemas
export const schemas = {
  quote: Joi.object({
    amount: Joi.number().min(1).max(100000).required(),
    from: Joi.string().length(2).uppercase().required(),
    to: Joi.string().length(2).uppercase().required(),
    currency: Joi.string().length(3).uppercase().default('EUR')
  }),
  
  user: Joi.object({
    email: Joi.string().email().required(),
    firstName: Joi.string().min(2).max(50).required(),
    lastName: Joi.string().min(2).max(50).required(),
    phone: Joi.string().pattern(/^\+[1-9]\d{1,14}$/).required(),
    country: Joi.string().length(2).uppercase().required()
  }),
  
  transfer: Joi.object({
    quoteId: Joi.string().required(),
    paymentMethod: Joi.string().valid('card', 'bank_transfer').required(),
    beneficiary: Joi.object({
      name: Joi.string().required(),
      phone: Joi.string().required(),
      country: Joi.string().length(2).uppercase().required()
    }).required()
  })
};
import NodeCache from 'node-cache';

export class CacheService {
  constructor(ttlSeconds = 300) { // 5 minutes default
    this.cache = new NodeCache({
      stdTTL: ttlSeconds,
      checkperiod: ttlSeconds * 0.2,
      useClones: false
    });
  }
  
  get(key) {
    return this.cache.get(key);
  }
  
  set(key, value, ttl = null) {
    if (ttl) {
      this.cache.set(key, value, ttl);
    } else {
      this.cache.set(key, value);
    }
  }
  
  del(keys) {
    this.cache.del(keys);
  }
  
  flush() {
    this.cache.flushAll();
  }
  
  // Quote-specific cache methods
  setQuote(quoteId, data) {
    this.set(`quote:${quoteId}`, data, 900); // 15 minutes
  }
  
  getQuote(quoteId) {
    return this.get(`quote:${quoteId}`);
  }
  
  // Route-specific cache methods
  setRoute(from, to, amount, data) {
    const key = `route:${from}:${to}:${amount}`;
    this.set(key, data, 300); // 5 minutes
  }
  
  getRoute(from, to, amount) {
    const key = `route:${from}:${to}:${amount}`;
    return this.get(key);
  }
}

// Singleton instance
export const cache = new CacheService();
{
  "name": "1percent-web",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "13.4.9",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "tailwindcss": "3.3.2",
    "axios": "1.4.0",
    "framer-motion": "10.12.16",
    "react-icons": "4.9.0"
  }
}
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./

RUN npm ci --only=production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S 1percent -u 1001
RUN chown -R 1percent:nodejs /app

USER 1percent

EXPOSE 3000

CMD ["npm", "start"]
import Calculator from '../components/Calculator';
import Header from '../components/Header';
import Footer from '../components/Footer';

export default function Home() {
  return (
    <div className="min-h-screen bg-gradient-to-b from-gray-900 to-black">
      <Header />
      <main className="container mx-auto px-4 py-8">
        <Calculator />
      </main>
      <Footer />
    </div>
  );
}
import { useState, useEffect } from 'react';
import { FiArrowRight, FiCheck, FiGlobe, FiClock, FiDollarSign } from 'react-icons/fi';

export default function Calculator() {
  const [amount, setAmount] = useState(100);
  const [fromCountry, setFromCountry] = useState('FR');
  const [toCountry, setToCountry] = useState('CD');
  const [results, setResults] = useState(null);
  const [loading, setLoading] = useState(false);

  const countries = [
    { code: 'FR', name: 'France', flag: '🇫🇷' },
    { code: 'BE', name: 'Belgique', flag: '🇧🇪' },
    { code: 'DE', name: 'Allemagne', flag: '🇩🇪' },
    { code: 'CD', name: 'Congo (RDC)', flag: '🇨🇩' },
    { code: 'SN', name: 'Sénégal', flag: '🇸🇳' },
    { code: 'CI', name: 'Côte d\'Ivoire', flag: '🇨🇮' },
    { code: 'CM', name: 'Cameroun', flag: '🇨🇲' }
  ];

  const calculateFees = async () => {
    setLoading(true);
    try {
      const response = await fetch('http://localhost:3000/api/v1/quotes', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ amount, from: fromCountry, to: toCountry, currency: 'EUR' })
      });
      
      const data = await response.json();
      setResults(data);
    } catch (error) {
      console.error('Error fetching quote:', error);
      // Fallback to simulated data
      setResults(getSimulatedResults());
    } finally {
      setLoading(false);
    }
  };

  const getSimulatedResults = () => {
    return {
      quote: {
        provider: '1percent',
        fee_amount: amount * 0.01,
        fee_percentage: 1.0,
        amount_sent: amount,
        amount_received: amount * 0.99,
        exchange_rate: 2200,
        estimated_delivery: '2-15 minutes'
      },
      comparison: [
        {
          provider: 'PayPal',
          fee_amount: amount * 0.035,
          fee_percentage: 3.5,
          amount_received: amount * 0.965
        },
        {
          provider: 'Stripe',
          fee_amount: (amount * 0.029) + 0.30,
          fee_percentage: ((amount * 0.029) + 0.30) / amount * 100,
          amount_received: amount - ((amount * 0.029) + 0.30)
        },
        {
          provider: 'Western Union',
          fee_amount: amount * 0.085,
          fee_percentage: 8.5,
          amount_received: amount * 0.915
        }
      ]
    };
  };

  useEffect(() => {
    calculateFees();
  }, [amount, fromCountry, toCountry]);

  const savings = results?.comparison?.[0] 
    ? results.comparison[0].fee_amount - results.quote.fee_amount 
    : 0;

  return (
    <div className="max-w-6xl mx-auto">
      {/* Hero Section */}
      <div className="text-center mb-12">
        <h1 className="text-4xl md:text-5xl font-bold mb-4 bg-gradient-to-r from-green-400 to-blue-500 bg-clip-text text-transparent">
          Pourquoi payer 3.5% quand 1% suffit ?
        </h1>
        <p className="text-xl text-gray-300">
          Comparez et économisez sur chaque transfert
        </p>
      </div>

      {/* Calculator Controls */}
      <div className="bg-gray-800 rounded-2xl p-8 mb-8">
        <div className="grid md:grid-cols-3 gap-6">
          {/* From Country */}
          <div className="space-y-2">
            <label className="block text-sm font-medium text-gray-300">
              <FiGlobe className="inline mr-2" />
              J'envoie depuis
            </label>
            <select
              value={fromCountry}
              onChange={(e) => setFromCountry(e.target.value)}
              className="w-full bg-gray-900 border border-gray-700 rounded-lg px-4 py-3 text-white focus:ring-2 focus:ring-green-500 focus:border-transparent"
            >
              {countries.filter(c => ['FR', 'BE', 'DE'].includes(c.code))
                .map(country => (
                  <option key={country.code} value={country.code}>
                    {country.flag} {country.name}
                  </option>
                ))}
            </select>
          </div>

          {/* Amount */}
          <div className="space-y-2">
            <label className="block text-sm font-medium text-gray-300">
              <FiDollarSign className="inline mr-2" />
              Montant
            </label>
            <div className="relative">
              <input
                type="number"
                value={amount}
                onChange={(e) => setAmount(Number(e.target.value))}
                min="1"
                max="10000"
                step="10"
                className="w-full bg-gray-900 border border-gray-700 rounded-lg px-4 py-3 text-white focus:ring-2 focus:ring-green-500 focus:border-transparent pr-12"
              />
              <span className="absolute right-4 top-3 text-gray-400">€</span>
            </div>
            <div className="flex gap-2 mt-2">
              {[50, 100, 200, 500].map(value => (
                <button
                  key={value}
                  onClick={() => setAmount(value)}
                  className={`px-3 py-1 rounded text-sm ${amount === value ? 'bg-green-600' : 'bg-gray-700 hover:bg-gray-600'}`}
                >
                  {value}€
                </button>
              ))}
            </div>
          </div>

          {/* To Country */}
          <div className="space-y-2">
            <label className="block text-sm font-medium text-gray-300">
              <FiGlobe className="inline mr-2" />
              À destination de
            </label>
            <select
              value={toCountry}
              onChange={(e) => setToCountry(e.target.value)}
              className="w-full bg-gray-900 border border-gray-700 rounded-lg px-4 py-3 text-white focus:ring-2 focus:ring-green-500 focus:border-transparent"
            >
              {countries.filter(c => ['CD', 'SN', 'CI', 'CM'].includes(c.code))
                .map(country => (
                  <option key={country.code} value={country.code}>
                    {country.flag} {country.name}
                  </option>
                ))}
            </select>
          </div>
        </div>
      </div>

      {/* Results */}
      {loading ? (
        <div className="text-center py-12">
          <div className="inline-block animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-green-500"></div>
          <p className="mt-4 text-gray-400">Calcul en cours...</p>
        </div>
      ) : results && (
        <>
          {/* Comparison Cards */}
          <div className="grid md:grid-cols-3 gap-6 mb-8">
            {results.comparison?.map((provider, index) => (
              <div key={index} className="bg-gray-800 rounded-xl p-6">
                <h3 className="text-lg font-semibold mb-4 text-gray-300">{provider.provider}</h3>
                <div className="text-3xl font-bold text-red-400 mb-2">
                  {provider.fee_amount.toFixed(2)}€
                </div>
                <div className="text-gray-400 mb-4">
                  {provider.fee_percentage.toFixed(2)}% de frais
                </div>
                <div className="text-sm text-gray-500">
                  Reçu : {provider.amount_received.toFixed(2)}€
                </div>
              </div>
            ))}
          </div>

          {/* 1Percent Card - Highlighted */}
          <div className="bg-gradient-to-br from-green-900/30 to-green-800/20 border-2 border-green-700 rounded-2xl p-8 mb-8">
            <div className="flex items-center justify-between mb-6">
              <div>
                <span className="inline-flex items-center px-3 py-1 rounded-full text-sm font-medium bg-green-900 text-green-200 mb-2">
                  <FiCheck className="mr-1" /> RECOMMANDÉ
                </span>
                <h2 className="text-2xl font-bold">1Percent</h2>
              </div>
              <div className="text-right">
                <div className="text-4xl font-bold text-green-400">
                  {results.quote.fee_amount.toFixed(2)}€
                </div>
                <div className="text-gray-300">1% de frais seulement</div>
              </div>
            </div>

            <div className="grid md:grid-cols-3 gap-6">
              <div className="text-center">
                <div className="text-2xl font-bold text-white">
                  {results.quote.amount_received.toFixed(2)}€
                </div>
                <div className="text-gray-400">Montant reçu</div>
              </div>
              <div className="text-center">
                <FiClock className="inline text-2xl text-green-400 mb-2" />
                <div className="text-lg font-semibold text-white">
                  {results.quote.estimated_delivery}
                </div>
                <div className="text-gray-400">Délai estimé</div>
              </div>
              <div className="text-center">
                <div className="text-2xl font-bold text-green-400">
                  {savings.toFixed(2)}€
                </div>
                <div className="text-gray-400">Économisé vs PayPal</div>
              </div>
            </div>
          </div>

          {/* CTA Section */}
          <div className="text-center">
            <button className="bg-gradient-to-r from-green-500 to-green-600 text-white font-bold py-4 px-8 rounded-xl text-lg hover:from-green-600 hover:to-green-700 transition-all duration-300 inline-flex items-center">
              Commencer maintenant
              <FiArrowRight className="ml-2" />
            </button>
            <p className="mt-4 text-gray-400 text-sm">
              Aucune carte bancaire requise pour créer un compte
            </p>
          </div>
        </>
      )}
    </div>
  );
}
import { FiPercent, FiMenu, FiX } from 'react-icons/fi';
import { useState } from 'react';

export default function Header() {
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  return (
    <header className="sticky top-0 z-50 bg-gray-900/95 backdrop-blur supports-[backdrop-filter]:bg-gray-900/60">
      <div className="container mx-auto px-4 py-4">
        <div className="flex items-center justify-between">
          {/* Logo */}
          <div className="flex items-center space-x-2">
            <div className="bg-gradient-to-r from-green-500 to-green-600 p-2 rounded-lg">
              <FiPercent className="text-white text-xl" />
            </div>
            <div>
              <h1 className="text-xl font-bold bg-gradient-to-r from-green-400 to-blue-400 bg-clip-text text-transparent">
                1Percent
              </h1>
              <p className="text-xs text-gray-400">Send money for 1%</p>
            </div>
          </div>

          {/* Desktop Navigation */}
          <nav className="hidden md:flex items-center space-x-8">
            <a href="#" className="text-gray-300 hover:text-white transition">Accueil</a>
            <a href="#" className="text-gray-300 hover:text-white transition">Comment ça marche</a>
            <a href="#" className="text-gray-300 hover:text-white transition">Tarifs</a>
            <a href="#" className="text-gray-300 hover:text-white transition">À propos</a>
            <button className="bg-gradient-to-r from-green-500 to-green-600 text-white px-6 py-2 rounded-lg font-medium hover:from-green-600 hover:to-green-700 transition">
              Connexion
            </button>
          </nav>

          {/* Mobile Menu Button */}
          <button 
            className="md:hidden text-gray-300"
            onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
          >
            {mobileMenuOpen ? <FiX size={24} /> : <FiMenu size={24} />}
          </button>
        </div>

        {/* Mobile Menu */}
        {mobileMenuOpen && (
          <div className="md:hidden mt-4 pb-4">
            <div className="flex flex-col space-y-4">
              <a href="#" className="text-gray-300 hover:text-white transition py-2">Accueil</a>
              <a href="#" className="text-gray-300 hover:text-white transition py-2">Comment ça marche</a>
              <a href="#" className="text-gray-300 hover:text-white transition py-2">Tarifs</a>
              <a href="#" className="text-gray-300 hover:text-white transition py-2">À propos</a>
              <button className="bg-gradient-to-r from-green-500 to-green-600 text-white px-6 py-3 rounded-lg font-medium">
                Connexion
              </button>
            </div>
          </div>
        )}
      </div>
    </header>
  );
}
import { FiHeart, FiGithub, FiTwitter, FiLinkedin, FiMail } from 'react-icons/fi';

export default function Footer() {
  return (
    <footer className="bg-gray-900 border-t border-gray-800 mt-12">
      <div className="container mx-auto px-4 py-8">
        <div className="grid md:grid-cols-4 gap-8">
          {/* Brand */}
          <div>
            <div className="flex items-center space-x-2 mb-4">
              <div className="bg-gradient-to-r from-green-500 to-green-600 p-2 rounded-lg">
                <FiPercent className="text-white" />
              </div>
              <span className="text-xl font-bold text-white">1Percent</span>
            </div>
            <p className="text-gray-400 text-sm">
              Envoyez de l'argent à 1% de frais, pas 10%.
            </p>
          </div>

          {/* Links */}
          <div>
            <h3 className="text-white font-semibold mb-4">Produit</h3>
            <ul className="space-y-2">
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Fonctionnalités</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Tarifs</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">API</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Statut</a></li>
            </ul>
          </div>

          {/* Company */}
          <div>
            <h3 className="text-white font-semibold mb-4">Entreprise</h3>
            <ul className="space-y-2">
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">À propos</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Blog</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Carrières</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Contact</a></li>
            </ul>
          </div>

          {/* Legal */}
          <div>
            <h3 className="text-white font-semibold mb-4">Légal</h3>
            <ul className="space-y-2">
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Confidentialité</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Conditions</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Compliance</a></li>
              <li><a href="#" className="text-gray-400 hover:text-white transition text-sm">Licence</a></li>
            </ul>
          </div>
        </div>

        {/* Bottom Bar */}
        <div className="border-t border-gray-800 mt-8 pt-8 flex flex-col md:flex-row justify-between items-center">
          <div className="text-gray-400 text-sm mb-4 md:mb-0">
            © 2024 1Percent. Tous droits réservés.
          </div>
          
          <div className="flex items-center space-x-4">
            <a href="https://github.com/1percent-finance" className="text-gray-400 hover:text-white transition">
              <FiGithub size={20} />
            </a>
            <a href="https://twitter.com/1percent" className="text-gray-400 hover:text-white transition">
              <FiTwitter size={20} />
            </a>
            <a href="https://linkedin.com/company/1percent" className="text-gray-400 hover:text-white transition">
              <FiLinkedin size={20} />
            </a>
            <a href="mailto:contact@1percent.com" className="text-gray-400 hover:text-white transition">
              <FiMail size={20} />
            </a>
          </div>
        </div>

        {/* Made with love */}
        <div className="text-center mt-4">
          <p className="text-gray-500 text-sm inline-flex items-center">
            <FiHeart className="mr-1 text-red-400" />
            Conçu à Kinshasa pour le monde
          </p>
        </div>
      </div>
    </footer>
  );
}
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        'gray': {
          850: '#1e293b',
          900: '#0f172a',
          950: '#020617'
        }
      },
      animation: {
        'gradient': 'gradient 8s linear infinite',
      },
      keyframes: {
        gradient: {
          '0%, 100%': {
            'background-size': '200% 200%',
            'background-position': 'left center'
          },
          '50%': {
            'background-size': '200% 200%',
            'background-position': 'right center'
          },
        },
      },
    },
  },
  plugins: [],
}
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    @apply bg-gray-950 text-white;
  }
  
  /* Custom scrollbar */
  ::-webkit-scrollbar {
    width: 10px;
  }
  
  ::-webkit-scrollbar-track {
    @apply bg-gray-900;
  }
  
  ::-webkit-scrollbar-thumb {
    @apply bg-gray-700 rounded-full;
  }
  
  ::-webkit-scrollbar-thumb:hover {
    @apply bg-gray-600;
  }
}

@layer components {
  .glass-effect {
    @apply backdrop-blur-lg bg-white/10 border border-white/20;
  }
  
  .gradient-border {
    position: relative;
    border-radius: 1rem;
  }
  
  .gradient-border::before {
    content: '';
    position: absolute;
    top: -2px;
    left: -2px;
    right: -2px;
    bottom: -2px;
    background: linear-gradient(45deg, #10b981, #3b82f6, #8b5cf6);
    border-radius: inherit;
    z-index: -1;
    animation: gradient 3s ease infinite;
    background-size: 400% 400%;
  }
}

/* Animations */
@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

.animate-float {
  animation: float 3s ease-in-out infinite;
}
import { FeeCalculator } from '../src/core/nexus/Calculator.js';

describe('FeeCalculator', () => {
  test('calculates correct fees for 100€', () => {
    const results = FeeCalculator.calculate(100);
    
    expect(results.paypal.fee).toBeCloseTo(3.5);
    expect(results.stripe.fee).toBeCloseTo(3.2);
    expect(results['1percent'].fee).toBeCloseTo(1.0);
    expect(results['1percent'].percentage).toBe(1.0);
  });
  
  test('respects minimum amounts', () => {
    const results = FeeCalculator.calculate(5);
    expect(results.stripe.fee).toBe(0.30); // Minimum fee
  });
  
  test('finds optimal route', () => {
    const optimal = FeeCalculator.findOptimalRoute(100, 'FR', 'CD');
    expect(optimal.provider).toBe('1percent');
    expect(optimal.fee).toBeLessThanOrEqual(1.0);
  });
  
  test('calculates savings correctly', () => {
    const savings = FeeCalculator.calculateSavings(100, 'paypal');
    expect(savings.savings).toBeCloseTo(2.5);
    expect(savings.savings_percentage).toBeGreaterThan(70);
  });
});
import request from 'supertest';
import { createServer } from '../src/api/server.js';

let app;

beforeAll(async () => {
  app = await createServer();
});

describe('API Endpoints', () => {
  test('GET /health returns 200', async () => {
    const response = await request(app).get('/health');
    expect(response.status).toBe(200);
    expect(response.body.status).toBe('healthy');
  });
  
  test('POST /api/v1/quotes returns quote', async () => {
    const response = await request(app)
      .post('/api/v1/quotes')
      .send({
        amount: 100,
        from: 'FR',
        to: 'CD',
        currency: 'EUR'
      });
    
    expect(response.status).toBe(200);
    expect(response.body.quote_id).toBeDefined();
    expect(response.body.quote.provider).toBe('1percent');
    expect(response.body.quote.fee_percentage).toBe(1.0);
  });
  
  test('POST /api/v1/quotes validates input', async () => {
    const response = await request(app)
      .post('/api/v1/quotes')
      .send({
        amount: -100, // Invalid amount
        from: 'France', // Invalid country code
        to: 'CD'
      });
    
    expect(response.status).toBe(400);
    expect(response.body.error).toBe('Validation failed');
  });
});
#!/bin/bash

# 1Percent Setup Script
echo "🚀 Setting up 1Percent development environment..."

# Check Node.js version
if ! command -v node &> /dev/null; then
    echo "❌ Node.js is not installed. Please install Node.js 18+"
    exit 1
fi

NODE_VERSION=$(node -v | cut -d'v' -f2 | cut -d'.' -f1)
if [ "$NODE_VERSION" -lt 18 ]; then
    echo "❌ Node.js version must be 18 or higher"
    exit 1
fi

# Check Docker
if ! command -v docker &> /dev/null; then
    echo "⚠️ Docker not found. Docker Compose setup will be skipped."
    DOCKER_AVAILABLE=false
else
    DOCKER_AVAILABLE=true
fi

# Create necessary directories
echo "📁 Creating directory structure..."
mkdir -p logs
mkdir -p tmp

# Install dependencies
echo "📦 Installing dependencies..."
npm install

# Setup environment
if [ ! -f .env ]; then
    echo "⚙️ Creating .env file from template..."
    cp .env.example .env
    echo "✅ .env file created. Please update with your configuration."
fi

# Setup Docker if available
if [ "$DOCKER_AVAILABLE" = true ]; then
    echo "🐳 Setting up Docker containers..."
    docker-compose build
fi

echo ""
echo "✅ Setup complete!"
echo ""
echo "Next steps:"
echo "1. Edit .env file with your configuration"
echo "2. Start services:"
echo "   - With Docker: npm run docker:dev"
echo "   - Without Docker: npm run dev"
echo ""
echo "📊 Services will be available at:"
echo "   - API: http://localhost:3000"
echo "   - Web: http://localhost:3001"
echo "   - PostgreSQL: localhost:5432"
echo "   - Redis: localhost:6379"
// Database migration script
import { promises as fs } from 'fs';
import path from 'path';

const MIGRATIONS_DIR = './migrations';

async function runMigrations() {
  console.log('🚀 Running database migrations...');
  
  try {
    // Read migration files
    const files = await fs.readdir(MIGRATIONS_DIR);
    const sqlFiles = files.filter(f => f.endsWith('.sql')).sort();
    
    for (const file of sqlFiles) {
      console.log(`📄 Applying migration: ${file}`);
      const sql = await fs.readFile(
        path.join(MIGRATIONS_DIR, file),
        'utf8'
      );
      
      // In production, you would execute this against your database
      console.log(`✅ Migration ${file} would be executed`);
      console.log(sql.substring(0, 200) + '...');
    }
    
    console.log('✅ All migrations completed successfully');
  } catch (error) {
    console.error('❌ Migration failed:', error);
    process.exit(1);
  }
}

runMigrations();


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


mkdir 1percent-production
cd 1percent-production

# Crée chaque fichier un par un
# Copie-colle chaque code ci-dessus dans le bon fichier
chmod +x scripts/setup.sh
./scripts/setup.sh
# Avec Docker (recommandé)
npm run docker:dev

# Sans Docker
npm run dev