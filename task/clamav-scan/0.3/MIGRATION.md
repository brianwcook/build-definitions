# Migration Guide: ClamAV Scan 0.2 → 0.3

## Overview

ClamAV Scan 0.3 is a **drop-in replacement** for 0.2 with significant performance improvements. No configuration changes are required.

## Key Changes

### ✅ What Stays the Same
- **All parameters** remain identical
- **Log format** unchanged
- **Enterprise Contract integration** preserved
- **Task interface** fully compatible
- **Results structure** unchanged
- **Artifact upload** behavior identical

### 🚀 What's Improved

#### Performance Enhancement
- **Before (0.2)**: `clamscan` loads virus database for each scan process
- **After (0.3)**: `clamdscan` uses cached database in memory via `clamd` daemon
- **Result**: Faster scanning, especially for multi-threaded operations

#### Smart Daemon Management
- **Auto-detection**: Checks if `clamd` is already running
- **Fallback**: Starts own daemon if needed
- **Lifecycle management**: Automatic cleanup on task completion

#### Better Resource Usage
- **Memory efficiency**: Shared virus database across scanning processes
- **CPU optimization**: Less database loading overhead
- **Improved throughput**: Better performance on large images

## Migration Steps

### For Pipeline Users
**No action required!** Simply update the task version:

```yaml
# Before
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
      value: "4"

# After (identical)
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
      value: "4"
```

### For Bundle Users
Update your bundle to reference version 0.3:

```yaml
# Before
resolver: bundles
params:
  - name: bundle
    value: quay.io/konflux-ci/tekton-catalog/task-clamav-scan:0.2

# After
resolver: bundles
params:
  - name: bundle
    value: quay.io/konflux-ci/tekton-catalog/task-clamav-scan:0.3
```

### For Git Resolver Users
Update the revision to use 0.3:

```yaml
# Before
resolver: git
params:
  - name: url
    value: https://github.com/konflux-ci/build-definitions.git
  - name: revision
    value: main
  - name: pathInRepo
    value: task/clamav-scan/0.2/clamav-scan.yaml

# After
resolver: git
params:
  - name: url
    value: https://github.com/konflux-ci/build-definitions.git
  - name: revision
    value: main
  - name: pathInRepo
    value: task/clamav-scan/0.3/clamav-scan.yaml
```

## Verification

After migration, verify the task works as expected:

1. **Check task logs** for successful daemon startup
2. **Verify scan results** match previous behavior
3. **Monitor performance** improvements
4. **Ensure EC policies** still pass correctly

## Expected Performance Improvements

### Single-threaded Scanning
- **Startup time**: Faster after daemon initialization
- **Memory usage**: More efficient database loading
- **CPU usage**: Reduced database loading overhead

### Multi-threaded Scanning
- **Throughput**: Significant improvement with shared database
- **Resource usage**: Better CPU and memory utilization
- **Scalability**: More efficient with higher thread counts

## Troubleshooting

### If you encounter issues after migration:

1. **Check daemon logs**:
   ```bash
   # Look for clamd startup messages in task logs
   grep -i "clamd" <task-log>
   ```

2. **Verify socket connection**:
   ```bash
   # Task logs should show successful daemon detection
   grep -i "socket\|daemon\|ping" <task-log>
   ```

3. **Compare scan results**:
   - Results should be identical to 0.2
   - Same threat detection capabilities
   - Same EC policy compliance

### Common Issues

1. **Permission errors**: Ensure proper container security context
2. **Socket issues**: Check if daemon starts successfully
3. **Database loading**: Verify virus database is accessible

## Rollback Plan

If needed, rollback is simple:

```yaml
# Rollback to 0.2
resolver: bundles
params:
  - name: bundle
    value: quay.io/konflux-ci/tekton-catalog/task-clamav-scan:0.2
```

## Questions?

For issues or questions:
1. Check the [README.md](README.md) for detailed documentation
2. Review task logs for error messages
3. File issues in the build-definitions repository

## Summary

ClamAV Scan 0.3 provides significant performance improvements with zero configuration changes. The migration is seamless and backward-compatible, with enhanced performance being the primary benefit. 