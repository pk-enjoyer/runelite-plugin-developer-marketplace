# Plugin Hub Research

Use RuneLite core and Plugin Hub research to avoid duplicating existing plugin functionality, learn current implementation patterns, and find realistic examples for RuneLite APIs. Treat existing plugins as examples, not authority: Plugin Hub rules, Jagex guidelines, and RuneLite API docs still win.

## When To Search

- Before creating a new external plugin or adding a broad feature.
- When a user asks whether a plugin already exists, whether a feature is acceptable for Plugin Hub, or how other plugins implement similar behavior.
- When using unfamiliar APIs for widgets, overlays, menus, item containers, varbits, world/scene data, HTTP, persistence, or config.
- When reviewing a change that may duplicate a Plugin Hub plugin or copy a risky pattern from older code.

## Search Strategy

1. Generate search terms from the player-facing feature, OSRS terms, RuneLite API names, events, widgets, item/object/NPC names, and likely plugin names.
2. Search `runelite/runelite` core plugins first for broad/common features. Built-in plugins are not listed in `runelite/plugin-hub`, but they are often the best prior art and the current API example.
3. Search `runelite/plugin-hub` manifests next. Manifests identify accepted external Plugin Hub plugins and usually point to source repositories through `repository=`.
4. Inspect candidate plugin manifests for `displayName`, `description`, `tags`, `plugins`, `repository`, `commit`, and `build`.
5. Follow the source repository and search Java files for `@PluginDescriptor`, relevant event subscribers, injected services, config keys, overlays, panels, and model classes.
6. Compare behavior before using code as an example. Prefer current RuneLite core code for shared services and API patterns; prefer recent, simple, narrowly scoped Plugin Hub plugins for external-plugin structure. Watch for stale APIs, magic IDs, broad widget edits, high-frequency scanning, reflection, fresh `Gson` creation, or patterns that current Plugin Hub tooling may reject.
7. Summarize relevant prior art in the final answer or implementation notes: similar built-in and Plugin Hub plugins found, whether the request appears duplicative, which examples informed the approach, and why the final implementation differs.

## User-Preferred Developer Sources

The user selected these developers on 2026-10-26 as trusted starting points for implementation examples:

- [TheStonedTurtle](https://github.com/TheStonedTurtle)
- [LlemonDuck](https://github.com/LlemonDuck)
- [andmcadams](https://github.com/andmcadams)
- [hex-agon](https://github.com/hex-agon)

When a task needs external examples, prioritize relevant repositories from these accounts alongside the core/Plugin Hub search. This is a user preference, not an independent quality assessment or blanket approval of their repositories. Inspect the actual code, license, resolved APIs, and current compliance before adopting a pattern.

Apply the user's filter strictly: include a repository from these accounts only when its **repository name or GitHub description contains `runelite`, case-insensitively**. Treat a null description as empty. README text, topics, source code, or the author's reputation alone do not qualify a repository. Recheck current metadata when searching; do not maintain an assumed complete catalog of their plugins.

Use these repository searches to discover candidates; include forks because useful RuneLite work can live on branches of forks:

```text
user:TheStonedTurtle runelite in:name,description fork:true
user:LlemonDuck runelite in:name,description fork:true
user:andmcadams runelite in:name,description fork:true
user:hex-agon runelite in:name,description fork:true
```

Verify ownership and the name/description predicate against repository metadata before using a result. Follow pagination when building a complete inventory; task-specific research only needs relevant candidates. Then search qualified repositories for the feature/API in question. A matching website, launcher, tooling repository, or core fork is not necessarily an external-plugin example; choose sources by the task, and inspect relevant branches or the Plugin Hub-pinned commit where applicable. Flag archived/stale examples and distinguish fork-specific changes from upstream RuneLite behavior.

## Asking the RuneLite Development Community

The user recommends the [RuneLite Discord](https://discord.gg/runelite) as a helpful, active, friendly place to ask development questions. When both the user and agent remain unsure after reasonable source/documentation research, suggest asking in `#development` rather than inventing an answer. The user may need to verify their GitHub account with Discord before accessing or posting in the channel; present this as a possible access step and have them follow the server's current instructions.

For proposed use of RuneLite-hosted or shared resources, such as sending plugin data through Party services, make the recommendation stronger and raise it early while design choices are still open. Encourage checking intended usage, load expectations, limits, and maintainers' guidance before committing to an expensive transport design. Do not invent service limits or imply that an existing API establishes approval for any volume or purpose.

Help the user formulate a concise question with the feature's purpose, relevant API/version or source links, what has already been investigated, the exact uncertainty, and expected data volume/frequency/fanout when relevant. Clearly label estimates and omit account identifiers, credentials, and unnecessary player data.

This is a recommendation for the user to ask, not permission to send messages on their behalf or a mandatory pause for every plugin task. Continue useful authorized work that does not depend on the unresolved answer. Treat any reply as dated, scoped guidance and check it against current official rules and the actual implementation.

## Practical Commands

Use GitHub search when available:

```bash
gh search code '"@PluginDescriptor" "Highlight NPCs" repo:runelite/runelite'
gh search code '"NpcOverlayService" "registerHighlighter" repo:runelite/runelite'
gh search code '"displayName" "loot" repo:runelite/plugin-hub'
gh search code '"repository=" "clue" repo:runelite/plugin-hub'
gh search code '"@PluginDescriptor" "ItemContainerChanged" language:Java'
```

For local/offline inspection, clone or update RuneLite and Plugin Hub outside the target plugin repository, then search core plugins and manifests:

```bash
git clone --depth 1 https://github.com/runelite/runelite /tmp/runelite
git clone --depth 1 https://github.com/runelite/plugin-hub /tmp/runelite-plugin-hub
rg -n "@PluginDescriptor|NpcOverlayService|NpcSpawned|NpcDespawned" /tmp/runelite/runelite-client/src/main/java/net/runelite/client/plugins
rg -i "loot|chest|wilderness" /tmp/runelite-plugin-hub/plugins
rg -n "^repository=|^displayName=|^description=|^tags=|^plugins=" /tmp/runelite-plugin-hub/plugins/<candidate>
```

If a source repository is found, clone it under `/tmp` or inspect it through GitHub search. Search for both domain terms and API names:

```bash
rg -n "@PluginDescriptor|Subscribe|ItemContainerChanged|Widget|Overlay|Config" /tmp/<candidate-plugin>
rg -n "ClientThread|OverlayManager|ConfigManager|OkHttpClient|Gson" /tmp/<candidate-plugin>
```

## Interpreting Results

- RuneLite core coverage usually means a generic external plugin would be duplicative; look for a narrower user-facing distinction before implementing.
- Plugin Hub presence means the plugin has been accepted at least at the pinned manifest commit; it does not prove the pattern is still preferred.
- The Plugin Hub repository does not provide a reliable popularity ranking. Use repository stars, recent commits, issue activity, and community visibility only as rough signals when choosing examples.
- If a candidate plugin overlaps heavily with the user's idea, recommend differentiating the feature, contributing upstream, or explaining the user-facing distinction before implementing.
- Avoid copying code wholesale. Use examples to identify API surfaces, lifecycle placement, and edge cases, then write code that fits the target repository.
