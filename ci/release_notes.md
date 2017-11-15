* New `shared` plan which allows Kafka producers or an orchestrator will create Kafka topics on demand. Unlike the `topic` plan, `shared` does not provide a pre-created Kafka topic, nor does the `uri` include a topic in the path. Please use the `topicNamePrefix` attribute as the prefix for all topic names. These topic names will be deleted by Deprovision (see below).

  The credentials for the new `shared` topic look like:

  ```json
  {
    "hostname": "10.244.0.146:9092,10.244.0.145:9092,10.244.0.144:9092",
    "topicNamePrefix": "69c87ea6-02bd-4657-967b-924c64b6aea1",
    "uri": "kafka://10.244.0.146:9092,10.244.0.145:9092,10.244.0.144:9092",
    "zkPeers": "10.244.0.141:2181,10.244.0.142:2181,10.244.0.143:2181"
  }
  ```

* Deprovision now deletes ALL topics that have the Service ID as a prefix. This means that you can use `topicName` as a prefix and dynamically create any number of topics that you need. Deprovision will clean them up with they are all named with `topicName` (the Service ID value) as the prefix.

  Remember, your Kafka `server.properties` needs to have `delete.topic.enable=true` configured to allow topics to be deleted. If `delete.topic.enable=false` then Deprovision will quietly ignore deleting topics and complete successfully.

  The property `delete.topic.enable=true` is already enabled in the `manifests/kafka-service-broker.yml` via the `kafka-boshrelease` defaults.

* `sanity-test` errand uses `eden` CLI to show catalog/provision/bind/unbind/deprovision against a service broker for both service plans
