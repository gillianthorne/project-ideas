# Service health checker (Java)

**Status:** exploring
**Started:** 2026-07-01
**Tags:** #learning #java #oop #vps #cybersecurity

## What is it?
A console-based Java tool that checks the health of both local VPS services/ports and external URLs, runs on a schedule via cron, logs results to a file, and emails on failure.

## Why
Wanted a Java project to properly learn/relearn classes — console-based, cybersecurity/Linux-tooling flavored rather than a game this time. This one doubles as a genuinely useful tool for vm666, addressing the "no visibility if something breaks" problem, similar in spirit to the earlier VPS monitoring dashboard idea but as a lightweight scheduled checker instead of a dashboard app.

## Brainstorming
- Checks both local services/ports on the VPS and external URLs/websites
- `HealthCheck` as an interface/abstract class, with different implementations per check type (e.g. `PortCheck`, `HttpCheck`, maybe `ProcessCheck`) — clean polymorphism example, each overriding how the check is actually performed
- Runs on a schedule via cron rather than manually — fits with other cron-based plans already in progress for the VPS
- Results reported two ways: printed to console (for manual runs) and written to a log file (for the scheduled runs to leave a history)
- Alerting: email on failure — likely needs a mail-sending approach from Java (e.g. JavaMail, or shelling out to a local `sendmail`/`mail` command already available on the VPS)

## Specs / requirements
- **Language:** Java
- **Interface:** console/terminal only, no GUI
- **Checks:** local ports/services + external URLs, both supported
- **Scheduling:** run via cron (not built-in scheduling inside the app itself)
- **Output:** console output + log file
- **Alerting:** email notification on failure

### Class design (rough)
- `HealthCheck` — interface/abstract class defining a common `check()` method
- `PortCheck` — implements `HealthCheck`, checks a local port is open/listening
- `HttpCheck` — implements `HealthCheck`, checks a URL responds (status code, maybe response time)
- `CheckResult` — outcome of a single check (target, success/failure, timestamp, details)
- `Logger`/`ReportWriter` — writes results to console + log file
- `Alerter` — sends an email when a `CheckResult` indicates failure
- `HealthCheckRunner` — owns a list of `HealthCheck`s, runs them all, coordinates logging + alerting

## Open questions
- Email sending approach — JavaMail library vs. shelling out to a local mail command on the VPS?
- Config format for the list of checks (ports/URLs to monitor) — hardcoded, a properties file, or a simple text/JSON config?
- Log file format/rotation — plain append-only log, or something that rotates/caps size over time?
- Failure threshold — alert on first failure, or only after N consecutive failures (to avoid noise from a blip)?

## Links
- 
