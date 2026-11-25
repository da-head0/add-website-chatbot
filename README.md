# Add an AI chatbot to a Lovable, Bolt, v0, Next.js or Vite site with Cursor or Claude Code

> **Status:** this is a free, vendor-neutral skill that works today with tawk.to, Crisp, Tidio, Chatbase, Channel Talk, or any chat widget that ships as a script snippet. It's maintained by the person building [OneLineChat](https://onelinechat.com/?utm_source=github&utm_medium=readme&utm_campaign=add-website-chatbot), an agent-installable chatbot that is **in early access**.

Most hosted chatbots install as one `<script>` tag. AI coding agents still get it wrong in predictable ways. They put it in a page instead of the root layout, so it disappears on navigation. They load it twice. They make up a widget ID. They forget the CSP. Or the vendor blocks the preview domain. This repo gives Cursor and Claude Code a short, checked procedure for doing it right:

- [`skills/add-website-chatbot/SKILL.md`](skills/add-website-chatbot/SKILL.md): the full procedure (an Agent Skill)
- [`rules/website-chatbot.mdc`](rules/website-chatbot.mdc): the same rules condensed into a Cursor rule

## Install

**Claude Code**

```
/plugin marketplace add da-head0/add-website-chatbot
/plugin install add-website-chatbot@add-website-chatbot
```

Or copy just the skill into your project:

```bash
mkdir -p .claude/skills/add-website-chatbot
curl -fsSL https://raw.githubusercontent.com/da-head0/add-website-chatbot/main/skills/add-website-chatbot/SKILL.md \
  -o .claude/skills/add-website-chatbot/SKILL.md
```

**Cursor**

Install it from [cursor.directory](https://cursor.directory/plugins), or copy the rule into your project:

```bash
mkdir -p .cursor/rules
curl -fsSL https://raw.githubusercontent.com/da-head0/add-website-chatbot/main/rules/website-chatbot.mdc \
  -o .cursor/rules/website-chatbot.mdc
```

Then ask: *"Add my tawk.to widget to this site"* and paste the vendor's snippet.

## Where the script goes

| Project | File | How |
|---|---|---|
| Next.js App Router (and v0) | `app/layout.tsx` | `next/script` inside `<body>`, after `{children}`. Inline snippets need an `id` |
| Next.js Pages Router | `pages/_app.tsx` | `next/script` after `<Component {...pageProps} />` |
| Vite: React, Vue, Svelte (and most Lovable / Bolt projects) | root `index.html` | paste before `</body>` |
| Create React App | `public/index.html` | paste before `</body>` |
| Plain HTML | every page or the shared footer | paste before `</body>` |
| WordPress | vendor plugin, or a header/footer code plugin | Footer box, then purge cache |
| Webflow | Site settings → Custom code → Footer code | publish; needs a paid site plan |

### Next.js example

```tsx
// app/layout.tsx
import Script from 'next/script';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}
        {/* external snippet: <script src="..."> */}
        <Script src="https://VENDOR_URL_FROM_THE_SNIPPET" strategy="lazyOnload" />
        {/* or an inline snippet (tawk.to, Crisp, ...): the id is required */}
        {/* <Script id="chat-widget" strategy="lazyOnload">{`...vendor JavaScript...`}</Script> */}
      </body>
    </html>
  );
}
```

### Lovable / Bolt / v0 prompt

> Add this script tag to index.html, just before the closing body tag. Change nothing else. It's a public widget key, so keep it inline: `<PASTE THE VENDOR SNIPPET>`

For v0, ask it to use `next/script` in `app/layout.tsx` instead.

## Where to get each vendor's snippet

| Vendor | In the dashboard | Docs |
|---|---|---|
| tawk.to | Administration → Chat Widget → Widget Code | [guide](https://help.tawk.to/article/adding-a-widget-to-your-website) |
| Crisp | Settings → Workspace Settings → Setup & Integrations → Chatbox setup instructions | [guide](https://help.crisp.chat/en/article/how-to-install-crisp-live-chat-software-on-a-custom-website-10wcj3l/) |
| Tidio | Settings → Live Chat → Installation → Manual install | [guide](https://help.tidio.com/articles/install-website/), [Next.js](https://help.tidio.com/articles/install-nextjs/) |
| Chatbase | Channels → Chat Bubble → Manage → Deploy → Website widget | [guide](https://chatbase.co/docs/user-guides/chatbot/channels.md) |
| Channel Talk | install guide | [guide (KO)](https://docs.channel.io/help/ko/articles/채널톡-설치하기-1bfa136b) |

Menu paths were checked against each vendor's docs in November 2026. Vendors rename menus, so trust the linked guide if they differ.

## When the widget doesn't show up

- **CSP**: add the vendor's domains to `script-src`, `connect-src` (often `wss:` too), `img-src`, `style-src`, `frame-src`. Get the list from the vendor's docs or the console errors.
- **Allowed domains**: add `localhost` and preview domains (`*.vercel.app`, `*.lovable.app`, `*.netlify.app`) in the vendor dashboard.
- **Shows twice**: the snippet is in two places.
- **Gone after navigation**: it was added to a page component, not the root layout.
- **Ad blocker**: test in a clean browser profile first.

## 한국어 요약

러버블(Lovable), 볼트(Bolt), v0, 커서(Cursor), Claude Code로 만든 사이트에 AI 챗봇이나 상담 위젯(채널톡, tawk.to, Crisp, Tidio, Chatbase 등)을 붙일 때 쓰는 스킬입니다.

- Next.js는 `app/layout.tsx`에 `next/script`로, Vite 기반인 러버블과 볼트는 루트 `index.html`의 `</body>` 바로 앞에 넣습니다.
- 위젯 ID는 업체 대시보드에서 복사한 값을 씁니다. AI가 지어내면 안 됩니다.
- 위젯이 안 보이면 CSP, 허용 도메인(미리보기 주소 포함), 중복 삽입을 확인합니다.

## About OneLineChat

I'm building [OneLineChat](https://onelinechat.com/?utm_source=github&utm_medium=readme&utm_campaign=add-website-chatbot), a website chatbot meant to be installed by a coding agent without anyone opening a dashboard. If that's something you'd use, you can [request early access](https://onelinechat.com/get-api-key?src=gh-skill&utm_source=github&utm_medium=readme&utm_campaign=add-website-chatbot). Otherwise, everything above works with the vendors you already use.

## License

MIT
