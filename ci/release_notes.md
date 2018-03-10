* Renaming to Stark & Wayne kStreams in catalog descriptions
* Catalog enables shareable services by default. See:
  * https://docs.cloudfoundry.org/devguide/services/sharing-instances.html
  * https://docs.cloudfoundry.org/services/enable-sharing.html
* Manifest and operator files have switched to `vm_resources` instead of `vm_type: default`
* `kafka-service-broker` now bundles two sets of sanity test subcommands, which are being used in `sanity-test` errand
* Bumped kafka-boshrelease to v1.1.2
