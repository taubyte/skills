# Network Logic

## Network Types

### Remote Network
- Cloud-based Taubyte network
- Requires FQDN (Fully Qualified Domain Name)
- Example: `seer.tau.example.com`
- Validated for accessibility
- Added to network history

### Dream Network
- Local development network
- Requires Dream universe to be running
- Universe names from Dream status
- Example: `default`, `my-universe`
- Only available if Dream is running

## Network Selection

- Interactive selection if no flags provided
- Shows available options:
  - Remote network
  - Dream network (if available)
  - Network history (previously used networks)
- Previous selection is highlighted

## Network History

- Previously used networks are remembered
- Shown in selection menu
- Automatically added when using `--fqdn`
- Stored in profile configuration

## FQDN Validation

- Remote network FQDN is validated
- Checks Seer FQDN accessibility
- Must be accessible/valid
- Format: `seer.tau.<domain>`

## Network Validation

When selecting remote network:
- FQDN is validated for Seer accessibility
- Must follow format: `seer.tau.<domain>`
- Network must be reachable
