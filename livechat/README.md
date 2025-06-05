# Livechat

A simple real-time chat application using WebSockets, built with a Node.js backend and a Vue 3 + TypeScript + Vite frontend.

---

## Features

- Real-time chat using WebSockets
- Broadcasts messages to all connected clients except the sender
- Welcome/info messages on connect/disconnect
- Modern Vue 3 frontend with TypeScript
- Easy local development and testing

---

## Project Structure

```
livechat/
  server.js           # Node.js WebSocket server
  package.json        # Server dependencies
  client/             # Vue 3 + Vite frontend
    src/              # Vue components and assets
    public/           # Static assets
    index.html        # App entry point
    package.json      # Client dependencies
```

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (v23 or newer recommended)
- npm (comes with Node.js)

---

### 1. Setup & Run the Server

```sh
cd livechat
npm install
node server.js
```

The WebSocket server will start on `ws://localhost:8080`.

---

### 2. Setup & Run the Client

```sh
cd livechat/client
npm install
npm run dev
```

The frontend will be available at [http://localhost:5173](http://localhost:5173) (default Vite port).

---

## Usage

1. **Start the server** (`node server.js`).
2. **Start the client** (`npm run dev` in `client/`).
3. Open [http://localhost:5173](http://localhost:5173) in two or more browser windows/tabs.
4. Type messages and see them appear in real time across all connected clients.

---

## Customization

- **WebSocket server port:** Change the port in [`server.js`](server.js).
- **Frontend WebSocket URL:** Update the `ws://localhost:8080` URL in [`Chat.vue`](client/src/components/Chat.vue) if needed.

---

## Scripts

### Server

- `npm install` — Install server dependencies
- `node server.js` — Start the WebSocket server

### Client

- `npm install` — Install client dependencies
- `npm run dev` — Start Vite dev server
- `npm run build` — Build for production
- `npm run preview` — Preview production build

---