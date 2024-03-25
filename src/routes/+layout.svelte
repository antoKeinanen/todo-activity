<script lang="ts">
  import { DiscordSDK } from "@discord/embedded-app-sdk";
  import { PUBLIC_DISCORD_CLIENT_ID } from "$env/static/public";
  import "./app.css";

  let auth;

  const discordSdk = new DiscordSDK(PUBLIC_DISCORD_CLIENT_ID);

  setupDiscordSdk().then(() => {
    console.log("Discord is authenticated!");
  });

  async function setupDiscordSdk() {
    await discordSdk.ready();
    console.log("Discord SDK is ready!");

    const { code } = await discordSdk.commands.authorize({
      client_id: PUBLIC_DISCORD_CLIENT_ID,
      response_type: "code",
      state: "",
      prompt: "none",
      scope: ["identify", "guilds"],
    });

    const response = await fetch("/api/token", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ code }),
    });

    const { access_token } = await response.json();

    auth = await discordSdk.commands.authenticate({ access_token });

    if (auth == null) {
      throw new Error("Failed to authenticate with Discord");
    }
  }
</script>

<slot />
