# AWS Secrets Manager on EKS

The shared chart can render an External Secrets Operator `SecretStore` and `ExternalSecret`. The store configures AWS access through a Kubernetes service account. The external secret maps remote secret names to keys in a Kubernetes Secret.

## Configure IAM

1. Get the cluster OIDC issuer and associate it with an IAM OIDC provider. [Amazon EKS IRSA documentation](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)

   ```sh
   aws eks describe-cluster --name example-cluster --region us-east-2 --query 'cluster.identity.oidc.issuer' --output text
   ```

2. Replace the account, region, issuer ID, namespace, and service account placeholders in [trust-policy.json](trust-policy.json).
3. Use [secretsmanager-readonly-policy.json](secretsmanager-readonly-policy.json) as a starting point and scope the policy to the specific secrets the store reads.
4. Create the IAM role with the trust policy, attach the permissions policy, and annotate the Kubernetes service account with `eks.amazonaws.com/role-arn`.

   From `docs/aws`, run these commands after replacing the placeholders in the JSON examples. Replace `ACCOUNT_ID` in the final command with your AWS account ID.

   ```sh
   aws iam create-policy --policy-name eks-secretsmanager-readonly --policy-document file://secretsmanager-readonly-policy.json
   aws iam create-role --role-name eks-irsa-secrets-reader --assume-role-policy-document file://trust-policy.json
   aws iam attach-role-policy --role-name eks-irsa-secrets-reader --policy-arn arn:aws:iam::ACCOUNT_ID:policy/eks-secretsmanager-readonly
   ```

## Configure the application chart

Install External Secrets Operator with CRDs that serve `external-secrets.io/v1`. In the application values file, configure these settings under `app:`. [External Secrets Operator AWS provider documentation](https://external-secrets.io/latest/provider/aws-secrets-manager/)

| Key | Setting |
| --- | --- |
| `serviceAccount` | Enable it to create the service account. Set its name and IAM role annotation. |
| `secretStore` | Enable it, select provider type `aws`, and set the region and service account name. Use an empty `provider.aws.role` when the service account's role already has access. This field is for assuming a separate role. |
| `externalSecret` | Enable it, set the namespace, reference the store, and map each `remoteKey` to a Kubernetes `secretKey`. The generated Secret has the same name as the ExternalSecret. |
| `deployment.container.envFrom` | Reference the generated Secret if the workload reads its keys as environment variables. |

See the [shared chart values](../../helm/app/values.yaml) for the complete fields. Check synchronization in the application's namespace.

```sh
kubectl get secretstore,externalsecret -n example-app
kubectl describe externalsecret example-app-external-secret -n example-app
kubectl get secret example-app-external-secret -n example-app
```
