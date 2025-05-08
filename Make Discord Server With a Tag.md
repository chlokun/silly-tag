# Automation Script for Getting Discord Server with a Tag

Since around May 6th, 2025, newly created Discord servers have a small, random chance (approximately 1 in 500) of including the experimental tag feature; this script automates the repetitive process of creating and deleting servers to help find one.

> [!NOTE]
> Successfully obtaining a server with this flag is only the first step; actually *using* the tag functionality requires boosting the server 3 times


Before running the script, ensure you have completed the following steps:

1.  **Disable 2FA**: The script needs 2FA disabled to automatically delete temporary servers without requiring confirmation prompts. Re-enable 2FA after you finish using the script.
2.  **Install Vencord**: This script relies on functions exposed by the Vencord client modification. Download and install it from the official source.
3.  **Enable Vencord Plugin**: Open Discord Settings, go to the Vencord "Plugins" tab, find and enable the `ConsoleShortcuts` plugin. This plugin exposes useful library functions (like `findByProps`) to the developer console, which are necessary for this script to work.
4.  **Restart Discord Client**: Close and reopen Discord completely to ensure Vencord and the plugin are loaded correctly.
5.  **Create One Server Manually (Required)**: Before running the script make sure to create a single server through the normal Discord interface first. It's a required step.

## Instructions

1.  **Open Developer Console**: Use the standard browser/Electron method to open the developer console (usually `Ctrl+Shift+I` on Windows/Linux or `Cmd+Option+I` on Mac, or sometimes F12).
2.  **Allow Pasting (If Prompted)**: The console might require you to explicitly type `allow pasting` before accepting pasted code. 
3.  **Paste and Run Code**: Copy the entire JavaScript code block below, paste it into the console, and press Enter.

**Remember to re-enable 2FA on your account once the script has found a server or you decide to stop it.**

---

## The Script

> [!CAUTION]
> ## **🚨 WARNING: ACTIVE BAN ENFORCEMENT 🚨**
> * **Discord is actively issuing bans (temporary and potentially permanent) to accounts using this script. This is a direct violation of their Terms of Service.**  
> * **DO NOT USE THIS SCRIPT ON ANY ACCOUNT YOU VALUE. The risk of your account being flagged and banned is extremely high due to ongoing enforcement.**  
> * By proceeding to paste and run this code, you acknowledge these significant dangers and accept full personal responsibility for any resulting account actions, including irreversible bans.  
> * **Expand the code below ONLY if you fully understand and accept these severe consequences**

<details>

<summary> <b>THE SCRIPT (click to expand)</b> </summary>

```js
// CONFIGURABLE CONSTANTS

// Waiting times in milliseconds. Lower values increase creation speed but also ban risk.
// Do not set BASE_INTERVAL lower than 120 seconds or you'll hit the rate limit
const BASE_INTERVAL = 120_000;
const DELETE_DELAY = 2_000;
const MAX_RANDOM_ADDITIONAL_DELAY = 3_000;

const SERVER_NAME = "Tag server";
const STOP_ON_FOUND = true; // Stop the script when a guild with the tag is found,
                            // or keep running to find more guilds with the tag

/// DO NOT EDIT BELOW THIS LINE ///
console.clear();

function murmurhash3_32_gc(e, _) {
  // no im not gonna use the discord's own hash function

  let $ = (_ = _ || 0),
    c,
    l = new TextEncoder(),
    t = l.encode(e),
    u = t.length,
    i = Math.floor(u / 4),
    m = new DataView(t.buffer, t.byteOffset);
  for (let b = 0; b < i; b++) {
    let n = 4 * b;
    ($ ^= c =
      Math.imul(
        (c =
          ((c = Math.imul((c = m.getUint32(n, !0)), 3432918353)) << 15) |
          (c >>> 17)),
        461845907
      )),
      ($ = Math.imul(($ = ($ << 13) | ($ >>> 19)), 5) + 3864292196),
      ($ >>>= 0);
  }
  c = 0;
  let f = 4 * i;
  switch (3 & u) {
    case 3:
      c ^= t[f + 2] << 16;
    case 2:
      c ^= t[f + 1] << 8;
    case 1:
      (c ^= t[f + 0]),
        ($ ^= c =
          Math.imul(
            (c = ((c = Math.imul(c, 3432918353)) << 15) | (c >>> 17)),
            461845907
          ));
  }
  return (
    ($ ^= u),
    ($ ^= $ >>> 16),
    ($ = Math.imul($, 2246822507)),
    ($ ^= $ >>> 13),
    ($ = Math.imul($, 3266489909)),
    ($ ^= $ >>> 16) >>> 0
  );
}

{
  // Check if the required functions are available
  if (typeof findByProps !== "function") {
    throw new Error(
      "Essential function `findByProps` is missing. Please ensure the 'ConsoleShortcuts' Vencord plugin is installed and enabled."
    );
  }
  if (!findByProps("createGuildFromTemplate")) {
    throw new Error(
      "Could not find the `createGuildFromTemplate` function. Create a server manually once and then try running the script again."
    );
  }
}

const deleteGuild = findByProps(
  "deleteGuild",
  "bulkAddMemberRoles"
).deleteGuild;
const createGuildFromTemplate = findByProps(
  "createGuildFromTemplate"
).createGuildFromTemplate;

class GuildCreator {
  constructor() {
    this.keepRunning = true;
  }

  isInExperimentRange(guild) {
    let hash = murmurhash3_32_gc(`2025-02_skill_trees:${guild.id}`) % 10000;
    return (hash >= 10 && hash < 20) || (hash >= 60 && hash < 100);
  }

  async processGuildCycle() {
    if (!this.keepRunning) {
      console.log("Script instructed to stop. Exiting guild creation cycle.");
      return;
    }

    console.log("Attempting to create a new guild...");
    const newGuild = await createGuildFromTemplate(
      SERVER_NAME,
      null,
      {
        id: "CREATE",
        label: "Create My Own",
        channels: [],
        system_channel_id: null,
      },
      false,
      false
    );

    if (!newGuild || !newGuild.id) {
      console.error("Failed to create guild.");
      // Schedule next attempt even if this one failed
      if (this.keepRunning) {
        this.scheduleNextCycle();
      }
      return;
    }

    console.log(`Guild created: ${newGuild.name} (ID: ${newGuild.id})`);
    if (this.isInExperimentRange(newGuild)) {
      console.log(
        `🎉 FOUND GUILD WITH TAG: ${newGuild.name} (ID: ${newGuild.id}) 🎉`
      );
      if (STOP_ON_FOUND) {
        console.log("Stopping script as a guild with a tag has been found.");
        this.keepRunning = false;
        return;
      }
      console.log("Guild with tag found, finding more guilds with a tag...");
      this.scheduleNextCycle();
      return;
    } else {
      console.log(
        `Guild (ID: ${newGuild.id}) does not have the tag experiment. Scheduling deletion...`
      );
      setTimeout(async () => {
        console.log(`Deleting guild: ${newGuild.name} (ID: ${newGuild.id})`);
        try {
          await deleteGuild(newGuild.id);
          console.log(`Guild (ID: ${newGuild.id}) deleted.`);
        } catch (err) {
          console.error(`Error deleting guild (ID: ${newGuild.id}):`, err);
        }
      }, DELETE_DELAY + Math.random() * MAX_RANDOM_ADDITIONAL_DELAY);
    }
  }

  scheduleNextCycle() {
    if (!this.keepRunning) return;

    const randomAdditionalDelay = Math.random() * MAX_RANDOM_ADDITIONAL_DELAY;
    const currentInterval = BASE_INTERVAL + randomAdditionalDelay;

    console.log(
      `Next attempt in ${(currentInterval / 1000).toFixed(2)} seconds.`
    );
    console.log("Please wait...");
    setTimeout(() => this.processGuildCycle(), currentInterval);
  }
}

// Initial start of the script
console.log("===== Guild Creation Script =====");
console.log("       Script by Bytexenon       ");
console.log("=================================");
console.log("Starting guild creation script with randomized intervals.");
console.log(
  `Base interval: ${BASE_INTERVAL / 1000}s. Max additional random delay: ${
    MAX_RANDOM_ADDITIONAL_DELAY / 1000
  }s.`
);
console.log("----------------------------------------");

// Create an instance of the GuildCreator class
const guildCreator = new GuildCreator();
guildCreator.scheduleNextCycle();

```

</details>

---

### Troubleshooting

1. **Why do I get the `Uncaught TypeError: Cannot read properties of null (reading 'createGuildFromTemplate')` error?**  
   1.1. You need to manually create a new server before running the script; otherwise, the script won't be able to create a new server. If you ever need to run this script again, you'll need to repeat this step.

2. **Why do I get the `Uncaught ReferenceError: findByProps is not defined` error?**  
   2.1. You need to install **Vencord** and enable the **ConsoleShortcuts** plugin (not the **ConsoleJanitor** one!). After that, restart your client, manually create a server, and then run the script.
  
3. **How can I remove the "Hold Up!" messages from the console?**  
   3.1. You can use the **ConsoleJanitor** plugin to eliminate messages like these, but it's not a necessary step.