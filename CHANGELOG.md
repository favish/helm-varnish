# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.3.0] - 2024-09-28
- Dynamic naming based on the alias. This one solves conflicts when adding this chart as a subchart and you want to provision multi instances.

## [2.2.0] - 2023-01-28
- Cache aggregated CSS and JS files for 1 hour.

## [2.1.1] - 2023-09-28
- Remove the lowercasing of the X-Varnish-Purge header value.  This is not necessary and can cause issues with the secret.

## [2.1.0] - 2023-09-28
- Use secret to authorize BAN requests.

## [2.0.0] - 2023-09-25
- Cache GraphQL POST requests by hashing the request body so that they can be invalidated using cache tags.  

## [1.0.6] - 2022-06-23
- Add XDEBUG_SESSION cookie to list of cookies to ignore in config map.

## [1.0.5] - 2022-06-09
- Add 499 and 500 to the list of temporarily cached Backend responses to avoid a potential drowning of the drupal pod and sql instance if the site returns errors.  

## [1.0.4] - 2022-06-09
- Add bigger header size to all varnish instances instead of making an option to avoid 502 header too big error from drupal cache tags

## [1.0.3] - 2022-01-26
### Added
- Let requests pass through to enable cache-tags on the frontend.

## [1.0.2] - 2021-12-30
### Added
- Update github pages domain.

## [1.0.1] - 2021-12-30
### Added
- Update github pages domain.

## [1.0.0] - 2021-11-19
### Added
- Initial release.
