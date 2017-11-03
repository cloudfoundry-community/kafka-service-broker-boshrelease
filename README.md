# BOSH release for kafka-service-broker

This BOSH release and deployment manifest deploy a cluster of kafka/kafka manager/zookeeper, plus a Cloud Foundry service broker (running on Cloud Foundry).

## Install

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=kafka-service-broker

# name of cf deployment in BOSH, e.g. 'cf'
cf_deployment=cf

system_domain=$(bosh -d $cf_deployment manifest | bosh int - --path /instance_groups/name=api/jobs/name=cloud_controller_ng/properties/system_domain)
skip_verify=$(bosh -d $cf_deployment manifest | bosh int - --path /instance_groups/name=api/jobs/name=cloud_controller_ng/properties/ssl/skip_cert_verify)
admin_password=$(bosh -d $cf_deployment manifest | bosh int - --path /instance_groups/name=uaa/jobs/name=uaa/properties/uaa/scim/users/name=admin/password)

bosh deploy manifests/kafka-service-broker.yml \
  --vars-store tmp/creds.yml \
  -v cf-route=kafka-service-broker.$system_domain \
  -v cf-api-url=https://api.$system_domain \
  -v cf-skip-ssl-validation=$skip_verify \
  -v cf-admin-username=admin \
  -v "cf-admin-password=$admin_password"

bosh run-errand broker-deploy
bosh run-errand broker-registrar
```

The `cf-*` variables are used to:
1. Deploy the `kafka-service-broker` to the `system/starkandwayne-kafka` org/space.
2. Register the broker as a service broker

If your BOSH has Credhub, then you can omit `--vars-store` flag. It is used to generate any passwords/credentials/certificates required by `manifests/kafka-service-broker.yml`.

This deployment assumes you have Dingo Redis installed (`dingo-redis/ready` service/plan). If you are using [`docker-broker-deployment`](https://github.com/cloudfoundry-community/docker-broker-deployment) to deploy `redis32/free`, the include the `manifests/operators/redis32-free.yml` operator patch file in your deploy command:

```
bosh deploy manifests/kafka-service-broker.yml \
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
bosh run-errand broker-deregistrar
bosh run-errand broker-delete
bosh delete-deployment
```

## Issues


### "Cannot get Jedis connection"

When creating your first service instance, you get an internal Stark & Wayne Kafka broker error: `Cannot get Jedis connection`.

```
$ cf create-service starkandwayne-kafka topic test
Creating service instance test in org system / space starkandwayne-kafka as admin...
FAILED
Server error, status code: 502, error code: 10001, message: Service broker error: Cannot get Jedis connection; nested exception is redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
```

This probably means that the `starkandwayne-kafka-broker` application running on Cloud Foundry does not have network access to the Redis service instance.

1. Create security group JSON file, or use the insecure sample `src/everywhere.json`. Preferrably now is the time to [learn more about Application Security Groups](https://docs.cloudfoundry.org/concepts/asg.html).

2. Update `public_networks` security group

    ```
    cf update-security-group public_networks src/everywhere.json
    ```

3. Restart `starkandwayne-kafka-broker` application to update its internal networking permissions

    ```
    cf restart starkandwayne-kafka-broker
    ```

4. Try provisioning service instance again

    ```
    cf create-service starkandwayne-kafka topic test
    ```
