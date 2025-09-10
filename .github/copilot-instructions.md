# Copilot Instructions for SBPP Discord Plugin

## Repository Overview

This repository contains a SourcePawn plugin that integrates SourceBans++ (SBPP) with Discord webhooks. The plugin automatically sends notifications to Discord channels when punishments (bans, mutes, gags) are issued through SourceBans++.

**Key Information:**
- **Language**: SourcePawn
- **Platform**: SourceMod 1.11+ (see sourceknight.yaml for exact version)
- **Build System**: SourceKnight (modern SourcePawn build system)
- **Main Plugin**: `addons/sourcemod/scripting/sbpp_discord.sp`
- **Purpose**: Bridge between SourceBans++ punishment system and Discord notifications

## Project Structure

```
/
├── .github/
│   └── workflows/ci.yml          # GitHub Actions CI/CD
├── addons/sourcemod/scripting/
│   └── sbpp_discord.sp            # Main plugin source
└── sourceknight.yaml             # Build configuration
```

## Dependencies and Architecture

### Core Dependencies (defined in sourceknight.yaml):
- **SourceMod 1.11+**: Base scripting platform
- **RelayHelper**: Discord webhook functionality (required)
- **SourceBans++**: Punishment system integration (optional)
- **SourceBansChecker**: Additional ban checking features (optional)
- **SourceComms**: Communication punishment system (optional)
- **DiscordWebhookAPI**: Discord integration library

### Plugin Architecture:
The plugin uses optional includes (`#tryinclude`) for SBPP components, making it flexible:
- Works with or without SourceBans++ installed
- Supports multiple punishment systems (bans, comms)
- Leverages RelayHelper for Discord communication
- Uses ConVars for configuration

## SourcePawn Coding Standards for This Project

### Style Guidelines:
- **Indentation**: Tabs (4 spaces equivalent)
- **Variables**: camelCase for locals, PascalCase for functions, g_ prefix for globals
- **Required pragmas**: `#pragma semicolon 1` and `#pragma newdecls required`
- **Memory management**: Use `delete` without null checks, avoid `.Clear()` on StringMap/ArrayList

### Code Patterns Used:
```sourcepawn
// ConVar handling
g_Sbpp.enable = CreateConVar("sbpp_discord_enable", "1", "Description", _, true, 0.0, true, 1.0);

// Optional plugin integration
#if defined _sourcebanspp_included
public void SBPP_OnBanPlayer(int admin, int target, int length, const char[] reason) {
    // Implementation
}
#endif

// Client validation
if(IsFakeClient(client) || IsClientSourceTV(client)) {
    return;
}
```

## Build System - SourceKnight

### Building the Plugin:
The project uses SourceKnight instead of traditional spcomp compilation:

```bash
# Build using SourceKnight action (in CI)
uses: maxime1907/action-sourceknight@v1
with:
  cmd: build
```

### Key Configuration (sourceknight.yaml):
- **Output**: `/addons/sourcemod/plugins`
- **Target**: `sbpp_discord`
- **Dependencies**: Automatically fetched from GitHub repos and archives
- **Root**: `/` (project root)

### Local Development:
If developing locally, ensure SourceKnight is installed and dependencies are resolved before building.

## Testing and Validation

### Testing Approach for SourcePawn Plugins:
Since SourcePawn plugins don't have traditional unit tests, validation focuses on:

1. **Compilation**: Plugin must compile without errors/warnings
2. **Server Testing**: Load plugin on development server
3. **Integration Testing**: Verify Discord webhook functionality
4. **Event Testing**: Trigger SBPP events and verify Discord notifications

### CI/CD Process:
- **Build**: Compile plugin using SourceKnight
- **Package**: Create distributable archive
- **Release**: Automatic releases on tags and main branch pushes

## Discord Integration

### Webhook Configuration:
```sourcepawn
// ConVars for Discord integration
sbpp_discord_enable "1"                    // Enable/disable notifications
sbpp_discord ""                            // Discord webhook URL (protected)
sbpp_website "https://bans.example.com"    // SourceBans website URL
```

### Message Types Supported:
- Ban notifications (`SBPP_OnBanPlayer`)
- Mute/Unmute notifications (`SourceComms_OnBlockAdded`)
- Gag/Ungag notifications (`SourceComms_OnBlockAdded`)

## Common Development Tasks

### Adding New Punishment Types:
1. Check if the punishment system provides hooks
2. Add optional include for the system
3. Implement the callback function with proper guards
4. Use existing `SendDiscordMessage` function with appropriate `MessageType`

### Modifying Discord Messages:
- Discord message formatting is handled by RelayHelper
- Check RelayHelper documentation for message customization
- Avatar fetching is implemented for enhanced notifications

### Configuration Changes:
- Add new ConVars in `OnPluginStart()`
- Use `AutoExecConfig(true, PLUGIN_NAME)` for auto-generation
- Follow FCVAR_PROTECTED for sensitive data like webhooks

## Dependencies Management

### Adding New Dependencies:
1. Update `sourceknight.yaml` dependencies section
2. Add appropriate `#tryinclude` or `#include` in plugin
3. Test with and without the dependency present
4. Update CI if dependency affects build process

### Optional vs Required Dependencies:
- **Required**: RelayHelper (Discord functionality)
- **Optional**: All SBPP-related includes (graceful degradation)
- Use `#if defined` guards around optional functionality

## Memory Management

### Best Practices Applied:
- Use `delete` directly without null checks
- Avoid StringMap/ArrayList `.Clear()` - use `delete` and recreate
- Proper cleanup in `OnClientDisconnect`
- Handle late loads in `OnPluginStart`

## Performance Considerations

### Optimizations Used:
- Early returns for invalid clients
- ConVar caching with BoolValue property
- Avatar caching per client
- Minimal string operations in frequently called functions

## Troubleshooting

### Common Issues:
1. **Plugin not loading**: Check SourceMod version compatibility
2. **Discord not working**: Verify RelayHelper is loaded and webhook URL is valid
3. **No notifications**: Check if SBPP plugins are loaded and `sbpp_discord_enable` is 1
4. **Build failures**: Ensure all dependencies in sourceknight.yaml are accessible

### Debug Steps:
1. Check SourceMod error logs
2. Verify ConVar values with `sm_cvar sbpp_discord_enable`
3. Test webhook URL independently
4. Confirm SBPP plugins are functioning

## Version Control

- **Semantic versioning**: Update version in plugin info
- **Tags**: CI automatically creates releases for tags
- **Main branch**: Latest development, auto-tagged as "latest"

## Integration Points

### With SourceBans++:
- Hooks into `SBPP_OnBanPlayer` for ban notifications
- Optional integration - works without SBPP installed

### With SourceComms:
- Hooks into `SourceComms_OnBlockAdded` for communication punishments
- Handles mute/gag/unmute/ungag events

### With Discord:
- Uses RelayHelper for webhook communication
- Supports rich embed formatting with avatars
- Configurable webhook URL and website links

This plugin serves as a bridge between game server punishment systems and Discord community management, providing real-time notifications for administrative actions.