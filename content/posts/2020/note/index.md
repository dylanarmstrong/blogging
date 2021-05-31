---
title: Sister, Notes, & Vacation
date: '2020-02-15'
spoiler: Some little thoughts and a python app
---

Sister is engaged! <3

---

I still have to write my Ghana post, but it's taking some time getting
the motiviation to do that.

In the meantime, I wrote a quick little app that I've been meaning to
for a while. It's a python note app for taking quick notes from the
terminal.

It's available at [github](https://github.com/dylanarmstrong/note) under
an MIT license.

```bash
$ note I should find a gatsby plugin that converts emojis automatically
$ cat ~/notes.yml
February 15, 2020:
- (09:51) I should find a gatsby plugin that converts emojis automatically
```

Eh. Cool.

---

I was really hoping to do the Inca trail, but all the tour permits are
sold out for April.

I'm trying to find a tour that goes from April 17th - April 27th. It'd
be a quick, short tour before going to see my sister's graduation on
the 28th.

It's difficult to get the motiviation for any specific trip or tour
as I really had my heart set on doing the Inca trail.

I found a tour that seems cool, it goes north from Costa Rica to Guatemala.
It's 17 days long, and that just seems too long. I will be gone in August
as well for about that long, so it's a non-starter.

---

So this blog is cool, but deploying it is a bit difficult. I've started
tinkering with Docker in the hopes that I can create a docker image that
runs the whole thing. Ideally, I'd just run `rsync` and have the blog
updated.

Unfortunately for the build and run at the moment, I have to run:

```bash
$ rm -rf public .cache && gatsby build --prefix-paths
$ gatsby serve --prefix-paths
```

They are in my package.json, so it's as simple as `yarn build && yarn start`
but it's still going to cause ~5 minutes of downtime.

---

Also, getting an AWS certification. So maybe I'll switch some things over to
their services.

At the moment, I'm using DigitalOcean, and their pricing is pheneomenal.
I am worried about hosting the images here though, I think it might be
better to put them on github or something, and link directly to them.
