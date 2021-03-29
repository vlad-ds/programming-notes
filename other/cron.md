# cron

https://crontab.guru/

Time-based job scheduler. 

 `Crontab` is a central file to keep track of cron jobs. 

```bash
#see all tasks currently scheduled
crontab -l
#add a scheduler to crontab
echo "* * * * *  python create_model.py" | crontab
```

The granularity limit is 60 seconds. 

The components are minute, hour, day of month, month, day of week. 