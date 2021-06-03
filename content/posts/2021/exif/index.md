---
title: Exif Data
date: '2021-06-02'
spoiler: Adding exif data to my photo application
---

I have decided to update my photo site to have exif data below each picture.
This will require about 65 GB of data egress from S3 to run `exiv2` on each
original image file.

To start, I grabbed a small folder, in my case `rwanda-2018` with:

```sh
aws s3 sync s3://bucket/rwanda-2018 rwanda-2018
```

Then I took a look at the exif data to see what I actually wanted to store.

The tool used, `exiv2` can be intalled with `brew install exiv2`.

[Here's a link to the example image with exif data](https://photos.dylan.is/rwanda-2018/IMG_1768.jpeg)

```sh
exiv2 -p a IMG_1768.jpeg
```

```text
Exif.Image.Make                              Ascii       6  Apple
Exif.Image.Model                             Ascii       9  iPhone 7
...
Exif.Image.DateTime                          Ascii      20  2018:11:15 07:32:58
...
Exif.Photo.PixelXDimension                   Long        1  4032
Exif.Photo.PixelYDimension                   Long        1  3024
...
Exif.GPSInfo.GPSLatitudeRef                  Ascii       2  South
Exif.GPSInfo.GPSLatitude                     Rational    3  1deg 39' 28"
Exif.GPSInfo.GPSLongitudeRef                 Ascii       2  East
Exif.GPSInfo.GPSLongitude                    Rational    3  30deg 6' 42"
```

At this point, I can see that the image has information that I'd like displayed.

Namely, the make, model, resolution, and GPS coordinates of the photo.

---

So the next step was to determine how to store this information:

1. RDS would cost about $13/month to run from some quick napkin math.
2. A `_exif` file stored in S3 that is called along with every image via `fetch`
  was unattractive as:
    * It would require javascript which this photo site mostly avoids
    * It would be slow, and not available on initial render
    * Every single image would have to request ~20 per page for every user
3. Sqlite3 local database (spoiler: I went with this one)

---

I ended up going with a local sqlite3 database, it's dead simple and is no nonsense.

So here's the quick schema I came up with:

```sql
create table images (
  id integer primary key,
  -- The name of the album, such as turkey-2020
  album text not null,
  -- The img file name, such as IMG_1414.jpeg
  file text not null,

  -- Do not allow duplicates
  -- So turkey-2020/IMG_2020.jpeg conflicts with turkey-2020/IMG_2020.jpeg
  -- But not rwanda-2018/IMG_2020.jpeg
  unique (album, file)
);
```

I decided to split exif data from image data in case I ever wanted to extend in the future
with more information such as tags.

Some might have gone with a fully flat table where `images` also stored `exif` data, but
it just didn't feel right. On a small application like this (especially where there's only
a one-to-one data mapping) that's just personal preference, and I wouldn't say one way
was absolutely correct.

```sql
create table exif (
  id integer primary key,

  -- I truncated fields that aren't being used on the actual application
  -- ...
  datetime timestamp,
  -- ...
  gps_altitude text,
  gps_altitude_ref text,
  gps_latitude text,
  gps_latitude_ref text,
  gps_longitude text,
  gps_longitude_ref text,
  -- ...
  make text,
  model text,
  pixel_x_dimension text,
  pixel_y_dimension text,
  -- ...

  image integer,
  foreign key (image) references images (id) on delete cascade
);

```

---


I figured I would be frequently dropping the database while doing development so
I wrote a quick helper python script that runs the above commands called `setup-db.py`.

It uses `better-sqlite3` and just runs those exact inserts.

Then I began work on an `insert.py` script that will run `pyexiv2` on each image
in an album. This one was a little more bothersome, just due to the busy work of
typing out 25 columns.

The exact keys that we want are the same as the `exiv2` output above.

Parsing dates was a little tricky as the exif dates come with `:` separators for
year:month:day. I have no idea why?

```python
  # Get is a helper and will return None or the value
  date = get(data, 'Exif.Image.DateTime'),
  if date:
      try:
          # Uganda pictures are coming back as a tuple
          if isinstance(date, tuple):
              date = date[0]
          date = datetime.strptime(
              date,
              '%Y:%m:%d %H:%M:%S'
          )
      except:
          # Inserting null is fine
          date = None
```

After syncing every album folder from S3, I ran the insert script on every
album with:

```sh
for f in $(ls /tmp/photos); do python3 scripts/insert.py /tmp/photos/$f; done
```

And it nicely inserted 6003 images with all of their exif data like I was
hoping for.

Both scripts are in the [GitHub repo](https://github.com/dylanarmstrong/photos).

---

Searching for images then is a pretty simple SQL query:

```sql
select
  i.file,
  e.gps_latitude,
  e.gps_latitude_ref,
  e.gps_longitude,
  e.gps_longitude_ref,
  e.datetime,
  e.make,
  e.model,
  e.pixel_x_dimension,
  e.pixel_y_dimension
from images i
join exif e
on
  i.id = e.image
where
  album = ?
```

---

I updated the server to populate an exif cache for each album on load.

Ending up with a nice cache that looks like this:

```js
exifCache[albumName][fileName] = {
  make: 'Apple',
  model: 'iPhone 7',
  // ...
};
```

So on every page request, I get the exif information for each image.

```js
// Images is Array<String>
const data = images
  .slice((page - 1) * imagesPerPage, page * imagesPerPage)
  .map((image) => {
    // rwanda-2018/IMG_1768.jpeg -> IMG_1768.jpeg
    const file = image.split('/')[1];
    // This can be undefined
    const exif = exifCache[album][file];
    return {
      exif,
      // Force a boolean, so we don't show a figcaption when exif is empty
      hasExif: !!exif,
      image: `https://${domain}/${image}`,
      thumb: `https://${domain}/${image.replace(/\./, '_thumb.')}`,
    };
  });
```

And that's really about all there was to it.

This can be seen [here](https://dylan.is/photos) with
[source code](https://github.com/dylanarmstrong/photos).
