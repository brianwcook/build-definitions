# Migration from 0.2 to 0.3

Version 0.3:

This version introduces significant performance improvements by switching from `clamscan` to `clamdscan`. The `clamdscan` command connects to the ClamAV daemon (clamd) which keeps the virus database loaded in memory, resulting in faster scanning operations.

### Key changes:
- **Scanner**: Changed from `clamscan` to `clamdscan` for improved performance
- **Daemon management**: Automatically starts and stops the clamd daemon
- **Performance**: Faster scanning due to virus database staying in memory
- **Memory efficiency**: More efficient memory usage with shared database

### Compatibility:
- **100% compatible** with 0.2 version parameters
- **Identical** log format and output structure
- **Same** results format and Enterprise Contract integration
- **No changes** required to existing pipeline configurations

## Action from users

Renovate bot PR will be created with warning icon for a clamav-scan which is expected, no actions from users are required.

The 0.3 version is a drop-in replacement for 0.2 with improved performance characteristics. 