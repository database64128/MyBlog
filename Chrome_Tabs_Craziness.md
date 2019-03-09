# Chrome Tabs Craziness
_Written on 2019-03-09._
## An Unfortunate Incident
Two days ago, on 2019-03-07, I got a lot of time working on Linux, Hyper-V and related stuff. I was trying to use an undocumented feature of Hyper-V - `GPU-P`, or GPU Partition. So natually, a lot of research work turned into a bunch of browser tabs. I opened at least 50 tabs before the incident happened, and peaked over 65 tabs on that day.

Fortunately, my laptop's hardware has no problem running so many tabs. Chrome can easily scale up too. You see, the problem is usually not like what we anticipated. After [leaving the VM session in disappointment](Hyper-V_notes.md), I opened a notification from Twitter. It opened a new tab, and immediately, Windows had a `bugcheck`. After restarting, the VM was still running, the proxy PAC was on, and relaunching Chrome caused a `bugcheck` again. Before the fourth relaunch of Chrome Canary, I shut down the VM. Running Firefox and Vivaldi didn't cause `bugcheck`. But opening Chrome Canary still did. The fifth time, I disabled proxy. No bugcheck this time.

There is no way to tell if turning off the VM or disabling the proxy prevented the `bugcheck`. Mostly likely the `bugcheck` was caused by Windows's proxy service. But that's not the problem right now.

After successfully relaunched Chrome, the previous session is gone. Turns out Chrome only stores one current session and one previous session. `%AppData%\Local\Google\Chrome SxS\User Data\Default\Current Session` gets renamed as `%AppData%\Local\Google\Chrome SxS\User Data\Default\Last Session` when the next session starts. During the incident, 5 sessions have been created. So my precious little session is long gone.

## Future Prevention
After the incident, I had to look through Chrome's History manually, line by line, to recover my lost session. The process is long, dull, and indeed very painful. To prevent such incident, as suggested by a Stack Exchange user, we can use a session-management extension, __[Session Buddy](https://chrome.google.com/webstore/detail/session-buddy/edacconmaakjimmfgnblocblbcdcpbko)__. It automatically saves sessions, so you can recover any time without losing any opening tabs.

## Reference
* [Can I restore closed tabs after quitting Chrome? - Super User](https://superuser.com/questions/635436/can-i-restore-closed-tabs-after-quitting-chrome)