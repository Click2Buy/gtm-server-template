# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Changed
* Installation guides: the test uses the test parameter provided by Click2Buy (`?c2bm=rid:<retailer ID>;test:1`), recorded as a test sale that is never counted in the statistics. No change to the template.
* Installation guides rewritten in the style of the GTM web template guides: the tag is tested in Preview before publishing, its parameters are described, and the template is looked up under its actual name (**Click2Buy tag for retailers**).

## [1.2.0] - 2026-02-17

### Added
* Added support for `value`, `currency`, `transaction_id` and `items` in the purchase event.

## [1.1.0] - 2026-01-08

### Added
* Added support for `categories` in the template configuration.

### Fixed
* Fixed minor issues in the installation guides.

## [1.0.0] - 2025-11-13

### Added
* Initial version of the Click2Buy server-side tag template.
* Handled attribution via the `c2bm` URL parameter on the `page_view` event.
* Stored the attribution ID in a 1st-party cookie (`_c2b_attribution_id`).
* Sent the conversion event to the Click2Buy backend upon the `purchase` event.
* Configured security permissions (Cookies, HTTP Request, Event Data).
* Added unit tests for `page_view` (positive and negative) and `purchase` cases.