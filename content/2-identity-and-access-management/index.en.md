---
title : "Identity and Access Management"
weight : 30
---

### Identity and Access Management

[Identity and Access Man[agement (IAM)](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) is an AWS service that performs two essential functions: Authentication and Authorization. Authentication involves the verification of a identity whereas authorization governs the actions that can be performed by AWS resources. Within AWS, a resource can be another AWS service, e.g. EC2, or an AWS [principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html#intro-structure-principal) such as an [IAM User](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html#id_iam-users) or [Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html#id_iam-roles). The rules governing the actions that a resource is allowed to perform are expressed as IAM [policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html). 


### Controlling Access to EKS Clusters

The Kubernetes project supports a variety of different strategies to authenticate requests to the kube-apiserver service, e.g. Bearer Tokens, X.509 certificates, OIDC, etc. EKS currently has native support for [webhook token authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication), [service account tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens), and as of February 21, 2021, OIDC authentication.

The webhook authentication strategy calls a webhook that verifies bearer tokens. On EKS, these bearer tokens are generated by the AWS CLI or the [aws-iam-authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator) client when you run kubectl commands. As you execute commands, the token is passed to the kube-apiserver which forwards it to the authentication webhook. If the request is well-formed, the webhook calls a pre-signed URL embedded in the token's body. This URL validates the request's signature and returns information about the user, e.g. the user's account, Arn, and UserId to the kube-apiserver. 

 Each token starts with k8s-aws-v1. followed by a base64 encoded string.To manually generate a authentication token, type the following command in a terminal window.
 
  The string, when decoded, should resemble this. 

```bash
TOKEN_DATA=$(aws eks get-token --cluster-name eksworkshop-eksctl | jq -r '.status.token')
echo $TOKEN_DATA
IFS='.' read -r -a array <<< "$TOKEN_DATA"
echo "${array[1]}"==== | fold -w 4 | sed '$ d' | tr -d '\n' | base64 --decode
```

The output looks like something like this.

::expand[https://sts.us-west-2.amazonaws.com/?Action=GetCallerIdentity&Version=2011-06-15&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAQAHCJ2QPBFJYF6M5%2F20230228%2Fus-west-2%2Fsts%2Faws4_request&X-Amz-Date=20230228T075742Z&X-Amz-Expires=60&X-Amz-SignedHeaders=host%3Bx-k8s-aws-id&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEMj%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQDG2ef8SQiWF04CqAz4u3JLQWs%2Bhfk7si8jVKQwbGRfDAIgChTi5my0BugBetYDzW4P%2B8nD6CLiSKzMMjG2bUZ6ankqsgUIcBADGgwwMDA0NzQ2MDA0NzgiDFh6XvFedD%2BwWObAzCqPBeAKIDUDWZN1rvrRwapBl7eFVUJJL%2FWLtnFY%2FNmMl6cb9HgSvP%2BeRZpX%2FLOkhNT2MYSEo%2BPKJ2%2FQf0%2FrGfLOQ1JS52rtKuobXFOTLnB%2BBFoal%2B%2FJmEGqh9z71FRCYHAVEhKST3F2N1JAT7BHV3vyt9QaSz%2BmdreVclJkyFAVqxXxdySVZHtZdnwetJ0YbU9dxI0Neuilu5XBR79owkwJ3cdDgz4tYSzC7JcOmrsvb2IQFLhFlZyWercXZICSYEZpTLwuFNMN4nci7B4mrSTfz5bZheYOQ%2F2vdu3kq1j2CapDrIpgpLYhe1slCSBetLPu6ZdMp8nIsfMwOjtSrqGgrLxKlP2wNdGwUE4EwODXJOM%2FeJHqooGFBpScle%2FLCg9Et4PCrnPaJ7zxltWtcT1l7rWOnlXVTWTC17%2FG3K5qlkWbdJMt40xXGzfdxP5Mf0ykmMQ1%2BfHnsgFoiRDTcHTBAHbgavy27kUI8%2FCUk6WSZQeLKTQplI6GQBAfRtMI38H2Eh1iugsvRWlNaCTHpTLjYZ7vkzFrgqXK6YlulCJ6dwzRqzTrK9UZn8EzEsphuEejxwOJ2Rkc7FSwH1%2BNo4KHJgbe5hVv3xOpWxDqWjvet3c5056MtbJVxhm2N0Z1ZaYYNdEdv%2Bvfkyy7c9Bd6fOsCPIYUHiOMmvGNYrAbqQgp0VC7U4%2FVO1%2BYQ0N6u4Arjns98MVrhOOfcl0%2BMbQkSrs8m6Z3OBq1c%2FYL1BTMKMgTwGpmOzOnmbxna58M%2F6T3ZD2%2F03g%2BcKx7dxipEGaC08rzggJI43EPLD5%2BD%2FfDjxWDCPevBPBztU1HSYLdJFMnXkWeHwzDor75Vit4hgjAWzRrU%2F5bgfHeohclJpLtvHDhTAws9X2nwY6sgHfBDGEhFkvcFmnasbtv%2BuUBWtFBMgSPdrwy0SebbkxhbtPeozsM21w8yPbDQgQWq0ETNVDx1sHrw%2F0eF6GX8NOYsKNM6xD3y7Y7TSLVIcsdnjms6VUsnl1VHMiUsO196%2B8m3NdMYwkMFwxiqbfOsX6YaxavuFxH37%2BDL28rSiMXHdT%2BNc2VJFO2HPMnllfISh1FpfPeKmogjpwreQfEiydSunHrUCwy1qW5FL3YyYpAN89&X-Amz-Signature=4b0e9fc4867a0a9f9c0044e9f167ae2a1e64e192a33e622e578a575098e52092]{header="Expand to see Sample Output"}

The token consists of a pre-signed URL that includes an Amazon credential and signature. For additional details see https://docs.aws.amazon.com/STS/latest/APIReference/API_GetCallerIdentity.html.

The token has a time to live (TTL) of 15 minutes after which a new token will need to be generated. This is handled automatically when you use a client like kubectl, however, if you're using the Kubernetes dashboard, you will need to generate a new token and re-authenticate each time the token expires.

Once the user's identity has been authenticated by the AWS IAM service, the kube-apiserver reads the aws-auth ConfigMap in the kube-system Namespace to determine the RBAC group to associate with the user. The aws-auth ConfigMap is used to create a static mapping between IAM principals, i.e. IAM Users and Roles, and Kubernetes RBAC groups. RBAC groups can be referenced in Kubernetes RoleBindings or ClusterRoleBindings. They are similar to IAM Roles in that they define a set of actions (verbs) that can be performed against a collection of Kubernetes resources (objects).
