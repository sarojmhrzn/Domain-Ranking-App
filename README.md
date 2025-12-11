# Domain Ranking App - Frontend

A simple and modern Vue.js application developed in Vue 3 that allows users to fetch and visualize Tranco domain ranking trends.
The app displays historical rank data on interactive line charts using Chart.js and supports multiple domain comparisons.

This frontend connects to the companion NestJS backend that fetches rank data from the Tranco API and performs caching using PostgreSQL (Neon DB).

## Features

- Search Any Domain
  Enter any domain name (e.g., google.com, facebook.com) and get data fetched from Tranco Rank
- Compare Multiple Domains
  Users can enter comma-separated domains, and the UI will dynamically display multiple datasets on a single chart.
  ```bash
    google.com, facebook.com, apple.com
  ```
- Backend driven with Caching
  The frontend retrieves all data from your NestJS backend via API
  ```bash
  /ranking/:domain
  ```
- Loading UI Feedback
  Full-screen blurred overlay with spinner

## Tech Stack

- Vue 3 + Vite
- Chart.js
- Tailwind + Custom CSS

## Project Setup

Create a .env file in your project root:

```bash
VITE_API_BASE=https://your-backend-url.com
```

For Development

```bash
# development
$ npm install
$ npm run dev

# build
$ npm run build
```
