# docker-prune

Guidelines to set up a script that will be used by cron to run a `docker system prune` every Sunday at 4:00 AM...

### 🛡️ **"Prune only if disk usage > 80%"**

You create a small **bash script** that:

1.  Checks current disk usage.

2.  If usage is above 80%, then it runs:

    docker system prune -a --volumes -f

### 🛠️ **Script Example**

Create a file like `/usr/local/bin/docker-prune-if-needed.sh`:

```bash
#!/bin/bash

# Set threshold (80%)\
THRESHOLD=80

# Get current disk usage % for /\
USAGE=$(df / | tail -1 | awk '{print $5}' | tr -d '%')

# Check if usage exceeds threshold\
if [ "$USAGE" -ge "$THRESHOLD" ]; then\
    echo "$(date) - Disk usage at ${USAGE}%. Running docker system prune..."\
    /usr/bin/docker system prune -a --volumes -f\
else\
    echo "$(date) - Disk usage at ${USAGE}%. No action needed."\
fi
```

**Notes:**

-   `df /`: checks root partition (where Docker usually stores data).

-   Adjust `/usr/bin/docker` if needed (`which docker` to confirm).

-   Logs to stdout, or you can redirect logs to a file if you want (`>> /var/log/docker-prune.log`).

Then make it executable:

    sudo chmod +x /usr/local/bin/docker-prune-if-needed.sh

### 🕓 **Then in your crontab**

Edit crontab:

    sudo crontab -e

Add:

```bash
0 4 * * 0 /usr/local/bin/docker-prune-if-needed.sh >/dev/null 2>&1
# or if you want to keep a small log:
0 4 * * 0 /usr/local/bin/docker-prune-if-needed.sh >> /var/log/docker-prune.log 2>&1
```
