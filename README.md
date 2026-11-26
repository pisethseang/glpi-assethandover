# Asset Handover & Return for GLPI

A GLPI plugin for recording **IT asset handovers and returns**, keeping a traceable history of equipment issued to staff and received back by IT.

**Author:** Piseth SEANG  
**Repository:** [pisethseang/glpi-assethandover](https://github.com/pisethseang/glpi-assethandover)  
**License:** GPLv2+ (as declared in the plugin metadata)

> **Version note:** This repository contains separate plugin directories for GLPI 10, 11 and 12. Install the directory for your GLPI major version. The GLPI 12 implementation declares plugin version **1.2.0** and requires **PHP 8.3+**.

## Features

- **Handover forms** — record who receives equipment, when it is issued, and who prepared or assigned it.
- **Return forms** — create a return linked to a previous handover and prefill the staff and asset information.
- **Multiple assets per record** — add several items to one handover or return, with serial numbers, asset numbers, descriptions, conditions and remarks.
- **GLPI asset lookup** — search for computers, monitors, printers, phones, peripherals and network equipment in the existing GLPI inventory.
- **Employee details** — capture the GLPI user, department, unit, position, employee ID, manager, and other form details.
- **Asset assignment check** — display a warning if an asset matches an active handover, using its GLPI identity, serial number or asset number.
- **Handover / return history** — review records per user and trace assets through their previous handover and return records.
- **Activity log** — record creation and update activity; inspect changes associated with a handover record.
- **Search and list views** — browse all records, handovers, returns, user history and the trash; sort and paginate results.
- **Printable slips** — produce A4-landscape handover/return forms through the browser print dialog. Select **Save as PDF** in the browser to create a PDF file.
- **Role-based access** — uses the GLPI `plugin_assethandover` profile right for viewing and modifying plugin records.

> Asset lookup uses existing GLPI inventory data. A handover or return creates records in the **plugin's own tables**; it is not the same as updating an asset's native GLPI assigned-user field.

## Compatibility

| GLPI release | Plugin version declared in `setup.php` | Source directory | Declared requirements |
| --- | --- | --- | --- |
| GLPI 10 | 1.0.0 | [`glpi_version_10/assethandover`](glpi_version_10/assethandover) | GLPI 10.0.0 through 10.0.99 |
| GLPI 11 | 1.1.0 | [`glpi_version_11/assethandover`](glpi_version_11/assethandover) | GLPI 11.0.0 through 11.0.99 |
| GLPI 12 | 1.2.0 | [`glpi_version_12/assethandover`](glpi_version_12/assethandover) | GLPI 12.0.0+; PHP 8.3+ |

The requirements above reflect plugin metadata in the repository; they do not imply that every patch level or hosting configuration has been independently tested.

## Installation

### 1. Back up your GLPI instance

Before installing or upgrading, back up your GLPI database, configuration and existing plugin directory. The installation routine creates plugin database tables and profile-right entries.

### 2. Download the source

```bash
git clone https://github.com/pisethseang/glpi-assethandover.git
cd glpi-assethandover
```

### 3. Copy the correct plugin folder

The final plugin directory **must be named `assethandover`**, with `setup.php` directly inside it:

```text
<GLPI_ROOT>/plugins/assethandover/
├── setup.php
├── hook.php
├── inc/
└── front/
```

For example, on a Linux host with GLPI 12 installed at `/var/www/html`:

```bash
sudo mkdir -p /var/www/html/plugins/assethandover
sudo cp -a glpi_version_12/assethandover/. /var/www/html/plugins/assethandover/
```

Adjust the directory and ownership/permissions to match your GLPI installation. For GLPI 10 or 11, replace `glpi_version_12` with the matching version directory.

### 4. Install and enable in GLPI

1. Sign in with an administrator account.
2. Open **Setup → Plugins**.
3. Locate **Asset Handover & Return**.
4. Choose **Install**, then **Enable/Activate** as offered by your GLPI installation.
5. Verify that authorized users can see **Assets → Asset Handover**.

If the plugin does not appear, verify the directory name, the location of `setup.php`, and the version requirements.

### Docker Desktop example (GLPI 12)

The following example assumes your running GLPI container is named `glpi` and its plugin directory is `/var/www/html/plugins`:

```bash
# Run from the cloned repository directory.
# Back up an existing installed plugin before replacing it.
docker cp glpi:/var/www/html/plugins/assethandover ./assethandover-backup

# Create the target plugin directory and copy GLPI 12 files.
docker exec glpi mkdir -p /var/www/html/plugins/assethandover
docker cp glpi_version_12/assethandover/. glpi:/var/www/html/plugins/assethandover/

# Verify the expected plugin metadata file exists.
docker exec glpi ls -l /var/www/html/plugins/assethandover/setup.php
```

**For a first-time install**, skip the backup command if the plugin is not installed yet. If your plugin directory is bind-mounted, copy the files into the host-mounted path instead so they persist when the container is recreated. After copying, visit **Setup → Plugins** to install, update (when available) and enable as appropriate.

## How to use

### Create an asset handover

1. Go to **Assets → Asset Handover**.
2. Choose **New Handover**.
3. Search for and select the GLPI user.
4. Review or enter the department, unit, position, employee ID, manager and handover date as needed.
5. Add one or more assets, using the GLPI asset search or the available form fields.
6. Check asset name, serial/asset number, condition on issue and any remarks.
7. Review any warning about another active handover, then save the form.
8. Open **Print Slip** to print or save an A4-landscape handover document as a PDF using your browser.

### Record a return

1. Open the asset handover list.
2. Start a **Return** for the relevant existing handover when available.
3. Review the prefilled user and assets.
4. Enter return date, condition on return, return status and remarks for each item.
5. Save the return record and print the return slip if required.

The plugin links a return to its source handover with `from_handover_id` and uses that relationship when showing returned versus active records.

### Search, history and cleanup

- Use the **All**, **Handover**, **Return** and **Users** views to find records and review employee history.
- Use the **Trash** view to restore deleted records when permitted.
- **Purge** permanently removes the handover header and its item rows. Treat this as destructive and verify your backup first.
- Review the activity log and asset history for supporting context; these are useful operational records, not a substitute for an independently managed audit trail.

## Permissions and security

The plugin declares the right `plugin_assethandover`. Its PHP handlers check GLPI login and applicable `READ`, `CREATE`, `UPDATE`, `DELETE` and `PURGE` permissions for supported operations.

In the GLPI 12 installer, the plugin creates a default permission entry for the **Super-Admin** profile (and the configured default-profile identifier used by the source). Other profiles may need rights provisioned by an administrator.

**Before production use:**

- Verify the roles that can access handovers, employee details, asset searches, printable forms, and destructive actions.
- Test entity isolation and multi-entity data visibility in your environment; do not assume every query is entity-scoped solely because GLPI has entity permissions.
- Protect backups and database access because handover records can contain employee information and asset identifiers.
- Test creation, editing, returning, deleting and printing in a non-production GLPI instance first.

## Plugin data

The GLPI 12 install routine creates or updates these tables:

| Table | Purpose |
| --- | --- |
| `glpi_plugin_assethandover_handovers` | Handover/return headers, employee details, dates, related handover and deletion status |
| `glpi_plugin_assethandover_items` | Items included in a record, asset identity, serial numbers, conditions and remarks |
| `glpi_plugin_assethandover_logs` | Activity entries associated with plugin records |

The GLPI 12 `plugin_assethandover_uninstall()` function **does not drop these tables** and deliberately preserves existing profile-right entries. Uninstalling the plugin is therefore **not** the same as deleting its stored data. Back up all three plugin tables before database maintenance or migrations.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Plugin not listed in GLPI | Exact `assethandover` directory name, `setup.php` location, PHP and GLPI version requirements |
| Asset Handover menu missing | Plugin activation, active user profile and `plugin_assethandover` READ right |
| Asset lookup returns no results | At least two search characters, source assets exist, and access/query behavior in your test environment |
| Cannot save a record | CREATE or UPDATE right, selected GLPI user, server error log and plugin database tables |
| Print slip does not create a file | It opens browser printing; select **Save as PDF**, A4 landscape, and disable browser headers/footers |
| Changes disappear after container recreation | A Docker copy into a non-persistent container layer; use a bind mount or rebuild the image |

For a Docker-based deployment, a basic log check is:

```bash
docker logs --tail 100 glpi
```

You may also need to inspect GLPI's own PHP/application log files in the configured GLPI files/log directory.

## Updating and rollback

1. Back up the GLPI database and the current `assethandover` directory.
2. Match the code directory to the **installed GLPI major version**.
3. Replace plugin files without deleting your data tables.
4. Follow any **Setup → Plugins** update action required by GLPI.
5. Test listing, handover, linked return, print, permissions and history.
6. If necessary, restore **both** the previous plugin code and a database snapshot taken before the attempted upgrade.

Do not use the GLPI 10 or 11 code directory on GLPI 12 merely because it has a similar plugin name.

## Source references

- [GLPI 12 plugin metadata](glpi_version_12/assethandover/setup.php)
- [GLPI 12 database installation and lifecycle hooks](glpi_version_12/assethandover/hook.php)
- [GLPI 12 handover model](glpi_version_12/assethandover/inc/handover.class.php)
- [GLPI 12 list UI](glpi_version_12/assethandover/front/handover.php)
- [GLPI 12 form UI](glpi_version_12/assethandover/front/handover.form.php)
- [GLPI 12 print endpoint](glpi_version_12/assethandover/front/handover.pdf.php)

## Contributing and support

Bugs, feature requests and contributions can be submitted through the repository's [Issues](https://github.com/pisethseang/glpi-assethandover/issues) and [Pull Requests](https://github.com/pisethseang/glpi-assethandover/pulls) pages.

---

**Asset Handover & Return for GLPI**  
Developed by **Piseth SEANG**.
