# MagicMirror² AI Coding Guidelines

## Project Overview
MagicMirror² is a modular smart mirror platform built on Electron. It uses a client-server architecture where the Express server runs Node.js code and communicates with frontend modules via Socket.io. Each module renders content to a specific region on the mirror display.

## Architecture Fundamentals

### Client-Server Communication
- **Server**: [js/server.js](js/server.js) - Express app with Socket.io namespace per module
- **Client**: [js/main.js](js/main.js) - Frontend orchestrator that manages module lifecycle
- **Socket Bridge**: [js/socketclient.js](js/socketclient.js) - Client-side Socket.io wrapper; [js/node_helper.js](js/node_helper.js) - Server-side Node.js module backend

**Data Flow**: Modules use `sendSocketNotification()` (client) ↔ `socketNotificationReceived()` (server) for bidirectional communication. Notifications use Socket.io namespaces (`/moduleName`).

### Module System
Modules extend the base `Module` class via `Module.register(name, definition)`. Each module:
- Implements lifecycle methods: `init()`, `start()`, `getDom()`, `notificationReceived()`
- Returns DOM via `getDom()` (or async Promise)
- Uses `updateDom()` to refresh display
- Declares styles via `getStyles()`, scripts via `getScripts()`, translations via `getTranslations()`

**Key file**: [js/module.js](js/module.js) - Base class; [module-types.ts](module-types.ts) - TypeScript definitions

### Notification Pattern
Inter-module communication uses a broadcast system (`sendNotification()`):
- Modules call `this.sendNotification(notification, payload)` to broadcast
- All other modules receive via `notificationReceived(notification, payload, sender)`
- Sender can target specific modules (see [js/main.js](js/main.js#L97))

**Example**: Clock module sends `CLOCK_SECOND` to notify other modules of time updates

## Configuration & Environment

### Config System
- **User config**: [config/config.js](config/config.js) - User-specific settings (copied from sample)
- **Defaults**: [js/defaults.js](js/defaults.js) - All configuration options with defaults
- **Validation**: `npm run config:check` validates config syntax before startup

### Environment Variables
- `MM_CONFIG_FILE` - Override default config path
- `MM_PORT` - Override port (default 8080)
- `DISPLAY` / `WAYLAND_DISPLAY` - Set display server on Linux
- `mmFetchTimeout` - Global fetch timeout (default 30s)

## Development Workflows

### Starting the Application
```bash
npm start              # X11 (default Linux)
npm run start:dev      # Dev mode with debug logging
npm run start:wayland  # Wayland display server
npm run server         # Backend-only (no Electron)
npm run server:watch   # Auto-reload on file changes
```

### Testing
- `npm test` - All tests (sequential, single worker for stability)
- `npm run test:unit` - Unit tests only
- `npm run test:e2e` - End-to-end (Playwright) browser tests
- `npm run test:electron` - Electron app tests
- `npm run test:watch` - Watch mode with UI
- `npm run test:coverage` - Coverage report

**Key config**: [vitest.config.mjs](vitest.config.mjs) - Tests run sequentially on shared port 8080

### Code Quality
- `npm run lint:js` - ESLint with auto-fix (tabs, import-x plugin)
- `npm run lint:css` - Stylelint with auto-fix
- `npm run lint:markdown` - Markdown linting
- `npm run lint:prettier` - Code formatting
- `npm run test:css` / `npm run test:js` - Lint without fix

## Key Code Patterns

### Module Definition Template
```javascript
Module.register("mymodule", {
  defaults: { /* config options */ },
  getScripts: () => ["lib.js"],
  getStyles: () => ["style.css"],
  getTranslations: () => ({ en: "en.json" }),
  
  start() {
    Log.info(`Starting ${this.name}`);
    this.updateDom();
  },
  
  getDom() {
    const wrapper = document.createElement("div");
    wrapper.innerHTML = "Content";
    return wrapper;
  },
  
  notificationReceived(notification, payload, sender) {
    if (notification === "MODULE_DOM_CREATED") {
      this.updateDom();
    }
  },
  
  socketNotificationReceived(notification, payload) {
    if (notification === "DATA_FETCHED") {
      this.data = payload;
      this.updateDom();
    }
  }
});
```

### Node Helper (Server-Side Module)
```javascript
const NodeHelper = require("node_helper");

module.exports = NodeHelper.create({
  start() {
    Log.log(`${this.name} helper started`);
  },
  
  socketNotificationReceived(notification, payload) {
    if (notification === "FETCH_DATA") {
      fetch("https://api.example.com")
        .then((r) => r.json())
        .then((data) => {
          this.sendSocketNotification("DATA_READY", data);
        })
        .catch(Log.error);
    }
  }
});
```

### DOM Update with Animation
```javascript
this.updateDom({
  options: {
    speed: 1000,
    animate: {
      out: "fadeOut",
      in: "fadeIn"
    }
  }
});
```

## Code Style & Conventions

- **Indentation**: Tabs (see [eslint.config.mjs](eslint.config.mjs#L40))
- **Semicolons**: Required
- **Naming**: camelCase for functions/vars, UpperCamelCase for classes
- **Logging**: Use `Log.info()`, `Log.log()`, `Log.warn()`, `Log.error()` (global)
- **Comments**: JSDoc format for public methods
- **Async**: Use `async/await` (supported in Node 22+)
- **CSS**: BEM-like naming; no vendor prefixes (browser support is modern)

## Directory Structure

- `js/` - Core backend code (server, modules, utils)
- `modules/default/` - Built-in modules (clock, weather, calendar, etc.)
- `css/` - Global styles
- `translations/` - i18n JSON files (40+ languages)
- `tests/` - Unit, e2e, and electron tests
- `config/` - Configuration samples

## Testing Patterns

- **Unit tests**: Mock external APIs, test pure functions
- **E2E tests**: Use Playwright, run against live server
- **Test setup**: [tests/utils/vitest-setup.js](tests/utils/vitest-setup.js)
- **Sequential execution**: All tests share port 8080; parallel would break this

Example unit test:
```javascript
describe("ModuleName", () => {
  it("should update DOM on notification", () => {
    // test implementation
  });
});
```

## Common Tasks

### Adding a New Module
1. Create folder: `modules/default/mymodule/`
2. Add `mymodule.js` with `Module.register("mymodule", {...})`
3. Add CSS/translations if needed
4. Register in [modules/default/defaultmodules.js](modules/default/defaultmodules.js)
5. Add config entry to [js/defaults.js](js/defaults.js)

### Fetching External Data
- Use `fetch()` with timeout handling
- Node helpers handle API calls to avoid CORS issues
- Send data to frontend via `sendSocketNotification()`
- See [modules/default/weather](modules/default/weather) for example

### Handling Errors
- Use `Log.error()` with descriptive messages
- In node_helpers: `catch(Log.error)` for promise chains
- Avoid uncaught rejections (the app won't crash, but logs them)

## Important Notes

- **Security**: IP whitelist enforced via [js/ip_access_control.js](js/ip_access_control.js)
- **Performance**: Use `updateDom()` sparingly; batches are better
- **Translations**: Always provide `en.json`; use [translations/](translations/) for keys
- **Fetch timeout**: Set global `mmFetchTimeout` env var if modules need longer
- **Module isolation**: Modules should not directly access global state beyond `config` and `Log`
