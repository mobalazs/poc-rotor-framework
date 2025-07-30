# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Roku BrightScript application implementing the **Rotor Framework** - a custom reactive framework for Roku development that brings modern development patterns (similar to React/Redux) to the Roku platform. The framework consists of two main subsystems:

1. **View Builder** - Declarative UI rendering system running in the render thread
2. **Multi-Thread MVI** - State management pattern running across task threads

## Development Commands

### Build Commands
- `npm run build-dev` - Build development version with debug symbols
- `npm run build-tests` - Build test version
- `npm run release` - Build production release
- `npm run package` - Create deployment package
- `npm run watch` - Build in watch mode

### Code Quality
- `npm run lint` - Run BSLint static analysis
- `npm run precompiler` - Generate assets from JS files (runs automatically before builds)

### Testing
- Use VSCode Launch configurations: "Run Debug" or "Run Tests"
- Tests use Rooibos framework and are located in `**/*.spec.bs` files
- Test configuration in `bsconfig-tests.json`

## Architecture

### Thread Structure
- **Render Thread** (`src/components/app/renderThread/`): UI components and view logic
  - `MainScene.bs` - Entry point scene
  - `appRender/` - Main application renderer
  - `viewModels/` - UI component definitions using Rotor framework
- **Task Thread** (`src/components/app/taskThread/`): Business logic and state management
  - `appTask.bs` - Main task entry point, creates dispatchers for state management
  - Contains models and reducers for different app domains (user, content, etc.)

### Rotor Framework Core
Located in `src/source/libs/rotorFramework/`:
- `RotorFramework.bs` - Main framework for render thread
- `RotorFrameworkTask.bs` - Task thread framework
- `engine/` - Core rendering and state management engine
- `plugins/` - Extensible plugin system for UI behaviors
- `base/` - Base classes for models, reducers, and view components

### Key Patterns
- **Declarative UI**: Use `m.appFw.render()` with object definitions instead of imperative node creation
- **MVI State Management**: Models hold state, Reducers handle mutations, ViewModels map state to UI
- **Multi-thread Communication**: Dispatchers sync state between render and task threads
- **Plugin Architecture**: Extensible system for UI behaviors (focus, fields, styling, etc.)

## Asset Management

Assets are generated from JavaScript files in `assetsJs/` and compiled to BrightScript:
- `assetsJs/theme.js` → `src/assets/generated/bsTheme.bs`
- `assetsJs/translation.js` → `src/assets/generated/aaTranslation.brs`

## Configuration Files

- `bsconfig.json` - Main BrightScript compiler configuration
- `bsconfig-dev.json` - Development build configuration  
- `bsconfig-tests.json` - Test build configuration
- `bslint.json` - Static analysis rules
- `manifest` - Roku channel manifest

## Setup Requirements

1. Rename `.vscode/sample.settings.json` to `.vscode/settings.json`
2. Rename `sample.env` to `.env` with:
   - `ROKU_DEV_TARGET=<roku_device_ip>`
   - `ROKU_DEV_PASS=<device_password>`
3. Run `npm install`
4. Use VSCode debug configurations for deployment

## Framework Usage Examples

### Creating UI Components
```brightscript
m.appFw.render({
    nodeType: "LayoutGroup",
    fields: { translation: [90, 60] },
    children: [
        {
            nodeType: "Label",
            fields: { text: "Hello World" }
        }
    ]
})
```

### State Management
```brightscript
' In task thread - create dispatcher
model = new Models.UserModel()
reducer = new Reducers.UserReducer()
Rotor.createDispatcher("user", model, reducer)

' In render thread - listen to state
dispatcher = m.getDispatcher("user")
dispatcher.addListener({
    mapStateToProps: sub(props, state)
        props.username = state.username
    end sub
})
```
