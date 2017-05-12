# BOSH release for kafka-service-broker

This BOSH release and deployment manifest deploy a cluster of kafka/kafka manager/zookeeper, plus a Cloud Foundry service broker (running on Cloud Foundry).

## Install

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka-service-broker
bosh2 deploy manifests/kafka-service-broker.yml
bosh2 run-errand broker-deploy
bosh2 run-errand broker-registrar
```

### Uninstall

To uninstall the service broker and kafka/zookeeper clusters:

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka-service-broker
bosh2 run-errand broker-deregistrar
bosh2 run-errand broker-delete
bosh2 delete-deployment
```

## Issues


### Error: Cannot get Jedis connection

When creating your first service instance, you get an internal Dingo Kafka broker error:

```
$ cf create-service dingo-kafka topic test
Creating service instance test in org system / space dingo-kafka as admin...
FAILED
Server error, status code: 502, error code: 10001, message: Service broker error: Cannot get Jedis connection; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
```

This probably means that the `dingo-kafka-broker` application running on Cloud Foundry does not have network access to the Redis service instance.

1. Create security group JSON file, or use the insecure sample `src/everywhere.json`. Preferrably now is the time to [learn more about Application Security Groups](https://docs.cloudfoundry.org/concepts/asg.html).
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
