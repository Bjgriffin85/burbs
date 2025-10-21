# Burbs - Property Search Application

A Vue.js application for browsing and analyzing property data from Australian suburbs. View houses with detailed information including pricing, land size, and price per square meter calculations.

## Features

- 🏘️ Browse properties by suburb (Belmont North by default)
- 💰 View asking prices with formatted display
- 📏 See land sizes and calculate price per square meter
- 🔄 Sort properties by:
  - Price (High to Low / Low to High)
  - Land Size (High to Low / Low to High)
  - Price per m² (High to Low / Low to High)
- 🏠 Filter to show houses only (excludes units)

## Tech Stack

- **Vue 3** - Progressive JavaScript framework
- **Vue CLI** - Standard tooling for Vue.js development
- **Composition API** - Modern Vue 3 feature for component logic

## Project Setup

### Install dependencies
```bash
npm install
```

### Run development server
```bash
npm run serve
```

The app will be available at `http://localhost:8080`

### Build for production
```bash
npm run build
```

### Lint and fix files
```bash
npm run lint
```

## API Configuration

The app uses a proxy to connect to the Microburbs API. The proxy configuration is in `vue.config.js`:

- Development endpoint: `/api/suburb/properties`
- Production endpoint: `https://www.microburbs.com.au/report_generator/api`

**Note:** You'll need a valid API token to fetch live data. Update the `Authorization` header in `src/App.vue`.

## Customize Configuration

See [Vue CLI Configuration Reference](https://cli.vuejs.org/config/).
