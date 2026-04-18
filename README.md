# 💰 Sistema Financiero

> **Sistema de gestión financiera personal con IA integrada**
> Dashboard moderno + Chat con IA + OCR para tickets + Análisis en tiempo real

[![Next.js](https://img.shields.io/badge/Next.js-15.5.4-black?logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-green?logo=supabase)](https://supabase.com/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-4.0-38bdf8?logo=tailwindcss)](https://tailwindcss.com/)

---

## 🎯 ¿Qué es esto?

Un sistema completo para **rastrear gastos e ingresos** con:
- 📊 **Dashboard visual** con KPIs y gráficas
- 🤖 **Agente IA** que registra transacciones por chat
- 📸 **OCR** para escanear tickets/recibos
- 📝 **Registro manual** con formularios
- 📈 **Reportes** con análisis de tendencias

**Perfecto para:** Individuos, familias, o pequeños negocios que quieren controlar sus finanzas.

---

## 🤖 FOR AI ASSISTANTS - READ THIS FIRST

> **Atención Claude/GPT/Gemini:** Esta sección te explica cómo modificar este proyecto.

### Arquitectura del Proyecto

```
┌─────────────────────────────────────────────────────┐
│         Next.js 15 (Full-Stack Framework)           │
├─────────────────────────────────────────────────────┤
│  FRONTEND (Client-Side - Runs in Browser)          │
│  ├─ app/page.tsx          → Dashboard principal    │
│  ├─ app/registro/         → Formulario manual      │
│  ├─ app/agente-mejorado/  → Chat con IA            │
│  └─ components/*.tsx      → UI reutilizable        │
├─────────────────────────────────────────────────────┤
│  BACKEND (Server-Side - Runs in Node.js)           │
│  └─ app/api/              → API Routes             │
│      ├─ transacciones/route.ts  → GET transacciones│
│      ├─ chat/stream/route.ts    → Chat con IA     │
│      └─ upload-image/route.ts   → OCR de tickets  │
├─────────────────────────────────────────────────────┤
│  DATABASE (Supabase - PostgreSQL)                   │
│  └─ transacciones table   → ÚNICA tabla necesaria  │
└─────────────────────────────────────────────────────┘
```

### Database Schema (CRITICAL - Copy/Paste to Supabase)

**ONLY ONE TABLE NEEDED:** `transacciones`

```sql
-- Table: transacciones (Stores all income and expenses)
CREATE TABLE transacciones (
  -- Primary Key
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),

  -- Core Transaction Fields
  fecha TIMESTAMP NOT NULL DEFAULT NOW(),
  tipo TEXT CHECK (tipo IN ('ingreso', 'gasto')) NOT NULL,
  monto NUMERIC(10, 2) NOT NULL CHECK (monto > 0),
  categoria TEXT NOT NULL,

  -- Optional Details
  concepto TEXT DEFAULT 'Transacción manual',
  descripcion TEXT,
  metodo_pago TEXT CHECK (metodo_pago IN ('Efectivo', 'Tarjeta', 'Transferencia')),
  registrado_por TEXT,
  foto_url TEXT,

  -- Metadata
  usuario_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_transacciones_fecha ON transacciones(fecha DESC);
CREATE INDEX idx_transacciones_tipo ON transacciones(tipo);
CREATE INDEX idx_transacciones_usuario ON transacciones(usuario_id);

-- Enable Row Level Security (RLS)
ALTER TABLE transacciones ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own transactions
CREATE POLICY "Users can view own transactions"
  ON transacciones FOR SELECT
  USING (auth.uid() = usuario_id);

CREATE POLICY "Users can insert own transactions"
  ON transacciones FOR INSERT
  WITH CHECK (auth.uid() = usuario_id);
```

### Valid Categories (Hardcoded - NOT a table)

```typescript
// Gastos (Expenses)
CATEGORIAS_GASTOS = [
  'Alimentación', 'Transporte', 'Vivienda', 'Salud',
  'Entretenimiento', 'Educación', 'Otros Gastos'
]

// Ingresos (Income)
CATEGORIAS_INGRESOS = [
  'Salario', 'Ventas', 'Servicios', 'Inversiones', 'Otros Ingresos'
]

// Métodos de Pago
METODOS_PAGO = ['Efectivo', 'Tarjeta', 'Transferencia']
```

### Key Files Map

| File Path | Purpose | When to Modify |
|-----------|---------|----------------|
| `app/page.tsx` | Dashboard with KPIs and charts | Change dashboard layout/KPIs |
| `app/api/transacciones/route.ts` | GET endpoint for transactions | Change how data is fetched |
| `app/api/chat/stream/route.ts` | AI chat with function calling | Modify AI behavior/prompts |
| `app/api/upload-image/route.ts` | OCR for ticket scanning | Change OCR logic |
| `components/DataViews.tsx` | Transaction table with filters | Modify table columns/filters |
| `components/TrendChart.tsx` | Line chart for trends | Change chart visualization |
| `hooks/useEnhancedChat.ts` | Chat state management | Add chat features |
| `lib/supabase.ts` | Supabase client config | Change DB connection |

### Common Modifications

**1. Add a new category:**
```typescript
// Location: app/registro/page.tsx or app/api/chat/stream/route.ts
const CATEGORIAS_GASTOS = [
  'Alimentación', 'Transporte', 'Vivienda', 'Salud',
  'Entretenimiento', 'Educación', 'Otros Gastos',
  'Tu Nueva Categoría'  // ← Add here
]
```

**2. Add a new field to transactions:**
```sql
-- 1. First, add column to Supabase:
ALTER TABLE transacciones ADD COLUMN nuevo_campo TEXT;

-- 2. Then update TypeScript interface in components/DataViews.tsx:
interface Transaccion {
  id: string
  fecha: string
  tipo: 'gasto' | 'ingreso'
  categoria: string
  monto: number
  descripcion: string
  metodo_pago: string
  nuevo_campo: string  // ← Add here
}
```

**3. Change AI model:**
```typescript
// Location: app/api/chat/stream/route.ts (line ~54)
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  body: JSON.stringify({
    model: 'google/gemini-2.5-flash',  // ← Change model here
    // Available: 'anthropic/claude-3.5-sonnet', 'openai/gpt-4o', etc.
  })
})
```

**4. Customize dashboard KPIs:**
```typescript
// Location: app/page.tsx (lines ~70-97)
// The fetchKPIs() function calculates totals
// Modify the logic to add new KPIs or change calculations
```

### Data Flow

```
User Action → Frontend Component → API Route → Supabase → Response
     ↓              ↓                   ↓           ↓          ↓
  Click       app/page.tsx      app/api/**/*.ts  PostgreSQL  JSON
```

**Example: Registering a transaction via AI chat**
1. User types: "Gasté $200 en gasolina"
2. `hooks/useEnhancedChat.ts` sends to `/api/chat/stream`
3. AI (Gemini 2.5 Flash) calls `registrar_gasto` function
4. Backend inserts to Supabase `transacciones` table
5. Frontend updates automatically

---

## 🚀 Quick Start (For Users)

### Prerequisites

- **Node.js 20+** ([Download](https://nodejs.org/))
- **Supabase Account** ([Sign Up Free](https://supabase.com/))
- **OpenRouter API Key** ([Get it here](https://openrouter.ai/)) - Optional for AI features

### Step 1: Clone Repository

```bash
git clone https://github.com/isairey/Sistema-Financero.git
cd Sistema-Financiero
```

### Step 2: Install Dependencies

```bash
npm install
```

### Step 3: Setup Supabase

1. Go to [supabase.com](https://supabase.com/) → Create New Project
2. Go to **SQL Editor** → Paste the SQL schema from above ↑
3. Click **Run** to create the `transacciones` table
4. Go to **Project Settings** → **API** → Copy:
   - `Project URL`
   - `anon public` key

### Step 4: Configure Environment Variables

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
# Supabase (REQUIRED)
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# OpenRouter (OPTIONAL - only for AI features)
OPENROUTER_API_KEY=sk-or-v1-...

# Site URL (for OpenRouter attribution)
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### Step 5: Run Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) 🎉

**First time?**
- Dashboard will show $0.00 (no data yet)
- Go to **Registro** to add your first transaction
- Or go to **Agente IA** to chat with the AI

---

## 📁 Project Structure

```
sistema-financiero-app/
├── app/                          # Next.js App Router
│   ├── page.tsx                 # 📊 Dashboard (KPIs + Charts)
│   ├── registro/page.tsx        # 📝 Manual Form
│   ├── agente-mejorado/page.tsx # 🤖 AI Chat Interface
│   ├── corte-diario/page.tsx    # 📅 Daily Cut (bulk entry)
│   └── api/                     # Backend API Routes
│       ├── transacciones/route.ts  # GET /api/transacciones
│       ├── chat/stream/route.ts    # POST /api/chat/stream
│       └── upload-image/route.ts   # POST /api/upload-image (OCR)
│
├── components/                   # React Components
│   ├── Header.tsx               # Navigation bar
│   ├── KPICard.tsx              # Metric cards (Ingresos/Gastos/Balance)
│   ├── TrendChart.tsx           # Line chart (Chart.js)
│   ├── DataViews.tsx            # Transaction table with filters
│   └── ThemeToggle.tsx          # Dark/Light mode switcher
│
├── hooks/                        # Custom React Hooks
│   ├── useEnhancedChat.ts       # Chat state + streaming
│   └── useImageUpload.ts        # Image upload to Supabase Storage
│
├── lib/                          # Utilities
│   └── supabase.ts              # Supabase client singleton
│
├── .env.example                  # Environment variables template
├── .env.local                    # Your secrets (git-ignored)
├── package.json                  # Dependencies
└── README.md                     # You are here!
```

---

## 🎨 Features

### 1. Dashboard (Home Page)

**What it shows:**
- KPIs: Total ingresos, gastos, balance, # transacciones
- Trend chart: Income vs Expenses over time
- Transaction table: Grouped by date, filterable
- Date range selector: Daily, Weekly, Monthly, Custom

**How it works:**
- Fetches from `/api/transacciones?vista=mensual`
- Groups data by date in frontend (`TrendChart.tsx`)
- Calculates totals in `app/page.tsx`

### 2. Agente IA (AI Chat)

**What it does:**
- Natural language transaction entry
- OCR: Upload ticket photos → auto-extract amount/category
- Streaming responses with "thinking" indicator
- Function calling to register transactions

**Tech:**
- Model: `google/gemini-2.5-flash` via OpenRouter
- Streaming: Server-Sent Events (SSE)
- OCR: Gemini Vision API
- Storage: Supabase Storage (`facturas` bucket)

**Example conversation:**
```
User: "Gasté $200 en gasolina"
AI: [Thinking...] → [Writing...]
    "¡Perfecto! Registré tu gasto:
    💰 Monto: $200 MXN
    📁 Categoría: Transporte
    💳 Método: Efectivo

    ✅ Guardado en la base de datos"
```

### 3. Registro Manual

**What it does:**
- Form-based transaction entry
- Upload invoice/receipt photo (optional)
- Select category from dropdown
- Choose payment method

**Use case:** When you prefer forms over chatting with AI

### 4. Corte Diario (Daily Cut)

**What it does:**
- Bulk entry for multiple transactions at once
- Useful for end-of-day reconciliation
- Registers all categories in one form

**Use case:** Restaurant/retail businesses doing daily cash counts

---

## 🛠️ Development

### Available Scripts

```bash
npm run dev      # Start development server (port 3000)
npm run build    # Build for production
npm run start    # Run production build locally
```

### Tech Stack Details

**Frontend:**
- **Next.js 15.5.4** - React framework with App Router
- **React 19** - UI library
- **TypeScript 5** - Type safety
- **Tailwind CSS 4** - Utility-first styling
- **Chart.js** - Data visualization
- **Lucide React** - Icon library
- **next-themes** - Dark mode support

**Backend:**
- **Next.js API Routes** - Serverless functions
- **Node.js 20** - Runtime
- **Supabase SDK** - Database client
- **OpenRouter API** - Multi-LLM gateway

**Database:**
- **PostgreSQL** (via Supabase)
- **Row Level Security** (RLS) enabled
- **Real-time subscriptions** (not used yet, but available)

---

## 🚢 Deployment

### Deploy to Vercel (Recommended)

1. Push your code to GitHub
2. Go to [vercel.com](https://vercel.com/) → **New Project**
3. Import your repository
4. Add environment variables:
   ```
   NEXT_PUBLIC_SUPABASE_URL=...
   NEXT_PUBLIC_SUPABASE_ANON_KEY=...
   OPENROUTER_API_KEY=...
   NEXT_PUBLIC_SITE_URL=https://your-app.vercel.app
   ```
5. Click **Deploy** → Done in ~2 minutes!

**Why Vercel?**
- Zero config (detects Next.js automatically)
- Free hobby tier
- Global CDN
- Automatic HTTPS
- CI/CD built-in

### Other Platforms

Also works on:
- **Netlify** - Similar to Vercel
- **Railway** - Supports PostgreSQL + Next.js
- **Render** - Good for full-stack apps
- **Cloudflare Pages** - Fast edge deployment

---

## 🔐 Security Notes

### Environment Variables
- **NEVER commit `.env.local`** to GitHub
- Use Vercel/Netlify UI to set production secrets
- `.env.example` is safe to commit (no real values)

### Supabase RLS Policies
- **Row Level Security (RLS)** is ENABLED
- Users can only see their own transactions
- Policies are defined in the SQL schema above

### API Keys
- **OpenRouter API Key** - Keep secret, server-side only
- **Supabase Anon Key** - Safe to expose (read-only without RLS bypass)

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repo
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit changes: `git commit -m 'feat: add amazing feature'`
4. Push to branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

### Commit Convention

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: new feature
fix: bug fix
docs: documentation changes
style: formatting, semicolons, etc.
refactor: code refactoring
test: add tests
chore: maintenance
```

---

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details.

---

## 🙏 Credits

- **[Next.js](https://nextjs.org/)** - The React Framework
- **[Supabase](https://supabase.com/)** - Open Source Firebase Alternative
- **[OpenRouter](https://openrouter.ai/)** - Unified LLM API
- **[Tailwind CSS](https://tailwindcss.com/)** - Utility-First CSS
- **[Chart.js](https://www.chartjs.org/)** - Simple Charting
- **[Lucide Icons](https://lucide.dev/)** - Beautiful Icons

---

## 📧 Support

- **Issues:** [GitHub Issues](https://github.com/isairey/Sistema-Financiero/issues)
- **Discussions:** [GitHub Discussions](https://github.com/isairey/Sistema-Financiero/discussions)
- **Email:** isaireyes2003@gmail.com

---

## 🎓 Learn More

- **Next.js Docs:** [nextjs.org/docs](https://nextjs.org/docs)
- **Supabase Docs:** [supabase.com/docs](https://supabase.com/docs)
- **OpenRouter Docs:** [openrouter.ai/docs](https://openrouter.ai/docs)
- **Tailwind CSS Docs:** [tailwindcss.com/docs](https://tailwindcss.com/docs)

---

<div align="center">

**Built with ❤️ using [Claude Code](https://claude.com/claude-code)**

⭐ Star this repo if you find it useful!

</div>
