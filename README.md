# Event Spec Check

A web tag template for testing. Fire it on every custom event and it checks each one against GA4's naming rules and your own list of required parameters, then reports what it finds in Preview mode.

It checks:

1. **The event name:** 40 characters or fewer, starts with a letter, letters (including non-English ones), numbers and underscores only, and not a reserved name that GA4 uses for its own events, such as `session_start` or `first_visit`, or the `query_id` prefix.
2. **The parameter names:** the same character rules, the reserved prefixes (`_`, `firebase_`, `ga_`, `google_`, `gtag.`), and names that look like personal data, such as `user_email`.
3. **The parameter values:** 100 characters or fewer, or 300 for `page_title`, 420 for `page_referrer` and 1,000 for `page_location`.
4. **The number of parameters:** 25 or fewer per event, counting `ecommerce` fields as well. Google doesn't say whether these count towards the limit, so this is a cautious estimate.
5. **Required parameters:** from your own list, for example `transaction_id`, `value`, `currency` and `items` on `purchase`.

## Why

Most GA4 data problems start in the dataLayer: a parameter that's missing on one template, a name with a space or a hyphen, or an email address in a form event. Checking every event while you test catches them before they reach reports. The rules match Metric Owl's [event name checker](https://www.metricowl.co.uk/ga4-event-name-checker), and the required list can come straight from a [dataLayer specification](https://www.metricowl.co.uk/guides/datalayer-specification-template).

## Set up

1. Add the template from the gallery, or import `template.tpl` under **Templates**, as a tag template.
2. Create a tag from it. Under **Required parameters**, list one event per line, such as `purchase: ecommerce.transaction_id, ecommerce.value, ecommerce.currency, ecommerce.items`.
3. Fire it on a **Custom Event** trigger with the event name `.*` and **Use regex matching** ticked.
4. In Preview, go through the site and read the results in the browser console, or in the `event_spec_check` events in Tag Assistant.

Use it while testing, or on a staging site. It isn't meant to run for every visitor.

## The result

| Field | Example |
|---|---|
| `event_spec_event` | `purchase` |
| `event_spec_status` | `ok` or `issues` |
| `event_spec_issues` | `missing:ecommerce.transaction_id`, `event_name_invalid_characters`, `event_name_reserved`, `param_reserved_prefix:ga_source`, `param_may_be_personal_data:user_email`, `param_value_too_long:note`, `over_25_parameters`, `push_not_found` |

## Things to know

- **Each event's own push:** the tag checks the exact push that triggered it, matched by Tag Manager's `gtm.uniqueEventId`, so two pushes of the same event are checked separately. A parameter left over from an earlier push doesn't hide a missing one.
- **Names Google collects automatically,** such as `page_view` or `form_submit`, aren't flagged, because sending them through Tag Manager is normal. Google's list of reserved names isn't complete, and Google may add to it.
- **Custom dimension names:** names such as `user_id` and `currency` can't be registered as custom dimensions in GA4, but they're fine to send, so they aren't flagged.
- **Pushes from other templates:** a push made by another custom template can arrive wrapped in an outer object. The tag looks one level inside, so these are found too.
- **gtag() events:** events sent with `gtag('event', …)` aren't found, and report `push_not_found`. It's built for `dataLayer.push`.
- **Skipped events:** Tag Manager's own events (`gtm.js`, `gtm.dom` and so on) and the tag's own results are skipped, so it can't loop.
- **Personal data:** the check is a warning based on the parameter's name. It can't see whether a value really is personal data.
- **A renamed dataLayer:** the tag reads the standard `dataLayer`. If your container uses another name, it reports `push_not_found`.
- **A generic GA4 event tag:** if you have a GA4 tag that fires on every custom event using `{{Event}}` as its name, add an exception for `event_spec_check`, or the results go to GA4.
- **Testing only:** it copies the dataLayer on every event, which is fine while testing but not something to run for every visitor.

## Permissions

- **dataLayer:** reads `event` and `gtm.uniqueEventId`, reads the dataLayer, and writes to it for the results event.
- **Logging:** in Preview only.

## Licence

Apache License 2.0. See [LICENSE](LICENSE).

Built and maintained by Marc Alexander at [Metric Owl](https://www.metricowl.co.uk), a GA4 and Google Tag Manager consultancy in London. Issues and suggestions are welcome.
