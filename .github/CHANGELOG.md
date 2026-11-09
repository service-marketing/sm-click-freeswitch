# Changelog

All notable changes to this project will be documented in this file. See [standard-version](https://github.com/conventional-changelog/standard-version) for commit guidelines.

## [1.1.0-alpha.11](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.10...v1.1.0-alpha.11) (2026-11-09)


### Bug Fixes

* install missing runtime libraries in FreeSWITCH image ([e251d6b](https://github.com/service-marketing/sm-click-freeswitch/commit/e251d6b37a6f38561fdf6d3a6624c95967824ac5))

## [1.1.0-alpha.10](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.9...v1.1.0-alpha.10) (2026-11-06)


### Bug Fixes

* force LF line endings for build/script files in .gitattributes ([a2d1fa9](https://github.com/service-marketing/sm-click-freeswitch/commit/a2d1fa95e0cea53161564e340c7c05aa2a8701e5))
* use git checkout in Docker to apply eol=lf, finalize build deps ([87fca33](https://github.com/service-marketing/sm-click-freeswitch/commit/87fca33cc55979750ca1965121e655c2457bfa7b))

## [1.1.0-alpha.9](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.8...v1.1.0-alpha.9) (2026-11-06)


### Bug Fixes

* disable mod_mongo (deprecated mongoc API, -Werror breaks build) ([95a1ec0](https://github.com/service-marketing/sm-click-freeswitch/commit/95a1ec08c06ee641ae294fcb58c84707bebbd196))
* remove -Werror from configure.ac before bootstrap ([374560f](https://github.com/service-marketing/sm-click-freeswitch/commit/374560f4efdc103b0a4d9a8a6aed5591b02896c8))

## [1.1.0-alpha.8](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.7...v1.1.0-alpha.8) (2026-11-06)


### Bug Fixes

* disable mod_erlang_event (needs erlang-dev/ei.h) ([118b55e](https://github.com/service-marketing/sm-click-freeswitch/commit/118b55ee225214f3203464a31628f0c15097a8e8))

## [1.1.0-alpha.7](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.6...v1.1.0-alpha.7) (2026-11-06)


### Bug Fixes

* drop libilbc-dev/libflite1-dev, disable mod_ilbc/mod_flite ([a59813e](https://github.com/service-marketing/sm-click-freeswitch/commit/a59813e449db299f022bb4fea420119279f9f2eb))

## [1.1.0-alpha.6](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.5...v1.1.0-alpha.6) (2026-11-06)


### Bug Fixes

* add all missing deps and disable unavailable-dep modules ([358c51b](https://github.com/service-marketing/sm-click-freeswitch/commit/358c51bdbc62b9f92baab065c4574c3d2fb9ca84))
* add librabbitmq-dev for mod_amqp ([1a29b46](https://github.com/service-marketing/sm-click-freeswitch/commit/1a29b468b6e9d70ce9eea0b410a38b0d049e959b))

## [1.1.0-alpha.5](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.4...v1.1.0-alpha.5) (2026-11-06)


### Bug Fixes

* add yasm for libvpx bundled configure ([8772067](https://github.com/service-marketing/sm-click-freeswitch/commit/87720676e3cfe83c7faa12fcee13b8669ce6a43e))

## [1.1.0-alpha.4](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.3...v1.1.0-alpha.4) (2026-11-06)


### Bug Fixes

* replace FFmpeg build-deps with system libvpx-dev ([be626ab](https://github.com/service-marketing/sm-click-freeswitch/commit/be626ab59430e270733745c9d9a8be092e625fc1))

## [1.1.0-alpha.3](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.2...v1.1.0-alpha.3) (2026-11-06)


### Bug Fixes

* add autoconf-archive, disable mod_av (libvpx build failure) ([2a2ad9a](https://github.com/service-marketing/sm-click-freeswitch/commit/2a2ad9a28b391cacd1438e1fee63ed38ebdd1053))

## [1.1.0-alpha.2](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.1...v1.1.0-alpha.2) (2026-11-06)


### Bug Fixes

* disable libks modules, add missing deps, use modules.conf.most ([39d71c1](https://github.com/service-marketing/sm-click-freeswitch/commit/39d71c107c9aa86967a64b70cace24fe4ae626d6))

## [1.1.0-alpha.1](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.1.0-alpha.0...v1.1.0-alpha.1) (2026-11-06)


### Bug Fixes

* add missing build deps (sqlite3, mariadb, ffmpeg libs) ([c3f6999](https://github.com/service-marketing/sm-click-freeswitch/commit/c3f6999b56b8b10734a3bb0cbe9b7fd56c66b547))

## [1.1.0-alpha.0](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.0.1-alpha.5...v1.1.0-alpha.0) (2026-11-06)


### Features

* full from-source build, no SignalWire image dependency ([0383e30](https://github.com/service-marketing/sm-click-freeswitch/commit/0383e303b8d42f1b97f1e86e42a8004360558539))

### [1.0.1-alpha.5](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.0.1-alpha.4...v1.0.1-alpha.5) (2026-11-06)


### Bug Fixes

* build sofia-sip 1.13.19 from source inside CI base image ([ea84a2d](https://github.com/service-marketing/sm-click-freeswitch/commit/ea84a2d12bc7838cc9b5b0c0ae083ffe77e2d8be))

### [1.0.1-alpha.4](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.0.1-alpha.3...v1.0.1-alpha.4) (2026-11-06)


### Bug Fixes

* use signalwire public CI base image as builder ([267e521](https://github.com/service-marketing/sm-click-freeswitch/commit/267e521666dbab9258f2a8aba65635dd199470ca))

### [1.0.1-alpha.3](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.0.1-alpha.2...v1.0.1-alpha.3) (2026-11-06)


### Bug Fixes

* wget --no-check-certificate for sofia-sip tarball ([510fc50](https://github.com/service-marketing/sm-click-freeswitch/commit/510fc5002a0c377a330fbdabf907a21669611388))

### [1.0.1-alpha.2](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.0.1-alpha.1...v1.0.1-alpha.2) (2026-11-06)


### Bug Fixes

* use wget tarball for sofia-sip, avoid git SSL CA issue on Debian ([5ebda08](https://github.com/service-marketing/sm-click-freeswitch/commit/5ebda082fceaa49539b68d3e629c71050e46cf97))

### [1.0.1-alpha.1](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.0.1-alpha.0...v1.0.1-alpha.1) (2026-11-06)


### Bug Fixes

* bump sofia-sip to v1.13.19 (required by FreeSWITCH) ([17cec22](https://github.com/service-marketing/sm-click-freeswitch/commit/17cec22f450aec803ef1c6fc2b7d5ae6f3b42b0e))
* replace APT Dockerfile with source-build, remove SignalWire token ([cfd54e1](https://github.com/service-marketing/sm-click-freeswitch/commit/cfd54e1728138988c837a7886b1b6e9678dc936a))

### [1.0.1-alpha.0](https://github.com/service-marketing/sm-click-freeswitch/compare/v1.6.0...v1.0.1-alpha.0) (2026-11-06)

# Changelog

All notable changes to this project will be documented in this file. See [standard-version](https://github.com/conventional-changelog/standard-version) for commit guidelines.
