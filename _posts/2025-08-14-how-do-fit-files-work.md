# How do .FIT files work?

I was on a run the other day and I started wondering how my GPS watch records my run. It's one of
those technologies I've used for years without really thinking about it.

Since the data gets synced between my Garmin watch and my Strava account, I assume it must be some
file format. If it's a file format, then there must be a spec, right?

After doing a bit of research, I discovered that Garmin's native data format is .FIT, which stands
for **F**lexible and **I**nteroperable Data **T**ransfer.

.FIT files contain the following data:

- Date & time
- Sport types
- Lap & split data
- GPS track
- Sensor data
- Events for active session

Here's an interesting diagram from the Garmin developer documentation:

![diagram](/img/how-do-fit-files-work/decode-activity-multisport.png)

This digram surprised me because I had never considered that you could record multiple
sports in one session. But if you're participating in a something like a triathalon, that's pretty important.

It's the GPS track that I'm most interested in, because I notice the pace calculation (i.e. minutes per mile) on my watch
is jumpy and inaccurate in certain cases.  I notice the pace calculation is off &mdash; usually slower than it should be &mdash; when I am rounding corners or going up or down hills. This makes sense because it's calculating acceleration from what I
assume is a GPS vector, which is probably pretty noisy itself.

GPS data is stored in the part of .FIT files called Records. According to the [docs](https://developer.garmin.com/fit/file-types/activity/):

> Record messages are where the moment-by-moment GPS coordinate, speed, distance, heart rate, power,
> etc. values are stored within Activity files. Records contain timestamps with a one-second resolution,
> although devices may store data at a lower rate.

Each record essentially a tuple of (timestamp, latitude, longitude), though it can store other metadata such as altitude, heart rate, etc.

The interesting thing is that latitude and longitude are stored as a unit called **semicircles** instead of degrees. Semicircles encode 180 degrees into an int32 type, so they have much better resolution than degrees while avoiding the annoying rounding errors you get with float types. Plus with semicircles, there are only four bytes per coordinate.

Since records can also store altitude data, you could in theory calculate a 3D vector in real time from a stream of records containg latitude, longitude, and altitude data. But since the processing power of my smartwatch is limited, I wonder how sophisticated its calculations really are. Perhaps it's just using a 2D model?

At any rate, pace estimates are just estimates. It was interesting to look into the .FIT file format. I'm struck by the fact that writing good specifications is not easy, and yet good specifications can create useful value in the real world!