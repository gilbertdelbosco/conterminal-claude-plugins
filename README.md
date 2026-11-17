# Conterminal plugins for Claude

Claude plugins for [Conterminal](https://www.conterminal.com), the drayage TMS. This repository is a Claude plugin marketplace with one plugin:

- [Conterminal Quoting](conterminal-quoting/README.md): price a pasted quote request, fill in a customer's multi-lane bid sheet, compare a quote with saved rates, and review or change quote settings, using the Conterminal connector. Nothing is sent to a customer.

The plugins work with the Conterminal connector, which each person connects to their own Conterminal carrier workspace. Setup steps are at <https://www.conterminal.com/connectors/shipments>.

## Add this marketplace

- **claude.ai, one account:** Customize > Plugins > Add > Add marketplace, and enter `https://github.com/gilbertdelbosco/conterminal-claude-plugins`.
- **Team or Enterprise organization:** an Owner opens Organization settings > Plugins & skills > Add > Sync from GitHub with the same repository, then sets each plugin's Default access.
- **Claude Code:** run `/plugin marketplace add gilbertdelbosco/conterminal-claude-plugins`, then `/plugin install conterminal-quoting@conterminal`.

Conterminal maintains these files in its own repository and copies each reviewed release here. Report problems to <support@conterminal.com>.
