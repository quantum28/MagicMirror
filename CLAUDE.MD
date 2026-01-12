# MagicMirror² - Claude AI Context

## Project Overview

MagicMirror² is an open source modular smart mirror platform built with Electron. It allows users to convert mirrors into personal assistants by displaying customizable information modules.

- **Version**: 2.34.0
- **License**: MIT
- **Homepage**: https://magicmirror.builders
- **Documentation**: https://docs.magicmirror.builders
- **Repository**: https://github.com/MagicMirrorOrg/MagicMirror

## Technology Stack

- **Runtime**: Node.js (>=22.21.1 <23 || >=24)
- **Application Wrapper**: Electron ^39.2.7
- **Backend**: Express.js 5.2.1
- **Real-time Communication**: Socket.io ^4.8.3
- **Templating**: Nunjucks ^3.2.4
- **Date/Time**: Moment.js with timezone support
- **CSS Framework**: Animate.css
- **Icons**: Font Awesome
- **Testing**: Vitest, Playwright (E2E)
- **Linting**: ESLint, Stylelint, Prettier, markdownlint

## Project Structure

```
MagicMirror/
├── .github/              # GitHub Actions workflows and templates
├── .husky/               # Git hooks (pre-commit, etc.)
├── clientonly/           # Client-only scripts
├── config/               # Configuration files
├── css/                  # Global stylesheets
├── js/                   # Core JavaScript files
│   ├── electron.js       # Main Electron entry point
│   ├── module_functions.js
│   ├── server_functions.js
│   └── check_config.js
├── modules/              # Module system
│   └── default/          # Built-in default modules
│       ├── alert/
│       ├── calendar/
│       ├── clock/
│       ├── compliments/
│       ├── helloworld/
│       ├── newsfeed/
│       ├── updatenotification/
│       └── weather/
├── serveronly/           # Server-only scripts
├── tests/                # Test suite
│   ├── e2e/              # End-to-end tests
│   ├── electron/         # Electron tests
│   ├── unit/             # Unit tests
│   ├── utils/            # Test utilities
│   ├── mocks/            # Mock data
│   └── configs/          # Test configurations
├── translations/         # Internationalization files
├── index.html            # Main HTML file
├── package.json          # Dependencies and scripts
└── vitest.config.mjs     # Test configuration
```

## Key Files

- **js/electron.js**: Main Electron application entry point
- **index.html**: Main HTML template
- **config/config.js**: User configuration (not in repo, created by user)
- **package.json**: Project dependencies and npm scripts
- **eslint.config.mjs**: ESLint configuration
- **vitest.config.mjs**: Vitest test configuration
- **stylelint.config.mjs**: Stylelint configuration

## Module System

MagicMirror² uses a modular plugin system. Modules are self-contained components that display information.

### Default Modules

- **alert**: Display notifications
- **calendar**: Show calendar events (iCal support)
- **clock**: Display time and date
- **compliments**: Show random compliments
- **helloworld**: Sample module for developers
- **newsfeed**: RSS feed reader
- **updatenotification**: Notify when updates available
- **weather**: Display weather information

### Module Structure

Each module typically contains:
- `modulename.js`: Main module code
- `modulename.css`: Module-specific styles
- `node_helper.js`: Backend/server-side code (optional)

## Development Workflow

### Installation

```bash
npm install --no-audit --no-fund --no-update-notifier
```

For development with testing tools:
```bash
npm run install-mm:dev
```

### Running the Application

```bash
# Standard start (X11)
npm start

# Development mode
npm run start:dev

# Wayland
npm run start:wayland

# Windows
npm run start:windows

# Server only (no GUI)
npm run server
```

### Testing

```bash
# Run all tests
npm test

# Run specific test suites
npm run test:unit           # Unit tests
npm run test:e2e            # End-to-end tests
npm run test:electron       # Electron tests

# Code coverage
npm run test:coverage

# Watch mode
npm run test:watch

# UI test runner
npm run test:ui

# Specific module debugging
npm run test:calendar
```

### Linting and Formatting

```bash
# JavaScript
npm run lint:js             # Fix JS issues
npm run test:js             # Check only

# CSS
npm run lint:css            # Fix CSS issues
npm run test:css            # Check only

# Markdown
npm run lint:markdown       # Fix markdown issues
npm run test:markdown       # Check only

# Prettier (all files)
npm run lint:prettier       # Fix formatting
npm run test:prettier       # Check only

# Spell checking
npm run test:spelling
```

### Git Hooks

The project uses Husky and lint-staged:
- **Pre-commit**: Runs Prettier, ESLint, and Stylelint on staged files
- Configured in `.husky/` directory
- lint-staged config in `package.json`

## Configuration

User configuration is stored in `config/config.js` (not tracked in git). It defines:
- Port and address settings
- Language and timezone
- Module configurations
- Custom CSS

## Common Development Tasks

### Adding a New Module

1. Create directory in `modules/default/` or custom location
2. Implement `modulename.js` with Module.register()
3. Add optional `modulename.css` for styles
4. Add optional `node_helper.js` for backend logic
5. Test the module
6. Document in module README

### Making Core Changes

1. Understand the affected files in `js/` directory
2. Write or update tests in `tests/`
3. Ensure all linters pass
4. Test in both development and production modes

### Debugging

- Use `npm run start:dev` for development mode
- Check browser console (Electron DevTools)
- Check server logs in terminal
- Use `console-stamp` for enhanced logging

## Code Style

- **JavaScript**: ESLint with @stylistic/eslint-plugin
- **CSS**: Stylelint with standard config
- **Formatting**: Prettier
- **Line Endings**: LF (enforced by .editorconfig)
- **Indentation**: Tabs for JS/CSS, varies by file type

## Testing Philosophy

- Unit tests for isolated functionality
- E2E tests for user workflows
- Electron tests for app-specific behavior
- Vitest for test runner
- Playwright for browser automation

## Dependencies Management

- Avoid unnecessary dependencies
- Use `--no-audit --no-fund --no-update-notifier` flags
- Production vs dev dependencies clearly separated
- Regular dependency updates before releases

## Release Process

Managed by core collaborators (@rejas, @sdetweil, @khassel, @KristjanESPERANTO):
1. Update dependencies
2. Create prep-release branch from develop
3. Update version in package.json
4. Merge to develop, then to master
5. Tag release
6. Publish to npm

## Important Notes

- **Branch Strategy**: develop → master (releases only)
- **Node Version**: Requires Node 22.21.1+ or 24+
- **Electron**: Optional dependency (can run server-only)
- **Module Type**: CommonJS (not ESM)
- **Security**: Uses Helmet for HTTP headers, ipaddr.js for IP handling
- **Real-time**: Socket.io for server-client communication

## Contributing

See [Collaboration.md](Collaboration.md) for collaborator guidelines.
Full contribution guidelines: https://docs.magicmirror.builders/about/contributing.html

## Resources

- **Forum**: https://forum.magicmirror.builders
- **Discord**: https://discord.gg/J5BAtvx
- **Blog**: https://michaelteeuw.nl/tagged/magicmirror
- **Technical Discussions**: https://forum.magicmirror.builders/category/11/core-system

## Common Issues

- Ensure Node version compatibility (22.21.1+ or 24+)
- Run `npm install` after pulling updates
- Check config.js syntax for configuration errors
- Verify module paths are correct
- Clear browser cache if styles not updating
