# Why are Windows Inbox Drivers all signed 2006?
_Written on 2019-03-09._

If you ever meddled with Windows drivers, you may have noticed that, most inbox drivers are signed 2006-06-21, even though they were freshly built with the newest Windows build number.

A quick search gives us the answer:
> (MS Dev here) There's a very good reason for that, and it has nothing to do with the age of the driver or anything.

> When PNP ranks drivers, it first looks at the hardware ID that the driver matches. If any two drivers match identical hardware, the first tiebreaker is the date of the driver. So if you had a device that could use a built-in driver, but you had installed some custom/OEM driver on your device, every time MS updates our driver, it would overwrite your custom driver because the date is newer than the one you wanted. How do we avoid this? Every driver we ship has the Vista RTM date, regardless of when it was last updated (we update the version number, which is the next tiebreaker if the date is the same). Since only drivers as far back as Vista are compatible with new versions of Windows, every driver should have a date newer than Vista RTM, preserving the driver you installed as the best ranked driver.

## Reference
* [Why are all Windows drivers dated June 21, 2006? Don’t you ever update drivers? – The Old New Thing](https://blogs.msdn.microsoft.com/oldnewthing/20170208-00/?p=95395)
* [Reddit: Is Windows 10 still a polished Vista kernel?](https://www.reddit.com/r/windows/comments/5hediy/is_windows_10_still_a_polished_vista_kernel/dazjyiw/)