---
courseTitle: "How Notion Notes Are Fetched and Rendered"
courseDescription: "A comprehensive guide on how the CMS fetches Notion page blocks, protects against rate limits, caches with multi-tier Redis and RAM, and renders interactive documentation with React Notion X."
tag: "how to"
title: "Smart Navigation and Anchor Scrolling"
order: 6
---

## 1. Overview

In this chapter, you will learn how the CMS intercepts internal Notion links, translates them into seamless Single Page Application (SPA) routes, and provides smooth scrolling with highlight animations.

By the end of this chapter, you will understand:
- Why raw Notion links break when rendered inside a custom web app.
- How `resolveNotionTarget` climbs block `parent_id` hierarchies to find target chapters.
- How to intercept link clicks while respecting native browser shortcuts (e.g. `Cmd+Click`).
- How to scroll smoothly with fixed navbar offsets (84px) and trigger Notion's signature yellow flash animation.

---

## 2. The Problem

Authors frequently link to other chapters or specific headings within their Notion workspace. 

When Notion renders an internal link, it creates an anchor tag with a raw Notion URL:
`<a href="https://notion.so/my-workspace/Advanced-Caching-8f2a1b9e...">Read More</a>`

### The Broken State: External Redirects and Hidden Headings

```tsx
// ❌ NAIVE APPROACH: Standard Browser Links
function NaiveRenderer({ recordMap }) {
  // If a user clicks an internal Notion link:
  // 1. They are booted out of your app and redirected to notion.so (asking them to log in!)
  // 2. If it's a hash anchor (#blockId), the browser jumps instantly, hiding the heading
  //    directly BEHIND your fixed 80px navbar!
  return <NotionRenderer recordMap={recordMap} />;
}
```

This causes two major UX problems:
1. **Broken App Experience**: Users get booted from your website to `notion.so` or experience a slow, jarring full-page browser refresh.
2. **The Obscured Anchor Bug**: Standard browser hash jumping (`#heading`) aligns elements to `top: 0`, leaving headings completely hidden underneath your fixed navigation bar.

### The Root Cause

> **So the root cause is:**
> Notion links contain workspace URLs and raw block UUIDs that know nothing about your application's client-side router (`react-router`) or your layout's fixed sticky navbar.

### So How Can We Solve This?

We implement:
1. A **Custom Link Interceptor** that replaces standard `<a>` tags in `NotionRenderer`.
2. A **Hierarchy Resolver** that climbs `parent_id` pointers in the `recordMap` to match sub-blocks to chapter routes.
3. An **Offset Scroll Handler** with reflow-triggered flash highlight animations.

---

## 3. Core Concept & Architecture

### Link Interception and Hierarchy Resolution Flow

```mermaid
sequenceDiagram
    participant User
    participant Link as CustomLink
    participant Resolver as resolveNotionTarget
    participant Router as React Router
    participant Scroller as scrollToBlockElement

    User->>Link: Clicks Notion Link (#block-id)
    Link->>Link: Check Cmd/Ctrl key (pass through if true)
    Link->>Resolver: Resolve UUID in recordMap
    Resolver->>Resolver: Climb parent_id tree until Chapter is found
    Resolver-->>Link: Returns /course-slug/chapter-slug#block-id
    Link->>Router: navigate() without full page reload
    Router->>Scroller: Calculate (window.scrollY + rect.top - 84px)
    Scroller->>User: Smooth scroll & pulse yellow flash animation
```

### First-Principles Chain of Thought

1. **Why do we need to climb `parent_id`?**
   Suppose an author creates a link pointing directly to a sub-block (like a specific callout box or code snippet). The link URL contains the *block's* UUID, not the *chapter's* UUID. By traversing up `block.parent_id` in the `recordMap`, our resolver discovers which chapter owns that block and routes the user to the correct chapter!

2. **Why must we check `e.metaKey` and `e.ctrlKey`?**
   Power users frequently use `Cmd+Click` (macOS) or `Ctrl+Click` (Windows/Linux) to open documentation chapters in a new tab. If you run `e.preventDefault()` without checking modifier keys, you break standard browser tabs.

3. **Why do we need `void element.offsetWidth`?**
   When triggering CSS keyframe animations, if a class was already applied, simply removing and adding it in the same JavaScript tick will not replay the animation. Reading `element.offsetWidth` forces the browser to calculate layout (reflow), successfully restarting the flash highlight.

### As a human, what questions should I ask to learn this?
- *"How do I find a Notion block in the DOM?"* (Notion blocks have class names like `.notion-block-[cleanId]` or data attributes like `data-id="[cleanId]"`).
- *"How do we prevent infinite loops when climbing parent blocks?"* (Keep a `Set<string>` of visited block IDs).

---

## 4. Code & Implementation

### 1. Hierarchy Resolver (`slug.ts`)

```typescript
// frontend/src/lib/slug.ts
export function resolveNotionTarget({
  rawUrlOrId,
  chaptersData,
  subjectName,
  currentChapterName,
  recordMap
}: {
  rawUrlOrId: string;
  chaptersData: any[] | undefined;
  subjectName: string;
  currentChapterName?: string;
  recordMap?: any;
}) {
  // Extract 32-character ID from path or URL
  const match = rawUrlOrId.match(/([a-f0-9]{8}-?[a-f0-9]{4}-?[a-f0-9]{4}-?[a-f0-9]{4}-?[a-f0-9]{12}|[a-f0-9]{32})/i);
  const extractedId = match ? match[0].replaceAll("-", "").toLowerCase() : null;

  let targetChapter = chaptersData?.find(
    (ch) => ch.pageId?.replaceAll("-", "").toLowerCase() === extractedId
  );

  // If not a direct chapter, climb parent_id hierarchy in recordMap
  if (!targetChapter && extractedId && recordMap?.block) {
    let block = recordMap.block[extractedId]?.value;
    const visited = new Set<string>();

    while (block && !visited.has(block.id)) {
      visited.add(block.id);
      const parentId = block.parent_id?.replaceAll("-", "").toLowerCase();
      targetChapter = chaptersData?.find((ch) => ch.pageId?.replaceAll("-", "").toLowerCase() === parentId);
      if (targetChapter) break;
      block = recordMap.block[parentId]?.value;
    }
  }

  const subjectSlug = subjectName.toLowerCase().replace(/[^a-z0-9]/g, "-");
  const chapterSlug = targetChapter ? targetChapter.chapterName.toLowerCase().replace(/[^a-z0-9]/g, "-") : currentChapterName;

  return {
    destination: `/${subjectSlug}/${chapterSlug}${extractedId ? `#${extractedId}` : ""}`,
    targetChapter
  };
}
```

### 2. Custom Link & Scroll Handler (`Noteui.tsx`)

```tsx
// frontend/src/pages/note/Noteui.tsx
import { useCallback, useEffect } from "react";
import { useNavigate, useLocation } from "react-router";
import { resolveNotionTarget } from "@/lib/slug";

export function useNotionNavigation(data: any, subjectName: string, chapterName?: string) {
  const navigate = useNavigate();
  const location = useLocation();

  // Scroll with 84px fixed navbar offset + yellow flash animation
  const scrollToBlock = (element: HTMLElement) => {
    const navbarOffset = 84;
    const rect = element.getBoundingClientRect();
    const targetY = window.scrollY + rect.top - navbarOffset;

    window.scrollTo({ top: Math.max(0, targetY), behavior: "smooth" });

    // Restart Notion flash animation via forced reflow
    element.classList.remove("notion-block-highlighted");
    void element.offsetWidth; // Force reflow
    element.classList.add("notion-block-highlighted");

    setTimeout(() => element.classList.remove("notion-block-highlighted"), 2400);
  };

  // Intercept Notion links
  const CustomLink = useCallback(
    ({ href, children, ...props }: any) => {
      if (!href) return <a {...props}>{children}</a>;

      const isInternal = href.includes("notion.so") || href.startsWith("/") || href.startsWith("#");

      if (isInternal) {
        return (
          <a
            href={href}
            onClick={(e) => {
              // Respect Cmd+Click / Ctrl+Click for new tab
              if (e.button !== 0 || e.metaKey || e.ctrlKey || e.shiftKey) return;
              e.preventDefault();

              const { destination } = resolveNotionTarget({
                rawUrlOrId: href,
                chaptersData: data?.chaptersData,
                subjectName,
                currentChapterName: chapterName,
                recordMap: data?.recordMap,
              });

              navigate(destination);
            }}
            {...props}
          >
            {children}
          </a>
        );
      }

      return <a href={href} target="_blank" rel="noopener noreferrer" {...props}>{children}</a>;
    },
    [data, subjectName, chapterName, navigate]
  );

  return { CustomLink, scrollToBlock };
}
```

---

## 5. Key Gotchas

- **Breaking Native Browser Gestures**: Always permit default behavior if `e.metaKey`, `e.ctrlKey`, `e.shiftKey`, or middle click (`e.button !== 0`) are triggered.
- **Timing on Block Rendering**: When navigating to a new chapter with a `#hash`, the target DOM node might not exist on the initial render. Use a small `setTimeout` (200–300ms) or `requestAnimationFrame` before querying the DOM element.
- **Circular Block Trees**: Always maintain a `visited = new Set<string>()` when climbing `parent_id` pointers to guard against corrupted or cyclical block references.

---

## 6. Summary Checklist

- [ ] Override `Link` and `PageLink` in `NotionRenderer` with a custom link handler.
- [ ] Climb `block.parent_id` in `recordMap` to map deep block links to the correct chapter route.
- [ ] Respect `Cmd+Click` and `Ctrl+Click` so users can open links in new tabs.
- [ ] Offset anchor scrolling by 84px to prevent content from hiding behind fixed sticky navigation bars.
- [ ] Force DOM reflow with `void element.offsetWidth` to replay the CSS flash highlight reliably.
