# Salvador

A sleek, user-friendly interface for OpenAI's DALL·E image generation API, built with SvelteKit. Generate AI artwork with simple prompts using either DALL·E 2 or DALL·E 3.

## Features

- Support for both DALL·E 2 and DALL·E 3 models
- Multiple image size options
- Local storage of previous prompts
- (Optional) personal API key integration
- Mobile-first, responsive design

## How It Works

Salvador Dalle provides a clean interface where users can:

1. Select their preferred DALL·E model (2 or 3)
2. Choose image dimensions from the supported dimensions for each model
3. Enter a text prompt (which is stored on device) to generate an image

Users can either use the application's built-in server backend (with some limitations) or provide their own OpenAI API key for unlimited and direct access to DALL.E.

## Getting Started

To run your own version of Salvador:

1. Clone this repository

```bash
git clone https://github.com/gianlucatruda/salvador.git
cd salvador
```

2. Install dependencies

```bash
npm install
```

3. Create a `.env` file in the root directory and add your OpenAI API key:

```
OPENAI_API_KEY=<sk-...>
```

4. Start the development server

```bash
npm run dev
```

5. Visit `http://localhost:5173` in your browser

## Building for Production

```bash
npm run build
```

## Technical Stack

- SvelteKit
- TypeScript
- Vite
- OpenAI API

## Environment Variables

- `OPENAI_API_KEY`: Your OpenAI API key (required for server-side operations)

## Created By

[Gianluca Truda](https://gianluca.ai)

---

Here's a technical overview of how the Svelte front-end handles API requests for OpenAI's DALL-E:

### Request Routing Logic

1. **API Key Check**

```typescript
let requestURL = "/api/generate";
if (apiKey != null && apiKey != "") {
  requestURL = "https://api.openai.com/v1/images/generations";
}
```

- The application first checks if a user-provided API key exists
- If no API key exists, requests are routed to the local server endpoint `/api/generate`
- If an API key exists, requests go directly to OpenAI's endpoint

### Direct OpenAI Request (With User's API Key)

When a user provides their own API key:

1. The key is stored securely in the browser's localStorage
2. Requests are made directly to `https://api.openai.com/v1/images/generations`
3. The user's API key is included in the Authorization header
4. This allows unlimited requests and access to all features

```typescript
const response = await fetch(requestURL, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${apiKey}`,
  },
  body: JSON.stringify(bodyData),
});
```

### Server-Side Proxy (Without API Key)

When no user API key is provided:

1. Requests go to `/api/generate` endpoint on the Svelte server
2. The server uses its own API key (stored in environment variables)
3. The server acts as a proxy, forwarding the request to OpenAI
4. The response is sent back to the client

```typescript
// +server.ts
export async function POST({ request }) {
  const url = "https://api.openai.com/v1/images/generations";
  const bodyData = await request.json();

  const response = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${OPENAI_API_KEY}`,
    },
    body: JSON.stringify(bodyData),
  });

  return json(await response.json());
}
```

### Security Benefits

1. The server's API key (`OPENAI_API_KEY`) is never exposed to the client
2. User API keys are stored locally and only sent directly to OpenAI
3. The proxy server provides rate limiting and access control
4. No sensitive credentials are included in the client-side bundle

This architecture provides a secure and flexible way to handle both authenticated and anonymous users while protecting API credentials.
