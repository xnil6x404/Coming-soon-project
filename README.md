# Coming-soon-project
# GoatBot V2 (Telegram-ChatBot) - Project Documentation

## Table of Contents
1. [Overview](#overview)
2. [Project Structure](#project-structure)
3. [System Architecture](#system-architecture)
4. [API Documentation](#api-documentation)
5. [Command System](#command-system)
6. [Game Data System](#game-data-system)
7. [Setup Instructions](#setup-instructions)
8. [Recent Updates](#recent-updates)

---

# Overview

GoatBot V2 is a comprehensive Telegram bot providing an economy system, AI image generation, interactive games, and entertainment features for Telegram groups, built with Node.js. It features a modular architecture, supports commands, events, and uses MongoDB for persistent user data. Users can manage balances, play games, generate AI art, interact with an AI chatbot, and receive automated group messages. The bot also includes a web dashboard displaying real-time status and statistics.

## Key Features

- 🎨 **AI Image Generation** with MidJourney & Niji Journey integration
- 💰 **Economy System** with user balance tracking and daily rewards
- 🎮 **Interactive Games** (Dice, Slots, Wheel of Fortune)
- 🤖 **AI Chatbot** with conversational replies
- 📊 **MongoDB Database** for persistent data storage
- 🔧 **Modular Command System** with aliases support
- 👋 **Event Handlers** for group member join/leave
- ⚡ **Fast Execution** with role-based access control
- 📥 **Auto-Download** videos from social media platforms
- 🌐 **Web Dashboard** on port 5000 with real-time stats
- 🎯 **Simplified API** - context-aware, no need to pass message object

## User Preferences

Preferred communication style: Simple, everyday language.

---

# Project Structure

## Directory Structure

```
GoatbotV2/
├── commands/              # Bot commands (21 total)
│   ├── autodl.js         # Auto-download videos from social media
│   ├── baby.js           # AI chatbot with conversational replies
│   ├── balance.js        # Check user balance
│   ├── daily.js          # Claim daily rewards (up to 5x/day)
│   ├── demo.js           # NEW! API demo command
│   ├── dice.js           # Dice betting game with stats
│   ├── gamestats.js      # View all game statistics
│   ├── help.js           # List all commands
│   ├── info.js           # Bot information
│   ├── mj.js             # MidJourney AI image generation
│   ├── niji.js           # Niji Journey anime AI
│   ├── ping.js           # Check bot status
│   ├── prefix.js         # Configure command prefix
│   ├── profile.js        # User profile information
│   ├── slot.js           # Slot machine game
│   ├── tid.js            # Get thread/chat ID
│   ├── uid.js            # Get user ID
│   ├── unseen.js         # Unseen message tracking
│   ├── uptime.js         # Bot uptime information
│   ├── waifu.js          # Random anime waifu images
│   └── wheel.js          # Wheel of Fortune game
│
├── events/               # Event handlers
│   ├── welcome.js       # Welcome new members
│   └── left.js          # Goodbye to leaving members
│
├── handlers/            # Core handlers
│   └── handlerEvent.js  # Loads and manages commands & events
│
├── database/            # Database system
│   ├── connectDB/       # Database connections (MongoDB & SQLite)
│   ├── controller/      # Data controllers (users, threads, global)
│   └── models/          # Database schemas
│       ├── mongodb/     # MongoDB models
│       └── sqlite/      # SQLite models
│
├── utils/               # Utility modules
│   ├── api.js           # Telegram API helper (simplified)
│   ├── loaddatabase.js  # Database initialization
│   └── utils.js         # Helper functions
│
├── public/              # Web dashboard
│   └── index.html       # Dashboard interface
│
├── main.js              # Main bot entry point
├── config.json          # Bot configuration
├── package.json         # NPM dependencies
└── project.md          # This documentation file
```

---

# System Architecture

## Backend Architecture

The bot is a Node.js application utilizing `telegraf` with an event-driven architecture. Core components include:
- **main.js** for initialization
- **modular command handler** for dynamic command loading
- **event handler** for Telegram events
- **managers** for user and thread data

Design patterns employed:
- **Singleton** (HandlerEvent)
- **Module** (commands/events)
- **Factory** (user creation)

## Data Storage

**MongoDB Atlas** is used with Mongoose ODM. Two primary schemas exist:

### User Schema
Stores:
- `userId` - Telegram user ID
- `username` - Telegram username
- `firstName`, `lastName` - User names
- `money` - Balance (default: 1000)
- `dailyCount`, `lastDaily` - Daily reward tracking
- `commandCooldowns` - Cooldown tracking
- `gameData` - Flexible game statistics
- `createdAt`, `lastUpdated` - Timestamps

### Thread Schema
Stores:
- `threadId` - Chat/group ID
- `threadName` - Chat/group name
- `totalUsers` - Member count
- `isGroup` - Group or private chat
- `prefix` - Custom command prefix
- `createdAt`, `lastUpdated` - Timestamps

Abstraction layers (`usersData.js`, `threadsData.js`) handle database operations.

## Command System Architecture

Commands are modular, exporting:
- `config` - Metadata (name, aliases, description, role, cooldown)
- `onStart` - Main command logic
- `onChat` - Processes messages without prefix (optional)
- `onReply` - Handles replies to bot messages (optional)
- `onCallbackQuery` - Handles inline button clicks (optional)

### Role System
- **Role 0**: All users
- **Role 1**: Group administrators
- **Role 2**: Bot administrator only

### Cooldown System
Prevents spam with `countDown` (in seconds)

### Supported Commands

**AI & Media:**
- `mj/midjourney` - AI image generation with upscale/variation/refresh
- `niji/nijijourney/anime` - Anime-style AI art
- `baby/bot/bbu` - AI chatbot
- `autodl` - Auto-download videos
- `waifu/wife` - Random anime images

**Economy:**
- `balance/bal` - Check balance
- `daily/claim` - Daily rewards
- `profile/pf` - User profile

**Games:**
- `dice` - Dice betting
- `slot` - Slot machine
- `wheel` - Wheel of Fortune
- `gamestats` - View statistics

**Info:**
- `help/h` - Commands list
- `ping/pong` - Bot status
- `uptime` - Bot uptime
- `uid/userid` - User ID
- `tid` - Thread/chat ID
- `info/botinfo` - Bot information

**Config:**
- `prefix` - View/set command prefix

**Demo:**
- `demo` - Test all API features

## Event System Architecture

Events handle Telegram chat events:
- `welcome` - New member greeting
- `left` - Member departure message

Similar modular structure to commands.

## Message Processing Flow

Messages are processed through a pipeline:
1. **Context creation** - Create messageObj with reply/unsend methods
2. **User/thread update** - Ensure user/thread exists in database
3. **onReply check** - Handle replies to bot messages
4. **Prefix-based command** - Lookup and execute commands
5. **Role/cooldown checks** - Verify permissions
6. **Command execution** - Run command logic
7. **Event handling** - Process new/left members
8. **onChat processing** - Run for all applicable commands
9. **onCallbackQuery** - Handle button clicks

## Telegram API Utility (`utils/api.js`)

A context-aware utility providing **simplified helper methods** for sending various media types via the Telegram API.

### Recent Updates (Oct 28, 2025)

✅ Implemented **simplified API syntax** - no need to pass `message` object  
✅ Added `message.reply({body, attachment})` with auto file type detection  
✅ Enhanced error handling for 403 errors (bot removed from chat)  
✅ All commands updated to use simplified syntax  

### Simplified API Examples

```javascript
// New simplified syntax (recommended)
await api.sendMessage("Hello!");
await api.sendPhoto(photoUrl, "Caption here");
await api.sendVideo(videoUrl, "Watch this!");

// Reply with attachment (auto-detects photo/video/audio/document)
await message.reply({
  body: "Message text",
  attachment: "https://example.com/file.jpg"
});

// Interactive buttons
await api.sendMessage({
  body: "Click a button:",
  buttons: [
    { text: "👍 Like", callback: "like_action" }
  ]
});

// Edit message
const msg = await api.sendMessage("Old text");
setTimeout(() => api.editMessage(msg.message_id, "Edited! ✅"), 2000);

// Location
await api.sendLocation(23.7805733, 90.2792399);

// Typing indicator
await api.sendAction("typing");
```

**Updated Commands**: `waifu.js`, `profile.js`, `autodl.js`, `uid.js` now use simplified API  
**Demo Command**: New `/demo` command showcases all API features

## MidJourney Integration

Implemented in `commands/mj.js` and `commands/niji.js`, uses TrueAI API endpoint for image generation.

Features:
- Initial image generation
- Upscaling (U1-U4)
- Variations (V1-V4)
- Refresh functionality
- Interactive inline keyboards
- Progress tracking with polling

## Web Dashboard Architecture

**Express.js** web server integrated into `main.js`, serves dashboard on **port 5000**.

Features:
- `/api/status` endpoint - real-time bot info
- Responsive UI (`public/index.html`)
- Modern design with gradient effects
- Bot status and command listings
- Auto-refresh every 5 seconds

## Configuration Management

Configuration via `config.json`:
```json
{
  "botToken": "YOUR_BOT_TOKEN",
  "mongodbUri": "YOUR_MONGODB_URI",
  "adminId": "ADMIN_TELEGRAM_ID",
  "prefix": "/"
}
```

Prefix is customizable globally or per-group.

## External Dependencies

### Third-Party Services
- **Telegram Bot API**: `telegraf` for bot interaction
- **MongoDB Atlas**: Cloud database via Mongoose
- **TrueAI API**: `https://ai.trueai.org` for AI image generation

### NPM Packages
- `telegraf` - Telegram bot framework
- `mongoose` - MongoDB ODM
- `axios` - HTTP client
- `express` - Web server
- `dotenv` - Environment variables
- `moment-timezone` - Date/time utilities
- `lodash` - Utility functions
- `fs-extra` - File system operations
- `tinyurl` - URL shortening
- `sequelize`, `sqlite3` - SQLite support

---

# API Documentation

## All Working API Methods

### 1. Simple Message
```javascript
api.sendMessage("Hello from Telegraf!");
```

### 2. Photo Sending
```javascript
api.sendPhoto("example.jpg", "Here's a photo!");
api.sendPhoto("https://example.com/photo.jpg", "Remote photo!");
```

### 3. Video Sending
```javascript
api.sendVideo("video.mp4", "Watch this!");
api.sendVideo("https://example.com/video.mp4", "Remote video!");
```

### 4. Audio Sending
```javascript
api.sendAudio("music.mp3", "Listen 🎧");
api.sendAudio("https://example.com/song.mp3", "Great music!");
```

### 5. Message Editing
```javascript
api.sendMessage("Old message").then(async (msg) => {
  setTimeout(() => api.editMessage(msg.message_id, "Edited message ✅"), 2000);
});
```

### 6. Custom Buttons
```javascript
api.sendMessage({
  body: "Click a button:",
  buttons: [
    { text: "👍 Like", callback: "like" },
    { text: "👎 Dislike", callback: "dislike" },
  ],
});
```

### 7. Location Sending
```javascript
// Dhaka, Bangladesh coordinates
api.sendLocation(23.7805733, 90.2792399);
```

### 8. Typing Indicator
```javascript
api.sendAction("typing");
// Other actions: "upload_photo", "record_video", "upload_video", etc.
```

### 9. Simple Reply
```javascript
message.reply("hi");
```

### 10. Reply with Attachment (NEW!)
```javascript
message.reply({
  body: "hi", 
  attachment: "https://example.com/photo.jpg"
});
```

**Auto-detected file types:**
- **Photos**: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`
- **Videos**: `.mp4`, `.avi`, `.mov`, `.webm`
- **Audio**: `.mp3`, `.wav`, `.ogg`, `.m4a`
- **Documents**: All other file types

## Additional API Methods

### Document Sending
```javascript
api.sendDocument("file.pdf", "Important document");
```

### Animation/GIF
```javascript
api.sendAnimation("funny.gif", "LOL 😂");
```

### Voice Message
```javascript
api.sendVoice("voice.ogg");
```

### Video Note (Circular)
```javascript
api.sendVideoNote("circle.mp4");
```

### Media Group (Album)
```javascript
api.sendMediaGroup([
  { type: 'photo', media: 'url1.jpg', caption: 'Photo 1' },
  { type: 'photo', media: 'url2.jpg', caption: 'Photo 2' },
  { type: 'video', media: 'video.mp4', caption: 'Video' }
]);
```

### Contact Sharing
```javascript
api.sendContact("+1234567890", "John Doe");
```

### Poll Creation
```javascript
api.sendPoll("Your favorite color?", ["Red", "Blue", "Green"]);
```

### Dice/Emoji Animation
```javascript
api.sendDice({ emoji: "🎲" });
// Other emojis: 🎯 🏀 ⚽ 🎰 🎳
```

### Message Deletion
```javascript
api.deleteMessage(messageId);
```

### Edit Message Caption
```javascript
api.editMessageCaption(messageId, "New caption");
```

## Complete API Method List

| Method | Simplified Syntax | Description |
|--------|------------------|-------------|
| `sendMessage` | `api.sendMessage("text")` | Send text message |
| `sendPhoto` | `api.sendPhoto(url, caption)` | Send photo |
| `sendVideo` | `api.sendVideo(url, caption)` | Send video |
| `sendAudio` | `api.sendAudio(url, caption)` | Send audio |
| `sendDocument` | `api.sendDocument(url, caption)` | Send document |
| `sendAnimation` | `api.sendAnimation(url, caption)` | Send GIF |
| `sendVoice` | `api.sendVoice(url)` | Send voice |
| `sendVideoNote` | `api.sendVideoNote(url)` | Send circular video |
| `sendMediaGroup` | `api.sendMediaGroup([media])` | Send album |
| `sendLocation` | `api.sendLocation(lat, lng)` | Send location |
| `sendContact` | `api.sendContact(phone, name)` | Send contact |
| `sendPoll` | `api.sendPoll(question, [options])` | Send poll |
| `sendDice` | `api.sendDice({emoji})` | Send dice |
| `sendAction` | `api.sendAction("typing")` | Send chat action |
| `editMessage` | `api.editMessage(id, text)` | Edit message |
| `deleteMessage` | `api.deleteMessage(id)` | Delete message |

## Key Benefits

✅ **Context-Aware**: No need to pass `message` object to `api` methods  
✅ **Backward Compatible**: Old commands still work  
✅ **Auto File Detection**: Automatically sends correct media type  
✅ **Error Handling**: Built-in handling for bot removal (403 errors)  
✅ **Flexible Syntax**: Multiple ways to use same features  
✅ **Interactive Buttons**: Easy callback query handling  

## Example Command Structure

```javascript
module.exports = {
  config: {
    name: "example",
    description: { en: "Example command" }
  },

  onStart: async ({ message, api, args }) => {
    // Use any api method directly!
    await api.sendMessage("Hello!");
    await api.sendPhoto("https://example.com/pic.jpg", "Caption");
    
    await message.reply("Simple reply");
    await message.reply({
      body: "Reply with image!",
      attachment: "https://example.com/image.jpg"
    });
  },
  
  onCallbackQuery: async ({ api, callbackQuery }) => {
    // Handle button clicks
    if (callbackQuery.data === "like") {
      await api.sendMessage("You clicked Like! 👍");
    }
  }
};
```

---

# Command System

## Creating a New Command

```javascript
// commands/mycommand.js
module.exports = {
  config: {
    name: "mycommand",
    aliases: ["mc", "cmd"],
    version: "1.0",
    author: "Your Name",
    countDown: 5, // Cooldown in seconds
    role: 0, // 0: all, 1: admins, 2: bot admin
    description: {
      en: "My awesome command"
    },
    category: "fun",
    guide: {
      en: "/mycommand - does something cool"
    }
  },

  onStart: async function({ message, api, usersData, threadsData, event, args }) {
    // Your command logic here
    await api.sendMessage("Hello from my command!");
  },
  
  // Optional handlers
  onChat: async function({ message, event, api }) {
    // Runs on every message
  },
  
  onReply: async function({ message, event, Reply, api }) {
    // Runs when user replies to bot
  },
  
  onCallbackQuery: async function({ api, callbackQuery }) {
    // Runs when user clicks inline buttons
  }
};
```

## Available Parameters

```javascript
onStart: async function({ 
  message,      // Message utilities (reply, bot, chatId)
  api,          // Telegram API helpers
  usersData,    // User database operations
  threadsData,  // Thread/group database operations
  event,        // Raw Telegram event
  args,         // Command arguments
  commandHandler, // Access to other commands
  prefix,       // Command prefix (default: /)
  role          // User role (0: user, 1: admin, 2: bot admin)
})
```

## Updated Commands (Oct 28, 2025)

### Before & After Comparison

**waifu.js:**
```javascript
// Before
await api.sendPhoto(message, waifuUrl, { caption: "✨ Here's your waifu!" });

// After
await api.sendPhoto(waifuUrl, "✨ Here's your waifu!");
```

**profile.js:**
```javascript
// Before
await api.sendPhoto(message, fileId, { caption: profileMsg });

// After
await api.sendPhoto(fileId, profileMsg);
```

**autodl.js:**
```javascript
// Before
await api.sendMessage({body: caption, attachment: Buffer}, threadID, msgID);

// After
await api.sendVideo(videoUrl, caption);
```

**uid.js:**
```javascript
// Before
await api.sendContact(message, userId, firstName, options);

// After
await api.sendContact(userId, firstName, options);
```

## Command Status

- **Total Commands**: 21
- **Updated Commands**: 4 (waifu, profile, autodl, uid)
- **Already Compatible**: 15 commands
- **Special Commands**: 2 (mj, niji - using advanced features)
- **New Commands**: 1 (demo)

---

# Game Data System

## Overview

The flexible game data system allows storing ANY game statistics without editing database schema files. All game data is stored in the `gameData` field using dot notation.

## Core Features

✅ **No Schema Edits Required** - Add new game data dynamically  
✅ **Dot Notation Support** - Access nested data easily  
✅ **Helper Methods** - Convenient getters/setters  
✅ **Backward Compatible** - Works with existing code  

## Helper Methods

```javascript
// Get game data
await usersData.getGameData(userId, 'dice.totalWins')
await usersData.getGameData(userId, 'slots') // Get all slots data
await usersData.getGameData(userId) // Get all game data

// Set game data
await usersData.setGameData(userId, 'dice.totalWins', 10)
```

## Usage Examples

### Dice Game Data
```javascript
// Store dice statistics
await usersData.set(userId, {
  'gameData.dice.totalWins': 10,
  'gameData.dice.totalLosses': 5,
  'gameData.dice.totalGames': 15,
  'gameData.dice.lastPlayedAt': new Date(),
  'gameData.dice.highestRoll': 6
});

// Retrieve
const diceData = await usersData.getGameData(userId, 'dice');
const wins = await usersData.getGameData(userId, 'dice.totalWins');
```

### Slot Game Data
```javascript
await usersData.set(userId, {
  'gameData.slots.totalSpins': 100,
  'gameData.slots.totalWagered': 5000,
  'gameData.slots.totalWinnings': 6500,
  'gameData.slots.jackpots': 3,
  'gameData.slots.lastPlayedAt': new Date()
});
```

### Incrementing Counters
```javascript
const currentWins = (await usersData.getGameData(userId, 'dice.totalWins')) || 0;
await usersData.setGameData(userId, 'dice.totalWins', currentWins + 1);
```

### Multiple Games
```javascript
await usersData.set(userId, {
  'gameData.dice.totalWins': 10,
  'gameData.slots.jackpots': 2,
  'gameData.poker.handsWon': 15,
  'gameData.blackjack.totalGames': 50,
  'gameData.roulette.biggestWin': 100000
});

const allGames = await usersData.getGameData(userId);
```

### Achievements
```javascript
await usersData.set(userId, {
  'gameData.achievements.firstWin': true,
  'gameData.achievements.jackpotMaster': true,
  'gameData.achievements.luckyStreak': 10,
  'gameData.achievements.unlockedAt': new Date()
});
```

## Data Structure Example

```javascript
{
  userId: "123456",
  money: 5000,
  gameData: {
    dice: {
      totalWins: 10,
      totalLosses: 5,
      totalGames: 15,
      lastPlayedAt: "2024-10-15T12:00:00Z"
    },
    slots: {
      totalSpins: 100,
      totalWagered: 50000,
      totalWinnings: 65000,
      jackpots: 3,
      lastPlayedAt: "2024-10-15T13:00:00Z"
    }
  }
}
```

## Best Practices

- Always provide default values: `|| 0` or `|| {}`
- Use `new Date()` for timestamps
- Group related updates in one `set()` call
- Use helper methods for cleaner code
- Check for null/undefined before accessing nested data

---

# Setup Instructions

## Prerequisites

- Node.js (v16 or higher)
- MongoDB database (local or MongoDB Atlas)
- Telegram Bot Token from [@BotFather](https://t.me/BotFather)

## Installation

```bash
npm install
```

## Configuration

Edit `config.json`:

```json
{
  "botToken": "YOUR_BOT_TOKEN_HERE",
  "adminId": "YOUR_ADMIN_TELEGRAM_ID",
  "mongodbUri": "YOUR_MONGODB_URI_HERE",
  "prefix": "/"
}
```

### Getting Bot Token

1. Open Telegram and search for [@BotFather](https://t.me/BotFather)
2. Send `/newbot` and follow instructions
3. Copy the token provided

### Getting Your Telegram ID

1. Search for [@userinfobot](https://t.me/userinfobot) on Telegram
2. Send `/start` to get your user ID

### MongoDB Setup

- **Local**: `mongodb://localhost:27017/goatbot`
- **Cloud**: Get connection string from [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)

## Running the Bot

```bash
npm start
```

The bot will start and the web dashboard will be available at http://localhost:5000

## Testing

Try the new `/demo` command to test all features:

```
/demo text      - Simple message
/demo photo     - Photo with caption
/demo video     - Video sending
/demo audio     - Audio file
/demo edit      - Edit message demo
/demo buttons   - Interactive buttons
/demo location  - Send GPS location
/demo typing    - Typing indicator
/demo reply     - Reply with attachment
/demo all       - Run all demos at once!
```

## Troubleshooting

### Bot not responding
- Check bot token is correct
- Ensure MongoDB is running and URI is correct
- Check console for error messages

### Database connection issues
- Verify MongoDB URI is correct
- Check network connectivity
- Ensure MongoDB service is running

### Command not working
- Check command file is in `commands/` folder
- Ensure file exports correct structure
- Restart bot after adding new commands

---

# Recent Updates

## October 28, 2025 - Simplified API Update

### What Changed

✅ **Simplified API Syntax** - No need to pass `message` object anymore  
✅ **Enhanced message.reply()** - Now supports `{body, attachment}` with auto file type detection  
✅ **Fixed messageObj** - Added missing `messageObj` definition in main.js  
✅ **Better Error Handling** - Built-in handling for 403 errors (bot removed from chat)  
✅ **Updated Commands** - 4 commands updated to use new simplified API  
✅ **New Demo Command** - `/demo` command showcases all API features  

### Updated Commands

1. **waifu.js** - Random anime images
2. **profile.js** - User profile with photo
3. **autodl.js** - Video downloader
4. **uid.js** - User ID & contact card

### Benefits Comparison

| Before | After |
|--------|-------|
| `api.sendMessage(message, "Hi")` | `api.sendMessage("Hi")` |
| `api.sendPhoto(message, url, {caption})` | `api.sendPhoto(url, caption)` |
| More verbose code | Cleaner, shorter code |
| Manual error handling | Built-in error handling |

### Quick Example

```javascript
module.exports = {
  config: {
    name: "example",
    description: { en: "Example command" }
  },

  onStart: async ({ message, api }) => {
    // Send message
    await api.sendMessage("Hello! 👋");
    
    // Send photo
    await api.sendPhoto("https://picsum.photos/400", "Random photo");
    
    // Reply with attachment
    await message.reply({
      body: "Here's an image!",
      attachment: "https://picsum.photos/300"
    });
    
    // Interactive buttons
    await api.sendMessage({
      body: "Choose an option:",
      buttons: [
        { text: "Option 1", callback: "opt1" },
        { text: "Option 2", callback: "opt2" }
      ]
    });
  },
  
  onCallbackQuery: async ({ api, callbackQuery }) => {
    if (callbackQuery.data === "opt1") {
      await api.sendMessage("You chose option 1!");
    }
  }
};
```

### What You Can Do Now

All these API methods work seamlessly:

✅ Send text messages  
✅ Send photos with captions  
✅ Send videos  
✅ Send audio files  
✅ Edit messages after sending  
✅ Send interactive buttons  
✅ Send GPS locations  
✅ Show typing indicators  
✅ Reply with attachments  
✅ Auto-detect file types  
✅ Send contacts, polls, dice  

## Bot Status

```
✅ Bot is running
📦 21 commands loaded
📌 Prefix: /
💾 Database: MongoDB (5150 users, 233 groups)
🌐 Dashboard: http://0.0.0.0:5000
```

---

## License

ISC

## Author

GoatBot Development Team

---

**Last Updated**: October 28, 2025  
**Version**: GoatBot V2 with Simplified API  
**Status**: ✅ All systems operational!
