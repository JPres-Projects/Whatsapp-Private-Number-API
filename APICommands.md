# WhatsApp API Server - Windows Commands

## Server Setup
- **Port:** 8080
- **Compile:** `go build main.go`
- **Run:** `./main`

## Available Endpoints

### 1. Server Status
```cmd
curl.exe -X GET http://localhost:8080/api/status
```

### 2. List Chats
```cmd
curl.exe -X GET http://localhost:8080/api/chats
```

### 3. Search Contacts
```cmd
curl.exe -X GET "http://localhost:8080/api/contacts?search=491"
```

### 4. Get Messages
```cmd
curl.exe -X GET "http://localhost:8080/api/messages?chat_jid=4915510974808@s.whatsapp.net&limit=20"
```

### 5. Send Text Message
```cmd
curl.exe -X POST http://localhost:8080/api/send -H "Content-Type: application/json" -d "{\"recipient\":\"4915510974808\",\"message\":\"Hello!\"}"
```

### 6. Send Image/Video/Document
```cmd
curl.exe -X POST http://localhost:8080/api/sendFile -F "recipient=4915510974808" -F "caption=Test Image" -F "file=@C:/Users/stand/Pictures/image.png"

curl.exe -X POST http://localhost:8080/api/sendFile -F "recipient=4915510974808" -F "caption=Test Video" -F "file=@C:/Users/stand/Videos/video.mp4"

curl.exe -X POST http://localhost:8080/api/sendFile -F "recipient=4915510974808" -F "caption=Document" -F "file=@C:/Users/stand/Documents/file.pdf"
```

### 7. Send Audio/Voice Message
```cmd
curl.exe -X POST http://localhost:8080/api/sendAudio -F "recipient=4915510974808" -F "audio=@C:/Users/stand/Music/audio.ogg"
```

### 8. Download Media
```cmd
curl.exe -X POST http://localhost:8080/api/download -H "Content-Type: application/json" -d "{\"message_id\":\"MESSAGE_ID_HERE\",\"chat_jid\":\"4915510974808@s.whatsapp.net\"}"
```

## File Formats & Limits

### Images → ImageMessage
- `.jpg, .jpeg, .png, .gif, .webp`
- Max: 5 MB

### Videos → VideoMessage  
- `.mp4, .avi, .mov`
- Max: 16 MB

### Audio → AudioMessage (Voice)
- `.ogg` (Opus codec) - **BEST for voice messages**
- `.m4a, .aac, .amr`
- Max: 16 MB

### Documents → DocumentMessage
- All other files (`.pdf, .txt, .doc, etc.`)
- Max: 100 MB

## Response Format
```json
{
  "success": true,
  "message": "Operation completed",
  "data": {...}
}
```

## Quick Test Sequence
```cmd
curl.exe -X GET http://localhost:8080/api/status
curl.exe -X GET http://localhost:8080/api/chats
curl.exe -X POST http://localhost:8080/api/send -H "Content-Type: application/json" -d "{\"recipient\":\"4915510974808\",\"message\":\"API Test\"}"
```

**Note:** Replace `localhost:8080` with your domain if using Cloudflare tunnel.