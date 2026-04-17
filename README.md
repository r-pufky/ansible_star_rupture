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

> Hairpin NAT is required. Local connections are **not** supported.

> Tasks [potentially touching Network Mounted Filesystems][o] will be run as
> the task user and fallback to the service user. Manage these locations
> externally if these fail.

> VM's are highly recommended to provide strong isolation. Many servers require
> low-level access to networking and hardware. Containers may be enabled but
> are not supported for issues.

### SECURITY
> Do **not** expose **7777/TCP** to the Internet. There are currently RCE's for
> exposed remote management. This has been hard disabled in the role until
> resolved. See defaults and [Vulnerability Announcement][p].

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
  4    | star_rupture_flg_config    | Set configuration files.
  5    | star_rupture_flg_backup    | Enable local scheduled backup.

### Example Playbooks

* **[templates/default][q]** - pre-defined server settings.

#### New Server
New server creation requires an admin to start the game and create a save.

``` yaml
- name: 'Create new Star Rupture server.'
  ansible.builtin.include_role:
     name: 'r_pufky.game.star_rupture'
  vars:
    star_rupture_flg_backup: true
    # Loads templates/default/new_game.json
    star_rupture_srv_admin: '{{ vault_admin_password }}'
    star_rupture_srv_user: '{{ vault_user_password }}'

# 1. Connect to server as admin to initialize game start.
# 2. Save game and disconnect.
# 3. Update server configure to auto-load save on start.
```

Apply [Deploy Existing Server](#deploy-existing-server) to auto-load save on
launch.

#### Deploy Existing Server
Configuration files will be interpreted as templates, allowing for vault use of
server configuration files.

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