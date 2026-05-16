# 🎨 headless-wp-starter

> **Headless WordPress + Next.js starter kit with WPGraphQL, ISR, custom Gutenberg blocks, and Vercel deployment.**

[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178c6?style=flat&logo=typescript&logoColor=white)](https://typescriptlang.org)
[![Next.js](https://img.shields.io/badge/Next.js-14-000000?style=flat&logo=next.js&logoColor=white)](https://nextjs.org)
[![WordPress](https://img.shields.io/badge/WordPress-6.4+-21759b?style=flat&logo=wordpress&logoColor=white)](https://wordpress.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Overview

The fastest way to build a headless WordPress site with Next.js. Everything is wired up — WPGraphQL queries, Incremental Static Regeneration, custom Gutenberg block rendering, preview mode, and one-click Vercel deployment.

Keep WordPress as your CMS. Get all the performance benefits of a static Next.js frontend.

---

## Features

- ⚡ **Next.js 14** with App Router and React Server Components
- 🔌 **WPGraphQL** — typed GraphQL API for WordPress
- 📄 **ISR** — pages regenerate on content change via webhook
- 🧱 **Custom block renderer** — maps Gutenberg blocks to React components
- 👁️ **Preview mode** — draft post preview in the frontend
- 🎨 **Tailwind CSS** — utility-first styling
- 📱 **Responsive** — mobile-first layouts out of the box
- 🚀 **Vercel-ready** — one-click deployment config included

---

## Project Structure

```
headless-wp-starter/
├── app/
│   ├── (site)/
│   │   ├── page.tsx              # Homepage
│   │   ├── [slug]/page.tsx       # Pages
│   │   └── blog/
│   │       ├── page.tsx          # Blog index
│   │       └── [slug]/page.tsx   # Single post
│   ├── api/
│   │   ├── revalidate/route.ts   # ISR webhook handler
│   │   └── preview/route.ts      # Preview mode handler
│   └── layout.tsx
├── components/
│   ├── blocks/                   # Gutenberg block components
│   │   ├── Paragraph.tsx
│   │   ├── Heading.tsx
│   │   ├── Image.tsx
│   │   ├── Gallery.tsx
│   │   └── index.tsx             # Block renderer
│   ├── ui/                       # Shared UI components
│   └── layout/                   # Header, footer, nav
├── lib/
│   ├── graphql/
│   │   ├── client.ts             # Apollo/fetch GraphQL client
│   │   └── queries/              # All GraphQL query files
│   └── wordpress.ts              # WordPress API helpers
├── types/
│   └── wordpress.ts              # Generated TypeScript types
└── vercel.json
```

---

## Quick Start

### 1. WordPress Setup

Install required plugins on your WordPress site:
- [WPGraphQL](https://wordpress.org/plugins/wp-graphql/)
- [WPGraphQL for ACF](https://acf.local/add-ons/wpgraphql-for-acf/) (optional)
- [WPGraphQL Offset Pagination](https://github.com/valu-digital/wp-graphql-offset-pagination)

### 2. Next.js Setup

```bash
git clone https://github.com/TradeStackDev/headless-wp-starter.git
cd headless-wp-starter
npm install

cp .env.local.example .env.local
```

Edit `.env.local`:

```env
WORDPRESS_API_URL=https://your-wp-site.com/graphql
WORDPRESS_PREVIEW_SECRET=your-preview-secret
REVALIDATION_SECRET=your-revalidation-secret
```

```bash
npm run dev
```

---

## GraphQL Queries

All queries are in `lib/graphql/queries/`:

```typescript
// lib/graphql/queries/posts.ts
export const GET_POST_BY_SLUG = gql`
  query GetPostBySlug($slug: String!) {
    postBy(slug: $slug) {
      id
      title
      date
      content
      excerpt
      featuredImage {
        node {
          sourceUrl
          altText
          mediaDetails { width height }
        }
      }
      categories {
        nodes { name slug }
      }
      author {
        node { name avatar { url } }
      }
      blocks {
        name
        ... on CoreParagraphBlock { attributes { content } }
        ... on CoreHeadingBlock { attributes { content level } }
        ... on CoreImageBlock { attributes { url alt width height } }
      }
    }
  }
`;
```

---

## Block Renderer

Maps Gutenberg blocks to React components automatically:

```typescript
// components/blocks/index.tsx
const blockComponents: Record<string, React.ComponentType<any>> = {
  'core/paragraph':  Paragraph,
  'core/heading':    Heading,
  'core/image':      Image,
  'core/gallery':    Gallery,
  'core/list':       List,
  'core/quote':      Quote,
  'core/code':       Code,
  'acf/hero':        AcfHero,      // Custom ACF block
  'acf/cta':         AcfCta,
};

export function BlockRenderer({ blocks }: { blocks: Block[] }) {
  return (
    <>
      {blocks.map((block, i) => {
        const Component = blockComponents[block.name];
        if (!Component) return null;
        return <Component key={i} {...block.attributes} />;
      })}
    </>
  );
}
```

---

## ISR + On-Demand Revalidation

Pages are statically generated and revalidate when content changes:

```typescript
// app/(site)/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getAllPostSlugs();
  return posts.map(({ slug }) => ({ slug }));
}

export const revalidate = 3600; // Fallback: revalidate every hour
```

Set up the WordPress webhook to trigger on-demand revalidation:

1. Install [WP Webhooks](https://wordpress.org/plugins/wp-webhooks/) on WordPress
2. Add webhook URL: `https://your-site.vercel.app/api/revalidate`
3. Trigger: Post published / updated / deleted

---

## Preview Mode

```typescript
// app/api/preview/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const secret = searchParams.get('secret');
  const slug   = searchParams.get('slug');

  if (secret !== process.env.WORDPRESS_PREVIEW_SECRET) {
    return new Response('Invalid token', { status: 401 });
  }

  // Enable preview mode and redirect to post
  const response = NextResponse.redirect(new URL(`/blog/${slug}`, request.url));
  response.cookies.set('__prerender_bypass', 'true');
  return response;
}
```

In WordPress, your preview button will open the live Next.js frontend in draft/preview mode.

---

## Deploy to Vercel

```bash
npm i -g vercel
vercel --prod
```

Or connect your GitHub repo to Vercel for automatic deployments on push.

---

## Requirements

- Node.js 18+
- WordPress 6.4+ with WPGraphQL plugin
- Vercel account (for deployment)

---

## License

MIT © Adam Johnson
