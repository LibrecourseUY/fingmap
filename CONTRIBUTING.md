# Contributing to fingmap

Thank you for your interest in contributing to fingmap! This guide will walk you through how to contribute.

## What is fingmap?

fingmap is an interactive map of the Faculty of Engineering. It helps students navigate the campus by finding classrooms, laboratories, and other spaces.

## Why Contribute?

- Learn Vue.js 3 (modern JavaScript framework)
- Work with interactive maps (Leaflet)
- Create a useful tool for incoming students
- Add projects to your portfolio

## Code of Conduct

Respect and kindness toward everyone. This is a community for learning together.

## Contribution Cycle

### Step 1: Fork

1. Go to: https://github.com/fingdev/fingmap
2. Click "Fork" (top right)
3. Select your GitHub account

### Step 2: Clone Your Fork

```bash
git clone https://github.com/YOUR_USERNAME/fingmap.git
cd fingmap
```

### Step 3: Create a New Branch

```bash
git checkout -b feature/add-new-location
# or
git checkout -b fix/fix-render-error
```

### Step 4: Install Dependencies

```bash
npm install
```

### Step 5: Make Your Changes

Edit the necessary files. If it's your first time with Vue.js, don't worry:
- Components are in `src/components/`
- Main code is in `src/App.vue`
- Styles are in `src/assets/`

### Step 6: Test Locally

```bash
npm run dev
```

This starts a development server. Open `http://localhost:5173` to see changes.

### Step 7: Commit

```bash
git add .
git commit -m "type: description of what you did"
```

#### Commit Types

| Type | When to use |
|------|-------------|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `docs:` | Documentation |
| `style:` | Formatting, CSS |
| `refactor:` | Rewrite code |
| `chore:` | Maintenance |

**Examples:**
```
feat: add marker for computer laboratory
fix: fix position of classroom 101 marker
```

### Step 8: Push and Create PR

```bash
git push origin feature/your-feature
```

Go to GitHub and create a Pull Request describing your changes.

## Requirements

- Node.js 18+
- npm (comes with Node)

## Useful Commands

```bash
# Development
npm run dev

# Production build
npm run build

# Preview build
npm run preview

# Check for code errors
npm run lint
```

## Project Structure

```
fingmap/
├── src/
│   ├── components/    # Vue components
│   │   └── Map.vue  # Map component
│   ├── assets/       # Images and styles
│   │   └── main.css
│   └── App.vue      # Main component
├── public/           # Public files
├── index.html        # Main HTML
├── vite.config.js   # Vite config
└── package.json     # Dependencies
```

## Adding a New Location to the Map

1. Find the file where markers are defined
2. Add a new object with:
   - Place name
   - Coordinates (latitude, longitude)
   - Description
3. Commit and create the PR

## Code Standards

- Components in English
- Props and events in camelCase
- Scoped CSS (when possible)
- Small and reusable components

## Signed Commits (Required)

All commits must be signed. This is a mandatory requirement for contributing.

### Why Signed Commits?

- **Security**: Verifies that you are who you claim to be
- **Integrity**: Guarantees the commit wasn't modified after creation
- **Trust**: Other contributors know changes genuinely come from you
- **Professionalism**: Industry best practice for open source projects

### Setup

Follow our Git Guide to set up your GPG key:
- [GPG Key Setup](https://fingdev.github.io/git)

Configure Git to sign commits by default:
```bash
git config --global commit.gpgsign true
```

## Contributor License Agreement (CLA)

Before contributing, you must read and accept our [Contributor License Agreement (CLA)](https://fingdev.github.io/CLA).

By submitting a Pull Request with cryptographically signed commits, you acknowledge and agree to the CLA terms. This includes:

- Licensing your contribution under MIT
- Confirming you have the rights to contribute the code
- Accepting the project's non-affiliation with FING/Udelar
- Understanding that all commits must be GPG-signed as acceptance of the CLA

Contributions without signed commits or that do not accept the CLA may be rejected.

## Questions?

Don't know how to do something? Create an Issue and ask. The community will help you.

---

Your first contribution can be as simple as fixing a text or adding a marker! It doesn't have to be something big to get started.
