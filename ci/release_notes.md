The is a major change to the boshrelease, the method of deployment of the broker, the set of operator files, and removal of dependencies. It should be a drop-in upgrade, even with the removal of the redis dependency, but I've not yet confirmed this. Please contact me if you are running `kafka-service-broker-boshrelease` in production and we can discuss the upgrade.

The broker is now written in golang https://github.com/starkandwayne/kafka-service-broker and instead of using Redis to store information about the service instances, it infers the existence of a service instance by looking into ZooKeeper. This allows us to remove Redis as a dependency.

Removing the redis dependency, and no longer running the broker as a java/spring app, means that it is much easier to just run the broker via BOSH directly; rather than deploy it to Cloud Foundry.

So now there is a `kafka-service-broker` job that runs the broker. The configuration properties are the same as for the old broker-deploy errand (which has no been removed).

This job also allows you to modify the `/v2/catalog` JSON entirely or just to modify the GUID/names. I added this mostly for me - to allow multiple deployments to be registered against a single CF in our test environments.

The base deployment manifest `manifests/kafka-service-broker.yml` no longer assumes you will be deploying/registering the broker with Cloud Foundry. Instead you are welcome to deploy the broker + kafka/zookeeper and never use Cloud Foundry. Perhaps use https://github.com/starkandwayne/eden CLI to interact with the broker.

The instructions in the README currently do assume that you will be registering the broker to CF and demonstrates the new `-o manifests/operators/cf-integration.yml` operator file.

The base manifest also adds a `sanity-test` errand which provisions/binds/unbinds/deprovisions. It does not yet actually interact with the kafka cluster.

```
bosh run-errand sanity-test
```
