# WhatsApp-MCP: A Self-Hosted WhatsApp API Bridge

This project acts as a self-hosted bridge to the WhatsApp network, allowing you to send and receive messages programmatically from your own phone number. It leverages the powerful `go.mau.fi/whatsmeow` library and exposes a simple REST API for easy integration with your own scripts, bots, or home automation systems.

This allows you to create a "Master Control Program" (MCP) for WhatsApp, controlled entirely by you.

## Core Features

-   **Send & Receive Messages:** Automate sending text messages and media files directly from your own WhatsApp number.
-   **REST API:** A simple, straightforward HTTP API for sending messages. If your script or application can make a POST request, it can integrate with this bridge.
-   **Self-Hosted:** Runs entirely on your own machine. No third-party services, subscriptions, or fees are required. You are in complete control of your data and session.
-   **Media Support:** Send images, audio, videos, and general documents as attachments.
-   **Persistent Session:** After the initial QR code scan, your session is saved locally, allowing the application to reconnect automatically on restart.

## How It Works

The application uses the `whatsmeow` library to connect to WhatsApp's Web servers, simulating a linked device, just like the official WhatsApp Web client.

1.  **Initial Setup:** On the first run, the application will generate a QR code in your terminal.
2.  **Pairing:** You scan this QR code using your phone (in WhatsApp, go to `Settings` > `Linked Devices` > `Link a Device`).
3.  **Connection:** Once paired, the application establishes a persistent connection. Your session credentials are saved locally in a `store` directory (e.g., `whatsapp.db`).
4.  **API Server:** The application simultaneously runs a local HTTP server (by default on port `8080`) that listens for API calls.
5.  **Sending Messages:** When you make a POST request to the `/api/send` endpoint, the application instructs the WhatsApp client to send your message to the specified recipient.

## Use Cases

-   **Home Automation:** Send notifications from Home Assistant, Node-RED, or other systems (e.g., "Garage door was left open!").
-   **Custom Bots:** Create personal reminder bots, information fetchers, or simple command-and-response bots.
-   **System Monitoring:** Get alerts and status updates from your servers, cron jobs, or other scripts directly on WhatsApp.
-   **Script Integration:** Connect any script (Python, Bash, etc.) that can make an HTTP request to your WhatsApp for notifications.

---

## **DISCLAIMER: UNOFFICIAL API**

**This is extremely important:**

This project uses an unofficial method to communicate with WhatsApp. This is against the WhatsApp Terms of Service. There is a **real risk** that your phone number could be temporarily or permanently **banned** by WhatsApp for using applications like this.

**Use this software entirely at your own risk.** It is highly recommended to use a secondary, non-critical phone number for any development or automation. The author and contributors are not responsible for any consequences of using this software.

---

## API Endpoints

The server runs on `http://localhost:8080`.

### Send Message

-   **Endpoint:** `/api/send`
-   **Method:** `POST`
-   **Body (JSON):**
    ```json
    {
      "recipient": "1234567890",
      "message": "Hello from my API!",
      "media_path": "C:\\path\\to\\my\\image.jpg"
    }
    ```
-   `recipient`: The phone number in international format, without `+` or `00`. For groups, use the group JID (e.g., `1234567890@g.us`).
-   `message`: The text content of the message. Required if no media is sent.
-   `media_path`: (Optional) The absolute local path to a media file to send.

**Example using `curl` (Text Message):**
```bash
curl -X POST -H "Content-Type: application/json" -d "{\"recipient\":\"1234567890\", \"message\":\"This is a test.\"}" http://localhost:8080/api/send```

**Example using `curl` (Media Message):**
```bash
curl -X POST -H "Content-Type: application/json" -d "{\"recipient\":\"1234567890\", \"message\":\"Check out this file!\", \"media_path\":\"C:\\Users\\user\\Documents\\report.pdf\"}" http://localhost:8080/api/send