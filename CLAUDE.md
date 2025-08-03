# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A web application for optimizing baseball batting lineups using Monte Carlo simulation and heuristic optimization. Users can import player data from Google Spreadsheets, manually arrange lineups, and run simulations to find optimal batting orders.

## Architecture

* **Frontend**: React 18 + Vite + TypeScript 5
* **State Management**: TanStack Query (CSV data fetching), Zustand (lineup/history state)
* **Data Processing**: PapaParse for CSV parsing
* **Simulation**: Web Worker with Monte Carlo algorithms
* **Visualization**: Chart.js 4 for statistical charts
* **Deployment**: GitHub Pages with GitHub Actions

## Development Commands

Since the project is not yet implemented, these are the planned commands based on the design:

```bash
# Development
npm run dev              # Start development server
npm run build           # Build for production
npm run lint            # Run ESLint
npm run typecheck       # Run TypeScript compiler checks

# Testing
# Tests will be implemented using automated UI verification with curl + grep
```

## Key Components Structure

```
src/
├── config.ts                 # Application configuration
├── hooks/
│   └── usePlayers.ts         # Player data fetching hook
├── worker/
│   └── simulator.ts          # Monte Carlo simulation worker
├── components/
│   ├── PlayerTable.tsx       # Player selection table
│   ├── LineupEditor.tsx      # Drag-and-drop lineup editor
│   └── HistoryGrid.tsx       # Optimization history display
└── stores/                   # Zustand state stores
```

## Data Source

Primary data source is a Google Spreadsheet containing NPB player statistics:
[https://docs.google.com/spreadsheets/d/18OlCqMkP8gNGskMy-NWRXU8Cfb4F7K5g7FelTauhLn4/edit?usp=sharing](https://docs.google.com/spreadsheets/d/18OlCqMkP8gNGskMy-NWRXU8Cfb4F7K5g7FelTauhLn4/edit?usp=sharing)

Expected CSV format includes: Team, Player Name, Batting Average, Games, At-Bats, Hits, Home Runs, etc.

## Simulation Engine

* Uses cumulative probability distributions cached in Float32Array
* Binary search for efficient random outcome selection
* Base runner state managed with bit patterns (0b000-0b111)
* Heuristic optimization via single-position swaps with improvement tracking

## Development Workflow

### Git Worktree Setup

Use git worktree for feature branch development:

```bash
# Create and switch to feature branch with worktree
git worktree add ../batting-lineup-optimizer-2-feature feature/branch-name

# Work in the worktree directory
cd ../batting-lineup-optimizer-2-feature

# When done, remove worktree
git worktree remove ../batting-lineup-optimizer-2-feature
```

### Development Process

1. **Feature Development**: Create feature branches using git worktree and test locally
2. **GitHub Pages Deployment**: Merge to gh-pages branch for staging
3. **Production**: Merge to main branch after staging verification
4. **Testing**: Automated DOM element verification using curl + grep patterns

## Testing Strategy

Automated verification checks for key DOM elements:

```bash
curl -sL <url> | grep -q "<h1>Batting Lineup Optimizer</h1>"
```

Test coverage includes main routes: `/`, `/simulation`, `/history`

