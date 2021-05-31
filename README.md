### Blog

[dylan.is/blogging](https://dylan.is/blogging)

### Required

Install [hugo](https://gohugo.io).

### Building

```
rm -rf public && hugo -b https://dylan.is/blogging -e production
```

### Development

```
hugo -p 4000 -b http://localhost/blogging/ -D server
```

### Deployment

```
rsync -avP --delete-before public/ https://server.example/blogging/
```

### License

The underlying source code for this blog is provided under the
[ISC](LICENSE) license.

All files under the directory `content/` are not included 
under this license, and all rights are reserved to me.
This includes the posts, photos, videos, and any other
accompanying files.
