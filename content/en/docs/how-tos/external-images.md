---
title: "Using External Images in CI"
description: How to mirror external images to the CI environments for use in jobs.
---

The `ci-operator` config only allows to reference `ImageStreamTags`, it does not allow to specify arbitrary Docker pull specs. In order
to use external images, they need to be mirrored to [the central CI registry](/docs/how-tos/use-registries-in-build-farm/#summary-of-available-registries).

## Mirror Public Images

If the source image is open to the public, we can mirror the image by adding it into a mapping file in
[the `core-services/image-mirroring/supplemental-ci-images` folder](https://github.com/openshift/release/tree/master/core-services/image-mirroring/supplemental-ci-images/) of `openshift/release` repository. The following line in the `mapping_supplemental_ci_images_ci` file mirrors 
`gcr.io/k8s-staging-boskos/boskos:latest` to `registry.ci.openshift.org/ci/boskos:latest`. The naming convention of the mapping file is `mapping_supplemental_ci_images_<namespace>`, e.g., the images in `mapping_supplemental_ci_images_ci` are mirrored to the namespace `ci`.

{{< highlight text >}}
gcr.io/k8s-staging-boskos/boskos:latest registry.ci.openshift.org/ci/boskos:latest
{{< / highlight >}}

{{< alert title="Warning" color="warning" >}}
- We cannot mirror images from `docker.io` due to rate limiting constraints. Please, instead, push up an image to quay and mirror that to the CI registry.
- It is not possible to use Red Hat managed namespaces. Therefore, you cannot mirror your image to **any** namespace that
matches the following regular expression: (^kube.*|^openshift.*|^default$|^redhat.*). The mirroring job will fail if the namespace does not exist on `app.ci`.
See [the steps](/docs/how-tos/use-registries-in-build-farm/) about claiming a namespace.
- We cannot mirror images to an imagestream with the tags that are [promoted by ci-operator](/docs/architecture/ci-operator/#publishing-to-an-integration-stream). Otherwise, the target images will be cleaned up by the periodic Prow job `periodic-promoted-image-governor`.

{{< /alert >}}

The hourly periodic job [`periodic-image-mirroring-supplemental-ci-images`](https://prow.ci.openshift.org/?job=periodic-image-mirroring-supplemental-ci-images)
mirrors all the images defined in the mapping files.  Note that it operates on
the contents of the `master` branch, so the changes to the files have to be
merged before the images can be used in jobs and/or pull request rehearsals.
Once it is mirrored, you can use the image like this:

{{< highlight yaml >}}
base_images:
  my-external-image:
    namespace: ci
    name:  boskos
    tag: latest
{{< / highlight >}}

## Mirror Private Images

It is not supported at the moment to mirror external private images to the central CI registry.
