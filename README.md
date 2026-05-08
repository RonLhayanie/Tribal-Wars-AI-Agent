# Tribal Wars AI Strategic Agent

An advanced logic engine and AI-powered tactical advisor for the "Tribal Wars" strategy game. This agent acts as a personal military consultant, processing real-time game data to provide optimized strategic decisions.

## Key Features
- **AI-Powered Analysis:** Integrated with Google Gemini 2.5 Flash to provide context-aware strategic advice.
- **Automated Report Parsing:** A custom extraction engine that maps raw game reports into structured data.
- **Precision Timing Tools:** Algorithms for calculating complex launch windows (Backtiming) and travel durations.
- **Dynamic Memory Layer:** Persistent storage for coordinates, enemy data, and tactical notes using a JSON-based database.
- **Operational Reminders:** An asynchronous background task system for mission-critical notifications.

## Tech Stack
- **Language:** Python 3.x
- **AI Engine:** Google Generative AI (Gemini API)
- **Interface:** Telegram Bot API (via python-telegram-bot)
- **Image Processing:** PIL (Pillow) for visual report analysis.
- **Environment Management:** python-dotenv for secure API key handling.

## Strategic Logic
The agent utilizes a "Strict Array Mapping" algorithm to decode game unit icons and counts, ensuring 100% accuracy in combat simulations. It also features a custom Catapult & Nuke calculator based on game-specific mechanical data.

## Security
This project uses environment variables for all sensitive keys. API tokens are managed via a `.env` file which is excluded from version control.