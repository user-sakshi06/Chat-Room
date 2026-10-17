# Chat-Room
Real time Chat Application
# Chat Room App — Client Side (Participant 1)

## Role
Participant 1 — Client Side Developer

## Tech Stack
| Layer        | Technology                      |
|-------------|----------------------------------|
| Frontend    | HTML5, CSS3, Vanilla JavaScript  |
| Backend     | Node.js + Express.js             |
| Real-time   | Socket.IO (Client)               |
| Auth        | bcryptjs + JWT                   |
| HTTP Proxy  | axios                            |
| Storage     | JSON Files (clients.json)        |

## Ports
- **Client Backend: `http://localhost:3000`**
- **Admin Backend (required): `http://localhost:4000`**

## Folder Structure
```
client-side/
├── server.js               # Entry point
├── routes/
│   ├── authRoutes.js       # POST /api/auth/register, /login, /logout
│   ├── roomRoutes.js       # Proxy → Admin: /rooms, /joinRoom, /leaveRoom, /participants
│   └── chatRoutes.js       # Proxy → Admin: /messages/:roomId
├── modules/
│   ├── authModule.js       # registerUser, loginUser, logoutUser, validateUser
│   ├── httpModule.js       # All HTTP calls to Admin Backend (axios)
│   └── dbHelper.js         # readJSON, writeJSON
├── middleware/
│   └── authMiddleware.js   # clientAuth JWT middleware
├── data/
│   └── clients.json        # Registered user accounts
└── public/
    ├── index.html          # Client UI (Auth + Rooms + Chat)
    ├── css/style.css
    └── js/
        ├── socket.js       # Socket module (connect, join, send, listen)
        └── app.js          # Main app logic (all client modules)
```

## Client Modules (in app.js)
| Module              | Functions                                                     |
|---------------------|---------------------------------------------------------------|
| Auth Module         | handleRegister, handleLogin, handleLogout                     |
| Room Module         | fetchRooms, displayRooms, joinRoom, leaveRoom                 |
| Chat Module         | loadMessages, displayMessages, appendMessage, handleSendMessage |
| Participant Module  | fetchParticipants, fetchActiveUsers, displayParticipants      |
| HTTP Module         | All fetch() calls through Client Backend proxy                |
| Socket Module       | connectSocket, joinSocketRoom, sendSocketMessage, listenMessages |

## Socket Events (socket.js)
| Direction       | Event        | Description                     |
|-----------------|--------------|---------------------------------|
| Client → Admin  | joinRoom     | Join a socket room              |
| Client → Admin  | leaveRoom    | Leave a socket room             |
| Client → Admin  | sendMessage  | Send a chat message             |
| Client → Admin  | typing       | User started typing             |
| Client → Admin  | stopTyping   | User stopped typing             |
| Admin → Client  | receiveMessage    | New message broadcast      |
| Admin → Client  | userJoined        | User joined notification   |
| Admin → Client  | userLeft          | User left notification     |
| Admin → Client  | roomClosed        | Room closed by admin        |
| Admin → Client  | activeUsersUpdated| Live online user list       |
| Admin → Client  | typingIndicator   | Typing status update        |

## User Workflow
```
Register / Login
      ↓
Fetch Available Rooms (from Admin via HTTP)
      ↓
Select & Join Room (HTTP → Admin records membership)
      ↓
Load Previous Chat History (HTTP → Admin)
      ↓
Connect Socket + joinSocketRoom
      ↓
Send / Receive Messages (via Socket.IO)
      ↓
Leave Room
      ↓
Logout
```

## How to Run
```bash
# 1. Start Admin Backend FIRST (on port 4000)
cd admin-side && npm start

# 2. Start Client Backend (on port 3000)
cd client-side && npm start
# App available at http://localhost:3000
```
# Chat Room App — Admin Side (Participant 2)

## Role
Participant 2 — Admin Side Developer

## Tech Stack
| Layer        | Technology                        |
|-------------|-----------------------------------|
| Frontend    | HTML5, CSS3, Vanilla JavaScript   |
| Backend     | Node.js + Express.js              |
| Real-time   | Socket.IO (Server)                |
| Auth        | bcryptjs + JWT                    |
| Storage     | JSON Files (admins, rooms, chats) |

## Port
**Admin Backend runs on: `http://localhost:4000`**

## Folder Structure
```
admin-side/
├── server.js               # Entry point
├── routes/
│   ├── authRoutes.js       # POST /api/auth/login, /logout
│   ├── roomRoutes.js       # GET/POST/PUT/DELETE /api/rooms
│   ├── memberRoutes.js     # POST /api/joinRoom, /leaveRoom | GET /participants, /activeUsers
│   └── chatRoutes.js       # GET/POST/DELETE /api/messages
├── modules/
│   ├── authModule.js       # loginAdmin, logoutAdmin, validateAdmin
│   ├── roomModule.js       # createRoom, updateRoom, deleteRoom, closeRoom
│   ├── memberModule.js     # joinRoom, leaveRoom, getParticipants, activeUsers
│   ├── chatModule.js       # saveMessage, getMessages, deleteRoomMessages
│   └── dbHelper.js         # readJSON, writeJSON
├── middleware/
│   └── authMiddleware.js   # adminAuth JWT middleware
├── socket/
│   └── socketHandler.js    # Socket.IO event handler (all real-time logic)
├── data/
│   ├── admins.json         # Admin credentials
│   ├── rooms.json          # Room definitions
│   ├── roomMembers.json    # Room membership records
│   └── chats.json          # Persisted chat messages
└── public/
    ├── index.html          # Admin Dashboard UI
    ├── css/style.css
    └── js/app.js
```

## HTTP API Endpoints
| Method   | Endpoint                      | Auth     | Description              |
|----------|-------------------------------|----------|--------------------------|
| POST     | /api/auth/login               | None     | Admin login              |
| POST     | /api/auth/logout              | Admin JWT| Admin logout             |
| GET      | /api/rooms                    | None     | List all rooms           |
| GET      | /api/rooms/:roomId            | None     | Get single room          |
| POST     | /api/rooms                    | Admin JWT| Create room              |
| PUT      | /api/rooms/:roomId            | Admin JWT| Update room              |
| DELETE   | /api/rooms/:roomId            | Admin JWT| Delete room              |
| POST     | /api/joinRoom                 | None     | Add member to room       |
| POST     | /api/leaveRoom                | None     | Remove member from room  |
| GET      | /api/participants/:roomId     | None     | Get room members         |
| GET      | /api/activeUsers/:roomId      | None     | Get online users         |
| GET      | /api/messages/:roomId         | None     | Fetch chat history       |
| DELETE   | /api/messages/:roomId         | Admin JWT| Clear room messages      |

## Socket Events
| Direction       | Event                | Payload                                    |
|-----------------|----------------------|--------------------------------------------|
| Client → Admin  | joinRoom             | { roomId, userId, username }               |
| Client → Admin  | leaveRoom            | { roomId, userId, username }               |
| Client → Admin  | sendMessage          | { roomId, userId, username, message }      |
| Client → Admin  | typing               | { roomId, username }                       |
| Client → Admin  | stopTyping           | { roomId, username }                       |
| Admin → Client  | receiveMessage       | { messageId, roomId, userId, username, message, timestamp } |
| Admin → Client  | userJoined           | { userId, username, timestamp }            |
| Admin → Client  | userLeft             | { userId, username, timestamp }            |
| Admin → Client  | roomClosed           | {}                                         |
| Admin → Client  | activeUsersUpdated   | { activeUsers: [...] }                     |
| Admin → Client  | typingIndicator      | { username, isTyping }                     |

## Default Admin Credentials
- **Email**: admin@chatroom.com
- **Password**: admin123

## How to Run
```bash
cd admin-side
npm install
npm start
# Server starts on http://localhost:4000
```

