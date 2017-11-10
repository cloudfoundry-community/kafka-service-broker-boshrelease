# Kakfa v1.0.0

The deployment manifest `manifests/kafka-service-broker.yml` now deploys Apache Kafka v1.0.0.

## Different upstream releases

The deployemnt manifest has also switched from bigdata-boshrelease to using two smaller, dedicated releases:

* https://github.com/cppforlife/zookeeper-release
* https://github.com/cloudfoundry-community/kafka-boshrelease

## Link to Cloud Foundry admin user

This release includes an experiment to allow a deployment discover a Cloud Foundry admin user via BOSH links.

Currently no Cloud Foundry release includes any links to expose the URL + admin credentials. So, I've created a new job https://github.com/cloudfoundry-community/collection-of-pullrequests-boshrelease/tree/master/jobs/cf-admin-user that you can add to your CF `api` instance group:

```
- type: replace
  path: /instance_groups/name=api/jobs/-
  value:
    name: cf-admin-user
    release: collection-of-pullrequests
    provides:
      cf-admin-user:
        as: cf-admin-user
        shared: true
    properties:
      api_url: "https://api.((system_domain))"
      ca_cert: "((router_ssl.certificate))"
      admin_username: admin
      admin_password: "((cf_admin_password))"

- type: replace
  path: /releases/-
  value:
    name: collection-of-pullrequests
    version: 1.0.0
    url: https://github.com/cloudfoundry-community/collection-of-pullrequests-boshrelease/releases/download/v1.0.0/collection-of-pullrequests-1.0.0.tgz
    sha1: 0212de66395208486749a054014458f0483399f8
```

After updating your CF deployment, a link `cf-admin-user` will be available to all of ther deployments.

Including your `kafka-service-broker` deployment!

You can now skip the malarky of figuring out your CF admin information and just use the link. This has been made even simpler with an operator file:

```
bosh deploy manifests/kafka-service-broker.yml -o manifests/operators/cf-admin-user.yml
```
