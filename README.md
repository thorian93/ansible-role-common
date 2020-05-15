# Ansible Role: Common

This role manages several parts of a Linux system which are not worth their own role.

## Here be Dragons!

When managing DNS resolution with this role be aware of the following: On Ubuntu this role will remove the symlink on /etc/resolv.conf if it exists and replace it with a static file. The symlink originates in the `systemd-resolved` daemon. Managing that daemon is at least currently out of scope for this role. I know this not a beautiful solution but it works for me. If you know how to handle this better feel free to contact me or create a PR.

When managing sudo configuration this role has some defaults you might want to review before applying.

## Known issues

- Fedora 30: The dropping support for Python 2 in Fedora causes problems for Ansible. This can be fixed by setting the `ansible_python_interpreter` variable to the appropriate Python 3 binary.
- **openSUSE Leap 15 and 42**: A missing dependency does not allow installation of a dependent tool. A workaround is in place but does not work properly.

## Requirements

No special requirements; note that this role requires root access, so either run it in a playbook with a global `become: yes`, or invoke the role in your playbook like:

    - hosts: foobar
      roles:
        - role: ansible-role-common
          become: yes

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    common_crontabs_configure: false
    common_disks_configure: false
    common_dns_configure: false
    common_groups_configure: false
    common_mail_configure: false
    common_zsh_configure: false
    common_scripting_configure: false
    common_software_configure: false
    common_users_configure: false
    common_timezone_configure: false
    common_sudo_configure: false
    common_vim_configure: false

Enable and disable managed sections of this role.

    common_sudo_sudoers_files: "*_sudoers"

Configure lookup path for custom sudoers files.

    common_sudo_defaults:
      - line: "# Defaults targetpw # ask for the password of the target user i.e. root"
        regex: '^Defaults\stargetpw.*'
        state: 'present'
      - line: "# ALL ALL=(ALL) ALL # WARNING! Only use this together with 'Defaults targetpw'!"
        regex: '^ALL\sALL=\(ALL\)\sALL.*'
        state: 'present'
      - line: '%{{ common_admin_group }} ALL=({% if ansible_os_family == "Debian" %}ALL:ALL{% else %}ALL{% endif %}) ALL'
        regex: '(# |^)\%{{ common_admin_group }}\sALL=\({% if ansible_os_family == "Debian" %}ALL:ALL{% else %}ALL{% endif %}\)\sALL.*'
        state: 'present'

Configure sudo options. There is a default set of three options which basically ensure the common sudo behaviour known by Debian derivates works on any distribution.

    common_optional_apps_install: false

Enable installation of optional apps.

    common_epel_enabled: false

Enable EPEL repository on RedHat derivates.

    common_dns_search: []

Configure DNS search path e.g for your local network.

    common_dns_servers:
      - 1.1.1.1
      - 1.0.0.1

Configure your DNS servers.

    common_dns_options:
      # - "timeout:3"
      # - "attempts:2"
      # - "rotate"

Configure additional options for resolv.conf

    common_timezone: "Europe/Berlin"

Configure the timezone.

    common_etc_aliases:
      - name: "Redirect root mails."
        line: 'root:		root'
        regexp: '^root\:.*'
        insertbefore: EOF
        setype: etc_aliases_t
        state: present
        backup: yes

Configure /etc/aliases.

    scripting_path: /opt/control/scripts
    config_path: /opt/control/config

Configure general paths.

    shopt_options:
      - shopt -s cdspell
      - shopt -s nocaseglob

Configure global shopt options.

    common_zsh_users: root

Configure ZSH for this users.

    common_zsh_theme: clean

Configure the oh my zsh theme.

    common_zsh_path: /opt/oh-my-zsh

Configure the oh my zsh installation path.

    common_zsh_plugins:
    - git
    - rsync
    - wd

Enable these oh my zsh plugins.

## Dependencies

None.

## OS Compatibility

This role ensures that it is not used against unsupported or untested operating systems by checking, if the right distribution name and major version number are present in a dedicated variable named like `<role-name>_stable_os`. You can find the variable in the role's default variable file at `defaults/main.yml`:

    common_stable_os:
      - Debian 10
      - Ubuntu 18
      - CentOS 7
      - Fedora 30

If the combination of distribution and major version number do not match the target system, the role will fail. To allow the role to work add the distribution name and major version name to that variable and you are good to go. But please test the new combination first!

Kudos to [HarryHarcourt](https://github.com/HarryHarcourt) for this idea!

## Example Playbook

    ---
    - name: "Run role."
      hosts: all
      become: yes
      roles:
        - ansible-role-common

## Contributing

Please feel free to open issues if you find any bugs, problems or if you see room for improvement. Also feel free to contact me anytime if you want to ask or discuss something.

## Disclaimer

This role is provided AS IS and I can and will not guarantee that the role works as intended, nor can I be accountable for any damage or misconfiguration done by this role. Study the role thoroughly before using it.

## License

MIT

## Author Information

This role was created in 2020 by [Thorian93](http://thorian93.de/).
