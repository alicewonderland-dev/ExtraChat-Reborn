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

Build with `dotnet build -c Release` in `client/` and load the resulting DLL as
a Dalamud dev plugin. This build does not auto-update.
