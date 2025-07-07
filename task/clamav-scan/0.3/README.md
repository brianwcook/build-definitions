# clamav-scan task

## Description:

The clamav-scan task scans files for viruses and other malware using the ClamAV antivirus scanner.
ClamAV is an open-source antivirus engine that can be used to check for viruses, malware, and other malicious content.
The task will extract compiled code to compare it against the latest virus database to identify any potential threats.
The logs will provide both the version of ClamAV and the version of the database used in the comparison scan.

## Version 0.3:

This version introduces significant performance improvements by using `clamdscan` instead of `clamscan`. The `clamdscan` command connects to the ClamAV daemon (clamd) which keeps the virus database loaded in memory, resulting in faster scanning operations compared to the previous version that had to reload the database for each scan.

### Key improvements in 0.3:
- **Performance**: Uses `clamdscan` with the clamd daemon for faster scanning
- **Memory efficiency**: Virus database is loaded once and kept in memory
- **Compatibility**: 100% compatible with 0.2 version parameters, logs, and results
- **Daemon management**: Automatically starts and manages the clamd daemon lifecycle

### Migration from 0.2:
- All input parameters remain the same
- Log format and output remain identical
- Results structure is unchanged
- No user action required for migration

## --max-filesize:

Is set to the same value as the default value according to the ClamAV official Documentation.

https://wiki.debian.org/ClamAV

https://docs.clamav.net/manual/Development/tips-and-tricks.html?highlight=max-filesize#general-debugging

## Parameters

| name                     | description                                                            | default value | required |
| ------------------------ | ---------------------------------------------------------------------- | ------------- | -------- |
| image-digest             | Image digest to scan.                                                  |               | true     |
| image-url                | Image URL.                                                             |               | true     |
| docker-auth              | unused                                                                 | ""            | false    |
| ca-trust-config-map-name | The name of the ConfigMap to read CA bundle data from.                 | trusted-ca    | false    |
| ca-trust-config-map-key  | The name of the key in the ConfigMap that contains the CA bundle data. | ca-bundle.crt | false    |
| scan-threads             | Number of threads to run in clamdscan parallel. Should be <= 8.         | 1             | false    |

## Results

| name             | description                   |
| ---------------- | ----------------------------- |
| TEST_OUTPUT      | Tekton task test output.      |
| IMAGES_PROCESSED | Images processed in the task. |

## Source repository for image:

https://github.com/konflux-ci/konflux-test/tree/main/clamav

## Additional links:

https://docs.clamav.net/

https://docs.clamav.net/manual/Usage/Scanning.html#clamdscan 