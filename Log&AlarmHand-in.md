Hand-in for LSD
==============
Logging, Alarms, post mortem report.

### Logging
We have setup logging, see Logging [Elk stack](139.59.157.29:5601) on our [HackerNews Github Wiki](https://github.com/DanielHauge/HackerNews-Grp8/wiki/ELK-Stack)

[![https://gyazo.com/712280fefb2068e37c84188079262ed4](https://i.gyazo.com/712280fefb2068e37c84188079262ed4.png)](https://gyazo.com/712280fefb2068e37c84188079262ed4)

#### Alternative
We have are currently in the process of trying to shift our logging to a more oldschool way of logging to save money on the ELK stack droplet. We have considered to just log to flatfile and use logrotate to manage those logfiles.

To follow progress on this, follow our github wiki documentation on the matter at: [Logging Flatfile](https://github.com/DanielHauge/HackerNews-Grp8/wiki/Logging---Flatfile-(Logrotate))

### Alarms
In our [Grafana](http://207.154.213.133:3000) we have set up an alarm on the number of messages in RabbitMQ channel, if it gets too big, that means the database inserters cannot follow the rate at which they come in. Also if no data is available, that means the server is down and needs to get up as fast as possible.
we have implemented a LINE alarm that alarms us through a phone app. We also have a general alarm set up for [Slack](https://slack.com/), that post all alarm states to that channel.

*Screen shots of the alarms in Grafana:*
[![https://gyazo.com/88443bb6a289f5419c69e47ab64159bf](https://i.gyazo.com/88443bb6a289f5419c69e47ab64159bf.png)](https://gyazo.com/88443bb6a289f5419c69e47ab64159bf)

*Screen shots of SMS alarms we received from Grafana:*
[![alarm](https://cdn.discordapp.com/attachments/352040342327132163/380309387551440896/image.png)](https://cdn.discordapp.com/attachments/352040342327132163/380309387551440896/image.png)

### Crashing the system
We intentionally crashed our system, with too many requests to see comments on a thread with alot of comments. This will crash the Web API (gowebapi) if being done in very quick succession. 17:20 - 11/13

Our operators discovered the outage X hours later.
```Note: the operators have not yet reported and/or noticed the chrash, X's will be replaced by actual values when the operators report it```

### Post-mortem report

#### Summary
GoWebApi crashed and wasn't up for use.
- Affected users: Website user
- outage time: From 17:20 - 11/13 to XX:XX - XX/XX (XX hours)
```Note: the operators have not yet reported and/or noticed the chrash, X's will be replaced by actual values when the operators report it```
#### Detailed description
As per 17:20 - 11/13 the GoWebApi docker container exited with an exit code of 2. This means the website can no longer contact the GoWebApi to fetch data, login, create a user, submit posts or practically anything.

#### Root cause
The GoWebApi crashed because of a big amount of requests to see comments on a thread with a lot of comments, this means that the goapi needs to query the database for a lot of information for every request to be displayed to the user. This information can take a fair bit of time to query for. This means, that it was starting up a lot requests for a lot of data many times, so many times the database threw an error saying there were too many connections to the database. Because it couldn't finish fetching all the data fast enough. This meant the gowebapi crashed.

#### The fix
```When the operators notice the chrash```

We start up the docker container again.

#### Lessons learned

- To never intentionally chrash a docker container which needs to run for the system to be functional.
- To deny requests if the database is to busy or to optimize the way the query is run or upgrade the hardware for the database to work faster.
- To implement a restart functionality, where it will restart if chrashing.
