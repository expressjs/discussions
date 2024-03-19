## Date/Time

| Timezone | Date/Time |
|----------|-----------|
<%= [
  'America/Los_Angeles',
  'America/Denver',
  'America/Chicago',
  'America/New_York',
  'Europe/London',
  'Europe/Amsterdam',
  'Europe/Moscow',
  'Asia/Kolkata',
  'Asia/Shanghai',
  'Asia/Tokyo',
  'Australia/Sydney'
].map((zone) => {
  return `| ${zone} | ${date.setZone(zone).toFormat('EEE dd-MMM-yyyy HH:mm (hh:mm a)')} |`
}).join('\n') %>

Or in your local time:
* https://www.timeanddate.com/worldclock/?iso=<%= date.toFormat("yyyy-MM-dd'T'HH:mm:ss") %>

## Agenda

Extracted from **<%= agendaLabel %>** labelled issues and pull requests from **<%= owner %>/<%= repo %>** prior to the meeting.

<%= agendaIssues.map((i) => {
  return `* ${i.html_url}`
}).join('\n') %>

## Invited

This meeting is open for anyone who wants to attend. Reminder to follow our [Code of Conduct](https://github.com/expressjs/express/blob/master/Code-Of-Conduct.md).

@expressjs/express-tc
@expressjs/triagers
@expressjs/security-wg

## Links

* Minutes:

### Joining the meeting

* link for participants: https://zoom-lfx.platform.linuxfoundation.org/meeting/95266714809?password=f37cff1b-cb3a-4a21-9425-210e4714c72e
* For those who just want to watch: https://www.youtube.com/@expressjs-official
