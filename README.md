# 🃏 Virtual Cards — Recognition Card Application for ServiceNow

> A custom ServiceNow scoped application that lets employees send collectible recognition cards to colleagues. Inspired by CreatorCon collectables. Built with vibe-coding.

---

## What is this?

We all work hard. And most of the time, the good work happens quietly — someone unblocks a colleague, picks up something that wasn't theirs to fix, or just consistently shows up for the team. And nothing gets said about it. Not because nobody noticed, but because there was nowhere to put it.

This app fixes that. **Virtual Cards** gives every employee a simple way to send a recognition card to a colleague — a proper collectable, not just a Teams message that gets buried in the feed.

The idea came from **CreatorCon collectables** and collided with an itch to test ServiceNow's vibe-coding capabilities. How far could you get building a real, functional application through prompting and iteration? Turns out, pretty far. The app was born from both of those things at once.

---

## Features

- 🎴 **Collectible recognition cards** — visual cards stored as assets, not just notifications
- 📬 **Email notification** on top of the in-platform record
- 🗂️ **Dedicated space** — cards live in a portal, not buried in chat
- 📊 **Reportable** — who gave what, to whom, and when
- 🔒 **Freeze period support** — configurable cooldown per category (currently set to 0 — no blocking)
- 🤖 **AI card generation** — workflow step to generate card images via OpenAI (infrastructure ready, cards currently hand-crafted)
- 🧩 **Two catalog items** — Request a new card design, or assign an existing card to a colleague

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                 User Interface                  │
│  Portal Widget    Portal Widget    Catalog x2   │
│  (My Cards)       (All Cards)      (Request/    │
│                                     Assign)     │
└──────────────┬──────────────────────┬───────────┘
               ↓                      ↓
┌─────────────────────────┐  ┌────────────────────┐
│   RecognitionCardUtils  │←─│  Flow Designer     │
│   Script Include        │  │  Fulfillment       │
│   x_konl_visual_card    │  │  Workflows         │
└──────────┬──────────────┘  └────────────────────┘
           ↓
┌──────────────────────────────────────────────────┐
│                   Data Layer                     │
│  alm_information_asset   cmdb_model (cards)      │
│  (assigned cards)        cmdb_model_category     │
└──────────────────────────────────────────────────┘
           ↕ (dotted)                    ↕ (dotted)
    System Properties            OpenAI API
    x_konl_visual_card.*         (image generation)
```

**Key components:**
| Component | Type | Notes |
|---|---|---|
| `RecognitionCardUtils` | Script Include | Core logic hub — freeze periods, filtering, card creation |
| `alm_information_asset` | Table (reused) | Stores assigned cards per user |
| `cmdb_model` | Table (reused) | Card designs catalogue |
| `cmdb_model_category` | Table (reused) | Card type taxonomy |
| My Cards widget | Service Portal | Shows cards assigned to the logged-in user |
| All Cards widget | Service Portal | Browse the full card library |
| Request New Recognition Card | Catalog Item | Create a new card design |
| Assign Recognition Card | Catalog Item | Send an existing card to a colleague |

No custom tables. Built on reusable platform components wherever possible.

---

## System Properties

| Property | Description | Default |
|---|---|---|
| `x_konl_visual_card.recognition.freeze_categories` | Comma-separated category sys_ids subject to freeze | _(your instance sys_id)_ |
| `x_konl_visual_card.recognition.freeze_window_days` | Cooldown window in days | `0` (disabled) |
| `x_konl_visual_card.recognition.manager_only_categories` | Categories restricted to manager role | _(your instance sys_id)_ |
| `x_konl_visual_card.model` | OpenAI model for card generation | `gpt-4o` |
| `x_konl_visual_card.instruction` | Full system prompt for AI card generation | _(see property)_ |
| `x_konl_visual_card.logo_id` | Attachment sys_id of the logo used in card generation | _(your instance sys_id)_ |
| `x_konl_visual_card.character` | Attachment sys_id of the character sheet used in card generation | _(your instance sys_id)_ |

---

## Installation

> ⚠️ This is not a plug-and-play install. Some configuration and workflow wiring will be required for your instance.

1. **Import the scoped app** via Studio Source Control or Update Set
2. **Update system properties** — replace all instance-specific sys_ids with values from your own instance
3. **Update the hardcoded sys_id** in `RecognitionCardUtils`:
   ```javascript
   RECOGNITION_PARENT_CATEGORY: 'd1cb5c763b4bf610ebe9c15765e45abe'
   ```
   Replace with the sys_id of your `cmdb_model_category` parent record for "Recognition Cards"
4. **Configure the Service Portal** — add the widgets to your portal page
5. **Set up OpenAI credentials** if you want AI card generation (store the API key in a ServiceNow credential record, not in properties)
6. **Style the cards and portal** to match your organisation's branding

---

## Known Limitations

- **Not fully plug-and-play** — platform-specific workflows and notification rules need wiring for your instance
- **Styling needs work** — card visuals, portal theme, and email templates are functional but not fully polished
- **AI card generation is a placeholder** — the flow step exists and works, but output quality was inconsistent so all current cards are hand-crafted and fine-tuned
- **Sys_ids are instance-specific** — several properties and one constant in the script include point to knvldev sys_ids and must be updated

---

## Licence

This project is licensed under the **PolyForm Noncommercial License 1.0.0**.

Free to use, adapt, and share for non-commercial purposes. No warranties provided. Use at your own risk.

Full licence text: https://polyformproject.org/licenses/noncommercial/1.0.0

---

## Contributing

Contributions are welcome. If you improve something — styling, the AI generation flow, portal widgets, documentation — please open a PR.

If you use it and run into something that doesn't work, open an issue. No guarantees on response time but it will be read.

---

## Built by

Dzmitry Peshkur using Konversational isntance

Inspired by CreatorCon collectables. Powered by ServiceNow vibe-coding.
