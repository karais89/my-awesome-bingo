# Soc Ops - AI Coding Agent Instructions

## Project Overview
Social Bingo game for in-person mixers. Find people who match questions and get 5 in a row. Built with React 19, TypeScript, Vite, and Tailwind CSS v4. Deploys automatically to GitHub Pages.

## Architecture

### Core Data Flow
The game follows a unidirectional data flow pattern:
1. **useBingoGame hook** (`src/hooks/useBingoGame.ts`) - Central state management with localStorage persistence
2. **Pure game logic** (`src/utils/bingoLogic.ts`) - Immutable functions for board operations
3. **Stateless UI components** (`src/components/`) - Presentational React components

### Key Types (`src/types/index.ts`)
- `BingoSquareData`: Individual square with id, text, marked state, free space flag
- `BingoLine`: Winning line definition (type, index, square indices)
- `GameState`: 'start' | 'playing' | 'bingo'

### State Management Pattern
**Critical**: The hook manages game state with automatic localStorage persistence:
- Uses `STORAGE_VERSION` for schema validation (currently v1)
- Validates stored data with strict runtime type checking (`validateStoredData`)
- Persists on every state change via `useEffect`
- Loads persisted state on initialization, falls back to 'start' state
- SSR-safe: All localStorage access guarded with `typeof window === 'undefined'`

### Board Logic (Pure Functions)
All game operations in `bingoLogic.ts` are **pure and immutable**:
- `generateBoard()`: Creates new 5x5 board with shuffled questions, FREE space at index 12
- `toggleSquare(board, squareId)`: Returns new board with square toggled (free space immutable)
- `checkBingo(board)`: Returns first complete line or null
- `getWinningSquareIds(line)`: Converts winning line to Set<number> for O(1) lookup

**Never mutate board state directly** - always return new arrays/objects.

## Development Workflow

### Commands
```bash
npm run dev       # Start dev server (currently running)
npm run build     # TypeScript check + Vite build
npm run test      # Run Vitest tests
npm run lint      # ESLint
```

### Testing
- Framework: Vitest with jsdom environment
- Test setup: `src/test/setup.ts` (jest-dom matchers, auto-cleanup)
- Tests: `src/utils/bingoLogic.test.ts` - comprehensive coverage of pure functions
- Pattern: Test pure logic functions, not UI components
- Run tests before committing logic changes

## Tailwind CSS v4 Convention

### Configuration (No JS config)
Uses `@theme` directive in `src/index.css` (NOT `tailwind.config.js`):
```css
@theme {
  --color-accent: #2563eb;
  --color-accent-light: #3b82f6;
  --color-marked: #dcfce7;
  --color-marked-border: #22c55e;
  --color-bingo: #fbbf24;
}
```

### Usage
- Refer to `.github/instructions/tailwind-4.instructions.md` for v4 patterns
- Use `@import "tailwindcss"` - no plugins or config files
- Vite plugin: `@tailwindcss/vite` in `vite.config.ts`
- Use semantic color tokens from `@theme` (e.g., `bg-marked`, `border-marked-border`)
- v4 changes: `bg-black/50` instead of `bg-opacity-50`

## Component Patterns

### Component Structure
- **App.tsx**: Routes between Start/Game screens, modal conditionals
- **StartScreen.tsx**: Entry point with "Start Game" button
- **GameScreen.tsx**: Header, instructions, board, reset button
- **BingoBoard.tsx**: 5x5 grid layout, maps squares to BingoSquare components
- **BingoSquare.tsx**: Individual square with mark state, winning line styling, free space handling
- **BingoModal.tsx**: Victory celebration modal

### Component Conventions
- **Props**: Type interfaces above component, e.g., `interface GameScreenProps`
- **Styling**: Tailwind utility classes only (no CSS modules or styled-components)
- **Interactions**: `onClick` handlers passed down, never modify state in components
- **Accessibility**: Use `aria-pressed` for toggleable squares, `aria-label` for screen readers
- **Stateless components**: All state lives in `useBingoGame` hook

### BingoSquare Pattern (Critical)
Square styling logic is complex - three states:
1. Unmarked: `bg-white text-gray-700 active:bg-gray-100`
2. Marked, non-winning: `bg-marked border-marked-border text-green-800`
3. Marked, winning: `bg-amber-200 border-amber-400 text-amber-900`
Free space has special styling + `disabled` attribute.

## Adding New Questions

Edit `src/data/questions.ts`:
- Array of 25+ question strings
- Each should be a short, social-interaction prompt
- FREE_SPACE constant for center square
- Questions are shuffled on game start

## Build & Deployment

### Vite Config
- Base path auto-detects from `VITE_REPO_NAME` env var (set by GitHub Actions)
- Plugins: `@vitejs/plugin-react`, `@tailwindcss/vite`
- Test: Vitest with jsdom, auto-run disabled (`watch: false`)

### GitHub Actions
- `.github/workflows/deploy.yml` - Auto-deploys to GitHub Pages on push to main
- Node 22 required
- Sets `VITE_REPO_NAME` for correct base path in production

## Design Philosophy

When adding UI features, refer to `.github/instructions/frontend-design.instructions.md`:
- **Avoid generic AI aesthetics**: No Inter/Roboto, purple gradients, or cookie-cutter designs
- **Typography**: Use distinctive, context-appropriate fonts
- **Color**: Commit to cohesive palettes with dominant colors and sharp accents
- **Motion**: Prioritize CSS-only animations, focus on high-impact moments
- **Creativity**: Make unexpected choices that feel genuinely designed

Current design is minimal/refined - maintain restraint, precision, and careful attention to spacing.

## Common Tasks

### Adding a New Game Feature
1. Update types in `src/types/index.ts` if needed
2. Add pure logic to `src/utils/bingoLogic.ts` if stateless
3. Extend `useBingoGame` hook for state management (with localStorage persistence if needed)
4. Create/update UI components (keep them stateless)
5. Add tests to `src/utils/bingoLogic.test.ts` for logic changes

### Debugging State Issues
- Check localStorage in DevTools → Application → Local Storage → `bingo-game-state`
- Clear storage if `STORAGE_VERSION` mismatch
- SSR guard failures check for missing `typeof window === 'undefined'` guards

### Breaking Changes
If changing stored state shape:
1. Increment `STORAGE_VERSION` in `useBingoGame.ts`
2. Update `validateStoredData` function
3. Consider migration logic for existing users
