##

# Issues 
Mattermost is different than the other containers ive built since its a platform utilized for internal business communcation, like Slack.

I have current found my vaules through 
```bash
$ podman search docker.io/mattermost/mattermost-team-edition && podman inspect docker.io/mattermost/mattermost-team-edition
```

I was able to find values such as
    - Image name
    - database
    -volume mounts
    - port(s)

mattermost also needs access to postgresql so have to remember to add ENVFILE 
    -POSTGRES_USERNAME
    -POSTGRES_PASSWORD



mattermost was difficult for me because theyre documentation wasnt as straightforward as others. Feels like a real world situation, having to read an applications documentation file for the necessary values needed to be set. Since I am using Podman, I aso have to remember to correlate their Docker instructions tothe correct Podman Values

found a template on [mattermost](https://docs.mattermost.com/administration-guide/configure/configuration-in-your-database)

```txt
[Unit]
Description=Mattermost
After=network.target
After=postgresql.service
Requires=postgresql.service

[Service]
Type=notify
EnvironmentFile=/opt/mattermost/config/mattermost.environment
ExecStart=/opt/mattermost/bin/mattermost
TimeoutStartSec=3600
KillMode=mixed
Restart=always
RestartSec=10
WorkingDirectory=/opt/mattermost
User=mattermost
Group=mattermost
LimitNOFILE=49152

[Install]
WantedBy=postgresql.service
```



took about 10 - 15 mins of skimming but I found the page with vaules needed to be set for database connection (here)[https://docs.mattermost.com/administration-guide/configure/environment-configuration-settings#database]
I will need 
    -MM_SQLSETTINGS_DRIVERNAME: Telling mattermost we'll be using postgres for our database
    -MM_SQL SETTINGS_DATASOURCE: telling mattermost these are the vaules needed to access postgres account

When setting driver to postgres, need to use connection string postgres://mmuser:password@hostname_or_IP:5432/mattermost_test?sslmode=disable&connect_timeout=10

making executive decision to make a dedicated .env for this build due to mattermost having variables I want to utilize down the line. Instead of having to go through the entire doc section, will create brief explation with the envirment values hashed out for ones I wont be using
