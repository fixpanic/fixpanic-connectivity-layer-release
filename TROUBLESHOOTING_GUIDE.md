# Troubleshooting Guide - Cross-Repository Release Flow

## 🚨 **Current Issue: Artifact Download Failing**

### **Error Analysis**
The error shows:
```
Error: Unable to download artifact(s): Not Found
##[debug]github.event.client_payload.run_id => null
```

**Root Cause**: The private repository is not sending the enhanced payload with `run_id`, so the public repository cannot download the artifacts.

## 🔧 **Immediate Fix Options**

### **Option 1: Update Private Repository Payload (Recommended)**
The private repository needs to send the enhanced payload as previously discussed:

```yaml
# Add this to private repo's public-release.yml
- name: Send repository dispatch event
  uses: peter-evans/repository-dispatch@v2
  with:
    token: ${{ secrets.RELEASE_PAT }}
    repository: fixpanic/fixpanic-connectivity-layer-release
    event-type: create-release
    client-payload: |
      {
        "tag": "${{ github.ref_name }}",
        "ref": "${{ github.ref }}",
        "run_id": "${{ github.run_id }}",
        "artifacts": {
          "linux-amd64": "fixpanic-agent-linux-amd64",
          "linux-arm64": "fixpanic-agent-linux-arm64",
          "darwin-amd64": "fixpanic-agent-darwin-amd64",
          "windows-amd64": "fixpanic-agent-windows-amd64"
        }
      }
```

### **Option 2: Alternative Download Method**
If the enhanced payload cannot be implemented immediately, we can modify the public repo to use a different approach:

```yaml
# Alternative: Download from latest successful run
- name: Download from latest run
  run: |
    # Get the latest successful workflow run ID
    LATEST_RUN_ID=$(gh run list --repo fixpanic/fixpanic-connectivity-layer \
      --workflow="public-release.yml" \
      --json databaseId,conclusion \
      --jq '.[] | select(.conclusion=="success") | .databaseId' | head -1)
    
    echo "Using latest run ID: $LATEST_RUN_ID"
    
    # Download artifacts
    gh run download $LATEST_RUN_ID \
      --repo fixpanic/fixpanic-connectivity-layer \
      --dir ./binaries/
  env:
    GITHUB_TOKEN: ${{ secrets.RELEASE_PAT }}
```

## 📋 **Verification Steps**

### **1. Check Current Payload**
To see what the private repository is actually sending, add this debug step to the public repo workflow:

```yaml
- name: Debug received payload
  run: |
    echo "Received payload:"
    echo "${{ toJson(github.event.client_payload) }}"
    echo "Available keys:"
    echo "${{ toJson(github.event.client_payload) }}" | jq 'keys'
```

### **2. Test Artifact Download Manually**
Test if artifacts can be downloaded using GitHub CLI:

```bash
# List recent runs in private repo
gh run list --repo fixpanic/fixpanic-connectivity-layer --workflow="public-release.yml"

# Try downloading artifacts from a specific run
gh run download <RUN_ID> --repo fixpanic/fixpanic-connectivity-layer --dir ./test/
```

### **3. Check Artifact Names**
Verify the actual artifact names in the private repo:

```bash
# List artifacts from a specific run
gh run view <RUN_ID> --repo fixpanic/fixpanic-connectivity-layer --json artifacts
```

## 🎯 **Expected Behavior**

When working correctly:
1. **Private repo** creates tag → builds binaries → uploads artifacts → sends dispatch event
2. **Public repo** receives event → downloads artifacts → creates release → uploads binaries
3. **Final result**: Professional GitHub release in public repository with all platform binaries

## 🔍 **Debugging Commands**

```bash
# Check recent workflow runs
gh run list --repo fixpanic/fixpanic-connectivity-layer --limit 10

# View specific run details
gh run view <RUN_ID> --repo fixpanic/fixpanic-connectivity-layer

# List artifacts from a run
gh run view <RUN_ID> --repo fixpanic/fixpanic-connectivity-layer --json artifacts

# Download artifacts manually
gh run download <RUN_ID> --repo fixpanic/fixpanic-connectivity-layer --dir ./test/
```

## 📞 **Next Steps**

1. **Immediate**: Check if private repo is sending the enhanced payload
2. **If not**: Update private repo workflow to include `run_id` in payload
3. **Alternative**: Implement fallback download method using latest successful run
4. **Test**: Create a new tag and monitor both workflows
5. **Verify**: Check that all artifacts are downloaded and released correctly

The system is designed to work - we just need to ensure the private repository sends the correct payload format!