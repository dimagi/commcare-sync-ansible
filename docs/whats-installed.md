What's Installed
================

The following are installed by default.

| Name                   | Description / Reason                                                       |
|------------------------|----------------------------------------------------------------------------|
| Python 3.14            | Runs CommCare Data Pipeline                                                |
| PostgreSQL             | The database that stores CommCare Data Pipeline's metadata                 |
| CommCare Data Pipeline | A web service, and a Celery process for background tasks                   |
| Superset (optional)    | A BI tool                                                                  |
| Redis                  | A message broker between CommCare Data Pipeline’s web and Celery processes |
| nginx                  | A web server that sits in front of CommCare Data Pipeline and Superset     |
| Supervisor             | Process management for CommCare Data Pipeline, Celery, and Superset        |

> [!note]
> Setting `superset_enabled: no` in your environment will prevent
> Superset from being installed.
