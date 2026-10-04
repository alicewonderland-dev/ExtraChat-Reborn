# Patch notes

This is a fork of ExtraChat by Anna, originally hosted at
https://git.sharlayan.cloud/anna/ExtraChat. All credit for the plugin goes to
the original author. The upstream source has no license file, so no license is
granted here beyond whatever the original author intended.

## Fix: "Not on main thread!" when registering / loading channels

On current Dalamud, reading `GameObject.Name` off the framework thread throws
`InvalidOperationException: Not on main thread!`. ExtraChat read the local
player's name and world from background tasks, which caused:

- Registration hanging on "receiving challenge" (`Client.GetChallenge`)
- The channel list failing to populate after login (`Client.HandleList`)

The fix is a `PlayerSnapshot` record (name, home world, current world, region)
captured in `Plugin.FrameworkUpdate` on the main thread. `Plugin.LocalPlayer`
now returns that snapshot, and every off-thread reader uses it instead of the
live `IPlayerCharacter`.

Files changed: `client/ExtraChat/Plugin.cs`, `Client.cs`, `Ui/PluginUi.cs`,
`Ui/ChannelList.cs`.

## 1.3.12: own internal name ("ExtraChatReborn")

Earlier builds kept the original's internal name, `ExtraChat`. Dalamud matches
plugins, their install state, hidden-plugin list and settings file by internal
name, so this fork collided with the original ExtraChat from the main
repository. It is now `ExtraChatReborn` (assembly, manifest and settings file),
and shows as "ExtraChat Reborn".

- **Settings:** on the first run under the new name, settings are copied from
  `pluginConfigs/ExtraChat.json` (keys, channels, order, colours) into
  `pluginConfigs/ExtraChatReborn.json`. The original file isn't changed, so the
  original plugin still works if switched back on.
- **Run only one:** both plugins use the same `/extrachat`, `/ec`, `/eclcmd`
  and `/ecl1`… commands, the same ChatTwo IPC names (kept so ChatTwo keeps
  working), and, after the copy, the same account. Disable the original while
  using this one.

Files changed: `client/ExtraChat/ExtraChat.csproj`, `Plugin.cs`,
`ExtraChat.yaml` → `ExtraChatReborn.yaml`.

## Building

Build with `dotnet build -c Release` in `client/`. The installable package is
`client/ExtraChat/bin/Release/ExtraChatReborn/latest.zip`; clear `bin/` first if
an older `ExtraChat` build is in it, or the package picks up its files.
