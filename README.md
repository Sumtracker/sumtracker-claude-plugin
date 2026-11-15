# Sumtracker

[Sumtracker](https://www.sumtracker.com) is inventory management for e-commerce sellers on Shopify, Amazon, Walmart,
eBay and Etsy. This plugin connects Claude to your Sumtracker account, so you can check stock, review orders, and
manage warehouses, sales channels and suppliers from a conversation. It also adds skills that guide Claude through
common inventory tasks safely.

## What's included

- **Sumtracker connector.** Sumtracker's hosted MCP server at `https://mcp.sumtracker.com/mcp`. Claude uses it to read
  and change the data in your Sumtracker account.
- **Skills.**
  - `adjust-inventory`: adjusts the stock of one or more SKUs at a warehouse. It can set the stock to a counted
    number, add received units, or remove damaged or lost ones. Claude looks up each SKU, shows you the current and
    new stock, offers your account's adjustment reasons, and makes no change until you confirm. In Claude Code, run it
    with `/sumtracker:adjust-inventory`, or just ask, for example "We counted 40 of TSHIRT-RED-M at the Main
    warehouse" or "3 units of MUG-BLUE arrived damaged".

## Requirements

- A Sumtracker account with an active subscription or free trial.
- A Sumtracker user to connect with. Claude gets the same permissions and warehouse access as that user, so it can
  only read and change what the user can in Sumtracker.
- Each user can have 2 AI apps connected at a time. To connect another, disconnect one in Sumtracker, or contact
  support to raise the limit.

## Install

- **Claude (web and desktop):** open **Customize > Plugins > Discover**, find Sumtracker, and install it.
- **Claude Code:** run `/plugin`, open **Discover**, find Sumtracker, and install it.

The plugin uses the same server as the Sumtracker connector in the Claude directory, so if you've already added the
connector, you see one set of Sumtracker tools, not two.

To try the plugin from this repository instead, clone it and start Claude Code with
`claude --plugin-dir ./sumtracker-claude-plugin`.

## Connect your account

- **Claude (web and desktop):** connect Sumtracker from the plugin's **Connectors** tab.
- **Claude Code:** run `/mcp` and pick `sumtracker`.

Sumtracker opens in your browser. Sign in and approve the connection, and you're sent back to Claude. Claude never
sees your password.

## Things to ask

- "Which products are below 10 units at the Main warehouse?"
- "Set SKU TSHIRT-RED-M to 40 units at the Main warehouse, reason: stock count."
- "Which supplier is primary for SKU TSHIRT-RED-M, and at what cost?"
- "Why is my Shopify store not syncing?"

## Data and privacy

- **No local code.** The plugin runs nothing on your computer: no scripts, hooks or local servers. It holds only the
  connector settings and the skill instructions.
- **One destination.** Claude connects only to Sumtracker's MCP server, `https://mcp.sumtracker.com/mcp`, over HTTPS,
  with OAuth. The plugin stores nothing itself.
- **Reads and changes.** Claude reads from and writes to your Sumtracker account. Changes, such as stock adjustments,
  take effect right away and are sent to your sales channels on their next sync, so buyers see the new stock. Every
  Sumtracker tool is marked as read-only or as one that changes data, and Claude asks you before it changes anything.
- **Logs.** Sumtracker logs each request Claude makes, with the tool, its arguments and the connected user, to run
  the service and investigate problems. See the [privacy policy](https://www.sumtracker.com/privacy-policy).

## Manage or disconnect

**Settings > Connected AI Apps** in Sumtracker lists your connected apps, and **Disconnect** stops an app's access
right away. A connection also stops working after 30 days without use, when its user is deactivated, or when the
account's subscription is cancelled or its trial ends. To use Claude with Sumtracker again, connect it again.

## Support

- MCP server docs: [developers.sumtracker.com/mcp](https://developers.sumtracker.com/mcp)
- Help center: [help.sumtracker.com](https://help.sumtracker.com)
- Contact: [support@sumtracker.com](mailto:support@sumtracker.com) or
  [sumtracker.com/contact-us](https://www.sumtracker.com/contact-us)
- [Privacy policy](https://www.sumtracker.com/privacy-policy) and [terms of service](https://www.sumtracker.com/terms-of-use)

## License

Copyright © 2026 Sumtracker Software Private Limited. All rights reserved.

This plugin is proprietary software, not open source. The repository is public only so that Claude can install the
plugin. You may use the plugin only with a Sumtracker account, under the [Sumtracker Plugin License](LICENSE) and the
[Sumtracker Terms of Service](https://www.sumtracker.com/terms-of-use). It may not be copied, modified or
redistributed.
