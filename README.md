const express = require("express");
const OAuth2 = require("discord-oauth2");
require("dotenv").config();

const app = express();
const port = 3000;

const oauth = new OAuth2({
  clientId: process.env.CLIENT_ID,
  clientSecret: process.env.CLIENT_SECRET,
  redirectUri: process.env.REDIRECT_URI,
});

app.get("/", (req, res) => {
  res.send("Discord Connection Backend Running!");
});

// OAuth2 endpoint
app.get("/auth/discord", (req, res) => {
  const url = oauth.generateAuthUrl({
    scope: ["identify", "connections"],
    state: "randomstring",
  });
  res.redirect(url);
});

app.get("/callback", async (req, res) => {
  const code = req.query.code;

  try {
    const token = await oauth.tokenRequest({
      code,
      scope: ["identify", "connections"],
      grantType: "authorization_code",
    });

    const user = await oauth.getUser(token.access_token);
    res.send(`Hello ${user.username}, you have connected!`);
  } catch (error) {
    res.send("Error connecting to Discord.");
  }
});

app.listen(port, "0.0.0.0", () => {
  console.log(`App running on http://localhost:${port}`);
});
