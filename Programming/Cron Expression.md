# Cron Expression
cron is a system utility for scheduling tasks to be run at certain intervals, these tasks (**cron jobs**) can be run as either the system, or as an individual user. There are many different implementations of cron, but they are take a similar format, a cron expression for a time, followed by the command to be executed. Each line in this file (**crontab**) is its own cron job, these jobs will be run through the cron users shell. An examples for a couple of cron jobs are below:
```bash
# This job will execute every 5 minutes where it appends to a file
*/5 * * * * echo "it's been 5 minutes!" >> /tmp/cronjob.txt
0 12 0 4 * echo "run at 12:00 on first of april"
```

## Basics
Every line of the crontab file is a different job, they consist of 5 white-space separated fields followed by a shell command and then a newline. A `#` symbol can be place at the start of a line to turn it into a comment. The basic format is as follows below:
```
# ┏━━ minute (0-59)
# ┃ ┏━━ hour (0-23)
# ┃ ┃ ┏━━ day of month (1-31)
# ┃ ┃ ┃ ┏━━ month (1-12)
# ┃ ┃ ┃ ┃     _alt. JAN-DEC_
# ┃ ┃ ┃ ┃ ┏━━ day of the week (0-6)
# ┃ ┃ ┃ ┃ ┃     _alt. SUN-SAT
# * * * * * <command>
```
Within each of the fields there are valid ranges and special characters that can be used to control when it matches. `*` will match with any time. `,` can be used to list values that will match (`3,4` matches 3 and 4). `-` can be used to match on a range of values, inclusive (`3-5` matches 3, 4 and 5). There is also the step value which will match division of the legal range (`*/2` match every second value).
| **Symbol** | **Meaning**          |
|------------|----------------------|
| *          | Match anything       |
| ,          | List value separator |
| -          | Range of values      |
| /          | Step value           |

### Non-standard Additions
As with most POSIX tools there are non-standard additions, these are used as shortcuts for common cron expressions. Even though it was included above, use of the step operator (`/`) is actually not POSIX compliant, it is however supported by most cron daemons. Below is a list of these common non-standard shortcuts:
| **Alias**           | **Meaning**                       | **Equivalent** |
|---------------------|-----------------------------------|----------------|
| @yearly             | Once a year on Jan 1st            | 0 0 1 1 *      |
| @monthly            | On each first minute of the month | 0 0 1 * *      |
| @weekly             | On each midnight Sunday morning   | 0 0 * * 0      |
| @daily or @midnight | Every day at midnight             | 0 0 * * *      |
| @hourly             | At the start of every hour        | 0 * * * *      |
| @reboot             | When the cron daemon is started   | None           |
