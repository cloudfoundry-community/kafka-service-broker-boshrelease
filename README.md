# BOSH release for kafka-service-broker

This BOSH release and deployment manifest deploy a cluster of kafka/kafka manager/zookeeper, plus a Cloud Foundry service broker (running on Cloud Foundry).

## Install

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka-service-broker
bosh2 deploy manifests/kafka-service-broker.yml \
  --vars-store tmp/creds.yml \
  -v cf-system-domain=... \
  -v cf-api-url=... \
  -v cf-admin-username=... \
  -v cf-admin-password=... \
  -v cf-skip-ssl-validation=...

bosh2 run-errand broker-deploy
bosh2 run-errand broker-registrar
```

The `cf-*` variables are used to:
1. Deploy the `kafka-service-broker` to the `system/dingo-kafka` org/space.
2. Register the broker as a service broker

If your BOSH has Credhub, then you can omit `--vars-store` flag. It is used to generate any passwords/credentials/certificates required by `manifests/kafka-service-broker.yml`.

This deployment assumes you have Dingo Redis installed (`dingo-redis/ready` service/plan). If you are using [`docker-broker-deployment`](https://github.com/cloudfoundry-community/docker-broker-deployment) to deploy `redis32/free`, the include the `manifests/operators/redis32-free.yml` operator patch file in your deploy command:

```
bosh2 deploy manifests/kafka-service-broker.yml \
  -o manifests/operators/redis32-free.yml \
  --vars-store tmp/creds.yml \
  ...
```

Create your own `redis.yml` operator file if you have a different Redis service broker installed.

### Uninstall

To uninstall the service broker and kafka/zookeeper clusters=... \

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka-service-broker
bosh2 run-errand broker-deregistrar
bosh2 run-errand broker-delete
bosh2 delete-deployment
```

## Issues


### Error=... \

When creating your first service instance, you get an internal Dingo Kafka broker error=... \

```
$ cf create-service dingo-kafka topic test
Creating service instance test in org system / space dingo-kafka as admin...
FAILED
Server error, status code=... \
```

This probably means that the `dingo-kafka-broker` application running on Cloud Foundry does not have network access to the Redis service instance.

1. Create security group JSON file, or use the insecure sample `src/everywhere.json`. Preferrably now is the time to [learn more about Application Security Groups](https=... \
2. Update `public_networks` security group

    ```
    cf update-security-group public_networks src/everywhere.json
    ```

3. Restart `dingo-kafka-broker` application to update its internal networking permissions

    ```
    cf restart dingo-kafka-broker
    ```

4. Try provisioning service instance again

    ```
    cf create-service dingo-kafka topic test
    ```
