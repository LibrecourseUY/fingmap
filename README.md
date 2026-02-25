# fingmap - Interactive Map of Fing
[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/Fingdev/fingmap)

Welcome to fingmap! This project is an interactive map of the Faculty of Engineering.

## What is this project?

fingmap is a web application that allows students to explore the Faculty of Engineering campus interactively. You can:
- View the location of different classrooms and laboratories
- Search for rooms by number or name
- View information about each location

It's designed to help incoming students navigate the campus.

## Tech Stack

- **Frontend**: Vue.js 3
- **Build tool**: Vite
- **Maps**: Leaflet (open source map library)

## How to Install and Run

### Prerequisites

- Node.js (version 18 or higher)
- npm (comes with Node.js)

### Local Development Setup

1. Enter the project folder:
   ```bash
   cd fingmap
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```

4. Open your browser at: `http://localhost:5173`

### For Production

```bash
npm run build
```

This generates a `dist/` folder with optimized files.

### With Docker

```bash
docker build -t fingmap .
docker run -p 80:80 fingmap
```

## Project Structure

```
fingmap/
├── src/              # Source code
│   ├── components/   # Vue components
│   ├── assets/       # Images and styles
│   └── App.vue       # Main component
├── public/           # Public files
├── index.html        # Main HTML file
├── vite.config.js    # Vite configuration
└── package.json      # Node dependencies
```

## How to Contribute

Want to add new locations to the map or improve the interface?

Check out our contribution guide in [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT License - see [LICENSE](LICENSE)

---

Never used Vue.js before? Don't worry, it's a very accessible framework. You can learn the basics in the [official Vue documentation](https://vuejs.org/).
