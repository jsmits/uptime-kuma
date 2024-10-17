# Instructions to deploy to Fly

## Set up the app

```shell
$ fly launch --dockerfile Dockerfile
Creating app in /Users/jsmits/d/dev/repos/github.com/jsmits/uptime-kuma
Using dockerfile Dockerfile
? Choose an app name (leave blank to generate one): uptime-kuma-smits-dev
automatically selected personal organization: Sander Smits
Some regions require a paid plan (bom, fra, maa).
See https://fly.io/plans to set up a plan.

? Choose a region for deployment: Amsterdam, Netherlands (ams)
App will use 'ams' region as primary

Created app 'uptime-kuma-smits-dev' in organization 'personal'
Admin URL: https://fly.io/apps/uptime-kuma-smits-dev
Hostname: uptime-kuma-smits-dev.fly.dev
? Would you like to set up a Postgresql database now? No
? Would you like to set up an Upstash Redis database now? No
Wrote config file fly.toml
? Would you like to deploy now? No
Validating /Users/jsmits/d/dev/repos/github.com/jsmits/uptime-kuma/fly.toml
Platform: machines
✓ Configuration is valid
Your app is ready! Deploy with `flyctl deploy`
```

## Create a volume

```shell
$ fly volumes create uptime_kuma --region ams --size 1
Warning! Individual volumes are pinned to individual hosts. You should create two or more volumes per application. You will have downtime if you only create one. Learn more at https://fly.io/docs/reference/volumes/
? Do you still want to use the volumes feature? Yes
        ID: vol_q4qllp6mokq9n26r
      Name: uptime_kuma
       App: uptime-kuma-smits-dev
    Region: ams
      Zone: f6b7
   Size GB: 1
 Encrypted: true
Created at: 13 Aug 23 11:22 UTC
```

### Add it to `fly.toml`

```
[mounts]
  source = "uptime_kuma"
  destination = "/app/data"
```

## Deploy

```shell
$ fly deploy
```

If configuration is changed, just run this command again.

## Use custom domain

Create CNAME record that points to uptime-kuma-smits-dev.fly.dev

For example:

```
CNAME uptime.smits.dev. uptime-kuma-smits-dev.fly.dev.
```

Create the certificate:

```
$ flyctl certs create uptime.smits.dev
```

## Configure

Now go to https://uptime.smits.dev to configure it.

## Backup

Create the backup directory, for example:

```
$ mkdir -p /Users/jsmits/d/backup/uptime-kuma/20241017201800
```

Issue a new SSH certificate and add it to the SSH agent:

```
$ fly ssh issue --agent
```

Download the `kuma.db*` files from the mounted `/app/data` volume in the `uptime-kuma` container:

```
$ fly ssh sftp get /app/data/kuma.db /Users/jsmits/d/backup/uptime-kuma/20241017201800/kuma.db
$ fly ssh sftp get /app/data/kuma.db-shm /Users/jsmits/d/backup/uptime-kuma/20241017201800/kuma.db-shm
$ fly ssh sftp get /app/data/kuma.db-wal /Users/jsmits/d/backup/uptime-kuma/20241017201800/kuma.db-wal
```

### Use locally

To run an `uptime-kuma` container with these files, ideally using the same version as deployed on `fly`, do something like this:

```
$ docker run -it --rm -p 3001:3001 -v /Users/jsmits/d/backup/uptime-kuma/20241017201800:/app/data elestio/uptime-kuma:1.23.13
```
