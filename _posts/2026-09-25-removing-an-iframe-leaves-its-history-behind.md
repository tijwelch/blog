---
layout: post
title: "Removing an Iframe Leaves Its History Behind"
date: 2026-09-25
tags: [browsers, iframes, react, history-api, frontend]
---

The browser has one Back button, so it keeps one history list. When a page embeds an iframe and the user clicks around inside it, those clicks go onto the same list as the parent page's navigations. That part makes sense. What surprised me: if you throw the iframe away and build a new one, the old frame's entries don't go with it. They stay in the list, pointing at a frame that no longer exists, and each one costs the user a Back press that does nothing.

This came up while Tim was embedding a separately hosted app into a React single-page app. The embed page had a small header with a Reload button. The obvious React way to reload an iframe is to bump a counter in its `key`. That unmounts the old `<iframe>` element and mounts a new one:

```tsx
<Fragment key={generation}>
  <iframe src={frameSrc} />
</Fragment>
```

It looks correct and it passes every test that checks what's on screen. A reviewer subagent, sent to try the finished PR in a real browser before merge, pressed Back a few times and wrote down what happened:

```
before Reload   frame=/app/~/campaigns/42
after Reload    frame=/app/~
back #1         frame=/app/~     (dead)
back #2         frame=/app/~     (dead)
back #3         top=/lists
```

The user had made two clicks inside the app, so the old frame had pushed two entries. After the remount, both are still in the list. Back pops the first one, the browser looks for the frame that owns it, finds nothing, and the screen stays put. Then the second. Only the third press gets the user out. To them the Back button just looks broken.

![The parent page and the iframe share one history list; after the iframe is replaced, its entries remain as dead Back presses](/blog/assets/images/removing-an-iframe-leaves-its-history-behind.svg)
*The frame is new. Its history entries are not.*{: .caption}

The reviewer also ruled out two easy explanations. First, it isn't about this particular app. The same thing happens on a bare page that removes an iframe with `pushState`. Second, giving the iframe a stable `name` doesn't help. The browser doesn't tie the new frame to the old frame's history.

This problem isn't specific to a Reload button. Anything that remounts the iframe does the same thing: a `key` that changes with the URL, or leaving the page and coming back. The earlier fix in this same PR had keyed the frame on its `src` to fix a stale-URL bug, and that fix brought in a quieter version of the Back problem.

If you really need to reload, navigate the existing frame instead of replacing the element:

```ts
frame.contentWindow?.location.replace(frameSrc)
```

Setting `location` on a cross-origin frame is one of the few things the same-origin policy allows. `replace` changes the current entry instead of adding a new one. The reviewer checked this version: after a reload, Back walks the app's own history in order, then leaves, with no dead presses.

Tim took the simpler route and deleted the button. The embedded app's own standalone shell never had a Reload button, and the browser's refresh already does the job. The case with no local fix, where the user leaves the app and then presses Back, went into the next ticket. The long-term answer there is a small message bridge: the app replaces its own history, and the parent adds one entry per navigation it hears about.

I think this bug is worth writing up because of how it hides. React's model is that the DOM is disposable: throw away the element, render a new one, same result. That's true for everything you can see. Browser history isn't part of the DOM, so it doesn't get thrown away. Component tests can't catch it, and neither can screenshots. You only find it by pressing Back more than once.
