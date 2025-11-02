# Parse Cloud Provider Cluster

Parses the cloud provider tuple format used throughout KoalaOps workflows.

## Usage

```yaml
- name: Parse cloud configuration
  uses: KoalaOps/parse-cloud-provider-cluster@v1
  id: cloud
  with:
    cloud_provider_cluster: aws/123456789/us-east-1/production
    
- name: Use parsed values
  run: |
    echo "Provider: ${{ steps.cloud.outputs.provider }}"
    echo "Account: ${{ steps.cloud.outputs.account }}"
    echo "Region: ${{ steps.cloud.outputs.location }}"
    echo "Cluster: ${{ steps.cloud.outputs.cluster }}"
    echo "Registry: ${{ steps.cloud.outputs.registry }}"
```

## Inputs

| Input | Description | Required |
|-------|-------------|----------|
| `cloud_provider_cluster` | The cloud provider, account/project, location and cluster name, in format: provider/account/location/cluster | ✅ |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `provider` | Cloud provider (aws, gcp, azure) | `aws` |
| `account` | Account/Project ID | `123456789` |
| `location` | Region/Location | `us-east-1` |
| `cluster` | Cluster name | `production` |
| `registry` | Default container registry for this cloud provider | `123456789.dkr.ecr.us-east-1.amazonaws.com` |

## Tuple Formats

### AWS
```
aws/123456789012/us-east-1/prod-cluster
```

### GCP
```
gcp/my-project-id/us-central1/gke-cluster
```

### Azure
```
azure/subscription-id/eastus/aks-cluster
```

## Examples

### Use with cloud-login
```yaml
- name: Parse cloud config
  uses: KoalaOps/parse-cloud-provider-cluster@v1
  id: cloud
  with:
    cloud_provider_cluster: ${{ inputs.cloud_tuple }}

- name: Authenticate to cloud
  uses: KoalaOps/cloud-login@v1
  with:
    provider: ${{ steps.cloud.outputs.provider }}
    account: ${{ steps.cloud.outputs.account }}
    region: ${{ steps.cloud.outputs.location }}
    cluster: ${{ steps.cloud.outputs.cluster }}
```

### Validation
```yaml
- name: Parse and validate
  uses: KoalaOps/parse-cloud-provider-cluster@v1
  id: cloud
  with:
    cloud_provider_cluster: ${{ inputs.cluster_config }}
    
- name: Validate provider
  if: steps.cloud.outputs.provider != 'aws' && steps.cloud.outputs.provider != 'gcp'
  run: |
    echo "::error::Unsupported cloud provider: ${{ steps.cloud.outputs.provider }}"
    exit 1
```

## Error Handling

The action will fail if:
- Input is not in the correct format (missing slashes)
- Required fields are empty (provider, location, cluster)
- Input contains invalid characters

## Notes

- The tuple format is standardized across all KoalaOps workflows
- Account field may be empty for some providers
- Location refers to region (AWS), location (GCP), or region (Azure)