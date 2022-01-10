# Testing out GitHub OpenID Connect w/ Amazon AWS

We simply follow the [official instructions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services).

This solution does not require any (long-lived) secrets being stored in GitHub.
If only, the arn of the role to be assumed can be stored as GitHub secrets, which prevents the AWS account id from
being publicly accessible in the repository.

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
