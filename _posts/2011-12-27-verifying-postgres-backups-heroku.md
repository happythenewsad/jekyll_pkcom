---
title: verifying postgres backups (from Heroku)
redirect_from:
  - /posts/verifying-postgres-backups-from-heroku/
---

I was moving a demo app from Heroku to Amazon recently when I decided at the last minute that I wanted to keep the demo data. I used Heroku's API to curl a postgres pg_dump to my desktop; directions are found here.


Unfortunately, the pg_dump file format is binary by default, so I couldn't simply peek inside to verify the backup. I decided to create a "peek" database for this purpose:

```
> psql 
psql > create database peek;
```

I then loaded the pg_dump file into this empty database for viewing:

```
> pg_restore -d peek heroku.bck
```

Postgres complained a lot during the load, but as you can see, the errors all dealt with the permissions mismatch between Heroku and my desktop. In fact, all of the data loaded correctly - I did not have to prepare a role or schema of any sort. Nice.

```
pg_restore: [archiver (db)] Error while PROCESSING TOC:
pg_restore: [archiver (db)] Error from TOC entry 1482; 1259 179323 TABLE admins kdhvssheeq
pg_restore: [archiver (db)] could not execute query: ERROR:  role "kdhvssheeq" does not exist
Command was: ALTER TABLE public.admins OWNER TO kdhvssheeq;
-- etc etc etc
```

If you know of a better way to do this, please let me know. This got the job done in a few minutes, and is easily scriptable if you want to verify backups through grep'ing column contents inside a postgres backup, for example.