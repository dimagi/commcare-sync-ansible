What's Installed
================

The following are installed by default.

| Name          | Description / Reason |
| ------------- | ----------- |
| Python 3.8    | Running commcare-sync |
| Postgres      | Default database that houses all data (including configuration details). The default version is version 10. |
| CommCare Sync | Django process for web application, and Celery process for background and scheduled tasks |
| Superset      | BI Tool |
| Redis         | Used as a message broker between CommCare Sync’s web and Celery processes |
| Nginx         | Web Server that sits in front of the CommCare Sync and Superset |
| Supervisord   | Process management for CommCare Sync, Celery, and Superset |

The above will all be configured, and after install, commcare-sync should be properly set up.

Note that some of these can be enabled/disabled based on variables. For example, setting `superset_enabled: no`
in your environment will prevent Superset from being installed.
