# Windows (not working yet)

## sqlserver
r_profile::database::sql_server::install_media: 'E:/' # path to DVD installation media

## IIS


## Git site

```yaml
r_profile::webapp::git_site::sites:
  'c:/inetpub/wwwroot':
    source: 'https://github.com/GeoffWilliams/asp_noticeboard'
```
