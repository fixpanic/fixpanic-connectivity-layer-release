# Testing Guide - Manual Workflow Triggering

## 🎯 **How to Test Without New Private Repository Workflow**

You can manually trigger the `Receive Release` workflow using GitHub's repository dispatch API. Here are several methods:

## **Method 1: Using GitHub CLI (Recommended)**

### **Install GitHub CLI** (if not already installed)
```bash
# macOS
brew install gh

# Linux
sudo apt install gh

# Or download from https://cli.github.com/
```

### **Trigger Workflow with Enhanced Payload**
```bash
# Test with enhanced payload (with artifact URLs for GitHub REST API)
gh api repos/fixpanic/fixpanic-connectivity-layer-release/dispatches \
  --field event_type="create-release" \
  --field client_payload[tag]="v1.0.9-test-enhanced" \
  --field client_payload[ref]="refs/tags/v1.0.9-test-enhanced" \
  --field client_payload[artifacts][linux-amd64][name]="fixpanic-agent-linux-amd64" \
  --field client_payload[artifacts][linux-amd64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456789/zip" \
  --field client_payload[artifacts][linux-arm64][name]="fixpanic-agent-linux-arm64" \
  --field client_payload[artifacts][linux-arm64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456790/zip" \
  --field client_payload[artifacts][darwin-amd64][name]="fixpanic-agent-darwin-amd64" \
  --field client_payload[artifacts][darwin-amd64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456791/zip" \
  --field client_payload[artifacts][windows-amd64][name]="fixpanic-agent-windows-amd64" \
  --field client_payload[artifacts][windows-amd64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456792/zip"
```

### **Trigger with Minimal Payload** (Fallback test)
```bash
# Test with minimal payload (creates release only, no artifacts)
gh api repos/fixpanic/fixpanic-connectivity-layer-release/dispatches \
  --field event_type="create-release" \
  --field client_payload[tag]="v1.0.9-test-minimal" \
  --field client_payload[ref]="refs/tags/v1.0.9-test-minimal"
```

## **Method 2: Using curl with GitHub API**

### **Get GitHub Token**
Make sure you have a GitHub token with `repo` scope.

### **Enhanced Payload Test (with Artifact URLs)**
```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer-release/dispatches \
  -d '{
    "event_type": "create-release",
    "client_payload": {
      "tag": "v1.0.9-test-enhanced",
      "ref": "refs/tags/v1.0.9-test-enhanced",
      "artifacts": {
        "linux-amd64": {
          "name": "fixpanic-agent-linux-amd64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456789/zip"
        },
        "linux-arm64": {
          "name": "fixpanic-agent-linux-arm64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456790/zip"
        },
        "darwin-amd64": {
          "name": "fixpanic-agent-darwin-amd64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456791/zip"
        },
        "windows-amd64": {
          "name": "fixpanic-agent-windows-amd64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456792/zip"
        }
      }
    }
  }'
```

### **Minimal Payload Test** (Fallback scenario)
```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer-release/dispatches \
  -d '{
    "event_type": "create-release",
    "client_payload": {
      "tag": "v1.0.9-test-minimal",
      "ref": "refs/tags/v1.0.9-test-minimal"
    }
  }'
```

## **Method 3: Using GitHub Web Interface**

### **Manual Dispatch via GitHub UI**
1. Go to: `https://github.com/fixpanic/fixpanic-connectivity-layer-release/actions`
2. Click on **"Receive Release"** workflow
3. Click **"Run workflow"** dropdown
4. Select **"Trigger workflow via repository_dispatch"**
5. Enter the payload JSON manually

## **Method 4: Create a Test Script**

### **Create Test Script** (`test-dispatch.sh`)
```bash
#!/bin/bash

# Set your GitHub token
GITHUB_TOKEN="your_token_here"

# Test with different payload formats
echo "Testing with enhanced payload (GitHub REST API approach)..."
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer-release/dispatches \
  -d '{
    "event_type": "create-release",
    "client_payload": {
      "tag": "v1.0.9-test-enhanced",
      "ref": "refs/tags/v1.0.9-test-enhanced",
      "artifacts": {
        "linux-amd64": {
          "name": "fixpanic-agent-linux-amd64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456789/zip"
        },
        "linux-arm64": {
          "name": "fixpanic-agent-linux-arm64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456790/zip"
        },
        "darwin-amd64": {
          "name": "fixpanic-agent-darwin-amd64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456791/zip"
        },
        "windows-amd64": {
          "name": "fixpanic-agent-windows-amd64",
          "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456792/zip"
        }
      }
    }
  }'

echo ""
echo "Testing with minimal payload..."
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer-release/dispatches \
  -d '{
    "event_type": "create-release",
    "client_payload": {
      "tag": "v1.0.9-test-minimal",
      "ref": "refs/tags/v1.0.9-test-minimal"
    }
  }'

echo "Dispatches sent! Check the Actions tab for workflow runs."
```

### **Make Script Executable**
```bash
chmod +x test-dispatch.sh
./test-dispatch.sh
```

## **🔍 Monitoring the Test**

### **Check Workflow Runs**
1. Go to: `https://github.com/fixpanic/fixpanic-connectivity-layer-release/actions`
2. Look for **"Receive Release"** workflow runs
3. Click on the latest run to see detailed logs
4. Check the **"Debug received payload"** step to see artifact URLs
5. Monitor the **"Download artifact from private repo via API"** steps for success/failure

### **Check Final Release**
1. Go to: `https://github.com/fixpanic/fixpanic-connectivity-layer-release/releases`
2. Look for releases with tags like `v1.0.9-test-enhanced` or `v1.0.9-test-minimal`
3. Verify that binaries are uploaded and checksums are generated

## **⚠️ Important Notes**

1. **Authentication**: Ensure your GitHub token has `repo` scope
2. **Rate Limiting**: Be mindful of API rate limits when testing frequently
3. **Tag Conflicts**: Use unique test tags to avoid conflicts
4. **Debug Mode**: The workflow includes debug logging - check the logs for detailed information

## **🎯 Expected Results**

When successful, you should see:
- ✅ Workflow triggered and running
- ✅ Debug logs showing received payload with artifact URLs
- ✅ GitHub REST API downloading artifacts from private repository
- ✅ Professional release created with all platform binaries
- ✅ Checksums and manifest files generated
- ✅ Success confirmation in workflow logs

## **🚀 Next Steps After Testing**

Once the manual testing is successful:
1. **Update private repository** to send the enhanced payload with artifact URLs
2. **Test with real tag** from private repository
3. **Monitor both workflows** for complete end-to-end success
4. **Document the working configuration** for future reference

The manual testing approach allows you to validate the GitHub REST API implementation without needing to trigger new workflows in the private repository! 🎉