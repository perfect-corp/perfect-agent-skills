# Set up YouCam AI Agent

Connect the bundled `youcam-ai-agent` MCP server when Claude prompts for access.
Complete the OAuth sign-in in the browser, then return to Claude and retry the
original request.

Never ask the user to paste an access token into chat. Claude should manage the
OAuth token and refresh flow through the connector.

If the connector does not become available:

1. Confirm that the YouCam AI Agent connector is enabled for this plugin.
2. Reconnect it to restart OAuth when authentication is missing or expired.
3. Retry the original request only after authentication succeeds.
4. If connection still fails, explain that YouCam is temporarily unavailable
   and offer either a later retry or clearly labeled general guidance.
