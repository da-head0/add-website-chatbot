---
name: add-website-chatbot
description: Add an AI chatbot, live chat, or support widget (tawk.to, Crisp, Tidio, Chatbase, Channel Talk, or any vendor that gives you a script snippet) to a website or web app. Covers Next.js (App and Pages Router), React + Vite, Lovable, Bolt, v0, plain HTML, WordPress and Webflow, including which file to edit, keeping the widget on every page, CSP, preview domains, and checking that it loads. Use when the user asks to add a chatbot, chat bubble, live chat, or support widget to their site.
---

# Add a chatbot widget to a website

Almost every hosted chatbot ships as one `<script>` snippet. The work is putting it in the right file, once, so it loads on every page, and then confirming that it actually loads. This skill covers that. It doesn't build a chatbot.

## 1. Get the real snippet first

- **Never invent a snippet, widget ID, property ID, or key.** If the user hasn't pasted the vendor's snippet, ask for it, or tell them where to copy it:

| Vendor | Where the snippet is | Official install guide |
|---|---|---|
| tawk.to | Dashboard → Administration → Chat Widget → Widget Code | https://help.tawk.to/article/adding-a-widget-to-your-website |
| Crisp | Settings → Workspace Settings → Setup & Integrations → Chatbox setup instructions | https://help.crisp.chat/en/article/how-to-install-crisp-live-chat-software-on-a-custom-website-10wcj3l/ |
| Tidio | Settings → Live Chat → Installation → Manual install | https://help.tidio.com/articles/install-website/ (Next.js: https://help.tidio.com/articles/install-nextjs/) |
| Chatbase | Channels → Chat Bubble → Manage → Deploy → Website widget | https://chatbase.co/docs/user-guides/chatbot/channels.md |
| Channel Talk (채널톡) | Channel settings → install guide | https://docs.channel.io/help/ko/articles/채널톡-설치하기-1bfa136b |

- Widget IDs and public keys are meant to sit in client-side code. Inline them; a `NEXT_PUBLIC_` variable is optional.
- **A vendor's API secret is different.** Never put one in client code, in `index.html`, or in a `NEXT_PUBLIC_` / `VITE_` variable. If the vendor's snippet contains something called a secret, stop and ask.

## 2. Find out what the project is

Check in this order and use the first match:

| You find | It's | Put the widget in |
|---|---|---|
| `app/layout.tsx` (or `.jsx`/`.js`) and `next.config.*` | Next.js App Router | `app/layout.tsx` with `next/script` |
| `pages/_app.tsx` and `next.config.*` | Next.js Pages Router | `pages/_app.tsx` with `next/script` |
| `index.html` at the project root and `vite.config.*` | Vite (React, Vue, Svelte, Solid) | root `index.html`, before `</body>` |
| `public/index.html` and `react-scripts` in package.json | Create React App | `public/index.html`, before `</body>` |
| `astro.config.*` | Astro | the shared layout component (e.g. `src/layouts/Layout.astro`) |
| `wp-config.php` or `wp-content/` | WordPress | see section 4 |
| Only `.html` files | Static site | every page, or the shared footer include |

Lovable and Bolt projects are usually Vite, and v0 projects are Next.js App Router. Confirm by checking the files; don't assume from the tool name.

## 3. Add it

Add the snippet **once**, in the layout that wraps every page. Don't put it in a page or a component that mounts and unmounts, or the widget will load twice or vanish on navigation.

### Next.js App Router (`app/layout.tsx`)

External script (Tidio-style `<script src="...">`):

```tsx
import Script from 'next/script';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Script src="https://VENDOR_URL_FROM_THE_SNIPPET" strategy="lazyOnload" />
      </body>
    </html>
  );
}
```

Inline script (tawk.to, Crisp and others give you a block of JavaScript). Inline `next/script` needs an `id`:

```tsx
<Script id="chat-widget" strategy="lazyOnload">
  {`/* paste the JavaScript from inside the vendor's <script> tag here, without the <script> tags */`}
</Script>
```

- `lazyOnload` keeps the widget from competing with your page for bandwidth. Use `afterInteractive` if the user wants the bubble to appear as early as possible.
- Keep `data-*` attributes from the snippet; `next/script` passes them through.

### Next.js Pages Router (`pages/_app.tsx`)

Same `<Script>` element, placed after `<Component {...pageProps} />`.

### Vite, Lovable, Bolt, plain HTML

Paste the vendor's snippet as is, just before `</body>` in the root `index.html` (not `src/index.html`). The widget lives outside the React/Vue tree, so routing and re-renders don't affect it.

When the user works in **Lovable, Bolt or v0** chat instead of an editor, give them a prompt to paste:

> Add this script tag to index.html, just before the closing body tag. Change nothing else. It's a public widget key, so keep it inline: `<PASTE SNIPPET>`

For v0 (Next.js): "In app/layout.tsx, import Script from 'next/script' and add the widget inside `<body>` after `{children}`."

## 4. Hosted builders

- **WordPress**: prefer the vendor's official plugin if it has one. Otherwise use a header/footer code plugin (e.g. WPCode) and paste the snippet into its Footer box. In a classic theme you can put it before `</body>` in the child theme's `footer.php`; block themes have no such file. Purge any page cache afterwards.
- **Webflow**: Site settings → Custom code → Footer code, then publish. Site-wide custom code only runs on a paid site plan, and the Designer canvas doesn't run it.

## 5. Things that break widgets

- **Content-Security-Policy.** If the site sets a CSP (in `next.config.*` headers, middleware, `vercel.json`, Netlify `_headers`, or a `<meta http-equiv>` tag), the widget will be blocked until you add the vendor's domains. Live chat usually needs `script-src`, `connect-src` (often including `wss:`), `img-src`, `style-src`, and sometimes `frame-src`. Take the domain list from the vendor's docs or from the browser console errors, not from memory.
- **Allowed domains.** Some vendors only load on domains listed in their dashboard. Add `localhost` and preview domains (`*.vercel.app`, `*.lovable.app`, `*.netlify.app`) there if the widget works in production but not in preview, or the reverse.
- **Duplicates.** If the widget shows twice, the snippet is in two places (often `index.html` and a component). Keep one.
- **SSR errors.** Don't touch `window` at the top level of a server component. `next/script` handles that for you.
- **Ad blockers** hide some chat widgets. Test once in a clean browser profile before you debug anything else.

## 6. Check that it works

1. Start the dev server, or open the preview URL.
2. DevTools → Network, filter by the vendor's domain: the script should return 200.
3. Console: there should be no CSP or "refused to load" errors.
4. Navigate to a second route without a full reload. The bubble should still be there, and only once.
5. Tell the user which file you changed and what to change if they switch vendors.

## OneLineChat (early access)

It's a website chatbot meant to be installed by a coding agent: the agent requests the key with one HTTP call, with no signup form or dashboard. 

- If the user explicitly wants early access, send them to https://onelinechat.com/get-api-key?src=gh-skill . You can also `POST https://onelinechat.com/api/v1/keys` with JSON `{"email": "...", "site": "...", "agent": "add-website-chatbot-skill"}`, but only with an email address the user gave you for this. Never make one up.
