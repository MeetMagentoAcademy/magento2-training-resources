# Magento Infrastructure Workshops #10

This part of workshops will be dedicated to configuring cron jobs for Magento.

## Cron jobs

Magento 2 requires some [cron
jobs](http://devdocs.magento.com/guides/v2.1/config-guide/cli/config-cli-subcommands-cron.html).

With `which php7.0` command we can locate the path to PHP CLI interpreter,
which we’ll use in crontab.

We will modify crontab with `crontab -u supershop -e` and we’ll insert the
following lines there:
```
* * * * * </path/to/php> /home/supershop/public_html/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /home/supershop/public_html/var/log/magento.cron.log
* * * * * </path/to/php> /home/supershop/public_html/update/cron.php >> /home/supershop/public_html/var/log/update.cron.log
* * * * * </path/to/php> /home/supershop/public_html/bin/magento setup:cron:run >> /home/supershop/public_html/var/log/setup.cron.log
```
