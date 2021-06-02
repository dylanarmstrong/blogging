---
title: New Photos
date: '2021-06-01'
spoiler: My photos application could be better
---

I updated my [photos site](https://dylan.is/photos) and added several new countries.
The layout of the index page has been adjusted too, so it's a little easier to read.

I also clarified the license on my blog and photo site, I realized it could be misleading
that I was granting an ISC license on the source code, but not on the actual content
of the demo or my blog posts.

Since I think I'm the only one who knows the photo site exists, I doubt it really matters.

---

I didn't clarify it in last post, but I also updated neovim configuration entirely to lua.
This was mostly a time wasting exercise and I'm not sure I've found any benefit to it.

I don't want my neovim to be an IDE, and I still occasionally use vim without any
configuration on some servers.

I tried writing my first lua plugin, but I really need the autocmd functionality to be
merged in first.

The only nifty thing is being able to do my keybindings like so:

```lua
local maps = {
  i = {
    ['jj'] = '<Esc>',
  },
  -- ...
}

for mode, mappings in pairs(maps) do
  for keys, mapping in pairs(mappings) do
    vim.api.nvim_set_keymap(mode, keys, mapping, { noremap = true })
  end
end
```

As always, source code is available [here](https://github.com/dylanarmstrong/dotfiles).

I really want a new side project idea that I can do in lua. It's a really fun language.

---

Truncated this blog post, as I'm writing a much longer post about updating the photos
application itself.
