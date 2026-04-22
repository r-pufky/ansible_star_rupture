# Star Rupture
Star Rupture dedicated server.

## [Requirements][i]
Requires [r_pufky.game][g] galaxy-ng collection. See
[reference documentation][h] and [reference documentation][l] for
troubleshooting and config variables.

  Players | CPU           | Memory        | Disk
 ---------|---------------|---------------|------
  2       | 4c/4t @3.0Ghz | 8GB  @2933Mhz | 20GB (excluding saves)
  4       | 6c/6t @3.5Ghz | 16GB @3200Mhz | 20GB (excluding saves)

* Requirements scale **~1GB RAM** per user additional after 4.
* Late game saves may be very large depending on complexity (**>5GB**).

> Hairpin NAT is required for player connections.

> Tasks [potentially touching Network Mounted Filesystems][o] will be run as
> the task user and fallback to the service user. Manage these locations
> externally if these fail.

> VM's are highly recommended to provide strong isolation. Many servers require
> low-level access to networking and hardware. Containers may be enabled but
> are not supported for issues.

### SECURITY
> Do **not** expose **7777/TCP** to the Internet. There are currently RCE's for
> exposed remote management. 'Manage Server' can still be used locally by
> directly connecting to the local IP. After configuring the
> [game client must be restarted to connect][s]. See defaults and
> [Vulnerability Announcement][p].

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage
Additional user created [utilities][r].

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                       | Notes
 ------|----------------------------|-------
  1    | star_rupture_flg_container | Deploy container specific settings.
  2    | star_rupture_flg_cdn       | Statically set Steam CDN IP.
  3    | star_rupture_flg_update    | Update server on launch or if already installed.
  4    | star_rupture_flg_auto_save | Auto-detect latest AutoSave for Session.
  5    | star_rupture_flg_config    | Set configuration files.
  6    | star_rupture_flg_backup    | Enable local scheduled backup.

### Example Playbooks
* **[templates/default][q]** - pre-defined server settings.

#### Static Deployment
Due to the existing [vulnerabilities][p] this is the recommended way to setup
a server.

Start a new game in single player, save, exit, and import save files to be used
for the dedicated server start. This is currently the most consistent way to
start a new game.

##### Deploy Existing Server
Configuration files will be interpreted as templates, allowing for vault use of
server configuration files. With static deployments, the service will
**automatically** load the latest AutoSave*.sav from the specified session in
DSSettings.txt.

``` yaml
- name: 'Deploy a Star Rupture with custom game settings.'
  ansible.builtin.include_role:
     name: 'r_pufky.game.star_rupture'
  vars:
    star_rupture_flg_backup: true
    star_rupture_flg_config: true
    # This will automatically load the last auto save on start.
    star_rupture_srv_ds: 'host_vars/sr.example.com/load_save.json'
    star_rupture_srv_admin: '{{ vault_admin_password }}'
    star_rupture_srv_user: '{{ vault_user_password }}'
```

##### New Server
Not recommended: see static deployment, above.

New server creation requires deploying, connecting to initialize the new
session, and then updating DSSettings.txt to automatically load the session on
server reboots.

``` yaml
- name: 'Create new Star Rupture server.'
  ansible.builtin.include_role:
     name: 'r_pufky.game.star_rupture'
  vars:
    star_rupture_flg_backup: true
    # Loads templates/default/new_game.json
    star_rupture_srv_admin: '{{ vault_admin_password }}'
    star_rupture_srv_user: '{{ vault_user_password }}'
```

After the server is deployed you **must**:

1. Connect to server Public IP as player to start game.
2. Disconnect.
3. Stop server.
4. Update **star_rupture_srv_ds** file to use (see [load_save.json][t]):
   * StartNewGame=false
   * LoadSaveGame=true
5. Run role to set server to automatically load the specified session
   and latest AutoSave*.sav file on boot.

Any server reboot before this happens will reset the session state.

#### Remote Management
Not recommended: see static deployment, above.

The server may also be remotely managed via the game client. See
[Security](#security) notice above. State is not kept between service restarts
when configured to use remote management.

> There is a [known issue][s] where configuring the server and then attempting
> to connect as a player will result in a 'More than one server at this IP'.
> You will need to fully restart the client to connect if this happens.

1. Start Client ➔ Manage Server ➔ local IP
2. Any IP in which port **7777/TCP** is accessible will work. Use the admin
   password.
3. A new game may be created or an old save can be loaded. Be sure to start the
   game.
4. Exit the client.
5. Start Client ➔ Join Game ➔ Public IP
6. Use player password.

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

Testing variables:

  Variable            | Type | Description
 ---------------------|------|-------------
  molecule_flg_inject | bool | Disable **get_url** to inject files locally.

### [Releases][b]

  Release | Debian | Ansible | Notes
 ---------|--------|---------|-------
  1.x.x   | 13     | 2.20    | Initial release.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]

[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_star_rupture/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_game
[h]: https://wiki.starrupture-utilities.com/en/dedicated-server
[i]: https://github.com/r-pufky/ansible_star_rupture/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_star_rupture/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_star_rupture/blob/main/defaults/main/ports.yml
[l]: https://www.survivalservers.com/wiki/index.php/How_to_Create_a_StarRupture_Server_Guide
[o]: https://r-pufky.github.io/ansible_docs/best_practice/patterns/#network-mounts
[p]: https://wiki.starrupture-utilities.com/en/dedicated-server/Vulnerability-Announcement
[q]: https://github.com/r-pufky/ansible_star_rupture/tree/main/templates/default
[r]: https://starrupture-utilities.com
[s]: https://r-pufky.github.io/docs/game/star_rupture
[t]: https://github.com/r-pufky/ansible_star_rupture/blob/main/templates/default/load_save.json