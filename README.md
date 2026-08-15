# Paquant

**Natural-language quantitative research ideas → structured Python strategy code.**

Paquant is an AI-powered quantitative research prototype that converts plain-English trading ideas into clean, backtest-ready Python.

Instead of asking a general-purpose coding assistant to interpret a trading strategy from scratch, Paquant uses a deliberately engineered system prompt that constrains the model around **quantitative research conventions, point-in-time data handling, execution assumptions, transaction costs, risk metrics, and reproducible code structure**.

> **Portfolio project:** Paquant is no longer hosted as a live application. This repository contains the original MVP source code and can be run locally with your own OpenAI API key.

---

## What Paquant Does

A researcher can describe a strategy in ordinary English:

```text
Buy SPY when the 20-day moving average crosses above the 50-day moving average.
Sell when it crosses back below.
```

Paquant turns that idea into a complete Python research module with a standardized structure for:

```python
validate_ohlcv()
ingest()
features()
signal()
execute()
metrics()
run()
```

The goal is simple:

**reduce the distance between a trading idea and testable code.**

Paquant does **not** predict markets, execute trades, connect to brokerages, or provide investment advice.

It is a research and prototyping tool.

---

## Why I Built It

Quantitative strategy development often involves a large amount of repetitive implementation work before an idea can even be tested.

A researcher may understand exactly what they want to investigate but still need to write:

* data validation
* feature calculations
* signal logic
* execution assumptions
* position alignment
* transaction-cost modeling
* performance metrics
* backtest orchestration

Paquant explores a simple question:

> **Can an LLM act less like a chatbot and more like a constrained compiler for quantitative research?**

Rather than optimizing the model for conversation, I optimized the application for one task:

**take a research hypothesis and return executable Python.**

---

# System Prompt Engineering

The most important part of Paquant is not the interface.

It is the **behavioral specification given to the language model**.

I designed the system prompt to behave more like an engineering contract than a traditional chatbot instruction.

The model is given explicit rules for what it can generate, how strategies must be structured, how financial data must be handled, and what kinds of mistakes it should avoid.

## 1. Constrained Identity

The model is explicitly instructed to behave as a:

> quantitative finance code compiler

rather than a conversational assistant.

Its output is restricted to Python code instead of explanations, markdown, tutorials, or general financial commentary.

This keeps the application's behavior focused on a single workflow.

---

## 2. Standardized Research Interface

Every generated strategy follows the same functional architecture:

```python
def validate_ohlcv(df):
    ...

def ingest(df):
    ...

def features(df):
    ...

def signal(df):
    ...

def execute(df, pos, cost_bps, exec_convention="open_to_open"):
    ...

def metrics(df):
    ...

def run(df, cost_bps=0.0, exec_convention="open_to_open"):
    ...
```

This gives generated strategies a predictable structure regardless of the trading idea supplied by the user.

Instead of producing arbitrary scripts, Paquant attempts to generate interchangeable research modules.

---

## 3. Point-in-Time Discipline

Financial backtests are extremely sensitive to accidental look-ahead bias.

The prompt therefore includes explicit requirements around:

* chronological data
* unique timestamps
* signal alignment
* lagged positions
* point-in-time feature availability
* avoiding forward-looking calculations
* avoiding double-lagging during execution

For example, positions are required to be aligned to the market-data index before applying the execution lag:

```python
pos_lag = pos.reindex(df.index).shift(1).fillna(0)
```

The purpose is to make correct temporal alignment part of the generation contract rather than something the model is expected to remember implicitly.

---

## 4. Explicit Execution Conventions

The model is given defined interpretations for several common execution assumptions:

```text
open_to_open
close_to_close
close_to_open
open_to_close
```

That matters because a strategy's apparent performance can change dramatically depending on when a signal becomes available and when a trade is assumed to execute.

Paquant forces the generated code to make those assumptions explicit.

---

## 5. Transaction-Cost Modeling

Generated strategies also account for turnover and execution costs.

The prompt defines:

```text
turnover = |position(t) - position(t-1)|

cost = turnover × cost_bps × 1e-4
```

This prevents the model from treating backtests as frictionless by default.

---

## 6. Standardized Performance Metrics

Paquant requires generated research modules to calculate a consistent group of performance statistics, including:

* Sharpe ratio
* CAGR
* maximum drawdown
* Calmar ratio
* win rate
* trade count
* average turnover
* transaction-cost drag
* information ratio when a benchmark is available

This makes results easier to compare across generated strategies.

---

## 7. Restricted Dependency Surface

To make generated strategies portable, the default computational stack is intentionally small:

```text
Python
NumPy
pandas
typing
math
datetime
```

The model is instructed not to introduce additional libraries unless the user explicitly opts into them.

This reduces dependency assumptions and makes the generated code easier to move between research environments.

---

## 8. Fail-Fast Validation

Generated strategies are instructed to validate incoming OHLCV data rather than silently repair malformed inputs.

Expected fields include:

```text
open
high
low
close
volume
```

The generated code checks characteristics such as:

* required columns
* numeric types
* chronological timestamps
* duplicate timestamps
* missing data
* valid volume values

Invalid data should produce a clear error instead of silently changing the dataset.

---

## 9. Deterministic Generation

The OpenAI request is configured with:

```javascript
temperature: 0.0
```

Paquant is intended to behave more like a repeatable engineering tool than a creative assistant.

The system prompt therefore emphasizes:

* deterministic behavior
* explicit interfaces
* constrained outputs
* predictable code structure
* minimal ambiguity

---

# Architecture

Paquant uses a deliberately small architecture.

```text
┌─────────────────────────────┐
│         Researcher          │
│                             │
│  Plain-English strategy     │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│       Browser Interface     │
│                             │
│  HTML / CSS / JavaScript    │
└──────────────┬──────────────┘
               │
               │ POST /api/ask
               ▼
┌─────────────────────────────┐
│       Server API Route      │
│                             │
│  Validation                 │
│  Payload limits             │
│  Rate limiting              │
│  Conversation handling      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│         OpenAI API          │
│                             │
│  Quant-specific system      │
│  prompt + user hypothesis   │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│      Python Strategy        │
│                             │
│  Validation                 │
│  Features                   │
│  Signal                     │
│  Execution                  │
│  Metrics                    │
└─────────────────────────────┘
```

There is no database, brokerage connection, trading engine, or complicated agent architecture.

The application is intentionally focused on the transformation:

**research idea → structured quantitative code**

---

# Application Features

### Natural-language strategy input

Users describe a quantitative idea without having to manually construct the initial Python implementation.

### Python code generation

The backend sends the conversation and Paquant's system specification to the OpenAI API.

### Syntax-highlighted output

Generated Python is rendered using Prism.js.

### Editable results

The generated code can be edited directly inside the interface.

### One-click copy

Users can copy generated strategies into their preferred research environment.

### Conversation context

Paquant maintains short conversational context so a researcher can refine a strategy across multiple prompts.

### Request validation

The API validates:

* HTTP method
* content type
* JSON structure
* message count
* individual message length
* total conversation size

### Rate limiting

The backend includes lightweight IP-based request throttling.

### Server-side API credentials

The OpenAI API key is read only from:

```javascript
process.env.OPENAI_API_KEY
```

It is never included in the client-side application.

---

# Tech Stack

**Frontend**

* HTML
* CSS
* Vanilla JavaScript
* Three.js
* Prism.js

**Backend**

* JavaScript
* Serverless API route
* Node.js

**AI**

* OpenAI API
* GPT-4.1
* custom quantitative-finance system prompt

**Quantitative output**

* Python
* pandas
* NumPy

---

# Running Paquant Locally

## 1. Clone the repository

```bash
git clone <your-repository-url>
cd paquant
```

## 2. Install dependencies

```bash
npm install
```

## 3. Add your OpenAI API key

Create a `.env.local` file:

```env
OPENAI_API_KEY=your_openai_api_key_here
```

Do not commit this file.

A typical `.gitignore` should contain:

```gitignore
.env
.env.local
node_modules/
.vercel/
```

## 4. Run the project

Run the application using your preferred local/serverless development environment.

The frontend sends requests to:

```text
POST /api/ask
```

and the API route communicates with OpenAI using the API key supplied through your environment variables.

---

# Example Workflow

### Input

```text
Create a momentum strategy that goes long when the 20-day moving average
crosses above the 50-day moving average and exits when it crosses below.
```

### Paquant

The system interprets the hypothesis and generates a complete research module containing:

```text
OHLCV validation
        ↓
data ingestion
        ↓
feature engineering
        ↓
signal generation
        ↓
position alignment
        ↓
execution simulation
        ↓
transaction costs
        ↓
performance metrics
        ↓
backtest result
```

The researcher can then inspect, modify, and test the resulting Python in their own research environment.

---

# Engineering Philosophy

Paquant was built around a principle I use frequently when developing AI applications:

> **LLMs become significantly more useful when their freedom is intentionally constrained.**

Instead of giving the model a vague instruction such as:

```text
Write a trading strategy in Python.
```

Paquant defines:

* the model's role
* the allowable libraries
* the expected input schema
* the required function signatures
* the execution methodology
* the temporal-alignment rules
* the transaction-cost calculation
* the required performance metrics
* the permitted output format
* the expected error behavior

The system prompt effectively acts as a **domain-specific specification layer between the user and the foundation model**.

The objective was not to make the AI more creative.

It was to make it **more predictable, useful, and auditable for one specific workflow**.

---

# What This Project Demonstrates

Paquant was built as an applied AI project rather than as a production trading platform.

From an engineering perspective, it demonstrates experience with:

* LLM application architecture
* system prompt engineering
* domain-specific AI workflows
* structured model outputs
* prompt constraint design
* OpenAI API integration
* API security patterns
* server-side environment variables
* input validation
* rate limiting
* frontend/backend integration
* asynchronous UI state
* quantitative-finance concepts
* rapid prototyping
* deploying an end-to-end AI product

Most importantly, Paquant demonstrates the ability to take a broad idea—

**"use AI to accelerate quantitative research"**

—and turn it into a functioning, narrowly scoped software product.

---

# Project Scope

Paquant is a **research prototype**.

It does not:

* place trades
* manage capital
* connect to brokerage accounts
* retrieve live market data
* guarantee profitable strategies
* guarantee generated code is error-free
* provide investment advice

Generated strategies should always be independently reviewed and tested before use.

The project is intended to demonstrate AI-assisted quantitative research tooling and LLM workflow engineering.

---

# Demo

An original MVP demonstration is available here:

**YouTube:**
https://youtu.be/n4O5ZCSSCF8

The demo shows Paquant converting a plain-English moving-average strategy into Python code.

---

# Status

**Archived portfolio project**

Paquant was originally built in 2025 as an independent AI application and quantitative-finance experiment.

The hosted application is no longer active, but the source code is preserved publicly as part of my software and applied-AI portfolio.

---

## Built by Niko DiCarlo

Solo-designed and developed as an independent applied AI project.
