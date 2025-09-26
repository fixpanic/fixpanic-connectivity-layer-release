# Private Repository Update Guide

## 🎯 **Mission: Update Private Repository to Send Artifact URLs**

The public repository now uses **GitHub REST API** to download artifacts from the private repository. This requires the private repository to send **artifact download URLs** in the repository dispatch payload.

## 🔧 **What Needs to Change**

The private repository's [`public-release.yml`](.github/workflows/public-release.yml) workflow needs to:

1. **Get artifact URLs** after uploading artifacts
2. **Send enhanced payload** with download URLs to the public repository
3. **Include proper authentication** for cross-repository access

## 📋 **Complete Private Repository Workflow Update**

Replace the current repository dispatch step with this enhanced version:

```yaml
      - name: Get artifact URLs and send dispatch
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.RELEASE_PAT }}
          script: |
            console.log('=== Getting artifact URLs and sending dispatch ===');
            
            // Get the current workflow run ID
            const runId = context.runId;
            console.log(`Run ID: ${runId}`);
            
            try {
              // List artifacts from this workflow run
              const artifactsResponse = await github.rest.actions.listWorkflowRunArtifacts({
                owner: context.repo.owner,
                repo: context.repo.repo,
                run_id: runId
              });
              
              console.log('Artifacts found:', artifactsResponse.data.artifacts.length);
              
              // Build artifacts payload
              const artifacts = {};
              
              for (const artifact of artifactsResponse.data.artifacts) {
                console.log(`Processing artifact: ${artifact.name}`);
                
                // Get download URL for each artifact
                const downloadResponse = await github.rest.actions.downloadArtifact({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  artifact_id: artifact.id,
                  archive_format: 'zip'
                });
                
                // The download URL is in the response
                const downloadUrl = downloadResponse.url;
                console.log(`Download URL for ${artifact.name}: ${downloadUrl}`);
                
                // Extract platform info from artifact name
                const match = artifact.name.match(/fixpanic-agent-(linux|darwin|windows)-(amd64|arm64)/);
                if (match) {
                  const platform = `${match[1]}-${match[2]}`;
                  artifacts[platform] = {
                    name: artifact.name,
                    download_url: downloadUrl,
                    artifact_id: artifact.id
                  };
                }
              }
              
              console.log('Artifacts payload:', JSON.stringify(artifacts, null, 2));
              
              // Send repository dispatch with artifact URLs
              await github.rest.repos.createDispatchEvent({
                owner: 'fixpanic',
                repo: 'fixpanic-connectivity-layer-release',
                event_type: 'create-release',
                client_payload: {
                  tag: '${{ github.ref_name }}',
                  ref: '${{ github.ref }}',
                  run_id: runId,
                  artifacts: artifacts
                }
              });
              
              console.log('✅ Repository dispatch sent with artifact URLs');
              
            } catch (error) {
              console.error('❌ Error getting artifacts or sending dispatch:', error.message);
              throw error;
            }
```

## 🧪 **Testing the Enhanced Workflow**

### **1. Test with Enhanced Payload**
```bash
# Test the new enhanced payload format
gh api repos/fixpanic/fixpanic-connectivity-layer-release/dispatches \
  --field event_type="create-release" \
  --field client_payload[tag]="v1.0.9-test-enhanced" \
  --field client_payload[ref]="refs/tags/v1.0.9-test-enhanced" \
  --field client_payload[run_id]="123456789" \
  --field client_payload[artifacts][linux-amd64][name]="fixpanic-agent-linux-amd64" \
  --field client_payload[artifacts][linux-amd64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456789/zip" \
  --field client_payload[artifacts][linux-amd64][artifact_id]="123456789" \
  --field client_payload[artifacts][linux-arm64][name]="fixpanic-agent-linux-arm64" \
  --field client_payload[artifacts][linux-arm64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456790/zip" \
  --field client_payload[artifacts][linux-arm64][artifact_id]="123456790" \
  --field client_payload[artifacts][darwin-amd64][name]="fixpanic-agent-darwin-amd64" \
  --field client_payload[artifacts][darwin-amd64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456791/zip" \
  --field client_payload[artifacts][darwin-amd64][artifact_id]="123456791" \
  --field client_payload[artifacts][windows-amd64][name]="fixpanic-agent-windows-amd64" \
  --field client_payload[artifacts][windows-amd64][download_url]="https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456792/zip" \
  --field client_payload[artifacts][windows-amd64][artifact_id]="123456792"
```

### **2. Monitor the Results**
- **Private repo workflow**: Check for "✅ Repository dispatch sent with artifact URLs"
- **Public repo workflow**: Look for "✅ Downloaded fixpanic-agent-*.zip" messages
- **Final release**: Verify all binaries are uploaded to GitHub releases

## 🔍 **Expected Debug Output**

### **Private Repository Logs:**
```
=== Getting artifact URLs and sending dispatch ===
Run ID: 123456789
Artifacts found: 4
Processing artifact: fixpanic-agent-linux-amd64
Download URL for fixpanic-agent-linux-amd64: https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456789/zip
Processing artifact: fixpanic-agent-linux-arm64
Download URL for fixpanic-agent-linux-arm64: https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456790/zip
✅ Repository dispatch sent with artifact URLs
```

### **Public Repository Logs:**
```
=== DEBUG: Received payload ===
Artifacts: {
  "linux-amd64": {
    "name": "fixpanic-agent-linux-amd64",
    "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456789/zip",
    "artifact_id": "123456789"
  },
  "linux-arm64": {
    "name": "fixpanic-agent-linux-arm64",
    "download_url": "https://api.github.com/repos/fixpanic/fixpanic-connectivity-layer/actions/artifacts/123456790/zip",
    "artifact_id": "123456790"
  }
}
✅ Downloaded fixpanic-agent-linux-amd64.zip
✅ Extracted fixpanic-agent-linux-amd64
✅ Found binary: fixpanic-agent-linux-amd64
✅ Release bundle created successfully
```

## ⚠️ **Fallback Support**

The public repository workflow still supports **minimal payloads** for backward compatibility:

```json
{
  "tag": "v1.0.9",
  "ref": "refs/tags/v1.0.9"
}
```

In this case, the workflow will:
1. ❌ Fail to download artifacts (expected)
2. ✅ Still create the GitHub release
3. ⚠️ Show clear error messages about missing artifacts

## 🎯 **Success Criteria**

✅ **Private repository sends enhanced payload** with artifact URLs
✅ **Public repository downloads artifacts** using GitHub REST API
✅ **Cross-repository authentication** works with PAT token
✅ **All platform binaries** are successfully transferred
✅ **Professional release** created with checksums and manifest
✅ **Clear debug logs** show the complete process

## 🚀 **Implementation Steps**

1. **Update private repository** with the enhanced workflow code
2. **Test with a new tag** to trigger the complete flow
3. **Monitor both repositories** for success messages
4. **Verify final release** contains all binaries and metadata

This approach **guarantees** cross-repository artifact transfer using the **official GitHub REST API** with proper authentication! 🎉