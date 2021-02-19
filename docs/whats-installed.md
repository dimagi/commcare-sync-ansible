What's Installed
================

The following are installed by default.

| Name          | Description / Reason |
| ------------- | ----------- |
| Postgres 10   | Default database |
| Redis         | Message Broker for Celery |
| Python 3.8    | Running commcare-sync |
| Nginx         | Web Server |
| Supervisord   | Process manager |
| CommCare Sync | Django process for web application, and Celery process for background and scheduled tasks |
| Superset      | BI Tool |

The above will all be configured, and after install, commcare-sync should be properly set up.

Note that some of these can be enabled/disabled based on variables. For example, setting `superset_enabled: no`
in your environment will prevent Superset from being installed.

