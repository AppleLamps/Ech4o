# Ech4o

Ech4o is a polished Next.js chat application powered by OpenRouter. The app
provides a ChatGPT-inspired interface with persistent conversations, a
collapsible sidebar, multimodal uploads, and a secure server-side OpenRouter
proxy.

## Features

- ChatGPT-style light interface with collapsible desktop sidebar and mobile
  sidebar overlay
- Persistent local conversation history with rename and delete actions
- Server-side Ech4o system prompt
- Markdown and GitHub-flavored Markdown response rendering
- OpenRouter chat completions through `app/api/chat/route.ts`
- Default model: `openai/gpt-5.4-nano`
- Text, image, and PDF attachments from local files
- Hosted image and hosted PDF URL attachments
- Multimodal OpenRouter payloads for images and PDFs
- PDF parsing through OpenRouter's `file-parser` plugin
- Response caching via OpenRouter cache headers
- Copy, retry, share/export, voice input, and extended-answer controls

## Tech Stack

- Next.js App Router
- React
- TypeScript
- Tailwind CSS
- OpenRouter Chat Completions API

## Requirements

- Node.js 20
- npm
- OpenRouter API key

## Environment Variables

For local development, create a local environment file:

```bash
cp .env.example .env.local
```

Required:

```bash
OPENROUTER_API_KEY=sk-or-v1-your-key-here
```

Optional:

```bash
OPENROUTER_SITE_URL=https://ech4o.vercel.app
OPENROUTER_APP_TITLE=Ech4o
OPENROUTER_CACHE_TTL_SECONDS=300
```

`OPENROUTER_SITE_URL` and `OPENROUTER_APP_TITLE` are sent as attribution
headers. `OPENROUTER_CACHE_TTL_SECONDS` controls the requested OpenRouter
response-cache TTL and defaults to `300` seconds when omitted.

Do not commit `.env` or `.env.local`. They are ignored by Git and should be
configured in Vercel project settings for production.

## Development

Install dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

Build for production:

```bash
npm run build
```

Start the production server after building:

```bash
npm run start
```

## OpenRouter Integration

The browser calls the local route at `/api/chat`. The API key is only read on
the server from `process.env.OPENROUTER_API_KEY` and is never exposed to the
client.

The server route forwards requests to:

```text
POST https://openrouter.ai/api/v1/chat/completions
```

Request fields sent by the client:

- `model`
- `responseMode`
- `messages`

The server prepends the Ech4o system prompt from `lib/system-prompt.ts`.
Extended mode reserves up to 128,000 output tokens for
`openai/gpt-5.4-nano`. Standard mode uses a smaller output cap and leaves more
of the 400,000-token context window for input.

When a user attaches images or PDFs, the server converts user message content
from plain text to a multimodal content array. Text is sent first, followed by
image and file entries.

Supported local image types:

- `image/png`
- `image/jpeg`
- `image/webp`
- `image/gif`

PDF attachments are sent as OpenRouter `file` content items. Local PDF uploads
are converted in the browser to `data:application/pdf;base64,...`; hosted PDF
URLs are passed through directly.

The route enables OpenRouter response caching with:

- `X-OpenRouter-Cache: true`
- `X-OpenRouter-Cache-TTL`

## Project Structure

```text
app/
  api/chat/route.ts   OpenRouter proxy route
  globals.css         Global styles and markdown formatting
  layout.tsx          App metadata and root layout
  page.tsx            App entry page
components/
  ChatApp.tsx         Main chat UI and client-side state
lib/
  defaults.ts         Default model, system prompt, and starter prompts
```

## Deployment

The app is ready for Vercel with `vercel.json` and a standard Next.js setup.
The production URL is [https://ech4o.vercel.app](https://ech4o.vercel.app).
The project pins Node to `20.x` in `package.json` so Vercel does not
automatically jump to a future major Node release.

1. Push the repository to GitHub.
2. Import the project in Vercel.
3. Add `OPENROUTER_API_KEY` in Vercel project environment variables.
4. Set `OPENROUTER_SITE_URL` to `https://ech4o.vercel.app`.
5. Set `OPENROUTER_APP_TITLE` to `Ech4o`.
6. Add `OPENROUTER_CACHE_TTL_SECONDS` if you want a custom cache TTL.
7. Deploy.

`.vercelignore` excludes local environment files, local build output, dependency
folders, and test artifacts from CLI deployments. GitHub-backed Vercel deploys
will use the committed files only.

## Notes

- Conversation history is stored in browser `localStorage`.
- Sidebar collapsed/expanded state is stored in browser `localStorage`.
- Large conversations are trimmed client-side before sending if they exceed the
  configured large-context character budget.
- The app is intentionally public-facing; usage cost should be controlled from
  the OpenRouter account side.
