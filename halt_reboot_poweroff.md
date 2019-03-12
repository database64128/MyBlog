# `halt` vs `reboot` vs `poweroff`

_Written on 2019-03-11._

Most of the time when I want to shut down my linux VM, I just click shutdown on Hyper-V manager. But today, I executed `systemctl halt`. The system printed a halt message, stopped responding, and required a force shutdown. I kind of got confused by these commands.

On modern Linux distros with `systemd`, use `systemctl` sub-commands to do power management.

Use `systemctl reboot` to reboot the computer. Use `systemctl poweroff` to shutdown the computer.

`systemctl halt` should not be used. It is here only because of some historical reasons - Some legacy BSD servers can't control power or have broken ACPI.

## Reference
* [rhel - What is the difference between these commands for bringing down a Linux server? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/195898/what-is-the-difference-between-these-commands-for-bringing-down-a-linux-server/196014#196014)