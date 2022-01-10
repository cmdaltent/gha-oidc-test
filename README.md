# Testing out GitHub OpenID Connect w/ Amazon AWS

We simply follow the [official instructions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services).

This solution does not require any (long-lived) secrets being stored in GitHub.
If only, the arn of the role to be assumed can be stored as GitHub secrets, which prevents the AWS account id from
being publicly accessible in the repository.

## Accessing AWS

As a minimal working example, we

* created the [GitHub OIDC provider](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#adding-the-identity-provider-to-aws)
  in AWS
* created a [role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html)
* connected the role to the GitHub OIDC provider
* gave the role permission to access the Parameter Store of the AWS Systems Manager via the `AmazonSSMReadOnlyAccess`
  policy
* created a parameter `HelloWorld` in the AWS Systems Manager Parameter Store, which will be read in the workflow
* granted the entire repository (`repo:cmdaltent/gha-oidc-test:*`)
  [permission](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services#configuring-the-role-and-trust-policy)
  to access AWS via the OpenID Connect and
  the created role

## Lessons Learnt

If a repository as a whole is supposed to be granted access, the following condition for the trust relationship
between OIDC provider and AWS role does **not** work:

```json
"Condition": {
  "StringEquals": {
    "token.actions.githubusercontent.com:sub": "repo:cmdaltent/gha-oidc-test:*"
  }
}
```

Instead, The `StringLike` comparator worked for us:

```json
"Condition": {
  "StringLike": {
    "token.actions.githubusercontent.com:sub": "repo:cmdaltent/gha-oidc-test:*"
  }
}
```
