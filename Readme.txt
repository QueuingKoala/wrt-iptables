(document best view with word wrap)

OVERVIEW

The wrt-iptables (this package) is designed to facilitate Netfilter rule loading and saving through the iptables-restore and iptables-save frontends on the OpenWRT platform. An initscript controls the process along with a supporting UCI configuration file and compressed storage of the rulesets.


LICENSING / CONTACT

This code is licensed under the GPL version 3.
Full license text can be found in the folder 'Licensing' or online at:
http://www.gnu.org/licenses/gpl-3.0-standalone.html

Copyright 2013 Josh Cepek <josh.cepek AT usa.net>

Bug reports and patches for this project may be sent here. Please do not contact me for support regarding your own firewall rules as you will be ignored. Google, manpages, mailing lists, or IRC is where you want to go if you need help learning about Netfilter or firewalling.


PURPOSE

This projects is designed to replace the OpenWRT "firewall" initscript functionality. While this system is understandable for integration into the web-based LuCI system, it is slow to call iptables commands as it does, the changes are not atomic, and it creates very ugly firewall rules that can be hard to read and lack efficiency.

For users able and interested in designing their own firewall rulesets, this wrt-iptables package provides a way to load the firewall rules on boot and interact with the netfilter system to save/restore firewall rules.


INSTALLATION

Use of this package requires the iptables & gzip support, symlinks for iptables-save and iptables-restore, and relevant kernel support for any netfilter modules used in the rulesets.

To use the wrt-iptables functionality, copy the files listed below to the following locations on your OpenWRT system. Continue reading the next section to understand how the rules are saved and loaded.

	iptables.init -> /etc/init.d/iptables (must be executable, ie: chmod +x)
	iptables.uci -> /etc/config/iptables
	iptables-blank.gz -> /etc/iptables-blank.gz
	(optional ruleset of choice, gzipped) -> /etc/iptables-save.gz


RULE FILES

The rule files are in iptables-save syntax run through gzip, and may optionally have excess comments and rule counters removed depending on configuration, which is described further below. While these files can be manipulated by hand, it is recommended to use the initscript to avoid creating files that cannot be loaded. The rules may also be located at a different point in the filesystem by editing the configuration file.


CONFIGURATION WITH UCI

The configuration file at /etc/config/iptables controls options that impact the initscript's operation, stored under a 'config iptables' stanza. This package distribution includes a sample file named 'iptables.uci' that holds the default settings, including optional setting support. The following option values are allowed:

	save: defines the full path to the 'iptables-save.gz' file (may be renamed here)
	empty: defines the full path to the 'iptables-blank.gz' file (may be renamed)
	up_script: (undefined by default) path to a post-start script (see UPSCRIPT below)
	strip: (bool, defaults to 1/true) if true, strips comments after the first line
	counters: (bool, defaults to 0/false) if true, saves/restores rule hit counters


UPSCRIPT

If defined in the UCI config, the up_script is a path to an executable command that will be called after the ruleset is successfully loaded, similar to firewall.user in the OpenWRT stock firewall script. Note that it is probably a bad idea to use this same name due to conflicts. This file must be present and executable in order to be run; note that it is not sourced. The exit code to the up_script is ignored and not acted on.

The up_script support is designed to handle dynamic settings that are otherwise not possible to save in the rule file. When possible, placement of static rules in the rule file is preferred as changes are atomic, take far less time to apply, and are compressed to save space.


INITSCRIPT USAGE

The initscript supports the following commands: enable, disable, start, stop, restart, save, and show. The restart and start commands are operationally identical.

If enabled on boot by calling '/etc/init.d/iptables enable', this script will run lexically after the stock "firewall" initscript; however, it is a waste of resources to run the earlier script, and as such the firewall init should be disabled when using this package on-boot. If enabled, the script can be disabled by calling with the 'disable' command.

The 'save' command will save the currently loaded ruleset to the file defined in the UCI config file; this allows modification of the live rules though calls to the iptables system command, and if happy with the result the rules may be saved to flash memory for use on boot (if enabled to run at startup.) The save command does not impact the running firewall.

The 'start' (and 'restart') commands load the saved firewall; it is an error to call this without the save ruleset present. You must design your own rules, or start from one of the samples this package provides (see the SAMPLES section.)

The 'stop' command will load the empty ruleset; it is an error to call this without the empty ruleset present. The included 'iptables-blank.gz' ruleset contains an all-ACCEPT policy on chains for the raw, nat, filter, and mangle tables (use of additional tables requires an updated ruleset file to get a truly blank state.) It is possible to define your own ruleset here; a use-case may be to provide the "most minimal" firewall setup while still providing NAT and essential firewall support.

The 'show' command will dump the live ruleset to standard output in the same format it would be saved in the file, but without being run through gzip. This is primarily intended to be analogous  to the output from iptables-save while following the config choices for 'strip' and 'counters' defined earlier; this can make reading rules easier as the output is cleaner.


SAMPLES

Some sample rulesets are provided in the directory called 'samples.' You should be sure to understand the samples before using them as modification may well be required for your setup or needs. The samples assume the WAN interface is eth0.1 and the LAN is br-lan. All sample files must be gzipped and placed at the expected path as defined above.

List of samples and brief descriptions follow:

rules-blank: This ruleset is identical to the iptables-save.gz file noted earlier, but uncompressed for easy comparison or editing.

rules-generic: The "most basic" statefully-aware firewall configuration, including NAT support for outbound traffic.

rules-fancy: A fancier example; do not use without completely understanding the ruleset as some advanced concepts are used. Compared to the generic sample, this ruleset rejects traffic to/from upstream that uses RFC1918 private addressing and more readily supports DNAT features through conntrack state matches. A commented-out NAT entry demonstrates all that's necessary to pass traffic to a private host as the filter table uses conntrack to accept such forwarded traffic.