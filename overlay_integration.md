# Overlay Integration Guide

Welcome to the Overlay Game Engine integration guide. This document provides comprehensive instructions for integrating overlay systems into your game projects.

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Getting Started](#getting-started)
4. [Integration Steps](#integration-steps)
5. [API Reference](#api-reference)
6. [Configuration](#configuration)
7. [Code Examples](#code-examples)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

## Overview

The Overlay Game Engine provides a flexible and powerful system for managing game overlays, including:

- **HUD Elements**: Health bars, score displays, mini-maps
- **Menu Systems**: Pause menus, settings, inventory screens
- **Dialog Boxes**: Interactive conversations and notifications
- **Debug Overlays**: Performance metrics, collision boxes, debug information

## Architecture

### Core Components

The overlay system is built on three main components:

```
┌─────────────────────────────────────┐
│      Overlay Manager                │
│  - Manages overlay lifecycle        │
│  - Handles rendering order          │
│  - Coordinates input events         │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │             │
┌───────▼───────┐ ┌──▼──────────────┐
│ Overlay Layer │ │ Input Handler   │
│ - Z-ordering  │ │ - Event routing │
│ - Visibility  │ │ - Focus mgmt    │
└───────────────┘ └─────────────────┘
```

### Overlay Lifecycle

1. **Initialize**: Set up overlay resources and configuration
2. **Activate**: Add overlay to the rendering pipeline
3. **Update**: Process logic and input events
4. **Render**: Draw overlay elements to the screen
5. **Deactivate**: Remove overlay from active state
6. **Cleanup**: Release resources

## Getting Started

### Prerequisites

- Overlay Game Engine v1.0 or higher
- Compatible graphics API (OpenGL 3.3+, DirectX 11+, Vulkan 1.0+)
- C++17 or higher

### Basic Setup

```cpp
#include <overlay_engine/overlay_manager.h>
#include <overlay_engine/overlay.h>

int main() {
    // Initialize the overlay manager
    OverlayEngine::OverlayManager manager;
    manager.initialize();
    
    // Your game loop here
    while (running) {
        manager.update(deltaTime);
        manager.render();
    }
    
    manager.shutdown();
    return 0;
}
```

## Integration Steps

### Step 1: Include the Overlay Engine

Add the overlay engine headers to your project:

```cpp
#include <overlay_engine/overlay_manager.h>
#include <overlay_engine/overlay.h>
#include <overlay_engine/ui_elements.h>
```

### Step 2: Create Your First Overlay

```cpp
class MyHUDOverlay : public OverlayEngine::Overlay {
public:
    void onInitialize() override {
        // Load textures, fonts, and other resources
        healthBar = createUIElement<ProgressBar>("health_bar");
        scoreText = createUIElement<TextLabel>("score_text");
    }
    
    void onUpdate(float deltaTime) override {
        // Update overlay state
        healthBar->setValue(player->getHealth());
        scoreText->setText("Score: " + std::to_string(gameState->getScore()));
    }
    
    void onRender() override {
        // Render UI elements
        healthBar->render();
        scoreText->render();
    }
    
private:
    std::shared_ptr<ProgressBar> healthBar;
    std::shared_ptr<TextLabel> scoreText;
};
```

### Step 3: Register and Activate

```cpp
// Create overlay instance
auto hudOverlay = std::make_shared<MyHUDOverlay>();

// Register with manager
manager.registerOverlay("hud", hudOverlay);

// Activate the overlay
manager.activateOverlay("hud");
```

### Step 4: Handle Input

```cpp
class MenuOverlay : public OverlayEngine::Overlay {
public:
    bool onInputEvent(const InputEvent& event) override {
        if (event.type == InputEventType::KeyPress) {
            if (event.key == Key::Escape) {
                // Handle escape key
                manager->deactivateOverlay("menu");
                return true; // Event consumed
            }
        }
        return false; // Event not handled
    }
};
```

## API Reference

### OverlayManager

**Methods:**

- `void initialize()` - Initialize the overlay system
- `void shutdown()` - Clean up and release resources
- `void update(float deltaTime)` - Update all active overlays
- `void render()` - Render all active overlays
- `void registerOverlay(const std::string& name, std::shared_ptr<Overlay> overlay)` - Register an overlay
- `void activateOverlay(const std::string& name)` - Activate an overlay
- `void deactivateOverlay(const std::string& name)` - Deactivate an overlay
- `void setOverlayZOrder(const std::string& name, int zOrder)` - Set rendering order

### Overlay Base Class

**Virtual Methods:**

- `void onInitialize()` - Called when overlay is first created
- `void onActivate()` - Called when overlay becomes active
- `void onDeactivate()` - Called when overlay is deactivated
- `void onUpdate(float deltaTime)` - Called every frame while active
- `void onRender()` - Called to draw the overlay
- `bool onInputEvent(const InputEvent& event)` - Handle input events
- `void onResize(int width, int height)` - Handle window resize events

## Configuration

### Overlay Configuration File

Create a `overlay_config.json` file to configure default settings:

```json
{
    "overlays": {
        "hud": {
            "enabled": true,
            "zOrder": 10,
            "inputPriority": 1
        },
        "menu": {
            "enabled": true,
            "zOrder": 100,
            "inputPriority": 100,
            "pauseGame": true
        },
        "debug": {
            "enabled": false,
            "zOrder": 1000,
            "inputPriority": 0
        }
    },
    "rendering": {
        "blendMode": "alpha",
        "defaultFont": "assets/fonts/roboto.ttf",
        "defaultFontSize": 16
    }
}
```

### Loading Configuration

```cpp
manager.loadConfiguration("overlay_config.json");
```

## Code Examples

### Example 1: Health Bar Overlay

```cpp
class HealthBarOverlay : public OverlayEngine::Overlay {
public:
    void onInitialize() override {
        healthBar = createUIElement<ProgressBar>("health_bar");
        healthBar->setPosition(10, 10);
        healthBar->setSize(200, 30);
        healthBar->setColor(255, 0, 0); // Red
        healthBar->setMaxValue(100);
    }
    
    void onUpdate(float deltaTime) override {
        float currentHealth = playerController->getHealth();
        healthBar->setValue(currentHealth);
        
        // Change color based on health level
        if (currentHealth < 30) {
            healthBar->setColor(255, 0, 0); // Red (critical)
        } else if (currentHealth < 60) {
            healthBar->setColor(255, 165, 0); // Orange (warning)
        } else {
            healthBar->setColor(0, 255, 0); // Green (healthy)
        }
    }
    
    void onRender() override {
        healthBar->render();
    }
    
private:
    std::shared_ptr<ProgressBar> healthBar;
    PlayerController* playerController;
};
```

### Example 2: Pause Menu Overlay

```cpp
class PauseMenuOverlay : public OverlayEngine::Overlay {
public:
    void onInitialize() override {
        // Create menu background
        background = createUIElement<Panel>("menu_background");
        background->setPosition(0, 0);
        background->setSize(screenWidth, screenHeight);
        background->setColor(0, 0, 0, 180); // Semi-transparent black
        
        // Create menu buttons
        resumeButton = createUIElement<Button>("resume");
        resumeButton->setText("Resume");
        resumeButton->setPosition(screenWidth/2 - 100, screenHeight/2 - 60);
        resumeButton->setSize(200, 50);
        resumeButton->onClick([this]() {
            manager->deactivateOverlay("pause_menu");
            gameState->resume();
        });
        
        settingsButton = createUIElement<Button>("settings");
        settingsButton->setText("Settings");
        settingsButton->setPosition(screenWidth/2 - 100, screenHeight/2);
        settingsButton->setSize(200, 50);
        settingsButton->onClick([this]() {
            manager->activateOverlay("settings_menu");
        });
        
        quitButton = createUIElement<Button>("quit");
        quitButton->setText("Quit to Menu");
        quitButton->setPosition(screenWidth/2 - 100, screenHeight/2 + 60);
        quitButton->setSize(200, 50);
        quitButton->onClick([this]() {
            gameState->quitToMainMenu();
        });
    }
    
    void onActivate() override {
        gameState->pause();
    }
    
    void onDeactivate() override {
        gameState->resume();
    }
    
    bool onInputEvent(const InputEvent& event) override {
        if (event.type == InputEventType::KeyPress && event.key == Key::Escape) {
            manager->deactivateOverlay("pause_menu");
            return true;
        }
        return false;
    }
    
    void onRender() override {
        background->render();
        resumeButton->render();
        settingsButton->render();
        quitButton->render();
    }
    
private:
    std::shared_ptr<Panel> background;
    std::shared_ptr<Button> resumeButton;
    std::shared_ptr<Button> settingsButton;
    std::shared_ptr<Button> quitButton;
};
```

### Example 3: Debug Overlay

```cpp
class DebugOverlay : public OverlayEngine::Overlay {
public:
    void onInitialize() override {
        fpsText = createUIElement<TextLabel>("fps");
        fpsText->setPosition(10, screenHeight - 30);
        fpsText->setColor(0, 255, 0);
        
        memoryText = createUIElement<TextLabel>("memory");
        memoryText->setPosition(10, screenHeight - 50);
        memoryText->setColor(0, 255, 0);
    }
    
    void onUpdate(float deltaTime) override {
        frameCount++;
        timeAccumulator += deltaTime;
        
        if (timeAccumulator >= 1.0f) {
            currentFPS = frameCount / timeAccumulator;
            frameCount = 0;
            timeAccumulator = 0.0f;
        }
        
        fpsText->setText("FPS: " + std::to_string(static_cast<int>(currentFPS)));
        memoryText->setText("Memory: " + std::to_string(getMemoryUsage()) + " MB");
    }
    
    void onRender() override {
        fpsText->render();
        memoryText->render();
    }
    
    bool onInputEvent(const InputEvent& event) override {
        if (event.type == InputEventType::KeyPress && event.key == Key::F3) {
            setVisible(!isVisible());
            return true;
        }
        return false;
    }
    
private:
    std::shared_ptr<TextLabel> fpsText;
    std::shared_ptr<TextLabel> memoryText;
    float currentFPS = 0.0f;
    int frameCount = 0;
    float timeAccumulator = 0.0f;
};
```

## Best Practices

### 1. Z-Order Management

Organize overlays in logical layers:
- **0-99**: Game HUD elements
- **100-199**: In-game menus (inventory, character sheet)
- **200-299**: System menus (pause, settings)
- **300+**: Modal dialogs and critical notifications

```cpp
manager.setOverlayZOrder("hud", 10);
manager.setOverlayZOrder("inventory", 100);
manager.setOverlayZOrder("pause_menu", 200);
manager.setOverlayZOrder("confirmation_dialog", 300);
```

### 2. Input Event Propagation

Higher priority overlays should consume input events to prevent them from reaching lower layers:

```cpp
bool onInputEvent(const InputEvent& event) override {
    if (handleEvent(event)) {
        return true; // Consume event, don't propagate
    }
    return false; // Allow event to propagate to lower layers
}
```

### 3. Resource Management

Load resources during initialization, not during render:

```cpp
void onInitialize() override {
    // Good: Load during initialization
    texture = textureLoader->load("ui/health_bar.png");
}

void onRender() override {
    // Bad: Don't load during render
    // auto texture = textureLoader->load("ui/health_bar.png");
    
    // Good: Use pre-loaded resource
    renderer->drawTexture(texture, position);
}
```

### 4. Performance Optimization

- Use dirty flags to avoid unnecessary redraws
- Batch similar UI elements together
- Cache text rendering when possible
- Minimize state changes during rendering

```cpp
class OptimizedOverlay : public OverlayEngine::Overlay {
public:
    void onUpdate(float deltaTime) override {
        if (player->getHealth() != lastHealth) {
            lastHealth = player->getHealth();
            isDirty = true;
        }
    }
    
    void onRender() override {
        if (isDirty) {
            rebuildRenderCache();
            isDirty = false;
        }
        renderer->drawCached(renderCache);
    }
    
private:
    bool isDirty = true;
    float lastHealth = 0.0f;
};
```

### 5. Responsive Design

Make overlays adapt to different screen resolutions:

```cpp
void onResize(int width, int height) override {
    // Update positions relative to new screen size
    healthBar->setPosition(width * 0.1f, height * 0.05f);
    scoreText->setPosition(width * 0.9f, height * 0.05f);
}
```

## Troubleshooting

### Overlay Not Visible

**Problem:** Overlay is activated but not showing on screen.

**Solutions:**
- Check if overlay's `onRender()` method is implemented
- Verify z-order settings (higher values render on top)
- Ensure overlay visibility is set to true
- Check if UI elements have valid positions within screen bounds

```cpp
// Debug visibility
if (!overlay->isVisible()) {
    overlay->setVisible(true);
}
```

### Input Not Working

**Problem:** Overlay doesn't respond to keyboard or mouse input.

**Solutions:**
- Verify `onInputEvent()` returns true when handling events
- Check input priority settings
- Ensure overlay is active and focused
- Confirm input events aren't consumed by higher priority overlays

```cpp
// Check input priority
manager.setInputPriority("menu", 100); // Higher priority
```

### Performance Issues

**Problem:** Frame rate drops when overlay is active.

**Solutions:**
- Profile render calls to identify bottlenecks
- Use render batching for similar elements
- Implement dirty flag optimization
- Reduce texture size and use texture atlases
- Cache text rendering

```cpp
// Enable profiling
manager.enableProfiling(true);
auto stats = manager.getProfilingStats();
std::cout << "Overlay render time: " << stats.renderTime << "ms" << std::endl;
```

### Memory Leaks

**Problem:** Memory usage increases over time.

**Solutions:**
- Use smart pointers for UI elements
- Properly clean up resources in overlay destructor
- Avoid circular references in callbacks
- Release textures and fonts when overlay is deactivated

```cpp
void onDeactivate() override {
    // Clean up resources
    texture.reset();
    uiElements.clear();
}
```

### Z-Order Conflicts

**Problem:** Overlays render in wrong order.

**Solutions:**
- Review z-order values for all overlays
- Use consistent z-order ranges for different overlay types
- Call `setOverlayZOrder()` after registration
- Debug with overlay visualization tool

```cpp
// Visualize overlay stack
manager.debugPrintOverlayStack();
```

## Additional Resources

- [API Documentation](docs/api/overlay_engine.md)
- [Tutorial Videos](tutorials/overlay_integration)
- [Sample Projects](examples/)
- [Community Forum](https://forum.overlayengine.com)
- [GitHub Issues](https://github.com/tap919/Overlay-Game-Engine/issues)

## Support

For additional help:
- Create an issue on [GitHub](https://github.com/tap919/Overlay-Game-Engine/issues)
- Join our [Discord community](https://discord.gg/overlayengine)
- Email support: support@overlayengine.com

---

**Last Updated:** 2025-10-13  
**Version:** 1.0  
**License:** MIT
