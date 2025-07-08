# ClamAV Scan Task 0.3

This task scans container images for viruses, malware, and other security threats using ClamAV antivirus scanner. Version 0.3 uses `clamdscan` with the ClamAV daemon for significantly improved performance through virus database caching.

## Virus Definition Sources

### Official and Secure Sources
The virus definitions come from **official and trusted sources** through a secure supply chain:

1. **Primary Source**: Official ClamAV virus database from [ClamAV.net](https://www.clamav.net/)
   - Updated via `freshclam` tool from official ClamAV database servers
   - Digitally signed virus definitions for authenticity
   - Maintained by the ClamAV team and Cisco Talos Intelligence

2. **Container Image**: `quay.io/konflux-ci/clamav-db:latest`
   - **Source Repository**: [konflux-ci/konflux-clamav](https://github.com/konflux-ci/konflux-clamav)
   - **Build Process**: Daily automated builds via GitHub Actions
   - **Base Image**: Red Hat UBI9 Minimal (enterprise-grade security)
   - **Update Frequency**: Daily builds ensure latest virus definitions

3. **Security Measures**:
   - ClamAV database signatures are cryptographically verified
   - Container image builds use secure CI/CD practices
   - Images are published to Red Hat's Quay registry with security scanning
   - Custom whitelist (`whitelist.ign2`) for known false positives

### Database Loading Process
- **Built-in Database**: Virus definitions are pre-loaded into the container image
- **No Network Dependencies**: No runtime database downloads required
- **Offline Operation**: Task runs entirely offline for security

## Performance Improvements in 0.3

### Traditional vs. Daemon-based Scanning

**Version 0.2 (Traditional `clamscan`)**:
- Loads virus database for each scan process
- Memory usage: ~1GB per process for database loading
- Slower startup time due to database loading
- No process reuse between scans

**Version 0.3 (Daemon-based `clamdscan`)**:
- Virus database loaded once in memory by `clamd` daemon
- Memory usage: Shared database across all scans
- Faster scanning after initial daemon startup
- Process reuse for multiple scans

### Key Benefits
- **Faster Scanning**: Database already loaded in memory
- **Memory Efficiency**: Shared database across scanning processes
- **Better Resource Usage**: Less CPU for database loading
- **Improved Throughput**: Especially beneficial for multi-threaded scans

## Smart Daemon Management

The task intelligently manages the ClamAV daemon:

1. **Existing Daemon Detection**: Checks if `clamd` is already running
   - Searches common socket locations: `/var/run/clamd.scan/clamd.sock`, `/var/run/clamd.sock`, etc.
   - Tests daemon responsiveness with clamdscan commands
   - Uses existing daemon if available and responsive

2. **Automatic Fallback**: Starts its own daemon if needed
   - Creates custom configuration for optimal scanning
   - Manages daemon lifecycle (start/stop)
   - Ensures proper cleanup on task completion

3. **Configuration Options**:
   - MaxScanSize: 4095MB (4GB files)
   - MaxFileSize: 2000MB for individual files
   - Comprehensive recursion and parsing limits
   - Optimized for container image scanning

## Parameters

All parameters maintain 100% compatibility with version 0.2:

- `image-url`: Container image URL to scan
- `image-digest`: Image digest for precise scanning
- `scan-threads`: Number of parallel scanning threads (default: 1, max: 8)
- `max-scan-threads`: Maximum allowed scanning threads (default: 8)
- `ca-trust-config-map-name`: Certificate bundle ConfigMap (default: trusted-ca)
- `ca-trust-config-map-key`: Certificate bundle key (default: ca-bundle.crt)

## Multi-threaded Scanning

Version 0.3 supports efficient multi-threaded scanning:

- **File Distribution**: Files sorted by size and distributed across threads
- **Load Balancing**: Round-robin distribution for even workload
- **Parallel Processing**: Multiple `clamdscan` processes against single daemon
- **Result Aggregation**: All scan results combined into unified output

## Enterprise Contract Integration

Maintains full compatibility with Enterprise Contract (EC) policies:
- Structured JSON output for policy evaluation
- Virus check policy: `/project/clamav/virus-check.rego`
- AppStudio output format support
- Automated policy validation

## Migration from 0.2

No changes required! Version 0.3 is a drop-in replacement:
- Same task interface and parameters
- Same log output format
- Same Enterprise Contract integration
- Same artifact upload behavior

The only difference is improved performance through `clamdscan` usage.

## Image Architecture Support

Supports multi-architecture images:
- Extracts and scans each architecture separately
- Handles Image Index manifests properly
- Provides per-architecture scan results
- Aggregates results across all architectures

## Security Features

- **Offline Operation**: No external network dependencies during scanning
- **Virus Database Verification**: Cryptographically verified definitions
- **Secure Image Supply Chain**: Daily builds from trusted sources
- **Enterprise Grade**: Built on Red Hat UBI9 minimal base image
- **Comprehensive Scanning**: Detects viruses, malware, and threats
- **Custom Whitelisting**: Filters known false positives

## Performance Optimizations

### Auto-Threading
- **Default**: Uses `auto` to detect optimal thread count (75% of available CPUs)
- **Manual Override**: Specify exact thread count if needed
- **Smart Scaling**: Automatically adapts to container resources

### Resource Utilization
- **Memory**: Optimized for 16GB containers with increased limits
- **CPU**: Scales up to 8 CPU cores with enhanced performance
- **I/O**: Parallel file discovery and intelligent load balancing
- **Priority**: Uses `nice -n -10` for higher process priority

### Advanced Features
- **Intelligent Load Balancing**: Files distributed by size for optimal performance
- **Parallel File Discovery**: Uses GNU parallel when available for faster file enumeration
- **Optimized Sorting**: Multi-threaded sorting with increased memory buffer
- **Stream Processing**: Uses `--fdpass` and `--stream` for faster data transfer
- **Progress Monitoring**: Real-time status updates during long scans

### Performance Recommendations

- **Auto-threading** (default): Optimal for all scenarios, auto-detects best configuration
- **High-CPU environments**: Will automatically use up to 8 threads
- **Memory allocation**: Optimized for 16GB+ containers
- **CPU allocation**: Scales from 4-8 cores for maximum performance

## Example Usage

```yaml
# Auto-optimized scanning (recommended)
- name: clamav-scan
  taskRef:
    name: clamav-scan
    kind: Task
  params:
    - name: image-url
      value: $(params.output-image)
    - name: image-digest
      value: $(tasks.build-image.results.IMAGE_DIGEST)
    # scan-threads: "auto" is the default - automatically detects optimal thread count

# Manual thread configuration (if needed)
- name: clamav-scan
  taskRef:
    name: clamav-scan
    kind: Task
  params:
    - name: image-url
      value: $(params.output-image)
    - name: image-digest
      value: $(tasks.build-image.results.IMAGE_DIGEST)
    - name: scan-threads
      value: "6"  # Manually specify thread count
    - name: max-scan-threads
      value: "8"  # Override maximum threads if needed
```

## Troubleshooting

### Common Issues

1. **Daemon Connection Issues**:
   - Check if socket file exists and is accessible
   - Verify clamd daemon is running and responsive
   - Review `/tmp/clamd.log` for daemon startup errors

2. **Performance Issues**:
   - Increase memory allocation for large images
   - Adjust thread count based on available CPU cores
   - Monitor resource usage during scanning

3. **Database Issues**:
   - Ensure latest container image is being used
   - Check for proper virus database loading
   - Verify database directory permissions

### Debugging Steps

1. Enable verbose logging in clamd configuration
2. Check daemon socket accessibility
3. Test manual `clamdscan` command
4. Review task logs for error messages
5. Verify image extraction success

## Version History

- **0.1**: Original implementation with sidecar pattern
- **0.2**: Simplified single-container approach with `clamscan`
- **0.3**: Daemon-based scanning with `clamdscan` for improved performance 