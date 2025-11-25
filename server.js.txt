const express = require("express");
const axios = require("axios");

const app = express();
const PORT = process.env.PORT || 3000;

// This will be set in Render (never put your API key in the code)
const ROBLOX_API_KEY = process.env.ROBLOX_API_KEY;

function extractId(obj) {
  if (!obj) return null;
  if (typeof obj.id === "number") return obj.id;
  if (typeof obj.gamePassId === "number") return obj.gamePassId;
  if (typeof obj.assetId === "number") return obj.assetId;
  return null;
}

app.get("/tipjar-gamepasses", async (req, res) => {
  const userId = parseInt(req.query.userId, 10);

  if (!userId) {
    return res.status(400).json({ error: "Missing or invalid userId" });
  }

  if (!ROBLOX_API_KEY) {
    return res.status(500).json({ error: "Backend missing ROBLOX_API_KEY env variable" });
  }

  try {
    const url = `https://apis.roblox.com/game-passes/v1/users/${userId}/game-passes?count=100`;

    const response = await axios.get(url, {
      headers: {
        "x-api-key": ROBLOX_API_KEY
      }
    });

    const payload = response.data;
    const list = payload?.data || payload?.gamePasses || [];

    const ids = list
      .map(extractId)
      .filter((id) => typeof id === "number");

    return res.json({ gamePassIds: ids });

  } catch (err) {
    console.error("Error from Open Cloud:", err.response?.status, err.response?.data);

    return res.status(500).json({
      error: "Open Cloud request failed",
      status: err.response?.status,
      details: err.response?.data
    });
  }
});

app.get("/", (req, res) => {
  res.send("TipJar backend online.");
});

app.listen(PORT, () => {
  console.log("Backend running on port", PORT);
});
