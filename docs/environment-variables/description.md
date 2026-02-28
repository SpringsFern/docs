# Environment Variables Configuration

TG-FileStream uses environment variables to configure Telegram access, server behavior, performance tuning, and access control.

## 1. Core & Application Environment Variables

| Variable | Required / Default | Description |
|----------|-------------------|------------|
| `API_ID` | ✅ | App ID from https://my.telegram.org |
| `API_HASH` | ✅ | API hash from https://my.telegram.org |
| `BOT_TOKEN` | ✅ | Bot token from https://t.me/BotFather |
| `BIN_CHANNEL` | ✅ | Channel ID where files sent to the bot are stored |
| `DB_BACKEND` | ✅ | Database backend: `mongodb` or `mysql` |
| `HOST` | `0.0.0.0` | Host address to bind the server |
| `PORT` | `8080` | Port to run the server |
| `PUBLIC_URL` | `https://0.0.0.0:8080` | Public-facing URL used to generate download links |
| `DEBUG` | `False` | Enable extra logging |
| `CONNECTION_LIMIT` | `5` | Connections per DC for a single client |
| `DOWNLOAD_PART_SIZE` | `1048576 (1MB)` | Bytes requested per chunk |
| `NO_UPDATE` | `False` | Disable replying to bot messages |
| `SEQUENTIAL_UPDATES` | `False` | Handle Telegram updates sequentially |
| `FILE_INDEX_LIMIT` | `10` | Files shown per `/files` command |
| `MAX_WARNS` | `3` | Maximum warns before banning a user |
| `ADMIN_IDS` | `None` | Comma-separated admin user IDs |
| `ALLOWED_IDS` | `None` | Comma-separated allowed user IDs |
| `MULTI_TOKENx` | Optional | Multiple Telegram bot tokens (`MULTI_TOKEN1`, `MULTI_TOKEN2`, etc.) |

### Database Configuration

You must configure database variables depending on the selected `DB_BACKEND`.

---

#### If Using MySQL

| Variable | Required | Description |
|----------|----------|------------|
| `MYSQL_HOST` | ✅ | MySQL server hostname |
| `MYSQL_PORT` | ✅ | MySQL server port |
| `MYSQL_USER` | ✅ | MySQL username |
| `MYSQL_PASSWORD` | ✅ | MySQL password |
| `MYSQL_DB` | ✅ | Database name |
| `MYSQL_MINSIZE` | `1` | Minimum connection pool size |
| `MYSQL_MAXSIZE` | `5` | Maximum connection pool size |

---

#### If Using MongoDB

| Variable | Required | Description |
|----------|----------|------------|
| `MONGODB_URI` | ✅ | MongoDB connection URI |
| `MONGODB_DBNAME` | `TGFS` | MongoDB database name |

---

## 2. Detailed Explanation of Variables

### Telegram Credentials

#### API_ID
App ID of your Telegram application.  
Obtained from https://my.telegram.org under "API Development Tools".

#### API_HASH
App Hash of your Telegram application.  
Obtained from https://my.telegram.org under "API Development Tools".

#### BOT_TOKEN
Token generated via @BotFather.  
Used to authenticate your bot.

#### BIN_CHANNEL
The Telegram channel ID which is used to share files between multiple bots.  
Must be the full numeric ID (usually starts with `-100`).  
The bot must be an admin in this channel.

---

### Server Configuration

#### HOST
IP address the server binds to.  
Use `0.0.0.0` to allow external access.

#### PORT
Port where the application runs.

#### PUBLIC_URL
The public domain used to generate download links.  
When using Nginx + SSL, this should be your domain.

Example:
```
https://yourdomain.com
```

#### DEBUG
Enables verbose logging.  
Set to `False` in production.

---

### Performance Settings

#### CONNECTION_LIMIT
Number of connections created per Telegram Data Center for a single client.  
Higher values increase performance but use more memory.

#### DOWNLOAD_PART_SIZE
Size of each chunk requested from Telegram (in bytes).  
Default is `1048576` (1MB).  
Increasing this may improve speed but increases memory usage.

#### MULTI_TOKENx
Defines multiple Telegram bot tokens.  
Each token runs as a separate client to reduce flood wait errors.

Example:
```
MULTI_TOKEN1=...
MULTI_TOKEN2=...
```

---

### Bot Behavior Settings

#### NO_UPDATE
If set to `True`, the bot will not reply to incoming messages.

#### SEQUENTIAL_UPDATES
If `True`, Telegram updates are processed sequentially instead of concurrently.

#### FILE_INDEX_LIMIT
Maximum number of files shown when using the `/files` command.

#### MAX_WARNS
Number of warnings allowed before banning a user.

#### ADMIN_IDS
Comma-separated Telegram user IDs allowed to use admin commands.

#### ALLOWED_IDS
If set, only these user IDs can interact with the bot.

---

## 4. Important Notes
- Add all MULTI_TOKENx Bots to BIN_CHANNEL
- Never expose `API_HASH`.
- Never share `BOT_TOKEN`.
- Restrict `ADMIN_IDS` carefully.
