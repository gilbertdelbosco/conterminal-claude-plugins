# Conterminal Quoting

Drayage quoting for Claude, built on the Conterminal connector. Paste a customer's quote request and Claude prices it with your carrier's own Conterminal rate engine, shows the route economics, saves the price you agree, and prepares the rate card and reply for you to review. It also fills in a customer's multi-lane bid sheet, compares a quote with your saved rates, and reads or changes your default charges. Nothing is ever sent to a customer.

## What you need

- The Conterminal connector, connected to your carrier workspace. Add it from Claude's connector directory when it's listed there, or as a custom connector with the steps and OAuth client ID at <https://www.conterminal.com/connectors/shipments>. This plugin has no connector of its own.
- A Manager or Admin role in that workspace for quoting. Changing default charges needs Admin.

## Install

- **From Claude's plugin directory:** add Conterminal Quoting under Customize > Plugins once it's listed.
- **From GitHub:** in Customize > Plugins > Add > Add marketplace, enter `https://github.com/gilbertdelbosco/conterminal-claude-plugins`. A Team or Enterprise Owner can instead use Organization settings > Plugins & skills > Add > Sync from GitHub and set the plugin's Default access. The connector is added separately, under Organization settings > Connectors.
- **As a zip:** from the folder that contains `conterminal-quoting`, run `zip -r conterminal-quoting.zip conterminal-quoting -x '*.DS_Store'` and upload it under Customize > Plugins > Add > Upload plugin, or Organization settings > Plugins & skills > Add > Upload a plugin.
- **Claude Code:** sessions pick up the plugins and connectors on your Claude account. Or run `/plugin marketplace add gilbertdelbosco/conterminal-claude-plugins`, then `/plugin install conterminal-quoting@conterminal`.

## Use it

| Ask Claude | Skill | Command in Claude Code and Cowork |
| --- | --- | --- |
| "Quote this", with a pasted email or request | quote-request | `/conterminal-quoting:quote` |
| Price a customer's bid or rate sheet | quote-bid | `/conterminal-quoting:bid` |
| "What did we charge on this lane before?" | quote-compare | |
| "What are my quote settings?" or change a default charge | rate-engine | `/conterminal-quoting:rates` |

In claude.ai chat the commands load as skills, so asking in your own words works the same way.

## What it sends

The plugin is instructions only: it stores nothing, runs no code and calls no service itself. When you use it, Claude sends the request text, lanes, prices and terms you work on to your Conterminal workspace through the Conterminal connector. Conterminal saves quotes, drafts and rate cards in that workspace under its [privacy policy](https://www.conterminal.com/privacy) and [terms](https://www.conterminal.com/terms). No tool sends email to a customer; you review and send replies yourself.

## License

MIT. See [LICENSE](LICENSE).

Support: <support@conterminal.com>
