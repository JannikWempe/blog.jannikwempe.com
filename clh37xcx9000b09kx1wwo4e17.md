---
title: "Create Your Personal, Pay-Per-Use ChatGPT Client in Minutes"
datePublished: Sun Apr 30 2023 09:37:37 GMT+0000 (Coordinated Universal Time)
cuid: clh37xcx9000b09kx1wwo4e17
slug: personal-chatgpt
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1682847180351/28c62519-991c-4ad9-9338-b0433a524e07.png
tags: ai, programming, web-development, webdev, chatgpt

---

I was hesitant to purchase ChatGPT Pro. While I often use ChatGPT, the $20 per month price tag seemed excessive for my needs. However, the slow response time and limited availability were frustrating. I appreciate the serverless, pay-per-use approach, as that's what I was seeking. Thankfully, [Chatbot UI](https://github.com/mckaywrigley/chatbot-ui) offers a solution!

In this article, I'll explain how to host your own ChatGPT UI, allowing you to experience faster response times and pay only for what you use.

## About ChatBot UI

> Chatbot UI is an open source chat UI for AI models.  
> – [ChatBot UI GitHub Repository](https://github.com/mckaywrigley/chatbot-ui)

ChatBot UI is a NextJS app and has an interface that is familiar if you have used ChatGPT:

![Chatbot UI](https://github.com/mckaywrigley/chatbot-ui/raw/main/public/screenshots/screenshot-0402023.jpg align="left")

It has features like different chats, saving prompt templates, switching AI models (e.g. `gpt-3.5-turbo`) and more. You just provide some configs like your Open API key and are good to go.

One feature that is missing though is saving chats and prompts to a database. Data is currently stored in local storage. But the APIs are quite nice and you could easily add a database yourself.

I won't go into details on configuration here. The [Readme.md](https://github.com/mckaywrigley/chatbot-ui/blob/main/README.md) is the single source of truth and will have the up to date instructions.

You could leave the `OPENAI_API_KEY` environment variable blank and have the user specify the key but I don't want anybody else to use my UI at all. That is why we are now looking into how to protect your ChatBot UI website.

## Adding BasicAuth

I am using the most basic way to protect a website: [Basic Auth.](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)

Basic auth is a simple HTTP authentication scheme used to protect web resources that require user authentication. It works by sending the combination of a username and password, encoded in base64 format, as plain text over the network (you should use HTTPS). It is good enough for this use case.

I am deploying my version of the ChatBot you to Vercel. [There is an example from Vercel on GitHub on how to add Basic Auth](https://github.com/vercel/examples/tree/main/edge-middleware/basic-auth-password). It all comes down to using an edge middleware and an API route. This is my slightly modified version of the middleware using environment variables to provide values for `user` and `pwd`:

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(req: NextRequest) {
  const basicAuth = req.headers.get('authorization');
  const url = req.nextUrl;

  if (basicAuth) {
    const authValue = basicAuth.split(' ')[1];
    const [user, pwd] = atob(authValue).split(':');

    if (
      user === process.env.BASIC_AUTH_USER &&
      pwd === process.env.BASIC_AUTH_PASSWORD
    ) {
      return NextResponse.next();
    }
  }
  url.pathname = '/api/auth';

  return NextResponse.rewrite(url);
}
```

It checks the provided `user` and `pwd` if the `authorization` header is provided and allows the requests to pass through if it is correct or otherwise rewrites the response to the `/api/auth` URL:

```typescript
// pages/api/auth.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default function handler(_: NextApiRequest, res: NextApiResponse) {
  res.setHeader('WWW-authenticate', 'Basic realm="Secure Area"');
  res.statusCode = 401;
  res.end(`Auth Required.`);
}
```

This is just setting a header and a `401` (Unauthorized) status code.The `WWW-authenticate` field is used to initiate the authentication process, and the value `Basic realm="Secure Area"` instructs the client to send encoded username and password credentials to access the "Secure Area" of the website.

You just have to add those two files to your ChatBot UI code and provide the `BASIC_AUTH_USER` and `BASIC_AUTH_PASSWORD` to add Basic Auth to your own version of ChatGPT.

## Conclusion

It is quite straightforward to deploy your own version of ChatGPT with fast response times without paying $20 per month. This can be accomplished in less than 10 minutes, and it also provides a solid starting point for customizing everything to your liking without having to code everything from scratch.

*Note: Check out* [*Hashnode Rix – The AI-powered chatbot for developers*](https://hashnode.com/rix/general) *for making development-specific prompts. It is also trained on various open-source documentation.*