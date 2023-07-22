---
title : "Container Image Signing and Verification with AWS Signer"
weight : 35
---

[AWS Signer](https://docs.aws.amazon.com/signer/latest/developerguide/Welcome.html) and [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/) [launched image signing](https://aws.amazon.com/about-aws/whats-new/2023/06/aws-container-image-signing/), a new feature that enables you to sign and verify container images. You can now use Signer, a managed signing service, to validate that only container images you have approved are deployed in your Amazon Elastic Kubernetes Service (EKS) clusters. 


You can use container image signing to help ensure the use of approved images inside your organization, which can help you meet your security and compliance requirements.  You begin by creating a signing profile, a unique AWS Signer identity, to cryptographically sign images in your repository with client-side tools. Signer manages the signing keys, rotates code signing certificates, provides audit logs, and stores the signatures alongside your images. Amazon EKS and Kubernetes customers can choose their preferred admission controllers – like [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) or [Kyverno](https://kyverno.io/), or develop their own tooling – to help enforce image verification before deploying images.


AWS Signer now supports signing and verifying container images, and is integrated with [Notation](https://github.com/notaryproject/notation), an open source Notary project within the [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/). Notary is an open standard and client implementation that allows for vendor-specific plugins for key management and other integrations.

Notation uses new [Open Containers Initiative (OCI)}(https://opencontainers.org/) distribution features built into Amazon ECR that enable you to store signatures and other artifacts in the registry right alongside the images they refer to. This allows you to sign your container images with a simple command that handles the interaction with Amazon ECR transparently for you, while AWS Signer manages signing material and simplifies lifecycle management and revocation operations.

## What is a cryptographic signature?

As you read this module in your web browser, you’ll find a lock icon in your browser’s address bar, indicating the connection to this page is secured by transport layer security (TLS). Your browser automatically trusts the certificate used to establish the TLS connection, as the certificate has a verifiable chain of trust that extends up to a trusted root certificate that is installed in all major operating systems and web browsers—in this case, the root certificate for Amazon. A similar trust mechanism is used to verify the identity of the person or organization who creates a cryptographic signature of a container image.

A typical cryptographic signature consists of (at least) two components: an encrypted hash of the data to be signed, and a certificate that provides identity information about the signer. In the simplest terms, a hash is a one-way (i.e. irreversible) function that takes data of any size and outputs a fixed length, unique string. Hashes are deterministic, meaning the same data will always produce the same hash, and any change to the data (however small) will produce a different hash from the original. This latter point is important when we consider that we are verifying the integrity of a container image: if the image has changed at all, the resulting hash of that image manifest will also change.

The hash of our original file would be encrypted with the private key of the signer, which should be treated with care: this private key uniquely identifies the signer and ensures the signature cannot be generated by anyone other than the signer. 

The below diagram shows the generic signing process in action:

![crypto-1](/static/images/image-security/image-signing/crypto-1.jpg)

In cryptography, this signature can then be checked against the artifact being validated to tell if the data has been modified since it was signed by decrypting the signature with the public key (typically included in the certificate along with the signature) and comparing the resulting decrypted hash to a hash of the original file. If the two hashes do not match, the original data has been changed in some way since it was signed. We can see this mechanism in action in below diagram.

![crypto-2](/static/images/image-security/image-signing/crypto-2.jpg)

the signature can also provide what’s called non-repudiation: the idea that the signer cannot deny having signed the file. Access to the signing key is needed to generate the signature. In practice, this means that someone verifying the signature of a file can be reasonably sure the file was actually authored or signed by the signer.


## Cryptographic Signing for Containers

Container image and artifact is described by a manifest, which is referenced by a digest that is a hash of its content. This enables clients to easily verify the **integrity** of a given image. Signing establishes **authenticity and provenance**, giving you the ability to determine if content comes from a particular party. This allows you to only allow images from trusted parties, and you can sign your own content to establish deployment policies.

The container image signing approach implemented in Notation leverages container image integrity features, and uses a simple mechanism to cryptographically sign an **image manifest**, rather than its layers. Image manifests are verifiable records of image content, and contain hashed digests for all layers of the image they describe. It’s as effective and much faster to sign the image digest rather than image layers, and this method works for remote images without needing to pull them fully to your signing environment. Once a signature is created, it is encoded as an **OCI artifact** and pushed to the image repository alongside the container image. At any point in time, a client can discover signatures for a given image and verify them against a trusted content publisher’s identity. While this module primarily covers signing container image manifests and related artifacts, cryptographic signatures can also be used to sign/verify documents, authentication tokens, software packages, and more.

Container image verification provides a logical gate for building trusted content policies for your deployments, for example with Kyverno or OPA Gatekeeper. This helps you ensure that only vetted and trusted images are running in your deployed workloads. Without image verification, policies can mandate that only images from certain registries or repositories can be deployed, or that only certain image tags are allowed to deploy in production environments. With image verification, you can go one step further and bring the trust of content into your policy decisions, only allowing use of images from known and vetted sources.

AWS Signer brings fully managed signing and verification features for container images to a simple workflow through its integration with Notation. In this workshop, we will learn how simple it is to get started with Signer and Notation, with simple commands to sign and verify your images.


## Container Image Signing and AWS Signer

Organizations validate images against a digital signature to confirm that the image is unaltered and from a trusted publisher. AWS Signer allows security administrators to have a single place to define your signing environment, including what [AWS Identity and Access Management (AWS IAM) role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) can sign code and in what regions. Integration with [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/) helps you track who is generating signatures and helps meet your compliance requirements.

AWS Signer supports features like cross account signing, signature validity duration, and profile lifecycle management with cancellation and revocation operations.

Let us now proceed to the section [Digital signature verification using openssl tool](1-digital-signature-using-openssl)